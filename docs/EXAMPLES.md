# Code Examples

Practical code examples for common ResonantGenesis use cases.

## Table of Contents

- [Basic Agent Creation](#basic-agent-creation)
- [Agent with Tools](#agent-with-tools)
- [Session Management](#session-management)
- [Agent Teams](#agent-teams)
- [Webhooks](#webhooks)
- [Blockchain Integration](#blockchain-integration)
- [Error Handling](#error-handling)
- [Advanced Patterns](#advanced-patterns)

---

## Basic Agent Creation

### Python

```python
from resonantgenesis import ResonantClient

# Initialize client
client = ResonantClient(api_key="your_api_key")

# Create a simple agent
agent = client.agents.create(
    name="Research Assistant",
    description="Helps with research tasks",
    system_prompt="""You are a helpful research assistant.
    - Search for accurate information
    - Summarize findings clearly
    - Cite sources when possible
    - Be concise and factual""",
    model="gpt-4-turbo",
    temperature=0.7
)

print(f"Created agent: {agent.id}")
print(f"Status: {agent.status}")
```

### JavaScript/TypeScript

```typescript
import { ResonantClient } from '@resonantgenesis/sdk';

const client = new ResonantClient({ apiKey: 'your_api_key' });

// Create a simple agent
const agent = await client.agents.create({
  name: 'Research Assistant',
  description: 'Helps with research tasks',
  systemPrompt: `You are a helpful research assistant.
    - Search for accurate information
    - Summarize findings clearly
    - Cite sources when possible
    - Be concise and factual`,
  model: 'gpt-4-turbo',
  temperature: 0.7
});

console.log(`Created agent: ${agent.id}`);
console.log(`Status: ${agent.status}`);
```

### cURL

```bash
curl -X POST https://resonantgenesis.xyz/api/v1/agents \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Research Assistant",
    "description": "Helps with research tasks",
    "system_prompt": "You are a helpful research assistant.",
    "model": "gpt-4-turbo",
    "temperature": 0.7
  }'
```

---

## Agent with Tools

### Python - Web Search Agent

```python
# Create agent with web search capability
search_agent = client.agents.create(
    name="Web Researcher",
    system_prompt="You search the web and summarize findings.",
    model="gpt-4-turbo",
    tools=["web_search"],
    tool_config={
        "web_search": {
            "max_results": 10,
            "safe_search": True
        }
    }
)

# Execute a search task
result = search_agent.execute(
    goal="Find the latest news about quantum computing"
)

print(result.output)
```

### Python - Code Execution Agent

```python
# Create agent with code execution
code_agent = client.agents.create(
    name="Code Assistant",
    system_prompt="""You are a Python expert.
    Write and execute code to solve problems.
    Always test your code before presenting results.""",
    model="gpt-4-turbo",
    tools=["code_exec"],
    tool_config={
        "code_exec": {
            "languages": ["python"],
            "timeout_seconds": 30,
            "allowed_packages": ["numpy", "pandas", "matplotlib"]
        }
    }
)

# Execute a coding task
result = code_agent.execute(
    goal="Create a bar chart showing sales by month: Jan=100, Feb=150, Mar=200"
)

print(result.output)
# Result includes generated chart
```

### Python - Multi-Tool Agent

```python
# Create agent with multiple tools
multi_agent = client.agents.create(
    name="Full Stack Assistant",
    system_prompt="You can search, code, and access files.",
    model="gpt-4-turbo",
    tools=["web_search", "code_exec", "file_access"],
    tool_config={
        "web_search": {"max_results": 5},
        "code_exec": {"languages": ["python", "javascript"]},
        "file_access": {"workspace_path": "/workspace"}
    }
)
```

---

## Session Management

### Python - Basic Session

```python
# Start a session
session = agent.execute(
    goal="Research AI trends for 2026",
    max_steps=10
)

# Wait for completion
result = session.wait()

print(f"Status: {result.status}")
print(f"Output: {result.output}")
print(f"Steps: {len(result.steps)}")
print(f"Tokens used: {result.tokens_used}")
```

### Python - Streaming Session

```python
import asyncio

async def stream_session():
    async for event in agent.execute_stream(
        goal="Write a blog post about AI"
    ):
        if event.type == "step_start":
            print(f"Starting: {event.action}")
        elif event.type == "step_complete":
            print(f"Completed: {event.output[:100]}...")
        elif event.type == "session_complete":
            print(f"Done: {event.result}")

asyncio.run(stream_session())
```

### Python - Session with Context

```python
# Execute with additional context
result = agent.execute(
    goal="Analyze this data and provide insights",
    context={
        "data_source": "sales_2026.csv",
        "focus_areas": ["trends", "anomalies", "predictions"],
        "output_format": "executive_summary"
    },
    max_steps=15,
    timeout_seconds=300
)
```

### JavaScript - Async Session

```typescript
// Start session and handle events
const session = await agent.execute({
  goal: 'Research AI trends',
  maxSteps: 10
});

// Poll for status
while (session.status === 'running') {
  await new Promise(resolve => setTimeout(resolve, 1000));
  await session.refresh();
  console.log(`Progress: ${session.stepsCompleted}/${session.maxSteps}`);
}

console.log(`Result: ${session.output}`);
```

---

## Agent Teams

### Python - Sequential Team

```python
# Create specialized agents
researcher = client.agents.create(
    name="Researcher",
    system_prompt="You research topics thoroughly."
)

writer = client.agents.create(
    name="Writer",
    system_prompt="You write engaging content from research."
)

editor = client.agents.create(
    name="Editor",
    system_prompt="You polish and improve written content."
)

# Create sequential team
team = client.teams.create(
    name="Content Team",
    coordination_mode="sequential",
    agent_ids=[researcher.id, writer.id, editor.id]
)

# Execute team task
result = await team.execute(
    task="Create a blog post about sustainable energy"
)

print(result.final_output)
```

### Python - Hierarchical Team

```python
# Create team with lead agent
team = client.teams.create(
    name="Development Team",
    coordination_mode="hierarchical",
    lead_agent_id=architect.id,
    agent_ids=[frontend.id, backend.id, tester.id],
    roles={
        architect.id: {"role": "coordinator"},
        frontend.id: {"role": "executor"},
        backend.id: {"role": "executor"},
        tester.id: {"role": "reviewer"}
    }
)

# Execute with delegation
result = await team.execute(
    task="Build a user authentication system",
    delegations={
        frontend.id: "Create login and signup forms",
        backend.id: "Implement auth API endpoints",
        tester.id: "Write integration tests"
    }
)
```

### Python - Parallel Team

```python
# Create parallel analysis team
team = client.teams.create(
    name="Analysis Team",
    coordination_mode="parallel",
    agent_ids=[market_analyst.id, tech_analyst.id, finance_analyst.id]
)

# All agents work simultaneously
result = await team.execute(
    task="Analyze Company XYZ for investment potential"
)

# Results from all agents combined
print(result.agent_contributions)
```

---

## Webhooks

### Python - Create Webhook Trigger

```python
# Create webhook for GitHub events
trigger = agent.triggers.create(
    name="GitHub Push Handler",
    type="webhook",
    config={
        "secret": "your-webhook-secret",
        "goal_template": """
            Process GitHub push to {{event.repository.full_name}}:
            - Branch: {{event.ref}}
            - Commits: {{event.commits | length}}
            - Author: {{event.pusher.name}}
            
            Review the changes and provide a summary.
        """
    }
)

print(f"Webhook URL: {trigger.webhook_url}")
```

### Python - Slack Integration

```python
# Create Slack message handler
trigger = agent.triggers.create(
    name="Slack Support Bot",
    type="webhook",
    config={
        "secret": "slack-signing-secret",
        "goal_template": """
            Respond to Slack message from {{event.user_name}}:
            
            "{{event.text}}"
            
            Provide helpful, friendly support.
        """
    }
)
```

### Python - Custom Webhook

```python
# Handle custom webhook events
trigger = agent.triggers.create(
    name="Order Processor",
    type="webhook",
    config={
        "goal_template": """
            Process new order #{{event.order_id}}:
            - Customer: {{event.customer_name}}
            - Items: {{event.items | join(', ')}}
            - Total: ${{event.total}}
            
            Generate confirmation and update inventory.
        """
    }
)

# Send test event
import requests
import hmac
import hashlib

payload = {
    "order_id": "12345",
    "customer_name": "John Doe",
    "items": ["Widget A", "Widget B"],
    "total": 99.99
}

signature = hmac.new(
    b"your-webhook-secret",
    json.dumps(payload).encode(),
    hashlib.sha256
).hexdigest()

response = requests.post(
    trigger.webhook_url,
    json=payload,
    headers={
        "X-Webhook-Signature": f"sha256={signature}",
        "Content-Type": "application/json"
    }
)
```

---

## Blockchain Integration

> **Note**: ResonantGenesis uses a hybrid blockchain architecture. Identity registration (DSIDs) uses Base (Ethereum L2). Agent registration and memory anchoring use an internal blockchain and are abstracted via the API.

### Python - Register Identity (Base - External)

```python
from web3 import Web3

# Connect to Base
w3 = Web3(Web3.HTTPProvider('https://mainnet.base.org'))

# Generate DSID
identity_name = "my-agent-identity"
dsid = w3.keccak(text=identity_name)

# Register on-chain
tx = identity_registry.functions.registerIdentity(
    dsid,
    public_key_bytes
).build_transaction({
    'from': account.address,
    'nonce': w3.eth.get_transaction_count(account.address),
    'gas': 200000
})

signed = account.sign_transaction(tx)
tx_hash = w3.eth.send_raw_transaction(signed.rawTransaction)
receipt = w3.eth.wait_for_transaction_receipt(tx_hash)

print(f"Identity registered: {dsid.hex()}")
```

### Python - Publish Agent (Internal Chain via API)

```python
# Publish agent (uses internal blockchain via API)
publication = agent.publish(
    price_per_execution=0.001,  # ETH
    rental_enabled=True,
    rental_price_per_day=0.01  # ETH
)

print(f"DSID: {publication.dsid}")
print(f"Transaction: {publication.transaction_hash}")
print(f"Block: {publication.block_number}")
```

### Python - Anchor Memory

```python
import json

# Create memory content
memory = {
    "session_id": "session-123",
    "context": "User discussed quantum computing",
    "key_points": ["qubits", "superposition", "entanglement"],
    "timestamp": "2026-02-21T10:00:00Z"
}

# Hash and anchor
content_hash = w3.keccak(text=json.dumps(memory, sort_keys=True))

tx = memory_anchors.functions.anchor(content_hash).build_transaction({
    'from': account.address,
    'nonce': w3.eth.get_transaction_count(account.address),
    'gas': 100000
})

signed = account.sign_transaction(tx)
tx_hash = w3.eth.send_raw_transaction(signed.rawTransaction)

print(f"Memory anchored: {content_hash.hex()}")
```

---

## Error Handling

### Python - Comprehensive Error Handling

```python
from resonantgenesis import (
    ResonantClient,
    AuthenticationError,
    RateLimitError,
    AgentError,
    SessionError,
    NetworkError
)

client = ResonantClient(api_key="your_api_key")

try:
    # Create and execute agent
    agent = client.agents.create(
        name="Test Agent",
        system_prompt="You are helpful."
    )
    
    result = agent.execute(goal="Test task")
    print(result.output)

except AuthenticationError as e:
    print(f"Auth failed: {e.message}")
    # Refresh token or re-authenticate

except RateLimitError as e:
    print(f"Rate limited. Retry after: {e.retry_after}s")
    import time
    time.sleep(e.retry_after)
    # Retry request

except AgentError as e:
    print(f"Agent error: {e.message}")
    print(f"Agent ID: {e.agent_id}")
    # Check agent configuration

except SessionError as e:
    print(f"Session error: {e.message}")
    print(f"Session ID: {e.session_id}")
    # Check session status, maybe retry

except NetworkError as e:
    print(f"Network error: {e.message}")
    # Check connectivity, retry with backoff

except Exception as e:
    print(f"Unexpected error: {e}")
    # Log and alert
```

### Python - Retry with Backoff

```python
import time
from functools import wraps

def retry_with_backoff(max_retries=3, base_delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except RateLimitError as e:
                    delay = e.retry_after or (base_delay * (2 ** attempt))
                    print(f"Rate limited. Waiting {delay}s...")
                    time.sleep(delay)
                except NetworkError:
                    delay = base_delay * (2 ** attempt)
                    print(f"Network error. Retrying in {delay}s...")
                    time.sleep(delay)
            raise Exception("Max retries exceeded")
        return wrapper
    return decorator

@retry_with_backoff(max_retries=5)
def execute_agent_task(agent, goal):
    return agent.execute(goal=goal)
```

---

## Advanced Patterns

### Python - Agent Factory

```python
class AgentFactory:
    def __init__(self, client):
        self.client = client
        self.templates = {
            "researcher": {
                "system_prompt": "You are an expert researcher.",
                "tools": ["web_search"],
                "model": "gpt-4-turbo"
            },
            "coder": {
                "system_prompt": "You are an expert programmer.",
                "tools": ["code_exec", "file_access"],
                "model": "gpt-4-turbo"
            },
            "writer": {
                "system_prompt": "You are a skilled writer.",
                "tools": [],
                "model": "gpt-4-turbo"
            }
        }
    
    def create(self, agent_type: str, name: str, **overrides):
        if agent_type not in self.templates:
            raise ValueError(f"Unknown type: {agent_type}")
        
        config = {**self.templates[agent_type], **overrides}
        return self.client.agents.create(name=name, **config)

# Usage
factory = AgentFactory(client)
researcher = factory.create("researcher", "My Researcher")
coder = factory.create("coder", "My Coder", temperature=0.3)
```

### Python - Async Batch Processing

```python
import asyncio

async def process_batch(agent, tasks: list[str]):
    """Process multiple tasks concurrently."""
    async def process_one(task):
        try:
            result = await agent.execute_async(goal=task)
            return {"task": task, "status": "success", "output": result.output}
        except Exception as e:
            return {"task": task, "status": "error", "error": str(e)}
    
    results = await asyncio.gather(*[process_one(t) for t in tasks])
    return results

# Usage
tasks = [
    "Research topic A",
    "Research topic B",
    "Research topic C"
]

results = asyncio.run(process_batch(agent, tasks))
for r in results:
    print(f"{r['task']}: {r['status']}")
```

### Python - Caching Layer

```python
from functools import lru_cache
import hashlib

class CachedAgent:
    def __init__(self, agent, cache_size=100):
        self.agent = agent
        self._cache = {}
        self.cache_size = cache_size
    
    def _cache_key(self, goal: str, context: dict = None):
        data = f"{goal}:{context}"
        return hashlib.sha256(data.encode()).hexdigest()
    
    def execute(self, goal: str, context: dict = None, use_cache=True):
        key = self._cache_key(goal, context)
        
        if use_cache and key in self._cache:
            return self._cache[key]
        
        result = self.agent.execute(goal=goal, context=context)
        
        if len(self._cache) >= self.cache_size:
            # Remove oldest entry
            oldest = next(iter(self._cache))
            del self._cache[oldest]
        
        self._cache[key] = result
        return result

# Usage
cached = CachedAgent(agent)
result1 = cached.execute("What is AI?")  # Executes
result2 = cached.execute("What is AI?")  # Returns cached
```

### Python - Monitoring Wrapper

```python
import time
import logging

class MonitoredAgent:
    def __init__(self, agent, logger=None):
        self.agent = agent
        self.logger = logger or logging.getLogger(__name__)
        self.metrics = {
            "total_executions": 0,
            "successful": 0,
            "failed": 0,
            "total_tokens": 0,
            "total_time_ms": 0
        }
    
    def execute(self, **kwargs):
        start = time.time()
        self.metrics["total_executions"] += 1
        
        try:
            result = self.agent.execute(**kwargs)
            self.metrics["successful"] += 1
            self.metrics["total_tokens"] += result.tokens_used
            return result
        except Exception as e:
            self.metrics["failed"] += 1
            self.logger.error(f"Execution failed: {e}")
            raise
        finally:
            elapsed = (time.time() - start) * 1000
            self.metrics["total_time_ms"] += elapsed
            self.logger.info(f"Execution took {elapsed:.2f}ms")
    
    def get_metrics(self):
        return {
            **self.metrics,
            "success_rate": self.metrics["successful"] / max(1, self.metrics["total_executions"]),
            "avg_time_ms": self.metrics["total_time_ms"] / max(1, self.metrics["total_executions"])
        }

# Usage
monitored = MonitoredAgent(agent)
result = monitored.execute(goal="Test task")
print(monitored.get_metrics())
```

---

## Next Steps

- **[API Reference](./API_REFERENCE.md)** - Complete API documentation
- **[SDK Guide](./SDK_GUIDE.md)** - SDK installation and usage
- **[Troubleshooting](./TROUBLESHOOTING.md)** - Common issues and solutions

---

**Need help?** Join our [Discord](https://discord.gg/resonantgenesis) or contact support@resonantgenesis.xyz.
