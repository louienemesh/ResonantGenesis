# Agent Tools Documentation

Complete guide to the tools available for AI agents in ResonantGenesis.

## Overview

Tools extend agent capabilities beyond conversation. They allow agents to search the web, execute code, access files, make API calls, and more.

## Available Tools

| Tool | Description | Trust Tier |
|------|-------------|------------|
| `web_search` | Search the internet | T1+ |
| `code_exec` | Execute code in sandbox | T2+ |
| `file_access` | Read and write files | T1+ |
| `api_call` | Make HTTP requests | T2+ |
| `browser` | Browse web pages | T2+ |
| `database` | Query databases | T2+ |
| `email` | Send emails | T2+ |
| `calendar` | Manage calendar | T2+ |

## Web Search

Search the internet for information.

### Configuration

```python
agent = client.agents.create(
    name="Research Agent",
    tools=["web_search"],
    tool_config={
        "web_search": {
            "max_results": 10,
            "safe_search": True,
            "regions": ["us", "uk"],
            "freshness": "week"  # day, week, month, any
        }
    }
)
```

### Usage in Agent

```python
# Agent can use web search in its execution
results = await self.tools.web_search(
    query="latest AI developments 2026",
    max_results=5
)

for result in results:
    print(f"Title: {result.title}")
    print(f"URL: {result.url}")
    print(f"Snippet: {result.snippet}")
```

### Response Format

```json
{
  "results": [
    {
      "title": "AI Breakthroughs in 2026",
      "url": "https://example.com/ai-2026",
      "snippet": "The latest developments in artificial intelligence...",
      "published_date": "2026-02-15"
    }
  ],
  "total_results": 1500,
  "search_time_ms": 250
}
```

## Code Execution

Execute code in a secure sandboxed environment.

### Supported Languages

| Language | Runtime | Version |
|----------|---------|---------|
| Python | CPython | 3.11 |
| JavaScript | Node.js | 20.x |
| TypeScript | ts-node | 5.x |
| Bash | GNU Bash | 5.x |
| SQL | SQLite | 3.x |

### Configuration

```python
agent = client.agents.create(
    name="Code Agent",
    tools=["code_exec"],
    tool_config={
        "code_exec": {
            "languages": ["python", "javascript"],
            "timeout_seconds": 30,
            "memory_mb": 512,
            "network_access": False,
            "allowed_packages": ["numpy", "pandas", "requests"]
        }
    }
)
```

### Usage in Agent

```python
# Execute Python code
result = await self.tools.code_exec(
    language="python",
    code="""
import pandas as pd
data = {'name': ['Alice', 'Bob'], 'age': [30, 25]}
df = pd.DataFrame(data)
print(df.to_string())
"""
)

print(result.stdout)  # DataFrame output
print(result.stderr)  # Any errors
print(result.exit_code)  # 0 for success
```

### Response Format

```json
{
  "stdout": "  name  age\n0  Alice   30\n1    Bob   25",
  "stderr": "",
  "exit_code": 0,
  "execution_time_ms": 150,
  "memory_used_mb": 45
}
```

### Security

- **Sandboxed**: Code runs in isolated containers
- **Resource limits**: CPU, memory, and time limits
- **No network by default**: Must explicitly enable
- **Filesystem isolation**: Cannot access host files
- **Package allowlist**: Only approved packages

## File Access

Read and write files in the agent's workspace.

### Configuration

```python
agent = client.agents.create(
    name="File Agent",
    tools=["file_access"],
    tool_config={
        "file_access": {
            "workspace_path": "/workspace",
            "max_file_size_mb": 10,
            "allowed_extensions": [".txt", ".json", ".csv", ".md"],
            "read_only": False
        }
    }
)
```

### Usage in Agent

```python
# Read a file
content = await self.tools.file_read(
    path="/workspace/data.json"
)

# Write a file
await self.tools.file_write(
    path="/workspace/output.txt",
    content="Analysis results..."
)

# List files
files = await self.tools.file_list(
    path="/workspace",
    pattern="*.csv"
)

# Delete a file
await self.tools.file_delete(
    path="/workspace/temp.txt"
)
```

