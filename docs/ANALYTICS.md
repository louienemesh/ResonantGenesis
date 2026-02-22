# Analytics Guide

Complete guide to platform analytics, usage metrics, and reporting features in ResonantGenesis.

## Overview

ResonantGenesis provides comprehensive analytics to help you understand agent performance, usage patterns, and optimize your AI workflows. This guide covers available metrics, dashboards, and reporting features.

## Analytics Dashboard

### Dashboard Overview

The analytics dashboard provides real-time visibility into:

- **Agent Performance** - Execution success rates, latency, errors
- **Usage Metrics** - API calls, tokens, sessions
- **Cost Analysis** - Spending by agent, model, feature
- **Trends** - Historical patterns and forecasts

### Accessing Analytics

```python
# Get analytics summary
analytics = client.analytics.get_summary(
    period="30d"
)

print(f"Total executions: {analytics.total_executions}")
print(f"Success rate: {analytics.success_rate}%")
print(f"Total tokens: {analytics.total_tokens}")
print(f"Total cost: ${analytics.total_cost}")
```

## Agent Analytics

### Execution Metrics

```python
# Get agent execution metrics
metrics = client.analytics.get_agent_metrics(
    agent_id="agent_123",
    period="7d"
)

print(f"Executions: {metrics.execution_count}")
print(f"Success rate: {metrics.success_rate}%")
print(f"Avg duration: {metrics.avg_duration_ms}ms")
print(f"Avg tokens: {metrics.avg_tokens}")
print(f"Error rate: {metrics.error_rate}%")
```

### Performance Breakdown

| Metric | Description |
|--------|-------------|
| `execution_count` | Total number of executions |
| `success_rate` | Percentage of successful executions |
| `error_rate` | Percentage of failed executions |
| `avg_duration_ms` | Average execution time |
| `p50_duration_ms` | 50th percentile latency |
| `p95_duration_ms` | 95th percentile latency |
| `p99_duration_ms` | 99th percentile latency |
| `avg_tokens` | Average tokens per execution |
| `avg_steps` | Average steps per execution |

### Error Analysis

```python
# Get error breakdown
errors = client.analytics.get_agent_errors(
    agent_id="agent_123",
    period="7d"
)

for error in errors.by_type:
    print(f"{error.type}: {error.count} ({error.percentage}%)")
    
# Common error types:
# - timeout: 45%
# - rate_limit: 30%
# - model_error: 15%
# - tool_error: 10%
```

### Agent Comparison

```python
# Compare multiple agents
comparison = client.analytics.compare_agents(
    agent_ids=["agent_1", "agent_2", "agent_3"],
    metrics=["success_rate", "avg_duration_ms", "cost"],
    period="30d"
)

for agent in comparison.agents:
    print(f"{agent.name}:")
    print(f"  Success: {agent.success_rate}%")
    print(f"  Duration: {agent.avg_duration_ms}ms")
    print(f"  Cost: ${agent.total_cost}")
```

## Session Analytics

### Session Metrics

```python
# Get session metrics
session_metrics = client.analytics.get_session_metrics(
    period="7d"
)

print(f"Total sessions: {session_metrics.total_sessions}")
print(f"Avg session duration: {session_metrics.avg_duration_sec}s")
print(f"Avg steps per session: {session_metrics.avg_steps}")
print(f"Completion rate: {session_metrics.completion_rate}%")
```

### Session Flow Analysis

```python
# Analyze session flows
flow_analysis = client.analytics.get_session_flows(
    agent_id="agent_123",
    period="7d"
)

for flow in flow_analysis.common_flows:
    print(f"Flow: {' -> '.join(flow.steps)}")
    print(f"  Frequency: {flow.count}")
    print(f"  Avg duration: {flow.avg_duration_ms}ms")
```

### Drop-off Analysis

