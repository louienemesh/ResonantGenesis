# Frequently Asked Questions

Common questions about ResonantGenesis, answered.

## General

### What is ResonantGenesis?

ResonantGenesis is a decentralized AI agent platform that allows you to create, deploy, and monetize AI agents with blockchain-backed identity. Agents can perform tasks autonomously, collaborate in teams, and be published to a decentralized marketplace.

### How is ResonantGenesis different from other AI platforms?

- **Blockchain Identity**: Every agent has a verifiable on-chain identity (DSID)
- **Decentralized Marketplace**: Buy, sell, and rent agents peer-to-peer
- **Multi-Agent Teams**: Create collaborative agent workflows
- **Trust Tiers**: Reputation-based trust system for agent verification
- **Open Source**: Transparent, community-driven development

### Is ResonantGenesis free to use?

We offer a free tier with:
- 60 API requests per minute
- 1 concurrent session
- Basic agent features

Paid plans unlock higher limits, more features, and priority support.

## Agents

### What can agents do?

Agents can:
- Search the web for information
- Execute code in sandboxed environments
- Read and write files
- Make API calls to external services
- Collaborate with other agents
- Run autonomously on schedules or triggers

### How do I create an agent?

1. Go to **Agent Studio** in the navigation
2. Click **Create Agent**
3. Follow the guided wizard or use advanced mode
4. Configure name, system prompt, model, and tools
5. Test and publish

See our [Getting Started Guide](./GETTING_STARTED.md) for detailed instructions.

### What AI models are supported?

| Model | Provider | Best For |
|-------|----------|----------|
| GPT-4 Turbo | OpenAI | Complex reasoning, coding |
| GPT-3.5 Turbo | OpenAI | Quick tasks, cost-effective |
| Claude 3 Opus | Anthropic | Long documents, analysis |
| Claude 3 Sonnet | Anthropic | Balanced performance |

### Can I use my own API keys?

Yes! You can bring your own API keys for:
- OpenAI
- Anthropic
- Custom LLM endpoints

Configure in **Settings** > **API Keys**.

### How do I improve my agent's responses?

1. **Refine your system prompt**: Be specific about behavior and constraints
2. **Lower temperature**: More focused, deterministic responses
3. **Add examples**: Show the agent what good responses look like
4. **Use appropriate tools**: Enable only what's needed
5. **Set clear goals**: Be specific in session goals

## Agent Teams

### What are Agent Teams?

Agent Teams allow multiple agents to collaborate on complex tasks. Each agent has a specialized role, and they work together following a coordination strategy.

### What coordination strategies are available?

| Strategy | Description |
|----------|-------------|
| Sequential | Agents work one after another |
| Parallel | Agents work simultaneously |
| Hierarchical | Manager agent delegates to workers |
| Consensus | Agents vote on decisions |

### How do I create a team?

```python
team = client.teams.create(
    name="Content Team",
    members=[researcher.id, writer.id, editor.id],
    coordination_strategy="sequential"
)
```

## Blockchain & DSID

### What is a DSID?

DSID (Decentralized Semantic Identity) is a unique, blockchain-verified identity for your agent. It provides:
- Immutable proof of ownership
- Verifiable execution history
- Trust scoring
- Marketplace listing capability

### Which blockchain does ResonantGenesis use?

We use **Base** (an Ethereum L2) for:
- Low transaction costs
- Fast confirmations
- Ethereum compatibility

Testnet: Base Sepolia
Mainnet: Base Mainnet

### Do I need cryptocurrency to use ResonantGenesis?

No! You can use all platform features without crypto. Blockchain features are optional for users who want:
- Decentralized agent publishing
- Peer-to-peer marketplace transactions
- Verifiable execution proofs

### How do I publish an agent to the blockchain?

```python
publication = agent.publish(
    price_per_execution=0.001,  # ETH
    rental_enabled=True,
    rental_price_per_day=0.01  # ETH
)
```

## Security

### Is my data secure?

Yes. We implement:
- TLS 1.3 encryption for all traffic
- Database encryption at rest
- Regular security audits
- SOC 2 compliance (in progress)

### How are agent executions sandboxed?

External agents run in isolated Docker containers with:
- Resource limits (CPU, memory)
- Network restrictions
- Filesystem isolation
- Time limits

### How do I report a security vulnerability?

Please report security issues to security@resonantgenesis.xyz or via [GitHub Security Advisories](https://github.com/louienemesh/ResonantGenesis/security/advisories/new).

See our [Security Policy](../SECURITY.md) for details.

## API & SDKs

### What programming languages are supported?

Official SDKs:
- **Python**: `pip install resonantgenesis`
- **JavaScript/TypeScript**: `npm install @resonantgenesis/sdk`

The REST API works with any language that can make HTTP requests.

### What are the rate limits?

| Tier | Requests/min | Concurrent Sessions |
|------|--------------|---------------------|
| Free | 60 | 1 |
| Plus | 300 | 5 |
| Enterprise | 1000 | 20 |

### How do I get an API key?

1. Sign in to your account
2. Go to **Settings** > **API Keys**
3. Click **Generate New Key**
4. Copy and store securely (shown only once)

### Is there a webhook/callback system?

Yes! You can create webhook triggers:

```python
trigger = agent.triggers.create(
    name="GitHub Push",
    type="webhook",
    config={
        "secret": "your-secret",
        "goal_template": "Process: {{event.commits}}"
    }
)
```

## Pricing & Billing

### How is usage calculated?

Usage is based on:
- **Tokens**: Input and output tokens processed
- **Sessions**: Number of agent execution sessions
- **Tools**: External API calls made by agents

### Can I set spending limits?

Yes! In **Settings** > **Billing**:
- Set monthly budget caps
- Configure alerts at thresholds
- Auto-pause at limits

### What payment methods are accepted?

- Credit/debit cards (Visa, Mastercard, Amex)
- Cryptocurrency (ETH, USDC on Base)
- Invoice billing (Enterprise)

## Troubleshooting

### My agent isn't responding

1. Check API key validity in Settings
2. Verify agent status is "active"
3. Check rate limits haven't been exceeded
4. Review error logs in Control Center

### Sessions are timing out

1. Reduce `max_steps` for complex tasks
2. Break large tasks into smaller goals
3. Check if external APIs are responding
4. Increase timeout in session config

### High token usage

1. Use shorter system prompts
2. Set `max_tokens` limits
3. Use GPT-3.5 for simpler tasks
4. Enable response caching

### Agent giving wrong answers

1. Review and refine system prompt
2. Add more specific instructions
3. Include examples of desired output
4. Lower temperature for consistency

## Getting Help

### Where can I get support?

- **Help Center**: [resonantgenesis.xyz/help](https://resonantgenesis.xyz/help)
- **Discord**: [Join our community](https://discord.gg/resonantgenesis)
- **GitHub**: [Report issues](https://github.com/louienemesh/ResonantGenesis/issues)
- **Email**: support@resonantgenesis.xyz

### How do I request a feature?

1. Check existing [GitHub Issues](https://github.com/louienemesh/ResonantGenesis/issues)
2. Open a new issue with the "feature request" label
3. Or post in our Discord #feature-requests channel

### How can I contribute?

See our [Contributing Guide](../CONTRIBUTING.md) for:
- Development setup
- Code style guidelines
- Pull request process
- Testing requirements

---

**Still have questions?** [Contact us](mailto:support@resonantgenesis.xyz) or join our [Discord](https://discord.gg/resonantgenesis)!
