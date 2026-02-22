# Onboarding Guide

Complete guide to onboarding new users, teams, and enterprises to ResonantGenesis.

## Overview

This guide covers the onboarding process for individual users, teams, and enterprise organizations. Follow these steps to get started quickly and effectively.

## Individual User Onboarding

### Step 1: Create Account

```python
# Sign up via API
response = requests.post(
    "https://api.resonantgenesis.xyz/v1/auth/signup",
    json={
        "email": "user@example.com",
        "password": "secure_password",
        "name": "John Doe"
    }
)

# Verify email
# Check inbox for verification link
```

### Step 2: Complete Profile

1. **Verify email** - Click link in verification email
2. **Set up profile** - Add name, avatar, timezone
3. **Enable 2FA** - Recommended for security
4. **Generate API key** - For programmatic access

### Step 3: Install SDK

```bash
# Python
pip install resonantgenesis

# JavaScript
npm install @resonantgenesis/sdk
```

### Step 4: Create First Agent

```python
from resonantgenesis import ResonantGenesis

client = ResonantGenesis(api_key="your_api_key")

# Create your first agent
agent = client.agents.create(
    name="My First Agent",
    system_prompt="You are a helpful assistant.",
    model="gpt-4-turbo"
)

# Run the agent
result = agent.execute(goal="Say hello!")
print(result.output)
```

### Step 5: Explore Features

| Feature | Description | Link |
|---------|-------------|------|
| Agent Studio | Visual agent builder | Dashboard |
| Templates | Pre-built agents | Marketplace |
| Tools | Add capabilities | Settings |
| Webhooks | Automation | Integrations |

## Team Onboarding

### Step 1: Create Organization

```python
# Create organization
org = client.organizations.create(
    name="Acme Corp",
    plan="pro"
)

print(f"Organization ID: {org.id}")
```

### Step 2: Invite Team Members

```python
# Invite team members
invites = client.organizations.invite_members(
    org_id=org.id,
    emails=[
        "alice@acme.com",
        "bob@acme.com",
        "carol@acme.com"
    ],
    role="member"
)

for invite in invites:
    print(f"Invited: {invite.email}")
```

### Step 3: Set Up Roles

| Role | Permissions |
|------|-------------|
| **Owner** | Full access, billing, delete org |
| **Admin** | Manage members, settings, agents |
| **Member** | Create/edit own agents |
| **Viewer** | View only access |

```python
# Assign roles
client.organizations.set_member_role(
    org_id=org.id,
    user_id="user_123",
    role="admin"
)
```

### Step 4: Configure Settings

```python
# Configure organization settings
client.organizations.update_settings(
    org_id=org.id,
    settings={
        "default_model": "gpt-4-turbo",
        "require_2fa": True,
        "allowed_tools": ["web_search", "code_exec"],
        "max_agents_per_user": 10
    }
)
```

### Step 5: Set Up Shared Resources

```python
# Create shared agent
shared_agent = client.agents.create(
    name="Team Support Bot",
    system_prompt="You are the team support assistant.",
    visibility="organization"
)

# Create shared template
template = client.templates.create(
    name="Team Standard Template",
    agent_id=shared_agent.id,
    visibility="organization"
)
```

## Enterprise Onboarding

### Phase 1: Planning (Week 1)

| Task | Owner | Duration |
|------|-------|----------|
| Requirements gathering | Account Manager | 2 days |
| Architecture review | Solutions Architect | 1 day |
| Security assessment | Security Team | 2 days |
| Contract finalization | Legal | 2 days |

### Phase 2: Setup (Week 2)

#### SSO Configuration

```python
# Configure SAML SSO
sso_config = client.enterprise.configure_sso(
    org_id=org.id,
    provider="okta",
    config={
        "idp_entity_id": "https://okta.com/...",
        "idp_sso_url": "https://okta.com/sso",
        "idp_certificate": "-----BEGIN CERTIFICATE-----...",
        "attribute_mapping": {
            "email": "user.email",
            "name": "user.name",
            "groups": "user.groups"
        }
    }
)
```

#### SCIM Provisioning

```python
# Enable SCIM provisioning
scim_config = client.enterprise.configure_scim(
    org_id=org.id,
    config={
        "enabled": True,
        "auto_provision": True,
        "auto_deprovision": True,
        "default_role": "member"
    }
)

print(f"SCIM endpoint: {scim_config.endpoint}")
print(f"SCIM token: {scim_config.token}")
```

#### Data Residency

```python
# Configure data residency
client.enterprise.set_data_residency(
    org_id=org.id,
    region="eu",
    enforce_strict=True
)
```

### Phase 3: Integration (Week 3)

#### API Integration

```python
# Generate enterprise API key
api_key = client.enterprise.create_api_key(
    org_id=org.id,
    name="Production API Key",
    permissions=["agents:*", "sessions:*"],
    ip_allowlist=["10.0.0.0/8"]
)
```

#### Webhook Setup

```python
# Configure enterprise webhooks
webhook = client.webhooks.create(
    url="https://enterprise.example.com/webhooks",
    events=["agent.*", "session.*", "user.*"],
    secret="webhook_secret",
    headers={"X-Enterprise-Key": "key123"}
)
```

#### Audit Log Export

```python
# Configure audit log export
client.enterprise.configure_audit_export(
    org_id=org.id,
    destination="s3",
    config={
        "bucket": "enterprise-audit-logs",
        "prefix": "resonant/",
        "format": "json"
    }
)
```

### Phase 4: Training (Week 4)

