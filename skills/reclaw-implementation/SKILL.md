# reClaw Implementation Skill

**Purpose:** Build reClaw from scratch OR evaluate and work with existing implementations

---

## When to Use This Skill

Use this skill when:
- Building reClaw for the first time
- Evaluating an existing reClaw implementation
- Rebuilding/replacing an existing implementation
- Need to understand what's already been built

---

## Step 1: Detect Existing Implementations

**ALWAYS run this first before any implementation work!**

### 1.1 Check for Existing reClaw

Look in the target directory (default: `./reclaw` or user-specified):

```bash
# Markers of an existing reClaw implementation
check_for_reclaw() {
    local dir="$1"
    local found=0
    
    # Config files
    if [ -f "$dir/config.yaml" ] || [ -f "$dir/reclaw.yaml" ]; then
        echo "✓ Found: Config file (config.yaml or reclaw.yaml)"
        found=1
    fi
    
    # Main entry points
    if [ -f "$dir/reclaw.py" ] || [ -f "$dir/main.py" ] || [ -f "$dir/index.ts" ] || [ -f "$dir/main.go" ]; then
        echo "✓ Found: Main entry point"
        found=1
    fi
    
    # Source directory
    if [ -d "$dir/src" ]; then
        echo "✓ Found: Source directory (src/)"
        found=1
    fi
    
    # Project files
    if [ -f "$dir/pyproject.toml" ] || [ -f "$dir/package.json" ] || [ -f "$dir/Cargo.toml" ] || [ -f "$dir/go.mod" ]; then
        echo "✓ Found: Project manifest"
        found=1
    fi
    
    # Memory directory
    if [ -d "$dir/memory" ]; then
        echo "✓ Found: Memory directory"
        found=1
    fi
    
    # Gateway indicator (WebSocket port 18789)
    if grep -r "18789" "$dir" 2>/dev/null | grep -q "websocket\|WebSocket\|ws"; then
        echo "✓ Found: Gateway WebSocket (port 18789)"
        found=1
    fi
    
    return $((1 - found))
}
```

### 1.2 Check for OpenClaw

```bash
# OpenClaw locations to check
OPENCLAW_PATHS=(
    "$HOME/.openclaw/openclaw.json"
    "$HOME/.openclaw/config.json"
    "./openclaw.json"
    "$HOME/openclaw/openclaw.json"
)

for path in "${OPENCLAW_PATHS[@]}"; do
    if [ -f "$path" ]; then
        echo "✓ Found: OpenClaw at $path"
    fi
done
```

### 1.3 Report Findings

After detection, report to user:

```
🔍 **Implementation Detection Results**

Target Directory: {target_dir}

Existing reClaw: {FOUND / NOT FOUND}
OpenClaw: {FOUND / NOT FOUND}
```

---

## Step 2: Evaluate Existing reClaw (If Found)

If an existing reClaw implementation is found, thoroughly evaluate it:

### 2.1 Read Config

```yaml
# What to look for in config.yaml
version: "1.0"                    # Config version

main:                             # Core settings
  workspace: ~/.reclaw/workspace
  log_level: info

channels:                         # Check which channels are enabled
  discord:
    enabled: true|false
  telegram:
    enabled: true|false
  # ... other channels

voice:                            # Check voice features
  tts:
    # Which TTS providers?
  stt:
    # Which STT providers?
  twilio/telnyx/plivo:
    # Phone providers?

agents:                           # Agent configuration
  default_model: ...
  max_concurrent: ...

memory:                           # Memory settings
  short_term:
  long_term:
```

### 2.2 Check Source Structure

Evaluate the `src/` or root directory for:

| Component | What to Look For | Status |
|-----------|------------------|--------|
| **Gateway** | `gateway/`, `server/`, `websocket.py`, `gateway.py` | ✅/❌ |
| **TUI** | `tui/`, `client/`, `terminal.py`, `ui.py` | ✅/❌ |
| **Channels** | `channels/`, `discord.py`, `telegram.py`, etc. | List |
| **Voice** | `voice/`, `tts/`, `stt/`, `twilio.py` | List |
| **Agents** | `agents/`, `orchestrator/`, `acp.py` | ✅/❌ |
| **Memory** | `memory/`, `storage/`, `short_term.py`, `long_term.py` | ✅/❌ |
| **Tools** | `tools/`, `tool_registry.py` | List |

### 2.3 Test if It Works

