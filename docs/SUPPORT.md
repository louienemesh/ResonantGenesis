# Support Guide

Complete guide to getting help and support for ResonantGenesis.

## Support Channels

### Community Support (Free)

| Channel | Response Time | Best For |
|---------|---------------|----------|
| **Discord** | Minutes-hours | Quick questions, community help |
| **GitHub Issues** | 1-3 days | Bug reports, feature requests |
| **Stack Overflow** | Varies | Technical questions |
| **Documentation** | Instant | Self-service learning |

### Paid Support

| Tier | Channel | Response Time | Availability |
|------|---------|---------------|--------------|
| Plus | Email | 48 hours | Business hours |
| Pro | Email + Chat | 24 hours | Business hours |
| Enterprise | Dedicated | 4 hours | 24/7 |

## Getting Help

### Discord Community

Join our Discord for real-time help:

**[discord.gg/resonantgenesis](https://discord.gg/resonantgenesis)**

**Channels:**
- `#general` - General discussion
- `#help` - Get help from the community
- `#showcase` - Share what you've built
- `#feature-requests` - Suggest features
- `#bug-reports` - Report issues
- `#announcements` - Platform updates

### GitHub Issues

Report bugs and request features:

**[github.com/louienemesh/ResonantGenesis/issues](https://github.com/louienemesh/ResonantGenesis/issues)**

**Issue Templates:**
- Bug Report
- Feature Request
- Documentation Issue
- Security Vulnerability (private)

### Email Support

For paid tiers:

- **General**: support@resonantgenesis.xyz
- **Billing**: billing@resonantgenesis.xyz
- **Security**: security@resonantgenesis.xyz
- **Enterprise**: enterprise@resonantgenesis.xyz

### Help Center

Self-service knowledge base:

**[resonantgenesis.xyz/help](https://resonantgenesis.xyz/help)**

## Service Level Agreements

### Response Times

| Priority | Free | Plus | Pro | Enterprise |
|----------|------|------|-----|------------|
| Critical | - | 24h | 4h | 1h |
| High | - | 48h | 8h | 4h |
| Medium | - | 72h | 24h | 8h |
| Low | - | 5 days | 48h | 24h |

### Priority Definitions

| Priority | Description | Examples |
|----------|-------------|----------|
| **Critical** | Service down, data loss | API outage, security breach |
| **High** | Major feature broken | Agent execution failing |
| **Medium** | Feature degraded | Slow performance, minor bugs |
| **Low** | Questions, enhancements | How-to questions, feature requests |

### Uptime SLA

| Tier | Uptime | Credits |
|------|--------|---------|
| Free | Best effort | None |
| Plus | 99.5% | None |
| Pro | 99.9% | 10% per 0.1% below |
| Enterprise | 99.99% | Custom |

## Reporting Issues

### Bug Reports

Include the following:

```markdown
## Bug Description
[Clear description of the issue]

## Steps to Reproduce
1. [Step 1]
2. [Step 2]
3. [Step 3]

## Expected Behavior
[What should happen]

## Actual Behavior
[What actually happens]

## Environment
- SDK Version: 
- Python/Node Version:
- OS:

## Additional Context
[Screenshots, logs, etc.]
```

### API Issues

Include request details:

```bash
# Request
curl -X POST https://resonantgenesis.xyz/api/v1/agents \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "Test"}'

# Response
{
  "error": "...",
  "request_id": "req_abc123"
}
```

**Always include the `request_id`** - it helps us trace the issue.

### Security Issues

**Do not report security issues publicly.**

Email: security@resonantgenesis.xyz

Or use our [security advisory form](https://github.com/louienemesh/ResonantGenesis/security/advisories/new).

See [SECURITY.md](../SECURITY.md) for our security policy.

## Status Page

Check system status:

**[status.resonantgenesis.xyz](https://status.resonantgenesis.xyz)**

### Subscribe to Updates

- **Email**: Subscribe on status page
- **RSS**: status.resonantgenesis.xyz/feed
- **Slack/Discord**: Webhook integrations

### Incident History

View past incidents and post-mortems on the status page.

## Escalation Process

### When to Escalate

- Issue not resolved within SLA
- Critical business impact
- Security concerns
- Billing disputes

### How to Escalate

1. **Reference ticket number** in your escalation
2. **Email**: escalations@resonantgenesis.xyz
3. **Enterprise**: Contact your account manager

## Enterprise Support

### Dedicated Support

Enterprise customers receive:

- **Dedicated account manager**
- **24/7 support access**
- **1-hour critical response**
- **Quarterly business reviews**
- **Custom training sessions**

### Technical Account Manager

Your TAM provides:

- Onboarding assistance
- Architecture reviews
- Best practices guidance
- Proactive monitoring
- Feature prioritization

### Contact

- **Sales**: enterprise@resonantgenesis.xyz
- **Support**: enterprise-support@resonantgenesis.xyz

## Training Resources

### Documentation

- [Getting Started](./GETTING_STARTED.md)
- [API Reference](./API_REFERENCE.md)
- [Best Practices](./BEST_PRACTICES.md)
- [Examples](./EXAMPLES.md)

### Video Tutorials

- [YouTube Channel](https://youtube.com/@resonantgenesis)
- [Getting Started Playlist](https://youtube.com/playlist?list=...)
- [Advanced Topics](https://youtube.com/playlist?list=...)

### Webinars

- Monthly product updates
- Technical deep dives
- Customer showcases

Register: [resonantgenesis.xyz/webinars](https://resonantgenesis.xyz/webinars)

### Office Hours

Weekly live Q&A sessions:

- **When**: Thursdays, 2 PM EST
- **Where**: Discord #office-hours
- **Format**: Open Q&A with engineers

## Feedback

### Product Feedback

Share ideas and vote on features:

- **Discord**: #feature-requests
- **GitHub**: Feature request issues
- **Email**: feedback@resonantgenesis.xyz

### Documentation Feedback

Help improve our docs:

- **Edit on GitHub**: Link at bottom of each page
- **Report issues**: Documentation issue template
- **Suggest improvements**: Discord #docs-feedback

## Contact Directory

| Department | Email | Purpose |
|------------|-------|---------|
| Support | support@resonantgenesis.xyz | Technical help |
| Sales | sales@resonantgenesis.xyz | Pricing, demos |
| Billing | billing@resonantgenesis.xyz | Invoices, payments |
| Security | security@resonantgenesis.xyz | Security issues |
| Legal | legal@resonantgenesis.xyz | Contracts, compliance |
| Press | press@resonantgenesis.xyz | Media inquiries |
| Partnerships | partners@resonantgenesis.xyz | Integrations |

## FAQ

**Q: How do I reset my password?**
A: Go to [resonantgenesis.xyz/reset-password](https://resonantgenesis.xyz/reset-password)

**Q: How do I upgrade my plan?**
A: Go to Settings > Billing > Upgrade

**Q: How do I cancel my subscription?**
A: Go to Settings > Billing > Cancel. Access continues until end of billing period.

**Q: How do I get a refund?**
A: Contact billing@resonantgenesis.xyz within 7 days of purchase.

**Q: How do I report a security issue?**
A: Email security@resonantgenesis.xyz. Do not report publicly.

**Q: How do I request a feature?**
A: Submit a GitHub issue or post in Discord #feature-requests.

---

**Need immediate help?** Join our [Discord](https://discord.gg/resonantgenesis) for real-time community support.
