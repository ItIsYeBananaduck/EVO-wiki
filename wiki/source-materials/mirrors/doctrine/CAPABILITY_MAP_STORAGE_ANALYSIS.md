---
title: CAPABILITY_MAP_STORAGE_ANALYSIS
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CAPABILITY_MAP_STORAGE_ANALYSIS.md"]
updated: 2026-07-24
---

# Capability Map Storage Analysis: JSON vs SQLite vs RAG

## Current Approach: JSON File

### How It Works

- Load entire JSON file (~1782 lines, ~114KB)
- Filter in memory by context (domain, tier, role)
- Format as compact JSON for system prompt
- Only included when `needsCapabilityMap()` returns true

### Performance

- **Load time**: ~5-10ms (file I/O + JSON parsing)
- **Filter time**: ~1-2ms (in-memory filtering)
- **Memory**: ~114KB loaded into memory
- **Token cost**: 200-500 tokens when included (on-demand)

---

## Option 1: SQLite Database

### How It Would Work

```sql
CREATE TABLE capabilities (
    id INTEGER PRIMARY KEY,
    action_name TEXT UNIQUE,
    action_type TEXT,  -- 'agentic', 'automatic', 'vision', 'trainer', 'admin'
    domain TEXT,       -- 'workout', 'planning', 'nutrition', etc.
    requires_pro BOOLEAN,
    agentic_enabled TEXT,  -- 'always allowed', 'must be true'
    payload_json TEXT,
    when_to_use_json TEXT,
    how_to_use_json TEXT,
    compact_json TEXT  -- Pre-computed compact format
);

CREATE INDEX idx_action_type ON capabilities(action_type);
CREATE INDEX idx_domain ON capabilities(domain);
CREATE INDEX idx_access ON capabilities(requires_pro, agentic_enabled);
```

### Query Example

```sql
-- Get relevant capabilities for context
SELECT compact_json FROM capabilities
WHERE action_type IN ('agentic', 'automatic')
  AND (domain = ? OR domain = 'all')
  AND (requires_pro = FALSE OR ? = TRUE)  -- user_is_pro
  AND (agentic_enabled = 'always allowed' OR ? = TRUE)  -- agentic_enabled
ORDER BY action_name;
```

### Pros

✅ **Indexed lookups**: O(log n) instead of O(n) filtering
✅ **Query only what's needed**: Don't load entire file
✅ **Pre-computed compact format**: No formatting overhead
✅ **Efficient filtering**: Database handles complex queries
✅ **Smaller memory footprint**: Only load matching rows
✅ **Faster for repeated queries**: Database caching

### Cons

❌ **Setup overhead**: Need to create schema, migrate data
❌ **Database file size**: ~50-100KB (similar to JSON)
❌ **Query overhead**: SQL parsing, index lookups (~2-5ms)
❌ **Maintenance**: Need to keep DB in sync with JSON source
❌ **Complexity**: More code to maintain

### Performance Estimate

- **Query time**: ~2-5ms (indexed lookup)
- **Memory**: ~10-50KB (only matching rows)
- **Token cost**: Same (200-500 tokens when included)

**Verdict**: **Moderate improvement** - Faster queries, but setup/maintenance cost

---

## Option 2: Vector RAG

### How It Would Work

```typescript
// Embed each capability into vector space
const embeddings = await embedCapabilities(capabilityMap);

// Store in vector database
await vectorDB.upsert({
  vectors: embeddings,
  metadata: capabilities,
});

// Query by semantic similarity
const results = await vectorDB.query({
  vector: userQueryEmbedding,
  topK: 10,
  filter: { domain: "workout", tier: "pro" },
});
```

### Pros

✅ **Semantic search**: Find capabilities by meaning, not exact match
✅ **Flexible queries**: "I want to modify my workout" → finds update_workout_plan
✅ **Natural language**: User queries map to capabilities automatically

### Cons

❌ **Overkill for structured data**: We know exact action names
❌ **Embedding overhead**: Need to embed every capability (~50-100ms)
❌ **Vector DB overhead**: More complex than needed
❌ **Token cost**: Still need to include results in prompt
❌ **Accuracy**: Semantic search might miss exact matches
❌ **Complexity**: Much more code to maintain

