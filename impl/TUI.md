# TUI (Terminal UI)

**Status:** Extension - Provides rich interactive interface

**Purpose:** Terminal-based chat interface for interacting with agents via the Gateway

---

## 1. What is the TUI?

A **terminal-based chat interface** that:
1. Connects to the Gateway via WebSocket
2. Displays chat history with streaming responses
3. Shows tool calls as expandable cards
4. Provides model/agent/session pickers
5. Supports local shell command execution

```
┌─────────────────────────────────────────────────────────────────┐
│  reClaw TUI │ ws://127.0.0.1:18789 │ agent:main │ session:main  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  [user] Hello, can you help me with Python?                    │
│                                                                 │
│  [assistant] Of course! What would you like to know?           │
│                                                                 │
│  [user] How do I read a file?                                  │
│                                                                 │
│  [assistant] You can use the built-in `open()` function:       │
│                                                                 │
│  ```python                                                      │
│  with open('file.txt', 'r') as f:                              │
│      content = f.read()                                        │
│  ```                                                           │
│                                                                 │
│  ┌─ Tool Call: read_file ───────────────────────────────────┐  │
│  │ file_path: "/Users/myclaw/.openclaw/config.yaml"          │  │
│  │                                                           │  │
│  │ Result: 124 lines read                                   │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│  connected | idle │ 0 tokens | deliver: off                   │
├─────────────────────────────────────────────────────────────────┤
│  > How about async file reading?                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Core Interface

```python
class TUI:
    # Lifecycle
    async start() -> void
    async stop() -> void
    
    # Connection
    async connect(url: str, auth: dict) -> void
    async disconnect() -> void
    async reconnect() -> void
    
    # UI Components
    render_header() -> void
    render_chat_log() -> void
    render_status_bar() -> void
    render_input() -> void
    
    # Actions
    async send_message(text: str) -> void
    async abort_run() -> void
    async execute_local_command(command: str) -> void
    
    # Pickers
    open_model_picker() -> void
    open_agent_picker() -> void
    open_session_picker() -> void
    open_settings_panel() -> void
```

---

## 3. Screen Layout

```
┌─────────────────────────────────────────────────────────────────┐
│ HEADER: URL | Agent | Session | Model                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ CHAT LOG:                                                       │
│   - User messages                                               │
│   - Assistant responses (streaming)                            │
│   - System notices                                              │
│   - Tool cards (collapsible)                                    │
│   - Error messages                                              │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│ STATUS BAR: Connection | State | Tokens | Deliver | Think       │
├─────────────────────────────────────────────────────────────────┤
│ INPUT: Text editor with auto-complete                           │
└─────────────────────────────────────────────────────────────────┘
```

### Header Fields
| Field | Example |
|-------|---------|
| URL | `ws://127.0.0.1:18789` |
| Agent | `agent:main` |
| Session | `session:main` |
| Model | `kimi-coding/k2p5` |

### Status Bar States
| State | Description |
|-------|-------------|
| `connecting` | Establishing WebSocket connection |
| `connected` | Connected, waiting for input |
| `idle` | Connected, agent idle |
| `running` | Agent processing |
| `streaming` | Receiving response stream |
| `error` | Connection or agent error |

---

## 4. Connection

### Local Gateway (Default)
```bash
reclaw tui
```

Connects to `ws://127.0.0.1:18789` with default settings.

### Remote Gateway
```bash
reclaw tui --url ws://host:18789 --token <token>
reclaw tui --url ws://host:18789 --password <password>
```

### Connection Options
```bash
reclaw tui \
  --url ws://127.0.0.1:18789 \
  --token $GATEWAY_TOKEN \
  --agent main \
  --session main \
  --deliver \
  --thinking low \
  --timeout-ms 60000
```

| Option | Description | Default |
|--------|-------------|---------|
| `--url` | Gateway WebSocket URL | From config or `ws://127.0.0.1:18789` |
| `--token` | Auth token | From config |
| `--password` | Auth password | From config |
| `--agent` | Agent ID | `main` |
| `--session` | Session key | `main` (or `global` if scope is global) |
| `--deliver` | Deliver to provider | `false` |
| `--thinking` | Thinking level | From agent config |
| `--timeout-ms` | Agent timeout | From agent config |
| `--history-limit` | Messages to load | `200` |

---

## 5. Keyboard Shortcuts

