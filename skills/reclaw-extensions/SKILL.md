# reClaw Extension Management Skill

**Purpose:** Add or remove Reclaw extensions at any time.

---

## When to Use This Skill

Use this skill when:
- Adding new extensions to an existing Reclaw implementation
- Removing extensions you no longer need
- Checking which extensions are installed
- Updating extension configurations

---

## Available Extensions

| Extension | Description | Status |
|-----------|-------------|--------|
| multi-account | Multiple accounts per channel | Available |
| dm-policies | Pairing, allowlist, open, disabled | Available |
| hybrid-search | Vector + keyword memory search | Available |
| edge-tts | Free TTS alternative | Available |

---

## Commands

### 1. List Installed Extensions

```
Show me all installed extensions and their status.
```

### 2. Add Extension

```
Add the [extension-name] extension to my Reclaw.
```

Example:
```
Add the multi-account extension to my Reclaw.
```

### 3. Remove Extension

```
Remove the [extension-name] extension from my Reclaw.
```

Example:
```
Remove the hybrid-search extension from my Reclaw.
```

### 4. Configure Extension

```
Configure [extension-name] with these options: [options]
```

Example:
```
Configure dm-policies with:
- default policy: pairing
- allowlist: ["+15555551234"]
```

### 5. Check Requirements

```
What do I need to add the [extension-name] extension?
```

---

## Extension Requirements

### multi-account
- Requires: Config updates for account credentials
- No new dependencies

### dm-policies
- Requires: Code changes for policy handling
- No new dependencies

### hybrid-search
- Requires: OpenAI/Gemini/Voyage API key (for embeddings)
- Dependencies: openai, or gemini, or voyage

### edge-tts
- Requires: edge-tts library
- Dependencies: edge-tts

---

## Extension Interface

All extensions follow this pattern:

```python
class Extension:
    name: str           # "multi-account"
    version: str        # "1.0.0"
    requires: list      # ["openai-api-key"]
    
    async def install(self, config) -> void:
        """Called when extension is added"""
        pass
    
    async def uninstall(self) -> void:
        """Called when extension is removed"""
        pass
    
    async def load(self, config) -> void:
        """Called when Reclaw starts"""
        pass
    
    def get_dependencies(self) -> list[str]:
        """List required packages"""
        return []
```

---

## How Extensions Work

### Installation Flow

```
1. User requests: "Add multi-account extension"
2. Skill checks requirements
3. If dependencies missing, prompt user to install
4. Add extension config to config.yaml
5. Import extension code into project
6. Update imports/registry
7. Test and verify
```

### Runtime Loading

```
Reclaw starts
  ↓
Load config.yaml
  ↓
Load enabled extensions
  ↓
Register extension handlers
  ↓
Start normal operation
```

### Removal Flow

```
1. User requests: "Remove multi-account"
2. Confirm removal
3. Remove from config
4. Remove or comment extension code
5. Test and verify
```

---

## Examples

### Example 1: Add Multi-Account

**User:** "Add multi-account support"

**AI Response:**
```
I'll add multi-account support to your Reclaw.

First, let me check your current config...
[Check config.yaml]

Now I'll add the multi-account configuration:

1. Add account structure to config
2. Show you how to add credentials
3. Update session handling for account isolation

Here's what I'll add to your config.yaml:

```yaml
channels:
  discord:
    accounts:
      default:
        token: $DISCORD_TOKEN
      bot-2:
        token: $DISCORD_TOKEN_2
```

Ready to proceed? Just say "go ahead" or "yes".
```

### Example 2: Add Hybrid Search

**User:** "I want better memory search"

**AI Response:**
```
I'll add hybrid search (vector + keyword) to your memory system.

Requirements:
- OpenAI API key (or Gemini/Voyage/Ollama)
- Additional storage for vector embeddings

Let me check what you have currently...
[Check current memory setup]

Options:
1. OpenAI embeddings (requires $OPENAI_API_KEY)
2. Gemini embeddings (requires $GEMINI_API_KEY)
3. Ollama (local, no API key)

Which would you prefer? Or should I set up all three as fallback?
```

---

## After Adding Extension

Once an extension is added:

1. **Restart** your Reclaw
2. **Test** the new feature
3. **Configure** as needed in config.yaml

The extension integrates seamlessly with core Reclaw.