# reClaw Architecture

**Purpose:** Language-agnostic system design for reimplementation

---

## 1. Core Philosophy

- **Language-agnostic** - Core concepts work in any language
- **Interface-driven** - Define interfaces, implement in your language
- **Pattern-first** - Show patterns, not syntax

---

## 2. System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              reClaw System                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                               │
│  │   TUI    │    │  Web UI  │    │   API    │                               │
│  │ (CLI)    │    │ (Browser)│    │ Clients  │                               │
│  └────┬─────┘    └────┬─────┘    └────┬─────┘                               │
│       │               │               │                                      │
│       └───────────────┼───────────────┘                                      │
│                       ▼                                                      │
│              ┌─────────────────┐                                             │
│              │  Gateway Server │  WebSocket (port 18789)                     │
│              │   (WebSocket)   │                                             │
│              └────────┬────────┘                                             │
│                       │                                                      │
│  ┌────────────────────┼──────────────────────────────────────────────────┐   │
│  │                    │                                                  │   │
│  │  ┌─────────────┐   │   ┌─────────────┐    ┌─────────────┐            │   │
│  │  │  Channel    │   │   │   Message   │    │    Agent    │            │   │
│  │  │  Adapters   │◄──┴──►│   Router    │───▶│ Orchestrator│            │   │
│  │  └─────────────┘       └──────┬──────┘    └──────┬──────┘            │   │
│  │                               │                  │                   │   │
│  │            ┌──────────────────┼──────────────────┘                   │   │
│  │            ▼                  ▼                                      │   │
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐              │   │
│  │  │   Memory    │◄──►│    Tool     │◄──►│    Voice    │              │   │
│  │  │   Manager   │    │  Registry   │    │   Adapter   │              │   │
│  │  └─────────────┘    └─────────────┘    └─────────────┘              │   │
│  │                               │                                       │   │
│  │                               ▼                                       │   │
│  │                     ┌─────────────┐                                   │   │
│  │                     │    Config  │                                   │   │
│  │                     │   Loader   │                                   │   │
│  │                     └─────────────┘                                   │   │
│  │                                                                       │   │
│  └───────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Components (Language-Agnostic)

### 3.1 Gateway Server

**Responsibility:** WebSocket server for client connections and message routing

**Interface (pseudocode):**

```
class GatewayServer:
    async start(host: string, port: int) -> void
    async stop() -> void
    async on_connect(websocket: WebSocket, client_info: dict) -> ClientConnection
    async route_to_agent(session_id: string, message: dict) -> void
    async route_to_client(client_id: string, message: dict) -> void
    async create_session(agent_id: str, session_key: str) -> Session
    async list_agents() -> list[AgentInfo]
```

**Key Features:**
- WebSocket protocol for real-time communication
- Session management (`agent:{agentId}:{sessionKey}`)
- Authentication (token or password)
- Multi-client support (TUI, Web UI, API clients)

### 3.2 TUI (Terminal UI)

**Responsibility:** Interactive terminal interface for agent chat

**Interface (pseudocode):**

```
class TUI:
    async start() -> void
    async stop() -> void
    async connect(url: str, auth: dict) -> void
    render_chat_log() -> void
    render_status_bar() -> void
    async send_message(text: str) -> void
    async execute_local_command(command: str) -> void
    open_model_picker() -> void
    open_agent_picker() -> void
    open_session_picker() -> void
```

**Key Features:**
- Connects to Gateway via WebSocket
- Streaming response display
- Tool cards (expandable)
- Slash commands (`/agent`, `/session`, `/deliver`, etc.)
- Local shell execution (`!command`)

### 3.3 Channel Adapter

**Responsibility:** Bridge to messaging platforms

**Interface (pseudocode):**

```
class ChannelAdapter:
    async connect() -> void
    async disconnect() -> void
    async send_message(target: string, content: string, options?: dict) -> message_id: string
    async send_voice(target: string, audio_data: bytes) -> message_id: string
    on_message(callback: function)
    on_voice(callback: function)
    on_error(callback: function)
```

**Methods to implement per channel:**
- Connect to platform
- Receive messages → normalize
- Send messages → format for platform

### 3.4 Message Router

**Responsibility:** Route normalized messages to agent pipeline

**Interface:**

```
class MessageRouter:
    async route(message: NormalizedMessage) -> void
    async enqueue(message: NormalizedMessage, priority: int) -> void
    async dequeue() -> NormalizedMessage | null
    set_rate_limiter(channel_id: string, limit: RateLimit) -> void
```

### 3.5 Agent Orchestrator

**Responsibility:** Manage agent lifecycle and message processing

**Interface:**

```
class AgentOrchestrator:
    async spawn_subagent(task: Task, config: AgentConfig) -> SubAgent
    async get_subagent_status(id: string) -> AgentStatus
    async kill_subagent(id: string) -> void
    async list_subagents() -> SubAgent[]
    async send_to_main(session: string, message: Message) -> void
```

### 3.6 Memory Manager

**Responsibility:** Store and retrieve conversation history

**Interface:**

