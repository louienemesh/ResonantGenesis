# Notifications Guide

Complete guide to the notification system, email notifications, in-app notifications, and notification preferences in ResonantGenesis.

## Overview

ResonantGenesis provides a comprehensive notification system to keep users informed about agent activity, system events, and important updates. Notifications can be delivered via email, in-app, webhooks, and mobile push.

## Notification Channels

| Channel | Description | Availability |
|---------|-------------|--------------|
| **Email** | Email notifications | All tiers |
| **In-App** | Dashboard notifications | All tiers |
| **Webhook** | HTTP callbacks | Plus+ |
| **Mobile Push** | iOS/Android push | Pro+ |
| **Slack** | Slack messages | Pro+ |
| **SMS** | Text messages | Enterprise |

## Notification Types

### Agent Notifications

| Type | Description | Default |
|------|-------------|---------|
| `agent.created` | New agent created | On |
| `agent.updated` | Agent configuration changed | Off |
| `agent.deleted` | Agent deleted | On |
| `agent.published` | Agent published to marketplace | On |
| `agent.error` | Agent execution error | On |

### Session Notifications

| Type | Description | Default |
|------|-------------|---------|
| `session.started` | Session execution started | Off |
| `session.completed` | Session completed successfully | On |
| `session.failed` | Session failed with error | On |
| `session.timeout` | Session timed out | On |

### Team Notifications

| Type | Description | Default |
|------|-------------|---------|
| `team.member_joined` | New member joined team | On |
| `team.member_left` | Member left team | On |
| `team.role_changed` | Member role changed | On |
| `team.task_assigned` | Task assigned to agent | Off |
| `team.task_completed` | Team task completed | On |

### Billing Notifications

| Type | Description | Default |
|------|-------------|---------|
| `billing.invoice_created` | New invoice generated | On |
| `billing.payment_succeeded` | Payment successful | On |
| `billing.payment_failed` | Payment failed | On |
| `billing.subscription_changed` | Plan changed | On |
| `billing.usage_warning` | Usage threshold reached | On |

### Security Notifications

| Type | Description | Default |
|------|-------------|---------|
| `security.login` | New login detected | On |
| `security.login_failed` | Failed login attempt | On |
| `security.password_changed` | Password changed | On |
| `security.api_key_created` | New API key created | On |
| `security.suspicious_activity` | Suspicious activity detected | On |

### System Notifications

| Type | Description | Default |
|------|-------------|---------|
| `system.maintenance` | Scheduled maintenance | On |
| `system.incident` | Service incident | On |
| `system.update` | Platform update | On |
| `system.announcement` | Important announcement | On |

## Configuring Notifications

### Get Current Preferences

```python
# Get notification preferences
preferences = client.notifications.get_preferences()

for category, settings in preferences.items():
    print(f"{category}:")
    for notification_type, channels in settings.items():
        print(f"  {notification_type}: {channels}")
```

### Update Preferences

```python
# Update notification preferences
client.notifications.update_preferences({
    "agent": {
        "agent.created": ["email", "in_app"],
        "agent.error": ["email", "in_app", "slack"],
        "agent.deleted": ["in_app"]
    },
    "session": {
        "session.completed": ["in_app"],
        "session.failed": ["email", "in_app", "slack"]
    },
    "billing": {
        "billing.payment_failed": ["email", "in_app", "sms"]
    }
})
```

### Disable All Notifications

```python
# Disable all notifications (not recommended)
client.notifications.disable_all()

# Re-enable with defaults
client.notifications.reset_to_defaults()
```

## Email Notifications

### Email Configuration

```python
# Configure email notifications
client.notifications.configure_email({
    "enabled": True,
    "email": "user@example.com",
    "frequency": "immediate",  # immediate, daily_digest, weekly_digest
    "format": "html"  # html, text
})
```

### Email Frequency Options