### Response Format

```json
{
  "path": "/workspace/data.json",
  "content": "{\"key\": \"value\"}",
  "size_bytes": 18,
  "modified_at": "2026-02-21T10:00:00Z"
}
```

## API Calls

Make HTTP requests to external APIs.

### Configuration

```python
agent = client.agents.create(
    name="API Agent",
    tools=["api_call"],
    tool_config={
        "api_call": {
            "allowed_domains": ["api.example.com", "*.github.com"],
            "blocked_domains": ["internal.company.com"],
            "timeout_seconds": 30,
            "max_response_size_mb": 5,
            "rate_limit_per_minute": 60
        }
    }
)
```

### Usage in Agent

```python
# GET request
response = await self.tools.api_call(
    method="GET",
    url="https://api.example.com/data",
    headers={"Authorization": "Bearer token"}
)

# POST request
response = await self.tools.api_call(
    method="POST",
    url="https://api.example.com/submit",
    headers={"Content-Type": "application/json"},
    body={"key": "value"}
)

print(response.status_code)
print(response.json())
```

### Response Format

```json
{
  "status_code": 200,
  "headers": {
    "content-type": "application/json"
  },
  "body": "{\"result\": \"success\"}",
  "response_time_ms": 150
}
```

## Browser

Browse web pages and interact with content.

### Configuration

```python
agent = client.agents.create(
    name="Browser Agent",
    tools=["browser"],
    tool_config={
        "browser": {
            "headless": True,
            "timeout_seconds": 60,
            "allowed_domains": ["*"],
            "blocked_domains": ["malware.com"],
            "javascript_enabled": True,
            "screenshot_enabled": True
        }
    }
)
```

### Usage in Agent

```python
# Navigate to page
page = await self.tools.browser_navigate(
    url="https://example.com"
)

# Get page content
content = await self.tools.browser_get_content()
print(content.text)
print(content.html)

# Take screenshot
screenshot = await self.tools.browser_screenshot()

# Click element
await self.tools.browser_click(
    selector="button.submit"
)

# Fill form
await self.tools.browser_fill(
    selector="input[name='email']",
    value="user@example.com"
)

# Extract data
data = await self.tools.browser_extract(
    selectors={
        "title": "h1",
        "price": ".price",
        "description": ".desc"
    }
)
```

### Response Format

```json
{
  "url": "https://example.com",
  "title": "Example Page",
  "text": "Page content...",
  "html": "<html>...</html>",
  "screenshot_url": "https://storage.../screenshot.png"
}
```

## Database

Query databases securely.

### Configuration

```python
agent = client.agents.create(
    name="Data Agent",
    tools=["database"],
    tool_config={
        "database": {
            "connection_string": "postgresql://...",
            "read_only": True,
            "allowed_tables": ["users", "orders"],
            "max_rows": 1000,
            "timeout_seconds": 30
        }
    }
)
```

### Usage in Agent

```python
# Execute query
result = await self.tools.database_query(
    sql="SELECT * FROM users WHERE created_at > :date",
    params={"date": "2026-01-01"}
)

for row in result.rows:
    print(row)

# Get schema
schema = await self.tools.database_schema(
    table="users"
)
```

### Response Format

```json
{
  "columns": ["id", "name", "email", "created_at"],
  "rows": [
    [1, "Alice", "alice@example.com", "2026-01-15"],
    [2, "Bob", "bob@example.com", "2026-02-01"]
  ],
  "row_count": 2,
  "execution_time_ms": 50
}
```

## Email

Send emails on behalf of users.

### Configuration

```python
agent = client.agents.create(
    name="Email Agent",
    tools=["email"],
    tool_config={
        "email": {
            "from_address": "agent@example.com",
            "allowed_recipients": ["*@company.com"],
            "max_per_hour": 10,
            "require_approval": True
        }
    }
)
```

