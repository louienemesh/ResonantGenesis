# Templates Guide

Complete guide to agent templates, marketplace templates, and template customization in ResonantGenesis.

## Overview

Templates are pre-configured agent blueprints that accelerate development. They include system prompts, tool configurations, and best practices for common use cases.

## Template Types

| Type | Description | Source |
|------|-------------|--------|
| **Built-in** | Platform-provided templates | ResonantGenesis |
| **Marketplace** | Community-created templates | Marketplace |
| **Custom** | Your own saved templates | Your account |
| **Organization** | Shared within your org | Team settings |

## Built-in Templates

### Research Assistant

```python
template = client.templates.get("research-assistant")

agent = client.agents.create_from_template(
    template_id="research-assistant",
    name="My Research Agent"
)
```

**Capabilities:**
- Web search and analysis
- Source citation
- Summary generation
- Multi-source synthesis

**Tools:** `web_search`

### Code Assistant

```python
agent = client.agents.create_from_template(
    template_id="code-assistant",
    name="My Code Helper"
)
```

**Capabilities:**
- Code generation
- Bug fixing
- Code review
- Documentation

**Tools:** `code_exec`

### Customer Support

```python
agent = client.agents.create_from_template(
    template_id="customer-support",
    name="Support Bot"
)
```

**Capabilities:**
- Ticket triage
- FAQ responses
- Escalation handling
- Sentiment analysis

**Tools:** `web_search`

### Data Analyst

```python
agent = client.agents.create_from_template(
    template_id="data-analyst",
    name="Analytics Agent"
)
```

**Capabilities:**
- Data processing
- Visualization
- Statistical analysis
- Report generation

**Tools:** `code_exec`, `file_access`

### Content Writer

```python
agent = client.agents.create_from_template(
    template_id="content-writer",
    name="Blog Writer"
)
```

**Capabilities:**
- Blog posts
- Social media content
- Email drafts
- SEO optimization

**Tools:** `web_search`

### Task Automation

```python
agent = client.agents.create_from_template(
    template_id="task-automation",
    name="Workflow Bot"
)
```

**Capabilities:**
- Scheduled tasks
- Multi-step workflows
- API integrations
- Data transformation

**Tools:** `api_call`, `code_exec`

## Using Templates

### List Available Templates

```python
# List all templates
templates = client.templates.list()

for t in templates:
    print(f"{t.id}: {t.name}")
    print(f"  Category: {t.category}")
    print(f"  Tools: {t.tools}")

# Filter by category
research_templates = client.templates.list(category="research")
```

### Get Template Details

```python
template = client.templates.get("research-assistant")

print(f"Name: {template.name}")
print(f"Description: {template.description}")
print(f"System Prompt: {template.system_prompt[:200]}...")
print(f"Tools: {template.tools}")
print(f"Model: {template.model}")
print(f"Downloads: {template.download_count}")
```

### Create Agent from Template

```python
# Basic creation
agent = client.agents.create_from_template(
    template_id="research-assistant",
    name="My Agent"
)

# With customizations
agent = client.agents.create_from_template(
    template_id="research-assistant",
    name="Custom Research Agent",
    customizations={
        "focus_area": "technology",
        "output_format": "bullet_points",
        "max_sources": 5
    }
)

# Override template settings
agent = client.agents.create_from_template(
    template_id="research-assistant",
    name="Premium Research Agent",
    model="gpt-4",  # Override default model
    tools=["web_search", "code_exec"]  # Add tools
)
```

## Template Customization

### Customization Variables

Templates can define variables for customization:

```python
# Template definition
template = {
    "name": "Industry Expert",
    "system_prompt": """You are an expert in {{industry}}.
    
Your communication style is {{tone}}.
You focus on {{focus_areas}}.
""",
    "variables": {
        "industry": {
            "type": "string",
            "required": True,
            "description": "Industry specialization"
        },
        "tone": {
            "type": "enum",
            "options": ["professional", "casual", "technical"],
            "default": "professional"
        },
        "focus_areas": {
            "type": "array",
            "description": "Areas of focus"
        }
    }
}

# Using the template
agent = client.agents.create_from_template(
    template_id="industry-expert",
    name="Tech Expert",
    customizations={
        "industry": "artificial intelligence",
        "tone": "technical",
        "focus_areas": ["machine learning", "NLP", "computer vision"]
    }
)
```

### System Prompt Customization

```python
# Get template
template = client.templates.get("research-assistant")

# Modify system prompt
custom_prompt = template.system_prompt + """

## Additional Guidelines
- Always cite sources with URLs
- Provide confidence scores
- Flag uncertain information
"""

# Create with custom prompt
agent = client.agents.create(
    name="Enhanced Research Agent",
    system_prompt=custom_prompt,
    tools=template.tools,
    model=template.model
)
```

### Tool Customization

