# Agent Orchestration

**Purpose:** How to implement sub-agents and message processing - language-agnostic patterns

---

## 1. What is Agent Orchestration?

The system that:
1. Receives user messages
2. Spawns sub-agents for background tasks  
3. Keeps main session responsive
4. Coordinates results back to user

```
User Message → Main Session → Spawn Sub-agent → Sub-agent works → Report result → User gets response
                                          ↓
                              Main session stays responsive
```

---

## 2. Interface

```
class AgentOrchestrator:
    
    # Session management
    async create_session(config: SessionConfig) -> Session
    async get_session(id: string) -> Session | null
    async delete_session(id: string) -> void
    
    # Sub-agents
    async spawn_subagent(parent_session: string, task: Task, options?: dict) -> SubAgent
    async get_subagent_status(id: string) -> AgentStatus
    async kill_subagent(id: string) -> void
    async list_subagents() -> list[SubAgent]
    
    # Communication
    async send_to_main(session: string, message: Message) -> void
```

---

## 3. Session

```
Session {
    id: string                    # "agent:chat:subagent:task"
    parent_id: string | null      # Parent session for sub-agents
    channel_id: string            # Where messages go
    user_id: string               # Who sent the message
    agent_id: string              # Which agent config
    model: string                 # LLM model to use
    messages: list[Message]      # Conversation history
    context: dict                 # Session-specific data
    status: SessionStatus         # idle, active, processing
    created_at: datetime
    updated_at: datetime
}
```

---

## 4. Spawning Sub-agents

### 4.1 Pattern

```
async function spawn_subagent(parent_id: str, task: Task, options: dict) -> SubAgent:
    
    # 1. Generate sub-agent ID
    subagent_id = generate_id(parent_id, task.id)
    
    # 2. Check depth limit
    depth = get_subagent_depth(parent_id)
    if depth >= MAX_SPAWN_DEPTH:
        raise Error("Max depth exceeded")
    
    # 3. Check concurrency limit
    active = count_active_subagents(parent_id)
    if active >= MAX_CHILDREN:
        raise Error("Too many children")
    
    # 4. Create session
    session = create_session(subagent_id, parent_id, task)
    
    # 5. Start sub-agent process
    # Could be: child process, thread, async task, remote service
    process = start_subagent_process(session, options)
    
    # 6. Initialize ACP connection
    await process.send({
        type: "initialize",
        session: subagent_id,
        capabilities: options.capabilities or DEFAULT_CAPABILITIES,
        model: options.model or DEFAULT_MODEL
    })
    
    # 7. Send task
    await process.send({
        type: "turn_start",
        prompt: task.prompt,
        turn_id: generate_turn_id()
    })
    
    return SubAgent(id: subagent_id, process: process, status: "running")
```

### 4.2 Depth Tracking

```
function get_subagent_depth(session_id: str) -> int:
    # Count ":subagent:" in the ID
    parts = session_id.split(":")
    count = 0
    for part in parts:
        if part == "subagent":
            count += 1
    return count

# Examples:
# agent:chat:main -> depth 0
# agent:chat:subagent:task -> depth 1
# agent:chat:subagent:task:subagent:nested -> depth 2
```

---

## 5. ACP (Agent Control Protocol)

### 5.1 What is ACP?

Communication between main session and sub-agents using **NDJSON** (newline-delimited JSON).

### 5.2 Message Flow

```
Main Session → Sub-agent:
    {"type": "initialize", "session": "...", "capabilities": [...]}
    {"type": "turn_start", "prompt": "...", "turn_id": "..."}
    {"type": "steer", "message": "..."}          # Optional: guide running agent
    {"type": "interrupt"}                         # Optional: stop current work
    {"type": "close"}                              # Done

Sub-agent → Main Session:
    {"type": "initialized", "session": "...", "agent_id": "..."}
    {"type": "message", "content": "..."}         # Streaming response
    {"type": "tool_call", "tool": "...", "params": {...}}
    {"type": "tool_result", "call_id": "...", "result": "..."}
    {"type": "turn_complete", "turn_id": "...", "result": "..."}
    {"type": "error", "message": "..."}
```

