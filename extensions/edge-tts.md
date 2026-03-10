# Edge TTS (Free Voice)

**Extension** - Not in core v0, add when needed.

---

## What It Enables

Free text-to-speech using Microsoft's Edge browser voices. No API key required.

**Quality:** Good (neural voices)
**Cost:** Free
**Languages:** 100+

---

## Implementation Pattern

### Install

```bash
# Python
pip install edge-tts

# Node.js
npm install edge-tts
```

### Basic Usage

```python
import edge_tts

async def speak(text, voice="en-US-JennyNeural"):
    communicate = edge_tts.Communicate(text, voice)
    audio = b""
    
    async for chunk in communicate.stream():
        if chunk["type"] == "audio":
            audio += chunk["data"]
    
    return audio
```

### Voice List

```python
import asyncio

async def list_voices():
    voices = await edge_tts.get_voices()
    for voice in voices:
        if voice["Locale"].startswith("en"):
            print(f"{voice['ShortName']}: {voice['Locale']}")
```

### Common Voices

| Voice | Language | Style |
|-------|----------|-------|
| en-US-JennyNeural | English (US) | Conversational |
| en-GB-SoniaNeural | English (UK) | Conversational |
| zh-CN-XiaoxiaoNeural | Chinese | Neural |
| ja-JP-NanamiNeural | Japanese | Neural |

---

## As Primary TTS

Replace OpenAI TTS with Edge for free:

```python
class VoiceAdapter:
    def __init__(self, config):
        self.config = config
    
    async def speak(self, text, options=None):
        voice = options.get("voice", "en-US-JennyNeural") if options else "en-US-JennyNeural"
        
        if self.config.get("use_edge"):
            return await self.edge_tts(text, voice)
        else:
            return await self.openai_tts(text, options)
    
    async def edge_tts(self, text, voice):
        communicate = edge_tts.Communicate(text, voice)
        
        audio_chunks = []
        async for chunk in communicate.stream():
            if chunk["type"] == "audio":
                audio_chunks.append(chunk["data"])
        
        return b"".join(audio_chunks)
```

---

## Pros & Cons

| Pros | Cons |
|------|------|
| Free | Slightly lower quality than OpenAI |
| No API key | Requires internet |
| Many languages | Rate limits |
| Fast | Not as expressive |

---

## See Also

- `impl/VOICE.md` - Base voice system