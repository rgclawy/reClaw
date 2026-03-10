# Multi-Account Support

**Extension** - Not in core v0, add when needed.

---

## What It Enables

Run multiple accounts on the same channel:
- Two Discord bots on different servers
- Personal + work accounts on Telegram
- Multiple phone numbers on Signal

---

## Implementation Pattern

### Config

```yaml
channels:
  discord:
    accounts:
      default:
        token: $DISCORD_TOKEN
      bot-2:
        token: $DISCORD_TOKEN_2
      work:
        token: $DISCORD_WORK_TOKEN
```

### Session Key Format

```
agent:{agentId}:{channel}:{accountId}:{sessionType}:{id}
```

Examples:
- `agent:main:discord:default:direct:123`    # Default account
- `agent:main:discord:bot-2:direct:456`     # Second account
- `agent:main:discord:work:group:789`       # Work account in group

### Account Selection

```python
# When receiving a message
def select_account(channel, sender):
    # Check which account received the message
    return get_account_for_channel(channel)
    
# When sending - specify account
await adapter.send_message(
    target="discord:channel:123",
    content="Hello",
    account="default"  # Which account to use
)
```

### Per-Account Settings

```yaml
channels:
  discord:
    accounts:
      default:
        token: $DISCORD_TOKEN
        allowFrom: ["+1555..."]
        dmPolicy: open
      work:
        token: $DISCORD_WORK_TOKEN
        allowFrom: ["+1556..."]
        dmPolicy: allowlist
```

---

## Key Points

1. **Separate credentials** - Each account has own token
2. **Separate allowlists** - Different permissions per account
3. **Session isolation** - Messages from different accounts stay separate
4. **CLI targeting** - Use `--account default` to specify which to use

---

## See Also

- `impl/CHANNELS.md` - Base channel adapter pattern
- `impl/CONFIG.md` - Full config schema