# Rate Limits Documentation

Complete guide to API rate limits and best practices for ResonantGenesis.

## Overview

Rate limits protect the platform from abuse and ensure fair usage for all users. Limits vary by subscription tier and endpoint type.

## Rate Limits by Tier

| Tier | Requests/Min | Requests/Hour | Concurrent Sessions | Daily Limit |
|------|--------------|---------------|---------------------|-------------|
| Free | 60 | 1,000 | 1 | 10,000 |
| Plus | 300 | 10,000 | 5 | 100,000 |
| Pro | 600 | 30,000 | 20 | 500,000 |
| Enterprise | 1,000+ | Custom | Custom | Custom |

## Endpoint-Specific Limits

### Agent Endpoints

| Endpoint | Free | Plus | Pro |
|----------|------|------|-----|
| `POST /agents` | 10/hour | 50/hour | 200/hour |
| `GET /agents` | 60/min | 300/min | 600/min |
| `POST /agents/{id}/execute` | 20/hour | 100/hour | 500/hour |
| `DELETE /agents/{id}` | 10/hour | 50/hour | 200/hour |

### Session Endpoints

| Endpoint | Free | Plus | Pro |
|----------|------|------|-----|
| `POST /sessions` | 20/hour | 100/hour | 500/hour |
| `GET /sessions/{id}` | 120/min | 600/min | 1200/min |
| `GET /sessions/{id}/stream` | 5 concurrent | 20 concurrent | 100 concurrent |

### Team Endpoints

| Endpoint | Free | Plus | Pro |
|----------|------|------|-----|
| `POST /teams` | 5/hour | 20/hour | 100/hour |
| `POST /teams/{id}/execute` | 10/hour | 50/hour | 200/hour |

### Webhook Endpoints

| Endpoint | Free | Plus | Pro |
|----------|------|------|-----|
| Incoming webhooks | 100/hour | 1,000/hour | 10,000/hour |
| Webhook triggers | 10/agent | 50/agent | 200/agent |

## Rate Limit Headers

Every API response includes rate limit information:

```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1708531200
X-RateLimit-Reset-After: 30
```

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Maximum requests allowed in window |
| `X-RateLimit-Remaining` | Requests remaining in current window |
| `X-RateLimit-Reset` | Unix timestamp when limit resets |
| `X-RateLimit-Reset-After` | Seconds until limit resets |

## Rate Limit Exceeded Response

When you exceed the rate limit:

```http
HTTP/1.1 429 Too Many Requests
Content-Type: application/json
Retry-After: 30

{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded. Please retry after 30 seconds.",
    "retry_after": 30,
    "limit": 60,
    "reset_at": "2026-02-21T10:05:00Z"
  }
}
```

## Handling Rate Limits

### Python - Basic Retry

```python
import time
from resonantgenesis import ResonantClient, RateLimitError

client = ResonantClient(api_key="your_key")

def make_request_with_retry(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except RateLimitError as e:
            if attempt == max_retries - 1:
                raise
            wait_time = e.retry_after or (2 ** attempt)
            print(f"Rate limited. Waiting {wait_time}s...")
            time.sleep(wait_time)

# Usage
result = make_request_with_retry(
    lambda: client.agents.list()
)
```

### Python - Exponential Backoff

```python
import time
import random

def exponential_backoff(func, max_retries=5, base_delay=1, max_delay=60):
    for attempt in range(max_retries):
        try:
            return func()
        except RateLimitError as e:
            if attempt == max_retries - 1:
                raise
            
            # Use server's retry_after if available
            delay = e.retry_after or min(
                base_delay * (2 ** attempt) + random.uniform(0, 1),
                max_delay
            )
            print(f"Attempt {attempt + 1} failed. Retrying in {delay:.2f}s...")
            time.sleep(delay)

# Usage
result = exponential_backoff(
    lambda: client.agents.execute(agent_id, goal="Task")
)
```

### Python - Rate Limiter Class

