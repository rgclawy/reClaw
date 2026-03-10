# reClaw Migration Skill

**Purpose:** Migrate from OpenClaw to reClaw

---

## When to Use This Skill

Use this skill when:
- You have an existing OpenClaw setup
- You want to move to reClaw
- Need to transfer config, memory, or workflow

---

## What Can Be Migrated

| Item | Migratable? | Notes |
|------|-------------|-------|
| Config | ⚠️ Partial | Need to convert JSON → YAML |
| Memory | ⚠️ Partial | File format similar |
| Workflow | ⚠️ Manual | Review required |
| Extensions | ❌ No | Start fresh |

---

## Migration Steps

### Step 1: Audit Current OpenClaw

What do you have in OpenClaw?

```bash
# Check config
cat ~/.openclaw/openclaw.json 2>/dev/null || echo "No openclaw.json found"

# Check memory
ls -la ~/.openclaw/workspace/memory/ 2>/dev/null || echo "No memory directory found"

# Check extensions
ls ~/.openclaw/workspace/ 2>/dev/null || echo "No workspace found"
```

List what you want to migrate:

- [ ] Config (need conversion)
- [ ] Memory files
- [ ] Workflow/prompts
- [ ] Nothing (start fresh)

---

### Step 2: Verify reClaw Implementation Exists

**Before migrating, ensure you have a working reClaw implementation:**

```bash
# Check for reClaw in target directory (default: ./reclaw)
if [ ! -f "{target_dir}/config.yaml" ] && [ ! -f "{target_dir}/reclaw.py" ]; then
    echo "❌ No reClaw implementation found at {target_dir}"
    echo "Please run the reclaw-implementation skill first to build reClaw."
    exit 1
fi
```

If no reClaw exists:
```
⚠️ **reClaw Implementation Required**

You need a working reClaw implementation before migrating.

Options:
1. Run reclaw-implementation skill to build reClaw first
2. Point to an existing reClaw directory

Cannot proceed with migration until reClaw is built.
```

---

### Step 3: Convert Config

OpenClaw uses JSON → reClaw uses YAML

**Manual conversion needed:**
- Copy `openclaw.json` fields
- Convert to `config.yaml` format
- Update paths

Example conversion:

```json
// OpenClaw (JSON)
{
  "agents": {
    "defaults": {
      "model": {"primary": "kimi-coding/k2p5"}
    }
  }
}
```

```yaml
# reClaw (YAML)
agents:
  default_model: kimi-coding/k2p5
```

**Save the converted config to:** `{target_dir}/config.yaml`

---

### Step 4: Copy Memory (Optional)

```bash
# Create memory directory if it doesn't exist
mkdir -p {target_dir}/memory

# Copy OpenClaw memory files
cp -r ~/.openclaw/workspace/memory/* {target_dir}/memory/ 2>/dev/null || echo "No memory to copy"

# List what was copied
ls -la {target_dir}/memory/
```

Review and clean memory files as needed.

---

### Step 5: Test Core CLI (CRITICAL)

**Verify the core CLI works after migration:**

```bash
cd {target_dir}

# Test 1: CLI basic commands
echo "=== Testing Core CLI ==="

# Version and help
reclaw --version 2>/dev/null || python -m reclaw --version 2>/dev/null || echo "❌ CLI not accessible"
reclaw --help 2>/dev/null || python -m reclaw --help 2>/dev/null || echo "❌ CLI help failed"

# Config commands
reclaw config validate 2>/dev/null || echo "⚠️ Config validate not available"
reclaw config get 2>/dev/null || echo "⚠️ Config get not available"

# Gateway status (before start)
reclaw gateway status 2>/dev/null || echo "⚠️ Gateway status not available"
```

**CLI Test checklist:**
- [ ] `reclaw --version` works
- [ ] `reclaw --help` works
- [ ] `reclaw config validate` works
- [ ] `reclaw gateway status` works (may show "stopped")
- [ ] CLI responds without errors

---

### Step 6: Start Gateway & Test Full CLI

**Start the reClaw gateway using CLI:**

