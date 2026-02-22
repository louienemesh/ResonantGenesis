# Data Privacy & Compliance Guide

Complete guide to data handling, privacy policies, and compliance for ResonantGenesis.

## Overview

ResonantGenesis is committed to protecting user data and maintaining compliance with global privacy regulations.

## Data Collection

### What We Collect

| Data Type | Purpose | Retention |
|-----------|---------|-----------|
| Account Info | Authentication | Account lifetime |
| Agent Configs | Service delivery | Until deleted |
| Session Data | Execution history | Per tier policy |
| Usage Metrics | Billing, analytics | 2 years |
| Logs | Debugging, security | Per tier policy |

### Data Sources

- **User-provided** - Account info, agent configurations
- **Generated** - Session outputs, execution logs
- **Derived** - Usage analytics, performance metrics
- **Third-party** - OAuth provider data

## Data Storage

### Infrastructure

- **Primary**: AWS US-East-1 (Virginia)
- **Backup**: AWS US-West-2 (Oregon)
- **EU Region**: AWS EU-West-1 (Ireland) - for EU customers

### Encryption

| Data State | Encryption |
|------------|------------|
| At rest | AES-256 |
| In transit | TLS 1.3 |
| Backups | AES-256 |
| API keys | bcrypt + AES |

### Database Security

- Encrypted connections required
- Network isolation (VPC)
- Regular security audits
- Automated backups

## Data Retention

### Retention by Tier

| Data Type | Free | Plus | Pro | Enterprise |
|-----------|------|------|-----|------------|
| Session logs | 7 days | 30 days | 90 days | Custom |
| Execution history | 30 days | 90 days | 1 year | Custom |
| Analytics | 30 days | 1 year | 2 years | Custom |
| Audit logs | 30 days | 1 year | 2 years | Custom |

### Automatic Deletion

```python
# Data automatically deleted after retention period
# No action required from users

# Check retention settings
retention = client.account.get_retention_settings()
print(f"Session logs: {retention.session_logs_days} days")
print(f"Execution history: {retention.execution_history_days} days")
```

### Manual Deletion

```python
# Delete specific session
client.sessions.delete(session_id)

# Delete all sessions for an agent
client.agents.delete_sessions(agent_id)

# Delete agent and all associated data
client.agents.delete(agent_id, delete_data=True)

# Request full account data deletion
client.account.request_deletion()
```

## GDPR Compliance

### Data Subject Rights

| Right | Implementation |
|-------|----------------|
| Access | Data export API |
| Rectification | Account settings |
| Erasure | Deletion API |
| Portability | Export in JSON |
| Restriction | Pause processing |
| Objection | Opt-out settings |

### Data Export

```python
# Export all personal data
export = client.account.export_data()

# Download as JSON
with open('my_data.json', 'w') as f:
    json.dump(export, f)

# Export includes:
# - Account information
# - Agent configurations
# - Session history
# - Usage data
# - Billing history
```

### Right to Erasure

```python
# Request account deletion
deletion_request = client.account.request_deletion()

print(f"Request ID: {deletion_request.id}")
print(f"Status: {deletion_request.status}")
print(f"Scheduled for: {deletion_request.scheduled_date}")

# Deletion completed within 30 days
# Confirmation email sent upon completion
```

### Data Processing Agreement

Enterprise customers can request a DPA:

1. Contact legal@resonantgenesis.xyz
2. Provide company details
3. Review and sign DPA
4. DPA applies to all data processing

## CCPA Compliance

### California Consumer Rights

- **Know** - What data we collect
- **Delete** - Request data deletion
- **Opt-out** - No sale of personal data
- **Non-discrimination** - Equal service

### Do Not Sell

ResonantGenesis does not sell personal information.

```python
# Verify opt-out status
status = client.account.get_privacy_settings()
print(f"Data sharing: {status.data_sharing}")
print(f"Marketing: {status.marketing_emails}")
```

## SOC 2 Compliance

### Type II Certification

ResonantGenesis maintains SOC 2 Type II certification covering:

- **Security** - Protection against unauthorized access
- **Availability** - System uptime and reliability
- **Confidentiality** - Data protection
- **Processing Integrity** - Accurate processing
- **Privacy** - Personal information handling

### Audit Reports

Enterprise customers can request SOC 2 reports:

1. Contact security@resonantgenesis.xyz
2. Sign NDA
3. Receive latest audit report

## HIPAA Compliance

### Healthcare Data

HIPAA compliance available for Enterprise customers:

- Business Associate Agreement (BAA)
- PHI encryption
- Access controls
- Audit logging
- Incident response

### Enabling HIPAA Mode

