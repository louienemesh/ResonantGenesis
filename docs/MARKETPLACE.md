# Marketplace Guide

Complete guide to the ResonantGenesis agent marketplace - publishing, discovery, and monetization.

## Overview

The Marketplace is where developers publish, discover, and monetize AI agents. It enables:

- **Publishers**: Monetize your agents through sales and rentals
- **Consumers**: Find pre-built agents for common use cases
- **Teams**: Share agents within organizations

## Marketplace Categories

| Category | Description | Examples |
|----------|-------------|----------|
| **Productivity** | Task automation, scheduling | Email assistant, meeting scheduler |
| **Research** | Information gathering, analysis | Web researcher, data analyst |
| **Development** | Code assistance, DevOps | Code reviewer, bug fixer |
| **Support** | Customer service, helpdesk | Ticket triage, FAQ bot |
| **Content** | Writing, editing, translation | Blog writer, translator |
| **Sales** | Lead gen, proposals, CRM | Lead qualifier, proposal writer |
| **Finance** | Analysis, reporting, compliance | Financial analyst, auditor |
| **Custom** | Specialized solutions | Industry-specific agents |

## Browsing the Marketplace

### Search Agents

```python
# Search by keyword
results = client.marketplace.search(
    query="customer support",
    category="support",
    sort_by="downloads",
    limit=20
)

for agent in results:
    print(f"{agent.name} by {agent.author}")
    print(f"  Rating: {agent.rating}/5 ({agent.review_count} reviews)")
    print(f"  Price: {'Free' if agent.price == 0 else f'${agent.price}'}")
```

### Filter Options

```python
# Advanced filtering
results = client.marketplace.search(
    query="data analysis",
    category="research",
    price_range=(0, 50),  # $0-$50
    min_rating=4.0,
    tools=["code_exec"],
    sort_by="rating",
    verified_only=True
)
```

### Get Agent Details

```python
# Get full agent details
listing = client.marketplace.get("mkt_abc123")

print(f"Name: {listing.name}")
print(f"Description: {listing.description}")
print(f"Author: {listing.author.name}")
print(f"Price: ${listing.price}")
print(f"Downloads: {listing.download_count}")
print(f"Rating: {listing.rating}/5")
print(f"Tools: {listing.tools}")
print(f"Model: {listing.model}")
```

## Purchasing Agents

### One-Time Purchase

```python
# Purchase agent
purchase = client.marketplace.purchase(
    listing_id="mkt_abc123",
    payment_method="card_default"
)

print(f"Purchase ID: {purchase.id}")
print(f"Agent ID: {purchase.agent_id}")

# Use the purchased agent
agent = client.agents.get(purchase.agent_id)
result = agent.execute(goal="My task")
```

### Rental

```python
# Rent agent for limited time
rental = client.marketplace.rent(
    listing_id="mkt_abc123",
    duration_days=30,
    payment_method="card_default"
)

print(f"Rental expires: {rental.expires_at}")
```

### Pay-Per-Execution

```python
# Use agent with per-execution pricing
result = client.marketplace.execute(
    listing_id="mkt_abc123",
    goal="Analyze this data",
    context={"data": my_data}
)

print(f"Cost: ${result.cost}")
print(f"Output: {result.output}")
```

## Publishing Agents

### Requirements

Before publishing, ensure:

1. **Trust Tier T3+** - Stake required for publishing
2. **Verified Identity** - DSID registered on Base
3. **Tested Agent** - Thoroughly tested and working
4. **Documentation** - Clear description and usage guide

### Publish Agent

```python
# Publish to marketplace
publication = client.marketplace.publish(
    agent_id="agent_123",
    name="Customer Support Pro",
    description="""
    AI-powered customer support agent that handles:
    - Ticket triage and categorization
    - FAQ responses
    - Escalation to humans
    - Sentiment analysis
    
    Perfect for support teams looking to automate tier-1 support.
    """,
    category="support",
    tags=["support", "customer-service", "automation"],
    pricing={
        "type": "one_time",
        "price": 49.99,
        "currency": "USD"
    },
    screenshots=["url1", "url2"],
    demo_video="youtube_url"
)

print(f"Listing ID: {publication.listing_id}")
print(f"URL: {publication.url}")
```

### Pricing Models

#### One-Time Purchase

```python
pricing = {
    "type": "one_time",
    "price": 99.99,
    "currency": "USD"
}
```

#### Subscription

```python
pricing = {
    "type": "subscription",
    "price": 19.99,
    "interval": "month",
    "currency": "USD"
}
```

#### Pay-Per-Execution

```python
pricing = {
    "type": "per_execution",
    "price": 0.10,
    "currency": "USD"
}
```

#### Rental

```python
pricing = {
    "type": "rental",
    "price_per_day": 5.00,
    "min_days": 7,
    "currency": "USD"
}
```

#### Free

```python
pricing = {
    "type": "free"
}
```

### Listing Details

```python
publication = client.marketplace.publish(
    agent_id="agent_123",
    name="Research Assistant Pro",
    tagline="Your AI research partner",
    description="Full markdown description...",
    category="research",
    tags=["research", "analysis", "web-search"],
    pricing={"type": "one_time", "price": 29.99},
    
    # Media
    icon="https://...",
    screenshots=["url1", "url2", "url3"],
    demo_video="https://youtube.com/...",
    
    # Documentation
    readme="# Usage Guide\n...",
    changelog="## v1.0.0\n- Initial release",
    
    # Support
    support_email="support@example.com",
    documentation_url="https://docs.example.com",
    
    # Visibility
    visibility="public",  # public, unlisted, private
    regions=["US", "EU", "APAC"]
)
```

