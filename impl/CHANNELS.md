# Channel Implementation Guide

**Purpose:** How to implement channel adapters - language-agnostic patterns

---

## 1. What is a Channel Adapter?

A channel adapter bridges reClaw to a messaging platform (Discord, Telegram, etc.)

```
Platform → [Channel Adapter] → Normalized Message → reClaw Core
reClaw Core → [Channel Adapter] → Platform Message → User
```

---

## 2. Interface (All Languages)

Every channel adapter implements this interface:

```
class ChannelAdapter:
    
    # Lifecycle
    async connect() -> void
    async disconnect() -> void
    
    # Send
    async send_message(target: string, content: string, options?: dict) -> message_id: string
    async send_voice(target: string, audio_data: bytes) -> message_id: string
    
    # Receive - set callbacks
    on_message(callback: function) -> void
    on_voice(callback: function) -> void
    on_error(callback: function) -> void
```

---

## 3. Pattern: How It Works

### 3.1 Connection

```
1. Initialize with config (token, API keys, etc.)
2. Connect to platform (WebSocket, polling, or webhook)
3. Handle connection events (connected, error, reconnect)
```

### 3.2 Receiving Messages

```
1. Platform sends event (new message, etc.)
2. Adapter converts to NormalizedMessage format
3. Call all registered on_message callbacks
```

### 3.3 Sending Messages

```
1. Receive target + content
2. Format for platform (markdown conversion, chunking, etc.)
3. Send via platform API
4. Return message_id
```

---

## 4. Normalized Message Format

All adapters output this same format:

```
NormalizedMessage {
    id: string              # "discord:123456789"
    channel_type: string    # "discord", "telegram", etc.
    channel_id: string      # "discord:channel:987654321"
    author_id: string       # "discord:user:111222333"
    author_name: string
    content: string
    timestamp: string       # ISO-8601
    reply_to?: string       # Message ID being replied to
    attachments?: list      # File attachments
    is_group?: boolean
    group_subject?: string   # Group name
    was_mentioned?: boolean  # Bot was @mentioned
}
```

---

## 5. Implement Each Channel

### 5.1 Discord

**Prerequisites:**
- Bot token from Discord Developer Portal
- Gateway intents (message content, guilds, etc.)

**Platform quirks:**
- 2000 char limit per message (chunk longer messages)
- Supports embeds, components, threads
- Has slash commands

**Key methods:**
- Gateway for real-time (WebSocket)
- REST API for sending
- Webhook for interactions

### 5.2 Telegram

**Prerequisites:**
- Bot token from BotFather

**Platform quirks:**
- 4096 char limit
- Supports inline keyboards, polls, stickers
- Long polling OR webhook

**Key methods:**
- sendMessage(chat_id, text, options?)
- sendVoice(chat_id, audio)
- answerCallbackQuery(callback_id)

### 5.3 Signal

**Prerequisites:**
- signal-cli or REST API

**Platform quirks:**
- Phone number based
- No rich formatting
- Ephemeral messages

### 5.4 WhatsApp

**Prerequisites:**
- Baileys library or WhatsApp Business API

**Platform quirks:**
- Media handling (images, voice)
- Group management
- Read receipts

### 5.5 Slack

**Prerequisites:**
- Bot token + App token (for Socket Mode)

**Platform quirks:**
- Block Kit for rich UI
- Socket Mode for real-time
- App Home for persistent UI

---

## 6. Common Features

### 6.1 Message Chunking

```
function chunk_message(text: string, max_length: int) -> list[str]:
    if len(text) <= max_length:
        return [text]
    
    chunks = []
    lines = text.split("\n")
    current = ""
    
    for line in lines:
        if len(current) + len(line) + 1 <= max_length:
            current += "\n" + line
        else:
            chunks.append(current)
            current = line
    
    if current:
        chunks.append(current)
    
    return chunks
```

### 6.2 DM Policy

```
function check_dm_policy(sender_id: str, policy: str, allowlist: list) -> dict:
    match policy:
        "disabled":
            return {allowed: False, reason: "DM disabled"}
        "allowlist":
            if sender_id in allowlist:
                return {allowed: True}
            return {allowed: False, reason: "Not in allowlist"}
        "pairing":
            if sender_id in allowlist:
                return {allowed: True}
            # Generate pairing code
            code = generate_pairing_code()
            return {allowed: False, requires_pairing: True, code: code}
        "open":
            return {allowed: True}
```

### 6.3 Echo Prevention

```
class EchoTracker:
    def __init__(self, max_items=100):
        self.recent = {}
        self.max_items = max_items
    
    def add(self, content: str, message_id: str) -> void:
        # Remove oldest if full
        if len(self.recent) >= self.max_items:
            oldest = next(iter(self.recent))
            del self.recent[oldest]
        
        hash_key = simple_hash(content)
        self.recent[hash_key + ":" + message_id] = now()
    
    def has(self, content: str, message_id: str) -> bool:
        hash_key = simple_hash(content)
        return (hash_key + ":" + message_id) in self.recent
```

---

## 7. Language Tips

Once you pick a language, add these:

### Python
```python
# HTTP requests
requests, aiohttp

# WebSocket
websockets, socketio

# Platform SDKs
discord.py, grammY, slack-bolt
```

### TypeScript/JavaScript
```javascript
// HTTP requests
axios, node-fetch

// WebSocket
ws, socket.io

// Platform SDKs
discord.js, grammY, @slack/bolt
```

### Go
```go
// HTTP requests
net/http, fasthttp

// WebSocket
gorilla/websocket

// Platform SDKs
discordgo, grammY (via WASM)
```

### Rust
```rust
// HTTP requests
reqwest, actix-web

// WebSocket
tokio-tungstenite

// Platform SDKs
serenity (Discord), telegram-bot (Telegram)
```

---

## 8. Testing

### Test Checklist

- [ ] Connect to platform
- [ ] Receive message → normalize
- [ ] Send text message
- [ ] Handle reply
- [ ] Send attachment
- [ ] DM policy works
- [ ] Echo prevention
- [ ] Disconnect cleanly

### Test with Real Messages

```python
# Pseudocode test
async def test_discord():
    adapter = DiscordAdapter(config)
    await adapter.connect()
    
    # Send test message
    msg_id = await adapter.send_message(
        target="discord:channel:123",
        content="Hello from test!"
    )
    
    assert msg_id.startswith("discord:")
    await adapter.disconnect()
```

---

## 9. Reference

See also:
- `PROTOCOLS.md` - Exact protocol formats
- `CONFIG.md` - Channel configuration schema
- `ARCHITECTURE.md` - System design