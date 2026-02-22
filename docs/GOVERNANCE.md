# Governance Guide

Complete guide to platform governance, voting, and community participation in ResonantGenesis.

## Overview

ResonantGenesis implements a decentralized governance model that allows community members to participate in platform decisions. This guide covers governance structure, voting mechanisms, and how to participate.

## Governance Structure

### Trust Tier Requirements

| Tier | Governance Rights |
|------|-------------------|
| T0-T2 | View proposals, comment |
| T3 | Vote on proposals |
| T4 | Create proposals, vote, delegate |

### Governance Bodies

| Body | Role | Composition |
|------|------|-------------|
| **Core Team** | Platform development | ResonantGenesis team |
| **Council** | Strategic decisions | Elected T4 members |
| **Community** | Voting, proposals | All T3+ members |
| **Working Groups** | Specific initiatives | Volunteer members |

## Proposals

### Proposal Types

| Type | Description | Quorum | Threshold |
|------|-------------|--------|-----------|
| **Protocol** | Core platform changes | 10% | 66% |
| **Feature** | New features | 5% | 51% |
| **Parameter** | Configuration changes | 5% | 51% |
| **Grant** | Funding requests | 5% | 51% |
| **Emergency** | Urgent fixes | 3% | 75% |

### Proposal Lifecycle

```
Draft → Discussion → Voting → Execution → Completed
  ↓         ↓          ↓          ↓
Rejected  Withdrawn  Failed    Cancelled
```

### Creating a Proposal

```python
# Create a proposal (T4 required)
proposal = client.governance.create_proposal(
    title="Add New Model Provider",
    description="""
    ## Summary
    Add support for Anthropic Claude 3.5 as a model provider.
    
    ## Motivation
    Users have requested Claude support for its strong reasoning capabilities.
    
    ## Specification
    - Add Claude 3.5 Sonnet and Opus models
    - Implement API integration
    - Add to model selection UI
    
    ## Timeline
    - Implementation: 2 weeks
    - Testing: 1 week
    - Rollout: 1 week
    """,
    type="feature",
    discussion_period_days=7,
    voting_period_days=5
)

print(f"Proposal ID: {proposal.id}")
print(f"Status: {proposal.status}")
```

### Proposal Requirements

| Requirement | Value |
|-------------|-------|
| Minimum stake | 1000 tokens |
| Discussion period | 3-14 days |
| Voting period | 3-7 days |
| Execution delay | 48 hours |

## Voting

### Voting Power

Voting power is determined by:

1. **Stake amount** - Tokens staked
2. **Stake duration** - Longer = more power
3. **Trust tier** - Higher tier = multiplier
4. **Reputation** - Past participation

```python
# Calculate voting power
power = client.governance.get_voting_power(user_id="user_123")

print(f"Base power: {power.base}")
print(f"Duration bonus: {power.duration_bonus}")
print(f"Tier multiplier: {power.tier_multiplier}")
print(f"Total power: {power.total}")
```

### Casting a Vote

```python
# Vote on a proposal
vote = client.governance.vote(
    proposal_id="prop_123",
    choice="for",  # for, against, abstain
    reason="I support this feature because..."
)

print(f"Vote recorded: {vote.id}")
print(f"Power used: {vote.power}")
```

### Vote Delegation

```python
# Delegate voting power
delegation = client.governance.delegate(
    delegate_to="user_456",
    power_percentage=100,  # Delegate all power
    proposal_types=["feature", "parameter"],  # Optional filter
    expiration="2026-12-31"
)

# Revoke delegation
client.governance.revoke_delegation(delegation_id=delegation.id)
```

### Voting Results

```python
# Get proposal results
results = client.governance.get_results(proposal_id="prop_123")

print(f"For: {results.for_votes} ({results.for_percentage}%)")
print(f"Against: {results.against_votes} ({results.against_percentage}%)")
print(f"Abstain: {results.abstain_votes}")
print(f"Quorum reached: {results.quorum_reached}")
print(f"Passed: {results.passed}")
```

## Staking

### Stake Tokens

```python
# Stake tokens for governance
stake = client.governance.stake(
    amount=1000,
    lock_period_days=90  # Longer lock = more power
)

print(f"Staked: {stake.amount}")
print(f"Lock until: {stake.unlock_date}")
print(f"Voting power: {stake.voting_power}")
```

### Staking Tiers

| Lock Period | Power Multiplier |
|-------------|------------------|
| 30 days | 1.0x |
| 90 days | 1.5x |
| 180 days | 2.0x |
| 365 days | 3.0x |

### Unstaking

```python
# Unstake tokens (after lock period)
unstake = client.governance.unstake(
    stake_id="stake_123"
)

print(f"Unstaked: {unstake.amount}")
print(f"Available: {unstake.available_date}")
```

## Council

### Council Elections

```python
# View current council
council = client.governance.get_council()

for member in council.members:
    print(f"{member.name}")
    print(f"  Term: {member.term_start} - {member.term_end}")
    print(f"  Votes received: {member.votes}")
```

### Running for Council

```python
# Submit candidacy (T4 required)
candidacy = client.governance.submit_candidacy(
    statement="""
    ## Platform
    I will focus on:
    - Improving developer experience
    - Expanding model support
    - Strengthening security
    
    ## Experience
    - 5 years in AI/ML
    - Active community member since launch
    - Contributed 10+ proposals
    """,
    term="2026-Q2"
)
```

### Council Responsibilities

