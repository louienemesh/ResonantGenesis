# Best Practices Guide

Complete guide to best practices for agent design, prompt engineering, and production deployment.

## Agent Design

### Naming Conventions

```python
# Good: Clear, descriptive names
agent = client.agents.create(
    name="Customer Support Assistant",
    description="Handles customer inquiries and support tickets"
)

# Bad: Vague names
agent = client.agents.create(
    name="Bot1",
    description="Does stuff"
)
```

### System Prompt Design

#### Structure

```
You are [role].

## Capabilities
- [capability 1]
- [capability 2]

## Guidelines
- [guideline 1]
- [guideline 2]

## Constraints
- [constraint 1]
- [constraint 2]

## Output Format
[expected format]
```

#### Example

```python
system_prompt = """You are a technical documentation writer.

## Capabilities
- Write clear, concise documentation
- Create code examples in Python and JavaScript
- Structure content with proper headings

## Guidelines
- Use simple language, avoid jargon
- Include practical examples
- Keep paragraphs short (3-4 sentences)

## Constraints
- Do not make up API endpoints
- Always verify code examples compile
- Stay within the documentation scope

## Output Format
Use Markdown with proper headings, code blocks, and lists.
"""
```

### Single Responsibility

```python
# Good: Focused agents
research_agent = client.agents.create(
    name="Research Agent",
    system_prompt="You research topics and gather information."
)

writer_agent = client.agents.create(
    name="Writer Agent",
    system_prompt="You write content from research notes."
)

# Bad: Overloaded agent
do_everything_agent = client.agents.create(
    name="Everything Agent",
    system_prompt="You research, write, edit, publish, and market content."
)
```

### Tool Selection

```python
# Good: Minimal necessary tools
agent = client.agents.create(
    name="Research Agent",
    tools=["web_search"],  # Only what's needed
    tool_config={
        "web_search": {"max_results": 5}
    }
)

# Bad: All tools enabled
agent = client.agents.create(
    name="Research Agent",
    tools=["web_search", "code_exec", "file_access", "api_call", "browser", "database", "email", "calendar"]
)
```

## Prompt Engineering

### Be Specific

```python
# Good: Specific instructions
goal = """
Analyze the Q4 2025 sales report and:
1. Identify top 3 performing products
2. Calculate month-over-month growth
3. Highlight any concerning trends
4. Provide 3 actionable recommendations

Output as a structured report with sections.
"""

# Bad: Vague instructions
goal = "Look at the sales data and tell me what you think."
```

### Provide Context

```python
# Good: Rich context
result = agent.execute(
    goal="Draft a response to this customer complaint",
    context={
        "customer_name": "John Smith",
        "account_tier": "Premium",
        "complaint": "Order delayed by 5 days",
        "order_id": "ORD-12345",
        "previous_issues": 0,
        "company_policy": "Full refund for delays over 3 days"
    }
)

# Bad: Missing context
result = agent.execute(
    goal="Respond to the customer"
)
```

### Use Examples

```python
system_prompt = """You classify customer feedback.

## Examples

Input: "The product is amazing, works perfectly!"
Output: {"sentiment": "positive", "category": "product_quality", "priority": "low"}

Input: "I've been waiting 2 weeks for my order!"
Output: {"sentiment": "negative", "category": "shipping", "priority": "high"}

Input: "How do I reset my password?"
Output: {"sentiment": "neutral", "category": "support", "priority": "medium"}
"""
```

### Chain of Thought

```python
system_prompt = """You solve complex problems step by step.

When given a problem:
1. Understand the problem completely
2. Break it into smaller parts
3. Solve each part systematically
4. Verify your solution
5. Present the final answer

Always show your reasoning process.
"""
```

### Output Formatting

```python
# Good: Structured output
system_prompt = """
Output your analysis as JSON:
{
    "summary": "Brief summary",
    "findings": ["finding 1", "finding 2"],
    "recommendations": ["rec 1", "rec 2"],
    "confidence": 0.0-1.0
}
"""

# Bad: Unstructured output
system_prompt = "Tell me what you found."
```

