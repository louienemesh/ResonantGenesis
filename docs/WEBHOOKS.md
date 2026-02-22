# Webhooks Documentation

Complete guide to webhook triggers and integrations in ResonantGenesis.

## Overview

Webhooks allow external services to trigger agent executions automatically. When an event occurs (e.g., GitHub push, Slack message), a webhook sends data to ResonantGenesis, which starts an agent session.

## How Webhooks Work

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   External   │───▶│  ResonantGen │───▶│    Agent     │
│   Service    │    │   Webhook    │    │  Execution   │
└──────────────┘    └──────────────┘    └──────────────┘
     GitHub              API              Your Agent
     Slack            Validates           Processes
     Custom           & Routes            Event Data
```

## Creating a Webhook Trigger

### Via API

```bash
curl -X POST https://resonantgenesis.xyz/api/v1/agents/{agent_id}/triggers \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "GitHub Push Trigger",
    "type": "webhook",
    "config": {
      "secret": "your-webhook-secret",
      "goal_template": "Process GitHub push: {{event.commits | length}} commits to {{event.ref}}"
    }
  }'
```

**Response:**
```json
{
  "id": "trigger-uuid",
  "webhook_url": "https://resonantgenesis.xyz/api/v1/agents/triggers/webhook/trigger-uuid",
  "secret": "your-webhook-secret",
  "created_at": "2026-02-21T00:00:00Z"
}
```

### Via SDK

**Python:**
```python
trigger = client.agents.get(agent_id).triggers.create(
    name="GitHub Push Trigger",
    type="webhook",
    config={
        "secret": "your-webhook-secret",
        "goal_template": "Process GitHub push: {{event.commits | length}} commits"
    }
)

print(f"Webhook URL: {trigger.webhook_url}")
```

**JavaScript:**
```typescript
const trigger = await client.agents.get(agentId).triggers.create({
  name: 'GitHub Push Trigger',
  type: 'webhook',
  config: {
    secret: 'your-webhook-secret',
    goalTemplate: 'Process GitHub push: {{event.commits.length}} commits'
  }
});

console.log(`Webhook URL: ${trigger.webhookUrl}`);
```

## Webhook Security

### Signature Verification

All webhook requests should include a signature header for verification:

```
X-Webhook-Signature: sha256=abc123...
```

**Verification Example (Python):**
```python
import hmac
import hashlib

def verify_signature(payload: bytes, signature: str, secret: str) -> bool:
    expected = hmac.new(
        secret.encode(),
        payload,
        hashlib.sha256
    ).hexdigest()
    
    return hmac.compare_digest(f"sha256={expected}", signature)

# In your webhook handler
@app.post("/webhook")
async def handle_webhook(request: Request):
    payload = await request.body()
    signature = request.headers.get("X-Webhook-Signature")
    
    if not verify_signature(payload, signature, WEBHOOK_SECRET):
        raise HTTPException(status_code=401, detail="Invalid signature")
    
    # Process webhook...
```

**Verification Example (JavaScript):**
```typescript
import crypto from 'crypto';

function verifySignature(payload: string, signature: string, secret: string): boolean {
  const expected = crypto
    .createHmac('sha256', secret)
    .update(payload)
    .digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(`sha256=${expected}`),
    Buffer.from(signature)
  );
}
```

### IP Allowlisting

For additional security, you can restrict webhook sources by IP:

```bash
curl -X PATCH https://resonantgenesis.xyz/api/v1/agents/{agent_id}/triggers/{trigger_id} \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "config": {
      "allowed_ips": ["192.30.252.0/22", "185.199.108.0/22"]
    }
  }'
