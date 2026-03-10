# Voice Extensions

**Extension** - Add voice capabilities as needed.

---

## Available Voice Options

### TTS (Text-to-Speech)

| Provider | Quality | Cost | Status |
|----------|---------|------|--------|
| OpenAI | High | Paid | ✅ Available |
| Edge | Good | Free | ✅ Available |
| ElevenLabs | Highest | Paid | ✅ Available |

### STT (Speech-to-Text)

| Provider | Quality | Cost | Status |
|----------|---------|------|--------|
| Whisper | High | Paid | ✅ Available |
| Deepgram | High | Paid | ✅ Available |
| Google | Good | Paid | ✅ Available |

### Phone

| Provider | Status |
|----------|--------|
| Twilio | ✅ Available |
| Telnyx | ✅ Available |
| Plivo | ✅ Available |

---

## Implementation Pattern

### Config

```yaml
voice:
  # Choose TTS provider
  default_provider: openai
  providers:
    openai:
      api_key: $OPENAI_API_KEY
    edge_tts:
      enabled: true
    elevenlabs:
      api_key: $ELEVENLABS_API_KEY
  
  # Choose STT provider
  default_stt: whisper
  stt:
    whisper:
      api_key: $OPENAI_API_KEY
    deepgram:
      api_key: $DEEPGRAM_API_KEY
  
  # Phone (optional)
  twilio:
    account_sid: $TWILIO_ACCOUNT_SID
    auth_token: $TWILIO_AUTH_TOKEN
    phone_number: $TWILIO_PHONE_NUMBER
```

### Usage

```python
# TTS
audio = await voice.speak("Hello", provider="openai")

# STT
text = await voice.transcribe(audio, provider="whisper")

# Phone call
call = await voice.make_call("+15555551234", twiml)
```

---

## Start Minimal

| Minimal | Full |
|---------|------|
| OpenAI TTS + Whisper STT | All providers |
| Twilio | All phone providers |

Start with OpenAI for TTS/STT + Twilio for phone. Add more as needed.

---

## See Also

- `impl/VOICE.md` - Base voice system
- `extensions/edge-tts.md` - Free TTS guide
- `impl/CONFIG.md` - Full voice config