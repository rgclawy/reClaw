# reClaw Update Skill

**Purpose:** Update existing Reclaw implementation when specs change

---

## When to Use This Skill

Use this skill when:
- New Reclaw spec released
- You want to compare your implementation with latest specs
- Specs have been updated and you need to update your implementation

---

## What This Skill Does

1. **Pull Latest** - Fetch latest reclaw specs from GitHub main branch
2. **Audit** - Compare current implementation against new specs
3. **Report** - Show what's changed, what's missing
4. **Guide** - Ask user what they want to do next (update, ask questions, etc.)

---

## Steps

### Step 1: Pull Latest Specs

```bash
cd /path/to/reclaw
git fetch origin main
git pull origin main

# Or clone fresh if needed
git clone https://github.com/rgclawy/reclaw.git /tmp/reclaw-latest
```

### Step 2: Audit Against Specs

Compare your implementation with the latest specs:

**Check each impl file:**
- ARCHITECTURE.md - Any new components?
- CONFIG.md - Any new config options?
- PROTOCOLS.md - Any protocol changes?
- CHANNELS.md - New channels?
- VOICE.md - New voice features?
- AGENTS.md - New agent features?
- MEMORY.md - New memory features?

### Step 3: Report Findings

Generate a report:

```
## Spec Changes Since Last Update

### New Features
- [list new features added]

### Breaking Changes
- [list any breaking changes]

### Missing from Implementation
- [list what your implementation is missing]

### Recommendations
- [suggestions for updates]
```

### Step 4: Ask User

After audit, ask user:
1. **Questions?** "Do you have any questions about the new specs?"
2. **Update?** "Want me to update the implementation to match new specs?"
3. **Skip?** "Want to skip this update and continue as-is?"

---

## Audit Questions

When auditing, check:

### Architecture
- New components added?
- Interface changes?
- New patterns?

### Channels
- New channels supported?
- New DM policies?
- New features per channel?

### Voice
- New TTS/STT providers?
- New phone providers?
- Audio format changes?

### Agents
- ACP protocol changes?
- New queue modes?
- New concurrency options?

### Memory
- New search features?
- New storage options?
- Index format changes?

### Config
- New config options?
- Schema changes?
- New environment variables?

---

## Example Audit Output

```
## reClaw Spec Update Audit
Date: 2026-03-09

### Files Changed
- SPEC.md (major update)
- impl/CHANNELS.md (new channels)
- impl/MEMORY.md (hybrid search added)

### Your Implementation Status
- Channels: 3/15 (Discord, Telegram, WhatsApp)
- Voice: Partial (TTS only, no STT)
- Agents: Basic ACP implemented
- Memory: Short-term only

### New in Specs
- 12 more channels
- Full voice (TTS + STT + Twilio)
- Advanced agent features
- Hybrid search in memory

### Recommendation
Consider adding: STT, Twilio, and hybrid memory search
```

---

## What To Do Next

After audit, prompt user:

```
📋 **Audit Complete**

What's next?
1. **Q&A** - Ask questions about new specs
2. **Update** - Update implementation to match new specs
3. **Skip** - Continue without updating
4. **Partially Update** - Update specific components only
```