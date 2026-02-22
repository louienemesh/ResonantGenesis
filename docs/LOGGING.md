# Logging Guide

Complete guide to logging configuration, log levels, structured logging, and log management in ResonantGenesis.

## Overview

ResonantGenesis provides comprehensive logging capabilities for debugging, monitoring, and auditing. This guide covers log configuration, structured logging, log aggregation, and best practices.

## Log Levels

| Level | Value | Description | Use Case |
|-------|-------|-------------|----------|
| `DEBUG` | 10 | Detailed diagnostic info | Development |
| `INFO` | 20 | General operational info | Normal operations |
| `WARNING` | 30 | Unexpected but handled | Potential issues |
| `ERROR` | 40 | Error occurred | Failures |
| `CRITICAL` | 50 | System failure | Critical failures |

## Configuration

### Basic Configuration

```python
from resonantgenesis import ResonantGenesis

client = ResonantGenesis(
    api_key="your_key",
    log_level="INFO",
    log_format="json"
)
```

### Environment Variables

```bash
# Set log level
export RG_LOG_LEVEL=DEBUG

# Set log format
export RG_LOG_FORMAT=json

# Set log output
export RG_LOG_OUTPUT=stdout

# Enable file logging
export RG_LOG_FILE=/var/log/resonant/app.log
```

### Configuration File

```yaml
# config/logging.yaml
logging:
  level: INFO
  format: json
  output: stdout
  
  file:
    enabled: true
    path: /var/log/resonant/app.log
    max_size: 100MB
    max_files: 10
    compress: true
  
  handlers:
    console:
      enabled: true
      level: INFO
      format: colored
    
    file:
      enabled: true
      level: DEBUG
      format: json
    
    syslog:
      enabled: false
      host: localhost
      port: 514
```

## Structured Logging

### JSON Format

```python
import logging
from resonantgenesis.logging import JSONFormatter

logger = logging.getLogger("resonant")
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)

logger.info("User action", extra={
    "user_id": "user_123",
    "action": "create_agent",
    "agent_id": "agent_456"
})
```

Output:
```json
{
  "timestamp": "2026-02-21T18:30:00.000Z",
  "level": "INFO",
  "message": "User action",
  "user_id": "user_123",
  "action": "create_agent",
  "agent_id": "agent_456",
  "service": "resonant",
  "hostname": "server-1"
}
```

### Context Logging

```python
from resonantgenesis.logging import LogContext

# Add context to all logs in scope
with LogContext(user_id="user_123", session_id="sess_456"):
    logger.info("Processing request")
    # All logs include user_id and session_id
    do_something()
    logger.info("Request completed")
```

### Request Logging

```python
from resonantgenesis.logging import RequestLogger

# Automatic request/response logging
@RequestLogger()
async def handle_request(request):
    # Logs: request received, processing time, response status
    return await process(request)
```

## Agent Logging

### Agent Execution Logs

```python
# Get agent execution logs
logs = client.agents.get_logs(
    agent_id="agent_123",
    level="DEBUG",
    limit=100
)

for log in logs:
    print(f"[{log.timestamp}] {log.level}: {log.message}")
```

### Session Logs

```python
# Get session logs
logs = client.sessions.get_logs(
    session_id="sess_123",
    include_steps=True
)

for log in logs:
    print(f"Step {log.step}: {log.message}")
```

### Tool Execution Logs

```python
# Get tool execution logs
logs = client.tools.get_logs(
    agent_id="agent_123",
    tool="web_search",
    period="1h"
)

for log in logs:
    print(f"Tool: {log.tool}, Duration: {log.duration_ms}ms")
```

## Log Aggregation

### Elasticsearch

```python
from resonantgenesis.logging import ElasticsearchHandler

handler = ElasticsearchHandler(
    hosts=["http://localhost:9200"],
    index="resonant-logs",
    auth=("user", "password")
)

logger.addHandler(handler)
```

