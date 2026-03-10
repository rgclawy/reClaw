# Data Formats

**Purpose:** Exact file formats and storage structures

---

## 1. Session Storage

### 1.1 Session Store (JSON)

Location: `sessions/store.json`

```json
{
  "version": 1,
  "sessions": {
    "agent:main:discord:default:direct:123456789": {
      "id": "agent:main:discord:default:direct:123456789",
      "parentId": null,
      "channelId": "discord:channel:987654321",
      "userId": "discord:user:111222333",
      "agentId": "main",
      "model": "kimi-coding/k2p5",
      "status": "active",
      "createdAt": "2026-03-09T10:00:00Z",
      "updatedAt": "2026-03-09T10:05:00Z",
      "metadata": {
        "lastMessageId": "discord:123456789012345678",
        "messageCount": 10
      }
    }
  },
  "agents": {
    "agent:main:subagent:task-123": {
      "id": "agent:main:subagent:task-123",
      "parentId": "agent:main:discord:default:direct:123456789",
      "status": "completed",
      "model": "kimi-coding/k2p5",
      "createdAt": "2026-03-09T10:00:00Z",
      "completedAt": "2026-03-09T10:00:05Z",
      "result": "The weather is sunny.",
      "error": null
    }
  }
}
```

### 1.2 Session Transcript (JSONL)

Location: `sessions/transcripts/agent_main_discord_123456789.jsonl`

```jsonl
{"role": "user", "content": "Hello", "timestamp": "2026-03-09T10:00:00Z"}
{"role": "assistant", "content": "Hi! How can I help?", "timestamp": "2026-03-09T10:00:01Z"}
{"role": "user", "content": "What's the weather?", "timestamp": "2026-03-09T10:00:02Z"}
{"role": "assistant", "content": "Let me check that.", "timestamp": "2026-03-09T10:00:03Z", "tool_calls": [{"tool": "web_search", "params": {"query": "weather"}}]}
{"role": "tool", "tool": "web_search", "result": "Sunny, 72°F", "timestamp": "2026-03-09T10:00:04Z"}
{"role": "assistant", "content": "It's sunny and 72°F!", "timestamp": "2026-03-09T10:00:05Z"}
```

---

## 2. Memory Storage

### 2.1 Daily Memory File

Location: `memory/2026-03-09.md`

```markdown
# Memory Entry - User preference for weather

## Date: 2026-03-09

### Session
agent:main:discord:default:direct:123456789

### Type
learned

### Content
User prefers Celsius over Fahrenheit. Remember to convert temperatures.

### Importance
normal

### Tags
preference, weather, temperature

---

# Memory Entry - Project context

## Date: 2026-03-09

### Session
agent:main:telegram:default:chat:987654321

### Type
conversation

### Content
User: Working on the reclaw project
Assistant: Great! It's an interesting project. What's the status?

### Importance
normal

### Tags
reclaw, project

---
```

### 2.2 Curated Memory

Location: `memory/MEMORY.md`

```markdown
# Memory Entry - Core lessons

## Date: 2026-01-21

### Session
agent:main:discord

### Type
learned

### Content
IMPORTANT: Never log API keys or tokens. Always use environment variables ($VAR) for secrets.

### Importance
critical

### Tags
security, secrets

---

# Memory Entry - User preferences

## Date: 2026-01-22

### Session
agent:main

### Type
learned

### Content
User prefers concise responses. Avoid long explanations unless asked.

### Importance
high

### Tags
preference, communication
```

---

## 3. Voice Storage

### 3.1 Call Transcript

Location: `voice/transcripts/call-2026-03-09-12345.json`

```json
{
  "id": "call-2026-03-09-12345",
  "provider": "twilio",
  "direction": "inbound",
  "caller_id": "+15555550123",
  "status": "completed",
  "started_at": "2026-03-09T10:00:00Z",
  "ended_at": "2026-03-09T10:05:00Z",
  "duration_seconds": 300,
  "transcript": [
    {
      "timestamp": "2026-03-09T10:00:00Z",
      "speaker": "caller",
      "text": "Hello?"
    },
    {
      "timestamp": "2026-03-09T10:00:01Z",
      "speaker": "assistant",
      "text": "Hello! What can I help you with?"
    },
    {
      "timestamp": "2026-03-09T10:00:05Z",
      "speaker": "caller",
      "text": "What's the weather like?"
    }
  ]
}
```

### 3.2 TTS Output

Location: `voice/output/2026-03-09/response-12345.mp3`

(Standard MP3 audio file)

---

## 4. Logs

### 4.1 Main Log

Location: `logs/reclaw-2026-03-09.log`

```
2026-03-09T10:00:00.000Z INFO [agent-orchestrator] Sub-agent spawned id=agent:main:subagent:task-123 model=kimi-coding/k2p5
2026-03-09T10:00:01.000Z DEBUG [channel-discord] Message received author=discord:user:111222333 content="Hello"
2026-03-09T10:00:01.000Z INFO [memory] Loading context session=agent:main:discord:default
2026-03-09T10:00:02.000Z DEBUG [tool] Executing tool=web_search query="weather"
2026-03-09T10:00:03.000Z INFO [tool] Tool completed tool=web_search duration=1000ms
2026-03-09T10:00:04.000Z INFO [channel-discord] Sending message channel=discord:channel:987654321
```

