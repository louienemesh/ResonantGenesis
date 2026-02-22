# ResonantGenesis API Reference

Complete API documentation for the ResonantGenesis platform.

**Base URL**: `https://resonantgenesis.xyz/api/v1`

---

## Authentication

All API requests require authentication via Bearer token in the Authorization header:

```bash
Authorization: Bearer <your_access_token>
```

---

## Quick Start Guide

Get up and running with the ResonantGenesis API in minutes.

### 1. Get Your API Token

```bash
# Register a new account
curl -X POST https://resonantgenesis.xyz/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "you@example.com", "password": "your-secure-password"}'

# Login to get access token
curl -X POST https://resonantgenesis.xyz/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "you@example.com", "password": "your-secure-password"}'

# Response:
# {"access_token": "eyJ...", "token_type": "bearer", "expires_in": 3600}
```

### 2. Create Your First Agent

```bash
export TOKEN="your_access_token"

curl -X POST https://resonantgenesis.xyz/api/v1/agents \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My First Agent",
    "description": "A helpful research assistant",
    "system_prompt": "You are a helpful research assistant. Be concise and accurate.",
    "model": "gpt-4-turbo",
    "tools": ["web_search"]
  }'
```

### 3. Start an Execution Session

```bash
curl -X POST https://resonantgenesis.xyz/api/v1/agents/{agent_id}/sessions \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "goal": "Research the latest developments in quantum computing",
    "max_steps": 5
  }'
```

### 4. Check Session Status

```bash
curl -X GET https://resonantgenesis.xyz/api/v1/agents/{agent_id}/sessions/{session_id} \
  -H "Authorization: Bearer $TOKEN"
```

### Python SDK Example

```python
from resonantgenesis import ResonantClient

# Initialize client
client = ResonantClient(api_key="your_api_key")

# Create an agent
agent = client.agents.create(
    name="Research Assistant",
    system_prompt="You are a helpful research assistant.",
    model="gpt-4-turbo",
    tools=["web_search", "code_exec"]
)

# Start a session
session = agent.execute(
    goal="Analyze the top 5 AI papers from this week",
    max_steps=10
)

# Wait for completion and get results
result = session.wait()
print(result.output)
```

### JavaScript/TypeScript SDK Example

```typescript
import { ResonantClient } from '@resonantgenesis/sdk';

const client = new ResonantClient({ apiKey: 'your_api_key' });

// Create and execute an agent
const agent = await client.agents.create({
  name: 'Code Reviewer',
  systemPrompt: 'You are an expert code reviewer.',
  model: 'gpt-4-turbo',
  tools: ['code_exec']
});

const session = await agent.execute({
  goal: 'Review this pull request for security issues',
  context: { prUrl: 'https://github.com/...' }
});

console.log(session.result);
```

---

## Agents API

### Create Agent
```http
POST /agents
```

Create a new AI agent with specified configuration.

**Request Body:**
```json
{
  "name": "My Agent",
  "description": "Agent description",
  "system_prompt": "You are a helpful assistant...",
  "model": "gpt-4-turbo",
  "temperature": 0.7,
  "max_tokens": 4096,
  "tools": ["web_search", "code_exec"],
  "allowed_actions": [],
  "blocked_actions": []
}
```

**Response:** `201 Created`
```json
{
  "id": "uuid",
  "name": "My Agent",
  "status": "active",
  "created_at": "2026-02-21T00:00:00Z",
  "agent_public_hash": "0x...",
  "agent_version_hash": "0x..."
}
```

---

### List Agents
```http
GET /agents
```

Retrieve all agents owned by the authenticated user.

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | Filter by status: `active`, `archived` |
| `limit` | integer | Max results (default: 50) |
| `offset` | integer | Pagination offset |

**Response:** `200 OK`
```json
[
  {
    "id": "uuid",
    "name": "Agent Name",
    "status": "active",
    "created_at": "2026-02-21T00:00:00Z"
  }
]
```

---

### Get Agent
```http
GET /agents/{agent_id}
```

Retrieve details of a specific agent.

**Response:** `200 OK`
```json
{
  "id": "uuid",
  "name": "My Agent",
  "description": "...",
  "system_prompt": "...",
  "model": "gpt-4-turbo",
  "temperature": 0.7,
  "max_tokens": 4096,
  "tools": ["web_search"],
  "status": "active",
  "agent_public_hash": "0x...",
  "agent_version_hash": "0x...",
  "created_at": "2026-02-21T00:00:00Z"
}
```