```python
# Start with template tools
template = client.templates.get("data-analyst")

# Add additional tools
agent = client.agents.create_from_template(
    template_id="data-analyst",
    name="Advanced Analyst",
    tools=template.tools + ["database", "api_call"],
    tool_config={
        "database": {
            "connection_string": "postgresql://...",
            "read_only": True
        }
    }
)
```

## Creating Custom Templates

### Save Agent as Template

```python
# Create and test an agent
agent = client.agents.create(
    name="My Custom Agent",
    system_prompt="...",
    tools=["web_search", "code_exec"],
    model="gpt-4-turbo"
)

# Test the agent
result = agent.execute(goal="Test task")

# Save as template
template = client.templates.create(
    name="My Custom Template",
    description="Template for specialized tasks",
    agent_id=agent.id,
    category="custom",
    tags=["productivity", "automation"]
)

print(f"Template ID: {template.id}")
```

### Template Definition

```python
# Create template from scratch
template = client.templates.create(
    name="Sales Assistant",
    description="AI assistant for sales teams",
    system_prompt="""You are a sales assistant.

## Your Role
- Qualify leads
- Draft proposals
- Answer product questions
- Schedule follow-ups

## Guidelines
- Be professional and friendly
- Focus on customer needs
- Never make false promises
""",
    model="gpt-4-turbo",
    tools=["web_search", "calendar"],
    tool_config={
        "web_search": {"max_results": 5}
    },
    variables={
        "company_name": {
            "type": "string",
            "required": True
        },
        "product_focus": {
            "type": "string",
            "default": "all products"
        }
    },
    category="sales",
    tags=["sales", "crm", "productivity"]
)
```

## Marketplace Templates

### Browse Marketplace

```python
# Search marketplace templates
templates = client.marketplace.templates.search(
    query="customer support",
    category="support",
    sort_by="downloads",
    limit=20
)

for t in templates:
    print(f"{t.name} by {t.author}")
    print(f"  Rating: {t.rating}/5 ({t.reviews} reviews)")
    print(f"  Downloads: {t.download_count}")
    print(f"  Price: {'Free' if t.price == 0 else f'${t.price}'}")
```

### Purchase Template

```python
# Get template details
template = client.marketplace.templates.get("tmpl_abc123")

# Purchase (if paid)
if template.price > 0:
    purchase = client.marketplace.templates.purchase(template.id)
    print(f"Purchase ID: {purchase.id}")

# Use template
agent = client.agents.create_from_template(
    template_id=template.id,
    name="My Agent"
)
```

### Publish Template

```python
# Publish to marketplace
publication = client.marketplace.templates.publish(
    template_id="my-template-id",
    price=0,  # Free or set price
    visibility="public",
    description="Detailed description...",
    screenshots=["url1", "url2"],
    demo_video="youtube_url"
)

print(f"Published: {publication.url}")
```

## Organization Templates

### Share with Team

```python
# Create org template
template = client.templates.create(
    name="Company Support Agent",
    description="Standard support agent for our team",
    agent_id=agent.id,
    visibility="organization"
)

# List org templates
org_templates = client.templates.list(visibility="organization")
```

### Template Permissions

```python
# Set permissions
client.templates.set_permissions(
    template_id="tmpl_123",
    permissions={
        "view": ["team_support", "team_sales"],
        "use": ["team_support"],
        "edit": ["admin"]
    }
)
```

## Template Versioning

### Version History

```python
# List versions
versions = client.templates.list_versions("tmpl_123")

for v in versions:
    print(f"v{v.version}: {v.created_at}")
    print(f"  Changes: {v.changelog}")
```

### Update Template

```python
# Update template
client.templates.update(
    template_id="tmpl_123",
    system_prompt="Updated prompt...",
    changelog="Improved response quality"
)
```

### Rollback

```python
# Rollback to previous version
client.templates.rollback(
    template_id="tmpl_123",
    version=2
)
```

## Best Practices

### Template Design

1. **Clear purpose** - One template, one use case
2. **Good defaults** - Sensible default values
3. **Flexible variables** - Allow customization
4. **Documentation** - Explain how to use

### System Prompts

1. **Structure** - Use clear sections
2. **Examples** - Include example outputs
3. **Constraints** - Define boundaries
4. **Tone** - Specify communication style

### Tool Selection

1. **Minimal tools** - Only include necessary tools
2. **Safe defaults** - Conservative tool configs
3. **Documentation** - Explain tool usage

### Testing

1. **Test thoroughly** - Before publishing
2. **Edge cases** - Handle unusual inputs
3. **Feedback** - Incorporate user feedback

## API Reference

### List Templates

```bash
GET /api/v1/templates
```

### Get Template

```bash
GET /api/v1/templates/{template_id}
```

### Create Template

```bash
POST /api/v1/templates
```

### Update Template

```bash
PATCH /api/v1/templates/{template_id}
```

### Delete Template

```bash
DELETE /api/v1/templates/{template_id}
```

### Create Agent from Template

```bash
POST /api/v1/agents/from-template
```

---

**Need help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