```python
import time
import threading
from collections import deque

class RateLimiter:
    def __init__(self, max_requests: int, window_seconds: int):
        self.max_requests = max_requests
        self.window_seconds = window_seconds
        self.requests = deque()
        self.lock = threading.Lock()
    
    def acquire(self):
        with self.lock:
            now = time.time()
            
            # Remove old requests outside window
            while self.requests and self.requests[0] < now - self.window_seconds:
                self.requests.popleft()
            
            # Check if we can make a request
            if len(self.requests) >= self.max_requests:
                wait_time = self.requests[0] + self.window_seconds - now
                return False, wait_time
            
            self.requests.append(now)
            return True, 0
    
    def wait_and_acquire(self):
        while True:
            can_proceed, wait_time = self.acquire()
            if can_proceed:
                return
            time.sleep(wait_time)

# Usage
limiter = RateLimiter(max_requests=60, window_seconds=60)

def rate_limited_request(func):
    limiter.wait_and_acquire()
    return func()

result = rate_limited_request(
    lambda: client.agents.list()
)
```

### JavaScript - Retry with Backoff

```typescript
import { ResonantClient, RateLimitError } from '@resonantgenesis/sdk';

async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (error instanceof RateLimitError && attempt < maxRetries - 1) {
        const delay = error.retryAfter || Math.pow(2, attempt) * 1000;
        console.log(`Rate limited. Waiting ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const result = await withRetry(() => client.agents.list());
```

### JavaScript - Token Bucket

```typescript
class TokenBucket {
  private tokens: number;
  private lastRefill: number;
  
  constructor(
    private maxTokens: number,
    private refillRate: number // tokens per second
  ) {
    this.tokens = maxTokens;
    this.lastRefill = Date.now();
  }
  
  async acquire(): Promise<void> {
    this.refill();
    
    if (this.tokens < 1) {
      const waitTime = (1 - this.tokens) / this.refillRate * 1000;
      await new Promise(resolve => setTimeout(resolve, waitTime));
      this.refill();
    }
    
    this.tokens -= 1;
  }
  
  private refill(): void {
    const now = Date.now();
    const elapsed = (now - this.lastRefill) / 1000;
    this.tokens = Math.min(this.maxTokens, this.tokens + elapsed * this.refillRate);
    this.lastRefill = now;
  }
}

// Usage: 60 requests per minute = 1 per second
const bucket = new TokenBucket(60, 1);

async function rateLimitedRequest<T>(fn: () => Promise<T>): Promise<T> {
  await bucket.acquire();
  return fn();
}
```

## Best Practices

### 1. Monitor Your Usage

```python
# Check current rate limit status
response = client.rate_limits.get()

print(f"Limit: {response.limit}")
print(f"Remaining: {response.remaining}")
print(f"Resets at: {response.reset_at}")

# Set up alerts
if response.remaining < response.limit * 0.1:
    print("Warning: Less than 10% of rate limit remaining!")
```

### 2. Batch Requests

```python
# Instead of multiple individual requests
for id in agent_ids:
    agent = client.agents.get(id)  # 100 requests

# Use batch endpoint
agents = client.agents.get_many(agent_ids)  # 1 request
```

### 3. Use Caching

```python
from functools import lru_cache
import time

class CachedClient:
    def __init__(self, client, cache_ttl=60):
        self.client = client
        self.cache_ttl = cache_ttl
        self._cache = {}
    
    def get_agent(self, agent_id):
        cache_key = f"agent:{agent_id}"
        cached = self._cache.get(cache_key)
        
        if cached and time.time() - cached['time'] < self.cache_ttl:
            return cached['data']
        
        data = self.client.agents.get(agent_id)
        self._cache[cache_key] = {'data': data, 'time': time.time()}
        return data

# Usage
cached = CachedClient(client, cache_ttl=300)
agent = cached.get_agent("agent-123")  # Cached for 5 minutes
```

### 4. Use Webhooks Instead of Polling

```python
# Instead of polling for status
while True:
    status = client.sessions.get(session_id).status
    if status == "completed":
        break
    time.sleep(1)  # Many requests

