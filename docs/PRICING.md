# Pricing & Billing Guide

Complete guide to pricing tiers, billing, and usage limits for ResonantGenesis.

## Pricing Tiers

| Feature | Free | Plus | Pro | Enterprise |
|---------|------|------|-----|------------|
| **Price** | $0/mo | $29/mo | $99/mo | Custom |
| **Agents** | 5 | 25 | Unlimited | Unlimited |
| **Sessions/month** | 100 | 1,000 | 10,000 | Unlimited |
| **API Requests/min** | 60 | 300 | 600 | 1,000+ |
| **Concurrent Sessions** | 1 | 5 | 20 | Custom |
| **Tools** | Basic | All | All | All + Custom |
| **Support** | Community | Email | Priority | Dedicated |

## Tier Details

### Free Tier

**Best for:** Exploring the platform, personal projects, learning

**Includes:**
- 5 agents
- 100 sessions/month
- 60 API requests/minute
- 1 concurrent session
- Basic tools (web search, file access)
- Community support
- 7-day log retention

**Limitations:**
- No code execution tool
- No custom tools
- No team features
- No SLA

### Plus Tier - $29/month

**Best for:** Individual developers, small projects

**Includes:**
- 25 agents
- 1,000 sessions/month
- 300 API requests/minute
- 5 concurrent sessions
- All standard tools
- Email support (48h response)
- 30-day log retention
- Basic analytics

**Additional Features:**
- Code execution tool
- Webhook triggers
- Session streaming
- Export data

### Pro Tier - $99/month

**Best for:** Teams, production applications

**Includes:**
- Unlimited agents
- 10,000 sessions/month
- 600 API requests/minute
- 20 concurrent sessions
- All tools + custom tools
- Priority support (24h response)
- 90-day log retention
- Advanced analytics
- Team collaboration

**Additional Features:**
- Agent Teams
- Custom tool creation
- SSO integration
- Audit logs
- 99.9% SLA

### Enterprise Tier - Custom

**Best for:** Large organizations, high-volume applications

**Includes:**
- Everything in Pro
- Unlimited sessions
- Custom rate limits
- Dedicated support
- Custom log retention
- On-premise option
- Custom integrations
- Training & onboarding

**Additional Features:**
- Dedicated account manager
- Custom SLA (up to 99.99%)
- SOC 2 compliance
- HIPAA compliance (optional)
- Custom contracts

## Usage-Based Pricing

### Token Usage

| Model | Input (per 1M tokens) | Output (per 1M tokens) |
|-------|----------------------|------------------------|
| GPT-3.5 Turbo | $0.50 | $1.50 |
| GPT-4 Turbo | $10.00 | $30.00 |
| GPT-4 | $30.00 | $60.00 |
| Claude 3 Sonnet | $3.00 | $15.00 |
| Claude 3 Opus | $15.00 | $75.00 |

### Included Tokens

| Tier | Monthly Tokens |
|------|----------------|
| Free | 100K |
| Plus | 1M |
| Pro | 10M |
| Enterprise | Custom |

### Overage Pricing

Tokens beyond included amount are billed at model rates above.

```python
# Check token usage
usage = client.billing.get_usage()

print(f"Tokens used: {usage.tokens_used:,}")
print(f"Tokens included: {usage.tokens_included:,}")
print(f"Overage: {usage.tokens_overage:,}")
print(f"Overage cost: ${usage.overage_cost:.2f}")
```

## Billing

### Billing Cycle

- Monthly billing on subscription date
- Usage calculated at end of billing period
- Invoices available in dashboard

### Payment Methods

- Credit/debit cards (Visa, Mastercard, Amex)
- ACH bank transfer (Pro+)
- Wire transfer (Enterprise)
- Annual billing (10% discount)

### Managing Subscription

```python
# Get current subscription
subscription = client.billing.get_subscription()

print(f"Tier: {subscription.tier}")
print(f"Status: {subscription.status}")
print(f"Next billing: {subscription.next_billing_date}")
print(f"Amount: ${subscription.amount}")
```

### Upgrading

```python
# Upgrade to Pro
client.billing.upgrade(tier="pro")

# Changes take effect immediately
# Prorated for current billing period
```

### Downgrading

```python
# Downgrade to Plus
client.billing.downgrade(tier="plus")

# Takes effect at end of billing period
# Features remain until then
```

### Cancellation

```python
# Cancel subscription
client.billing.cancel()

# Access continues until end of billing period
# Data retained for 30 days after cancellation
```

## Usage Limits

### Rate Limits

| Tier | Requests/min | Requests/hour | Daily Limit |
|------|--------------|---------------|-------------|
| Free | 60 | 1,000 | 10,000 |
| Plus | 300 | 10,000 | 100,000 |
| Pro | 600 | 30,000 | 500,000 |
| Enterprise | Custom | Custom | Custom |

### Session Limits

