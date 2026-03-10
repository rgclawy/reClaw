# reClaw Functional Equivalence Test Plan

This document outlines testing strategy to verify that a reclaw implementation is functionally equivalent to OpenClaw.

---

## 1. Overview

### 1.1 Goal
Verify that a reimplemented reclaw system produces identical or equivalent behavior to OpenClaw for all supported features.

### 1.2 Test Philosophy
- **Black-box testing** - Test behavior, not implementation
- **Protocol-driven** - Verify spec compliance
- **Migration-focused** - Ensure config compatibility

---

## 2. Feature Coverage Matrix

### 2.1 Channel Tests

| Feature | Reclaw Spec | Test Case | OpenClaw Behavior |
|---------|-------------|-----------|------------------|
| Discord message receive | Section 6 | Send message to Discord bot | Verify message received and normalized |
| Discord message send | Section 6 | Bot responds to user | Verify response sent |
| Telegram bot commands | Section 6 | Send /start | Verify command handling |
| DM policy pairing | Section 6.3 | Unknown user sends DM | Verify pairing flow triggered |
| DM policy allowlist | Section 6.3 | Non-allowed user sends DM | Verify rejection |
| Multi-account Discord | Section 6 | Send via 2nd account | Verify correct account used |
| Echo prevention | Section 6 | Bot receives own message | Verify not echoed |
| Message chunking | Section 6 | Send long message | Verify chunked correctly |

### 2.2 Voice Tests

| Feature | Reclaw Spec | Test Case | OpenClaw Behavior |
|---------|-------------|-----------|------------------|
| Edge TTS | Section 7.3 | Generate speech | Verify audio output |
| OpenAI TTS | Section 7.3 | Generate speech | Verify audio output |
| Azure STT | Section 7.4 | Transcribe audio | Verify text output |
| Whisper STT | Section 7.4 | Transcribe audio | Verify text output |
| Twilio incoming call | Section 7.1 | Call Twilio number | Verify call answered |
| Twilio outgoing call | Section 7.1 | API triggers call | Verify call initiated |
| Voice message (Telegram) | Section 7.2 | Send voice message | Verify transcribed |

### 2.3 Agent Tests

| Feature | Reclaw Spec | Test Case | OpenClaw Behavior |
|---------|-------------|-----------|------------------|
| Sub-agent spawn | Section 8.2 | Spawn task | Verify sub-agent runs |
| Agent result return | Section 8.2 | Task completes | Verify result returned |
| Session isolation | Section 8.1 | Multiple sessions | Verify isolation |
| Queue modes | Section 8.3 | Test each mode | Verify behavior |

### 2.4 Memory Tests

| Feature | Reclaw Spec | Test Case | OpenClaw Behavior |
|---------|-------------|-----------|------------------|
| Short-term memory | Section 9.1 | Send messages | Verify context maintained |
| Long-term memory | Section 9.2 | Restart, check history | Verify persistence |
| MEMORY.md load | Section 9.2 | Check memory file | Verify content loaded |

### 2.5 Config Tests

| Feature | Reclaw Spec | Test Case | OpenClaw Behavior |
|---------|-------------|-----------|------------------|
| YAML parsing | Section 5 | Load config | Verify parsed correctly |
| Environment indirection | Section 5.3 | $VAR in config | Verify env vars resolved |
| Dynamic reload | Section 5.4 | Modify config | Verify reload without restart |

---

## 3. Protocol Compliance Tests

### 3.1 ACP (Agent Client Protocol)

```python
def test_acp_initialize():
    """Test ACP initialize handshake"""
    # Send initialize request
    # Verify response matches spec
    
def test_acp_thread_start():
    """Test ACP thread start"""
    # Send thread/start
    # Verify thread ID returned
    
def test_acp_turn_start():
    """Test ACP turn execution"""
    # Send turn/start with prompt
    # Verify response streamed
```

### 3.2 Channel Message Normalization

```python
def test_discord_message_normalized():
    """Test Discord message → standard format"""
    message = discord_adapter.receive(raw_message)
    assert message.id is not None
    assert message.channel_type == "discord"
    assert message.author_id is not None
    
def test_telegram_message_normalized():
    """Test Telegram message → standard format"""
    message = telegram_adapter.receive(raw_message)
    assert message.id is not None
    assert message.channel_type == "telegram"
```

### 3.3 Configuration Schema

```python
def test_config_yaml_valid():
    """Test config.yaml validates against schema"""
    config = load_config("config.yaml")
    assert config.version is not None
    assert config.channels is not None
    assert config.agents is not None
```

---

## 4. Migration Tests

### 4.1 Config Conversion