| Session | Audience | Duration |
|---------|----------|----------|
| Platform Overview | All users | 1 hour |
| Admin Training | Admins | 2 hours |
| Developer Workshop | Developers | 4 hours |
| Security Review | Security team | 2 hours |

### Phase 5: Go-Live

#### Pre-Launch Checklist

```markdown
- [ ] SSO configured and tested
- [ ] SCIM provisioning verified
- [ ] API integration complete
- [ ] Webhooks configured
- [ ] Audit logging enabled
- [ ] Security review passed
- [ ] User training completed
- [ ] Support escalation path defined
- [ ] Rollback plan documented
```

#### Launch Day

1. **Enable production access**
2. **Monitor error rates**
3. **Support team on standby**
4. **Communicate to users**

## Onboarding Checklist

### Individual Users

```markdown
## Day 1
- [ ] Create account
- [ ] Verify email
- [ ] Complete profile
- [ ] Enable 2FA
- [ ] Generate API key

## Day 2
- [ ] Install SDK
- [ ] Create first agent
- [ ] Run first session
- [ ] Explore templates

## Week 1
- [ ] Try different tools
- [ ] Set up webhooks
- [ ] Join community Discord
- [ ] Complete tutorials
```

### Teams

```markdown
## Week 1
- [ ] Create organization
- [ ] Invite team members
- [ ] Assign roles
- [ ] Configure settings

## Week 2
- [ ] Create shared agents
- [ ] Set up templates
- [ ] Configure integrations
- [ ] Team training

## Week 3
- [ ] Production deployment
- [ ] Monitor usage
- [ ] Gather feedback
- [ ] Optimize workflows
```

### Enterprise

```markdown
## Month 1
- [ ] Requirements and planning
- [ ] Security assessment
- [ ] Contract signed
- [ ] SSO configured

## Month 2
- [ ] SCIM provisioning
- [ ] API integration
- [ ] Webhook setup
- [ ] Audit logging

## Month 3
- [ ] User training
- [ ] Pilot deployment
- [ ] Go-live
- [ ] Optimization
```

## Training Resources

### Documentation

| Resource | Description |
|----------|-------------|
| [Getting Started](./GETTING_STARTED.md) | Quick start guide |
| [API Reference](./API_REFERENCE.md) | Complete API docs |
| [SDK Guide](./SDK_GUIDE.md) | Python/JS SDK |
| [Best Practices](./BEST_PRACTICES.md) | Production tips |

### Video Tutorials

| Tutorial | Duration | Level |
|----------|----------|-------|
| Platform Overview | 15 min | Beginner |
| Creating Agents | 20 min | Beginner |
| Advanced Tools | 30 min | Intermediate |
| Team Management | 25 min | Admin |
| Enterprise Setup | 45 min | Enterprise |

### Live Training

| Session | Frequency | Registration |
|---------|-----------|--------------|
| Getting Started | Weekly | Open |
| Developer Workshop | Monthly | Open |
| Admin Training | Monthly | By request |
| Enterprise Onboarding | Custom | Account Manager |

## Support During Onboarding

### Support Channels

| Channel | Response Time | Availability |
|---------|---------------|--------------|
| Documentation | Instant | 24/7 |
| Community Discord | < 1 hour | 24/7 |
| Email Support | < 24 hours | Business hours |
| Live Chat | < 5 min | Business hours |
| Phone Support | Immediate | Enterprise only |

### Dedicated Onboarding Support

Enterprise customers receive:

- **Dedicated Account Manager**
- **Solutions Architect**
- **Technical Support Engineer**
- **Custom training sessions**
- **Weekly check-ins**

## Common Onboarding Issues

### Account Issues

| Issue | Solution |
|-------|----------|
| Verification email not received | Check spam, resend |
| Password reset not working | Contact support |
| 2FA locked out | Use backup codes |

### Integration Issues

| Issue | Solution |
|-------|----------|
| API key not working | Check permissions, regenerate |
| SSO not redirecting | Verify IdP configuration |
| Webhook not receiving | Check URL, verify signature |

### Agent Issues

| Issue | Solution |
|-------|----------|
| Agent not responding | Check model, verify API key |
| Tools not working | Check trust tier, permissions |
| Session timeout | Increase timeout, optimize prompts |

## Success Metrics

### Individual Users

| Metric | Target | Timeframe |
|--------|--------|-----------|
| First agent created | 100% | Day 1 |
| First session run | 100% | Day 1 |
| 5+ agents created | 50% | Week 1 |
| Webhook configured | 30% | Week 2 |

### Teams

| Metric | Target | Timeframe |
|--------|--------|-----------|
| All members onboarded | 100% | Week 1 |
| Shared agents created | 80% | Week 2 |
| Production deployment | 100% | Week 3 |
| Active daily usage | 70% | Month 1 |

### Enterprise

| Metric | Target | Timeframe |
|--------|--------|-----------|
| SSO configured | 100% | Week 2 |
| Training completed | 90% | Week 4 |
| Go-live achieved | 100% | Month 1 |
| User adoption | 80% | Month 3 |

## API Reference

### Create Organization

```bash
POST /api/v1/organizations
```

### Invite Members

```bash
POST /api/v1/organizations/{org_id}/invites
```

### Configure SSO

```bash
POST /api/v1/enterprise/sso
```

### Configure SCIM

```bash
POST /api/v1/enterprise/scim
```

---

**Need onboarding help?** Contact onboarding@resonantgenesis.xyz or your Account Manager.
