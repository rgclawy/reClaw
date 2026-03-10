# reClaw Protocols

**Purpose:** Exact protocol formats for channel communication and agent control

---

## 1. ACP (Agent Control Protocol)

### 1.1 Overview

ACP is the protocol for communication between main session and sub-agents. It uses **NDJSON** (newline-delimited JSON) over stdio or WebSocket.

### 1.2 Message Format

Every message is a JSON object followed by newline (`\n`):

```json
{"type": "initialize", "session": "agent:main:subagent:task123", "capabilities": ["read", "web_search"]}
{"type": "thread/start", "thread_id": "thread-abc", "context": {...}}
{"type": "turn/start", "prompt": "Hello, how are you?", "turn_id": "turn-1"}
```

### 1.3 Message Types

#### Client → Agent

| Type | Description | Payload |
|------|-------------|---------|
| `initialize` | Announce client | `{session, capabilities, model?, thinking?}` |
| `thread/start` | Start new thread | `{thread_id, context?}` |
| `turn/start` | Execute turn | `{prompt, turn_id, context?}` |
| `steer` | Guide running agent | `{message, mode}` |
| `interrupt` | Stop current operation | `{reason?}` |
| `close` | Close connection | `{}` |

#### Agent → Client

| Type | Description | Payload |
|------|-------------|---------|
| `initialized` | Acknowledgment | `{session, agent_id}` |
| `thread_started` | Thread created | `{thread_id}` |
| `message` | Text chunk | `{content, chunk_id}` |
| `tool_call` | Request tool execution | `{tool: "read", params: {...}, call_id}` |
| `tool_result` | Tool response | `{call_id, result, error?}` |
| `available_commands` | Commands available | `{commands: ["steer", "interrupt"]}` |
| `turn_complete` | Turn done | `{turn_id, result, metadata?}` |
| `error` | Error occurred | `{error, message?}` |

### 1.4 Complete Example

**Client sends:**
```json
{"type": "initialize", "session": "agent:main:subagent:task123", "capabilities": ["read", "web_search"], "model": "kimi-coding/k2p5"}
{"type": "thread/start", "thread_id": "thread-1"}
{"type": "turn/start", "prompt": "What is the weather?", "turn_id": "turn-1"}
```

**Agent responds:**
```json
{"type": "initialized", "session": "agent:main:subagent:task123", "agent_id": "sub-456"}
{"type": "thread_started", "thread_id": "thread-1"}
{"type": "message", "content": "Let me check that for you", "chunk_id": "c1"}
{"type": "tool_call", "tool": "web_search", "params": {"query": "weather"}, "call_id": "call-1"}
... (tool execution) ...
{"type": "tool_result", "call_id": "call-1", "result": "Sunny, 72°F"}
{"type": "message", "content": "The weather is sunny and 72°F.", "chunk_id": "c2"}
{"type": "turn_complete", "turn_id": "turn-1", "result": "The weather is sunny and 72°F."}
```

---

## 2. Channel Message Normalization

### 2.1 Normalized Message Schema

All channel adapters must normalize messages to this format:

```typescript
interface NormalizedMessage {
  // Required fields
  id: string;                    // Unique message ID (platform:original_id)
  channel_type: string;          // discord, telegram, signal, whatsapp, etc.
  channel_id: string;            // Channel identifier (channel_type:platform_channel_id)
  author_id: string;             // Sender's platform ID
  author_name: string;           // Sender's display name
  content: string;               // Message text content
  timestamp: string;             // ISO-8601 timestamp
  
  // Optional fields
  reply_to?: string;             // Message ID being replied to
  attachments?: Attachment[];    // Attached files
  is_bot?: boolean;              // Whether sender is a bot
  is_group?: boolean;             // Whether in group/dm
  group_subject?: string;        // Group name (if group)
  group_participants?: string[]; // Group member IDs (if group)
  was_mentioned?: boolean;       // Whether bot was @mentioned
  raw?: Record<string, any>;     // Original platform message (for debugging)
}

interface Attachment {
  id: string;
  type: 'image' | 'video' | 'audio' | 'file';
  url: string;
  filename?: string;
  size?: number;
  mime_type?: string;
}
```

### 2.2 ID Generation Rules

- Format: `{channel_type}:{platform_specific_id}`
- Examples:
  - Discord message: `discord:123456789012345678`
  - Discord user: `discord:user:987654321`
  - Discord channel: `discord:channel:111222333`
  - Telegram message: `telegram:123456789012345678`
  - Telegram user: `telegram:user:987654321`

---

## 3. Discord-Specific Protocol

### 3.1 Connection

```typescript
interface DiscordConfig {
  token: string;              // Bot token
  intents: number;            // Gateway intents (from Discord developer portal)
  shards?: number;            // Number of shards (auto if not specified)
}
```

### 3.2 Events to Handle

| Event | Handling |
|-------|----------|
| `MESSAGE_CREATE` | Process message, check mentions |
| `MESSAGE_UPDATE` | Handle edits |
| `MESSAGE_DELETE` | Log deletion |
| `GUILD_CREATE` | Cache guild info |
| `GUILD_MEMBER_ADD/REMOVE` | Member events |
| `VOICE_STATE_UPDATE` | Voice channel state |
| `INTERACTION_CREATE` | Slash commands, buttons |

### 3.3 Send Message Format

