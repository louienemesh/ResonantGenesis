# Getting Started with ResonantGenesis

Welcome to ResonantGenesis! This guide will help you create your first AI agent in under 10 minutes.

## Prerequisites

- A ResonantGenesis account ([Sign up here](https://resonantgenesis.xyz/signup))
- Basic understanding of AI agents and prompts

## Quick Start: Create Your First Agent

### Step 1: Sign In

1. Go to [resonantgenesis.xyz](https://resonantgenesis.xyz)
2. Click **Sign In** or **Get Started**
3. Create an account or sign in with your existing credentials

### Step 2: Navigate to Agent Studio

1. Click **Agent Studio** in the top navigation menu
2. Click the **Create Agent** button
3. Choose **Guided Wizard** for a step-by-step experience

### Step 3: Configure Your Agent

#### Basic Information
- **Name**: Give your agent a descriptive name (e.g., "Research Assistant")
- **Description**: Briefly describe what your agent does

#### System Prompt
This is the most important part! The system prompt defines your agent's personality and capabilities.

**Example for a Research Assistant:**
```
You are a helpful research assistant. Your job is to:
1. Search for accurate, up-to-date information
2. Summarize findings clearly and concisely
3. Cite sources when possible
4. Ask clarifying questions when the request is unclear

Be professional, thorough, and honest about limitations.
```

#### Select AI Model
Choose the model that best fits your needs:

| Model | Best For | Speed | Cost |
|-------|----------|-------|------|
| GPT-4 Turbo | Complex reasoning, coding | Medium | Higher |
| GPT-3.5 Turbo | Quick tasks, simple queries | Fast | Lower |
| Claude 3 | Long documents, analysis | Medium | Medium |

#### Enable Tools
Select the tools your agent can use:

- **Web Search**: Search the internet for information
- **Code Execution**: Run code in a sandboxed environment
- **File Access**: Read and write files
- **API Calls**: Make HTTP requests to external services

### Step 4: Test Your Agent

1. Click **Save Agent**
2. Click **Test** to open the chat interface
3. Try a sample conversation:

```
You: Research the latest developments in quantum computing
Agent: I'll search for recent quantum computing news...
```

### Step 5: Deploy Your Agent

Once you're happy with your agent:

1. Click **Publish** to make it available
2. Choose visibility:
   - **Private**: Only you can use it
   - **Team**: Share with your organization
   - **Public**: Available on the marketplace

## Using the API

### Authentication

```bash
# Get your API key from Settings > API Keys
export RESONANT_API_KEY="your_api_key_here"
```

### Create an Agent via API

```bash
curl -X POST https://resonantgenesis.xyz/api/v1/agents \
  -H "Authorization: Bearer $RESONANT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My First Agent",
    "description": "A helpful assistant",
    "system_prompt": "You are a helpful assistant.",
    "model": "gpt-4-turbo",
    "tools": ["web_search"]
  }'
```

### Start a Session

```bash
curl -X POST https://resonantgenesis.xyz/api/v1/agents/{agent_id}/sessions \
  -H "Authorization: Bearer $RESONANT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "goal": "Research quantum computing trends",
    "max_steps": 5
  }'
```

## Using the SDK

### Python

```bash
pip install resonantgenesis
```

```python
from resonantgenesis import ResonantClient

client = ResonantClient(api_key="your_api_key")

# Create an agent
agent = client.agents.create(
    name="Research Assistant",
    system_prompt="You are a helpful research assistant.",
    model="gpt-4-turbo",
    tools=["web_search"]
)

# Run a task
result = agent.execute("Find the top 5 AI papers this week")
print(result.output)
```

### JavaScript/TypeScript

```bash
npm install @resonantgenesis/sdk
```

```typescript
import { ResonantClient } from '@resonantgenesis/sdk';

const client = new ResonantClient({ apiKey: 'your_api_key' });

// Create an agent
const agent = await client.agents.create({
  name: 'Research Assistant',
  systemPrompt: 'You are a helpful research assistant.',
  model: 'gpt-4-turbo',
  tools: ['web_search']
});

// Run a task
const result = await agent.execute('Find the top 5 AI papers this week');
console.log(result.output);
```

## Common Use Cases

### 1. Research Assistant

```python
agent = client.agents.create(
    name="Research Assistant",
    system_prompt="""You are a research assistant that:
    - Searches for accurate information
    - Summarizes findings clearly
    - Cites sources
    - Identifies key trends""",
    tools=["web_search", "file_access"]
)
```

### 2. Code Reviewer

```python
agent = client.agents.create(
    name="Code Reviewer",
    system_prompt="""You are an expert code reviewer that:
    - Identifies bugs and security issues
    - Suggests improvements
    - Follows best practices
    - Explains issues clearly""",
    tools=["code_exec", "file_access"]
)
```

### 3. Data Analyst

```python
agent = client.agents.create(
    name="Data Analyst",
    system_prompt="""You are a data analyst that:
    - Analyzes datasets
    - Creates visualizations
    - Identifies patterns
    - Provides actionable insights""",
    tools=["code_exec", "file_access"]
)
```

## Agent Teams

Create multi-agent teams for complex workflows:

```python
# Create individual agents
researcher = client.agents.create(name="Researcher", ...)
writer = client.agents.create(name="Writer", ...)
editor = client.agents.create(name="Editor", ...)

# Create a team
team = client.teams.create(
    name="Content Team",
    members=[researcher.id, writer.id, editor.id],
    coordination_strategy="sequential"
)

# Run a team workflow
result = team.execute("Write a blog post about AI trends")
```

## Blockchain Integration

Publish your agent to the decentralized network:

```python
# Publish agent with DSID (Decentralized Semantic Identity)
publication = agent.publish(
    price_per_execution=0.001,
    rental_enabled=True,
    rental_price_per_day=0.01
)

print(f"Agent published with DSID: {publication.dsid}")
print(f"Transaction: {publication.transaction_hash}")
```

## Monitoring & Analytics

Track your agent's performance:

1. Go to **Control Center** > **Overview**
2. View metrics:
   - Total executions
   - Success rate
   - Average response time
   - Token usage
   - Cost breakdown

## Troubleshooting

### Agent Not Responding

1. Check your API key is valid
2. Verify the agent is in "active" status
3. Check rate limits haven't been exceeded

### Unexpected Results

1. Review and refine your system prompt
2. Adjust temperature setting (lower = more focused)
3. Add more specific instructions

### High Costs

1. Use GPT-3.5 Turbo for simpler tasks
2. Set `max_tokens` limits
3. Reduce `max_steps` for sessions

## Next Steps

- **[API Reference](./API_REFERENCE.md)**: Complete API documentation
- **[Architecture](./ARCHITECTURE.md)**: System design overview
- **[Contributing](../CONTRIBUTING.md)**: How to contribute
- **[Security](../SECURITY.md)**: Security best practices

## Get Help

- **Help Center**: [resonantgenesis.xyz/help](https://resonantgenesis.xyz/help)
- **Discord**: [Join our community](https://discord.gg/resonantgenesis)
- **GitHub**: [Report issues](https://github.com/louienemesh/ResonantGenesis/issues)
- **Email**: support@resonantgenesis.xyz

---

**Ready to build?** [Create your first agent now →](https://resonantgenesis.xyz/agents/new)
