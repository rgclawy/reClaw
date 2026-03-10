# Hybrid Search (Memory)

**Extension** - Not in core v0, add when needed.

---

## What It Enables

Better memory search by combining two approaches:
1. **Keyword search** - Find exact words (BM25)
2. **Semantic search** - Find similar meanings (vector embeddings)

This helps find relevant memories even when users phrase things differently.

---

## Implementation Pattern

### Architecture

```
Query → ┌─────────────────┐ → Combined Results
        │  Vector Search  │ (semantic: "weather" matches "forecast")
        │   (embeddings)  │
        └─────────────────┘
               │
        ┌──────┴──────┐
        ▼             ▼
   BM25 Search    Vector Search
   (keyword)     (semantic)
        │             │
        └──────┬──────┘
               ▼
        ┌─────────────────┐
        │  Merge + Rerank │ (weighted combination)
        └─────────────────┘
```

### Basic Implementation

```python
class HybridSearch:
    def __init__(self, vector_weight=0.6, bm25_weight=0.4):
        self.vector_weight = vector_weight
        self.bm25_weight = bm25_weight
        self.bm25 = BM25Index()
        self.vector = VectorIndex()
    
    async def index(self, entry):
        # Index for both searches
        self.bm25.add(entry)
        await self.vector.add(entry)
    
    async def search(self, query, limit=10):
        # Run both searches
        bm25_results = self.bm25.search(query, limit)
        vector_results = await self.vector.search(query, limit)
        
        # Merge with weights
        combined = self.merge_results(
            bm25_results, 
            vector_results,
            weights=[self.bm25_weight, self.vector_weight]
        )
        
        return combined[:limit]
    
    def merge_results(self, bm25, vector, weights):
        scores = {}
        
        for entry, score in bm25:
            scores[entry.id] = score * weights[0]
        
        for entry, score in vector:
            if entry.id in scores:
                scores[entry.id] += score * weights[1]
            else:
                scores[entry.id] = score * weights[1]
        
        return sorted(scores.items(), key=lambda x: x[1], reverse=True)
```

### Vector Embedding

```python
class VectorIndex:
    def __init__(self, provider="openai"):
        self.provider = provider
        self.embeddings = {}
    
    async def embed(self, text):
        if self.provider == "openai":
            response = await openai.embeddings.create(
                model="text-embedding-3-small",
                input=text
            )
            return response.data[0].embedding
        # Other providers...
    
    async def add(self, entry):
        embedding = await self.embed(entry.content)
        self.embeddings[entry.id] = embedding
    
    async def search(self, query, limit):
        query_embedding = await self.embed(query)
        
        results = []
        for entry_id, embedding in self.embeddings.items():
            score = cosine_similarity(query_embedding, embedding)
            results.append((entry_id, score))
        
        return sorted(results, key=lambda x: x[1], reverse=True)[:limit]
```

### BM25 (Keyword)

```python
class BM25Index:
    def __init__(self):
        self.documents = {}
        self.terms = {}
    
    def add(self, entry):
        self.documents[entry.id] = entry.content
        # Build inverted index
        terms = entry.content.lower().split()
        for term in terms:
            if term not in self.terms:
                self.terms[term] = []
            self.terms[term].append(entry.id)
    
    def search(self, query, limit):
        query_terms = query.lower().split()
        scores = {}
        
        for term in query_terms:
            if term in self.terms:
                for doc_id in self.terms[term]:
                    scores[doc_id] = scores.get(doc_id, 0) + 1
        
        return sorted(scores.items(), key=lambda x: x[1], reverse=True)[:limit]
```

---

## Providers

| Provider | Type | API Key |
|----------|------|---------|
| OpenAI | Best quality | Required |
| Gemini | Good | Required |
| Voyage | Good | Required |
| Ollama | Local | Free |

```yaml
memory:
  search:
    provider: hybrid
    vector_weight: 0.6
    bm25_weight: 0.4
  
  embeddings:
    provider: openai
    api_key: $OPENAI_API_KEY
    model: text-embedding-3-small
```

---

## MMR Diversity (Optional)

For more diverse results, use Maximal Marginal Relevance:

```python
def rerank_for_diversity(self, results, k=5, diversity=0.5):
    selected = []
    remaining = results.copy()
    
    while len(selected) < k and remaining:
        # Add highest score
        selected.append(remaining.pop(0))
        
        # Reduce scores of similar items
        for i in range(len(remaining) - 1, -1, -1):
            similarity = compute_similarity(selected[-1], remaining[i])
            remaining[i] = (remaining[i][0], remaining[i][1] * (1 - diversity * similarity))
        
        remaining.sort(key=lambda x: x[1], reverse=True)
    
    return selected
```

---

## See Also

- `impl/MEMORY.md` - Base memory system