```python
# Enterprise only
# Contact sales for BAA

# Once enabled:
client = ResonantClient(
    api_key="...",
    hipaa_mode=True
)

# Additional protections applied:
# - Enhanced encryption
# - Restricted data access
# - Extended audit logs
# - No third-party sharing
```

## Data Processing

### Processing Locations

| Region | Location | Compliance |
|--------|----------|------------|
| US | Virginia, Oregon | SOC 2, CCPA |
| EU | Ireland | GDPR, SOC 2 |
| APAC | Singapore | PDPA |

### Sub-processors

| Provider | Purpose | Location |
|----------|---------|----------|
| AWS | Infrastructure | US, EU |
| OpenAI | AI models | US |
| Anthropic | AI models | US |
| Stripe | Payments | US |
| SendGrid | Email | US |

### Data Residency

```python
# Set data residency (Enterprise)
client.account.set_data_residency(region="eu")

# All data stored in EU region
# Processing limited to EU infrastructure
```

## Privacy Settings

### Managing Preferences

```python
# Get current settings
settings = client.account.get_privacy_settings()

# Update settings
client.account.update_privacy_settings(
    analytics_enabled=False,
    marketing_emails=False,
    third_party_sharing=False
)
```

### Opt-Out Options

| Setting | Description |
|---------|-------------|
| Analytics | Usage analytics collection |
| Marketing | Promotional emails |
| Third-party | Data sharing with partners |
| Telemetry | Product improvement data |

## Security Measures

### Access Controls

- Role-based access control (RBAC)
- Multi-factor authentication
- Session management
- IP allowlisting (Enterprise)

### Monitoring

- Real-time threat detection
- Anomaly detection
- Access logging
- Security alerts

### Incident Response

1. **Detection** - Automated monitoring
2. **Containment** - Isolate affected systems
3. **Investigation** - Root cause analysis
4. **Notification** - User notification within 72 hours
5. **Remediation** - Fix and prevent recurrence

## Audit Logs

### Available Logs

```python
# List audit logs
logs = client.audit.list(
    start_date="2026-02-01",
    end_date="2026-02-21",
    event_types=["login", "agent.created", "session.executed"]
)

for log in logs:
    print(f"{log.timestamp}: {log.event_type} by {log.user_id}")
```

### Log Events

| Event | Description |
|-------|-------------|
| `login` | User login |
| `logout` | User logout |
| `agent.created` | Agent created |
| `agent.deleted` | Agent deleted |
| `session.executed` | Session run |
| `api_key.created` | API key generated |
| `settings.changed` | Settings modified |

## Cookies & Tracking

### Cookie Policy

| Cookie | Purpose | Duration |
|--------|---------|----------|
| `session` | Authentication | Session |
| `preferences` | User settings | 1 year |
| `analytics` | Usage tracking | 1 year |

### Managing Cookies

```javascript
// Cookie consent
ResonantGenesis.setCookieConsent({
  necessary: true,
  analytics: false,
  marketing: false
});
```

## Third-Party Data

### OAuth Data

When connecting OAuth providers:

- Only requested scopes accessed
- Tokens encrypted at rest
- Revocable at any time
- No data sold to third parties

### Revoking Access

```python
# List connected services
connections = client.integrations.list()

# Revoke specific connection
client.integrations.disconnect("github")

# Revoke all connections
client.integrations.disconnect_all()
```

## Children's Privacy

ResonantGenesis is not intended for users under 18. We do not knowingly collect data from minors.

## International Transfers

### Transfer Mechanisms

- Standard Contractual Clauses (SCCs)
- Data Processing Agreements
- Privacy Shield (where applicable)

### EU-US Transfers

Data transferred from EU to US under:
- SCCs with AWS
- DPA with sub-processors

## Contact

### Privacy Inquiries

- **Email**: privacy@resonantgenesis.xyz
- **DPO**: dpo@resonantgenesis.xyz

### Data Requests

Submit requests via:
1. Dashboard: **Settings** > **Privacy** > **Data Request**
2. Email: privacy@resonantgenesis.xyz
3. API: `client.account.submit_privacy_request()`

### Response Times

| Request Type | Response Time |
|--------------|---------------|
| Access | 30 days |
| Deletion | 30 days |
| Rectification | 14 days |
| Portability | 30 days |

---

## Resources

- **Privacy Policy**: [resonantgenesis.xyz/privacy](https://resonantgenesis.xyz/privacy)
- **Terms of Service**: [resonantgenesis.xyz/terms](https://resonantgenesis.xyz/terms)
- **Security**: [resonantgenesis.xyz/security](https://resonantgenesis.xyz/security)
- **Trust Center**: [trust.resonantgenesis.xyz](https://trust.resonantgenesis.xyz)

---

**Questions?** Contact privacy@resonantgenesis.xyz or our Data Protection Officer at dpo@resonantgenesis.xyz.