---

### Delete Agent
```http
DELETE /agents/{agent_id}
```

Delete an agent. This action is irreversible.

**Response:** `204 No Content`

---

### Publish Agent to Blockchain
```http
POST /agents/{agent_id}/publish
```

Publish an agent's identity to the Base blockchain, creating an immutable record.

**Request Body:**
```json
{
  "price_per_execution": 0.001,
  "rental_enabled": true,
  "rental_price_per_day": 0.01
}
```

**Response:** `200 OK`
```json
{
  "transaction_hash": "0x...",
  "dsid": "0x...",
  "block_number": 12345678,
  "status": "confirmed"
}
```

---

## Sessions API

### Start Session
```http
POST /agents/{agent_id}/sessions
```

Start a new execution session with an agent.

**Request Body:**
```json
{
  "goal": "Research the latest AI developments",
  "context": {
    "user_preference": "concise"
  },
  "max_steps": 10
}
```

**Response:** `201 Created`
```json
{
  "id": "session-uuid",
  "agent_id": "agent-uuid",
  "status": "running",
  "goal": "Research the latest AI developments",
  "created_at": "2026-02-21T00:00:00Z"
}
```

---

### List Sessions
```http
GET /agents/{agent_id}/sessions
```

List all sessions for an agent.

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | Filter: `running`, `completed`, `failed`, `cancelled` |
| `limit` | integer | Max results |

**Response:** `200 OK`
```json
[
  {
    "id": "session-uuid",
    "status": "completed",
    "goal": "...",
    "steps_completed": 5,
    "created_at": "2026-02-21T00:00:00Z",
    "completed_at": "2026-02-21T00:05:00Z"
  }
]
```

---

### Get Session Details
```http
GET /agents/{agent_id}/sessions/{session_id}
```

Get detailed information about a specific session.

**Response:** `200 OK`
```json
{
  "id": "session-uuid",
  "agent_id": "agent-uuid",
  "status": "completed",
  "goal": "...",
  "result": "Session output...",
  "steps": [...],
  "tokens_used": 1500,
  "cost": 0.003,
  "created_at": "2026-02-21T00:00:00Z",
  "completed_at": "2026-02-21T00:05:00Z"
}
```

---

### Get Session Steps
```http
GET /sessions/{session_id}/steps
```

Retrieve all execution steps for a session.

**Response:** `200 OK`
```json
[
  {
    "id": "step-uuid",
    "step_number": 1,
    "action": "web_search",
    "input": {"query": "AI news 2026"},
    "output": "Search results...",
    "status": "completed",
    "created_at": "2026-02-21T00:00:01Z"
  }
]
```

---

### Cancel Session
```http
POST /sessions/{session_id}/cancel
```

Cancel a running session.

**Response:** `200 OK`
```json
{
  "id": "session-uuid",
  "status": "cancelled"
}
```

---

## Agent Teams API

### List Teams
```http
GET /agents/teams
```

List all agent teams owned by the user.

**Response:** `200 OK`
```json
[
  {
    "id": "team-uuid",
    "name": "Research Team",
    "description": "Multi-agent research team",
    "member_count": 3,
    "status": "active"
  }
]
```

---

### Create Team
```http
POST /agents/teams
```

Create a new agent team for multi-agent collaboration.

**Request Body:**
```json
{
  "name": "Research Team",
  "description": "Team for collaborative research",
  "coordination_strategy": "hierarchical",
  "member_ids": ["agent-uuid-1", "agent-uuid-2"]
}
```

**Response:** `201 Created`

---

### Get Team
```http
GET /agents/teams/{team_id}
```

Get team details including members and configuration.

---

### Update Team
```http
PUT /agents/teams/{team_id}
```

Update team configuration.

---

### Delete Team
```http
DELETE /agents/teams/{team_id}
```

Delete an agent team.

---

### Get Team Members
```http
GET /agents/teams/{team_id}/members
```

List all agents in a team.

---

### Get Team Workflows
```http
GET /agents/teams/{team_id}/workflows
```

List active and historical workflows for a team.

---