### Core Shortcuts
| Key | Action |
|-----|--------|
| `Enter` | Send message |
| `Esc` | Abort current run |
| `Ctrl+C` | Clear input (press twice to exit) |
| `Ctrl+D` | Exit TUI |
| `Ctrl+L` | Model picker |
| `Ctrl+G` | Agent picker |
| `Ctrl+P` | Session picker |
| `Ctrl+O` | Toggle tool output expansion |
| `Ctrl+T` | Toggle thinking visibility |
| `Tab` | Auto-complete slash command |
| `↑/↓` | Scroll history / navigate pickers |

### Picker Navigation
| Key | Action |
|-----|--------|
| `↑/↓` or `j/k` | Navigate items |
| `Enter` | Select item |
| `Esc` | Close picker |
| `/` | Filter list |

---

## 6. Slash Commands

### Core Commands
```
/help              Show available commands
/status            Show gateway and agent status
/agent <id>        Switch to agent (or /agents for list)
/session <key>     Switch session (or /sessions for list)
/model <model>     Set model (or /models for list)
```

### Session Control
```
/think <off|minimal|low|medium|high>    Set thinking level
/verbose <on|full|off>                  Set verbosity
/reasoning <on|off|stream>              Toggle reasoning
/usage <off|tokens|full>                Token usage display
/elevated <on|off|ask|full>             Elevated permissions
/activation <mention|always>            Activation mode
/deliver <on|off>                       Toggle delivery
```

### Session Lifecycle
```
/new               Reset current session
/reset             Same as /new
/abort             Abort current run
/settings          Open settings panel
/exit              Exit TUI
```

### Gateway Commands
Commands prefixed with `/` are forwarded to Gateway:
```
/context           Show current context
/capabilities      List agent capabilities
```

---

## 7. Local Shell Commands

Execute local commands with `!` prefix:

```
!ls -la
!git status
!cat ~/.bashrc
```

**Security:**
- TUI prompts once per session: "Allow local command execution?"
- If denied, `!` commands are sent as regular messages
- Commands run in TUI working directory
- No persistent state (each command is fresh shell)

**Special Cases:**
- `!` alone → sent as regular message
- `  !ls` (leading space) → not a command, sent as message

---

## 8. Tool Cards

Tool calls render as expandable cards:

```
┌─ Tool Call: web_search ───────────────────────────────────────┐
│ query: "python asyncio tutorial"                               │
│                                                               │
│ Result: 5 results found                                       │
│ 1. docs.python.org - asyncio — Asynchronous I/O ...          │
│ 2. realpython.com - Async IO in Python: A Complete Walkthrough│
│ ...                                                           │
└───────────────────────────────────────────────────────────────┘
```

**Controls:**
- `Ctrl+O` - Toggle all cards collapsed/expanded
- Click/Enter on card header - Toggle individual card
- Cards auto-expand while tool is running
- Stream partial updates into same card

---

## 9. Mental Model: Agents + Sessions

### Agent
- Unique slug (e.g., `main`, `research`, `code`)
- Gateway exposes list of available agents
- Each agent has its own configuration

### Session
- Belongs to a specific agent
- Session key format: `agent:{agentId}:{sessionKey}`
- Examples:
  - `/session main` → `agent:main:main`
  - `/session agent:research:project1` → Switch to research agent

### Session Scope
- `per-sender` (default): Multiple sessions per agent
- `global`: Single `global` session

---

## 10. Streaming & History

### History Loading
- On connect: Load last 200 messages (configurable)
- Scroll up to load older messages
- Full conversation context maintained

### Streaming Responses
- Assistant responses stream in real-time
- Token counter updates live
- Thinking blocks (if enabled) shown separately
- Tool calls appear when executed

### Message States
```
[sending] → [sent] → [streaming] → [complete]
                    ↓
               [error]
```

---

## 11. Pickers

### Model Picker (Ctrl+L)
```
┌─ Select Model ──────────────────────────┐
│ /kimi-coding/k2p5                  ●   │  ← current
│ /kimi-coding/k2p5-thinking             │
│ /anthropic/claude-sonnet-4             │
│ /openai/gpt-4o                         │
│                                         │
│ Override for this session only? [Y/n]  │
└─────────────────────────────────────────┘
```

### Agent Picker (Ctrl+G)
```
┌─ Select Agent ──────────────────────────┐
│ main                                ●  │  ← current
│ research                               │
│ code-assistant                         │
│ shell                                  │
└─────────────────────────────────────────┘
```

