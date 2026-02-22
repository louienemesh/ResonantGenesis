# Monitoring & Observability Guide

Complete guide to monitoring, metrics, logging, and alerting in ResonantGenesis.

## Overview

ResonantGenesis provides comprehensive observability through:
- **Metrics** - Quantitative measurements
- **Logging** - Event records
- **Tracing** - Request flow tracking
- **Alerting** - Automated notifications

## Metrics

### Available Metrics

#### Agent Metrics

| Metric | Type | Description |
|--------|------|-------------|
| `agent.executions.total` | Counter | Total executions |
| `agent.executions.success` | Counter | Successful executions |
| `agent.executions.failed` | Counter | Failed executions |
| `agent.execution.duration_ms` | Histogram | Execution duration |
| `agent.tokens.input` | Counter | Input tokens used |
| `agent.tokens.output` | Counter | Output tokens used |

#### Session Metrics

| Metric | Type | Description |
|--------|------|-------------|
| `session.active` | Gauge | Active sessions |
| `session.created.total` | Counter | Sessions created |
| `session.completed.total` | Counter | Sessions completed |
| `session.duration_ms` | Histogram | Session duration |
| `session.steps.total` | Counter | Total steps executed |

#### API Metrics

| Metric | Type | Description |
|--------|------|-------------|
| `api.requests.total` | Counter | Total API requests |
| `api.requests.latency_ms` | Histogram | Request latency |
| `api.requests.errors` | Counter | Error responses |
| `api.rate_limit.exceeded` | Counter | Rate limit hits |

#### System Metrics

| Metric | Type | Description |
|--------|------|-------------|
| `system.cpu.usage` | Gauge | CPU utilization |
| `system.memory.usage` | Gauge | Memory utilization |
| `system.disk.usage` | Gauge | Disk utilization |
| `db.connections.active` | Gauge | Active DB connections |
| `redis.connections.active` | Gauge | Active Redis connections |

### Accessing Metrics

#### Via API

```bash
curl https://resonantgenesis.xyz/api/v1/metrics \
  -H "Authorization: Bearer $TOKEN"
```

**Response:**
```json
{
  "period": "1h",
  "metrics": {
    "agent_executions_total": 1500,
    "agent_executions_success": 1450,
    "agent_executions_failed": 50,
    "avg_execution_duration_ms": 2500,
    "total_tokens_used": 150000,
    "active_sessions": 25
  }
}
```

#### Via SDK

```python
metrics = client.metrics.get(
    period="1h",
    metrics=["agent.executions.total", "agent.tokens.input"]
)

print(f"Executions: {metrics.agent_executions_total}")
print(f"Tokens: {metrics.agent_tokens_input}")
```

#### Via Dashboard

1. Go to **Control Center** > **Monitoring**
2. Select time range
3. View charts and graphs

### Prometheus Integration

Export metrics in Prometheus format:

```bash
curl https://resonantgenesis.xyz/api/v1/metrics/prometheus \
  -H "Authorization: Bearer $TOKEN"
```

**Response:**
```
# HELP agent_executions_total Total agent executions
# TYPE agent_executions_total counter
agent_executions_total{status="success"} 1450
agent_executions_total{status="failed"} 50

# HELP agent_execution_duration_ms Agent execution duration
# TYPE agent_execution_duration_ms histogram
agent_execution_duration_ms_bucket{le="100"} 50
agent_execution_duration_ms_bucket{le="500"} 500
agent_execution_duration_ms_bucket{le="1000"} 1000
```

### Grafana Dashboard

Import our pre-built dashboard:

```json
{
  "dashboard": {
    "title": "ResonantGenesis Overview",
    "panels": [
      {
        "title": "Executions per Minute",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(agent_executions_total[1m])"
          }
        ]
      }
    ]
  }
}
```

## Logging

### Log Levels

| Level | Description |
|-------|-------------|
| `DEBUG` | Detailed debugging information |
| `INFO` | General operational events |
| `WARN` | Warning conditions |
| `ERROR` | Error conditions |
| `FATAL` | Critical failures |

### Log Format

