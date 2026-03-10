# Voice System Implementation

**Purpose:** How to implement TTS, STT, and phone calls - language-agnostic patterns

---

## 1. Voice Components

```
┌─────────────────────────────────────────────┐
│              Voice System                    │
├─────────────────────────────────────────────┤
│                                              │
│  STT (Speech-to-Text)                        │
│  Audio → Transcript                           │
│                                              │
│  TTS (Text-to-Speech)                        │
│  Text → Audio                                 │
│                                              │
│  Phone (Twilio)                              │
│  Call handling + streaming                   │
│                                              │
└─────────────────────────────────────────────┘
```

---

## 2. Interface

```
class VoiceAdapter:
    
    # TTS
    async speak(text: string, options?: dict) -> audio: bytes
    
    # STT  
    async transcribe(audio: bytes, options?: dict) -> text: string
    
    # Phone
    async handle_incoming_call(webhook: dict) -> CallSession
    async make_call(to: string, twiml: str) -> CallSession
    async stream_audio(call_id: string, audio: bytes) -> void
    async end_call(call_id: string) -> void
```

---

## 3. TTS (Text-to-Speech)

### 3.1 Pattern

```
function speak(text: str, provider: str, options?: dict) -> audio:
    # 1. Select provider
    provider_impl = get_provider(provider)  # openai, edge, elevenlabs
    
    # 2. Build request
    request = {
        text: text,
        voice: options.voice or default,
        speed: options.speed or 1.0,
        format: options.format or "mp3"
    }
    
    # 3. Call provider API
    response = await provider_impl.speak(request)
    
    # 4. Return audio
    return response.audio
```

### 3.2 Providers

| Provider | Auth | Quality | Cost |
|----------|------|---------|------|
| OpenAI | API key | High | Paid |
| Edge TTS | None | Good | Free |
| ElevenLabs | API key | Highest | Paid |

### 3.3 Voice Selection

```
# Map logical voice names to provider-specific IDs
VOICE_MAP = {
    "default": {
        "openai": "alloy",
        "edge": "en-US-JennyNeural",
        "elevenlabs": "21m00..."
    },
    "male": {
        "openai": "onyx",
        "edge": "en-US-GuyNeural",
        "elevenlabs": "..."
    }
}
```

---

## 4. STT (Speech-to-Text)

### 4.1 Pattern

```
function transcribe(audio: bytes, provider: str, options?: dict) -> text:
    # 1. Normalize audio (16kHz mono WAV)
    normalized = normalize_audio(audio)
    
    # 2. Select provider
    provider_impl = get_provider(provider)  # whisper, deepgram
    
    # 3. Call API
    response = await provider_impl.transcribe(normalized, {
        language: options.language or "auto"
    })
    
    # 4. Return text
    return response.text
```

### 4.2 Providers

| Provider | Auth | Best For |
|----------|------|----------|
| OpenAI Whisper | API key | General |
| Deepgram | API key | Low latency |
| Google Cloud | API key | Accuracy |

### 4.3 Audio Normalization

```
function normalize_audio(audio: bytes, target_sample_rate=16000) -> bytes:
    # Convert to consistent format
    # - Sample rate: 16kHz
    # - Channels: mono
    # - Format: WAV or as provider expects
    
    # Use library like ffmpeg or platform-specific tools
    return converted_audio
```

---

## 5. Phone Calls (Twilio)

### 5.1 Flow

```
Incoming Call:
  1. Twilio → POST /voice/incoming
  2. Return TwiML → <Gather> or <Connect>
  3. Twilio → WebSocket /voice/stream
  4. Stream audio back and forth
  5. On end → cleanup

Outgoing Call:
  1. Your code → call API
  2. Twilio → calls number
  3. Answer → connect to WebSocket
  4. Same as above
```

### 5.2 TwiML (Response to Twilio)

```xml
<!-- Gather speech -->
<Response>
  <Gather input="speech" action="/voice/gather" timeout="5">
    <Say>What would you like to do?</Say>
  </Gather>
</Response>

<!-- Connect to stream -->
<Response>
  <Connect>
    <Stream url="wss://your-server.com/voice/stream" />
  </Connect>
</Response>
```

### 5.3 WebSocket Messages

From Twilio (incoming audio):
```
{
  "event": "media",
  "media": {
    "track": "inbound",
    "payload": "base64_audio..."
  }
}
```

To Twilio (outgoing audio):
```
{
  "event": "media", 
  "media": {
    "track": "outbound",
    "payload": "base64_audio..."
  }
}
```

### 5.4 Call State

```
CallSession {
    id: string
    caller_id: string
    status: "ringing" | "active" | "completed" | "failed"
    started_at: datetime
    transcript: list[TranscriptEntry]
}
```

---

## 6. Voice Message (Not Phone Calls)

For voice messages in chat (Telegram, Discord):

```
1. Receive voice message from channel
2. Download audio file
3. Normalize to 16kHz mono
4. Transcribe (STT)
5. Process text through agent
6. Generate TTS from response
7. Send voice message back
```

---

## 7. Language Tips

### Python
```python
# TTS
openai
edge-tts
elevenlabs

# STT  
openai (whisper)
deepgram

# Audio processing
pydub, scipy, ffmpeg-python
```

### TypeScript
```javascript
# TTS
openai (Audio API)
edge-tts (npm package)
elevenlabs (npm package)

# STT
openai (Whisper API)
deepgram (SDK)

# Audio
ffmpeg-static, wav-encoder
```

### Go
```go
# TTS
go-openai (has TTS)
edge-tts (via exec or WASM)

# STT
deepgram (SDK)
whisper.cpp (bindings)

# Audio
av (libav binding)
```

---

## 8. Testing

### Test TTS
```python
async def test_tts():
    voice = VoiceAdapter(config)
    audio = await voice.speak("Hello world", provider="openai")
    assert len(audio) > 0
    assert audio[:4] == b"RIFF"  # WAV header or MP3 header
```

### Test STT
```python
async def test_stt():
    voice = VoiceAdapter(config)
    # Load sample audio
    audio = load_file("test.wav")
    text = await voice.transcribe(audio, provider="whisper")
    assert "hello" in text.lower()
```

### Test Twilio
```python
async def test_incoming_call():
    # Simulate Twilio webhook
    webhook = {
        "CallSid": "CA123",
        "From": "+15555550123",
        "To": "+15555551234"
    }
    session = await voice.handle_incoming_call(webhook)
    assert session.status == "ringing"
```

---

## 9. Reference

See also:
- `ARCHITECTURE.md` - System design
- `CONFIG.md` - Voice config schema
- `DATA_FORMATS.md` - Audio format specs