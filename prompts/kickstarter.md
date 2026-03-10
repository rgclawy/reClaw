You are helping me build reClaw. I want a complete, working implementation.

## Step 0: Check for Existing Implementations (IMPORTANT!)

**BEFORE DOING ANYTHING ELSE**, check for existing installations:

### 1. Check for Existing reClaw Implementation
Look for these markers in the target directory (default: `./reclaw` or user-specified):
- `config.yaml` or `reclaw.yaml` (reClaw config file)
- `reclaw.py`, `main.py`, or `src/` directory with reClaw code
- `pyproject.toml`, `package.json`, or `Cargo.toml` with reClaw dependencies
- Gateway server implementation (WebSocket on port 18789)
- Memory directory with existing data

### 2. Check for OpenClaw
- Look for `~/.openclaw/openclaw.json`
- Look for `openclaw.json` in common locations

### 3. Decision Tree Based on Findings

| What Exists | Action |
|-------------|--------|
| **Existing reClaw** | Evaluate what's there → Ask user what to do |
| **OpenClaw only** | Offer migration via `reclaw-migrate` skill |
| **Neither** | Fresh implementation |
| **Both** | Ask user which to prioritize |

---

## My Answers

### Basic Info
- **Language:** _____________ (Python / TypeScript / Go / Rust / Other)
- **Start from scratch:** _____________ (yes / no - if no, what exists?)
- **Where to implement?** _____________ (e.g., ./reclaw, ~/projects/my-reclaw - default: ./reclaw)

> **⚠️ Have OpenClaw?** If you have an existing OpenClaw setup (check `~/.openclaw/`), mention it here and I'll offer to migrate it!

> **⚠️ Have reClaw already?** If you've built reClaw before, tell me where and I'll check what's there!

### Which Skill?
Choose one (or multiple):

| Skill | Purpose |
|-------|---------|
| `reclaw-implementation` | Build reClaw from scratch (default) |
| `reclaw-migrate` | Migrate from OpenClaw to reClaw (preserves config, memory, agents) |
| `reclaw-update` | Update existing implementation with latest specs |
| `reclaw-extensions` | Add/remove extensions anytime |

- **Skill:** _____________ (implementation / migrate / update / extensions)

> **💡 Tip:** If you have OpenClaw and want to keep your settings, choose `migrate`!
> **💡 Tip:** If you have reClaw and want to update it, choose `update`!

---

## Core (Always Included)

These are always in reClaw:
- **Gateway Server** - WebSocket server (port 18789) for client connections, session management, auth
- **CLI** - Single command-line interface with all commands (gateway, agent, config, etc.)
- Sub-agent system (ACP protocol)
- Memory (short-term + long-term files)
- Config (YAML + $VAR)

**Note:** TUI is available via `reclaw tui` if TUI extension is installed.

---

## Extensions (Your Choice)

Choose what you need:

| Category | Options | Status |
|----------|---------|--------|
| **Channels** | Discord, Telegram, WhatsApp, Slack, Signal, iMessage | Extensions |
| **Voice TTS** | OpenAI, Edge, ElevenLabs | Extensions |
| **Voice STT** | Whisper, Deepgram | Extensions |
| **Phone** | Twilio, Telnyx, Plivo | Extensions |
| **TUI** | Rich interactive terminal UI | Extension (`reclaw tui`)
| **Web** | web_search (Brave, Perplexity...), browser (Playwright) | Extensions |
| **Features** | Multi-account, DM policies, Hybrid search | Extensions |

### Feature Selection

Which extensions do you want?

- **Channels:** _____________ (all / none / list: Discord, Telegram, etc.)
- **Voice TTS:** _____________ (all / none / list: OpenAI, Edge, ElevenLabs)
- **Voice STT:** _____________ (all / none / list: Whisper, Deepgram)
- **Phone:** _____________ (all / none / list: Twilio, Telnyx, Plivo)
- **Features:** _____________ (all / none / list: multi-account, dm-policies, hybrid-search)

---

## Your Job

### Phase 1: Discovery (MUST DO FIRST)

1. **CHECK FOR EXISTING RECLAW** in the target directory:
   ```bash
   # Check for reClaw markers
   ls -la {target_dir}/config.yaml 2>/dev/null && echo "Found: config.yaml"
   ls -la {target_dir}/reclaw.py 2>/dev/null && echo "Found: reclaw.py"
   ls -la {target_dir}/src/ 2>/dev/null && echo "Found: src/"
   ls -la {target_dir}/memory/ 2>/dev/null && echo "Found: memory/"
   ```

2. **EVALUATE existing reClaw** (if found):
   - Read `config.yaml` to see what's configured
   - Check `src/` structure to see what's implemented
   - Look for gateway, TUI, channels, voice implementations
   - Check memory directory for existing data
   - Identify installed extensions

