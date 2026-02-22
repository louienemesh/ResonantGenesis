# Integrations Guide

Complete guide to integrating ResonantGenesis with third-party services.

## Overview

ResonantGenesis integrates with popular services for:
- **Communication** - Slack, Discord, Microsoft Teams
- **Development** - GitHub, GitLab, Jira
- **Productivity** - Google Workspace, Notion
- **Monitoring** - Datadog, PagerDuty
- **Custom** - Webhooks, APIs

## Slack Integration

### Setup

1. Go to **Settings** > **Integrations** > **Slack**
2. Click **Connect to Slack**
3. Authorize ResonantGenesis in your workspace
4. Select channels for notifications

### Features

- **Agent notifications** - Execution results in channels
- **Slash commands** - Run agents from Slack
- **Interactive messages** - Approve/reject agent actions
- **Thread replies** - Conversational agent interactions

### Slash Commands

```
/resonant run <agent-name> <goal>
/resonant status <session-id>
/resonant list agents
/resonant help
```

### Webhook Trigger

```python
# Create Slack-triggered agent
trigger = agent.triggers.create(
    name="Slack Support Bot",
    type="webhook",
    config={
        "goal_template": """
            Respond to Slack message from {{event.user_name}}:
            "{{event.text}}"
            
            Provide helpful support response.
        """,
        "response_channel": "{{event.channel}}"
    }
)
```

### Sending Messages

```python
# Send to Slack channel
client.integrations.slack.send_message(
    channel="#general",
    text="Agent completed task!",
    blocks=[
        {
            "type": "section",
            "text": {"type": "mrkdwn", "text": "*Task Complete*\nAgent finished processing."}
        }
    ]
)
```

## Discord Integration

### Setup

1. Go to **Settings** > **Integrations** > **Discord**
2. Click **Add to Discord**
3. Select your server
4. Configure bot permissions

### Bot Commands

```
!resonant run <agent-name> <goal>
!resonant status <session-id>
!resonant agents
!resonant help
```

### Webhook Trigger

```python
# Create Discord-triggered agent
trigger = agent.triggers.create(
    name="Discord Bot",
    type="webhook",
    config={
        "goal_template": """
            Respond to Discord message from {{event.author.username}}:
            "{{event.content}}"
        """,
        "response_channel_id": "{{event.channel_id}}"
    }
)
```

### Sending Messages

```python
# Send to Discord channel
client.integrations.discord.send_message(
    channel_id="123456789",
    content="Agent completed!",
    embed={
        "title": "Task Complete",
        "description": "Agent finished processing.",
        "color": 0x00ff00
    }
)
```

## GitHub Integration

### Setup

1. Go to **Settings** > **Integrations** > **GitHub**
2. Click **Connect GitHub**
3. Install the GitHub App
4. Select repositories

### Features

- **PR reviews** - Automated code review
- **Issue triage** - Auto-label and assign
- **CI/CD triggers** - Run agents on push
- **Commit analysis** - Summarize changes

### Webhook Events

| Event | Description |
|-------|-------------|
| `push` | Code pushed to branch |
| `pull_request` | PR opened/updated |
| `issues` | Issue created/updated |
| `release` | Release published |

### PR Review Agent

```python
# Create PR review trigger
trigger = agent.triggers.create(
    name="PR Reviewer",
    type="webhook",
    config={
        "events": ["pull_request.opened", "pull_request.synchronize"],
        "goal_template": """
            Review PR #{{event.pull_request.number}} in {{event.repository.full_name}}:
            
            Title: {{event.pull_request.title}}
            Description: {{event.pull_request.body}}
            Files changed: {{event.pull_request.changed_files}}
            
            Provide code review feedback.
        """
    }
)
```

### Posting Comments

```python
# Post PR comment
client.integrations.github.create_comment(
    repo="owner/repo",
    issue_number=123,
    body="## Code Review\n\nLooks good! Minor suggestions..."
)
```

## GitLab Integration

### Setup

1. Go to **Settings** > **Integrations** > **GitLab**
2. Enter GitLab URL and access token
3. Select projects

### Webhook Events

| Event | Description |
|-------|-------------|
| `push` | Code pushed |
| `merge_request` | MR events |
| `issue` | Issue events |
| `pipeline` | CI/CD events |

### MR Review Agent

```python
trigger = agent.triggers.create(
    name="MR Reviewer",
    type="webhook",
    config={
        "events": ["merge_request"],
        "goal_template": """
            Review MR !{{event.object_attributes.iid}}:
            {{event.object_attributes.title}}
        """
    }
)
```

## Jira Integration

### Setup

1. Go to **Settings** > **Integrations** > **Jira**
2. Enter Jira URL and API token
3. Select projects

### Features

- **Issue creation** - Create issues from agents
- **Status updates** - Update issue status
- **Comment sync** - Add agent responses as comments
- **Sprint planning** - Analyze and estimate

### Creating Issues

```python
# Create Jira issue
issue = client.integrations.jira.create_issue(
    project="PROJ",
    issue_type="Task",
    summary="Implement feature X",
    description="Agent-generated task description",
    labels=["agent-created"]
)
```

### Webhook Trigger

