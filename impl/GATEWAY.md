# Gateway Server

**Purpose:** WebSocket server that acts as the central hub for all client connections and agent orchestration

---

## 1. What is the Gateway?

The Gateway is a **WebSocket server** that:
1. Accepts connections from clients (TUI, web UI, external tools)
2. Routes messages between clients and agents
3. Manages session state
4. Exposes agent management APIs
5. Handles authentication

```
┌─────────┐     ┌──────────┐     ┌──────────┐
│   TUI   │◄───▶│          │◄───▶│ Channels │
│  (CLI)  │     │          │     │(Discord, │
└─────────┘     │  Gateway │     │Telegram, │
                │  Server  │     │WhatsApp) │
┌─────────┐     │ (port    │     └──────────┘
│  Web UI │◀───▶│  18789)  │◀───┐
└─────────┘     │          │    │
                └────┬─────┘    │
                     │          │
                ┌────┴─────┐    │
                │  Agents  │◀───┘
                │  (ACP)   │
                └──────────┘
```

---

## 2. Core Interface

```python
class GatewayServer:
    # Server lifecycle
    async start(host: str, port: int) -> void
    async stop() -> void
    
    # Client connections
    async on_connect(websocket: WebSocket, client_info: dict) -> ClientConnection
    async on_disconnect(client_id: str) -> void
    
    # Message routing
    async route_to_agent(session_id: str, message: dict) -> void
    async route_to_client(client_id: str, message: dict) -> void
    
    # Session management
    async create_session(agent_id: str, session_key: str) -> Session
    async get_session(session_id: str) -> Session | null
    async list_sessions(agent_id?: str) -> list[Session]
    
    # Agent management
    async list_agents() -> list[AgentInfo]
    async get_agent(agent_id: str) -> AgentInfo | null
```

---

## 3. WebSocket Protocol

### 3.1 Connection

**Client → Gateway:**
```json
{
  "type": "connect",
  "mode": "tui",           // "tui", "web", "api"
  "agent": "main",          // Default agent
  "session": "main",        // Session key
  "auth": {
    "type": "token",        // "token" or "password"
    "value": "xxx"
  }
}
```

**Gateway → Client:**
```json
{
  "type": "connected",
  "client_id": "uuid",
  "session_id": "agent:main:main",
  "agent": {
    "id": "main",
    "model": "kimi-coding/k2p5",
    "capabilities": [...]
  },
  "server_time": "2026-03-09T20:00:00Z"
}
```

### 3.2 Sending Messages

**Client → Gateway:**
```json
{
  "type": "message",
  "content": "Hello, agent!",
  "turn_id": "uuid",
  "options": {
    "thinking": "low",
    "deliver": true
  }
}
```

**Gateway → Client (streaming):**
```json
{"type": "thinking", "content": "..."}
{"type": "message_chunk", "content": "Hello"}
{"type": "message_chunk", "content": " there"}
{"type": "tool_call", "tool": "web_search", "params": {...}}
{"type": "tool_result", "call_id": "...", "result": "..."}
{"type": "complete", "turn_id": "..."}
```

### 3.3 Slash Commands

Clients can send slash commands:

| Command | Description |
|---------|-------------|
| `/help` | Show available commands |
| `/status` | Show gateway status |
| `/agent <id>` | Switch agent |
| `/session <key>` | Switch session |
| `/model <provider/model>` | Set model |
| `/think <level>` | Set thinking level |
| `/deliver <on/off>` | Toggle delivery |
| `/new` or `/reset` | Reset session |
| `/abort` | Abort current run |
| `/exit` | Disconnect |

### 3.4 Agent Events

**Gateway broadcasts to clients:**
```json
{"type": "agent_state", "agent_id": "...", "state": "idle|busy|error"}
{"type": "session_update", "session_id": "...", "message_count": 42}
{"type": "error", "code": "...", "message": "..."}
```

---

## 4. Session Keys

Format: `agent:{agentId}:{sessionKey}`

| Session Key | Meaning |
|-------------|---------|
| `agent:main:main` | Main session for "main" agent |
| `agent:main:dev` | Development session |
| `agent:research:project1` | Research agent, project 1 |
| `agent:main:global` | Global scope session |

**Session Scope:**
- `per-sender` (default): Each sender gets own session
- `global`: One shared session for all

---

## 5. Authentication Modes

### 5.1 Token Auth (Default)
```yaml
gateway:
  auth:
    mode: token
    token: $GATEWAY_TOKEN
```

Client sends: `Authorization: Bearer <token>`

### 5.2 Password Auth
```yaml
gateway:
  auth:
    mode: password
    password: $GATEWAY_PASSWORD
```

Client sends password in connection message.

### 5.3 No Auth (Development Only)
```yaml
gateway:
  auth:
    mode: none
```

---

## 6. Configuration

```yaml
gateway:
  enabled: true
  port: 18789
  bind: loopback           # loopback | lan | all | custom
  bind_address: "127.0.0.1" # Used when bind: custom
  
  auth:
    mode: token            # token | password | none
    token: $GATEWAY_TOKEN
    # OR
    # password: $GATEWAY_PASSWORD
  
  # CORS (for web UI)
  cors:
    enabled: true
    origins: ["http://localhost:3000"]
  
  # WebSocket settings
  websocket:
    ping_interval: 30      # seconds
    ping_timeout: 10       # seconds
    max_message_size: 10485760  # 10MB
  
  # HTTP API settings
  http:
    enabled: true
    endpoints:
      - /api/v1/status
      - /api/v1/agents
      - /api/v1/sessions
      - /api/v1/models
```