```json
{
  "timestamp": "2026-02-21T10:00:00.000Z",
  "level": "INFO",
  "service": "backend",
  "request_id": "req_abc123",
  "user_id": "user_123",
  "agent_id": "agent_456",
  "message": "Agent execution started",
  "metadata": {
    "goal": "Research AI trends",
    "model": "gpt-4-turbo"
  }
}
```

### Accessing Logs

#### Via API

```bash
curl "https://resonantgenesis.xyz/api/v1/logs?level=ERROR&limit=100" \
  -H "Authorization: Bearer $TOKEN"
```

#### Via SDK

```python
logs = client.logs.list(
    level="ERROR",
    start_time="2026-02-21T00:00:00Z",
    end_time="2026-02-21T23:59:59Z",
    limit=100
)

for log in logs:
    print(f"[{log.level}] {log.timestamp}: {log.message}")
```

#### Via Dashboard

1. Go to **Control Center** > **Logs**
2. Filter by level, time, service
3. Search by keyword

### Log Retention

| Tier | Retention |
|------|-----------|
| Free | 7 days |
| Plus | 30 days |
| Pro | 90 days |
| Enterprise | Custom |

### Structured Logging

```python
import structlog

logger = structlog.get_logger()

logger.info(
    "agent_execution_started",
    agent_id=agent.id,
    goal=session.goal,
    model=agent.model
)
```

### Log Aggregation

#### Elasticsearch

```yaml
# filebeat.yml
filebeat.inputs:
  - type: log
    paths:
      - /var/log/resonantgenesis/*.log
    json.keys_under_root: true

output.elasticsearch:
  hosts: ["elasticsearch:9200"]
  index: "resonantgenesis-%{+yyyy.MM.dd}"
```

#### Datadog

```python
import logging
from datadog import initialize, statsd

initialize(api_key="your_api_key")

handler = logging.handlers.DatadogLogHandler()
logger.addHandler(handler)
```

## Tracing

### Distributed Tracing

ResonantGenesis supports OpenTelemetry for distributed tracing.

#### Trace Structure

```
Trace: req_abc123
├── Span: API Request (50ms)
│   ├── Span: Authentication (5ms)
│   ├── Span: Agent Lookup (10ms)
│   └── Span: Session Execution (35ms)
│       ├── Span: Step 1: Web Search (15ms)
│       ├── Span: Step 2: Analysis (10ms)
│       └── Span: Step 3: Response (10ms)
```

#### Enabling Tracing

```python
from opentelemetry import trace
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# Configure exporter
jaeger_exporter = JaegerExporter(
    agent_host_name="localhost",
    agent_port=6831,
)

# Set up tracer
provider = TracerProvider()
processor = BatchSpanProcessor(jaeger_exporter)
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
```

#### Trace Headers

Include trace context in requests:

```bash
curl https://resonantgenesis.xyz/api/v1/agents \
  -H "Authorization: Bearer $TOKEN" \
  -H "traceparent: 00-abc123-def456-01"
```

### Jaeger Integration

```yaml
# docker-compose.yml
services:
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "16686:16686"  # UI
      - "6831:6831/udp"  # Thrift
```

## Alerting

### Alert Types

| Type | Description |
|------|-------------|
| `threshold` | Metric exceeds threshold |
| `anomaly` | Unusual pattern detected |
| `absence` | Expected data missing |
| `composite` | Multiple conditions |

### Creating Alerts

#### Via API

```bash
curl -X POST https://resonantgenesis.xyz/api/v1/alerts \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "High Error Rate",
    "type": "threshold",
    "metric": "agent.executions.failed",
    "condition": {
      "operator": "gt",
      "value": 10,
      "period": "5m"
    },
    "channels": ["email", "slack"],
    "severity": "critical"
  }'
```

#### Via SDK

```python
alert = client.alerts.create(
    name="High Error Rate",
    type="threshold",
    metric="agent.executions.failed",
    condition={
        "operator": "gt",
        "value": 10,
        "period": "5m"
    },
    channels=["email", "slack"],
    severity="critical"
)
```

### Alert Channels

#### Email

```python
client.alert_channels.create(
    type="email",
    config={
        "recipients": ["ops@example.com"],
        "subject_prefix": "[ResonantGenesis Alert]"
    }
)
```

#### Slack