### Usage in Agent

```python
# Send email
result = await self.tools.email_send(
    to=["user@company.com"],
    subject="Report Ready",
    body="Your analysis report is complete.",
    attachments=["/workspace/report.pdf"]
)

print(result.message_id)
print(result.status)
```

### Response Format

```json
{
  "message_id": "msg-123",
  "status": "sent",
  "sent_at": "2026-02-21T10:00:00Z"
}
```

## Calendar

Manage calendar events.

### Configuration

```python
agent = client.agents.create(
    name="Calendar Agent",
    tools=["calendar"],
    tool_config={
        "calendar": {
            "calendar_id": "primary",
            "read_only": False,
            "max_events_per_day": 5
        }
    }
)
```

### Usage in Agent

```python
# List events
events = await self.tools.calendar_list(
    start="2026-02-21",
    end="2026-02-28"
)

# Create event
event = await self.tools.calendar_create(
    title="Team Meeting",
    start="2026-02-22T10:00:00",
    end="2026-02-22T11:00:00",
    attendees=["team@company.com"]
)

# Update event
await self.tools.calendar_update(
    event_id=event.id,
    title="Updated Meeting"
)

# Delete event
await self.tools.calendar_delete(
    event_id=event.id
)
```

## Custom Tools

Create your own tools for agents.

### Creating a Custom Tool

```python
# Define tool via API
tool = client.tools.create(
    name="my_api",
    description="Call my custom API",
    endpoint="https://api.myservice.com/action",
    method="POST",
    headers={"Authorization": "Bearer {{secrets.MY_API_KEY}}"},
    body_template={
        "query": "{{input.query}}",
        "options": "{{input.options}}"
    },
    input_schema={
        "type": "object",
        "properties": {
            "query": {"type": "string"},
            "options": {"type": "object"}
        },
        "required": ["query"]
    }
)

# Use in agent
agent = client.agents.create(
    name="Custom Agent",
    tools=["my_api"]
)
```

### Tool Secrets

Store sensitive data securely:

```python
# Add secret
client.secrets.create(
    name="MY_API_KEY",
    value="sk-secret-key"
)

# Reference in tool config
headers={"Authorization": "Bearer {{secrets.MY_API_KEY}}"}
```

## Tool Permissions

### Per-Agent Configuration

```python
agent = client.agents.create(
    name="Limited Agent",
    tools=["web_search", "file_access"],
    tool_permissions={
        "web_search": {
            "enabled": True,
            "rate_limit": 10  # per minute
        },
        "file_access": {
            "enabled": True,
            "read_only": True
        },
        "code_exec": {
            "enabled": False  # Explicitly disabled
        }
    }
)
```

### Runtime Overrides

```python
# Execute with restricted tools
result = await agent.execute(
    goal="Research topic",
    tool_overrides={
        "web_search": {"max_results": 3}
    }
)
```

## Best Practices

### Security

1. **Principle of least privilege**: Only enable needed tools
2. **Use allowlists**: Restrict domains, tables, recipients
3. **Enable read-only**: When writes aren't needed
4. **Set rate limits**: Prevent abuse
5. **Review tool usage**: Monitor logs regularly

### Performance

1. **Set timeouts**: Prevent hanging operations
2. **Limit response sizes**: Avoid memory issues
3. **Cache results**: Reduce redundant calls
4. **Batch operations**: Combine when possible

### Reliability

1. **Handle errors**: Tools can fail
2. **Implement retries**: For transient failures
3. **Validate inputs**: Before tool calls
4. **Log operations**: For debugging

## Troubleshooting

### Tool Not Working

1. Check tool is enabled for agent
2. Verify trust tier requirements
3. Check configuration is valid
4. Review error logs

### Permission Denied

1. Verify domain/table allowlists
2. Check rate limits
3. Confirm trust tier
4. Review tool permissions

### Timeout Errors

1. Increase timeout setting
2. Reduce operation scope
3. Check external service status
4. Optimize queries

---

**Need help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
