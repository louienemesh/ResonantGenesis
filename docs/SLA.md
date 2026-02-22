# Service Level Agreement (SLA)

Complete guide to Service Level Agreements, uptime guarantees, and support response times for ResonantGenesis.

## Overview

ResonantGenesis is committed to providing reliable, high-performance services. This SLA defines our commitments for service availability, performance, and support response times.

## Service Availability

### Uptime Guarantees

| Tier | Monthly Uptime | Annual Uptime | Credits |
|------|----------------|---------------|---------|
| Free | 95.0% | 95.0% | None |
| Plus | 99.0% | 99.0% | 10% |
| Pro | 99.9% | 99.9% | 25% |
| Enterprise | 99.99% | 99.99% | 100% |

### Uptime Calculation

```
Uptime % = ((Total Minutes - Downtime Minutes) / Total Minutes) × 100

Monthly minutes: 43,200 (30 days)
```

### Downtime Definition

Downtime is defined as:
- API returning 5xx errors for >50% of requests
- API response time >30 seconds for >50% of requests
- Complete service unavailability

### Excluded from Downtime

The following are **not** counted as downtime:
- Scheduled maintenance (with 72-hour notice)
- Force majeure events
- Customer-caused issues
- Third-party service outages
- Beta/preview features

## Service Credits

### Credit Calculation

| Monthly Uptime | Credit Percentage |
|----------------|-------------------|
| < 99.99% | 10% |
| < 99.9% | 25% |
| < 99.0% | 50% |
| < 95.0% | 100% |

### Requesting Credits

```python
# Request SLA credit
credit_request = client.billing.request_sla_credit(
    month="2026-02",
    reason="Service unavailability on Feb 15, 2026",
    affected_services=["api", "sessions"],
    evidence_urls=["https://status.resonantgenesis.xyz/incident/123"]
)

print(f"Request ID: {credit_request.id}")
print(f"Status: {credit_request.status}")
```

### Credit Limitations

- Credits apply to next billing cycle
- Maximum credit: 100% of monthly fee
- Must request within 30 days of incident
- Credits are non-transferable

## Performance SLAs

### API Response Times

| Endpoint Category | P50 | P95 | P99 |
|-------------------|-----|-----|-----|
| Authentication | 100ms | 200ms | 500ms |
| Agent CRUD | 150ms | 300ms | 750ms |
| Session Start | 200ms | 500ms | 1000ms |
| Session Execute | 500ms | 2000ms | 5000ms |
| Webhooks | 100ms | 250ms | 500ms |

### Throughput Guarantees

| Tier | Requests/Second | Concurrent Sessions |
|------|-----------------|---------------------|
| Free | 10 | 5 |
| Plus | 100 | 50 |
| Pro | 1,000 | 500 |
| Enterprise | 10,000+ | 5,000+ |

### Data Durability

| Data Type | Durability | Replication |
|-----------|------------|-------------|
| Agent configurations | 99.999999999% | 3 regions |
| Session data | 99.99999% | 2 regions |
| Logs | 99.999% | 1 region |
| Backups | 99.999999999% | 3 regions |

## Support SLAs

### Response Times

| Priority | Free | Plus | Pro | Enterprise |
|----------|------|------|-----|------------|
| P1 (Critical) | 24h | 4h | 1h | 15min |
| P2 (High) | 48h | 8h | 4h | 1h |
| P3 (Medium) | 72h | 24h | 8h | 4h |
| P4 (Low) | 5 days | 48h | 24h | 8h |

### Priority Definitions

| Priority | Definition | Examples |
|----------|------------|----------|
| **P1** | Service completely unavailable | API down, data loss |
| **P2** | Major feature unavailable | Sessions failing, auth broken |
| **P3** | Feature degraded | Slow performance, intermittent errors |
| **P4** | Minor issue | UI bug, documentation error |

### Resolution Times

| Priority | Target Resolution |
|----------|-------------------|
| P1 | 4 hours |
| P2 | 24 hours |
| P3 | 72 hours |
| P4 | 7 days |

### Support Channels

| Channel | Availability | Response |
|---------|--------------|----------|
| Email | 24/7 | Per SLA |
| Chat | Business hours | 5 min |
| Phone | Enterprise only | Immediate |
| Emergency | 24/7 | 15 min |

## Maintenance Windows

### Scheduled Maintenance

| Type | Frequency | Duration | Notice |
|------|-----------|----------|--------|
| Routine | Weekly | 30 min | 72 hours |
| Major | Monthly | 2 hours | 7 days |
| Emergency | As needed | Varies | Best effort |

### Maintenance Schedule

```python
# Get upcoming maintenance windows
maintenance = client.status.get_maintenance_windows()

for window in maintenance:
    print(f"Type: {window.type}")
    print(f"Start: {window.start_time}")
    print(f"Duration: {window.duration_minutes} minutes")
    print(f"Affected: {window.affected_services}")
```

### Maintenance Notifications

```python
# Subscribe to maintenance notifications
client.notifications.subscribe(
    events=["maintenance.scheduled", "maintenance.started", "maintenance.completed"],
    channels=["email", "slack"]
)
```

