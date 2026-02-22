# Performance Optimization Guide

Complete guide to performance optimization, caching, and scaling strategies for ResonantGenesis.

## Overview

This guide covers:
- Performance best practices
- Caching strategies
- Database optimization
- API optimization
- Frontend performance
- Scaling strategies

## Performance Metrics

### Key Metrics

| Metric | Target | Critical |
|--------|--------|----------|
| API Response Time (p50) | < 100ms | < 500ms |
| API Response Time (p99) | < 500ms | < 2s |
| Agent Execution Time | < 5s | < 30s |
| Database Query Time | < 50ms | < 200ms |
| Cache Hit Rate | > 80% | > 50% |

### Monitoring Performance

```python
# Get performance metrics
metrics = client.metrics.get(
    metrics=[
        "api.latency.p50",
        "api.latency.p99",
        "agent.execution.duration",
        "cache.hit_rate"
    ],
    period="1h"
)

print(f"API p50: {metrics.api_latency_p50}ms")
print(f"API p99: {metrics.api_latency_p99}ms")
print(f"Cache hit rate: {metrics.cache_hit_rate}%")
```

## Caching Strategies

### Response Caching

#### Redis Caching

```python
import redis
import json
from functools import wraps

redis_client = redis.Redis(host='localhost', port=6379, db=0)

def cache_response(ttl_seconds=300):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            cache_key = f"{func.__name__}:{hash(str(args) + str(kwargs))}"
            
            # Check cache
            cached = redis_client.get(cache_key)
            if cached:
                return json.loads(cached)
            
            # Execute and cache
            result = await func(*args, **kwargs)
            redis_client.setex(cache_key, ttl_seconds, json.dumps(result))
            return result
        return wrapper
    return decorator

@cache_response(ttl_seconds=60)
async def get_agent(agent_id: str):
    return await db.agents.get(agent_id)
```

#### HTTP Caching

```python
from fastapi import Response

@app.get("/api/v1/agents/{agent_id}")
async def get_agent(agent_id: str, response: Response):
    agent = await agent_service.get(agent_id)
    
    # Set cache headers
    response.headers["Cache-Control"] = "public, max-age=60"
    response.headers["ETag"] = f'"{agent.version}"'
    
    return agent
```

### Query Caching

```python
from sqlalchemy import event
from sqlalchemy.orm import Query

# Cache query results
@event.listens_for(Query, "before_compile", retval=True)
def cache_query(query):
    cache_key = str(query)
    cached = redis_client.get(cache_key)
    if cached:
        return cached
    return query
```

### Session Caching

```python
class SessionCache:
    def __init__(self, redis_client, ttl=3600):
        self.redis = redis_client
        self.ttl = ttl
    
    async def get_session(self, session_id: str):
        cached = self.redis.get(f"session:{session_id}")
        if cached:
            return Session.parse_raw(cached)
        
        session = await db.sessions.get(session_id)
        self.redis.setex(
            f"session:{session_id}",
            self.ttl,
            session.json()
        )
        return session
    
    async def invalidate(self, session_id: str):
        self.redis.delete(f"session:{session_id}")
```

## Database Optimization

### Query Optimization

#### Use Indexes

```sql
-- Create indexes for common queries
CREATE INDEX idx_agents_owner_id ON agents(owner_id);
CREATE INDEX idx_agents_status ON agents(status);
CREATE INDEX idx_sessions_agent_id ON sessions(agent_id);
CREATE INDEX idx_sessions_created_at ON sessions(created_at DESC);

-- Composite index for filtered queries
CREATE INDEX idx_agents_owner_status ON agents(owner_id, status);
```

#### Optimize Queries

```python
# Bad: N+1 query
agents = await db.agents.list(owner_id=user_id)
for agent in agents:
    sessions = await db.sessions.list(agent_id=agent.id)  # N queries

# Good: Eager loading
agents = await db.agents.list(
    owner_id=user_id,
    include=["sessions"]  # Single query with JOIN
)
```

#### Use Pagination

