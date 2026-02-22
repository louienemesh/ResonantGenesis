# Trust Tiers Documentation

Complete guide to the ResonantGenesis trust tier system for agent verification and reputation.

## Overview

Trust Tiers are a reputation-based system that determines what actions agents and users can perform on the platform. Higher tiers unlock more capabilities and indicate greater trustworthiness.

## Tier Levels

| Tier | Name | Requirements | Capabilities |
|------|------|--------------|--------------|
| T0 | Unverified | New account | Read-only, limited API |
| T1 | Verified | Email verified | Basic agent execution |
| T2 | Trusted | Identity verified | Full agent access |
| T3 | Publisher | Staked identity | Network publishing |
| T4 | Governance | Community elected | Protocol voting |

## Tier Details

### T0 - Unverified

**Requirements:**
- Create an account

**Capabilities:**
- Browse marketplace
- View public agents
- Read documentation
- Limited API access (10 requests/hour)

**Limitations:**
- Cannot create agents
- Cannot execute agents
- Cannot publish to network
- Cannot access premium features

### T1 - Verified

**Requirements:**
- Verify email address
- Accept terms of service

**Capabilities:**
- Create up to 5 agents
- Execute agents (60 requests/min)
- Use basic tools (web search, file access)
- Access Help Center
- Join community Discord

**Limitations:**
- Cannot publish to decentralized network
- Cannot use advanced tools (code execution)
- Limited to 1 concurrent session

### T2 - Trusted

**Requirements:**
- Complete identity verification (KYC)
- Active account for 30+ days
- No policy violations

**Capabilities:**
- Create unlimited agents
- Execute agents (300 requests/min)
- Use all tools including code execution
- Create agent teams
- Access analytics dashboard
- 5 concurrent sessions

**Verification Process:**
1. Go to **Settings** > **Verification**
2. Submit government ID
3. Complete liveness check
4. Wait for review (1-3 business days)

### T3 - Publisher

**Requirements:**
- T2 status
- Stake minimum 0.1 ETH
- Pass security audit (for agents)
- Sign publisher agreement

**Capabilities:**
- Publish agents to decentralized network
- Earn from agent executions
- Rent agents to other users
- Access publisher analytics
- Priority support
- 20 concurrent sessions

**Staking:**
```python
# Stake to become publisher
stake_tx = client.trust.stake(
    amount_eth=0.1,
    duration_days=365
)

# Check stake status
stake = client.trust.get_stake()
print(f"Staked: {stake.amount} ETH")
print(f"Unlocks: {stake.unlock_date}")
```

### T4 - Governance

**Requirements:**
- T3 status for 6+ months
- Community nomination
- Governance election
- Significant platform contribution

**Capabilities:**
- Vote on protocol changes
- Propose governance actions
- Access governance dashboard
- Moderate community content
- Early access to features
- Unlimited concurrent sessions

## Trust Score

In addition to tiers, each user and agent has a trust score (0-100).

### Score Components

| Component | Weight | Description |
|-----------|--------|-------------|
| Account Age | 10% | Time since registration |
| Verification | 20% | Identity verification level |
| Activity | 15% | Platform engagement |
| Reputation | 25% | Community ratings |
| Compliance | 20% | Policy adherence |
| Stake | 10% | Amount staked |

### Viewing Trust Score

```python
# Get your trust score
trust = client.trust.get_score()

print(f"Tier: T{trust.tier}")
print(f"Score: {trust.score}/100")
print(f"Components: {trust.breakdown}")
```

### Agent Trust Score

Agents also have trust scores based on:

| Component | Weight | Description |
|-----------|--------|-------------|
| Owner Trust | 30% | Owner's trust tier |
| Execution History | 25% | Success rate |
| User Ratings | 20% | Community feedback |
| Security Audit | 15% | Audit status |
| Age | 10% | Time since creation |

```python
# Get agent trust score
agent_trust = client.agents.get(agent_id).trust

print(f"Agent Trust: {agent_trust.score}/100")
print(f"Executions: {agent_trust.total_executions}")
print(f"Success Rate: {agent_trust.success_rate}%")
```

## Verification Process

### Email Verification (T0 → T1)

```bash
# Request verification email
curl -X POST https://resonantgenesis.xyz/api/v1/auth/verify-email \
  -H "Authorization: Bearer $TOKEN"

# Confirm with code
curl -X POST https://resonantgenesis.xyz/api/v1/auth/confirm-email \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"code": "123456"}'
```

### Identity Verification (T1 → T2)

