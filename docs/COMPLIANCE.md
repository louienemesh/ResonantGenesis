# Compliance Guide

Complete guide to compliance certifications, audit requirements, and regulatory frameworks for ResonantGenesis.

## Overview

ResonantGenesis is designed to meet enterprise compliance requirements across multiple regulatory frameworks. This guide covers certifications, controls, and best practices for compliant deployments.

## Certifications

### Current Certifications

| Certification | Status | Scope | Report |
|---------------|--------|-------|--------|
| SOC 2 Type II | ✅ Active | Security, Availability | Available on request |
| ISO 27001 | ✅ Active | Information Security | Certificate available |
| GDPR | ✅ Compliant | Data Protection | DPA available |
| CCPA | ✅ Compliant | California Privacy | Privacy policy |
| HIPAA | 🔄 In Progress | Healthcare Data | BAA available |

### Planned Certifications

| Certification | Target Date | Status |
|---------------|-------------|--------|
| FedRAMP | Q4 2026 | Assessment phase |
| PCI DSS | Q3 2026 | Gap analysis |
| HITRUST | Q4 2026 | Planning |

## SOC 2

### Overview

SOC 2 Type II certification covers:

- **Security**: Protection against unauthorized access
- **Availability**: System uptime and performance
- **Confidentiality**: Data protection
- **Processing Integrity**: Accurate data processing
- **Privacy**: Personal information handling

### Controls

| Category | Controls | Status |
|----------|----------|--------|
| Access Control | MFA, RBAC, SSO | ✅ Implemented |
| Encryption | TLS 1.3, AES-256 | ✅ Implemented |
| Monitoring | Logging, alerting | ✅ Implemented |
| Incident Response | IR plan, testing | ✅ Implemented |
| Change Management | CI/CD, approvals | ✅ Implemented |

### Requesting Reports

```python
# Request SOC 2 report (Enterprise only)
request = client.compliance.request_report(
    report_type="soc2_type2",
    purpose="vendor_assessment",
    contact_email="security@company.com"
)
```

## GDPR

### Data Subject Rights

ResonantGenesis supports all GDPR data subject rights:

| Right | Implementation |
|-------|----------------|
| **Access** | Export all personal data |
| **Rectification** | Update personal data |
| **Erasure** | Delete account and data |
| **Portability** | Export in machine-readable format |
| **Restriction** | Pause data processing |
| **Objection** | Opt-out of processing |

### Data Processing

```python
# Export user data (GDPR Article 15)
export = client.compliance.export_data(
    user_id="user_123",
    format="json"
)

# Delete user data (GDPR Article 17)
deletion = client.compliance.delete_data(
    user_id="user_123",
    reason="user_request"
)
```

### Data Processing Agreement

Enterprise customers receive a Data Processing Agreement (DPA) covering:

- Data processing scope
- Sub-processor list
- Security measures
- Breach notification
- Audit rights

## HIPAA

### Business Associate Agreement

For healthcare customers, we provide a Business Associate Agreement (BAA) covering:

- PHI handling requirements
- Security safeguards
- Breach notification (within 60 days)
- Audit and compliance

### PHI Safeguards

| Safeguard | Implementation |
|-----------|----------------|
| Access Controls | Role-based, MFA required |
| Audit Logs | All PHI access logged |
| Encryption | At rest and in transit |
| Data Isolation | Dedicated infrastructure |
| Backup | Encrypted, tested recovery |

### HIPAA-Compliant Configuration

```python
# Enable HIPAA mode (Enterprise only)
client.compliance.enable_hipaa_mode(
    organization_id="org_123",
    baa_signed=True,
    phi_encryption_key="customer_managed_key"
)
```

## CCPA

### Consumer Rights

California Consumer Privacy Act compliance includes:

| Right | Implementation |
|-------|----------------|
| **Know** | Disclose data collected |
| **Delete** | Delete personal information |
| **Opt-Out** | No sale of personal information |
| **Non-Discrimination** | Equal service regardless of rights exercise |

### Do Not Sell

ResonantGenesis does **not** sell personal information. Users can verify this:

```python
# Verify no-sale status
status = client.compliance.get_ccpa_status(user_id="user_123")
print(f"Data sold: {status.data_sold}")  # Always False
```

## Data Residency

### Available Regions

| Region | Location | Data Center |
|--------|----------|-------------|
| US | Virginia | AWS us-east-1 |
| EU | Frankfurt | AWS eu-central-1 |
| APAC | Singapore | AWS ap-southeast-1 |
| UK | London | AWS eu-west-2 |

### Configuring Data Residency

```python
# Set data residency (Enterprise only)
client.compliance.set_data_residency(
    organization_id="org_123",
    region="eu",
    enforce_strict=True  # Block cross-region transfers
)
```

### Cross-Border Transfers

For EU data transfers outside the EU:

- Standard Contractual Clauses (SCCs) included in DPA
- Transfer Impact Assessments available
- Supplementary measures documented

## Audit Logs

