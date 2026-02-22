# Agents Guide

Complete guide to agent creation, configuration, and lifecycle management in ResonantGenesis.

## Overview

Agents are autonomous AI entities that execute tasks on your behalf. Each agent has a unique identity, capabilities, and configuration.

## Agent Anatomy

```
Agent
├── Identity
│   ├── ID (unique identifier)
│   ├── Name
│   └── Description
├── Configuration
│   ├── System Prompt
│   ├── Model
│   └── Model Config
├── Capabilities
│   ├── Tools
│   └── Permissions
└── Metadata
    ├── Created At
    ├── Updated At
    └── Status
```

## Creating Agents

### Basic Agent

```python
agent = client.agents.create(
    name="Research Assistant",
    description="Researches topics and provides summaries",
    system_prompt="You are a helpful research assistant."
)

print(f"Agent ID: {agent.id}")
print(f"Status: {agent.status}")
```

### Agent with Tools

```python
agent = client.agents.create(
    name="Web Researcher",
    system_prompt="""You are a web research specialist.
    
Search the web for information and provide accurate summaries.
Always cite your sources.""",
    tools=["web_search"],
    tool_config={
        "web_search": {
            "max_results": 10,
            "safe_search": True
        }
    }
)
```

### Agent with Model Config

```python
agent = client.agents.create(
    name="Creative Writer",
    system_prompt="You are a creative fiction writer.",
    model="gpt-4-turbo",
    model_config={
        "temperature": 0.9,
        "max_tokens": 4000,
        "top_p": 0.95
    }
)
```

### Full Configuration

```python
agent = client.agents.create(
    name="Enterprise Assistant",
    description="Multi-purpose enterprise assistant",
    system_prompt="""You are an enterprise assistant.

## Capabilities
- Answer questions about company policies
- Help with document drafting
- Schedule meetings
- Analyze data

## Guidelines
- Be professional and concise
- Protect confidential information
- Escalate sensitive requests
""",
    model="gpt-4-turbo",
    model_config={
        "temperature": 0.7,
        "max_tokens": 4000
    },
    tools=["web_search", "code_exec", "calendar"],
    tool_config={
        "code_exec": {
            "languages": ["python"],
            "timeout_seconds": 30
        }
    },
    fallback_models=["gpt-4", "gpt-3.5-turbo"],
    metadata={
        "department": "engineering",
        "owner": "team-alpha"
    }
)
```

## Agent Configuration

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Agent name (1-100 chars) |
| `system_prompt` | string | Instructions for the agent |

### Optional Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `description` | string | "" | Agent description |
| `model` | string | "gpt-4-turbo" | AI model to use |
| `model_config` | dict | {} | Model parameters |
| `tools` | list | [] | Enabled tools |
| `tool_config` | dict | {} | Tool configurations |
| `fallback_models` | list | [] | Backup models |
| `metadata` | dict | {} | Custom metadata |

### System Prompt Best Practices

```python
# Good: Structured, clear instructions
system_prompt = """You are a customer support agent for TechCorp.

## Your Role
- Answer customer questions
- Resolve issues
- Escalate complex problems

## Guidelines
- Be friendly and professional
- Never share internal information
- Always verify customer identity

## Response Format
- Greet the customer
- Address their concern
- Provide next steps
- Ask if they need more help
"""

# Bad: Vague, unstructured
system_prompt = "Help customers with their problems."
```

## Agent Lifecycle

### States

```
Created → Active → Paused → Archived
                ↓
              Deleted
```

| State | Description |
|-------|-------------|
| `active` | Ready to execute |
| `paused` | Temporarily disabled |
| `archived` | Soft deleted |
| `deleted` | Permanently removed |

### State Transitions

```python
# Pause agent
client.agents.pause(agent_id)

# Resume agent
client.agents.resume(agent_id)

# Archive agent
client.agents.archive(agent_id)

# Delete agent
client.agents.delete(agent_id)
```

## Managing Agents

### List Agents

```python
# List all agents
agents = client.agents.list()

for agent in agents:
    print(f"{agent.name}: {agent.status}")

# Filter by status
active_agents = client.agents.list(status="active")

# Paginate
page1 = client.agents.list(limit=10, offset=0)
page2 = client.agents.list(limit=10, offset=10)
```

### Get Agent

```python
agent = client.agents.get(agent_id)

print(f"Name: {agent.name}")
print(f"Model: {agent.model}")
print(f"Tools: {agent.tools}")
print(f"Created: {agent.created_at}")
```

### Update Agent

