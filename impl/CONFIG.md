# Configuration Reference

**Purpose:** Complete YAML configuration schema with all options

---

## 1. Root Configuration

```yaml
# Required - Config version
version: "1.0"

# Main settings
main:
  # Workspace directory
  workspace: ~/.reclaw/workspace
  
  # Log level: debug, info, warn, error
  log_level: info
  
  # Status server (optional)
  status:
    enabled: true
    port: 18789
    host: 127.0.0.1
```

---

## 2. Channels Configuration

### 2.1 Discord

```yaml
channels:
  discord:
    enabled: true
    
    # Bot token (from Discord Developer Portal)
    token: $DISCORD_TOKEN
    
    # Gateway intents (default: all non-privileged)
    # See https://discord.com/developers/docs/topics/gateway#gateway-intents
    intents:
      - GUILDS
      - GUILD_MESSAGES
      - MESSAGE_CONTENT
      - DIRECT_MESSAGES
      - GUILD_VOICE_STATES
    
    # Multi-account support
    accounts:
      default:
        token: $DISCORD_TOKEN
      bot-2:
        token: $DISCORD_TOKEN_2
    
    # DM Policy: pairing | allowlist | open | disabled
    dmPolicy: pairing
    
    # Group policy: open | allowlist
    groupPolicy: open
    
    # Allowed senders (user IDs, phone numbers, etc.)
    allowFrom:
      - "+15555550123"
      - "<@123456789>"
    
    # Message settings
    message:
      # Max characters per message (Discord: 2000)
      chunk_limit: 1900
      chunk_mode: paragraph  # length | paragraph | fence
      # Convert markdown to Discord-specific format
      markdown_mode: default  # default, compact
    
    # Echo prevention
    echo_prevention:
      enabled: true
      track_window: 100
      ignore_bots: true
      ignore_self: true
    
    # Voice channels
    voice:
      enabled: true
      join_timeout: 60
    
    # Thread settings
    threads:
      # Auto-create thread on first message
      auto_create: false
      # Retention time in days
      retention_days: 7
    
    # Components (buttons, selects)
    components:
      # Keep buttons active
      reusable: true
      # Expiry in seconds
      expiry_seconds: 3600
```

### 2.2 Telegram

```yaml
channels:
  telegram:
    enabled: true
    bot_token: $TELEGRAM_BOT_TOKEN
    
    # Connection: webhook | long_polling
    connection:
      mode: webhook
      # URL must be publicly accessible
      url: https://your-domain.com/webhooks/telegram
      # Secret token for verification
      secret_token: $TELEGRAM_SECRET
    
    # Or for long polling
    # connection:
    #   mode: long_polling
    #   timeout: 60
    #   limit: 100
    
    # Multi-account
    accounts:
      default:
        bot_token: $TELEGRAM_BOT_TOKEN
      bot-2:
        bot_token: $TELEGRAM_BOT_TOKEN_2
    
    # DM Policy
    dmPolicy: pairing
    
    # Allowlist
    allowFrom:
      - "+15555550123"
    
    # Message settings
    message:
      chunk_limit: 4096
      parse_mode: HTML  # Markdown, HTML
    
    # Inline keyboards
    inline_keyboard:
      reusable: true
    
    # Bot commands (will be registered with BotFather)
    commands:
      - command: start
        description: Start the bot
      - command: help
        description: Show help
      - command: voice
        description: Toggle voice responses
      - command: voices
        description: List available voices
```

### 2.3 Slack

```yaml
channels:
  slack:
    enabled: true
    
    # Bot token (xoxb-...)
    bot_token: $SLACK_BOT_TOKEN
    
    # App token (xapp-...) for Socket Mode
    app_token: $SLACK_APP_TOKEN
    
    # Signing secret for verification
    signing_secret: $SLACK_SIGNING_SECRET
    
    # Connection: socket | webhook
    connection: socket
    
    # Multi-account
    accounts:
      default:
        bot_token: $SLACK_BOT_TOKEN
    
    # DM Policy
    dmPolicy: allowlist
    allowFrom:
      - "U1234567890"
    
    # Message settings
    message:
      chunk_limit: 3000
      # Use Block Kit
      use_blocks: true
    
    # Slash commands
    commands:
      - command: /reclaw
        description: Interact with reClaw
        url: https://your-domain.com/slack/commands
```

### 2.4 Signal

```yaml
channels:
  signal:
    enabled: true
    
    # signal-cli configuration
    signal_cli_path: /usr/local/bin/signal-cli
    
    # Phone number (from signal-cli register)
    phone_number: +15555550123
    
    # Or use REST API (signal-api)
    api:
      enabled: true
      url: http://localhost:8080
      auth: $SIGNAL_AUTH
```

