# Streaming Guide

Complete guide to real-time streaming, Server-Sent Events (SSE), WebSocket connections, and streaming best practices in ResonantGenesis.

## Overview

ResonantGenesis supports real-time streaming for agent responses, session updates, and platform events. This guide covers streaming protocols, implementation patterns, and best practices for building responsive applications.

## Streaming Protocols

| Protocol | Use Case | Direction |
|----------|----------|-----------|
| **SSE** | Agent responses, events | Server → Client |
| **WebSocket** | Bidirectional communication | Bidirectional |
| **HTTP Streaming** | Large file transfers | Server → Client |

## Server-Sent Events (SSE)

### Basic SSE Connection

```python
import httpx

async def stream_agent_response():
    async with httpx.AsyncClient() as client:
        async with client.stream(
            "POST",
            "https://api.resonantgenesis.xyz/v1/sessions/execute",
            json={
                "agent_id": "agent_123",
                "goal": "Write a story",
                "stream": True
            },
            headers={"Authorization": "Bearer YOUR_API_KEY"}
        ) as response:
            async for line in response.aiter_lines():
                if line.startswith("data: "):
                    data = json.loads(line[6:])
                    print(data)
```

### SSE Event Types

| Event | Description |
|-------|-------------|
| `message` | Text content chunk |
| `tool_call` | Tool invocation |
| `tool_result` | Tool execution result |
| `step` | Session step update |
| `error` | Error occurred |
| `done` | Stream complete |

### SSE Event Format

```
event: message
data: {"type": "text", "content": "Once upon a time..."}

event: tool_call
data: {"type": "tool_call", "tool": "web_search", "args": {"query": "example"}}

event: tool_result
data: {"type": "tool_result", "tool": "web_search", "result": {...}}

event: done
data: {"type": "done", "session_id": "sess_123", "output": "..."}
```

### Python SDK Streaming

```python
from resonantgenesis import ResonantGenesis

client = ResonantGenesis(api_key="your_key")

# Stream agent execution
async for chunk in client.sessions.execute_stream(
    agent_id="agent_123",
    goal="Write a poem about AI"
):
    if chunk.type == "text":
        print(chunk.content, end="", flush=True)
    elif chunk.type == "tool_call":
        print(f"\n[Calling {chunk.tool}...]")
    elif chunk.type == "done":
        print(f"\n\nCompleted: {chunk.session_id}")
```

### JavaScript SDK Streaming

```javascript
import { ResonantGenesis } from '@resonantgenesis/sdk';

const client = new ResonantGenesis({ apiKey: 'your_key' });

// Stream agent execution
const stream = await client.sessions.executeStream({
  agentId: 'agent_123',
  goal: 'Write a poem about AI'
});

for await (const chunk of stream) {
  if (chunk.type === 'text') {
    process.stdout.write(chunk.content);
  } else if (chunk.type === 'tool_call') {
    console.log(`\n[Calling ${chunk.tool}...]`);
  } else if (chunk.type === 'done') {
    console.log(`\n\nCompleted: ${chunk.sessionId}`);
  }
}
```

## WebSocket Connections

### WebSocket Connection

```python
import websockets
import json

async def connect_websocket():
    uri = "wss://api.resonantgenesis.xyz/v1/ws"
    
    async with websockets.connect(
        uri,
        extra_headers={"Authorization": "Bearer YOUR_API_KEY"}
    ) as ws:
        # Subscribe to events
        await ws.send(json.dumps({
            "type": "subscribe",
            "channels": ["sessions", "agents"]
        }))
        
        # Listen for messages
        async for message in ws:
            data = json.loads(message)
            handle_event(data)

def handle_event(data):
    event_type = data.get("type")
    
    if event_type == "session.started":
        print(f"Session started: {data['session_id']}")
    elif event_type == "session.completed":
        print(f"Session completed: {data['session_id']}")
    elif event_type == "agent.error":
        print(f"Agent error: {data['error']}")
```

### WebSocket Message Types

| Type | Direction | Description |
|------|-----------|-------------|
| `subscribe` | Client → Server | Subscribe to channels |
| `unsubscribe` | Client → Server | Unsubscribe from channels |
| `ping` | Client → Server | Keep-alive ping |
| `pong` | Server → Client | Keep-alive response |
| `event` | Server → Client | Event notification |
| `error` | Server → Client | Error message |

### WebSocket Channels

| Channel | Events |
|---------|--------|
| `sessions` | Session lifecycle events |
| `agents` | Agent events |
| `teams` | Team events |
| `webhooks` | Webhook events |
| `billing` | Billing events |
| `system` | System notifications |

### JavaScript WebSocket