```python
def test_openclaw_to_reclaw_config():
    """Test converting OpenClaw JSON → reclaw YAML"""
    openclaw_config = json.load(open("openclaw.json"))
    reclaw_config = convert_to_yaml(openclaw_config)
    
    # Verify all channels preserved
    # Verify all agents preserved
    # Verify environment variables
```

### 4.2 Behavior Equivalence

```python
def test_identical_input_identical_output():
    """Test same input → same output"""
    input_message = "Hello via Discord"
    
    openclaw_result = openclaw.process(input_message)
    reclaw_result = reclaw.process(input_message)
    
    assert openclaw_result.text == reclaw_result.text
```

---

## 5. Integration Tests

### 5.1 Real Channel Tests

```python
class TestDiscordIntegration:
    def test_message_roundtrip(self):
        """Send message through Discord, verify roundtrip"""
        # 1. Send test message to bot
        # 2. Verify bot receives it
        # 3. Verify bot responds
        # 4. Verify response matches expected
        
    def test_voice_message(self):
        """Test Discord voice message"""
        # 1. Send voice message
        # 2. Verify transcribed
        # 3. Verify response sent

class TestTelegramIntegration:
    def test_bot_commands(self):
        """Test Telegram bot commands"""
        # Test /start, /help, etc.
        
    def test_inline_buttons(self):
        """Test Telegram inline buttons"""
        # Verify button handling
```

### 5.2 Voice Integration Tests

```python
class TestTwilioIntegration:
    def test_incoming_call(self):
        """Test incoming Twilio call"""
        # 1. Call Twilio number
        # 2. Verify webhook received
        # 3. Verify audio streams
        # 4. Verify TTS response
        
    def test_outgoing_call(self):
        """Test outbound Twilio call"""
        # 1. Trigger call via API
        # 2. Verify call initiated
        # 3. Verify connection
```

---

## 6. Test Execution

### 6.1 Prerequisites

```bash
# Install reclaw implementation
npm install reclaw

# Start OpenClaw (for comparison)
openclaw gateway start

# Set environment variables
export DISCORD_TOKEN=...
export TELEGRAM_BOT_TOKEN=...
```

### 6.2 Running Tests

```bash
# Run all tests
npm test

# Run specific category
npm test -- --group=channels
npm test -- --group=voice
npm test -- --group=agents
npm test -- --group=memory

# Run protocol tests only
npm test -- --group=protocol

# Run integration tests only
npm test -- --group=integration
```

### 6.3 Expected Results

| Test Category | Pass Criteria |
|---------------|---------------|
| Protocol | 100% compliance |
| Config Migration | All configs convertible |
| Channel Integration | Real messages work |
| Voice Integration | Real calls work |
| Memory | Persistence verified |
| A/B Equivalence | Within tolerance |

---

## 7. Acceptance Criteria

### 7.1 Must Pass (Blocker)

- [ ] All channel adapters send/receive messages correctly
- [ ] Config migration preserves all settings
- [ ] Sub-agent spawning works
- [ ] Memory persists across restarts

### 7.2 Should Pass (Major)

- [ ] DM policies function correctly
- [ ] Echo prevention works
- [ ] Voice TTS/STT functional
- [ ] Queue modes work as specified

### 7.3 Nice to Have (Minor)

- [ ] All 20+ channels implemented
- [ ] Hybrid search in memory
- [ ] Advanced ACP features

---

## 8. Test Artifacts

### 8.1 Test Data

```
tests/
├── fixtures/
│   ├── config/
│   │   ├── openclaw.json
│   │   └── reclaw.yaml
│   ├── messages/
│   │   ├── discord_long.txt
│   │   └── telegram_markdown.json
│   └── audio/
│       ├── sample.wav
│       └── sample.mp3
```

### 8.2 Test Reports

```
reports/
├── coverage/
│   ├── channels.json
│   ├── voice.json
│   ├── agents.json
│   └── memory.json
├── results/
│   └── test-results-YYYY-MM-DD.xml
└── equivalence/
    └── ab-comparison.json
```

---

## 9. Debugging Failed Tests

### 9.1 Common Issues

| Issue | Diagnosis | Fix |
|-------|-----------|-----|
| Message not received | Check webhook URL | Verify adapter registration |
| Config not loading | Check YAML syntax | Validate schema |
| Memory lost | Check file permissions | Verify storage path |
| Voice not working | Check API keys | Verify provider config |

### 9.2 Logging

Enable debug logging:

```bash
export LOG_LEVEL=debug
npm test 2>&1 | tee test-debug.log
```

---

## 10. Maintenance

- Review test coverage quarterly
- Add new tests for new features
- Update expected behavior for breaking changes
- Archive obsolete tests

---

*Document Version: 1.0*  
*Last Updated: 2026-03-09*