```python
# Bad: Load all records
all_agents = await db.agents.list()

# Good: Paginate
page = await db.agents.list(
    limit=50,
    offset=0,
    order_by="created_at DESC"
)
```

### Connection Pooling

```python
from sqlalchemy.ext.asyncio import create_async_engine

engine = create_async_engine(
    DATABASE_URL,
    pool_size=20,
    max_overflow=10,
    pool_pre_ping=True,
    pool_recycle=3600
)
```

### Read Replicas

```python
# Configure read replicas
class DatabaseRouter:
    def __init__(self, primary_url, replica_urls):
        self.primary = create_async_engine(primary_url)
        self.replicas = [create_async_engine(url) for url in replica_urls]
        self.replica_index = 0
    
    def get_read_engine(self):
        engine = self.replicas[self.replica_index]
        self.replica_index = (self.replica_index + 1) % len(self.replicas)
        return engine
    
    def get_write_engine(self):
        return self.primary
```

## API Optimization

### Request Optimization

#### Batch Requests

```python
# Bad: Multiple requests
agent1 = await client.agents.get("agent_1")
agent2 = await client.agents.get("agent_2")
agent3 = await client.agents.get("agent_3")

# Good: Single batch request
agents = await client.agents.get_many(["agent_1", "agent_2", "agent_3"])
```

#### Field Selection

```python
# Bad: Get all fields
agent = await client.agents.get(agent_id)

# Good: Select only needed fields
agent = await client.agents.get(
    agent_id,
    fields=["id", "name", "status"]
)
```

#### Compression

```python
from fastapi import FastAPI
from fastapi.middleware.gzip import GZipMiddleware

app = FastAPI()
app.add_middleware(GZipMiddleware, minimum_size=1000)
```

### Response Optimization

#### Streaming Responses

```python
from fastapi.responses import StreamingResponse

@app.get("/api/v1/sessions/{session_id}/stream")
async def stream_session(session_id: str):
    async def generate():
        async for event in session_service.stream(session_id):
            yield f"data: {event.json()}\n\n"
    
    return StreamingResponse(
        generate(),
        media_type="text/event-stream"
    )
```

#### Pagination

```python
@app.get("/api/v1/agents")
async def list_agents(
    limit: int = 50,
    cursor: str = None
):
    agents, next_cursor = await agent_service.list(
        limit=limit,
        cursor=cursor
    )
    
    return {
        "agents": agents,
        "next_cursor": next_cursor,
        "has_more": next_cursor is not None
    }
```

## Frontend Performance

### Code Splitting

```typescript
// Lazy load components
const AgentStudio = lazy(() => import('./pages/AgentStudio'));
const Marketplace = lazy(() => import('./pages/Marketplace'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/studio" element={<AgentStudio />} />
        <Route path="/marketplace" element={<Marketplace />} />
      </Routes>
    </Suspense>
  );
}
```

### Image Optimization

```typescript
// Use next/image for automatic optimization
import Image from 'next/image';

function AgentCard({ agent }) {
  return (
    <Image
      src={agent.avatar}
      alt={agent.name}
      width={100}
      height={100}
      loading="lazy"
      placeholder="blur"
    />
  );
}
```

### Memoization

```typescript
import { memo, useMemo, useCallback } from 'react';

// Memoize component
const AgentCard = memo(({ agent, onClick }) => {
  return (
    <div onClick={() => onClick(agent.id)}>
      {agent.name}
    </div>
  );
});

// Memoize expensive calculations
function AgentList({ agents }) {
  const sortedAgents = useMemo(
    () => agents.sort((a, b) => b.rating - a.rating),
    [agents]
  );
  
  const handleClick = useCallback((id) => {
    navigate(`/agents/${id}`);
  }, [navigate]);
  
  return sortedAgents.map(agent => (
    <AgentCard key={agent.id} agent={agent} onClick={handleClick} />
  ));
}
```

### Virtual Lists

```typescript
import { FixedSizeList } from 'react-window';

function AgentList({ agents }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      <AgentCard agent={agents[index]} />
    </div>
  );
  
  return (
    <FixedSizeList
      height={600}
      itemCount={agents.length}
      itemSize={100}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}
```

