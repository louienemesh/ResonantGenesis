# Plugins Guide

Complete guide to the plugin system, plugin development, and plugin marketplace in ResonantGenesis.

## Overview

ResonantGenesis supports a powerful plugin system that allows developers to extend platform functionality. Plugins can add new tools, integrations, models, and capabilities to agents. This guide covers plugin architecture, development, and distribution.

## Plugin Types

| Type | Description | Examples |
|------|-------------|----------|
| **Tool Plugins** | Add new agent tools | Web scraping, PDF parsing |
| **Integration Plugins** | Connect external services | CRM, ERP, databases |
| **Model Plugins** | Add custom models | Fine-tuned models, local LLMs |
| **UI Plugins** | Extend dashboard UI | Custom widgets, visualizations |
| **Middleware Plugins** | Process requests/responses | Logging, filtering, caching |

## Plugin Architecture

### Plugin Structure

```
my-plugin/
├── plugin.yaml          # Plugin manifest
├── src/
│   ├── __init__.py
│   ├── main.py          # Entry point
│   ├── tools/           # Tool implementations
│   └── handlers/        # Event handlers
├── tests/
│   └── test_plugin.py
├── README.md
└── LICENSE
```

### Plugin Manifest

```yaml
# plugin.yaml
name: my-awesome-plugin
version: 1.0.0
description: An awesome plugin for ResonantGenesis
author: Your Name
email: you@example.com
license: MIT

# Plugin type
type: tool

# Minimum platform version
min_platform_version: 1.0.0

# Dependencies
dependencies:
  - requests>=2.28.0
  - beautifulsoup4>=4.11.0

# Entry point
entry_point: src.main:Plugin

# Permissions required
permissions:
  - network:outbound
  - storage:read
  - storage:write

# Configuration schema
config:
  api_key:
    type: string
    required: true
    secret: true
  max_results:
    type: integer
    default: 10
```

## Developing Plugins

### Basic Plugin

```python
# src/main.py
from resonantgenesis.plugins import BasePlugin, tool

class Plugin(BasePlugin):
    """My awesome plugin."""
    
    def __init__(self, config):
        super().__init__(config)
        self.api_key = config.get("api_key")
    
    async def on_load(self):
        """Called when plugin is loaded."""
        self.logger.info("Plugin loaded!")
    
    async def on_unload(self):
        """Called when plugin is unloaded."""
        self.logger.info("Plugin unloaded!")
    
    @tool(
        name="my_tool",
        description="Does something awesome",
        parameters={
            "query": {"type": "string", "required": True}
        }
    )
    async def my_tool(self, query: str) -> dict:
        """Execute the tool."""
        result = await self.do_something(query)
        return {"result": result}
    
    async def do_something(self, query: str) -> str:
        # Implementation
        return f"Processed: {query}"
```

### Tool Plugin

```python
from resonantgenesis.plugins import BasePlugin, tool
import httpx

class WebScraperPlugin(BasePlugin):
    """Web scraping plugin."""
    
    @tool(
        name="scrape_url",
        description="Scrape content from a URL",
        parameters={
            "url": {"type": "string", "required": True},
            "selector": {"type": "string", "required": False}
        }
    )
    async def scrape_url(self, url: str, selector: str = None) -> dict:
        async with httpx.AsyncClient() as client:
            response = await client.get(url)
            content = response.text
            
            if selector:
                from bs4 import BeautifulSoup
                soup = BeautifulSoup(content, 'html.parser')
                elements = soup.select(selector)
                content = [el.get_text() for el in elements]
            
            return {
                "url": url,
                "content": content,
                "status": response.status_code
            }
```

### Integration Plugin

```python
from resonantgenesis.plugins import BasePlugin, integration
import httpx

class SalesforcePlugin(BasePlugin):
    """Salesforce CRM integration."""
    
    def __init__(self, config):
        super().__init__(config)
        self.instance_url = config.get("instance_url")
        self.access_token = config.get("access_token")
    
    @integration(
        name="salesforce",
        description="Salesforce CRM integration"
    )
    async def get_client(self):
        return SalesforceClient(
            instance_url=self.instance_url,
            access_token=self.access_token
        )
    
    @tool(name="sf_query", description="Query Salesforce")
    async def query(self, soql: str) -> dict:
        client = await self.get_client()
        return await client.query(soql)
    
    @tool(name="sf_create", description="Create Salesforce record")
    async def create(self, object_type: str, data: dict) -> dict:
        client = await self.get_client()
        return await client.create(object_type, data)
```

