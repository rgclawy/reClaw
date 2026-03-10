# CLI (Command Line Interface)

**Status:** Core - Single CLI tool for all reClaw operations

**Purpose:** Provide command-line interface for reClaw operations

---

## Philosophy

Single CLI tool (`reclaw`) with all functionality:
- **Core commands:** gateway, config, agent, status
- **TUI:** Available as subcommand if TUI dependencies installed
- **Extensions:** Commands for channels, voice, web tools, etc.

If dependencies for a command are missing, show helpful error message.

---

## Core Commands

### Gateway Management

```bash
reclaw gateway start     # Start Gateway server
reclaw gateway stop      # Stop Gateway server
reclaw gateway status    # Check Gateway status
reclaw gateway restart   # Restart Gateway
```

### Configuration

```bash
reclaw config get [key]              # Get config value
reclaw config set <key> <value>      # Set config value
reclaw config validate               # Validate config file
reclaw config reload                 # Reload config without restart
```

### Agent Operations

```bash
reclaw agent list                    # List configured agents
reclaw agent run "<message>"         # Run single agent turn
reclaw agent spawn <task>            # Spawn sub-agent for task
```

### Status & Info

```bash
reclaw status                        # Show system status
reclaw health                        # Check Gateway health
reclaw --version                     # Show version
reclaw --help                        # Show help
```

---

## Configuration

```yaml
cli:
  # Default command when no subcommand given
  default_command: help  # help | status | tui (if installed)
  
  # Output format
  output_format: text    # text | json
  
  # Gateway connection (for CLI commands that need Gateway)
  gateway_url: "ws://127.0.0.1:18789"
  
  # Authentication
  auth_token: $RECLAW_TOKEN
```

---

## Implementation Notes

### Minimal Dependencies

Core CLI should have minimal dependencies:
- HTTP client (for REST API calls)
- WebSocket client (for Gateway communication)
- YAML parser (for config)
- No TUI framework dependencies in core

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Invalid arguments |
| 3 | Gateway not running |
| 4 | Config error |
| 5 | Authentication failed |

### Output Formats

**Text (default):**
```
$ reclaw gateway status
Gateway: running
Port: 18789
Uptime: 2h 34m
Agents: 3
```

**JSON (`--format json`):**
```json
{
  "gateway": "running",
  "port": 18789,
  "uptime": "2h 34m",
  "agents": 3
}
```

---

## TUI Command

TUI is available as a subcommand:

```bash
reclaw tui    # Launch Terminal UI
```

If TUI dependencies are not installed:
```
Error: TUI not available. Install with: pip install reclaw[tui]
```

## Extension Commands

Extensions add commands dynamically:

```bash
# Channels extension
reclaw channels list
reclaw channels add discord
reclaw channels remove telegram

# Web extension
reclaw web search "query"
reclaw web fetch <url>
reclaw browser start

# Voice extension
reclaw voice call <number>
reclaw voice tts "Hello world"
```

If extension is not installed:
```
Error: 'channels' command not found. Install channels extension.
```

---

## Testing CLI

Core CLI must be tested after implementation:

```bash
# Basic commands
reclaw --version
reclaw --help

# Config
reclaw config validate
reclaw config get gateway.port
reclaw config set agents.default.model "kimi-coding/k2p5"

# Gateway lifecycle
reclaw gateway start
reclaw gateway status
reclaw gateway stop

# Agents
reclaw agent list
reclaw agent run "test message"
```

All core CLI commands should work without TUI installed.
