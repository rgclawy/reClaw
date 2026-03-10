# Channel Extensions

**Extension** - Add channels as needed.

---

## Available Channels

| Channel | Status |
|---------|--------|
| Discord | ✅ Available |
| Telegram | ✅ Available |
| WhatsApp | ✅ Available |
| Slack | ✅ Available |
| Signal | ✅ Available |
| iMessage | ✅ Available |
| LINE | ✅ Available |
| Feishu/Lark | ✅ Available |
| Matrix | ✅ Available |
| Mattermost | ✅ Available |
| Google Chat | ✅ Available |
| IRC | ✅ Available |
| Nostr | ✅ Available |
| MS Teams | ✅ Available |
| Twitch | ✅ Available |

---

## Implementation Pattern

### Config

```yaml
channels:
  discord:
    enabled: true
    token: $DISCORD_TOKEN
  telegram:
    enabled: true
    bot_token: $TELEGRAM_BOT_TOKEN
```

### Add Channel

1. Choose channel from list above
2. Add config to config.yaml
3. Install platform SDK
4. Implement adapter (see pattern in impl/CHANNELS.md)

---

## Core vs Extended

**Minimal Setup:**
Start with just the channels you need.

| Minimal | Full |
|---------|------|
| 1-2 channels | All 15+ channels |

Start small, add more as needed.

---

## See Also

- `impl/CHANNELS.md` - Base channel adapter pattern
- `impl/CONFIG.md` - Full config schema