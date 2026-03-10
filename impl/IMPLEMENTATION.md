# reClaw Implementation Notes

Based on analysis of OpenClaw's implementation, this document captures implementation patterns and extensions beyond the core SPEC.md.

---

## Channel Integration Extensions

### Multi-Account Support

```yaml
channels:
  discord:
    accounts:
      default:
        token: $DISCORD_TOKEN
      bot-2:
        token: $DISCORD_TOKEN_2
```

### DM Policy Configuration

```yaml
channels:
  discord:
    dmPolicy: pairing    # pairing | allowlist | open | disabled
    groupPolicy: open   # open | allowlist
    allowFrom:          # Whitelist specific senders
      - "+15555550123"
      - "<@123456789>"
```

### Echo Prevention

```yaml
channels:
  telegram:
    echoPrevention:
      enabled: true
      trackWindow: 100
```

### Message Chunking

```yaml
channels:
  whatsapp:
    textChunkLimit: 4096
    chunkMode: paragraph  # length | paragraph | fence
    markdownToWhatsApp: true
```

---

## Agent Orchestration Extensions

### Hierarchical Session Keys

```
agent:{agentId}:main                    # Main session
agent:{agentId}:subagent:{task}         # Subagent session
agent:{agentId}:discord:default:group:{id}  # Group session
agent:{agentId}:discord:default:direct:{peer}  # DM
```

### ACP (Agent Client Protocol)

The subagent communication protocol:

1. **initialize** - Client announces itself
2. **thread/start** - Start new conversation thread
3. **turn/start** - Execute a turn with prompt
4. **Session updates** - Stream progress, tool calls, results

### Queue Modes

```yaml
agents:
  queue:
    mode: steer          # collect | steer | followup | steer+backlog
    debounceMs: 500
    cap: 50
    dropPolicy: old     # old | new | summarize
```

### Tool Permissions

```yaml
agents:
  main:
    toolPolicy:
      allowlist:
        - read
        - web_search
        - web_fetch
  subagents:
    toolPolicy:
      allowlist:
        - read
        - write
        - exec
```

---

## Voice Integration Extensions

### Voice Configuration

```yaml
voice:
  default_provider: edge
  default_stt: azure
  
  providers:
    azure:
      key: $AZURE_SPEECH_KEY
      region: westus2
      voice: en-US-NovaMultilingualNeural
    
    edge_tts:
      enabled: true
      voice: en-US-JennyNeural
    
    elevenlabs:
      api_key: $ELEVENLABS_API_KEY
      voice_id: xxx
    
    sherpa_onnx:
      enabled: true
      model_path: /path/to/model

  phone:
    twilio:
      account_sid: $TWILIO_ACCOUNT_SID
      auth_token: $TWILIO_AUTH_TOKEN
      phone_number: $TWILIO_PHONE_NUMBER
```

### VoiceCall Entity

```yaml
# Voice call state tracking
voice_calls:
  active: []
  recent:
    - id: call_123
      channel: telegram
      caller_id: "+15551234567"
      status: completed
      started_at: "2026-03-09T10:00:00Z"
      ended_at: "2026-03-09T10:05:00Z"
```

---

## Memory Management Extensions

### Configuration

```yaml
memory:
  short_term:
    max_messages: 100
    ttl_minutes: 60
  
  long_term:
    enabled: true
    path: ./memory
    max_size_mb: 500
    
  maintenance:
    mode: enforce  # warn | enforce
    max_entries: 500
    prune_after_days: 30
```

### MEMORY.md Format

```markdown
# Memory Entry - <Title>

## Date: 2026-01-21

### What Happened
<Description>

### Lesson Learned
<Insight>

### Action Items
- [ ] Thing to do
```

---

## Session Management

### Session Reset Policies

```yaml
session:
  reset:
    mode: daily        # daily | idle | never
    at_hour: 4        # For daily mode
    idle_minutes: 60  # For idle mode
  
  maintenance:
    mode: enforce     # warn | enforce
    max_entries: 500
    max_disk_mb: 100
    prune_after_days: 30
```

### Session Storage Format

```typescript
// JSON store + JSONL transcripts
sessions/
  store.json          // Main session metadata
  transcripts/
    agent_main_main.jsonl   // Conversation history
    agent_main_subagent_xxx.jsonl
```

---

## Tools Integration

### Core Tools (Always Available)

| Tool | Description |
|------|-------------|
| read | Read file contents |
| write | Write file contents |
| exec | Execute shell commands |
| web_fetch | Fetch URL content (HTTP GET + HTML extraction) |
| message | Send messages via channels |
| tts | Text-to-speech |
| sessions_spawn | Spawn sub-agents |
| sessions_list | List active sessions |

### Extension Tools (Require Setup)

| Tool | Description | Extension |
|------|-------------|-----------|
| web_search | Web search (Brave, Perplexity, Gemini, etc.) | `extensions/web-search.md` |
| browser | Browser automation (Playwright/Puppeteer) | `extensions/browser.md` |

### Tool Permission Model

```yaml
tools:
  read:
    path_scope: ./workspace  # Restrict to workspace
    allowed_extensions: ["*"]
  
  exec:
    allowed_commands: ["git", "npm", "python3"]
    timeout_seconds: 300
```

---

## Plugin Hooks

```yaml
hooks:
  before_model_resolve:
    - script: ./hooks/filter.py
  before_prompt_build:
    - script: ./hooks/context.py
  message_received:
    - script: ./hooks/log.py
  message_sending:
    - script: ./hooks/format.py
  before_tool_call:
    - script: ./hooks/sandbox.py
```

---

## Migration from OpenClaw

### Config Mapping (JSON → YAML)

OpenClaw's `openclaw.json` → reclaw's `config.yaml`:

```yaml
# OpenClaw (JSON)
{
  "agents": {
    "defaults": {
      "model": {"primary": "kimi-coding/k2p5"},
      "workspace": "/path/to/workspace"
    }
  },
  "channels": {
    "discord": {"enabled": true, "token": "..."}
  }
}

# reClaw (YAML)
version: "1.0"

agents:
  default_model: kimi-coding/k2p5
  workspace: /path/to/workspace

channels:
  discord:
    enabled: true
    token: $DISCORD_TOKEN
```

### Key Differences

| Aspect | OpenClaw | reClaw |
|--------|----------|--------|
| Format | JSON | YAML |
| Secrets | Hardcoded or env | $VAR indirection |
| Versioning | None | Explicit version field |
| Voice config | plugins.entries | Top-level voice section |
| Memory config | None | Explicit memory section |

---

## Files to Implement

```
reclaw/
├── SPEC.md                    # This specification
├── IMPLEMENTATION.md          # Implementation patterns (this file)
├── config.yaml.example        # Example configuration
├── WORKFLOW.md.example       # Example workflow
├── adapters/
│   ├── channels/
│   │   ├── discord.py
│   │   ├── telegram.py
│   │   ├── signal.py
│   │   ├── whatsapp.py
│   │   └── slack.py
│   └── voice/
│       ├── twilio.py
│       ├── tts/
│       │   ├── azure.py
│       │   ├── edge.py
│       │   └── elevenlabs.py
│       └── stt/
│           ├── azure.py
│           └── whisper.py
├── core/
│   ├── orchestrator.py
│   ├── session.py
│   ├── memory.py
│   └── tools.py
└── cli/
    └── reclaw.py
```