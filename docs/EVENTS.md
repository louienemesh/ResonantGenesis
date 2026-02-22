# Events Guide

Complete guide to platform events, event types, and event-driven architecture in ResonantGenesis.

## Overview

ResonantGenesis uses an event-driven architecture to enable real-time notifications, webhooks, and system integrations. Events are emitted for all significant platform activities.

## Event Categories

| Category | Description | Examples |
|----------|-------------|----------|
| **Agent** | Agent lifecycle events | Created, updated, deleted |
| **Session** | Session execution events | Started, completed, failed |
| **Team** | Team coordination events | Task assigned, completed |
| **Webhook** | Webhook delivery events | Sent, failed, retried |
| **User** | User account events | Login, settings changed |
| **Billing** | Payment events | Invoice, payment, subscription |
| **System** | Platform events | Maintenance, incidents |

## Event Structure

### Base Event Format

```json
{
  "id": "evt_abc123",
  "type": "agent.created",
  "created_at": "2026-02-21T03:27:00Z",
  "data": {
    "agent_id": "agent_123",
    "name": "Research Assistant"
  },
  "metadata": {
    "user_id": "user_456",
    "organization_id": "org_789",
    "request_id": "req_xyz"
  }
}
```

### Event Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique event identifier |
| `type` | string | Event type (category.action) |
| `created_at` | datetime | When event occurred |
| `data` | object | Event-specific payload |
| `metadata` | object | Context information |

## Agent Events

### agent.created

Emitted when a new agent is created.

```json
{
  "type": "agent.created",
  "data": {
    "agent_id": "agent_123",
    "name": "Research Assistant",
    "model": "gpt-4-turbo",
    "tools": ["web_search"],
    "created_by": "user_456"
  }
}
```

### agent.updated

Emitted when an agent is modified.

```json
{
  "type": "agent.updated",
  "data": {
    "agent_id": "agent_123",
    "changes": {
      "name": {"old": "Old Name", "new": "New Name"},
      "model": {"old": "gpt-3.5-turbo", "new": "gpt-4-turbo"}
    },
    "updated_by": "user_456"
  }
}
```

### agent.deleted

Emitted when an agent is deleted.

```json
{
  "type": "agent.deleted",
  "data": {
    "agent_id": "agent_123",
    "deleted_by": "user_456",
    "permanent": false
  }
}
```

### agent.published

Emitted when an agent is published to marketplace.

```json
{
  "type": "agent.published",
  "data": {
    "agent_id": "agent_123",
    "listing_id": "mkt_abc",
    "price": 49.99,
    "dsid": "0x..."
  }
}
```

## Session Events

### session.started

Emitted when a session begins.

```json
{
  "type": "session.started",
  "data": {
    "session_id": "sess_123",
    "agent_id": "agent_456",
    "goal": "Research AI trends",
    "model": "gpt-4-turbo"
  }
}
```

### session.step.completed

Emitted when a session step completes.

```json
{
  "type": "session.step.completed",
  "data": {
    "session_id": "sess_123",
    "step_number": 3,
    "step_type": "tool_call",
    "tool": "web_search",
    "duration_ms": 1234
  }
}
```

### session.completed

Emitted when a session finishes successfully.

```json
{
  "type": "session.completed",
  "data": {
    "session_id": "sess_123",
    "agent_id": "agent_456",
    "status": "completed",
    "steps": 5,
    "duration_ms": 12345,
    "tokens_used": 2500
  }
}
```

### session.failed

Emitted when a session fails.

```json
{
  "type": "session.failed",
  "data": {
    "session_id": "sess_123",
    "agent_id": "agent_456",
    "error_code": "timeout",
    "error_message": "Session timed out after 300 seconds",
    "step_number": 3
  }
}
```

## Team Events

### team.task.assigned

Emitted when a task is assigned to a team member.

```json
{
  "type": "team.task.assigned",
  "data": {
    "team_id": "team_123",
    "task_id": "task_456",
    "assigned_to": "agent_789",
    "assigned_by": "agent_lead",
    "task_description": "Research competitor pricing"
  }
}
```

### team.task.completed

Emitted when a team task is completed.

```json
{
  "type": "team.task.completed",
  "data": {
    "team_id": "team_123",
    "task_id": "task_456",
    "completed_by": "agent_789",
    "result": "success",
    "duration_ms": 5000
  }
}
```

### team.execution.completed

Emitted when a team execution finishes.

```json
{
  "type": "team.execution.completed",
  "data": {
    "team_id": "team_123",
    "execution_id": "exec_456",
    "status": "completed",
    "agents_used": 3,
    "total_steps": 15,
    "duration_ms": 45000
  }
}
```

## Webhook Events

### webhook.sent

Emitted when a webhook is delivered.