```bash
# Try to run the implementation
cd {target_dir}

# Python
python reclaw.py --version 2>/dev/null || python -m reclaw --version 2>/dev/null

# Or check for run scripts
make --version 2>/dev/null
./run.sh --version 2>/dev/null
```

### 2.4 Generate Evaluation Report

```
📋 **Existing reClaw Evaluation**

Location: {path}
Language: {Python/TypeScript/Go/Rust}

**Core Components:**
- ✅ Gateway Server: {status} (port {port})
- ✅ TUI Client: {status}
- ✅ Agent Orchestrator: {status}
- ✅ Memory System: {status}

**Channels Implemented:**
- {channel}: {enabled/configured/running}
- ...

**Voice Features:**
- TTS: {providers}
- STT: {providers}
- Phone: {providers}

**Extensions:**
- {extension}: {status}
- ...

**Configuration:**
- Config version: {version}
- Default model: {model}
- Workspace: {path}

**Data:**
- Memory files: {count}
- Sessions: {count}

**Status:** {Working / Partial / Broken / Unknown}
```

---

## Step 3: Prompt User for Action

Based on evaluation, ask user what to do:

### Scenario A: Existing reClaw Found

```
📋 **Existing reClaw Detected**

I found an existing reClaw implementation at {path}.

What's there:
{evaluation summary}

What do you want to do?

1. **🔄 Update** - Update to latest specs (keeps your work, adds new features)
   → Use reclaw-update skill

2. **➕ Extend** - Add more extensions (channels, voice, features)
   → Use reclaw-extensions skill

3. **🏗️  Rebuild** - Start fresh (I'll backup the old one first)
   → Backup to {path}.backup.{timestamp}, then fresh implementation

4. **📦 Keep & New** - Keep this one, build new in different location
   → Ask for new target directory

5. **✅ Skip** - Do nothing, exit

6. **🔍 Inspect** - Show me more details about what's there
```

### Scenario B: OpenClaw Found (No reClaw)

```
📋 **OpenClaw Detected**

I found an existing OpenClaw setup at ~/.openclaw/

What do you want to do?

1. **🔄 Migrate** - Convert OpenClaw to reClaw (keeps config, memory, agents)
   → Use reclaw-migrate skill

2. **🆕 Fresh Start** - Build reClaw from scratch
   → OpenClaw stays untouched, new reClaw in {target_dir}

3. **🔄 Migrate & Keep** - Migrate AND keep OpenClaw running
   → Different ports, can switch between them
```

### Scenario C: Nothing Found

Proceed with fresh implementation (see Step 4).

---

## Step 4: Fresh Implementation

If user chooses fresh implementation (or nothing exists):

### 4.1 Project Setup

```bash
# Create project directory
mkdir -p {target_dir}
cd {target_dir}

# Initialize based on language
# Python
python -m venv .venv
source .venv/bin/activate
# Create pyproject.toml or requirements.txt

# TypeScript
npm init -y
npm install typescript @types/node --save-dev
npx tsc --init

# Go
go mod init reclaw

# Rust
cargo init --name reclaw
```

### 4.2 Core Implementation Order

**Iteration 1: Config & Project Structure**
- Create `config.yaml.example`
- Create `config.yaml` loader
- Set up project structure
- Test: Load config without errors

**Iteration 2: Gateway Server**
- WebSocket server on port 18789
- Session management
- Basic auth
- Test: Connect with WebSocket client

**Iteration 3: CLI**
- CLI entry point (`reclaw`)
- Gateway commands: `start`, `stop`, `status`
- Config commands: `get`, `set`, `validate`
- Agent commands: `run`, `list`
- TUI command: `tui` (if TUI extension installed)
- Test: CLI commands work correctly

**Iteration 4: First Channel**
- Implement Discord OR Telegram
- Receive message → Gateway → Response
- Send response
- Test: Send message, get response via CLI or WebSocket

**Iteration 5: TUI (Extension)**
- Terminal UI client (extension, not core)
- Connect to Gateway
- Rich chat interface with streaming
- Test: Interactive chat works

**Iteration 6: LLM Integration**
- Connect to LLM API
- Message → LLM → Response flow
- Test: Chat with AI through channel

**Iteration 7: Extensions**
- Add requested channels
- Add voice (TTS/STT/Phone)
- Add features
- Add TUI (if wanted)

**Iteration 8: Sub-agents (ACP)**
- Agent orchestrator
- ACP protocol implementation
- Spawn/kill sub-agents