```bash
cd {target_dir}

# Start the gateway using CLI command
reclaw gateway start 2>/dev/null || \
reclaw gateway 2>/dev/null || \
python -m reclaw gateway start 2>/dev/null || \
python -m reclaw gateway 2>/dev/null || \
echo "❌ Gateway start command failed"

# Check if gateway started successfully
if pgrep -f "reclaw.*gateway\|18789" > /dev/null; then
    echo "✅ Gateway is running on port 18789"
else
    echo "⚠️ Gateway may not have started - check logs above"
fi
```

**Test CLI with running gateway:**
```bash
# Test gateway status
reclaw gateway status 2>/dev/null && echo "✅ Gateway status works" || echo "❌ Gateway status failed"

# Test agent list
reclaw agent list 2>/dev/null && echo "✅ Agent list works" || echo "⚠️ Agent list not available"

# Test agent run (optional)
# reclaw agent run "hello" 2>/dev/null && echo "✅ Agent run works" || echo "⚠️ Agent run not available"
```

---

### Step 7: Verify Migration

**Final verification steps:**

```bash
# 1. Check gateway is listening on port 18789
netstat -tlnp 2>/dev/null | grep 18789 || \
ss -tlnp 2>/dev/null | grep 18789 || \
lsof -i :18789 2>/dev/null || \
echo "Port check unavailable"

# 2. Test config is valid
cd {target_dir}
python -c "import yaml; yaml.safe_load(open('config.yaml'))" 2>/dev/null && echo "✅ Config YAML is valid"

# 3. Check memory files are accessible
ls -la {target_dir}/memory/ 2>/dev/null
```

**Report to user:**
```
🎉 **Migration Complete!**

✅ Config: Converted from OpenClaw JSON → reClaw YAML
✅ Memory: Copied to {target_dir}/memory/
✅ Tests: Passed
✅ Gateway: Running on port 18789

Your reClaw is ready to use!

**Next Steps:**
1. Connect your TUI: {tui_command}
2. Test a message through your configured channels
3. Review migrated memory and clean up if needed

**OpenClaw is preserved at:** ~/.openclaw/
**New reClaw is at:** {target_dir}
```

---

## Q&A

### Q: Can I keep both running?
A: Yes, they can run on different ports. OpenClaw typically uses different ports than reClaw's 18789.

### Q: What about extensions?
A: Start fresh with extensions. Review what you had in OpenClaw and add selectively using the reclaw-extensions skill.

### Q: What if I can't convert something?
A: Skip it. Start with minimal reClaw and add back what you need.

### Q: How do I know it works?
A: Run tests from TEST_PLAN.md or manually test basic message flow. The migration skill now runs tests and starts the gateway automatically.

### Q: What if tests fail after migration?
A: Check:
1. Config YAML syntax is valid
2. All required environment variables are set
3. Memory files don't have incompatible formats
4. Review error messages and fix issues

### Q: Gateway won't start?
A: Check:
1. Port 18789 is not already in use: `lsof -i :18789`
2. Config file exists and is valid
3. Required dependencies are installed
4. Check logs for specific errors

---

## What NOT to Migrate

- ❌ Internal code structure
- ❌ Platform-specific SDKs (use reClaw extensions)
- ❌ OpenClaw-specific features not in reClaw spec

---

## After Migration

1. ✅ Verify core works (message → response)
2. ✅ Add needed extensions (use reclaw-extensions skill)
3. ✅ Update memory as needed
4. ✅ Test end-to-end

---

## Rollback

If needed, you can always go back to OpenClaw:
```bash
# Stop reClaw gateway
pkill -f reclaw 2>/dev/null || echo "reClaw not running"

# Restart OpenClaw
openclaw gateway start
```

Your OpenClaw data remains untouched at `~/.openclaw/`.

---

## Migration Checklist

- [ ] Audited OpenClaw setup
- [ ] Verified reClaw implementation exists
- [ ] Converted config (JSON → YAML)
- [ ] Copied memory files
- [ ] Ran tests
- [ ] Started gateway successfully
- [ ] Verified migration works
- [ ] Informed user of next steps
