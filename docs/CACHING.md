# Caching Guide

Complete guide to caching strategies, cache configuration, and performance optimization in ResonantGenesis.

## Overview

ResonantGenesis implements multi-layer caching to optimize performance and reduce costs. This guide covers cache configuration, strategies, and best practices for maximizing efficiency.

## Cache Layers

| Layer | Purpose | TTL | Scope |
|-------|---------|-----|-------|
| **Response Cache** | Cache agent responses | 1-24h | Per request |
| **Session Cache** | Cache session state | Session duration | Per session |
| **Model Cache** | Cache model outputs | 1-7d | Per model |
| **Tool Cache** | Cache tool results | 5m-1h | Per tool |
| **Memory Cache** | Cache memory lookups | 15m | Per agent |

## Cache Configuration

### Global Configuration

```python
from resonantgenesis import ResonantGenesis

client = ResonantGenesis(
    api_key="your_key",
    cache_config={
        "enabled": True,
        "default_ttl": 3600,  # 1 hour
        "max_size": "100MB",
        "strategy": "lru"
    }
)
```

### Per-Request Configuration

```python
# Disable cache for specific request
result = client.sessions.execute(
    agent_id="agent_123",
    goal="Get current weather",
    cache=False
)

# Custom TTL
result = client.sessions.execute(
    agent_id="agent_123",
    goal="Analyze document",
    cache_ttl=7200  # 2 hours
)
```

### Agent-Level Configuration

```python
# Configure agent caching
agent = client.agents.create(
    name="Research Agent",
    cache_config={
        "enabled": True,
        "response_ttl": 3600,
        "tool_cache": True,
        "memory_cache": True
    }
)
```

## Response Caching

### How It Works

Response caching stores agent outputs based on:
- Agent ID
- Input goal/prompt
- Model configuration
- Tool availability

### Cache Key Generation

```python
# Cache key is generated from:
cache_key = hash(
    agent_id,
    goal,
    model,
    temperature,
    tools_enabled
)
```

### Configure Response Cache

```python
# Enable response caching
agent = client.agents.create(
    name="FAQ Bot",
    cache_config={
        "response_cache": {
            "enabled": True,
            "ttl": 86400,  # 24 hours
            "max_entries": 10000,
            "similarity_threshold": 0.95
        }
    }
)
```

### Semantic Caching

```python
# Enable semantic similarity caching
agent = client.agents.create(
    name="Support Bot",
    cache_config={
        "semantic_cache": {
            "enabled": True,
            "similarity_threshold": 0.92,
            "embedding_model": "text-embedding-3-small"
        }
    }
)

# Similar queries will return cached responses
# "What's the weather?" ≈ "How's the weather today?"
```

## Tool Caching

### Configure Tool Cache

```python
# Configure tool-specific caching
agent = client.agents.create(
    name="Research Agent",
    tools=["web_search", "code_exec"],
    tool_cache_config={
        "web_search": {
            "enabled": True,
            "ttl": 300,  # 5 minutes
            "cache_key_params": ["query"]
        },
        "code_exec": {
            "enabled": False  # Don't cache code execution
        }
    }
)
```

### Tool Cache Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| `exact` | Exact parameter match | Deterministic tools |
| `semantic` | Similar input match | Search queries |
| `none` | No caching | Dynamic tools |

### Custom Tool Cache

```python
from resonantgenesis.tools import Tool, cache

class WeatherTool(Tool):
    @cache(ttl=300, key_params=["location"])
    async def get_weather(self, location: str) -> dict:
        # Fetch weather data
        return await self.fetch_weather(location)
```

## Memory Caching

### Configure Memory Cache

```python
# Configure memory system caching
agent = client.agents.create(
    name="Knowledge Agent",
    memory_config={
        "cache": {
            "enabled": True,
            "ttl": 900,  # 15 minutes
            "max_entries": 1000,
            "preload": True
        }
    }
)
```

### Memory Preloading

