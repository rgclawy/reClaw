# Web Search (Extension)

**Status:** Extension - Requires API key  
**Tool:** `web_search`

---

## What It Does

Search the web using various providers. Returns search results (title, URL, snippet) that the agent can use for research.

**Note:** This is different from `web_fetch` (core), which just fetches a specific URL.

---

## Providers

| Provider | Result Type | API Key Required |
|----------|-------------|------------------|
| **Brave** (default) | Structured results (title, URL, snippet) | `BRAVE_API_KEY` |
| **Perplexity** | AI-synthesized answers + citations | `PERPLEXITY_API_KEY` |
| **Gemini** | AI answers with Google Search grounding | `GEMINI_API_KEY` |
| **Grok** | AI answers with xAI web grounding | `XAI_API_KEY` |
| **Kimi** | AI answers with Moonshot web search | `KIMI_API_KEY` |

---

## Configuration

```yaml
tools:
  enabled:
    - web_fetch  # Core - always available
    - web_search  # This extension
  
  web_search:
    provider: brave  # brave | perplexity | gemini | grok | kimi
    
    # Brave-specific
    brave:
      api_key: $BRAVE_API_KEY
      count: 5  # Results per query (max 10)
      mode: standard  # standard | llm-context
    
    # Perplexity-specific
    perplexity:
      api_key: $PERPLEXITY_API_KEY
      model: perplexity/sonar-pro
    
    # Gemini-specific
    gemini:
      api_key: $GEMINI_API_KEY
    
    # Grok-specific
    grok:
      api_key: $XAI_API_KEY
      model: grok-4-1-fast
    
    # Kimi-specific
    kimi:
      api_key: $KIMI_API_KEY
      model: moonshot-v1-128k
    
    # Caching
    cache_ttl_minutes: 15
```

---

## Auto-Detection

If no provider is set, reClaw checks for API keys in this order:
1. Brave (`BRAVE_API_KEY`)
2. Gemini (`GEMINI_API_KEY`)
3. Grok (`XAI_API_KEY`)
4. Kimi (`KIMI_API_KEY` / `MOONSHOT_API_KEY`)
5. Perplexity (`PERPLEXITY_API_KEY` / `OPENROUTER_API_KEY`)

---

## Usage

Agent calls `web_search` with a query:

```yaml
# Tool call
{
  "tool": "web_search",
  "params": {
    "query": "latest Python 3.13 features",
    "count": 5
  }
}

# Result
[
  {
    "title": "What's New in Python 3.13",
    "url": "https://docs.python.org/3/whatsnew/3.13.html",
    "snippet": "Python 3.13 includes a new interactive interpreter..."
  },
  ...
]
```

---

## Provider Details

### Brave Search

- **Cost:** $5/month free tier (1,000 queries)
- **Signup:** https://brave.com/search/api/
- **Features:** 
  - Standard snippets
  - `llm-context` mode (extracted page chunks for grounding)
  - Country/language filters

### Perplexity

- **Cost:** Paid API
- **Signup:** https://perplexity.ai/settings/api
- **Features:** AI-synthesized answers with inline citations

### Gemini

- **Cost:** Paid API
- **Features:** Google Search grounding, real-time results

### Grok

- **Cost:** Paid API
- **Features:** xAI web-grounded responses

### Kimi

- **Cost:** Paid API
- **Features:** Moonshot web search integration

---

## Without This Extension

If `web_search` is not enabled, agents can still:
- Use `web_fetch` (core) to fetch specific URLs
- Ask users to provide URLs

---

## See Also

- `impl/CONFIG.md` - Core tools config
- `extensions/browser.md` - For JS-heavy sites