### Middleware Plugin

```python
from resonantgenesis.plugins import BasePlugin, middleware

class LoggingPlugin(BasePlugin):
    """Request/response logging middleware."""
    
    @middleware(order=1)
    async def log_request(self, request, next_handler):
        self.logger.info(f"Request: {request.method} {request.path}")
        
        # Call next handler
        response = await next_handler(request)
        
        self.logger.info(f"Response: {response.status_code}")
        return response
```

### Event Handlers

```python
from resonantgenesis.plugins import BasePlugin, event_handler

class NotificationPlugin(BasePlugin):
    """Send notifications on events."""
    
    @event_handler("session.completed")
    async def on_session_completed(self, event):
        await self.send_notification(
            title="Session Completed",
            message=f"Session {event.session_id} completed successfully"
        )
    
    @event_handler("agent.error")
    async def on_agent_error(self, event):
        await self.send_alert(
            title="Agent Error",
            message=f"Agent {event.agent_id} encountered an error: {event.error}"
        )
```

## Plugin Configuration

### Define Configuration Schema

```yaml
# plugin.yaml
config:
  api_key:
    type: string
    required: true
    secret: true
    description: API key for the service
  
  endpoint:
    type: string
    required: false
    default: https://api.example.com
    description: API endpoint URL
  
  timeout:
    type: integer
    required: false
    default: 30
    min: 1
    max: 300
    description: Request timeout in seconds
  
  features:
    type: array
    items:
      type: string
    default: []
    description: Enabled features
```

### Access Configuration

```python
class MyPlugin(BasePlugin):
    def __init__(self, config):
        super().__init__(config)
        
        # Access config values
        self.api_key = config.get("api_key")
        self.endpoint = config.get("endpoint", "https://api.example.com")
        self.timeout = config.get("timeout", 30)
        self.features = config.get("features", [])
```

## Plugin Permissions

### Available Permissions

| Permission | Description |
|------------|-------------|
| `network:outbound` | Make outbound HTTP requests |
| `network:inbound` | Receive webhooks |
| `storage:read` | Read from storage |
| `storage:write` | Write to storage |
| `secrets:read` | Access secrets |
| `agents:read` | Read agent data |
| `agents:write` | Modify agents |
| `sessions:read` | Read session data |
| `sessions:write` | Create/modify sessions |

### Request Permissions

```yaml
# plugin.yaml
permissions:
  - network:outbound
  - storage:read
  - storage:write
```

### Check Permissions

```python
class MyPlugin(BasePlugin):
    async def my_tool(self):
        # Check if permission is granted
        if not self.has_permission("network:outbound"):
            raise PermissionError("Network access not granted")
        
        # Proceed with network request
        ...
```

## Testing Plugins

### Unit Tests

```python
# tests/test_plugin.py
import pytest
from src.main import Plugin

@pytest.fixture
def plugin():
    config = {
        "api_key": "test_key",
        "max_results": 5
    }
    return Plugin(config)

@pytest.mark.asyncio
async def test_my_tool(plugin):
    result = await plugin.my_tool(query="test")
    assert "result" in result
    assert result["result"] == "Processed: test"

@pytest.mark.asyncio
async def test_on_load(plugin):
    await plugin.on_load()
    # Assert plugin is properly initialized
```

### Integration Tests

```python
# tests/test_integration.py
import pytest
from resonantgenesis.testing import PluginTestClient

@pytest.fixture
def client():
    return PluginTestClient("my-plugin")

@pytest.mark.asyncio
async def test_plugin_with_agent(client):
    # Create test agent with plugin
    agent = await client.create_agent(
        plugins=["my-plugin"],
        plugin_config={
            "my-plugin": {"api_key": "test"}
        }
    )
    
    # Execute agent
    result = await agent.execute(goal="Use my_tool with query 'hello'")
    assert result.success
```

### Local Testing

```bash
# Install plugin locally
rg plugins install ./my-plugin --dev

# Test plugin
rg plugins test my-plugin

# Run with verbose output
rg plugins test my-plugin --verbose
```

## Publishing Plugins

### Prepare for Publishing

```bash
# Validate plugin
rg plugins validate ./my-plugin

# Build plugin package
rg plugins build ./my-plugin

# Output: my-plugin-1.0.0.tar.gz
```