```python
# Preload frequently accessed memories
await client.memory.preload(
    agent_id="agent_123",
    queries=[
        "company policies",
        "product information",
        "pricing details"
    ]
)
```

## Session Caching

### Session State Cache

```python
# Configure session caching
session = client.sessions.create(
    agent_id="agent_123",
    cache_config={
        "state_cache": True,
        "context_cache": True,
        "history_limit": 50
    }
)
```

### Resume Cached Session

```python
# Resume session from cache
session = client.sessions.resume(
    session_id="sess_123",
    use_cache=True
)
```

## Cache Strategies

### LRU (Least Recently Used)

```python
# Default strategy - evicts least recently used items
cache_config = {
    "strategy": "lru",
    "max_size": "100MB"
}
```

### LFU (Least Frequently Used)

```python
# Evicts least frequently accessed items
cache_config = {
    "strategy": "lfu",
    "max_size": "100MB"
}
```

### TTL-Based

```python
# Time-based expiration
cache_config = {
    "strategy": "ttl",
    "default_ttl": 3600,
    "max_size": "100MB"
}
```

### Tiered Caching

```python
# Multi-tier cache configuration
cache_config = {
    "strategy": "tiered",
    "tiers": [
        {"name": "hot", "size": "10MB", "ttl": 300},
        {"name": "warm", "size": "50MB", "ttl": 3600},
        {"name": "cold", "size": "200MB", "ttl": 86400}
    ]
}
```

## Cache Invalidation

### Manual Invalidation

```python
# Invalidate specific cache entry
client.cache.invalidate(
    agent_id="agent_123",
    key="specific_query"
)

# Invalidate all agent cache
client.cache.invalidate(
    agent_id="agent_123"
)

# Invalidate by pattern
client.cache.invalidate(
    pattern="weather_*"
)
```

### Automatic Invalidation

```python
# Configure automatic invalidation
agent = client.agents.update(
    agent_id="agent_123",
    cache_config={
        "auto_invalidate": {
            "on_agent_update": True,
            "on_tool_update": True,
            "on_memory_update": True
        }
    }
)
```

### Event-Based Invalidation

```python
# Invalidate on events
@client.events.on("data.updated")
async def invalidate_cache(event):
    await client.cache.invalidate(
        tags=[event.data_type]
    )
```

## Cache Tags

### Tag Cache Entries

```python
# Tag cache entries for grouped invalidation
result = client.sessions.execute(
    agent_id="agent_123",
    goal="Get product info",
    cache_tags=["products", "catalog"]
)

# Invalidate by tag
client.cache.invalidate(tags=["products"])
```

### Hierarchical Tags

```python
# Use hierarchical tags
cache_tags = [
    "products",
    "products:electronics",
    "products:electronics:phones"
]

# Invalidate parent invalidates children
client.cache.invalidate(tags=["products"])
```

## Distributed Caching

### Redis Backend

```python
# Configure Redis cache backend
client = ResonantGenesis(
    api_key="your_key",
    cache_backend={
        "type": "redis",
        "url": "redis://localhost:6379",
        "db": 0,
        "prefix": "rg:"
    }
)
```

### Memcached Backend

```python
# Configure Memcached backend
client = ResonantGenesis(
    api_key="your_key",
    cache_backend={
        "type": "memcached",
        "servers": ["localhost:11211"],
        "prefix": "rg:"
    }
)
```

### Cluster Configuration

```python
# Redis cluster configuration
cache_backend = {
    "type": "redis_cluster",
    "nodes": [
        "redis://node1:6379",
        "redis://node2:6379",
        "redis://node3:6379"
    ],
    "read_from_replicas": True
}
```

## Cache Warming

### Warm Cache on Startup

```python
# Warm cache with common queries
async def warm_cache():
    common_queries = [
        "What are your business hours?",
        "How do I reset my password?",
        "What is your return policy?"
    ]
    
    for query in common_queries:
        await client.sessions.execute(
            agent_id="support_agent",
            goal=query,
            cache_ttl=86400  # 24 hours
        )
```