```json
{
  "type": "webhook.sent",
  "data": {
    "webhook_id": "wh_123",
    "delivery_id": "del_456",
    "url": "https://example.com/webhook",
    "status_code": 200,
    "duration_ms": 234
  }
}
```

### webhook.failed

Emitted when a webhook delivery fails.

```json
{
  "type": "webhook.failed",
  "data": {
    "webhook_id": "wh_123",
    "delivery_id": "del_456",
    "url": "https://example.com/webhook",
    "error": "Connection timeout",
    "retry_count": 2,
    "next_retry_at": "2026-02-21T03:30:00Z"
  }
}
```

## User Events

### user.login

Emitted when a user logs in.

```json
{
  "type": "user.login",
  "data": {
    "user_id": "user_123",
    "method": "password",
    "ip_address": "192.168.1.1",
    "user_agent": "Mozilla/5.0..."
  }
}
```

### user.settings.updated

Emitted when user settings change.

```json
{
  "type": "user.settings.updated",
  "data": {
    "user_id": "user_123",
    "changes": {
      "notification_email": {"old": false, "new": true}
    }
  }
}
```

## Billing Events

### billing.invoice.created

Emitted when an invoice is generated.

```json
{
  "type": "billing.invoice.created",
  "data": {
    "invoice_id": "inv_123",
    "amount": 99.00,
    "currency": "USD",
    "period_start": "2026-02-01",
    "period_end": "2026-02-28"
  }
}
```

### billing.payment.succeeded

Emitted when a payment succeeds.

```json
{
  "type": "billing.payment.succeeded",
  "data": {
    "payment_id": "pay_123",
    "invoice_id": "inv_456",
    "amount": 99.00,
    "currency": "USD",
    "method": "card"
  }
}
```

### billing.payment.failed

Emitted when a payment fails.

```json
{
  "type": "billing.payment.failed",
  "data": {
    "payment_id": "pay_123",
    "invoice_id": "inv_456",
    "amount": 99.00,
    "error": "Card declined",
    "retry_at": "2026-02-22T00:00:00Z"
  }
}
```

## Subscribing to Events

### Webhooks

```python
# Create webhook subscription
webhook = client.webhooks.create(
    url="https://example.com/webhook",
    events=[
        "agent.created",
        "session.completed",
        "session.failed"
    ],
    secret="your_webhook_secret"
)
```

### Server-Sent Events (SSE)

```python
# Subscribe to real-time events
async for event in client.events.subscribe(
    events=["session.*"],
    session_id="sess_123"
):
    print(f"{event.type}: {event.data}")
```

### Polling

```python
# Poll for events
events = client.events.list(
    since="2026-02-21T00:00:00Z",
    types=["agent.*", "session.*"],
    limit=100
)

for event in events:
    process_event(event)
```

## Event Filtering

### By Type

```python
# Filter by event type
events = client.events.list(
    types=["session.completed", "session.failed"]
)
```

### By Resource

```python
# Filter by resource
events = client.events.list(
    agent_id="agent_123"
)

events = client.events.list(
    session_id="sess_456"
)
```

### By Time Range

```python
# Filter by time
events = client.events.list(
    since="2026-02-20T00:00:00Z",
    until="2026-02-21T00:00:00Z"
)
```

## Event Replay

### Replay Events

```python
# Replay events for debugging
replay = client.events.replay(
    since="2026-02-20T00:00:00Z",
    until="2026-02-21T00:00:00Z",
    types=["session.*"],
    webhook_url="https://example.com/replay"
)

print(f"Replaying {replay.event_count} events")
```

### Replay Status

```python
# Check replay status
status = client.events.get_replay_status(replay.id)
print(f"Progress: {status.delivered}/{status.total}")
```

## Best Practices

### Event Handling

1. **Idempotency** - Handle duplicate events gracefully
2. **Ordering** - Don't assume event order
3. **Timeouts** - Set reasonable webhook timeouts
4. **Retries** - Implement retry logic
5. **Logging** - Log all received events

### Webhook Security

1. **Verify signatures** - Always verify webhook signatures
2. **Use HTTPS** - Only use HTTPS endpoints
3. **Rotate secrets** - Rotate webhook secrets periodically
4. **IP allowlist** - Allowlist ResonantGenesis IPs

### Performance

1. **Async processing** - Process events asynchronously
2. **Queue events** - Use a message queue
3. **Batch processing** - Process events in batches
4. **Monitor latency** - Track processing time

## API Reference

### List Events

```bash
GET /api/v1/events
```

### Get Event

```bash
GET /api/v1/events/{event_id}
```

### Subscribe to Events

```bash
GET /api/v1/events/subscribe (SSE)
```

### Replay Events

```bash
POST /api/v1/events/replay
```

---

**Need help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
