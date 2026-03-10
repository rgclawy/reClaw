# DM Policies (Access Control)

**Extension** - Not in core v0, add when needed.

---

## What It Enables

Control who can message your assistant:

| Policy | Behavior | Use Case |
|--------|----------|----------|
| `open` | Anyone can message | Public bot |
| `allowlist` | Only approved users | Private bot |
| `pairing` | First-time users need approval | Community |
| `disabled` | No DMs allowed | Server-only |

---

## Implementation Pattern

### Config

```yaml
channels:
  discord:
    dmPolicy: pairing          # Default policy
    groupPolicy: open         # Group messages
    allowFrom:
      - "+15555550123"       # Phone numbers
      - "<@123456789>"        # Discord IDs
      - "uuid:abc-123"        # Generic ID
```

### Policy Check

```python
async def check_dm_policy(sender_id, policy, allowlist):
    match policy:
        case "disabled":
            return {"allowed": False, "reason": "DM disabled"}
        
        case "open":
            return {"allowed": True}
        
        case "allowlist":
            if sender_id in allowlist:
                return {"allowed": True}
            return {"allowed": False, "reason": "Not in allowlist"}
        
        case "pairing":
            if sender_id in allowlist:
                return {"allowed": True}
            # Generate pairing code
            code = generate_pairing_code()
            return {
                "allowed": False, 
                "requires_pairing": True,
                "code": code,
                "message": f"Send `{code}` to pair your account"
            }
```

### Pairing Flow

```
1. Unknown user sends DM
2. System generates unique pairing code (e.g., "ABC123")
3. User must send the code back
4. System adds user to allowlist
5. User can now message freely
```

```python
async def handle_pairing(message):
    code = message.content.strip().upper()
    
    pending = get_pending_pairing(code)
    if pending:
        # Add to allowlist
        add_to_allowlist(pending.user_id)
        remove_pending_pairing(code)
        return {"success": True, "message": "Paired! You can now message me."}
    
    return {"success": False, "message": "Invalid pairing code."}
```

### Group Policies

```yaml
channels:
  discord:
    groupPolicy: open        # Anyone in group can message
    groupPolicy: allowlist  # Only allowlisted group members
```

---

## Channel-Specific Notes

### Discord
- DM via user ID
- Group via channel ID
- Supports @mention detection

### Telegram
- DM via chat ID
- Supports inline buttons for pairing

### Signal
- DM via phone number
- Ephemeral messages option

---

## See Also

- `impl/CHANNELS.md` - Channel adapter pattern
- `impl/CONFIG.md` - Full config schema