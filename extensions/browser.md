# Browser Automation (Extension)

**Status:** Extension - Requires Chrome/Playwright/Puppeteer  
**Tool:** `browser`

---

## What It Does

Run a managed browser instance that the agent can control for:
- JavaScript-heavy sites (SPAs)
- Sites requiring login/interaction
- Screenshots and PDFs
- Form filling and clicking

**Note:** This is different from `web_fetch` (core), which only does simple HTTP GET without JavaScript.

---

## Requirements

| Dependency | Purpose |
|------------|---------|
| Chrome/Brave/Edge/Chromium | Browser binary |
| Playwright or Puppeteer | Browser automation library |

---

## Configuration

```yaml
tools:
  enabled:
    - web_fetch  # Core
    - browser    # This extension
  
  browser:
    enabled: true
    
    # Browser binary (auto-detected if not set)
    executable_path: /Applications/Google Chrome.app/Contents/MacOS/Google Chrome
    
    # Default profile
    default_profile: openclaw  # openclaw | chrome
    
    # Profile settings
    profiles:
      openclaw:
        cdp_port: 18800
        headless: false
        color: "#FF4500"
        isolated: true  # Separate profile
      
      work:
        cdp_port: 18801
        headless: true
        color: "#0066CC"
    
    # Security
    ssrf_policy:
      dangerously_allow_private_network: false
      hostname_allowlist:
        - "*.example.com"
        - "localhost"
    
    # Timeouts
    cdp_timeout_ms: 1500
    handshake_timeout_ms: 3000
```

---

## Browser Actions

| Action | Description |
|--------|-------------|
| `start` | Launch browser instance |
| `stop` | Close browser |
| `open` | Open URL in new tab |
| `navigate` | Navigate current tab |
| `snapshot` | Get page content (HTML/text) |
| `screenshot` | Capture screenshot |
| `click` | Click element |
| `type` | Type text into input |
| `fill` | Fill form field |
| `select` | Select dropdown option |
| `scroll` | Scroll page |
| `wait` | Wait for element/timeout |
| `close_tab` | Close current tab |

---

## Usage

```yaml
# Start browser
{"tool": "browser", "action": "start", "profile": "openclaw"}

# Open URL
{"tool": "browser", "action": "open", "url": "https://example.com/login"}

# Fill form
{"tool": "browser", "action": "fill", "selector": "#username", "value": "user"}
{"tool": "browser", "action": "fill", "selector": "#password", "value": "pass"}
{"tool": "browser", "action": "click", "selector": "#login-btn"}

# Get content
{"tool": "browser", "action": "snapshot"}
# Result: "Welcome to the dashboard..."

# Screenshot
{"tool": "browser", "action": "screenshot", "path": "./screenshot.png"}

# Stop browser
{"tool": "browser", "action": "stop"}
```

---

## Profiles

### `openclaw` (Managed)
- Isolated browser profile
- Agent-only, doesn't touch your personal browser
- Orange accent color by default
- Recommended for automation

### `chrome` (Extension Relay)
- Uses your system browser
- Requires OpenClaw browser extension
- For sites where you need your logged-in sessions

---

## Security

- **SSRF Protection:** Blocks private network access by default
- **Isolation:** `openclaw` profile is separate from personal browser
- **Hostname Allowlist:** Restrict which sites browser can access

---

## Comparison: web_fetch vs browser

| Feature | `web_fetch` (Core) | `browser` (Extension) |
|---------|-------------------|----------------------|
| JavaScript | ❌ No | ✅ Yes |
| Login/Auth | ❌ No | ✅ Yes |
| Screenshots | ❌ No | ✅ Yes |
| Click/Type | ❌ No | ✅ Yes |
| Speed | ⚡ Fast | 🐢 Slower |
| Dependencies | None | Chrome + Playwright |
| Memory | Low | High |

---

## Without This Extension

If `browser` is not enabled:
- Use `web_fetch` (core) for simple static sites
- Use `web_search` (extension) to find information
- User must manually log in and provide URLs

---

## See Also

- `extensions/web-search.md` - Search before browsing
- `impl/CONFIG.md` - Core tools configuration