---

## 7. HTTP REST API

### GET /api/v1/status
```json
{
  "status": "healthy",
  "version": "0.1.0",
  "uptime": 3600,
  "connections": {
    "total": 5,
    "by_mode": {"tui": 2, "web": 3}
  },
  "agents": {
    "total": 3,
    "active": 2
  }
}
```

### GET /api/v1/agents
```json
{
  "agents": [
    {
      "id": "main",
      "name": "Main Agent",
      "model": "kimi-coding/k2p5",
      "status": "idle",
      "session_count": 2
    }
  ]
}
```

### GET /api/v1/sessions
```json
{
  "sessions": [
    {
      "id": "agent:main:main",
      "agent_id": "main",
      "message_count": 42,
      "created_at": "2026-03-09T20:00:00Z",
      "last_activity": "2026-03-09T20:30:00Z"
    }
  ]
}
```

### POST /api/v1/message
Send a message via HTTP (non-streaming):
```json
{
  "agent": "main",
  "session": "main",
  "content": "Hello!",
  "options": {"thinking": "low"}
}
```

Response:
```json
{
  "turn_id": "uuid",
  "response": "Hello! How can I help?",
  "tokens_used": 150
}
```

---

## 8. Implementation Patterns

### Python (websockets + FastAPI)
```python
import asyncio
import websockets
from fastapi import FastAPI, WebSocket

class GatewayServer:
    def __init__(self, config):
        self.config = config
        self.clients = {}  # client_id -> connection
        self.sessions = {}  # session_id -> session
        
    async def start(self):
        # Start WebSocket server
        async with websockets.serve(
            self.handle_connection,
            self.config.host,
            self.config.port
        ):
            await asyncio.Future()  # Run forever
    
    async def handle_connection(self, websocket, path):
        # 1. Authenticate
        auth_msg = await websocket.recv()
        client = await self.authenticate(auth_msg)
        
        # 2. Create/get session
        session = await self.get_or_create_session(
            client.agent_id, 
            client.session_key
        )
        
        # 3. Handle messages
        try:
            async for message in websocket:
                await self.route_message(session, message)
        except websockets.exceptions.ConnectionClosed:
            await self.on_disconnect(client.id)
```

### Node.js (ws + express)
```javascript
const WebSocket = require('ws');
const express = require('express');

class GatewayServer {
  constructor(config) {
    this.config = config;
    this.clients = new Map();
    this.sessions = new Map();
  }

  start() {
    const wss = new WebSocket.Server({ port: this.config.port });
    
    wss.on('connection', (ws, req) => {
      this.handleConnection(ws, req);
    });
  }

  async handleConnection(ws, req) {
    // Handle auth, session, message routing
    ws.on('message', (data) => {
      this.routeMessage(ws, JSON.parse(data));
    });
    
    ws.on('close', () => {
      this.onDisconnect(ws.clientId);
    });
  }
}
```

---

## 9. Client Connection Modes

| Mode | Purpose | Capabilities |
|------|---------|--------------|
| `tui` | Terminal UI client | Full chat, commands, local exec |
| `web` | Web UI client | Chat, settings, visual tools |
| `api` | Programmatic access | Send/receive messages |
| `channel` | Channel adapter connection | Bridge to messaging platforms |

---

## 10. Error Handling

**Gateway → Client Error Messages:**
```json
{"type": "error", "code": "AUTH_FAILED", "message": "Invalid token"}
{"type": "error", "code": "AGENT_NOT_FOUND", "message": "Agent 'foo' not found"}
{"type": "error", "code": "SESSION_INVALID", "message": "Session expired"}
{"type": "error", "code": "RATE_LIMITED", "message": "Too many requests"}
{"type": "error", "code": "MESSAGE_TOO_LARGE", "message": "Message exceeds 10MB"}
```

---

## 11. Security Considerations

1. **Always use TLS in production** (wss:// not ws://)
2. **Validate all input** - Check message sizes, JSON validity
3. **Rate limiting** - Per-client and global limits
4. **Authentication** - Token or password required
5. **CORS restrictions** - Whitelist allowed origins
6. **Session isolation** - Clients can't access other sessions

---

## 12. Testing

### WebSocket Test (Python)
```python
import asyncio
import websockets
import json

async def test_gateway():
    uri = "ws://localhost:18789"
    async with websockets.connect(uri) as ws:
        # Connect
        await ws.send(json.dumps({
            "type": "connect",
            "mode": "tui",
            "agent": "main",
            "session": "main",
            "auth": {"type": "token", "value": "xxx"}
        }))
        
        response = json.loads(await ws.recv())
        assert response["type"] == "connected"
        
        # Send message
        await ws.send(json.dumps({
            "type": "message",
            "content": "Hello!"
        }))
        
        # Receive response
        msg = json.loads(await ws.recv())
        print(f"Got: {msg}")

asyncio.run(test_gateway())
```

---

## 13. Reference

- Default port: **18789**
- Protocol: **WebSocket** (ws:// or wss://)
- Message format: **JSON**
- Ping interval: **30 seconds**

See also:
- `TUI.md` - Terminal UI client specification
- `PROTOCOLS.md` - ACP protocol details
- `ARCHITECTURE.md` - System architecture