```javascript
const ws = new WebSocket('wss://api.resonantgenesis.xyz/v1/ws');

ws.onopen = () => {
  // Authenticate
  ws.send(JSON.stringify({
    type: 'auth',
    token: 'YOUR_API_KEY'
  }));
  
  // Subscribe to channels
  ws.send(JSON.stringify({
    type: 'subscribe',
    channels: ['sessions', 'agents']
  }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  
  switch (data.type) {
    case 'session.started':
      console.log('Session started:', data.session_id);
      break;
    case 'session.completed':
      console.log('Session completed:', data.session_id);
      break;
    case 'agent.error':
      console.error('Agent error:', data.error);
      break;
  }
};

ws.onerror = (error) => {
  console.error('WebSocket error:', error);
};

ws.onclose = () => {
  console.log('WebSocket closed');
  // Implement reconnection logic
};
```

## Streaming Sessions

### Interactive Chat Streaming

```python
async def interactive_chat(agent_id: str):
    """Interactive chat with streaming responses."""
    
    while True:
        user_input = input("You: ")
        if user_input.lower() == "exit":
            break
        
        print("Agent: ", end="", flush=True)
        
        async for chunk in client.sessions.execute_stream(
            agent_id=agent_id,
            goal=user_input
        ):
            if chunk.type == "text":
                print(chunk.content, end="", flush=True)
        
        print()  # New line after response
```

### Streaming with Context

```python
# Create session with context
session = client.sessions.create(
    agent_id="agent_123",
    context={
        "user_name": "Alice",
        "preferences": {"language": "en"}
    }
)

# Stream multiple turns
for turn in conversation:
    async for chunk in session.execute_stream(goal=turn):
        if chunk.type == "text":
            print(chunk.content, end="", flush=True)
    print()
```

### Cancelling Streams

```python
import asyncio

async def stream_with_timeout():
    try:
        async with asyncio.timeout(30):  # 30 second timeout
            async for chunk in client.sessions.execute_stream(
                agent_id="agent_123",
                goal="Long task"
            ):
                print(chunk.content, end="", flush=True)
    except asyncio.TimeoutError:
        print("\nStream timed out")
        # Cancel the session
        await client.sessions.cancel(session_id)
```

## Event Streaming

### Subscribe to Events

```python
# Subscribe to real-time events
async for event in client.events.subscribe(
    events=["session.*", "agent.error"]
):
    print(f"Event: {event.type}")
    print(f"Data: {event.data}")
```

### Filter Events

```python
# Subscribe with filters
async for event in client.events.subscribe(
    events=["session.completed"],
    filters={
        "agent_id": "agent_123",
        "status": "success"
    }
):
    handle_completion(event)
```

### Event Replay

```python
# Replay missed events
events = await client.events.replay(
    since="2026-02-21T00:00:00Z",
    events=["session.completed"]
)

for event in events:
    process_event(event)
```

## Frontend Integration

### React Hook

```typescript
import { useState, useEffect } from 'react';
import { ResonantGenesis } from '@resonantgenesis/sdk';

function useAgentStream(agentId: string, goal: string) {
  const [content, setContent] = useState('');
  const [isStreaming, setIsStreaming] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    if (!goal) return;

    const client = new ResonantGenesis({ apiKey: process.env.API_KEY });
    let cancelled = false;

    async function stream() {
      setIsStreaming(true);
      setContent('');
      setError(null);

      try {
        const stream = await client.sessions.executeStream({
          agentId,
          goal
        });

        for await (const chunk of stream) {
          if (cancelled) break;
          
          if (chunk.type === 'text') {
            setContent(prev => prev + chunk.content);
          }
        }
      } catch (err) {
        if (!cancelled) {
          setError(err as Error);
        }
      } finally {
        if (!cancelled) {
          setIsStreaming(false);
        }
      }
    }

    stream();

    return () => {
      cancelled = true;
    };
  }, [agentId, goal]);

  return { content, isStreaming, error };
}
```

### React Component

```tsx
function ChatMessage({ agentId, goal }: Props) {
  const { content, isStreaming, error } = useAgentStream(agentId, goal);

  if (error) {
    return <div className="error">{error.message}</div>;
  }

  return (
    <div className="message">
      <p>{content}</p>
      {isStreaming && <span className="cursor">▌</span>}
    </div>
  );
}
```

### Vue Composable

```typescript
import { ref, watch } from 'vue';
import { ResonantGenesis } from '@resonantgenesis/sdk';

export function useAgentStream(agentId: string) {
  const content = ref('');
  const isStreaming = ref(false);
  const error = ref<Error | null>(null);

  async function execute(goal: string) {
    const client = new ResonantGenesis({ apiKey: import.meta.env.VITE_API_KEY });
    
    isStreaming.value = true;
    content.value = '';
    error.value = null;

    try {
      const stream = await client.sessions.executeStream({
        agentId,
        goal
      });

      for await (const chunk of stream) {
        if (chunk.type === 'text') {
          content.value += chunk.content;
        }
      }
    } catch (err) {
      error.value = err as Error;
    } finally {
      isStreaming.value = false;
    }
  }

  return { content, isStreaming, error, execute };
}
```

## Connection Management

### Reconnection Logic