### Scheduled Cache Warming

```python
# Schedule cache warming
from apscheduler.schedulers.asyncio import AsyncIOScheduler

scheduler = AsyncIOScheduler()
scheduler.add_job(warm_cache, 'cron', hour=0)  # Daily at midnight
scheduler.start()
```

## Cache Monitoring

### Cache Statistics

```python
# Get cache statistics
stats = client.cache.get_stats()

print(f"Hit rate: {stats.hit_rate}%")
print(f"Miss rate: {stats.miss_rate}%")
print(f"Total entries: {stats.entry_count}")
print(f"Memory usage: {stats.memory_usage}")
print(f"Evictions: {stats.eviction_count}")
```

### Cache Metrics

```python
# Get detailed metrics
metrics = client.cache.get_metrics(period="1h")

for metric in metrics:
    print(f"{metric.timestamp}: {metric.hits} hits, {metric.misses} misses")
```

### Cache Alerts

```python
# Configure cache alerts
client.alerts.create(
    name="Low Cache Hit Rate",
    condition={
        "metric": "cache_hit_rate",
        "operator": "less_than",
        "threshold": 50,
        "window": "5m"
    },
    actions=["email", "slack"]
)
```

## Cost Optimization

### Token Savings

```python
# Track token savings from caching
savings = client.cache.get_savings(period="30d")

print(f"Cached responses: {savings.cached_responses}")
print(f"Tokens saved: {savings.tokens_saved}")
print(f"Cost saved: ${savings.cost_saved}")
```

### Optimize Cache Size

```python
# Analyze cache efficiency
analysis = client.cache.analyze()

print(f"Recommended max_size: {analysis.recommended_size}")
print(f"Recommended TTL: {analysis.recommended_ttl}")
print(f"Low-value entries: {analysis.low_value_count}")
```

## Best Practices

### For Performance

1. **Cache deterministic responses** - Same input = same output
2. **Use semantic caching** - For similar queries
3. **Set appropriate TTLs** - Balance freshness and efficiency
4. **Monitor hit rates** - Aim for >70% hit rate
5. **Warm cache proactively** - Pre-populate common queries

### For Consistency

1. **Invalidate on updates** - Keep cache fresh
2. **Use cache tags** - Group related entries
3. **Version cache keys** - Include version in key
4. **Handle stale data** - Graceful degradation
5. **Test cache behavior** - Verify invalidation works

### For Cost

1. **Cache expensive operations** - Model calls, API requests
2. **Right-size cache** - Don't over-provision
3. **Monitor savings** - Track cost reduction
4. **Evict low-value entries** - Remove rarely accessed items
5. **Use tiered caching** - Hot/warm/cold tiers

## Troubleshooting

### Low Hit Rate

```python
# Diagnose low hit rate
diagnosis = client.cache.diagnose()

if diagnosis.issue == "key_variance":
    print("Cache keys vary too much - check key generation")
elif diagnosis.issue == "short_ttl":
    print("TTL too short - increase TTL")
elif diagnosis.issue == "small_cache":
    print("Cache too small - increase max_size")
```

### Cache Inconsistency

```python
# Check for stale entries
stale = client.cache.find_stale(
    agent_id="agent_123",
    max_age=3600
)

for entry in stale:
    print(f"Stale: {entry.key} (age: {entry.age}s)")
```

### Memory Issues

```python
# Check memory usage
memory = client.cache.memory_usage()

print(f"Used: {memory.used}")
print(f"Available: {memory.available}")
print(f"Largest entries: {memory.largest_entries}")
```

## API Reference

### Get Cache Stats

```bash
GET /api/v1/cache/stats
```

### Invalidate Cache

```bash
POST /api/v1/cache/invalidate
```

### Warm Cache

```bash
POST /api/v1/cache/warm
```

### Get Cache Entry

```bash
GET /api/v1/cache/{key}
```

### Delete Cache Entry

```bash
DELETE /api/v1/cache/{key}
```

---

**Need caching help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