| Frequency | Description |
|-----------|-------------|
| `immediate` | Send immediately when event occurs |
| `daily_digest` | Daily summary at configured time |
| `weekly_digest` | Weekly summary on configured day |

### Email Digest Configuration

```python
# Configure daily digest
client.notifications.configure_email({
    "frequency": "daily_digest",
    "digest_time": "09:00",  # UTC
    "digest_timezone": "America/New_York"
})

# Configure weekly digest
client.notifications.configure_email({
    "frequency": "weekly_digest",
    "digest_day": "monday",
    "digest_time": "09:00"
})
```

### Unsubscribe

```python
# Unsubscribe from specific notification type
client.notifications.unsubscribe(
    notification_type="session.completed",
    channel="email"
)

# Unsubscribe from all emails
client.notifications.unsubscribe_all_email()
```

## In-App Notifications

### Get Notifications

```python
# Get recent notifications
notifications = client.notifications.list(
    limit=50,
    unread_only=False
)

for notification in notifications:
    print(f"[{notification.type}] {notification.title}")
    print(f"  {notification.message}")
    print(f"  Read: {notification.read}")
```

### Mark as Read

```python
# Mark single notification as read
client.notifications.mark_read(notification_id="notif_123")

# Mark all as read
client.notifications.mark_all_read()
```

### Delete Notifications

```python
# Delete notification
client.notifications.delete(notification_id="notif_123")

# Clear all notifications
client.notifications.clear_all()
```

### Real-time Notifications

```python
# Subscribe to real-time notifications (WebSocket)
async for notification in client.notifications.subscribe():
    print(f"New notification: {notification.title}")
    
    # Handle notification
    if notification.type == "session.failed":
        handle_session_failure(notification.data)
```

## Webhook Notifications

### Configure Webhook

```python
# Create notification webhook
webhook = client.notifications.create_webhook(
    url="https://example.com/notifications",
    events=[
        "agent.error",
        "session.failed",
        "billing.payment_failed"
    ],
    secret="webhook_secret"
)

print(f"Webhook ID: {webhook.id}")
```

### Webhook Payload

```json
{
  "id": "notif_abc123",
  "type": "session.failed",
  "timestamp": "2026-02-21T03:37:00Z",
  "data": {
    "session_id": "sess_123",
    "agent_id": "agent_456",
    "error": "Timeout after 300 seconds"
  },
  "user_id": "user_789"
}
```

### Verify Webhook Signature

```python
import hmac
import hashlib

def verify_webhook(payload: bytes, signature: str, secret: str) -> bool:
    expected = hmac.new(
        secret.encode(),
        payload,
        hashlib.sha256
    ).hexdigest()
    
    return hmac.compare_digest(f"sha256={expected}", signature)
```

## Slack Integration

### Connect Slack

```python
# Connect Slack workspace
slack_config = client.notifications.connect_slack(
    oauth_code="slack_oauth_code"
)

print(f"Connected to: {slack_config.workspace_name}")
```

### Configure Slack Notifications

```python
# Configure Slack channel for notifications
client.notifications.configure_slack({
    "default_channel": "#alerts",
    "channels": {
        "agent.error": "#agent-errors",
        "session.failed": "#session-alerts",
        "billing.*": "#billing"
    }
})
```

### Slack Message Format

```python
# Customize Slack message format
client.notifications.configure_slack_format({
    "include_details": True,
    "mention_on_critical": True,
    "mention_users": ["U12345"],
    "mention_groups": ["@oncall"]
})
```

## Mobile Push Notifications

### Register Device

```python
# Register mobile device for push notifications
client.notifications.register_device(
    device_token="firebase_device_token",
    platform="ios",  # ios, android
    app_version="1.0.0"
)
```

### Configure Push Notifications

```python
# Configure push notification preferences
client.notifications.configure_push({
    "enabled": True,
    "quiet_hours": {
        "enabled": True,
        "start": "22:00",
        "end": "08:00",
        "timezone": "America/New_York"
    },
    "critical_override": True  # Allow critical notifications during quiet hours
})
```