### What's Logged

| Event Type | Details Captured |
|------------|------------------|
| Authentication | Login, logout, MFA |
| Authorization | Permission changes |
| Data Access | Read, write, delete |
| Configuration | Settings changes |
| Agent Activity | Executions, tool usage |

### Accessing Audit Logs

```python
# Get audit logs
logs = client.compliance.get_audit_logs(
    start_date="2026-01-01",
    end_date="2026-02-21",
    event_types=["authentication", "data_access"],
    user_id="user_123"  # Optional filter
)

for log in logs:
    print(f"{log.timestamp}: {log.event_type} - {log.details}")
```

### Log Retention

| Tier | Retention | Export |
|------|-----------|--------|
| Free | 7 days | Manual |
| Plus | 30 days | Manual |
| Pro | 90 days | Automated |
| Enterprise | 1 year+ | Automated, SIEM |

### SIEM Integration

```python
# Configure SIEM export (Enterprise only)
client.compliance.configure_siem(
    provider="splunk",
    endpoint="https://splunk.company.com:8088",
    token="splunk_hec_token",
    events=["all"]
)
```

## Security Controls

### Access Control

| Control | Implementation |
|---------|----------------|
| Authentication | Email/password, SSO, MFA |
| Authorization | RBAC with custom roles |
| Session Management | Secure tokens, timeout |
| API Security | API keys, rate limiting |

### Encryption

| Data State | Method |
|------------|--------|
| In Transit | TLS 1.3 |
| At Rest | AES-256 |
| Backups | AES-256, separate keys |
| Secrets | Vault, HSM-backed |

### Network Security

| Control | Implementation |
|---------|----------------|
| Firewall | WAF, DDoS protection |
| Segmentation | VPC isolation |
| Monitoring | IDS/IPS |
| Penetration Testing | Annual, third-party |

## Compliance Reports

### Available Reports

| Report | Frequency | Access |
|--------|-----------|--------|
| SOC 2 Type II | Annual | Enterprise |
| Penetration Test | Annual | Enterprise |
| Vulnerability Scan | Monthly | Pro+ |
| Compliance Summary | Quarterly | All |

### Requesting Reports

```python
# Request compliance report
request = client.compliance.request_report(
    report_type="penetration_test",
    year=2026,
    contact_email="security@company.com"
)

print(f"Request ID: {request.id}")
print(f"ETA: {request.estimated_delivery}")
```

## Vendor Assessment

### Security Questionnaire

We provide completed security questionnaires for:

- SIG Lite
- CAIQ (Cloud Security Alliance)
- Custom questionnaires

### Due Diligence Package

Enterprise customers receive:

- SOC 2 Type II report
- Penetration test summary
- Insurance certificates
- Business continuity plan
- Incident response plan

## Incident Response

### Notification Timeline

| Severity | Internal | Customer | Regulatory |
|----------|----------|----------|------------|
| Critical | 1 hour | 24 hours | Per regulation |
| High | 4 hours | 48 hours | Per regulation |
| Medium | 24 hours | 72 hours | As required |
| Low | 72 hours | Weekly | As required |

### Breach Notification

In case of a data breach:

1. **Detection**: Automated monitoring alerts
2. **Containment**: Immediate isolation
3. **Assessment**: Scope and impact analysis
4. **Notification**: Per regulatory requirements
5. **Remediation**: Root cause fix
6. **Post-mortem**: Lessons learned

## Best Practices

### For Compliance

1. **Enable MFA** - Required for all users
2. **Use SSO** - Centralized access control
3. **Review audit logs** - Regular monitoring
4. **Minimize data** - Collect only what's needed
5. **Encrypt secrets** - Use vault for sensitive data

### For Healthcare (HIPAA)

1. **Sign BAA** - Before processing PHI
2. **Enable HIPAA mode** - Dedicated infrastructure
3. **Train users** - HIPAA awareness
4. **Audit access** - Regular reviews
5. **Incident plan** - Documented procedures

### For Financial (PCI)

1. **Isolate cardholder data** - Separate environments
2. **Encrypt all data** - In transit and at rest
3. **Restrict access** - Need-to-know basis
4. **Monitor continuously** - Real-time alerts
5. **Test regularly** - Vulnerability scans

## API Reference

### Get Compliance Status

```bash
GET /api/v1/compliance/status
```

### Export User Data

```bash
POST /api/v1/compliance/export
```

### Delete User Data

```bash
POST /api/v1/compliance/delete
```

### Get Audit Logs

```bash
GET /api/v1/compliance/audit-logs
```

### Request Report

```bash
POST /api/v1/compliance/reports/request
```

## Contact

- **Compliance Team**: compliance@resonantgenesis.xyz
- **Security Team**: security@resonantgenesis.xyz
- **DPO (GDPR)**: dpo@resonantgenesis.xyz
- **Privacy**: privacy@resonantgenesis.xyz

---

**Need compliance documentation?** Contact compliance@resonantgenesis.xyz or your account manager.
