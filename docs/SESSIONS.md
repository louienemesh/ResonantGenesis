# Sessions Guide

Complete guide to session management, context handling, and conversation flows in ResonantGenesis.

## Overview

Sessions are the core execution unit in ResonantGenesis. Each session represents a single agent execution with a goal, context, and output.

## Session Lifecycle

```
Created → Running → Completed/Failed/Cancelled
```

### States

| State | Description |
|-------|-------------|
| `created` | Session initialized, not started |
| `running` | Agent actively executing |
| `completed` | Successfully finished |
| `failed` | Execution error |
| `cancelled` | Manually stopped |
| `timeout` | Exceeded time limit |

## Creating Sessions

### Basic Session

```python
# Create and execute session
result = agent.execute(
    goal="Research AI trends for 2026"
)

print(f"Status: {result.status}")
print(f"Output: {result.output}")
```

### Session with Context

```python
# Provide context for the session
result = agent.execute(
    goal="Summarize this document",
    context={
        "document": document_text,
        "format": "bullet_points",
        "max_length": 500
    }
)
```

### Session with Options

```python
result = agent.execute(
    goal="Complex analysis task",
    max_steps=20,
    max_tokens=8000,
    timeout_seconds=300,
    tools=["web_search", "code_exec"]
)
```

## Session Configuration

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `goal` | string | required | Task description |
| `context` | dict | {} | Additional context |
| `max_steps` | int | 10 | Maximum execution steps |
| `max_tokens` | int | 4000 | Token limit |
| `timeout_seconds` | int | 120 | Timeout in seconds |
| `tools` | list | agent default | Tools to enable |
| `model` | string | agent default | Model override |
| `stream` | bool | false | Enable streaming |

### Example

```python
session = client.sessions.create(
    agent_id="agent_123",
    goal="Analyze sales data",
    context={
        "data_source": "s3://bucket/sales.csv",
        "metrics": ["revenue", "growth"]
    },
    max_steps=15,
    max_tokens=6000,
    timeout_seconds=180,
    tools=["code_exec", "file_access"]
)
```

## Context Management

### Context Types

```python
# String context
result = agent.execute(
    goal="Summarize this",
    context="Long text to summarize..."
)

# Dictionary context
result = agent.execute(
    goal="Process order",
    context={
        "order_id": "ORD-123",
        "customer": {"name": "John", "tier": "premium"},
        "items": [{"sku": "ABC", "qty": 2}]
    }
)

# File context
result = agent.execute(
    goal="Analyze this file",
    context={
        "file_path": "/data/report.pdf",
        "file_type": "pdf"
    }
)
```

### Context Best Practices

1. **Keep context focused** - Only include relevant information
2. **Structure data** - Use clear keys and values
3. **Limit size** - Large contexts consume tokens
4. **Use references** - Point to files instead of embedding

```python
# Good: Structured, focused context
context = {
    "customer_id": "cust_123",
    "issue_type": "billing",
    "previous_tickets": 2,
    "account_status": "active"
}

# Bad: Unstructured, verbose context
context = "The customer John Smith with ID cust_123 has a billing issue. He has had 2 previous tickets and his account is active..."
```

## Session Steps

### Understanding Steps

Each session consists of multiple steps:

```python
result = agent.execute(goal="Research and summarize AI trends")

for step in result.steps:
    print(f"Step {step.number}: {step.type}")
    print(f"  Input: {step.input[:100]}...")
    print(f"  Output: {step.output[:100]}...")
    print(f"  Duration: {step.duration_ms}ms")
```

### Step Types

| Type | Description |
|------|-------------|
| `thinking` | Agent reasoning |
| `tool_call` | Tool execution |
| `tool_result` | Tool response |
| `output` | Final output |
| `error` | Error occurred |

### Step Limits

```python
# Limit steps to control execution
result = agent.execute(
    goal="Quick task",
    max_steps=5  # Stop after 5 steps
)

if result.status == "max_steps_reached":
    print("Task incomplete - hit step limit")
```

## Streaming Sessions

### Enable Streaming

```python
# Stream session events
async for event in agent.execute_stream(goal="Long task"):
    if event.type == "step_start":
        print(f"Starting step {event.step_number}")
    elif event.type == "thinking":
        print(f"Thinking: {event.content}")
    elif event.type == "tool_call":
        print(f"Calling tool: {event.tool_name}")
    elif event.type == "output":
        print(f"Output: {event.content}")
    elif event.type == "done":
        print(f"Completed: {event.status}")
```

### Event Types

| Event | Description |
|-------|-------------|
| `session_start` | Session began |
| `step_start` | Step began |
| `thinking` | Agent reasoning |
| `tool_call` | Tool invoked |
| `tool_result` | Tool response |
| `step_complete` | Step finished |
| `output` | Output generated |
| `error` | Error occurred |
| `done` | Session ended |

### JavaScript Streaming