### 2.5 WhatsApp

```yaml
channels:
  whatsapp:
    enabled: true
    
    # Baileys auth directory
    auth_dir: ~/.reclaw/whatsapp-auth
    
    # Multi-account
    accounts:
      default:
        auth_dir: ~/.reclaw/whatsapp-auth
    
    # Message settings
    message:
      chunk_limit: 4096
      # Convert markdown to WhatsApp format
      markdown_mode: whatsapp
    
    # Media
    media:
      # Auto-download images
      download_images: true
      # Auto-download voice
      download_voice: true
```

### 2.6 iMessage

```yaml
channels:
  imessage:
    enabled: true
    
    # BlueBubbles server
    server:
      url: http://localhost:1234
      api_key: $BLUEBUBBLES_API_KEY
    
    # Or use imsg CLI
    # imsg_path: /usr/local/bin/imsg
```

### 2.7 All Channel Options (Common)

```yaml
# These apply to all channels
channels:
  # Global channel settings
  defaults:
    # Default DM policy if not specified per channel
    dmPolicy: pairing
    
    # Rate limiting
    rate_limit:
      enabled: true
      max_requests: 30
      window_seconds: 60
    
    # Retry settings
    retry:
      max_attempts: 3
      backoff_seconds: 1
      exponential: true
```

---

## 3. Voice Configuration

### 3.1 TTS Providers

```yaml
voice:
  # Default TTS provider
  default_provider: openai
  default_voice: alloy
  
  # TTS providers
  tts:
    openai:
      api_key: $OPENAI_API_KEY
      model: tts-1
      # Response format: mp3, aac, opus, flac
      output_format: mp3
    
    edge_tts:
      enabled: true
    
    elevenlabs:
      api_key: $ELEVENLABS_API_KEY
      # Voice ID (or use default)
      voice_id: 21m00Tcm4TlvDq8ikWAM
```

### 3.2 STT Providers

```yaml
voice:
  # Default STT provider
  default_stt: whisper
  
  # STT providers
  stt:
    whisper:
      api_key: $OPENAI_API_KEY
      # Model: whisper-1
      model: whisper-1
      # Language (auto-detect if not specified)
      language: en
    
    deepgram:
      api_key: $DEEPGRAM_API_KEY
      # Model: nova-2, base, etc.
      model: nova-2
      # Punctuate, paragraphs
      features:
        punctuate: true
        paragraphs: true
```

### 3.3 Phone Providers

```yaml
voice:
  twilio:
    account_sid: $TWILIO_ACCOUNT_SID
    auth_token: $TWILIO_AUTH_TOKEN
    phone_number: +15555551234
    
    # Webhook URL (must be public)
    webhook_url: https://your-domain.com/voice
    
    # Answering machine detection
    machine_detection: true
    
    # Recording
    recording:
      enabled: false
      callback_url: https://your-domain.com/voice/recording
  
  telnyx:
    api_key: $TELNYX_API_KEY
    phone_number: +15555551234
  
  plivo:
    auth_id: $PLIVO_AUTH_ID
    auth_token: $PLIVO_AUTH_TOKEN
    phone_number: +15555551234
```

---

## 4. Agent Configuration

```yaml
agents:
  # Default model for main sessions
  default_model: kimi-coding/k2p5
  
  # Fallback models (tried in order if default fails)
  fallback_models:
    - bailian/qwen3.5-plus
    - bailian/MiniMax-M2.5
  
  # Sub-agent model (can differ from main)
  subagent:
    model: kimi-coding/k2p5
  
  # Concurrency limits
  max_concurrent: 4
  max_spawn_depth: 5
  max_children_per_agent: 3
  
  # Lane limits
  lane_limits:
    main: 2
    subagent: 4
    cron: 1
    session: 2
  
  # Task settings
  task_defaults:
    timeout_seconds: 300
    thinking: medium  # off, low, medium, high
    temperature: 0.7
    max_tokens: 4096
  
  # Tool capabilities
  tool_policy:
    # Auto-allow these tools for sub-agents
    auto_allow:
      - read
      - web_search
      - web_fetch
      - message
    
    # Require approval for these
    require_approval:
      - exec
      - write
      - delete
      - sessions_spawn
  
  # Queue settings
  queue:
    default_mode: collect  # collect, steer, followup, steer+backlog
    debounce_ms: 500
    cap: 50
    drop_policy: old  # old, new, summarize
  
  # Session reset
  session:
    reset:
      mode: daily  # daily, idle, never
      at_hour: 4
      idle_minutes: 60
    
    maintenance:
      mode: enforce  # warn, enforce
      max_entries: 500
      max_disk_mb: 100
      prune_after_days: 30
```