### Session Picker (Ctrl+P)
```
┌─ Select Session (agent:main) ───────────┐
│ main                                ●  │  ← current
│ dev                                    │
│ project-alpha                          │
│ [+ Create New Session]                 │
└─────────────────────────────────────────┘
```

---

## 12. Settings Panel

Access with `/settings`:

```
┌─ Settings ────────────────────────────────────────────────────┐
│                                                               │
│  [✓] Deliver responses to provider                          │
│  [✓] Expand tool output by default                          │
│  [ ] Show thinking blocks                                   │
│                                                               │
│  Thinking level: [low ▼]                                     │
│  Verbosity:      [full ▼]                                    │
│                                                               │
│  [Save] [Cancel]                                             │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## 13. Implementation Patterns

### Python (Textual)
```python
from textual.app import App
from textual.widgets import Input, Static, Header, Footer
import websockets

class ReclawTUI(App):
    CSS = """
    Screen { align: center middle; }
    .chat-log { width: 100%; height: 80%; }
    .input { dock: bottom; height: 3; }
    """
    
    def __init__(self, config):
        super().__init__()
        self.config = config
        self.ws = None
        self.messages = []
        
    def compose(self):
        yield Header(show_clock=True)
        yield Static(id="chat-log", classes="chat-log")
        yield Input(placeholder="Type a message...", classes="input")
        yield Footer()
    
    async def on_mount(self):
        # Connect to Gateway
        self.ws = await websockets.connect(self.config.url)
        await self.authenticate()
        asyncio.create_task(self.receive_loop())
    
    async def on_input_submitted(self, event):
        text = event.value
        
        if text.startswith("!"):
            await self.execute_local(text[1:])
        elif text.startswith("/"):
            await self.handle_slash_command(text)
        else:
            await self.send_message(text)
    
    async def receive_loop(self):
        async for message in self.ws:
            data = json.loads(message)
            await self.handle_message(data)
```

### Node.js (ink + ws)
```javascript
const { render, Box, Text, useInput } = require('ink');
const WebSocket = require('ws');

function TUI({ config }) {
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState('');
  const [status, setStatus] = useState('connecting');
  
  const ws = useMemo(() => new WebSocket(config.url), []);
  
  useEffect(() => {
    ws.on('open', () => setStatus('connected'));
    ws.on('message', (data) => {
      const msg = JSON.parse(data);
      setMessages(m => [...m, msg]);
    });
  }, []);
  
  useInput((input, key) => {
    if (key.return) {
      ws.send(JSON.stringify({ type: 'message', content: input }));
      setInput('');
    }
    // ... other key handlers
  });
  
  return (
    <Box flexDirection="column">
      <Box>{/* Header */}</Box>
      <Box>{/* Chat log */}</Box>
      <Box>{/* Status bar */}</Box>
      <Box>{/* Input */}</Box>
    </Box>
  );
}
```

---

## 14. Configuration

```yaml
tui:
  enabled: true
  
  # Default connection
  gateway_url: "ws://127.0.0.1:18789"
  
  # Display
  theme: "dark"              # dark | light
  show_tool_output: true     # Expand tool cards by default
  show_thinking: false       # Show thinking blocks
  history_limit: 200         # Messages to load on connect
  
  # Behavior
  auto_deliver: false        # Enable delivery by default
  confirm_local_exec: true   # Ask before running !commands
  
  # Keybindings (optional overrides)
  keys:
    send: "enter"
    abort: "escape"
    exit: "ctrl+c"
```

---

## 15. Error Handling

| Error | Display |
|-------|---------|
| Connection refused | `⚠ Cannot connect to Gateway at ws://...` |
| Auth failed | `⚠ Authentication failed - check token/password` |
| Agent not found | `⚠ Agent 'foo' not found` |
| Timeout | `⚠ Request timed out` |
| Rate limited | `⚠ Rate limited - retry in 5s` |

---

## 16. Reference

**Default Settings:**
- Gateway URL: `ws://127.0.0.1:18789`
- Default agent: `main`
- Default session: `main`
- History limit: `200`
- Ping interval: `30s`

**Environment Variables:**
```bash
export RECLAW_GATEWAY_URL="ws://host:18789"
export RECLAW_GATEWAY_TOKEN="xxx"
export RECLAW_AGENT="main"
export RECLAW_SESSION="main"
```

See also:
- `GATEWAY.md` - Gateway server specification
- `PROTOCOLS.md` - WebSocket message formats
- `ARCHITECTURE.md` - System architecture