### Datadog

```python
from resonantgenesis.logging import DatadogHandler

handler = DatadogHandler(
    api_key="your_datadog_api_key",
    service="resonant",
    env="production"
)

logger.addHandler(handler)
```

### CloudWatch

```python
from resonantgenesis.logging import CloudWatchHandler

handler = CloudWatchHandler(
    log_group="/resonant/production",
    log_stream="app",
    region="us-east-1"
)

logger.addHandler(handler)
```

### Splunk

```python
from resonantgenesis.logging import SplunkHandler

handler = SplunkHandler(
    host="splunk.example.com",
    port=8088,
    token="your_hec_token",
    index="resonant"
)

logger.addHandler(handler)
```

## Log Filtering

### Filter by Level

```python
# Get only error logs
logs = client.logs.query(
    level="ERROR",
    period="24h"
)
```

### Filter by Agent

```python
# Get logs for specific agent
logs = client.logs.query(
    agent_id="agent_123",
    period="1h"
)
```

### Filter by User

```python
# Get logs for specific user
logs = client.logs.query(
    user_id="user_123",
    period="7d"
)
```

### Complex Queries

```python
# Complex log query
logs = client.logs.query(
    filters={
        "level": ["ERROR", "WARNING"],
        "agent_id": "agent_123",
        "message": {"contains": "timeout"}
    },
    period="24h",
    limit=1000
)
```

## Log Retention

### Retention Policies

| Log Type | Default Retention | Max Retention |
|----------|-------------------|---------------|
| Debug | 7 days | 30 days |
| Info | 30 days | 90 days |
| Warning | 90 days | 1 year |
| Error | 1 year | 3 years |
| Audit | 7 years | Indefinite |

### Configure Retention

```python
# Set retention policy
client.logs.set_retention(
    log_type="debug",
    retention_days=14
)
```

### Archive Logs

```python
# Archive old logs
archive = client.logs.archive(
    before="2026-01-01",
    destination="s3://my-bucket/logs/"
)

print(f"Archived {archive.log_count} logs")
```

## Audit Logging

### Audit Events

```python
# Audit log is automatic for sensitive operations
# Examples:
# - User authentication
# - Agent creation/deletion
# - Permission changes
# - API key operations
# - Billing changes

# Query audit logs
audit_logs = client.audit.query(
    event_type="agent.deleted",
    period="30d"
)

for log in audit_logs:
    print(f"{log.timestamp}: {log.actor} deleted {log.resource}")
```

### Audit Log Format

```json
{
  "timestamp": "2026-02-21T18:30:00.000Z",
  "event_type": "agent.deleted",
  "actor": {
    "type": "user",
    "id": "user_123",
    "email": "user@example.com"
  },
  "resource": {
    "type": "agent",
    "id": "agent_456",
    "name": "My Agent"
  },
  "action": "delete",
  "result": "success",
  "ip_address": "192.168.1.1",
  "user_agent": "Mozilla/5.0..."
}
```

## Log Streaming

### Real-time Log Streaming

```python
# Stream logs in real-time
async for log in client.logs.stream(
    agent_id="agent_123",
    level="DEBUG"
):
    print(f"[{log.timestamp}] {log.message}")
```

### WebSocket Log Stream

```javascript
const ws = new WebSocket('wss://api.resonantgenesis.xyz/v1/logs/stream');

ws.onopen = () => {
  ws.send(JSON.stringify({
    type: 'subscribe',
    filters: {
      agent_id: 'agent_123',
      level: 'DEBUG'
    }
  }));
};

ws.onmessage = (event) => {
  const log = JSON.parse(event.data);
  console.log(`[${log.timestamp}] ${log.message}`);
};
```

## Log Analysis

### Log Statistics

```python
# Get log statistics
stats = client.logs.get_stats(period="24h")

print(f"Total logs: {stats.total}")
print(f"Errors: {stats.error_count}")
print(f"Warnings: {stats.warning_count}")
print(f"Error rate: {stats.error_rate}%")
```