## Session Management

### Set Appropriate Limits

```python
# Good: Reasonable limits
result = agent.execute(
    goal="Research topic X",
    max_steps=10,
    max_tokens=4000,
    timeout_seconds=120
)

# Bad: No limits (can run forever)
result = agent.execute(
    goal="Research topic X"
)
```

### Handle Long Tasks

```python
# Good: Break into subtasks
subtasks = [
    "Research market size",
    "Identify competitors",
    "Analyze pricing strategies",
    "Summarize findings"
]

results = []
for task in subtasks:
    result = agent.execute(goal=task, max_steps=5)
    results.append(result)

# Combine results
final = combine_results(results)
```

### Use Streaming for Long Operations

```python
# Good: Stream for real-time feedback
async for event in agent.execute_stream(goal="Long task"):
    if event.type == "step_complete":
        print(f"Progress: {event.step}/{event.total_steps}")
    elif event.type == "output":
        print(f"Output: {event.content}")
```

## Error Handling

### Graceful Degradation

```python
async def execute_with_fallback(agent, goal):
    try:
        return await agent.execute(goal=goal)
    except RateLimitError as e:
        await asyncio.sleep(e.retry_after)
        return await agent.execute(goal=goal)
    except ModelUnavailableError:
        # Fall back to simpler model
        agent.model = "gpt-3.5-turbo"
        return await agent.execute(goal=goal)
    except Exception as e:
        logger.error(f"Execution failed: {e}")
        return {"error": str(e), "status": "failed"}
```

### Validation

```python
def validate_agent_output(output, schema):
    """Validate agent output against expected schema."""
    try:
        jsonschema.validate(output, schema)
        return True, None
    except jsonschema.ValidationError as e:
        return False, str(e)

# Usage
result = agent.execute(goal="Generate report")
valid, error = validate_agent_output(result.output, report_schema)

if not valid:
    # Retry with clarification
    result = agent.execute(
        goal=f"Generate report. Previous output was invalid: {error}"
    )
```

### Retry Logic

```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=4, max=60)
)
async def reliable_execute(agent, goal):
    return await agent.execute(goal=goal)
```

## Production Deployment

### Environment Configuration

```python
import os

# Good: Environment-based config
client = ResonantClient(
    api_key=os.environ["RESONANT_API_KEY"],
    environment=os.environ.get("RESONANT_ENV", "production")
)

# Bad: Hardcoded values
client = ResonantClient(
    api_key="rg_live_sk_123456"  # Never do this!
)
```

### Logging

```python
import structlog

logger = structlog.get_logger()

async def execute_agent(agent_id: str, goal: str):
    logger.info(
        "agent_execution_started",
        agent_id=agent_id,
        goal_length=len(goal)
    )
    
    try:
        result = await agent.execute(goal=goal)
        logger.info(
            "agent_execution_completed",
            agent_id=agent_id,
            steps=len(result.steps),
            tokens=result.tokens_used
        )
        return result
    except Exception as e:
        logger.error(
            "agent_execution_failed",
            agent_id=agent_id,
            error=str(e)
        )
        raise
```

### Monitoring

```python
from prometheus_client import Counter, Histogram

executions_total = Counter(
    'agent_executions_total',
    'Total agent executions',
    ['agent_id', 'status']
)

execution_duration = Histogram(
    'agent_execution_duration_seconds',
    'Agent execution duration',
    ['agent_id']
)

async def monitored_execute(agent, goal):
    with execution_duration.labels(agent_id=agent.id).time():
        try:
            result = await agent.execute(goal=goal)
            executions_total.labels(agent_id=agent.id, status='success').inc()
            return result
        except Exception:
            executions_total.labels(agent_id=agent.id, status='failure').inc()
            raise
```

### Rate Limiting