---

## 5. Memory Configuration

```yaml
memory:
  # Short-term (in-memory cache)
  short_term:
    max_messages: 100
    ttl_minutes: 60
  
  # Long-term (disk)
  long_term:
    enabled: true
    path: ./memory
    backend: markdown  # markdown, json, sqlite
  
  # Search
  search:
    provider: hybrid  # vector, bm25, hybrid
    vector_weight: 0.6
    bm25_weight: 0.4
  
  # Embeddings
  embeddings:
    provider: openai  # openai, gemini, voyage, ollama
    openai:
      api_key: $OPENAI_API_KEY
      model: text-embedding-3-small
    ollama:
      url: http://localhost:11434
  
  # Temporal decay (boost recent memories)
  temporal_decay:
    enabled: true
    boost_recent_days: 7
  
  # MMR diversity
  diversity:
    enabled: true
    diversity_factor: 0.5
  
  # Maintenance
  maintenance:
    mode: enforce
    max_entries: 500
    prune_after_days: 30
```

---

## 6. Tools Configuration

### Core Tools (Always Enabled)

```yaml
tools:
  # Core tool settings (tools are always enabled)
  read:
    # Restrict to workspace directory
    path_scope: ./workspace
    # Allowed file extensions
    allowed_extensions:
      - "*"
  
  exec:
    # Allowed commands
    allowed_commands:
      - git
      - npm
      - python3
      - node
    # Timeout
    timeout_seconds: 300
  
  web_fetch:
    # User agent
    user_agent: reClaw/1.0
    # Max chars to extract
    max_chars: 50000
    # Timeout
    timeout_seconds: 30
```

### Extension Tools (Require Configuration)

See extension docs for detailed setup:
- `extensions/web-search.md` - Search providers (Brave, Perplexity, etc.)
- `extensions/browser.md` - Browser automation

```yaml
tools:
  # Enable extension tools
  enabled:
    - web_search  # Requires API key
    - browser     # Requires Playwright/Puppeteer + Chrome
  
  # Web search configuration
  web_search:
    provider: brave
    brave:
      api_key: $BRAVE_API_KEY
      count: 5
  
  # Browser configuration
  browser:
    executable_path: /usr/bin/google-chrome
    default_profile: openclaw
```

---

## 7. Observability

```yaml
observability:
  # Logging
  logging:
    level: info  # debug, info, warn, error
    format: json  # json, text
    output: file  # file, stdout
    
    # Log file settings
    file:
      path: ./logs/reclaw.log
      max_size_mb: 100
      max_files: 10
  
  # Metrics
  metrics:
    enabled: true
    port: 9090
    path: /metrics
  
  # Status server
  status:
    enabled: true
    port: 18789
    host: 127.0.0.1
    
    # API endpoints
    endpoints:
      - /api/v1/status
      - /api/v1/sessions
      - /api/v1/agents
      - /api/v1/memory
      - /api/v1/channels
      - /api/v1/voice
```

---

## 8. Security

```yaml
security:
  # Secrets - NEVER put actual values here, use $VAR
  # Values starting with $ are read from environment variables
  
  # Rate limiting
  rate_limit:
    global:
      enabled: true
      max_requests: 100
      window_seconds: 60
  
  # Input validation
  validation:
    # Max message size
    max_message_size: 100000
    # Sanitize HTML
    sanitize_html: true
  
  # Audit logging
  audit:
    enabled: true
    log_file: ./logs/audit.log
```

---

## 9. Complete Example

```yaml
version: "1.0"

main:
  workspace: ~/.reclaw/workspace
  log_level: info

channels:
  discord:
    enabled: true
    token: $DISCORD_TOKEN
    dmPolicy: pairing
    allowFrom:
      - "+15555550123"
  
  telegram:
    enabled: true
    bot_token: $TELEGRAM_BOT_TOKEN
    dmPolicy: pairing

voice:
  default_provider: openai
  default_voice: alloy
  tts:
    openai:
      api_key: $OPENAI_API_KEY
  stt:
    whisper:
      api_key: $OPENAI_API_KEY
  twilio:
    account_sid: $TWILIO_ACCOUNT_SID
    auth_token: $TWILIO_AUTH_TOKEN
    phone_number: $TWILIO_PHONE_NUMBER

agents:
  default_model: kimi-coding/k2p5
  max_concurrent: 4

memory:
  short_term:
    max_messages: 100
  long_term:
    enabled: true
    path: ./memory

tools:
  enabled:
    - read
    - web_search
    - web_fetch
    - message
    - tts

observability:
  status:
    enabled: true
    port: 18789
```