### Performance Estimate

- **Embedding time**: ~50-100ms (one-time, but needs updates)
- **Query time**: ~10-20ms (vector similarity search)
- **Memory**: ~500KB-1MB (vector embeddings)
- **Token cost**: Same or more (semantic results might be less precise)

**Verdict**: **Not recommended** - Overkill for structured data, adds complexity without clear benefit

---

## Recommendation: **Hybrid Approach - SQLite with Caching**

### Best of Both Worlds

```typescript
// 1. SQLite for fast indexed queries
const db = new CapabilityDatabase("capabilities.db");

// 2. In-memory cache for frequently accessed capabilities
const cache = new Map<string, Capability[]>();

// 3. Query with caching
function getRelevantCapabilities(context: Context): Capability[] {
  const cacheKey = `${context.domain}-${context.tier}-${context.role}`;

  // Check cache first
  if (cache.has(cacheKey)) {
    return cache.get(cacheKey)!;
  }

  // Query database
  const capabilities = db.query(context);

  // Cache for next time
  cache.set(cacheKey, capabilities);

  return capabilities;
}
```

### Implementation Strategy

**Phase 1: Keep JSON, Add Caching**

- Add in-memory cache for filtered results
- Cache key: `domain-tier-role`
- Cache TTL: 1 hour (or until app restart)
- **Benefit**: Fast repeated queries, minimal code changes

**Phase 2: Migrate to SQLite (Optional)**

- Only if caching isn't enough
- Create schema, migrate JSON to DB
- Keep JSON as source of truth
- **Benefit**: Faster first-time queries, indexed lookups

**Phase 3: Skip RAG**

- Not needed for structured data
- Keep for historical workout data (where it makes sense)

---

## Performance Comparison

| Approach           | Load Time            | Query Time    | Memory        | Complexity | Best For              |
| ------------------ | -------------------- | ------------- | ------------- | ---------- | --------------------- |
| **Current (JSON)** | 5-10ms               | 1-2ms         | 114KB         | Low        | ✅ Current use case   |
| **JSON + Cache**   | 5-10ms (first)       | <1ms (cached) | 114KB + cache | Low        | ✅ **Recommended**    |
| **SQLite**         | 0ms (already loaded) | 2-5ms         | 10-50KB       | Medium     | If cache isn't enough |
| **RAG**            | 50-100ms (embed)     | 10-20ms       | 500KB-1MB     | High       | ❌ Not recommended    |

---

## My Recommendation

### **Start with JSON + In-Memory Caching**

**Why:**

1. **Minimal code changes**: Just add a cache layer
2. **Immediate benefit**: Fast repeated queries
3. **Low complexity**: Easy to maintain
4. **Good enough**: Most queries are cached after first load

**Implementation:**

```kotlin
// Android
private val capabilityCache = mutableMapOf<String, Map<String, Any>>()

fun getRelevantCapabilities(...): Map<String, Any>? {
    val cacheKey = "$domain-$tier-$role"
    return capabilityCache.getOrPut(cacheKey) {
        loadAndFilterCapabilities(...)
    }
}
```

```swift
// iOS
private var capabilityCache: [String: [String: Any]] = [:]

func getRelevantCapabilities(...) -> [String: Any]? {
    let cacheKey = "\(domain)-\(tier)-\(role)"
    if let cached = capabilityCache[cacheKey] {
        return cached
    }
    let loaded = loadAndFilterCapabilities(...)
    capabilityCache[cacheKey] = loaded
    return loaded
}
```

### **Consider SQLite Later If:**

- Cache isn't enough (unlikely)
- Need more complex queries
- Want to query by multiple criteria simultaneously
- File size grows significantly

### **Skip RAG Because:**

- Data is structured, not semantic
- We know exact action names
- Semantic search adds complexity without clear benefit
- Keep RAG for historical workout data where it makes sense

---

## Conclusion

**Current JSON approach is fine** - just add caching for repeated queries.

**SQLite would help** if we need more complex queries, but adds maintenance overhead.

**RAG is overkill** - structured data doesn't need semantic search.

**Best path forward**: Add in-memory caching to current JSON approach. Simple, effective, low risk.

## Related