```python
from asyncio import Semaphore

class RateLimitedClient:
    def __init__(self, client, max_concurrent=10):
        self.client = client
        self.semaphore = Semaphore(max_concurrent)
    
    async def execute(self, agent, goal):
        async with self.semaphore:
            return await agent.execute(goal=goal)

# Usage
limited_client = RateLimitedClient(client, max_concurrent=5)
```

### Caching

```python
from functools import lru_cache
import hashlib

class CachedAgent:
    def __init__(self, agent, cache_ttl=3600):
        self.agent = agent
        self.cache = {}
        self.cache_ttl = cache_ttl
    
    def _cache_key(self, goal: str) -> str:
        return hashlib.sha256(goal.encode()).hexdigest()
    
    async def execute(self, goal: str, use_cache=True):
        key = self._cache_key(goal)
        
        if use_cache and key in self.cache:
            cached = self.cache[key]
            if time.time() - cached['time'] < self.cache_ttl:
                return cached['result']
        
        result = await self.agent.execute(goal=goal)
        self.cache[key] = {'result': result, 'time': time.time()}
        return result
```

## Security

### Input Validation

```python
def sanitize_goal(goal: str) -> str:
    """Sanitize user input before passing to agent."""
    # Remove potential injection attempts
    goal = goal.replace("{{", "").replace("}}", "")
    
    # Limit length
    if len(goal) > 10000:
        goal = goal[:10000]
    
    return goal

# Usage
user_goal = request.json.get("goal")
safe_goal = sanitize_goal(user_goal)
result = agent.execute(goal=safe_goal)
```

### Output Filtering

```python
def filter_sensitive_data(output: str) -> str:
    """Remove sensitive data from agent output."""
    import re
    
    # Remove potential API keys
    output = re.sub(r'[a-zA-Z0-9_]{32,}', '[REDACTED]', output)
    
    # Remove email addresses
    output = re.sub(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b', '[EMAIL]', output)
    
    return output
```

### Access Control

```python
def check_agent_access(user_id: str, agent_id: str) -> bool:
    """Verify user has access to agent."""
    agent = client.agents.get(agent_id)
    return agent.owner_id == user_id or user_id in agent.shared_with
```

## Testing

### Unit Tests

```python
import pytest
from unittest.mock import Mock, patch

class TestAgentExecution:
    @pytest.fixture
    def mock_agent(self):
        agent = Mock()
        agent.execute.return_value = Mock(
            status="completed",
            output="Test output"
        )
        return agent
    
    def test_execute_returns_output(self, mock_agent):
        result = mock_agent.execute(goal="Test goal")
        assert result.status == "completed"
        assert result.output == "Test output"
```

### Integration Tests

```python
@pytest.mark.integration
async def test_agent_web_search():
    agent = client.agents.create(
        name="Test Agent",
        tools=["web_search"]
    )
    
    result = await agent.execute(
        goal="What is the capital of France?"
    )
    
    assert result.status == "completed"
    assert "Paris" in result.output
    
    # Cleanup
    await client.agents.delete(agent.id)
```

### Load Tests

```python
import asyncio
import time

async def load_test(agent, num_requests=100):
    start = time.time()
    
    tasks = [
        agent.execute(goal=f"Task {i}")
        for i in range(num_requests)
    ]
    
    results = await asyncio.gather(*tasks, return_exceptions=True)
    
    elapsed = time.time() - start
    successful = sum(1 for r in results if not isinstance(r, Exception))
    
    print(f"Completed {successful}/{num_requests} in {elapsed:.2f}s")
    print(f"Throughput: {successful/elapsed:.2f} req/s")
```

## Checklist

### Before Production

- [ ] System prompts reviewed and tested
- [ ] Error handling implemented
- [ ] Rate limiting configured
- [ ] Logging enabled
- [ ] Monitoring set up
- [ ] Security review completed
- [ ] Load testing performed
- [ ] Fallback strategies defined

### Ongoing

- [ ] Monitor error rates
- [ ] Review agent outputs periodically
- [ ] Update prompts based on feedback
- [ ] Optimize token usage
- [ ] Review security logs

---

**Need help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