```
class MemoryManager:
    # Short-term (in-memory)
    load_short_term(session_id: string) -> Message[]
    save_short_term(session_id: string, messages: Message[]) -> void
    
    # Long-term (disk/database)
    load_long_term(session_id: string, query?: string) -> MemoryEntry[]
    save_long_term(entry: MemoryEntry) -> void
    search(query: string, options?: SearchOptions) -> SearchResult[]
    
    # Maintenance
    prune(before_date: Date) -> count: int
    get_stats() -> MemoryStats
```

### 3.7 Voice Adapter

**Responsibility:** Handle voice I/O (TTS, STT, phone calls)

**Interface:**

```
class VoiceAdapter:
    # Text-to-Speech
    async speak(text: string, options?: TTSOptions) -> audio: bytes
    
    # Speech-to-Text
    async transcribe(audio: bytes, options?: STTOptions) -> text: string
    
    # Phone calls
    async handle_incoming_call(webhook: dict) -> CallSession
    async make_call(to: string, twiml: str) -> CallSession
    async stream_audio(call_id: string, audio: bytes) -> void
```

### 3.8 Config Loader

**Responsibility:** Load and validate configuration

**Interface:**

```
class ConfigLoader:
    async load(path: string) -> ReclawConfig
    watch(callback: function) -> void
    unwatch() -> void
    resolve_env_vars(config: dict) -> dict
    validate(config: dict) -> ValidationResult
```

---

## 4. Data Structures

### 4.1 Normalized Message

All channel adapters output this format:

```
NormalizedMessage {
    id: string              # "channel_type:original_id"
    channel_type: string   # discord, telegram, etc.
    channel_id: string      # "channel_type:channel_id"
    author_id: string       # "channel_type:user_id"
    author_name: string
    content: string
    timestamp: string      # ISO-8601
    reply_to?: string
    attachments?: Attachment[]
    is_group?: boolean
    group_subject?: string
    was_mentioned?: boolean
}
```

### 4.2 Session

```
Session {
    id: string              # "agent:chat:subagent:task"
    parent_id?: string
    channel_id: string
    user_id: string
    agent_id: string
    model: string
    messages: Message[]
    context: dict
    status: SessionStatus
    created_at: DateTime
    updated_at: DateTime
}
```

### 4.3 Memory Entry

```
MemoryEntry {
    id: string
    session_id: string
    type: "conversation" | "learned" | "decision" | "fact"
    title: string
    content: string
    timestamp: DateTime
    importance: "low" | "normal" | "high" | "critical"
    tags: string[]
}
```

---

## 5. Language Tips

### Python Implementation
- Use `asyncio` for async operations
- Use `dataclasses` for data structures
- Use `pydantic` for config validation
- Use `python-dotenv` for env vars

### Go Implementation
- Use goroutines for concurrency
- Use interfaces for abstractions
- Use `godotenv` for env vars
- Use channels for message passing

### TypeScript/JavaScript Implementation
- Use `async/await` consistently
- Use TypeScript interfaces for type safety
- Use event emitters for callbacks

### Rust Implementation
- Use `async/await` with Tokio
- Use traits for abstractions
- Use `dotenv` crate for env vars

---

## 6. Storage Options

### File-Based (Simplest)
```
reclaw/
├── config.yaml
├── sessions/
│   ├── store.json
│   └── transcripts/
├── memory/
│   ├── YYYY-MM-DD.md
│   └── MEMORY.md
└── logs/
```

### SQLite (Better Querying)
```sql
CREATE TABLE sessions (...);
CREATE TABLE messages (...);
CREATE TABLE memory (...);
```

### PostgreSQL (Production)
- Use for high concurrency
- Add full-text search
- Consider vector extensions for semantic search

---

## 7. Key Design Patterns

### Adapter Pattern
Each channel is an adapter implementing the same interface.

### Observer Pattern
Channel adapters emit events; other components subscribe.

### Factory Pattern
Create channel adapters based on config.

### Strategy Pattern
Swap TTS/STT providers without changing code.

### Repository Pattern
Abstract storage (file, SQLite, PostgreSQL).

---

## 8. Dependencies by Component

| Component | What You Need |
|-----------|---------------|
| Channel Adapters | Platform SDKs (discord.js, grammY, etc.) |
| Voice | TTS/STT provider APIs |
| Memory | Vector DB (optional), file I/O |
| Config | YAML parser, env var handling |
| Logging | Structured logger |

---

## 9. Minimal Viable Implementation

For a barebones reClaw, implement:

1. **Config Loader** - Read YAML + env vars
2. **Session Manager** - Track active sessions
3. **Channel Adapter** - One channel (Discord)
4. **Agent** - Call LLM API directly (skip sub-agents initially)
5. **Memory** - Just in-memory (skip disk)

That's it! ~200-300 lines of code to get something working.

---

## 10. File Locations Reference

| Need | Check |
|------|-------|
| Architecture (this) | ARCHITECTURE.md |
| Protocol formats | impl/PROTOCOLS.md |
| Gateway server | impl/GATEWAY.md |
| TUI client | impl/TUI.md |
| Channel details | impl/CHANNELS.md |
| Voice details | impl/VOICE.md |
| Agent details | impl/AGENTS.md |
| Memory details | impl/MEMORY.md |
| Config schema | impl/CONFIG.md |
| File formats | impl/DATA_FORMATS.md |