### 5.3 Implementation

```
async function handle_acp_message(connection, message: dict):
    match message.type:
        "message":
            # Stream text to user
            await send_to_user(message.content)
            
        "tool_call":
            # Execute requested tool
            result = await execute_tool(message.tool, message.params)
            
            # Return result
            await connection.send({
                "type": "tool_result",
                "call_id": message.call_id,
                "result": result
            })
            
        "turn_complete":
            # Task done
            await cleanup_subagent(message.turn_id)
```

---

## 6. Queue Management

### 6.1 Multi-Lane Queue

```
                ┌─────────────┐
                │   Main Lane │  (user messages)
                └──────┬──────┘
                       │
    ┌──────────────────┼──────────────────┐
    ▼                  ▼                  ▼
Subagent Lane      Cron Lane         Session Lane
(tasks)            (scheduled)        (async ops)
```

### 6.2 Queue Modes

| Mode | Behavior |
|------|----------|
| `collect` | Batch messages, process together |
| `steer` | Allow real-time guidance to running agent |
| `followup` | Post-completion updates only |
| `steer+backlog` | Both steering and queued messages |

### 6.3 Drop Policies

| Policy | Behavior |
|--------|----------|
| `old` | Drop oldest when full |
| `new` | Drop newest when full |
| `summarize` | Keep summary of dropped |

---

## 7. Tool Permission

### 7.1 Pattern

```
async function request_tool_permission(subagent_id: str, tool: str, params: dict) -> Result:
    
    # Check if tool is auto-allowed
    if tool in AUTO_ALLOW_LIST:
        return await execute_tool(tool, params)
    
    # Check if requires approval
    if tool in REQUIRE_APPROVAL_LIST:
        # Ask user (via main session)
        approved = await ask_user(subagent_id, tool, params)
        
        if approved:
            return await execute_tool(tool, params)
        else:
            return Result(error: "Permission denied")
    
    # Otherwise allow
    return await execute_tool(tool, params)
```

### 7.2 Auto-Allow vs Require-Approval

**Core tools (always auto-allowed):**
```
AUTO_ALLOW = ["read", "web_fetch", "message", "tts"]
REQUIRE_APPROVAL = ["exec", "write", "delete", "sessions_spawn"]
```

**Extension tools:**
- `web_search` - Auto-allowed if enabled (API key required)
- `browser` - Requires approval (high impact)

---

## 8. Session Key Format

```
agent:{agentId}:{channelType}:{accountId}:{sessionType}:{id}

Examples:
agent:main:discord:default:direct:123456789     # Discord DM
agent:main:discord:default:group:987654321      # Discord Group
agent:main:telegram:default:chat:111222333       # Telegram chat
agent:main:subagent:task:task-123               # Sub-agent task
```

---

## 9. Language Tips

### Python
```python
# Async
asyncio, asyncio.create_task

# Subprocess
asyncio.create_subprocess_exec

# Message queue
asyncio.Queue, celery (if needed)

# IPC
asyncio duplex pipes, multiprocessing Pipe
```

### TypeScript
```typescript
// Async
async/await, Promise

// Child process
child_process.spawn, worker_threads

// Message queue
async iterator, rxjs subjects
```

### Go
```go
// Goroutines
go func() {}

// Channels
ch := make(chan Message)

// Subprocess
exec.Command
```

---

## 10. Testing

### Test Spawn
```python
async def test_spawn_subagent():
    orch = AgentOrchestrator(config)
    
    task = Task(prompt="Count to 10")
    subagent = await orch.spawn_subagent("main-session", task)
    
    assert subagent.status == "running"
    assert subagent.id.startswith("main-session:subagent:")
```

### Test ACP
```python
async def test_acp_message():
    # Send initialize
    await connection.send({"type": "initialize", "session": "test"})
    
    # Get response
    response = await wait_for_message(connection)
    assert response.type == "initialized"
```

---

## 11. Reference

See also:
- `ARCHITECTURE.md` - System design
- `PROTOCOLS.md` - Exact ACP message formats
- `CONFIG.md` - Agent configuration