```python
import asyncio
import websockets

class ReconnectingWebSocket:
    def __init__(self, uri, api_key):
        self.uri = uri
        self.api_key = api_key
        self.ws = None
        self.reconnect_delay = 1
        self.max_delay = 60
    
    async def connect(self):
        while True:
            try:
                self.ws = await websockets.connect(
                    self.uri,
                    extra_headers={"Authorization": f"Bearer {self.api_key}"}
                )
                self.reconnect_delay = 1  # Reset delay on success
                return
            except Exception as e:
                print(f"Connection failed: {e}")
                await asyncio.sleep(self.reconnect_delay)
                self.reconnect_delay = min(
                    self.reconnect_delay * 2,
                    self.max_delay
                )
    
    async def listen(self, handler):
        while True:
            try:
                await self.connect()
                async for message in self.ws:
                    await handler(json.loads(message))
            except websockets.ConnectionClosed:
                print("Connection closed, reconnecting...")
            except Exception as e:
                print(f"Error: {e}")
```

### Heartbeat

```python
async def maintain_connection(ws):
    """Send periodic heartbeats."""
    while True:
        try:
            await ws.send(json.dumps({"type": "ping"}))
            await asyncio.sleep(30)
        except:
            break
```

### Connection Pooling

```python
from contextlib import asynccontextmanager

class StreamingPool:
    def __init__(self, max_connections=10):
        self.semaphore = asyncio.Semaphore(max_connections)
        self.connections = []
    
    @asynccontextmanager
    async def get_connection(self):
        async with self.semaphore:
            conn = await self._create_connection()
            try:
                yield conn
            finally:
                await self._release_connection(conn)
```

## Performance Optimization

### Buffering

```python
class BufferedStream:
    def __init__(self, stream, buffer_size=10):
        self.stream = stream
        self.buffer_size = buffer_size
        self.buffer = []
    
    async def __aiter__(self):
        async for chunk in self.stream:
            self.buffer.append(chunk)
            
            if len(self.buffer) >= self.buffer_size:
                yield self.buffer
                self.buffer = []
        
        if self.buffer:
            yield self.buffer
```

### Compression

```python
# Enable compression for WebSocket
async with websockets.connect(
    uri,
    compression="deflate",
    extra_headers={"Authorization": f"Bearer {api_key}"}
) as ws:
    # Compressed messages
    pass
```

### Rate Limiting

```python
from asyncio import Semaphore

class RateLimitedStream:
    def __init__(self, stream, rate_limit=100):
        self.stream = stream
        self.semaphore = Semaphore(rate_limit)
    
    async def __aiter__(self):
        async for chunk in self.stream:
            async with self.semaphore:
                yield chunk
```

## Error Handling

### Stream Errors

```python
async def handle_stream_errors():
    try:
        async for chunk in client.sessions.execute_stream(
            agent_id="agent_123",
            goal="Task"
        ):
            process_chunk(chunk)
    
    except StreamError as e:
        if e.code == "RATE_LIMITED":
            await asyncio.sleep(e.retry_after)
            # Retry
        elif e.code == "TIMEOUT":
            # Handle timeout
            pass
        elif e.code == "CONNECTION_LOST":
            # Reconnect
            pass
        else:
            raise
```

### Partial Response Recovery

```python
async def stream_with_recovery():
    last_position = 0
    
    while True:
        try:
            async for chunk in client.sessions.execute_stream(
                agent_id="agent_123",
                goal="Task",
                resume_from=last_position
            ):
                process_chunk(chunk)
                last_position = chunk.position
            break  # Success
        except ConnectionError:
            print(f"Reconnecting from position {last_position}")
            await asyncio.sleep(1)
```

## Best Practices

### For Performance

1. **Use connection pooling** - Reuse connections
2. **Enable compression** - Reduce bandwidth
3. **Buffer appropriately** - Balance latency and throughput
4. **Handle backpressure** - Don't overwhelm clients
5. **Monitor metrics** - Track latency and errors

### For Reliability

1. **Implement reconnection** - Auto-reconnect on failure
2. **Use heartbeats** - Detect dead connections
3. **Handle timeouts** - Set appropriate timeouts
4. **Log errors** - Track issues for debugging
5. **Graceful degradation** - Fallback to polling if needed

### For User Experience

1. **Show progress** - Display streaming indicator
2. **Handle interrupts** - Allow cancellation
3. **Smooth rendering** - Avoid UI jank
4. **Error messages** - Clear error feedback
5. **Offline handling** - Queue for later

## API Reference

### Start Stream

```bash
POST /api/v1/sessions/execute
Content-Type: application/json
Accept: text/event-stream

{
  "agent_id": "agent_123",
  "goal": "Task",
  "stream": true
}
```

### WebSocket Connect

```bash
GET /api/v1/ws
Upgrade: websocket
Authorization: Bearer YOUR_API_KEY
```

### Subscribe to Events

```json
{
  "type": "subscribe",
  "channels": ["sessions", "agents"]
}
```

### Unsubscribe

```json
{
  "type": "unsubscribe",
  "channels": ["sessions"]
}
```

---

**Need streaming help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
