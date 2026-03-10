# reClaw Feature Overview

## Structure

reClaw has two parts:

### Core (Minimal, Always Included)
- Gateway Server (WebSocket, HTTP API)
- CLI (minimal command-line interface)
- Sub-agent system (ACP protocol)
- Memory (short-term + long-term files)
- Config (YAML + $VAR)

### Extensions (Your Choice)
All channels, voice providers, and features are extensions.

---

## Extensions

### Channels

| Channel | Description |
|---------|-------------|
| Discord | Primary |
| Telegram | Primary |
| WhatsApp | Primary |
| Slack | Common |
| Signal | Privacy |
| iMessage | macOS |
| LINE | Asia |
| Feishu/Lark | Enterprise China |
| Matrix | Decentralized |
| Mattermost | Self-hosted |
| Google Chat | Enterprise |
| IRC | Legacy |
| Nostr | Decentralized |
| MS Teams | Enterprise |
| Twitch | Streaming |

### Voice TTS

| Provider | Quality | Cost |
|----------|---------|------|
| OpenAI | High | Paid |
| Edge | Good | Free |
| ElevenLabs | Highest | Paid |

### Voice STT

| Provider | Quality | Cost |
|----------|---------|------|
| Whisper | High | Paid |
| Deepgram | High | Paid |
| Google | Good | Paid |

### Phone

| Provider | Status |
|----------|--------|
| Twilio | Primary |
| Telnyx | Alternative |
| Plivo | Alternative |

### Clients

| Client | Description | Status |
|--------|-------------|--------|
| CLI | Single CLI tool with all commands | Core |
| TUI | Rich interactive terminal UI | Extension (via `reclaw tui`)

### Web Tools

| Tool | Description | Status |
|------|-------------|--------|
| web_fetch | HTTP fetch + HTML extraction | Core (always) |
| web_search | Search providers (Brave, Perplexity, etc.) | Extension |
| browser | Browser automation (Playwright/Puppeteer) | Extension |

### Features

| Feature | Description |
|---------|-------------|
| Multi-account | Multiple accounts per channel |
| DM Policies | Pairing, allowlist, open, disabled |
| Hybrid Search | Vector + keyword memory search |

---

## How Extensions Work

1. **Select** - Choose which extensions you need
2. **Configure** - Add to config.yaml
3. **Install** - Install required packages
4. **Use** - Start with selected features

Extensions can be added or removed anytime.

---

## See Also

- `extensions/channels.md` - Channel setup
- `extensions/voice.md` - Voice setup
- `extensions/multi-account.md` - Multi-account
- `extensions/dm-policies.md` - Access control
- `extensions/hybrid-search.md` - Memory search
- `extensions/tui.md` - Rich Terminal UI
- `extensions/web-search.md` - Web search providers
- `extensions/browser.md` - Browser automation