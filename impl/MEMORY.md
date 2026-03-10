# Memory System

**Purpose:** How to implement memory management - language-agnostic patterns

---

## 1. What is Memory?

Two types:

| Type | Storage | Lifetime |
|------|---------|----------|
| **Short-term** | In-memory (RAM) | Until restart |
| **Long-term** | Disk/database | Forever |

```
┌─────────────────────────────────────────────────┐
│                 Memory System                    │
├─────────────────────────────────────────────────┤
│                                                  │
│  ┌─────────────────┐    ┌──────────────────┐   │
│  │   Short-Term    │◄──►│   Long-Term      │   │
│  │   (In-Memory)   │    │   (Disk/DB)       │   │
│  └─────────────────┘    └──────────────────┘   │
│          │                       │              │
│          ▼                       ▼              │
│    Context Builder       Search Engine         │
│                          (Vector + BM25)        │
│                                                  │
└─────────────────────────────────────────────────┘
```

---

## 2. Interface

```
class MemoryManager:
    
    # Short-term (in-memory)
    load_short_term(session_id: string) -> list[Message]
    save_short_term(session_id: string, messages: list[Message]) -> void
    clear_short_term(session_id?: string) -> void
    
    # Long-term (disk/database)
    load_long_term(session_id: string, options?: dict) -> list[MemoryEntry]
    save_long_term(entry: MemoryEntry) -> void
    delete_long_term(entry_id: string) -> void
    
    # Search
    search(query: string, options?: dict) -> list[SearchResult]
    
    # Maintenance
    prune(before_date: datetime) -> int  # count deleted
    get_stats() -> MemoryStats
```

---

## 3. Short-Term Memory

### 3.1 In-Memory Cache

```
class ShortTermMemory:
    def __init__(self, max_messages=100, ttl_minutes=60):
        self.cache = {}           # session_id -> messages
        self.max_messages = max_messages
        self.ttl_ms = ttl_minutes * 60 * 1000
    
    def load(self, session_id: str) -> list[Message]:
        messages = self.cache.get(session_id, [])
        
        # Filter expired
        now = now_ms()
        valid = [m for m in messages 
                 if now - m.timestamp_ms < self.ttl_ms]
        
        self.cache[session_id] = valid
        return valid
    
    def save(self, session_id: str, messages: list[Message]) -> void:
        # Keep last N messages
        trimmed = messages[-self.max_messages:]
        self.cache[session_id] = trimmed
    
    def add(self, session_id: str, message: Message) -> void:
        messages = self.load(session_id)
        messages.append(message)
        self.save(session_id, messages)
```

### 3.2 Token Counting

```
def count_tokens(messages: list[Message]) -> int:
    total = 0
    for msg in messages:
        total += len(msg.content.split())
    return total
```

---

## 4. Long-Term Memory

### 4.1 File Structure

```
memory/
├── 2026-01-21.md      # Daily entries
├── 2026-01-22.md
├── 2026-01-23.md
├── MEMORY.md           # Important/critical entries
└── index/
    ├── vectors/        # For semantic search
    └── bm25/           # For keyword search
```

### 4.2 Entry Format

```
MemoryEntry {
    id: string
    session_id: string
    type: "conversation" | "learned" | "decision" | "fact"
    title: string
    content: string
    timestamp: datetime
    importance: "low" | "normal" | "high" | "critical"
    tags: list[string]
}
```

### 4.3 Save to File

```
async function save_long_term(entry: MemoryEntry) -> void:
    # Determine file
    filename = "MEMORY.md" if entry.importance in ["high", "critical"] \
               else f"{entry.timestamp.date()}.md"
    
    path = f"memory/{filename}"
    
    # Format as markdown
    content = format_entry_markdown(entry)
    
    # Append to file
    append_to_file(path, content)
    
    # Update search index
    await update_search_index(entry)
```

---

## 5. Search

### 5.1 Hybrid Search

Combine two approaches:
1. **Vector search** - Semantic similarity (find "weather" in "how's the forecast")
2. **BM25** - Keyword matching (find exact words)

```
async function search(query: str, options: dict) -> list[SearchResult]:
    # 1. Vector search
    vector_results = await vector_search(query, options)
    
    # 2. BM25 search
    bm25_results = await bm25_search(query, options)
    
    # 3. Combine with weights
    combined = merge_results(
        vector_results, bm25_results,
        weights=[0.6, 0.4]  # vector, bm25
    )
    
    # 4. Rerank for diversity (optional)
    if options.get("diversity"):
        combined = rerank_for_diversity(combined)
    
    return combined[:options.limit]
```

