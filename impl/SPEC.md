# reClaw Service Specification

**Status:** Draft v0 (initial)

**Based on:** OpenClaw v2026.3.8 (latest release)

**Purpose:** Define a service that orchestrates AI agents across multiple messaging channels, voice platforms, and integrations — with human-auditable, open specifications.

---

## 1. Problem Statement

Reclaw is a personal AI assistant that runs on your own infrastructure, connecting to various messaging platforms and voice services. It processes messages, executes tasks via sub-agents, and maintains memory of past interactions.

The goal is to make the system **fully auditable** — the specification is open and public, while each user's implementation remains private.

---

## 2. Core Components

### 2.1 Gateway Server

WebSocket server that acts as the central hub for all connections.

| Feature | Status | Default |
|---------|--------|---------|
| WebSocket protocol | ✅ Core | Port 18789 |
| HTTP REST API | ✅ Core | Port 18789 |
| Token auth | ✅ Core | - |
| Password auth | ✅ Core | - |
| Session management | ✅ Core | - |
| Multi-client support | ✅ Core | Unlimited |

**CLI:** `reclaw gateway start`

### 2.2 CLI (Command Line Interface) - Core

Single CLI tool for all reClaw operations.

| Feature | Status | Default |
|---------|--------|---------|
| Gateway control | ✅ Core | start/stop/status |
| Config management | ✅ Core | get/set/validate |
| Agent operations | ✅ Core | run/spawn/list |
| Status/info | ✅ Core | health, version |
| TUI subcommand | Extension | `reclaw tui` |
| Extension commands | Extension | channels, web, voice, etc. |

**CLI:** `reclaw <command>`

Examples:
```bash
reclaw gateway start          # Start Gateway
reclaw agent run "hello"      # Run agent
reclaw tui                    # Launch TUI (if installed)
reclaw channels list          # Extension command
```

**Note:** If TUI or extension dependencies are missing, the CLI shows helpful error messages.

### 2.4 Supported Channels (20+)

### 2.4.1 Core Channels (First-Class Support)

| Channel | Status | Multi-Account | Implementation |
|---------|--------|---------------|----------------|
| Discord | ✅ Full | Yes | discord.js (Gateway + REST) |
| Telegram | ✅ Full | Yes | grammY (Long polling/Webhook) |
| WhatsApp | ✅ Full | Yes | Baileys (Web) |
| Slack | ✅ Full | Yes | @slack/bolt (Socket/HTTP) |
| Signal | ✅ Full | Yes | signal-cli (JSON-RPC + SSE) |
| iMessage | ✅ Full | Yes | BlueBubbles API (macOS) |
| LINE | ✅ Full | Yes | Official SDK |

### 2.4.2 Extension Channels (Plugin-Based)

| Channel | Status | Notes |
|---------|--------|-------|
| Feishu/Lark | ✅ Full | Enterprise messaging |
| Matrix | ✅ Full | Decentralized chat |
| Mattermost | ✅ Full | Team collaboration |
| Zalo | ✅ Full | Vietnam-centric |
| Google Chat | ⚠️ Partial | Basic implementation |
| IRC | ⚠️ Partial | Basic implementation |
| Nostr | ✅ Full | Decentralized social |
| MS Teams | ✅ Full | Enterprise |
| Twitch | ✅ Full | Streaming platform |
| Tlon/Urbit | ✅ Full | Novel social network |
| Synology Chat | ✅ Full | NAS-integrated |
| Nextcloud Talk | ✅ Full | Self-hosted |

### 2.4.3 Voice/Phone Channels

| Channel | Status | Implementation |
|---------|--------|----------------|
| Twilio | ✅ Full | Voice calls |
| Discord Voice | ✅ Full | Voice channels |
| OpenAI Realtime | ✅ Full | Real-time voice |

---

## 3. DM Policy & Access Control

### 3.1 Policy Types

```yaml
channels:
  discord:
    dmPolicy: pairing    # pairing | allowlist | open | disabled
    groupPolicy: open   # open | allowlist
    allowFrom:          # Whitelist specific senders
      - "+15555550123"
      - "<@123456789>"
      - "uuid:xxx"
```

**Policy Types:**
- **pairing** (default): Unknown senders receive expiring pairing code, require approval
- **allowlist**: Only pre-configured IDs can message
- **open**: Anyone can message
- **disabled**: No DMs accepted

### 3.2 Multi-Account Support

```yaml
channels:
  discord:
    accounts:
      default:
        token: $DISCORD_TOKEN
      bot-2:
        token: $DISCORD_TOKEN_2
```

Each account has:
- Separate credentials
- Separate allowlists
- Separate session keys
- Isolated state

### 3.3 Echo Prevention

Multi-layer protection:
1. **Bot flag detection** - Skip messages flagged as from bots
2. **Self-ID tracking** - Skip own user ID
3. **Content hash deduplication** - Skip identical recent messages
4. **Self-chat mode** - Special handling for personal number
5. **Bot-to-bot prevention** - Require @mention in bot-to-bot scenarios

---

## 4. Voice Integration

### 4.1 TTS Providers

| Provider | Status | API Key Required |
|----------|--------|------------------|
| **OpenAI** | ✅ Primary | Yes |
| **Edge TTS** | ✅ Free | No |
| **ElevenLabs** | ✅ Available | Yes |

**Note:** Azure Speech TTS was removed in v2026.3.8

### 4.2 STT Providers

| Provider | Status | Notes |
|----------|--------|-------|
| **OpenAI Whisper** | ✅ Primary | High quality |
| **Deepgram** | ✅ Available | Low latency |
| **Google** | ✅ Available | |
| **Groq** | ✅ Available | Fast inference |
| **MiniMax** | ✅ Available | |
| **Moonshot** | ✅ Available | |