```python
# Analyze where sessions fail
dropoff = client.analytics.get_session_dropoff(
    agent_id="agent_123",
    period="7d"
)

for step in dropoff.steps:
    print(f"Step {step.number}: {step.completion_rate}% continue")
    if step.drop_rate > 10:
        print(f"  WARNING: {step.drop_rate}% drop-off")
```

## Usage Analytics

### API Usage

```python
# Get API usage metrics
api_usage = client.analytics.get_api_usage(
    period="30d"
)

print(f"Total API calls: {api_usage.total_calls}")
print(f"Successful: {api_usage.successful_calls}")
print(f"Failed: {api_usage.failed_calls}")
print(f"Rate limited: {api_usage.rate_limited_calls}")

# By endpoint
for endpoint in api_usage.by_endpoint:
    print(f"{endpoint.path}: {endpoint.call_count}")
```

### Token Usage

```python
# Get token usage
token_usage = client.analytics.get_token_usage(
    period="30d"
)

print(f"Total tokens: {token_usage.total_tokens}")
print(f"Input tokens: {token_usage.input_tokens}")
print(f"Output tokens: {token_usage.output_tokens}")

# By model
for model in token_usage.by_model:
    print(f"{model.name}: {model.tokens} tokens")
```

### Tool Usage

```python
# Get tool usage metrics
tool_usage = client.analytics.get_tool_usage(
    period="30d"
)

for tool in tool_usage.tools:
    print(f"{tool.name}:")
    print(f"  Calls: {tool.call_count}")
    print(f"  Success rate: {tool.success_rate}%")
    print(f"  Avg duration: {tool.avg_duration_ms}ms")
```

## Cost Analytics

### Cost Breakdown

```python
# Get cost breakdown
costs = client.analytics.get_cost_breakdown(
    period="30d"
)

print(f"Total cost: ${costs.total}")
print(f"Model costs: ${costs.model_costs}")
print(f"Tool costs: ${costs.tool_costs}")
print(f"Storage costs: ${costs.storage_costs}")

# By agent
for agent in costs.by_agent:
    print(f"{agent.name}: ${agent.cost}")
```

### Cost Trends

```python
# Get cost trends
trends = client.analytics.get_cost_trends(
    period="90d",
    granularity="daily"
)

for day in trends.data:
    print(f"{day.date}: ${day.cost}")
```

### Cost Optimization

```python
# Get cost optimization recommendations
recommendations = client.analytics.get_cost_recommendations()

for rec in recommendations:
    print(f"{rec.title}")
    print(f"  Potential savings: ${rec.savings}")
    print(f"  Action: {rec.action}")
```

## Team Analytics

### Team Overview

```python
# Get team analytics
team_analytics = client.analytics.get_team_metrics(
    org_id="org_123",
    period="30d"
)

print(f"Total users: {team_analytics.total_users}")
print(f"Active users: {team_analytics.active_users}")
print(f"Total agents: {team_analytics.total_agents}")
print(f"Total executions: {team_analytics.total_executions}")
```

### User Activity

```python
# Get user activity
user_activity = client.analytics.get_user_activity(
    org_id="org_123",
    period="30d"
)

for user in user_activity.users:
    print(f"{user.name}:")
    print(f"  Agents created: {user.agents_created}")
    print(f"  Executions: {user.execution_count}")
    print(f"  Last active: {user.last_active}")
```

## Custom Reports

### Create Report

```python
# Create custom report
report = client.analytics.create_report(
    name="Weekly Performance Report",
    metrics=[
        "execution_count",
        "success_rate",
        "avg_duration_ms",
        "total_cost"
    ],
    filters={
        "agent_tags": ["production"],
        "period": "7d"
    },
    schedule="weekly",
    recipients=["team@company.com"]
)

print(f"Report ID: {report.id}")
```

### Report Templates

| Template | Description |
|----------|-------------|
| `executive_summary` | High-level overview for leadership |
| `performance_report` | Detailed performance metrics |
| `cost_report` | Cost breakdown and trends |
| `error_report` | Error analysis and recommendations |
| `usage_report` | API and token usage details |