## Scaling Strategies

### Horizontal Scaling

#### Load Balancing

```nginx
# nginx.conf
upstream backend {
    least_conn;
    server backend1:8000 weight=3;
    server backend2:8000 weight=2;
    server backend3:8000 weight=1;
    
    keepalive 32;
}

server {
    location /api {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
}
```

#### Kubernetes Autoscaling

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

### Queue-Based Processing

```python
from celery import Celery

celery = Celery('tasks', broker='redis://localhost:6379/0')

@celery.task
def execute_agent_async(agent_id: str, goal: str):
    """Process agent execution in background."""
    result = agent_service.execute(agent_id, goal)
    return result

# Queue the task
task = execute_agent_async.delay(agent_id, goal)
```

### Database Sharding

```python
class ShardRouter:
    def __init__(self, shard_count=4):
        self.shard_count = shard_count
        self.shards = [
            create_async_engine(f"postgresql://shard{i}/resonant")
            for i in range(shard_count)
        ]
    
    def get_shard(self, user_id: str):
        shard_index = hash(user_id) % self.shard_count
        return self.shards[shard_index]
```

## Agent Execution Optimization

### Parallel Tool Execution

```python
import asyncio

async def execute_tools_parallel(tools: list[ToolCall]):
    """Execute independent tools in parallel."""
    tasks = [execute_tool(tool) for tool in tools]
    results = await asyncio.gather(*tasks)
    return results
```

### Context Optimization

```python
def optimize_context(messages: list, max_tokens: int):
    """Trim context to fit token limit."""
    total_tokens = sum(count_tokens(m) for m in messages)
    
    if total_tokens <= max_tokens:
        return messages
    
    # Keep system message and recent messages
    system = messages[0]
    recent = messages[-5:]
    
    # Summarize middle messages
    middle = messages[1:-5]
    summary = summarize_messages(middle)
    
    return [system, summary] + recent
```

### Model Selection

```python
def select_model(task_complexity: str):
    """Select appropriate model based on task."""
    models = {
        "simple": "gpt-3.5-turbo",
        "moderate": "gpt-4-turbo",
        "complex": "gpt-4"
    }
    return models.get(task_complexity, "gpt-4-turbo")
```

## Monitoring Performance

### Application Performance Monitoring

```python
from opentelemetry import trace
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor

# Instrument FastAPI
FastAPIInstrumentor.instrument_app(app)

# Custom spans
tracer = trace.get_tracer(__name__)

async def execute_agent(agent_id: str, goal: str):
    with tracer.start_as_current_span("execute_agent") as span:
        span.set_attribute("agent_id", agent_id)
        span.set_attribute("goal_length", len(goal))
        
        result = await agent_service.execute(agent_id, goal)
        
        span.set_attribute("steps", len(result.steps))
        span.set_attribute("tokens_used", result.tokens_used)
        
        return result
```

### Performance Alerts

```python
# Alert on slow responses
client.alerts.create(
    name="Slow API Response",
    type="threshold",
    metric="api.latency.p99",
    condition={
        "operator": "gt",
        "value": 1000,  # 1 second
        "period": "5m"
    },
    severity="warning"
)
```

## Best Practices

### General

1. **Measure first** - Profile before optimizing
2. **Cache aggressively** - But invalidate correctly
3. **Use async** - Non-blocking I/O everywhere
4. **Batch operations** - Reduce round trips
5. **Set timeouts** - Prevent hanging requests

### Database

1. **Index strategically** - Based on query patterns
2. **Use connection pooling** - Reuse connections
3. **Optimize queries** - Avoid N+1 problems
4. **Partition large tables** - Improve query performance

### API

1. **Paginate results** - Never return unbounded lists
2. **Compress responses** - Reduce bandwidth
3. **Use ETags** - Enable conditional requests
4. **Stream large responses** - Reduce memory usage

### Frontend

1. **Code split** - Load only what's needed
2. **Lazy load images** - Defer off-screen images
3. **Memoize components** - Prevent unnecessary renders
4. **Use virtual lists** - Handle large datasets

---

**Need help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