1. **Document Upload**
   - Government-issued ID (passport, driver's license)
   - Proof of address (utility bill, bank statement)

2. **Liveness Check**
   - Video selfie verification
   - Real-time face matching

3. **Review**
   - Automated checks (1-24 hours)
   - Manual review if needed (1-3 days)

### Publisher Verification (T2 → T3)

1. **Stake ETH**
   ```python
   client.trust.stake(amount_eth=0.1)
   ```

2. **Agent Audit** (for publishing)
   - Submit agent for security review
   - Pass automated vulnerability scan
   - Manual code review (if required)

3. **Agreement**
   - Sign publisher terms
   - Accept revenue share terms

## Trust Violations

### Violation Types

| Violation | Severity | Consequence |
|-----------|----------|-------------|
| Spam | Low | Warning, rate limit |
| Policy breach | Medium | Temporary suspension |
| Security issue | High | Account review |
| Fraud | Critical | Permanent ban |

### Appeal Process

1. Go to **Settings** > **Trust** > **Appeals**
2. Submit appeal with explanation
3. Provide supporting evidence
4. Wait for review (5-10 business days)

## API Reference

### Get Trust Status

```bash
curl https://resonantgenesis.xyz/api/v1/trust/status \
  -H "Authorization: Bearer $TOKEN"
```

**Response:**
```json
{
  "tier": 2,
  "tier_name": "Trusted",
  "score": 85,
  "breakdown": {
    "account_age": 10,
    "verification": 20,
    "activity": 12,
    "reputation": 23,
    "compliance": 20,
    "stake": 0
  },
  "next_tier": {
    "tier": 3,
    "requirements": [
      "Stake 0.1 ETH",
      "Pass security audit"
    ]
  }
}
```

### Request Verification

```bash
curl -X POST https://resonantgenesis.xyz/api/v1/trust/verify \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "identity",
    "documents": {
      "id_front": "base64...",
      "id_back": "base64...",
      "selfie": "base64..."
    }
  }'
```

### Stake for Publisher

```bash
curl -X POST https://resonantgenesis.xyz/api/v1/trust/stake \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "amount_eth": "0.1",
    "duration_days": 365
  }'
```

### Get Agent Trust

```bash
curl https://resonantgenesis.xyz/api/v1/agents/{agent_id}/trust \
  -H "Authorization: Bearer $TOKEN"
```

**Response:**
```json
{
  "score": 92,
  "tier": 3,
  "total_executions": 1500,
  "success_rate": 98.5,
  "average_rating": 4.8,
  "audit_status": "passed",
  "created_at": "2025-11-01T00:00:00Z"
}
```

## Best Practices

### Building Trust

1. **Complete verification early**: Higher tiers unlock more features
2. **Maintain good standing**: Avoid policy violations
3. **Engage with community**: Ratings improve trust score
4. **Publish quality agents**: Good execution history matters
5. **Stake when ready**: Shows commitment to platform

### For Publishers

1. **Audit agents before publishing**: Catch issues early
2. **Monitor execution metrics**: Address failures quickly
3. **Respond to user feedback**: Improve ratings
4. **Keep agents updated**: Maintain compatibility
5. **Document thoroughly**: Help users succeed

### Security

1. **Protect your account**: Use strong passwords, 2FA
2. **Review agent permissions**: Limit tool access
3. **Monitor for abuse**: Report suspicious activity
4. **Keep stake secure**: Use hardware wallet

## Troubleshooting

### Verification Rejected

1. Ensure documents are clear and readable
2. Check document is not expired
3. Verify name matches account
4. Try different document type
5. Contact support if issues persist

### Trust Score Dropped

1. Check for policy violations
2. Review recent agent failures
3. Check for negative ratings
4. Verify account activity
5. Appeal if incorrect

### Stake Issues

1. Verify wallet connection
2. Check ETH balance
3. Confirm network (Base)
4. Review transaction status
5. Contact support for stuck transactions

---

## FAQ

**Q: How long does verification take?**
A: Email verification is instant. Identity verification takes 1-3 business days.

**Q: Can I lose my tier?**
A: Yes, policy violations or inactivity can result in tier reduction.

**Q: Is staking refundable?**
A: Yes, after the lock period ends. Early withdrawal incurs a penalty.

**Q: How are trust scores calculated?**
A: Scores are calculated daily based on the weighted components listed above.

**Q: Can I appeal a trust decision?**
A: Yes, through Settings > Trust > Appeals.

---

**Need help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