```typescript
const stream = await agent.executeStream({ goal: "Long task" });

for await (const event of stream) {
  switch (event.type) {
    case "thinking":
      console.log(`Thinking: ${event.content}`);
      break;
    case "output":
      console.log(`Output: ${event.content}`);
      break;
    case "done":
      console.log(`Status: ${event.status}`);
      break;
  }
}
```

## Session History

### List Sessions

```python
# List recent sessions
sessions = client.sessions.list(
    agent_id="agent_123",
    limit=50,
    status="completed"
)

for session in sessions:
    print(f"{session.id}: {session.goal[:50]}...")
```

### Get Session Details

```python
# Get full session details
session = client.sessions.get(session_id)

print(f"Goal: {session.goal}")
print(f"Status: {session.status}")
print(f"Steps: {len(session.steps)}")
print(f"Tokens: {session.tokens_used}")
print(f"Duration: {session.duration_ms}ms")
```

### Session Replay

```python
# Replay session steps
session = client.sessions.get(session_id)

for step in session.steps:
    print(f"\n--- Step {step.number} ({step.type}) ---")
    print(f"Input: {step.input}")
    print(f"Output: {step.output}")
```

## Conversation Sessions

### Multi-turn Conversations

```python
# Start conversation
conversation = client.conversations.create(agent_id="agent_123")

# First message
response1 = conversation.send("What is machine learning?")
print(response1.output)

# Follow-up (context preserved)
response2 = conversation.send("Can you give me an example?")
print(response2.output)

# Another follow-up
response3 = conversation.send("How is it used in healthcare?")
print(response3.output)

# End conversation
conversation.close()
```

### Conversation Context

```python
# Access conversation history
print(f"Messages: {len(conversation.messages)}")

for msg in conversation.messages:
    print(f"{msg.role}: {msg.content[:100]}...")
```

### Conversation Options

```python
conversation = client.conversations.create(
    agent_id="agent_123",
    system_override="You are a helpful coding assistant.",
    max_history=10,  # Keep last 10 messages
    context={
        "user_name": "Alice",
        "preferences": {"language": "Python"}
    }
)
```

## Session Cancellation

### Cancel Running Session

```python
# Start long-running session
session = client.sessions.create(
    agent_id="agent_123",
    goal="Very long task"
)

# Cancel if needed
client.sessions.cancel(session.id)

# Check status
session = client.sessions.get(session.id)
print(f"Status: {session.status}")  # "cancelled"
```

### Timeout Handling

```python
try:
    result = agent.execute(
        goal="Task",
        timeout_seconds=30
    )
except TimeoutError:
    print("Session timed out")
```

## Session Metrics

### Get Metrics

```python
# Session-level metrics
session = client.sessions.get(session_id)

print(f"Tokens used: {session.tokens_used}")
print(f"Input tokens: {session.input_tokens}")
print(f"Output tokens: {session.output_tokens}")
print(f"Duration: {session.duration_ms}ms")
print(f"Steps: {len(session.steps)}")
print(f"Tool calls: {session.tool_calls}")
```

### Aggregate Metrics

```python
# Agent session metrics
metrics = client.metrics.get(
    agent_id="agent_123",
    period="7d",
    metrics=["sessions_total", "avg_duration", "success_rate"]
)

print(f"Total sessions: {metrics.sessions_total}")
print(f"Avg duration: {metrics.avg_duration}ms")
print(f"Success rate: {metrics.success_rate}%")
```

## Error Handling

### Session Errors

```python
try:
    result = agent.execute(goal="Task")
except SessionError as e:
    print(f"Session failed: {e.message}")
    print(f"Error code: {e.code}")
    print(f"Session ID: {e.session_id}")
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

### Error Recovery

```python
result = agent.execute(goal="Task")

if result.status == "failed":
    # Get error details
    error = result.error
    print(f"Error: {error.message}")
    
    # Retry with modifications
    if error.code == "context_too_long":
        result = agent.execute(
            goal="Task",
            context=truncate_context(context)
        )
```

## Best Practices

### Session Design

1. **Clear goals** - Be specific about what you want
2. **Appropriate limits** - Set reasonable step/token limits
3. **Handle errors** - Always check session status
4. **Use streaming** - For long tasks, stream for feedback

### Performance

1. **Minimize context** - Only include necessary data
2. **Set timeouts** - Prevent runaway sessions
3. **Batch when possible** - Group related tasks
4. **Cache results** - Avoid duplicate executions

### Monitoring

1. **Track metrics** - Monitor duration, tokens, success rate
2. **Log sessions** - Keep records for debugging
3. **Set alerts** - Notify on failures or anomalies
4. **Review regularly** - Analyze session patterns

## API Reference

### Create Session

```bash
POST /api/v1/sessions
```

### Get Session

```bash
GET /api/v1/sessions/{session_id}
```

### List Sessions

```bash
GET /api/v1/sessions?agent_id={agent_id}&limit=50
```

### Cancel Session

```bash
POST /api/v1/sessions/{session_id}/cancel
```

### Stream Session

```bash
GET /api/v1/sessions/{session_id}/stream
```

---

**Need help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