## SMS Notifications

### Configure SMS (Enterprise)

```python
# Configure SMS notifications
client.notifications.configure_sms({
    "enabled": True,
    "phone_number": "+1234567890",
    "events": [
        "security.suspicious_activity",
        "billing.payment_failed",
        "system.incident"
    ]
})
```

## Notification Templates

### Custom Templates

```python
# Create custom notification template
template = client.notifications.create_template(
    name="Custom Session Alert",
    type="session.failed",
    channels=["email", "slack"],
    email_template={
        "subject": "Session Failed: {{agent_name}}",
        "body": """
        <h2>Session Failed</h2>
        <p>Agent: {{agent_name}}</p>
        <p>Error: {{error_message}}</p>
        <p>Time: {{timestamp}}</p>
        <a href="{{session_url}}">View Details</a>
        """
    },
    slack_template={
        "text": ":x: Session failed for {{agent_name}}: {{error_message}}"
    }
)
```

### Template Variables

| Variable | Description |
|----------|-------------|
| `{{agent_name}}` | Agent name |
| `{{agent_id}}` | Agent ID |
| `{{session_id}}` | Session ID |
| `{{error_message}}` | Error message |
| `{{timestamp}}` | Event timestamp |
| `{{user_name}}` | User name |
| `{{session_url}}` | Link to session |
| `{{dashboard_url}}` | Link to dashboard |

## Notification Rules

### Create Rules

```python
# Create notification rule
rule = client.notifications.create_rule(
    name="High-Priority Agent Errors",
    conditions={
        "type": "agent.error",
        "agent_tags": ["production"],
        "error_severity": "critical"
    },
    actions={
        "channels": ["email", "slack", "sms"],
        "slack_channel": "#critical-alerts",
        "mention": ["@oncall"]
    }
)
```

### Rule Conditions

| Condition | Description |
|-----------|-------------|
| `type` | Notification type |
| `agent_id` | Specific agent |
| `agent_tags` | Agent tags |
| `error_severity` | Error severity level |
| `time_range` | Time of day |
| `day_of_week` | Day of week |

## Notification History

### View History

```python
# Get notification history
history = client.notifications.get_history(
    start_date="2026-02-01",
    end_date="2026-02-21",
    types=["session.failed", "agent.error"],
    limit=100
)

for notification in history:
    print(f"{notification.timestamp}: {notification.type}")
    print(f"  Delivered to: {notification.delivered_channels}")
```

### Export History

```python
# Export notification history
export = client.notifications.export_history(
    start_date="2026-01-01",
    end_date="2026-02-21",
    format="csv"
)

print(f"Download URL: {export.download_url}")
```

## Best Practices

### For Users

1. **Enable critical notifications** - Always enable security and billing alerts
2. **Use digests** - Use daily digests for non-urgent notifications
3. **Configure quiet hours** - Avoid notifications during off-hours
4. **Review regularly** - Periodically review notification settings

### For Developers

1. **Use webhooks** - Integrate notifications into your workflow
2. **Handle failures** - Implement retry logic for webhook failures
3. **Filter wisely** - Don't subscribe to everything
4. **Monitor delivery** - Track notification delivery rates

### For Teams

1. **Centralize alerts** - Use shared Slack channels
2. **Define escalation** - Set up escalation rules
3. **Rotate on-call** - Distribute notification load
4. **Document procedures** - Document response procedures

## API Reference

### Get Preferences

```bash
GET /api/v1/notifications/preferences
```

### Update Preferences

```bash
PATCH /api/v1/notifications/preferences
```

### List Notifications

```bash
GET /api/v1/notifications
```

### Mark as Read

```bash
POST /api/v1/notifications/{id}/read
```

### Create Webhook

```bash
POST /api/v1/notifications/webhooks
```

### Get History

```bash
GET /api/v1/notifications/history
```

---

**Need help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