```typescript
interface DiscordSendOptions {
  content?: string;           // Message text
  embeds?: Embed[];           // Rich embeds
  components?: Component[];   // Buttons, selects
  files?: File[];             // Attachments
  reply?: string;             // Message ID to reply to
  nonce?: string;             // For deduplication
  tts?: boolean;              // Text-to-speech
}
```

### 3.4 Components V2 (Rich UI)

```typescript
interface Component {
  type: 1 | 2 | 3 | 4 | 5;    // 1=action_row, 2=button, 3=string_select, 4=text_input, 5=role_select
  components?: Component[];   // Nested for action_row
  style?: number;             // For button: 1-6
  label?: string;
  custom_id?: string;
  url?: string;
  disabled?: boolean;
  options?: SelectOption[];
}
```

---

## 4. Telegram-Specific Protocol

### 4.1 Connection

```typescript
interface TelegramConfig {
  bot_token: string;          // From BotFather
  webhook?: string;           // Webhook URL (or use long polling)
  long_poll_options?: {
    timeout?: number;
    limit?: number;
  };
}
```

### 4.2 Available Methods

Must implement:
- `sendMessage(chat_id, text, options?)` → returns Message
- `sendVoice(chat_id, audio, options?)` → returns Message
- `editMessageText(chat_id, message_id, text, options?)` → returns Message
- `deleteMessage(chat_id, message_id)` → returns boolean
- `answerCallbackQuery(callback_query_id, text?)` → returns boolean
- `createChatInviteLink(chat_id, expire_date?, member_limit?)` → returns InviteLink

### 4.3 Inline Keyboard

```typescript
interface InlineKeyboardMarkup {
  inline_keyboard: InlineKeyboardButton[][];
}

interface InlineKeyboardButton {
  text: string;
  url?: string;
  callback_data?: string;
  login_url?: LoginUrl;
}
```

### 4.4 Bot Commands

```typescript
interface BotCommand {
  command: string;    // e.g., "start"
  description: string; // e.g., "Start the bot"
}
```

---

## 5. Twilio Voice Protocol

### 5.1 Webhook Endpoints

```
POST /voice/incoming    # Incoming call
POST /voice/status      # Call status callbacks
```

### 5.2 TwiML Responses

**Gather speech:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Response>
  <Gather input="speech" action="/voice/gather" timeout="5" speechTimeout="5">
    <Say>What would you like to do?</Say>
  </Gather>
  <Say>I didn't hear anything. Goodbye.</Say>
</Response>
```

**Connect to media stream:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Response>
  <Connect>
    <Stream url="wss://your-server.com/voice/stream" />
  </Connect>
</Response>
```

### 5.3 WebSocket Message Format

**From Twilio (audio):**
```json
{
  "event": "media",
  "media": {
    "track": "inbound",
    "chunk": "1",
    "timestamp": "0",
    "payload": "base64_encoded_audio..."
  }
}
```

**To Twilio (audio):**
```json
{
  "event": "media",
  "media": {
    "track": "outbound",
    "payload": "base64_encoded_audio..."
  }
}
```

### 5.4 Call Status Callbacks

| Status | Description |
|--------|-------------|
| `queued` | Call initiated |
| `ringing` | Call is ringing |
| `in-progress` | Call answered |
| `completed` | Call ended normally |
| `busy` | Line busy |
| `failed` | Call failed |
| `no-answer` | No answer |

---

## 6. Webhook Signature Verification

### 6.1 Twilio

```typescript
import crypto from 'crypto';

function verify_twilio_signature(
  url: string,
  params: Record<string, string>,
  signature: string,
  auth_token: string
): boolean {
  const paramsString = Object.keys(params)
    .sort()
    .map(key => `${key}${params[key]}`)
    .join('');
  
  const signature_calculated = crypto
    .createHmac('sha1', auth_token)
    .update(url + paramsString)
    .digest('base64');
  
  return signature === signature_calculated;
}
```

### 6.2 Discord

```typescript
import crypto from 'crypto';

function verify_discord_interaction(
  body: string,
  signature: string,
  timestamp: string,
  public_key: string
): boolean {
  const signature_calculated = crypto
    .createSign('RSA-SHA256')
    .update(timestamp + body)
    .verify(public_key, signature, 'base64');
  
  return signature_calculated;
}
```

---

## 7. Rate Limiting

### 7.1 Discord Rate Limits

| Endpoint | Limit |
|----------|-------|
| Send Message | 5/5s per channel |
| Create Reaction | 1/1s |
| Edit Message | 5/5s |
| Delete Message | 5/5s |

### 7.2 Telegram Rate Limits

| Method | Limit |
|--------|-------|
| sendMessage | 30/m |
| sendMediaGroup | 15/m |
| sendVoice | 15/m |

---

## 8. Error Codes

### 8.1 Channel Errors

| Code | Description | Recovery |
|------|-------------|----------|
| `AUTH_INVALID` | Invalid credentials | Retry after fixing config |
| `AUTH_EXPIRED` | Token expired | Refresh token |
| `RATE_LIMIT` | Rate limited | Backoff and retry |
| `WEBHOOK_INVALID` | Webhook failed | Check URL |
| `MESSAGE_TOO_LONG` | Content exceeds limit | Chunk message |

### 8.2 Voice Errors

| Code | Description |
|------|-------------|
| `TTS_FAILED` | TTS generation failed |
| `STT_FAILED` | Transcription failed |
| `CALL_DROPPED` | Call disconnected unexpectedly |
| `PROVIDER_UNAVAILABLE` | STT/TTS provider down |