## Incident Management

### Incident Severity

| Severity | Impact | Response |
|----------|--------|----------|
| SEV-1 | Complete outage | All hands |
| SEV-2 | Major degradation | On-call team |
| SEV-3 | Partial degradation | Engineering |
| SEV-4 | Minor issue | Normal queue |

### Incident Communication

| Severity | Initial Update | Ongoing Updates | Post-mortem |
|----------|----------------|-----------------|-------------|
| SEV-1 | 15 min | 30 min | 48 hours |
| SEV-2 | 30 min | 1 hour | 5 days |
| SEV-3 | 1 hour | 4 hours | 7 days |
| SEV-4 | 4 hours | Daily | Optional |

### Status Page

Monitor service status at: [status.resonantgenesis.xyz](https://status.resonantgenesis.xyz)

```python
# Get current status
status = client.status.get_current()

print(f"Overall: {status.overall}")  # operational, degraded, outage

for service in status.services:
    print(f"{service.name}: {service.status}")
```

## Data Protection SLAs

### Backup Frequency

| Data Type | Frequency | Retention |
|-----------|-----------|-----------|
| Database | Hourly | 30 days |
| Files | Daily | 90 days |
| Configurations | On change | 1 year |

### Recovery Time Objectives

| Scenario | RTO | RPO |
|----------|-----|-----|
| Single server | 5 min | 0 |
| Availability zone | 15 min | 0 |
| Region | 1 hour | 15 min |
| Data corruption | 4 hours | 1 hour |

### Data Retention

| Data Type | Retention |
|-----------|-----------|
| Active data | Indefinite |
| Deleted data | 30 days |
| Logs | 90 days |
| Backups | 90 days |

## Enterprise SLA

### Custom SLAs

Enterprise customers can negotiate custom SLAs including:

- Higher uptime guarantees (99.999%)
- Faster response times
- Dedicated support team
- Custom maintenance windows
- Enhanced credits

### Dedicated Resources

| Resource | Standard | Enterprise |
|----------|----------|------------|
| Infrastructure | Shared | Dedicated |
| Support | Pooled | Dedicated |
| Account Manager | No | Yes |
| SLA Review | Annual | Quarterly |

### Enterprise Support

```python
# Enterprise support features
enterprise_support = {
    "dedicated_team": True,
    "24x7_phone": True,
    "slack_channel": True,
    "quarterly_reviews": True,
    "custom_integrations": True,
    "training_sessions": 4  # per year
}
```

## Monitoring

### Health Checks

```python
# Check service health
health = client.status.health_check()

print(f"API: {health.api}")
print(f"Database: {health.database}")
print(f"Cache: {health.cache}")
print(f"Queue: {health.queue}")
```

### Metrics

```python
# Get SLA metrics
metrics = client.sla.get_metrics(period="30d")

print(f"Uptime: {metrics.uptime_percentage}%")
print(f"Avg response time: {metrics.avg_response_ms}ms")
print(f"Error rate: {metrics.error_rate}%")
print(f"P99 latency: {metrics.p99_latency_ms}ms")
```

### Alerts

```python
# Configure SLA alerts
client.alerts.create(
    name="SLA Breach Warning",
    condition={
        "metric": "uptime",
        "operator": "less_than",
        "threshold": 99.9,
        "window": "1h"
    },
    actions=["email", "pagerduty"]
)
```

## Reporting

### SLA Reports

```python
# Generate SLA report
report = client.sla.generate_report(
    period="2026-02",
    include_incidents=True,
    include_credits=True
)

print(f"Uptime: {report.uptime}%")
print(f"Incidents: {report.incident_count}")
print(f"Credits earned: ${report.credits_earned}")
```

### Compliance Reports

| Report | Frequency | Availability |
|--------|-----------|--------------|
| Monthly SLA | Monthly | All tiers |
| Incident Summary | Monthly | Plus+ |
| Performance Report | Weekly | Pro+ |
| Custom Report | On demand | Enterprise |

## Terms and Conditions

### SLA Eligibility

To be eligible for SLA credits:

1. Account must be in good standing
2. Payment must be current
3. Issue must be reported within 30 days
4. Customer must not be cause of issue

### SLA Modifications

- SLA may be updated with 30 days notice
- Changes apply to next billing cycle
- Enterprise SLAs require mutual agreement

### Dispute Resolution

1. Submit dispute via support ticket
2. Include evidence and timeline
3. Resolution within 10 business days
4. Escalation to account manager if needed

## Contact

- **Support**: support@resonantgenesis.xyz
- **Enterprise**: enterprise@resonantgenesis.xyz
- **Status**: status.resonantgenesis.xyz
- **Emergency**: +1-XXX-XXX-XXXX

## API Reference

### Get SLA Status

```bash
GET /api/v1/sla/status
```

### Get SLA Metrics

```bash
GET /api/v1/sla/metrics
```

### Request Credit

```bash
POST /api/v1/sla/credits
```

### Get Maintenance Windows

```bash
GET /api/v1/status/maintenance
```

---

**Questions about SLA?** Contact support@resonantgenesis.xyz or your Account Manager.