### Rent Team
```http
POST /agents/teams/{team_id}/rent
```

Rent a published agent team for temporary use.

**Request Body:**
```json
{
  "duration_days": 7,
  "payment_method": "credits"
}
```

---

## Metrics API

### Platform Metrics
```http
GET /agents/metrics
```

Get platform-wide metrics.

**Response:** `200 OK`
```json
{
  "total_agents": 150,
  "active_agents": 45,
  "sessions": {
    "total": 10000,
    "running": 12,
    "completed": 9500,
    "failed": 488
  }
}
```

---

### Agent Metrics
```http
GET /agents/{agent_id}/metrics
```

Get metrics for a specific agent.

**Response:** `200 OK`
```json
{
  "sessions_total": 100,
  "sessions_completed": 95,
  "sessions_failed": 5,
  "tokens_used": 150000,
  "average_duration_seconds": 45.2,
  "last_session_at": "2026-02-21T00:00:00Z"
}
```

---

## Node API (Blockchain)

### Node Status
```http
GET /node/status
```

Get blockchain node status.

**Response:** `200 OK`
```json
{
  "running": true,
  "mode": "full",
  "chain_connected": true,
  "runtime_active": true,
  "indexer_synced": true
}
```

---

### Search Network Agents
```http
GET /node/agents
```

Search for agents published on the decentralized network.

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `category` | string | Filter by category |
| `owner` | string | Filter by owner address |
| `min_trust_tier` | integer | Minimum trust tier (1-5) |
| `max_price` | number | Maximum price per execution |
| `rental_only` | boolean | Only show rentable agents |
| `limit` | integer | Max results |

**Response:** `200 OK`
```json
[
  {
    "dsid": "0x...",
    "name": "Code Analyzer",
    "description": "Analyzes code for issues",
    "owner": "0x...",
    "trust_tier": 4,
    "price_per_execution": 0.001,
    "total_executions": 500
  }
]
```

---

### Execute Network Agent
```http
POST /node/execute
```

Execute an agent from the decentralized network.

**Request Body:**
```json
{
  "dsid": "0x...",
  "input": "Analyze this code...",
  "max_tokens": 4096
}
```

---

## Webhooks API

### Create Trigger
```http
POST /agents/{agent_id}/triggers
```

Create a webhook trigger for an agent.

**Request Body:**
```json
{
  "name": "GitHub Push Trigger",
  "type": "webhook",
  "config": {
    "secret": "your-webhook-secret",
    "goal_template": "Process push event: {{event.commits}}"
  }
}
```

---

### List Triggers
```http
GET /agents/{agent_id}/triggers
```

List all triggers for an agent.

---

### Webhook Endpoint
```http
POST /agents/triggers/webhook/{trigger_id}
```

Receive webhook events to trigger agent execution.

**Headers:**
```
X-Webhook-Signature: sha256=...
Content-Type: application/json
```

---

## Tools API

### List Available Tools
```http
GET /agents/tools
```

List all available tools that agents can use.

**Response:** `200 OK`
```json
[
  {
    "id": "web_search",
    "name": "Web Search",
    "description": "Search the web for information",
    "category": "research"
  },
  {
    "id": "code_exec",
    "name": "Code Execution",
    "description": "Execute code in a sandboxed environment",
    "category": "development"
  }
]
```

---

### Create Custom Tool
```http
POST /agents/tools/custom
```

Create a custom tool for your agents.

**Request Body:**
```json
{
  "name": "My API Tool",
  "description": "Calls my custom API",
  "endpoint": "https://api.example.com/action",
  "method": "POST",
  "headers": {"Authorization": "Bearer {{secret}}"},
  "body_template": {"query": "{{input}}"}
}
```

---

## Templates API

### List Templates
```http
GET /agents/templates
```

List available agent templates.

**Response:** `200 OK`
```json
[
  {
    "id": "research-assistant",
    "name": "Research Assistant",
    "description": "Pre-configured for research tasks",
    "category": "research",
    "tools": ["web_search", "file_access"]
  }
]
```

---

### Instantiate Template
```http
POST /agents/templates/{template_id}/instantiate
```

Create a new agent from a template.

**Request Body:**
```json
{
  "name": "My Research Agent",
  "customizations": {
    "system_prompt_suffix": "Focus on AI topics"
  }
}
```