### 5.2 Vector Search

```
async function vector_search(query: str, limit: int) -> list[Result]:
    # 1. Embed query
    query_embedding = await embed(query)
    
    # 2. Score all entries
    results = []
    for entry in all_memory_entries:
        score = cosine_similarity(query_embedding, entry.embedding)
        results.append({entry, score})
    
    # 3. Sort and return top N
    results.sort(key=lambda r: r.score, reverse=True)
    return results[:limit]
```

### 5.3 BM25

```
# Traditional keyword search
# Use library like Elasticsearch, Meilisearch, or simple implementation

# Pseudocode:
results = []
for entry in all_entries:
    score = bm25_score(query, entry.content)
    results.append({entry, score})

return top_n(results, limit)
```

---

## 6. Context Building

### 6.1 Build Context for LLM

```
async function build_context(session_id: str, current_message: str) -> Context:
    # Get recent messages
    recent = memory.load_short_term(session_id)
    
    # Search long-term for relevant memories
    relevant = await memory.search(current_message, {
        session_id: session_id,
        limit: 5
    })
    
    # Build context string
    context = []
    
    if relevant:
        context.append("Relevant memories:")
        for r in relevant:
            context.append(f"- {r.entry.content}")
    
    if recent:
        context.append("\nRecent conversation:")
        for msg in recent[-5:]:
            context.append(f"{msg.author_name}: {msg.content}")
    
    return Context(
        relevant_memories=relevant,
        recent_messages=recent,
        full_context="\n".join(context)
    )
```

---

## 7. Maintenance

### 7.1 Pruning

```
async function prune(before_date: datetime) -> int:
    count = 0
    
    # Find old files
    for filename in list_memory_files():
        if filename < before_date:
            # Check if entries are important
            entries = parse_file(filename)
            important = [e for e in entries 
                        if e.importance in ["high", "critical"]]
            
            if important:
                # Move important to MEMORY.md
                move_to_memory(important)
            
            # Delete file
            delete_file(filename)
            count += len(entries) - len(important)
    
    return count
```

### 7.2 Stats

```
function get_stats() -> MemoryStats:
    return MemoryStats(
        short_term_entries=sum(len(v) for v in short_term.cache.values()),
        long_term_entries=count_all_entries(),
        memory_size_bytes=get_directory_size("memory/"),
        oldest_entry=find_oldest_entry(),
        newest_entry=find_newest_entry()
    )
```

---

## 8. Language Tips

### Python
```python
# In-memory
dict, list

# File I/O
pathlib, aiofiles

# Vector search
openai (embeddings), numpy
# Or: qdrant, weaviate, milvus (vector DBs)

# Full-text search
whoosh, rank-bm25, elasticsearch
```

### TypeScript
```javascript
// In-memory
Map, Set

// File I/O
fs/promises, fs-extra

// Vector search
openai (embeddings), @pinecone/vanilla-js
// Or: Pinecone, Weaviate (vector DBs)

// Full-text search
flexsearch, minisearch
```

### Go
```runtime
// In-memory
map[string][]Message

// File I/O
os, ioutil

// Vector search
go-openai (embeddings), gomatrix
// Or: qdrant-go, milvus-sdk-go

// Full-text search
bleve, sonic
```

---

## 9. Testing

### Test Short-Term
```python
def test_short_term_memory():
    mem = ShortTermMemory(max_messages=10)
    
    mem.save("session1", [{"content": "Hello"}])
    mem.add("session1", {"content": "World"})
    
    loaded = mem.load("session1")
    assert len(loaded) == 2
```

### Test Long-Term
```python
async def test_save_and_load():
    mem = LongTermMemory("/tmp/test-memory")
    
    entry = MemoryEntry(
        id="test-1",
        content="User prefers Celsius",
        importance="high"
    )
    
    await mem.save_long_term(entry)
    loaded = await mem.load_long_term("test-1")
    
    assert len(loaded) == 1
    assert loaded[0].content == "User prefers Celsius"
```

### Test Search
```python
async def test_search():
    mem = MemoryManager()
    
    # Add entries
    await mem.save_long_term(MemoryEntry(
        content="User likes Python"
    ))
    await mem.save_long_term(MemoryEntry(
        content="User prefers dark mode"
    ))
    
    # Search
    results = await mem.search("programming language")
    
    # Should find "Python" entry via semantic search
    assert len(results) > 0
```

---

## 10. Reference

See also:
- `ARCHITECTURE.md` - System design
- `CONFIG.md` - Memory configuration
- `DATA_FORMATS.md` - Storage formats