### 4.3 Phone Providers

| Provider | Status |
|----------|--------|
| Twilio | ✅ Full |
| Telnyx | ✅ Available |
| Plivo | ✅ Available |

### 4.4 Voice Features

- **Model-driven TTS**: `[[tts:voice-name]]` directive in prompts
- **Auto-TTS modes**: off / always / inbound / tagged
- **OpenAI Realtime**: Streaming STT with VAD
- **DTMF support**: Phone key press handling
- **Transcript history**: Full call transcript storage

---

## 5. Agent Orchestration (ACP)

### 5.1 Session Keys

Hierarchical format:
```
agent:{agentId}:main                          # Main session
agent:{agentId}:subagent:{task}               # Subagent session
agent:{agentId}:discord:default:group:{id}   # Group session
agent:{agentId}:discord:default:direct:{peer}  # DM session
agent:{agentId}:thread:{topic}                # Thread session
```

### 5.2 ACP Protocol

**Communication:**
- stdio-based (default)
- NDJSON stream
- WebSocket (optional)

**Message Types:**
1. `initialize` - Client announces itself
2. `thread/start` - Start new conversation thread
3. `turn/start` - Execute a turn with prompt
4. `session_update` - Progress updates
5. `tool_permission` - Request tool access

### 5.3 Queue Management

**Multi-lane Architecture:**
- Main lane
- Subagent lane
- Cron lane
- Session lane

**Queue Modes:**
- `collect` - Batch messages
- `steer` - Real-time guidance
- `followup` - Post-completion updates
- `steer+backlog` - Both steering and backlog

**Drop Policies:**
- `old` - Drop oldest
- `new` - Drop newest
- `summarize` - Generate summary of dropped

### 5.4 Concurrency Control

```yaml
agents:
  max_concurrent: 4
  max_spawn_depth: 5        # Nested sub-agent depth
  max_children_per_agent: 3  # Concurrent children
  
  # Per-lane limits
  lane_limits:
    main: 2
    subagent: 4
    cron: 1
```

---

## 6. Memory Management

### 6.1 Short-Term Memory

- In-memory cache with TTL
- Token-based context management
- Pre-compaction flush (silent NO_REPLY turn before context compaction)

### 6.2 Long-Term Memory

**Hybrid Search:**
- Vector embeddings (semantic search)
- BM25 keyword search
- MMR (Maximal Marginal Relevance) for diversity ranking

**Embedding Providers:**
- OpenAI
- Gemini
- Voyage
- Mistral
- Ollama (local)
- Local embeddings

**Temporal Decay:**
- Recency boosting for recent memories
- Relevance scoring combines time + similarity

### 6.3 Memory Storage

```yaml
memory:
  short_term:
    max_messages: 100
    ttl_minutes: 60
  
  long_term:
    enabled: true
    path: ./memory
    backend: qmd  # or: json, sqlite
    
  search:
    provider: hybrid  # vector, bm25, or hybrid
    
  maintenance:
    mode: enforce
    max_entries: 500
    prune_after_days: 30
```

---

## 7. Configuration

### 7.1 File Format

YAML with environment variable indirection:

```yaml
version: "1.0"

channels:
  discord:
    enabled: true
    token: $DISCORD_TOKEN
    
  telegram:
    enabled: true
    bot_token: $TELEGRAM_BOT_TOKEN

voice:
  default_provider: openai
  openai:
    api_key: $OPENAI_API_KEY
    
  twilio:
    account_sid: $TWILIO_ACCOUNT_SID
    auth_token: $TWILIO_AUTH_TOKEN
    phone_number: $TWILIO_PHONE_NUMBER

agents:
  default_model: kimi-coding/k2p5
  max_concurrent: 4
  
session:
  reset:
    mode: daily
    at_hour: 4
    
  maintenance:
    mode: enforce
    max_entries: 500
    max_disk_mb: 100
```

### 7.2 Dynamic Reload

- Watch config files for changes
- Re-apply settings without restart
- Keep last known good config on invalid reload

---

## 8. Observability

### 8.1 Logging

Required context:
- `session_id`
- `channel_id`
- `user_id`
- `timestamp`

### 8.2 Status API

```
GET /api/v1/status      # System status
GET /api/v1/sessions    # Active sessions
GET /api/v1/memory      # Memory stats
GET /api/v1/agents      # Agent status
POST /api/v1/refresh    # Force poll
```

---

## 9. Implementation File Structure

```
reclaw/
├── SPEC.md              # This specification
├── IMPLEMENTATION.md    # Implementation patterns
├── TEST_PLAN.md         # Functional equivalence tests
├── config.yaml.example  # Example configuration
├── WORKFLOW.md          # Custom workflow (user-defined)
├── src/
│   ├── core/            # Orchestrator, session, memory
│   ├── adapters/
│   │   ├── channels/    # Discord, Telegram, etc.
│   │   └── voice/       # Twilio, STT, TTS
│   └── tools/           # Tool registry
└── tests/               # Test suite
```

---

## 10. Protocol Compliance Checklist

To verify reclaw implementation matches OpenClaw OSS:

- [ ] All 20+ channels send/receive messages
- [ ] DM policies (pairing/allowlist/open/disabled) work
- [ ] Multi-account per channel works
- [ ] Echo prevention prevents loops
- [ ] OpenAI TTS generates speech
- [ ] 6+ STT providers transcribe
- [ ] Twilio calls connect and stream
- [ ] ACP protocol communication works
- [ ] Session keys follow hierarchical format
- [ ] Queue modes (collect/steer/followup) work
- [ ] Hybrid memory search returns results
- [ ] Config reloads without restart

---

*This specification is based on OpenClaw v2026.3.8 (latest OSS release)*