3. **CHECK FOR OPENCLAW**:
   - Look for `~/.openclaw/openclaw.json`
   - Look for `openclaw.json` in common locations

### Phase 2: Prompt User Based on Findings

#### If Existing reClaw Found:
**Present findings and ask:**
```
📋 **Existing reClaw Found**

Location: {path}
Status: {evaluation summary}

What's implemented:
- ✅ Gateway: {yes/no}
- ✅ TUI: {yes/no}
- ✅ Channels: {list or none}
- ✅ Voice: {list or none}
- ✅ Memory: {yes/no}
- ✅ Extensions: {list}

What do you want to do?
1. **Update** - Update to latest specs (use reclaw-update skill)
2. **Extend** - Add new extensions (use reclaw-extensions skill)
3. **Rebuild** - Start fresh (backup old first)
4. **Skip** - Keep as-is, do nothing
5. **Migrate** - Migrate from OpenClaw instead
```

#### If OpenClaw Found (no reClaw):
```
📋 **OpenClaw Found**

I found an existing OpenClaw setup at ~/.openclaw/

What do you want to do?
1. **Migrate** - Convert OpenClaw to reClaw (preserves config/memory)
2. **Fresh Start** - Build reClaw from scratch (OpenClaw stays untouched)
3. **Both** - Migrate AND keep OpenClaw (different ports)
```

#### If Nothing Found:
Proceed with fresh implementation.

### Phase 3: Execute Based on User Choice

| User Choice | Skill to Use | Action |
|-------------|--------------|--------|
| Update existing | `reclaw-update` | Pull latest specs, compare, suggest changes |
| Add extensions | `reclaw-extensions` | Add requested extensions to existing |
| Rebuild | `reclaw-implementation` | Backup old, fresh implementation |
| Migrate from OpenClaw | `reclaw-migrate` | Convert config/memory from OpenClaw |
| Fresh start | `reclaw-implementation` | Build from scratch |

### Phase 4: Implementation (for fresh builds)

Based on selected skill and extensions:
- **Implementation:** Create project, implement core + selected extensions
- **Migrate:** Convert OpenClaw config/memory to reClaw format
- **Update:** Pull latest specs, compare with current, suggest changes
- **Extensions:** Add/remove requested extensions

Then:
1. Create the project structure
2. Implement everything
3. Write tests
4. Give me working code

---

## Process

### For Implementation Skill (Fresh Build)
**Iteration 1:** Project setup + Config loader → Test it works

**Iteration 2:** Gateway Server → WebSocket server, auth, session management

**Iteration 3:** CLI commands → Test gateway start, agent run, config, tui (if installed)

**Iteration 5:** One channel (Discord or Telegram) → Receive message, send response

**Iteration 6:** Connect to LLM → Message → LLM → Response

**Iteration 7:** Add selected extensions (channels/voice/features/TUI)

**Iteration 8:** Add sub-agents (ACP protocol)

**Iteration 9:** Add memory

**Iteration 10:** Tests + polish (focus on CLI testing)

### For Update Skill
1. Check current implementation
2. Pull latest reclaw specs from GitHub
3. Compare with current implementation
4. Report what's changed
5. Ask what to update

### For Extensions Skill
1. List available extensions
2. User selects which to add/remove
3. Integrate into existing code
4. Test works

### For Migrate Skill
1. Audit OpenClaw setup
2. Create reClaw project
3. Convert config (JSON → YAML)
4. Copy memory files
5. Test migration

---

## References

The `impl/` folder has core guides:
- ARCHITECTURE.md - System design
- GATEWAY.md - WebSocket server, session management, auth
- CLI.md - Command-line interface specification
- CONFIG.md - Config schema  
- PROTOCOLS.md - ACP formats
- AGENTS.md - ACP
- MEMORY.md - Storage

The `extensions/` folder has extension guides:
- TUI (rich Terminal UI with streaming, tool cards)
- Channels (Discord, Telegram, WhatsApp, Slack, Signal, iMessage)
- Voice (OpenAI TTS, Edge TTS, Whisper STT, Deepgram, Twilio)
- Web (web_search, browser)
- Features (multi-account, dm-policies, hybrid-search)

---

## Post-Implementation: Migration Support

After your reClaw implementation is complete, if you have an existing **OpenClaw** setup you'd like to migrate:

**Use the `reclaw-migrate` skill:**
```bash
# Kimi CLI
kimi -s "$(cat skills/reclaw-migrate/SKILL.md)"

# Or simply tell your AI:
"Migrate my OpenClaw setup to this new reClaw implementation"
```

The migration skill will help you:
- Convert OpenClaw JSON config → reClaw YAML config
- Copy and reformat memory files
- Transfer workflows and prompts
- Verify everything works in reClaw

---

## Start Now

Answer the questions above, then let's build!

> **Remember:** I'll first check if you already have reClaw or OpenClaw installed before doing anything!