```python
trigger = agent.triggers.create(
    name="Jira Handler",
    type="webhook",
    config={
        "events": ["jira:issue_created"],
        "goal_template": """
            Analyze new Jira issue {{event.issue.key}}:
            Summary: {{event.issue.fields.summary}}
            Description: {{event.issue.fields.description}}
            
            Provide initial analysis and suggestions.
        """
    }
)
```

## Google Workspace

### Setup

1. Go to **Settings** > **Integrations** > **Google**
2. Click **Connect Google Workspace**
3. Authorize access

### Gmail Integration

```python
# Send email via agent
client.integrations.google.send_email(
    to="user@example.com",
    subject="Agent Report",
    body="Here's your daily summary..."
)

# Read emails
emails = client.integrations.google.list_emails(
    query="is:unread",
    max_results=10
)
```

### Google Calendar

```python
# Create calendar event
event = client.integrations.google.create_event(
    calendar_id="primary",
    summary="Agent Meeting",
    start="2026-02-22T10:00:00",
    end="2026-02-22T11:00:00"
)

# List events
events = client.integrations.google.list_events(
    calendar_id="primary",
    time_min="2026-02-21T00:00:00Z"
)
```

### Google Drive

```python
# Upload file
file = client.integrations.google.upload_file(
    name="report.pdf",
    content=pdf_bytes,
    folder_id="folder_123"
)

# List files
files = client.integrations.google.list_files(
    query="name contains 'report'"
)
```

## Notion Integration

### Setup

1. Go to **Settings** > **Integrations** > **Notion**
2. Click **Connect Notion**
3. Select pages to share

### Features

```python
# Create page
page = client.integrations.notion.create_page(
    parent_id="database_123",
    properties={
        "Name": {"title": [{"text": {"content": "Agent Report"}}]},
        "Status": {"select": {"name": "Complete"}}
    }
)

# Query database
results = client.integrations.notion.query_database(
    database_id="database_123",
    filter={"property": "Status", "select": {"equals": "In Progress"}}
)
```

## Datadog Integration

### Setup

1. Go to **Settings** > **Integrations** > **Datadog**
2. Enter API key and app key
3. Configure metrics

### Sending Metrics

```python
# Send custom metric
client.integrations.datadog.send_metric(
    metric="resonant.agent.executions",
    value=1,
    tags=["agent:research-bot", "status:success"]
)
```

### Creating Events

```python
# Create Datadog event
client.integrations.datadog.create_event(
    title="Agent Execution Complete",
    text="Research agent completed task successfully",
    tags=["agent:research-bot"]
)
```

## PagerDuty Integration

### Setup

1. Go to **Settings** > **Integrations** > **PagerDuty**
2. Enter routing key
3. Configure alert mapping

### Triggering Incidents

```python
# Trigger incident
client.integrations.pagerduty.trigger(
    routing_key="your_routing_key",
    summary="Agent execution failed",
    severity="critical",
    source="resonant-agent",
    custom_details={
        "agent_id": "agent_123",
        "error": "Timeout exceeded"
    }
)
```

## Zapier Integration

### Setup

1. Search for "ResonantGenesis" in Zapier
2. Connect your account
3. Create Zaps

### Available Triggers

- Agent execution started
- Agent execution completed
- Session created
- Webhook received

### Available Actions

- Run agent
- Create agent
- Send message
- Update session

## Custom Webhooks

### Outgoing Webhooks

```python
# Configure outgoing webhook
webhook = client.webhooks.create(
    name="Custom Integration",
    url="https://your-server.com/webhook",
    events=["agent.execution.completed", "session.created"],
    secret="your_webhook_secret"
)
```

### Incoming Webhooks

```python
# Create incoming webhook trigger
trigger = agent.triggers.create(
    name="Custom Trigger",
    type="webhook",
    config={
        "secret": "your_secret",
        "goal_template": """
            Process incoming data:
            {{event | json}}
        """
    }
)

# Webhook URL provided
print(f"Webhook URL: {trigger.webhook_url}")
```

### Webhook Security

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

## OAuth Apps

### Creating OAuth App

1. Go to **Settings** > **Developer** > **OAuth Apps**
2. Click **Create App**
3. Configure redirect URIs
4. Get client ID and secret

### OAuth Flow

```python
# Generate authorization URL
auth_url = f"https://resonantgenesis.xyz/oauth/authorize?" \
           f"client_id={client_id}&" \
           f"redirect_uri={redirect_uri}&" \
           f"scope=agents:read+sessions:write&" \
           f"state={state}"

# Exchange code for token
response = requests.post(
    "https://resonantgenesis.xyz/oauth/token",
    data={
        "grant_type": "authorization_code",
        "code": auth_code,
        "redirect_uri": redirect_uri,
        "client_id": client_id,
        "client_secret": client_secret
    }
)
token = response.json()["access_token"]
```

## API Reference

### List Integrations

```bash
curl https://resonantgenesis.xyz/api/v1/integrations \
  -H "Authorization: Bearer $TOKEN"
```

### Get Integration Status

```bash
curl https://resonantgenesis.xyz/api/v1/integrations/{provider} \
  -H "Authorization: Bearer $TOKEN"
```

### Disconnect Integration

```bash
curl -X DELETE https://resonantgenesis.xyz/api/v1/integrations/{provider} \
  -H "Authorization: Bearer $TOKEN"
```

---

**Need help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