| Responsibility | Description |
|----------------|-------------|
| Strategic direction | Long-term platform vision |
| Emergency decisions | Urgent security issues |
| Grant approval | Large funding requests |
| Dispute resolution | Community conflicts |
| Partnership review | Major integrations |

## Working Groups

### Active Working Groups

| Group | Focus | Lead |
|-------|-------|------|
| Security | Platform security | @security-lead |
| Developer Experience | SDK, docs, tools | @dx-lead |
| Model Integration | New model support | @models-lead |
| Community | Events, support | @community-lead |

### Joining a Working Group

```python
# Apply to join a working group
application = client.governance.apply_working_group(
    group="developer-experience",
    statement="I want to contribute to SDK improvements...",
    skills=["python", "documentation", "api-design"]
)
```

### Creating a Working Group

```python
# Propose new working group (requires proposal)
proposal = client.governance.create_proposal(
    title="Create AI Safety Working Group",
    type="governance",
    description="""
    ## Purpose
    Focus on AI safety and alignment in agent development.
    
    ## Scope
    - Safety guidelines
    - Content filtering
    - Alignment research
    
    ## Resources
    - Budget: $10,000/quarter
    - Members: 5-10
    """
)
```

## Grants

### Grant Categories

| Category | Max Amount | Approval |
|----------|------------|----------|
| Micro | $1,000 | Working group |
| Small | $10,000 | Council |
| Medium | $50,000 | Community vote |
| Large | $100,000+ | Community vote |

### Applying for a Grant

```python
# Submit grant application
grant = client.governance.apply_grant(
    title="SDK Performance Optimization",
    category="small",
    amount=8000,
    description="""
    ## Objective
    Improve SDK performance by 50%.
    
    ## Deliverables
    - Benchmark suite
    - Optimized code
    - Documentation
    
    ## Timeline
    - Week 1-2: Benchmarking
    - Week 3-4: Optimization
    - Week 5: Documentation
    
    ## Budget
    - Development: $6,000
    - Testing: $1,500
    - Documentation: $500
    """,
    milestones=[
        {"name": "Benchmarks", "amount": 2000},
        {"name": "Optimization", "amount": 4000},
        {"name": "Documentation", "amount": 2000}
    ]
)
```

### Grant Reporting

```python
# Submit milestone report
report = client.governance.submit_milestone_report(
    grant_id="grant_123",
    milestone_id="milestone_1",
    report="""
    ## Completed Work
    - Created benchmark suite with 50 tests
    - Identified 5 performance bottlenecks
    
    ## Metrics
    - Test coverage: 95%
    - Baseline established
    
    ## Next Steps
    - Begin optimization phase
    """,
    evidence_urls=["https://github.com/..."]
)
```

## Community Participation

### Discussion Forums

| Forum | Purpose |
|-------|---------|
| General | Platform discussion |
| Proposals | Proposal discussion |
| Technical | Technical questions |
| Feedback | Feature requests |

### Reputation System

| Action | Points |
|--------|--------|
| Proposal created | +50 |
| Proposal passed | +100 |
| Vote cast | +5 |
| Comment upvoted | +2 |
| Grant completed | +200 |

### Badges

| Badge | Requirement |
|-------|-------------|
| Voter | Cast 10 votes |
| Proposer | Create 1 proposal |
| Council | Elected to council |
| Contributor | Complete 1 grant |
| Pioneer | Early adopter |

## Governance API

### List Proposals

```python
# List active proposals
proposals = client.governance.list_proposals(
    status="voting",
    type="feature"
)

for prop in proposals:
    print(f"{prop.title} - {prop.status}")
```

### Get Proposal

```python
# Get proposal details
proposal = client.governance.get_proposal(proposal_id="prop_123")

print(f"Title: {proposal.title}")
print(f"Status: {proposal.status}")
print(f"Votes: {proposal.vote_count}")
```

### Subscribe to Updates

```python
# Subscribe to governance updates
async for event in client.governance.subscribe():
    if event.type == "proposal.created":
        print(f"New proposal: {event.data.title}")
    elif event.type == "vote.cast":
        print(f"Vote on {event.data.proposal_id}")
```

## Best Practices

### For Proposers

1. **Research first** - Check existing proposals
2. **Discuss early** - Get community feedback
3. **Be specific** - Clear specifications
4. **Set realistic timelines** - Achievable goals
5. **Engage actively** - Respond to comments

### For Voters

1. **Read proposals** - Understand what you're voting on
2. **Consider impact** - Long-term effects
3. **Participate in discussion** - Share your views
4. **Vote consistently** - Align with your values
5. **Delegate wisely** - Choose trusted delegates

### For Council Members

1. **Represent community** - Not personal interests
2. **Stay informed** - Follow all proposals
3. **Be transparent** - Explain decisions
4. **Respond promptly** - Emergency situations
5. **Collaborate** - Work with other members

## API Reference

### List Proposals

```bash
GET /api/v1/governance/proposals
```

### Create Proposal

```bash
POST /api/v1/governance/proposals
```

### Vote

```bash
POST /api/v1/governance/proposals/{id}/vote
```

### Delegate

```bash
POST /api/v1/governance/delegate
```

### Stake

```bash
POST /api/v1/governance/stake
```

### Get Voting Power

```bash
GET /api/v1/governance/voting-power
```

---

**Questions about governance?** Contact governance@resonantgenesis.xyz or join the #governance channel on [Discord](https://discord.gg/resonantgenesis).