### Publish to Marketplace

```bash
# Login to marketplace
rg marketplace login

# Publish plugin
rg marketplace publish ./my-plugin-1.0.0.tar.gz

# Publish with visibility
rg marketplace publish ./my-plugin-1.0.0.tar.gz --visibility public
```

### Plugin Visibility

| Visibility | Description |
|------------|-------------|
| `private` | Only you can use |
| `organization` | Organization members |
| `public` | Everyone |

## Installing Plugins

### From Marketplace

```bash
# Search plugins
rg plugins search "web scraper"

# Install plugin
rg plugins install web-scraper

# Install specific version
rg plugins install web-scraper@1.2.0
```

### From URL

```bash
# Install from URL
rg plugins install https://example.com/my-plugin-1.0.0.tar.gz

# Install from GitHub
rg plugins install github:username/my-plugin
```

### From Local

```bash
# Install from local directory
rg plugins install ./my-plugin

# Install in development mode
rg plugins install ./my-plugin --dev
```

## Managing Plugins

### List Plugins

```bash
# List installed plugins
rg plugins list

# List with details
rg plugins list --verbose
```

### Update Plugins

```bash
# Update specific plugin
rg plugins update web-scraper

# Update all plugins
rg plugins update --all
```

### Uninstall Plugins

```bash
# Uninstall plugin
rg plugins uninstall web-scraper
```

### Configure Plugins

```bash
# Set plugin config
rg plugins config web-scraper --set api_key=xxx

# View plugin config
rg plugins config web-scraper --list
```

## Using Plugins with Agents

### Enable Plugin for Agent

```python
# Create agent with plugin
agent = client.agents.create(
    name="My Agent",
    plugins=["web-scraper", "salesforce"],
    plugin_config={
        "web-scraper": {
            "max_results": 10
        },
        "salesforce": {
            "instance_url": "https://mycompany.salesforce.com",
            "access_token": "xxx"
        }
    }
)
```

### Plugin Tools in Prompts

```python
# Agent can use plugin tools
agent = client.agents.create(
    name="Research Agent",
    system_prompt="""You are a research assistant.
    
    You have access to the following tools:
    - scrape_url: Scrape content from websites
    - sf_query: Query Salesforce data
    
    Use these tools to help users with their research tasks.
    """,
    plugins=["web-scraper", "salesforce"]
)
```

## Plugin SDK

### Installation

```bash
pip install resonantgenesis-plugin-sdk
```

### SDK Features

```python
from resonantgenesis.plugins import (
    BasePlugin,
    tool,
    integration,
    middleware,
    event_handler,
    PluginContext,
    PluginStorage
)

# Access plugin context
class MyPlugin(BasePlugin):
    async def my_tool(self):
        # Access context
        user_id = self.context.user_id
        org_id = self.context.org_id
        
        # Access storage
        await self.storage.set("key", "value")
        value = await self.storage.get("key")
        
        # Access secrets
        api_key = await self.secrets.get("api_key")
        
        # Log messages
        self.logger.info("Processing...")
```

## Best Practices

### For Development

1. **Follow conventions** - Use standard plugin structure
2. **Handle errors** - Graceful error handling
3. **Document well** - Clear README and docstrings
4. **Test thoroughly** - Unit and integration tests
5. **Version properly** - Semantic versioning

### For Security

1. **Minimal permissions** - Request only needed permissions
2. **Validate input** - Sanitize all user input
3. **Secure secrets** - Never log or expose secrets
4. **Rate limit** - Implement rate limiting
5. **Audit logging** - Log security-relevant events

### For Performance

1. **Async operations** - Use async/await
2. **Connection pooling** - Reuse connections
3. **Caching** - Cache expensive operations
4. **Lazy loading** - Load resources on demand
5. **Resource cleanup** - Clean up in on_unload

## API Reference

### Install Plugin

```bash
POST /api/v1/plugins/install
```

### List Plugins

```bash
GET /api/v1/plugins
```

### Get Plugin

```bash
GET /api/v1/plugins/{plugin_id}
```

### Configure Plugin

```bash
PATCH /api/v1/plugins/{plugin_id}/config
```

### Uninstall Plugin

```bash
DELETE /api/v1/plugins/{plugin_id}
```

---

**Need plugin help?** Contact plugins@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