### Export Data

```python
# Export analytics data
export = client.analytics.export(
    metrics=["executions", "tokens", "costs"],
    period="30d",
    format="csv"
)

print(f"Download URL: {export.download_url}")
```

## Real-time Analytics

### Live Dashboard

```python
# Subscribe to real-time metrics
async for metrics in client.analytics.subscribe_live():
    print(f"Active sessions: {metrics.active_sessions}")
    print(f"Executions/min: {metrics.executions_per_minute}")
    print(f"Error rate: {metrics.current_error_rate}%")
```

### Alerts

```python
# Create analytics alert
alert = client.analytics.create_alert(
    name="High Error Rate",
    condition={
        "metric": "error_rate",
        "operator": "greater_than",
        "threshold": 10,
        "window": "5m"
    },
    actions={
        "channels": ["email", "slack"],
        "message": "Error rate exceeded 10% in the last 5 minutes"
    }
)
```

## Dashboards

### Create Dashboard

```python
# Create custom dashboard
dashboard = client.analytics.create_dashboard(
    name="Production Monitoring",
    widgets=[
        {
            "type": "metric",
            "title": "Success Rate",
            "metric": "success_rate",
            "period": "24h"
        },
        {
            "type": "chart",
            "title": "Executions Over Time",
            "metric": "execution_count",
            "chart_type": "line",
            "period": "7d"
        },
        {
            "type": "table",
            "title": "Top Agents",
            "metric": "execution_count",
            "group_by": "agent",
            "limit": 10
        }
    ]
)
```

### Widget Types

| Type | Description |
|------|-------------|
| `metric` | Single value display |
| `chart` | Line, bar, or area chart |
| `table` | Tabular data |
| `pie` | Pie chart |
| `heatmap` | Activity heatmap |
| `funnel` | Conversion funnel |

## Integrations

### Export to BI Tools

```python
# Configure BI export
bi_config = client.analytics.configure_bi_export(
    provider="bigquery",
    config={
        "project_id": "my-project",
        "dataset": "resonant_analytics",
        "credentials": "service_account_json"
    },
    schedule="hourly"
)
```

### Supported Integrations

| Integration | Description |
|-------------|-------------|
| BigQuery | Google BigQuery export |
| Snowflake | Snowflake data warehouse |
| Redshift | AWS Redshift |
| Looker | Looker dashboards |
| Tableau | Tableau visualization |
| Grafana | Grafana dashboards |

## Best Practices

### For Monitoring

1. **Set up alerts** - Monitor critical metrics
2. **Review daily** - Check dashboards regularly
3. **Track trends** - Watch for patterns
4. **Compare periods** - Week-over-week analysis
5. **Share insights** - Distribute reports to team

### For Optimization

1. **Identify bottlenecks** - Find slow operations
2. **Reduce errors** - Address common failures
3. **Optimize costs** - Follow recommendations
4. **Right-size models** - Use appropriate models
5. **Cache results** - Reduce redundant calls

### For Reporting

1. **Automate reports** - Schedule regular reports
2. **Customize metrics** - Focus on relevant KPIs
3. **Include context** - Add explanations
4. **Track goals** - Monitor against targets
5. **Archive data** - Keep historical records

## API Reference

### Get Summary

```bash
GET /api/v1/analytics/summary
```

### Get Agent Metrics

```bash
GET /api/v1/analytics/agents/{agent_id}/metrics
```

### Get Usage Metrics

```bash
GET /api/v1/analytics/usage
```

### Get Cost Breakdown

```bash
GET /api/v1/analytics/costs
```

### Create Report

```bash
POST /api/v1/analytics/reports
```

### Create Dashboard

```bash
POST /api/v1/analytics/dashboards
```

### Export Data

```bash
POST /api/v1/analytics/export
```

---

**Need help with analytics?** Contact analytics@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