**Iteration 9: Memory**
- Short-term memory
- Long-term memory
- Search/index

**Iteration 10: Tests & Polish**
- Unit tests
- Integration tests
- CLI testing (critical!)
- Documentation

**Final Step: Run Tests**
```bash
cd {target_dir}

# Run all tests
make test
# OR
python -m pytest tests/ -v
# OR
npm test

# Verify critical tests pass
```

### Critical CLI Tests (MUST PASS)
```bash
# Test gateway commands
reclaw gateway status
reclaw gateway start
reclaw gateway stop

# Test config commands
reclaw config get
reclaw config set key value
reclaw config validate

# Test agent commands
reclaw agent list
reclaw agent run "test message"

# Test general
reclaw --version
reclaw --help
```

After tests pass, report to user:
```
✅ **Implementation Complete**

All tests passed. Your reClaw implementation is ready at {target_dir}.

To start:
1. Gateway: reclaw gateway start
2. CLI: reclaw --help
3. TUI (extension): See extensions/tui.md

Next: Run the migration skill to transfer your OpenClaw data (if applicable)
```

---

## Step 5: Rebuild (Backup First!)

If user chooses to rebuild existing:

```bash
# Create timestamped backup
BACKUP_DIR="{target_dir}.backup.$(date +%Y%m%d_%H%M%S)"
cp -r {target_dir} "$BACKUP_DIR"
echo "✓ Backed up to: $BACKUP_DIR"

# Clear for fresh build
rm -rf {target_dir}/*
# Keep .env if exists
[ -f "$BACKUP_DIR/.env" ] && cp "$BACKUP_DIR/.env" {target_dir}/
```

Then proceed with fresh implementation.

---

## Quick Reference (For AI While Coding)

### Core Interfaces

#### Channel Adapter
```
connect() → disconnect()
send_message(target, content, options?) → message_id
on_message(callback) → on_voice(callback)
```

#### Agent Orchestrator  
```
spawn_subagent(task, config) → SubAgent
get_subagent_status(id) → Status
kill_subagent(id) → void
```

#### Memory Manager
```
load_short_term(session_id) → messages
save_short_term(session_id, messages) → void
load_long_term(session_id, query?) → entries
save_long_term(entry) → void
search(query, options?) → results
```

#### Voice Adapter
```
speak(text, options?) → audio
transcribe(audio, options?) → text
handle_incoming_call(webhook) → CallSession
```

### Key Formats

#### Normalized Message
```
{
  id: "channel_type:original_id",
  channel_type: "discord",
  channel_id: "channel_type:channel_id", 
  author_id: "channel_type:user_id",
  author_name: "Name",
  content: "text",
  timestamp: "ISO-8601",
  is_group: boolean,
  was_mentioned: boolean
}
```

#### Session Key
```
agent:chat:subagent:task
```

#### ACP (NDJSON)
```
{"type": "initialize", "session": "...", "capabilities": [...]}
{"type": "turn/start", "prompt": "...", "turn_id": "..."}
{"type": "message", "content": "..."}
{"type": "tool_call", "tool": "...", "params": {...}}
{"type": "turn_complete", "turn_id": "...", "result": "..."}
```

### Files to Reference

| Need | File |
|------|------|
| Architecture | impl/ARCHITECTURE.md |
| Config schema | impl/CONFIG.md |
| Protocols | impl/PROTOCOLS.md |
| Channels | impl/CHANNELS.md |
| Voice | impl/VOICE.md |
| Agents | impl/AGENTS.md |
| Memory | impl/MEMORY.md |

### Language Tips

**Python:** asyncio, dataclasses, pydantic
**TypeScript:** async/await, interfaces  
**Go:** goroutines, interfaces, channels
**Rust:** async/await (Tokio), traits

---

## Related Skills

| Skill | When to Use |
|-------|-------------|
| `reclaw-update` | Update existing implementation to latest specs |
| `reclaw-migrate` | Migrate from OpenClaw to reClaw |
| `reclaw-extensions` | Add/remove extensions from existing implementation |

---

## Checklist

Before starting implementation:
- [ ] Checked for existing reClaw in target directory
- [ ] Checked for OpenClaw
- [ ] Evaluated existing implementation (if found)
- [ ] Prompted user for action
- [ ] Created backup (if rebuilding)
- [ ] Confirmed target directory
- [ ] Confirmed programming language
- [ ] Confirmed extensions to include