```

## Goal Templates

Goal templates use Jinja2 syntax to construct dynamic goals from webhook payloads.

### Template Syntax

| Syntax | Description | Example |
|--------|-------------|---------|
| `{{variable}}` | Insert variable | `{{event.action}}` |
| `{{var \| filter}}` | Apply filter | `{{commits \| length}}` |
| `{% if %}` | Conditional | `{% if event.action == 'opened' %}` |
| `{% for %}` | Loop | `{% for commit in commits %}` |

### Common Filters

| Filter | Description | Example |
|--------|-------------|---------|
| `length` | Array length | `{{commits \| length}}` |
| `first` | First item | `{{commits \| first}}` |
| `last` | Last item | `{{commits \| last}}` |
| `join` | Join array | `{{files \| join(', ')}}` |
| `truncate` | Truncate string | `{{message \| truncate(100)}}` |

### Example Templates

**GitHub Push:**
```
Analyze {{event.commits | length}} new commits to {{event.repository.full_name}}:
{% for commit in event.commits %}
- {{commit.message | truncate(50)}}
{% endfor %}
```

**Slack Message:**
```
Respond to Slack message from {{event.user.name}} in #{{event.channel.name}}:
"{{event.text}}"
```

**Custom Webhook:**
```
Process {{event.type}} event with data: {{event.data | tojson}}
```

## Integration Guides

### GitHub

#### Setup

1. Create a webhook trigger for your agent
2. Go to your GitHub repository → Settings → Webhooks
3. Add webhook:
   - **Payload URL**: Your trigger's webhook URL
   - **Content type**: `application/json`
   - **Secret**: Your webhook secret
   - **Events**: Select events to trigger

#### Supported Events

| Event | Description | Template Variables |
|-------|-------------|-------------------|
| `push` | Code pushed | `commits`, `ref`, `repository` |
| `pull_request` | PR opened/updated | `action`, `pull_request`, `repository` |
| `issues` | Issue created/updated | `action`, `issue`, `repository` |
| `issue_comment` | Comment on issue | `action`, `comment`, `issue` |
| `release` | Release published | `action`, `release`, `repository` |

#### Example: Code Review Agent

```python
trigger = agent.triggers.create(
    name="PR Review Trigger",
    type="webhook",
    config={
        "secret": "github-secret",
        "goal_template": """
Review pull request #{{event.pull_request.number}} in {{event.repository.full_name}}:
Title: {{event.pull_request.title}}
Description: {{event.pull_request.body | truncate(500)}}
Files changed: {{event.pull_request.changed_files}}

Analyze the code changes and provide feedback on:
1. Code quality
2. Potential bugs
3. Security issues
4. Performance concerns
"""
    }
)
```

### Slack

#### Setup

1. Create a Slack App at [api.slack.com](https://api.slack.com/apps)
2. Enable Event Subscriptions
3. Set Request URL to your webhook URL
4. Subscribe to events: `message.channels`, `app_mention`
5. Install app to workspace

#### Supported Events

| Event | Description | Template Variables |
|-------|-------------|-------------------|
| `message.channels` | Channel message | `text`, `user`, `channel`, `ts` |
| `app_mention` | Bot mentioned | `text`, `user`, `channel` |
| `reaction_added` | Reaction added | `reaction`, `item`, `user` |

#### Example: Support Bot

```python
trigger = agent.triggers.create(
    name="Slack Support Trigger",
    type="webhook",
    config={
        "secret": "slack-signing-secret",
        "goal_template": """
Respond to support request from {{event.user_name}} in #{{event.channel_name}}:

Message: "{{event.text}}"

Provide helpful support by:
1. Understanding the user's issue
2. Searching our knowledge base
3. Providing a clear, friendly response
"""
    }
)
```

### Discord

#### Setup

1. Go to Server Settings → Integrations → Webhooks
2. Create webhook and copy URL
3. Use a middleware service to forward to ResonantGenesis
4. Or use Discord Bot with webhook forwarding

#### Example: Community Bot

```python
trigger = agent.triggers.create(
    name="Discord Community Trigger",
    type="webhook",
    config={
        "goal_template": """
Respond to Discord message from {{event.author.username}}:

"{{event.content}}"

Be helpful and friendly. Follow community guidelines.
"""
    }
)
```

### Custom Webhooks

For any service that can send HTTP POST requests:

```bash
# Your service sends:
curl -X POST https://resonantgenesis.xyz/api/v1/agents/triggers/webhook/{trigger_id} \
  -H "Content-Type: application/json" \
  -H "X-Webhook-Signature: sha256=..." \
  -d '{
    "event_type": "order_placed",
    "order_id": "12345",
    "customer": "John Doe",
    "items": ["Widget A", "Widget B"],
    "total": 99.99
  }'