```python
# Update configuration
agent = client.agents.update(
    agent_id,
    name="Updated Name",
    system_prompt="Updated instructions...",
    model_config={
        "temperature": 0.5
    }
)

# Add tools
agent = client.agents.update(
    agent_id,
    tools=agent.tools + ["file_access"]
)
```

### Delete Agent

```python
# Soft delete (archive)
client.agents.archive(agent_id)

# Hard delete (permanent)
client.agents.delete(agent_id, permanent=True)
```

## Executing Agents

### Basic Execution

```python
result = agent.execute(goal="Research AI trends for 2026")

print(f"Status: {result.status}")
print(f"Output: {result.output}")
```

### With Context

```python
result = agent.execute(
    goal="Summarize this document",
    context={
        "document": document_text,
        "format": "bullet_points"
    }
)
```

### With Options

```python
result = agent.execute(
    goal="Complex analysis",
    max_steps=20,
    max_tokens=8000,
    timeout_seconds=300
)
```

### Streaming

```python
async for event in agent.execute_stream(goal="Long task"):
    if event.type == "thinking":
        print(f"Thinking: {event.content}")
    elif event.type == "output":
        print(f"Output: {event.content}")
```

## Agent Templates

### Using Templates

```python
# List available templates
templates = client.templates.list()

# Create from template
agent = client.agents.create_from_template(
    template_id="research-assistant",
    name="My Research Agent",
    customizations={
        "focus_area": "technology"
    }
)
```

### Creating Templates

```python
# Save agent as template
template = client.templates.create(
    name="Customer Support Template",
    description="Template for support agents",
    agent_id=agent.id
)
```

## Agent Cloning

```python
# Clone an existing agent
cloned = client.agents.clone(
    agent_id,
    name="Cloned Agent",
    modifications={
        "model": "gpt-4"
    }
)
```

## Agent Versioning

### Version History

```python
# List versions
versions = client.agents.list_versions(agent_id)

for v in versions:
    print(f"v{v.version}: {v.created_at}")
```

### Rollback

```python
# Rollback to previous version
client.agents.rollback(agent_id, version=3)
```

## Agent Metrics

### Get Metrics

```python
metrics = client.agents.get_metrics(
    agent_id,
    period="7d"
)

print(f"Executions: {metrics.total_executions}")
print(f"Success rate: {metrics.success_rate}%")
print(f"Avg duration: {metrics.avg_duration_ms}ms")
print(f"Tokens used: {metrics.total_tokens}")
```

### Execution History

```python
# List recent executions
executions = client.agents.list_executions(
    agent_id,
    limit=50,
    status="completed"
)

for ex in executions:
    print(f"{ex.id}: {ex.goal[:50]}...")
```

## Agent Permissions

### Setting Permissions

```python
# Share agent with team
client.agents.share(
    agent_id,
    users=["user_123", "user_456"],
    permission="execute"  # execute, edit, admin
)

# Make public
client.agents.update(
    agent_id,
    visibility="public"
)
```

### Permission Levels

| Level | Capabilities |
|-------|-------------|
| `execute` | Run agent |
| `edit` | Modify configuration |
| `admin` | Full control, delete |

## Agent Export/Import

### Export

```python
# Export agent configuration
config = client.agents.export(agent_id)

# Save to file
with open("agent_config.json", "w") as f:
    json.dump(config, f)
```

### Import

```python
# Import from file
with open("agent_config.json", "r") as f:
    config = json.load(f)

agent = client.agents.import_config(config)
```

## Best Practices

### Design

1. **Single responsibility** - One agent, one purpose
2. **Clear instructions** - Specific, structured prompts
3. **Appropriate tools** - Only enable needed tools
4. **Test thoroughly** - Validate before production

### Operations

1. **Monitor metrics** - Track performance
2. **Version control** - Use versioning
3. **Set limits** - Configure timeouts and token limits
4. **Handle errors** - Implement fallbacks

### Security

1. **Least privilege** - Minimal permissions
2. **Audit logs** - Review activity
3. **Secure secrets** - Use secret management
4. **Input validation** - Sanitize inputs

## API Reference

### Create Agent

```bash
POST /api/v1/agents
```

### Get Agent

```bash
GET /api/v1/agents/{agent_id}
```

### Update Agent

```bash
PATCH /api/v1/agents/{agent_id}
```

### Delete Agent

```bash
DELETE /api/v1/agents/{agent_id}
```

### List Agents

```bash
GET /api/v1/agents
```

### Execute Agent

```bash
POST /api/v1/agents/{agent_id}/execute
```

---

**Need help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