| Tier | Sessions/month | Concurrent | Max Duration |
|------|----------------|------------|--------------|
| Free | 100 | 1 | 5 min |
| Plus | 1,000 | 5 | 15 min |
| Pro | 10,000 | 20 | 30 min |
| Enterprise | Unlimited | Custom | Custom |

### Storage Limits

| Tier | File Storage | Log Retention |
|------|--------------|---------------|
| Free | 100 MB | 7 days |
| Plus | 1 GB | 30 days |
| Pro | 10 GB | 90 days |
| Enterprise | Custom | Custom |

## Monitoring Usage

### Dashboard

View usage in **Settings** > **Billing** > **Usage**

### API

```python
# Get current usage
usage = client.billing.get_usage()

print(f"Sessions: {usage.sessions_used}/{usage.sessions_limit}")
print(f"Tokens: {usage.tokens_used:,}/{usage.tokens_included:,}")
print(f"Storage: {usage.storage_used_mb}/{usage.storage_limit_mb} MB")
```

### Alerts

```python
# Set usage alert
client.alerts.create(
    type="usage",
    metric="sessions",
    threshold=0.8,  # 80% of limit
    channels=["email"]
)
```

## Cost Optimization

### Tips

1. **Choose appropriate models**
   - Use GPT-3.5 for simple tasks
   - Reserve GPT-4 for complex reasoning

2. **Optimize prompts**
   - Shorter system prompts = fewer tokens
   - Be concise in goals

3. **Use caching**
   - Cache repeated queries
   - Use session context wisely

4. **Set limits**
   ```python
   session = agent.execute(
       goal="Task",
       max_tokens=2000,
       max_steps=5
   )
   ```

5. **Monitor usage**
   - Review usage weekly
   - Set alerts at 80%

### Cost Calculator

```python
def estimate_cost(sessions_per_month, avg_tokens_per_session, model="gpt-4-turbo"):
    rates = {
        "gpt-3.5-turbo": {"input": 0.0005, "output": 0.0015},
        "gpt-4-turbo": {"input": 0.01, "output": 0.03},
        "gpt-4": {"input": 0.03, "output": 0.06}
    }
    
    rate = rates[model]
    total_tokens = sessions_per_month * avg_tokens_per_session
    
    # Assume 30% input, 70% output
    input_tokens = total_tokens * 0.3
    output_tokens = total_tokens * 0.7
    
    cost = (input_tokens / 1_000_000 * rate["input"] + 
            output_tokens / 1_000_000 * rate["output"])
    
    return cost

# Example: 1000 sessions, 5000 tokens each
monthly_cost = estimate_cost(1000, 5000, "gpt-4-turbo")
print(f"Estimated monthly cost: ${monthly_cost:.2f}")
```

## Invoices & Receipts

### Viewing Invoices

```python
# List invoices
invoices = client.billing.list_invoices()

for invoice in invoices:
    print(f"{invoice.date}: ${invoice.amount} - {invoice.status}")
```

### Downloading Receipts

```python
# Download receipt
receipt = client.billing.get_receipt(invoice_id)
receipt.download("receipt.pdf")
```

## Tax Information

### Tax Exemption

For tax-exempt organizations:

1. Go to **Settings** > **Billing** > **Tax**
2. Upload exemption certificate
3. Await verification (1-3 business days)

### VAT

- VAT charged for EU customers
- Provide VAT ID for reverse charge
- VAT ID validated automatically

## Enterprise Features

### Volume Discounts

| Annual Commitment | Discount |
|-------------------|----------|
| $10,000+ | 10% |
| $50,000+ | 15% |
| $100,000+ | 20% |
| $500,000+ | Custom |

### Committed Use

Pre-purchase tokens at discounted rates:

| Commitment | Discount |
|------------|----------|
| 100M tokens | 10% |
| 500M tokens | 15% |
| 1B tokens | 20% |

### Custom Contracts

Enterprise customers can negotiate:
- Custom pricing
- Payment terms
- SLA requirements
- Support levels
- Compliance requirements

## FAQ

**Q: Can I change tiers mid-cycle?**
A: Yes. Upgrades are immediate and prorated. Downgrades take effect at the end of the billing period.

**Q: What happens if I exceed limits?**
A: For sessions/requests, you'll receive 429 errors. For tokens, you'll be billed for overage.

**Q: Is there a free trial for paid tiers?**
A: Yes, 14-day free trial for Plus and Pro tiers.

**Q: Can I get a refund?**
A: Refunds available within 7 days of purchase for unused services.

**Q: How do I get Enterprise pricing?**
A: Contact sales@resonantgenesis.xyz for a custom quote.

---

## Contact

- **Sales**: sales@resonantgenesis.xyz
- **Billing Support**: billing@resonantgenesis.xyz
- **Pricing Page**: [resonantgenesis.xyz/pricing](https://resonantgenesis.xyz/pricing)

---

**Ready to upgrade?** Visit [resonantgenesis.xyz/pricing](https://resonantgenesis.xyz/pricing) or contact sales@resonantgenesis.xyz.