```

**Template:**
```
Process new order #{{event.order_id}} for {{event.customer}}:
Items: {{event.items | join(', ')}}
Total: ${{event.total}}

Generate order confirmation and update inventory.
```

## Webhook Management

### List Triggers

```bash
curl https://resonantgenesis.xyz/api/v1/agents/{agent_id}/triggers \
  -H "Authorization: Bearer $TOKEN"
```

### Get Trigger Details

```bash
curl https://resonantgenesis.xyz/api/v1/agents/{agent_id}/triggers/{trigger_id} \
  -H "Authorization: Bearer $TOKEN"
```

### Update Trigger

```bash
curl -X PATCH https://resonantgenesis.xyz/api/v1/agents/{agent_id}/triggers/{trigger_id} \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Updated Trigger Name",
    "config": {
      "goal_template": "New template: {{event.data}}"
    }
  }'
```

### Delete Trigger

```bash
curl -X DELETE https://resonantgenesis.xyz/api/v1/agents/{agent_id}/triggers/{trigger_id} \
  -H "Authorization: Bearer $TOKEN"
```

### Pause/Resume Trigger

```bash
# Pause
curl -X POST https://resonantgenesis.xyz/api/v1/agents/{agent_id}/triggers/{trigger_id}/pause \
  -H "Authorization: Bearer $TOKEN"

# Resume
curl -X POST https://resonantgenesis.xyz/api/v1/agents/{agent_id}/triggers/{trigger_id}/resume \
  -H "Authorization: Bearer $TOKEN"
```

## Webhook Logs

View webhook execution history:

```bash
curl https://resonantgenesis.xyz/api/v1/agents/{agent_id}/triggers/{trigger_id}/logs \
  -H "Authorization: Bearer $TOKEN"
```

**Response:**
```json
{
  "logs": [
    {
      "id": "log-uuid",
      "received_at": "2026-02-21T10:00:00Z",
      "status": "success",
      "session_id": "session-uuid",
      "payload_preview": "{\"event\": \"push\"...}",
      "response_time_ms": 150
    }
  ]
}
```

## Error Handling

### Retry Policy

Failed webhooks are retried with exponential backoff:

| Attempt | Delay |
|---------|-------|
| 1 | Immediate |
| 2 | 1 minute |
| 3 | 5 minutes |
| 4 | 30 minutes |
| 5 | 2 hours |

After 5 failures, the webhook is marked as failed and no further retries occur.

### Error Responses

| Status | Description |
|--------|-------------|
| 200 | Success |
| 400 | Invalid payload |
| 401 | Invalid signature |
| 404 | Trigger not found |
| 429 | Rate limited |
| 500 | Internal error |

## Best Practices

1. **Always use secrets**: Verify webhook signatures
2. **Use specific events**: Don't subscribe to all events
3. **Keep templates simple**: Complex logic should be in the agent
4. **Monitor logs**: Check webhook logs regularly
5. **Handle failures**: Implement retry logic in your service
6. **Test thoroughly**: Use webhook testing tools before production

## Troubleshooting

### Webhook not triggering

1. Check webhook URL is correct
2. Verify signature is being sent
3. Check trigger is not paused
4. Review webhook logs for errors

### Invalid signature errors

1. Ensure secret matches on both sides
2. Check payload encoding (UTF-8)
3. Verify signature algorithm (SHA-256)

### Rate limiting

1. Reduce webhook frequency
2. Upgrade to higher tier
3. Batch events if possible

---

**Need help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