### Error Analysis

```python
# Analyze errors
analysis = client.logs.analyze_errors(period="7d")

for error in analysis.top_errors:
    print(f"{error.message}: {error.count} occurrences")
    print(f"  First seen: {error.first_seen}")
    print(f"  Last seen: {error.last_seen}")
```

### Pattern Detection

```python
# Detect log patterns
patterns = client.logs.detect_patterns(
    period="24h",
    min_occurrences=10
)

for pattern in patterns:
    print(f"Pattern: {pattern.template}")
    print(f"Occurrences: {pattern.count}")
```

## Log Alerts

### Create Log Alert

```python
# Alert on error patterns
alert = client.alerts.create(
    name="High Error Rate",
    condition={
        "metric": "log_error_rate",
        "operator": "greater_than",
        "threshold": 5,
        "window": "5m"
    },
    actions=["email", "slack"]
)
```

### Alert on Specific Errors

```python
# Alert on specific error message
alert = client.alerts.create(
    name="Database Connection Error",
    condition={
        "log_message": {"contains": "database connection failed"},
        "count": {"greater_than": 3},
        "window": "5m"
    },
    actions=["pagerduty"]
)
```

## Performance Logging

### Request Timing

```python
from resonantgenesis.logging import TimingLogger

with TimingLogger("process_request") as timer:
    result = process_request()
    timer.add_metric("items_processed", len(result))

# Logs: "process_request completed in 150ms, items_processed=42"
```

### Slow Query Logging

```python
# Configure slow query logging
client.configure_logging(
    slow_query_threshold_ms=1000,
    log_slow_queries=True
)

# Automatically logs queries taking >1000ms
```

## SDK Logging

### Python SDK

```python
import logging

# Enable SDK debug logging
logging.getLogger("resonantgenesis").setLevel(logging.DEBUG)

# Custom handler
handler = logging.StreamHandler()
handler.setFormatter(logging.Formatter(
    '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
))
logging.getLogger("resonantgenesis").addHandler(handler)
```

### JavaScript SDK

```javascript
import { ResonantGenesis } from '@resonantgenesis/sdk';

const client = new ResonantGenesis({
  apiKey: 'your_key',
  logLevel: 'debug',
  logger: console // or custom logger
});
```

## Best Practices

### For Development

1. **Use DEBUG level** - Capture detailed info
2. **Include context** - Add relevant IDs
3. **Log entry/exit** - Track function flow
4. **Log exceptions** - Include stack traces
5. **Use structured logging** - JSON format

### For Production

1. **Use INFO level** - Reduce noise
2. **Aggregate logs** - Centralized logging
3. **Set retention** - Manage storage
4. **Monitor errors** - Alert on issues
5. **Secure logs** - Protect sensitive data

### For Security

1. **Never log secrets** - API keys, passwords
2. **Mask PII** - Personal information
3. **Audit access** - Track log access
4. **Encrypt at rest** - Secure storage
5. **Retain appropriately** - Compliance requirements

## Troubleshooting

### Missing Logs

```python
# Check log level
print(f"Current level: {logger.level}")

# Verify handlers
for handler in logger.handlers:
    print(f"Handler: {handler}, Level: {handler.level}")
```

### Log Performance

```python
# Use async logging for high volume
from resonantgenesis.logging import AsyncHandler

handler = AsyncHandler(
    target_handler=file_handler,
    queue_size=10000
)
```

## API Reference

### Query Logs

```bash
GET /api/v1/logs
```

### Stream Logs

```bash
GET /api/v1/logs/stream
```

### Get Log Stats

```bash
GET /api/v1/logs/stats
```

### Archive Logs

```bash
POST /api/v1/logs/archive
```

### Set Retention

```bash
PUT /api/v1/logs/retention
```

---

**Need logging help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