```python
client.alert_channels.create(
    type="slack",
    config={
        "webhook_url": "https://hooks.slack.com/...",
        "channel": "#alerts"
    }
)
```

#### PagerDuty

```python
client.alert_channels.create(
    type="pagerduty",
    config={
        "routing_key": "your_routing_key",
        "severity_map": {
            "critical": "critical",
            "warning": "warning"
        }
    }
)
```

#### Webhook

```python
client.alert_channels.create(
    type="webhook",
    config={
        "url": "https://your-server.com/alerts",
        "headers": {"Authorization": "Bearer token"}
    }
)
```

### Alert Payload

```json
{
  "id": "alert_123",
  "name": "High Error Rate",
  "severity": "critical",
  "status": "firing",
  "fired_at": "2026-02-21T10:00:00Z",
  "metric": "agent.executions.failed",
  "value": 15,
  "threshold": 10,
  "message": "Error rate exceeded threshold: 15 > 10 in last 5m"
}
```

### Alert Management

```python
# List alerts
alerts = client.alerts.list()

# Get alert status
alert = client.alerts.get(alert_id)
print(f"Status: {alert.status}")

# Acknowledge alert
client.alerts.acknowledge(alert_id)

# Resolve alert
client.alerts.resolve(alert_id)

# Silence alert
client.alerts.silence(alert_id, duration_hours=2)
```

## Health Checks

### Endpoints

```bash
# Basic health
curl https://resonantgenesis.xyz/api/v1/health

# Detailed health
curl https://resonantgenesis.xyz/api/v1/health/detailed \
  -H "Authorization: Bearer $TOKEN"
```

### Health Response

```json
{
  "status": "healthy",
  "version": "1.2.0",
  "uptime_seconds": 86400,
  "checks": {
    "database": {"status": "healthy", "latency_ms": 5},
    "redis": {"status": "healthy", "latency_ms": 2},
    "node": {"status": "healthy", "chain_connected": true},
    "ai_models": {"status": "healthy", "available": ["gpt-4", "claude-3"]}
  }
}
```

### Kubernetes Probes

```yaml
livenessProbe:
  httpGet:
    path: /api/v1/health
    port: 8000
  initialDelaySeconds: 10
  periodSeconds: 30

readinessProbe:
  httpGet:
    path: /api/v1/health/ready
    port: 8000
  initialDelaySeconds: 5
  periodSeconds: 10
```

## Dashboards

### Control Center

Built-in dashboards available at:
- **Overview** - Key metrics summary
- **Agents** - Agent performance
- **Sessions** - Session analytics
- **API** - API usage and errors
- **System** - Infrastructure health

### Custom Dashboards

Create custom dashboards via API:

```python
dashboard = client.dashboards.create(
    name="My Dashboard",
    panels=[
        {
            "title": "Executions",
            "type": "timeseries",
            "metric": "agent.executions.total",
            "period": "1h"
        },
        {
            "title": "Error Rate",
            "type": "gauge",
            "metric": "agent.executions.failed",
            "threshold": {"warning": 5, "critical": 10}
        }
    ]
)
```

## Best Practices

### Metrics

1. **Use labels wisely** - Don't create high-cardinality labels
2. **Set appropriate retention** - Balance cost vs. analysis needs
3. **Create SLIs/SLOs** - Define service level indicators

### Logging

1. **Use structured logging** - JSON format for parsing
2. **Include context** - Request ID, user ID, agent ID
3. **Don't log sensitive data** - Mask PII and secrets
4. **Set appropriate levels** - Use DEBUG sparingly in production

### Alerting

1. **Avoid alert fatigue** - Only alert on actionable items
2. **Use severity levels** - Critical, warning, info
3. **Include runbooks** - Link to remediation docs
4. **Test alerts** - Verify they fire correctly

## Troubleshooting

### Missing Metrics

1. Check metric name spelling
2. Verify time range
3. Check permissions
4. Review metric retention

### Log Search Issues

1. Use correct query syntax
2. Check time range
3. Verify log level filter
4. Check retention period

### Alert Not Firing

1. Verify alert is enabled
2. Check condition logic
3. Review metric values
4. Test alert manually

---

**Need help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