### 4.2 JSON Structured Log

```json
{"timestamp":"2026-03-09T10:00:00.000Z","level":"info","session_id":"agent:main:subagent:task-123","channel_id":"discord:123456789","user_id":"discord:111222333","component":"agent-orchestrator","message":"Sub-agent spawned","metadata":{"task_id":"task-123","model":"kimi-coding/k2p5"}}
{"timestamp":"2026-03-09T10:00:01.000Z","level":"debug","session_id":"agent:main:discord:default","channel_id":"discord:123456789","component":"channel-discord","message":"Message received","metadata":{"author":"discord:user:111222333","content":"Hello"}}
```

---

## 5. Configuration

### 5.1 Config File

Location: `config.yaml` (user config)

See **CONFIG.md** for full schema.

### 5.2 Cache

Location: `cache/`

```
cache/
├── sessions/           # Session cache
│   └── agent_main_discord_123456789.json
├── embeddings/         # Vector embeddings cache
│   └── embeddings.db
└── http/               # Web fetch cache
    └── example.com_index.html
```

---

## 6. Database Schema (SQLite Alternative)

If using SQLite backend:

```sql
-- Sessions table
CREATE TABLE sessions (
    id TEXT PRIMARY KEY,
    parent_id TEXT,
    channel_id TEXT NOT NULL,
    user_id TEXT NOT NULL,
    agent_id TEXT NOT NULL,
    model TEXT NOT NULL,
    status TEXT NOT NULL,
    created_at INTEGER NOT NULL,
    updated_at INTEGER NOT NULL,
    metadata TEXT
);

-- Messages table
CREATE TABLE messages (
    id TEXT PRIMARY KEY,
    session_id TEXT NOT NULL,
    role TEXT NOT NULL,
    content TEXT NOT NULL,
    timestamp INTEGER NOT NULL,
    metadata TEXT,
    FOREIGN KEY (session_id) REFERENCES sessions(id)
);

-- Memory entries table
CREATE TABLE memory (
    id TEXT PRIMARY KEY,
    session_id TEXT NOT NULL,
    type TEXT NOT NULL,
    title TEXT NOT NULL,
    content TEXT NOT NULL,
    timestamp INTEGER NOT NULL,
    importance TEXT NOT NULL,
    tags TEXT,
    metadata TEXT
);

-- Indexes
CREATE INDEX idx_messages_session ON messages(session_id);
CREATE INDEX idx_memory_session ON memory(session_id);
CREATE INDEX idx_memory_timestamp ON memory(timestamp);
```

---

## 7. API Formats

### 7.1 Status API Response

```json
{
  "status": "healthy",
  "version": "1.0.0",
  "uptime_seconds": 3600,
  "channels": {
    "discord": {
      "status": "connected",
      "latency_ms": 50
    },
    "telegram": {
      "status": "connected"
    }
  },
  "agents": {
    "active": 2,
    "idle": 5
  },
  "sessions": {
    "total": 10,
    "active": 3
  },
  "memory": {
    "short_term_entries": 150,
    "long_term_entries": 500
  }
}
```

### 7.2 Sessions API Response

```json
{
  "sessions": [
    {
      "id": "agent:main:discord:default:direct:123456789",
      "channel_id": "discord:channel:987654321",
      "user_id": "discord:user:111222333",
      "status": "active",
      "message_count": 10,
      "created_at": "2026-03-09T10:00:00Z"
    }
  ]
}
```

---

## 8. Webhook Payloads

### 8.1 Twilio Incoming Call

```
POST /voice/incoming

CallSid: CA1234567890abcdef
CallStatus: ringing
From: +15555550123
To: +15555551234
Direction: inbound
```

### 8.2 Discord Webhook

```json
POST /webhooks/discord/{webhook_id}/{webhook_token}

{
  "id": "123456789012345678",
  "type": 0,
  "channel_id": "987654321012345678",
  "author": {
    "id": "111222333444555666",
    "username": "user",
    "discriminator": "0001",
    "bot": false
  },
  "content": "Hello",
  "timestamp": "2026-03-09T10:00:00.000Z"
}
```

---

## 9. ACP Message Examples

### 9.1 Sub-agent Spawn Request

```json
{
  "type": "initialize",
  "session": "agent:main:subagent:task-123",
  "capabilities": ["read", "web_search"],
  "model": "kimi-coding/k2p5",
  "thinking": "medium"
}
```

### 9.2 Tool Call Request

```json
{
  "type": "tool_call",
  "tool": "web_search",
  "params": {
    "query": "weather"
  },
  "call_id": "call-abc123"
}
```

### 9.3 Turn Response

```json
{
  "type": "turn_complete",
  "turn_id": "turn-1",
  "result": "The weather is sunny and 72°F.",
  "metadata": {
    "tokens_used": 150,
    "duration_ms": 2000
  }
}
```