## Managing Listings

### Update Listing

```python
# Update listing details
client.marketplace.update(
    listing_id="mkt_abc123",
    description="Updated description...",
    pricing={"type": "one_time", "price": 39.99}
)
```

### Pause/Unpause

```python
# Pause listing (stop new sales)
client.marketplace.pause(listing_id="mkt_abc123")

# Resume listing
client.marketplace.unpause(listing_id="mkt_abc123")
```

### Deprecate

```python
# Deprecate listing (existing users keep access)
client.marketplace.deprecate(
    listing_id="mkt_abc123",
    message="This agent is being replaced by v2",
    replacement_id="mkt_xyz789"
)
```

### Delete

```python
# Delete listing (removes from marketplace)
client.marketplace.delete(listing_id="mkt_abc123")
```

## Revenue & Analytics

### View Earnings

```python
# Get earnings summary
earnings = client.marketplace.earnings(
    period="30d"
)

print(f"Total: ${earnings.total}")
print(f"Sales: {earnings.sales_count}")
print(f"Rentals: {earnings.rental_count}")
print(f"Executions: {earnings.execution_count}")
```

### Detailed Analytics

```python
# Get detailed analytics
analytics = client.marketplace.analytics(
    listing_id="mkt_abc123",
    period="30d"
)

print(f"Views: {analytics.views}")
print(f"Conversions: {analytics.conversions}")
print(f"Revenue: ${analytics.revenue}")
print(f"Avg Rating: {analytics.avg_rating}")
```

### Payouts

```python
# Get payout history
payouts = client.marketplace.payouts()

for payout in payouts:
    print(f"{payout.date}: ${payout.amount} - {payout.status}")

# Request payout
client.marketplace.request_payout(
    amount=500.00,
    method="bank_transfer"
)
```

## Reviews & Ratings

### View Reviews

```python
# Get reviews for a listing
reviews = client.marketplace.reviews(
    listing_id="mkt_abc123",
    sort_by="recent"
)

for review in reviews:
    print(f"{review.author}: {review.rating}/5")
    print(f"  {review.comment}")
```

### Respond to Reviews

```python
# Respond to a review (as publisher)
client.marketplace.respond_to_review(
    review_id="rev_123",
    response="Thank you for your feedback!"
)
```

### Leave Review

```python
# Leave review (as buyer)
client.marketplace.create_review(
    listing_id="mkt_abc123",
    rating=5,
    title="Excellent agent!",
    comment="This agent saved me hours of work..."
)
```

## Verification & Trust

### Verified Publishers

Verified publishers have:
- ✅ Identity verified (T2+)
- ✅ Stake deposited (T3+)
- ✅ Track record of quality
- ✅ Responsive support

### Verified Agents

Verified agents have:
- ✅ Security audit passed
- ✅ Performance benchmarks met
- ✅ Documentation complete
- ✅ Active maintenance

### Trust Badges

| Badge | Meaning |
|-------|---------|
| 🔵 Verified | Publisher identity verified |
| ⭐ Top Rated | 4.5+ rating with 50+ reviews |
| 🚀 Popular | 1000+ downloads |
| 🛡️ Audited | Security audit passed |
| 💎 Premium | Premium support included |

## Marketplace Policies

### Content Guidelines

- No malicious or harmful agents
- No agents that violate terms of service
- No misleading descriptions
- No copyright infringement

### Pricing Guidelines

- Fair and transparent pricing
- No hidden fees
- Clear refund policy
- Accurate feature descriptions

### Support Requirements

- Respond to issues within 48 hours
- Maintain agent functionality
- Provide documentation
- Handle refund requests

## Fees & Revenue Share

| Tier | Revenue Share | Platform Fee |
|------|---------------|--------------|
| Standard | 70% | 30% |
| Verified | 80% | 20% |
| Premium | 85% | 15% |

### Fee Breakdown

```
Sale Price: $100
Platform Fee (30%): $30
Payment Processing (3%): $3
Your Revenue: $67
```

## API Reference

### Search Marketplace

```bash
GET /api/v1/marketplace/search?q={query}
```

### Get Listing

```bash
GET /api/v1/marketplace/listings/{listing_id}
```

### Purchase

```bash
POST /api/v1/marketplace/purchase
```

### Publish

```bash
POST /api/v1/marketplace/publish
```

### Update Listing

```bash
PATCH /api/v1/marketplace/listings/{listing_id}
```

### Get Analytics

```bash
GET /api/v1/marketplace/analytics/{listing_id}
```

### Get Earnings

```bash
GET /api/v1/marketplace/earnings
```

## Best Practices

### For Publishers

1. **Quality first** - Test thoroughly before publishing
2. **Clear descriptions** - Explain what your agent does
3. **Good screenshots** - Show the agent in action
4. **Responsive support** - Answer questions quickly
5. **Regular updates** - Keep your agent current

### For Buyers

1. **Read reviews** - Check what others say
2. **Try free tier** - Test before buying
3. **Check tools** - Ensure it has what you need
4. **Verify publisher** - Look for verified badges
5. **Contact support** - Ask questions before buying

---

**Need help?** Contact marketplace@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