# Use webhooks
trigger = agent.triggers.create(
    type="session_complete",
    webhook_url="https://your-server.com/webhook"
)
```

### 5. Implement Circuit Breaker

```python
import time

class CircuitBreaker:
    def __init__(self, failure_threshold=5, reset_timeout=60):
        self.failure_threshold = failure_threshold
        self.reset_timeout = reset_timeout
        self.failures = 0
        self.last_failure = None
        self.state = "closed"  # closed, open, half-open
    
    def call(self, func):
        if self.state == "open":
            if time.time() - self.last_failure > self.reset_timeout:
                self.state = "half-open"
            else:
                raise Exception("Circuit breaker is open")
        
        try:
            result = func()
            if self.state == "half-open":
                self.state = "closed"
                self.failures = 0
            return result
        except RateLimitError:
            self.failures += 1
            self.last_failure = time.time()
            if self.failures >= self.failure_threshold:
                self.state = "open"
            raise

# Usage
breaker = CircuitBreaker()

try:
    result = breaker.call(lambda: client.agents.list())
except Exception as e:
    print(f"Request failed: {e}")
```

### 6. Distribute Load

```python
import asyncio
import random

async def distributed_requests(tasks, max_concurrent=10):
    semaphore = asyncio.Semaphore(max_concurrent)
    
    async def limited_task(task):
        async with semaphore:
            # Add jitter to avoid thundering herd
            await asyncio.sleep(random.uniform(0, 0.1))
            return await task
    
    return await asyncio.gather(*[limited_task(t) for t in tasks])

# Usage
tasks = [client.agents.get_async(id) for id in agent_ids]
results = await distributed_requests(tasks, max_concurrent=5)
```

## Upgrading Your Limits

### Check Current Tier

```python
account = client.account.get()
print(f"Current tier: {account.tier}")
print(f"Rate limit: {account.rate_limit}")
```

### Upgrade Options

| Tier | Price | Best For |
|------|-------|----------|
| Plus | $29/mo | Small teams, moderate usage |
| Pro | $99/mo | Growing teams, high usage |
| Enterprise | Custom | Large organizations |

### Request Limit Increase

For temporary increases (e.g., data migration):

```python
# Request temporary limit increase
request = client.support.request_limit_increase(
    reason="Data migration",
    requested_limit=1000,
    duration_hours=24
)

print(f"Request ID: {request.id}")
print(f"Status: {request.status}")
```

## Monitoring & Alerts

### Set Up Usage Alerts

```python
# Configure alerts
client.alerts.create(
    type="rate_limit_warning",
    threshold=0.8,  # Alert at 80% usage
    channels=["email", "slack"]
)
```

### Dashboard Metrics

View real-time usage in Control Center:
- Current usage vs limits
- Historical usage patterns
- Peak usage times
- Endpoint breakdown

## Troubleshooting

### Unexpected Rate Limiting

1. **Check all API keys**: Multiple keys share account limits
2. **Review background jobs**: Automated tasks may consume quota
3. **Check for retry loops**: Broken retry logic can exhaust limits
4. **Monitor concurrent sessions**: Streaming sessions count against limits

### Optimizing High-Volume Applications

1. **Use connection pooling**: Reduce connection overhead
2. **Implement request queuing**: Smooth out traffic spikes
3. **Cache aggressively**: Reduce redundant requests
4. **Use bulk endpoints**: Batch operations when possible

---

## API Reference

### Get Rate Limit Status

```bash
curl https://resonantgenesis.xyz/api/v1/rate-limits \
  -H "Authorization: Bearer $TOKEN"
```

**Response:**
```json
{
  "tier": "plus",
  "limits": {
    "requests_per_minute": 300,
    "requests_per_hour": 10000,
    "concurrent_sessions": 5
  },
  "usage": {
    "current_minute": 45,
    "current_hour": 1234,
    "active_sessions": 2
  },
  "reset_at": "2026-02-21T10:05:00Z"
}
```

---

**Need higher limits?** Contact sales@resonantgenesis.xyz or upgrade at [resonantgenesis.xyz/pricing](https://resonantgenesis.xyz/pricing).