---

## Error Responses

All endpoints may return the following error responses:

### 400 Bad Request
```json
{
  "detail": "Invalid request body",
  "errors": [{"field": "name", "message": "Name is required"}]
}
```

### 401 Unauthorized
```json
{
  "detail": "Invalid or expired token"
}
```

### 403 Forbidden
```json
{
  "detail": "You do not have permission to access this resource"
}
```

### 404 Not Found
```json
{
  "detail": "Agent not found"
}
```

### 429 Too Many Requests
```json
{
  "detail": "Rate limit exceeded",
  "retry_after": 60
}
```

### 500 Internal Server Error
```json
{
  "detail": "An unexpected error occurred"
}
```

---

## WebSocket Streaming

### Session Stream
```
wss://resonantgenesis.xyz/api/v1/agents/sessions/{session_id}/stream
```

Connect to receive real-time updates during agent execution.

**Message Types:**
```json
{"type": "step_start", "step_id": "...", "action": "web_search"}
{"type": "step_complete", "step_id": "...", "output": "..."}
{"type": "session_complete", "result": "..."}
{"type": "error", "message": "..."}
```

---

## Rate Limits

| Tier | Requests/min | Concurrent Sessions |
|------|--------------|---------------------|
| Free | 60 | 1 |
| Plus | 300 | 5 |
| Enterprise | 1000 | 20 |

---

## SDKs

Official SDKs are available for:
- **Python**: `pip install resonantgenesis`
- **JavaScript/TypeScript**: `npm install @resonantgenesis/sdk`

See [GitHub](https://github.com/louienemesh/ResonantGenesis) for SDK documentation.

---

## Pagination

All list endpoints support pagination using `limit` and `offset` parameters.

### Request Parameters
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | integer | 50 | Maximum number of results (max: 100) |
| `offset` | integer | 0 | Number of results to skip |

### Response Format

Paginated responses include metadata:

```json
{
  "data": [...],
  "pagination": {
    "total": 150,
    "limit": 50,
    "offset": 0,
    "has_more": true
  }
}
```

### Example: Paginating Through Agents

```bash
# First page
curl "https://resonantgenesis.xyz/api/v1/agents?limit=20&offset=0" \
  -H "Authorization: Bearer $TOKEN"

# Second page
curl "https://resonantgenesis.xyz/api/v1/agents?limit=20&offset=20" \
  -H "Authorization: Bearer $TOKEN"
```

---

## API Versioning

The API uses URL-based versioning. The current version is `v1`.

```
https://resonantgenesis.xyz/api/v1/...
```

### Version Lifecycle
- **Current**: `v1` - Stable, production-ready
- **Deprecated**: None
- **Sunset**: None

Breaking changes will result in a new version (e.g., `v2`). Non-breaking additions (new fields, endpoints) are added to the current version.

### Deprecation Policy
- Deprecated endpoints will be announced 6 months before removal
- Deprecation warnings are included in response headers:
  ```
  X-API-Deprecated: true
  X-API-Sunset-Date: 2027-01-01
  ```

---

## Changelog

### v1.3.0 (February 2026)
- **Added**: Agent Teams API for multi-agent collaboration
- **Added**: Team rental functionality
- **Added**: Workflow orchestration endpoints
- **Improved**: Session streaming with more event types

### v1.2.0 (January 2026)
- **Added**: Custom tools API
- **Added**: Webhook triggers for automated execution
- **Added**: Template instantiation endpoint
- **Improved**: Rate limit headers in responses

### v1.1.0 (December 2025)
- **Added**: Blockchain publishing (`/agents/{id}/publish`)
- **Added**: Network agent search and execution
- **Added**: Node status endpoint
- **Improved**: Error response consistency

### v1.0.0 (November 2025)
- Initial API release
- Core agents CRUD operations
- Session management
- Basic metrics endpoints

---

## Support

- **Documentation**: [https://resonantgenesis.xyz/help](https://resonantgenesis.xyz/help)
- **GitHub Issues**: [https://github.com/louienemesh/ResonantGenesis/issues](https://github.com/louienemesh/ResonantGenesis/issues)
- **Email**: support@resonantgenesis.xyz
- **Discord**: [Join our community](https://discord.gg/resonantgenesis)
