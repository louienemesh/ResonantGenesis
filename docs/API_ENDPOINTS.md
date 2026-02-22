# ResonantGenesis API Reference

## Base URLs

| Environment | URL |
|-------------|-----|
| Production | `https://dev-swat.com/api/v1` |
| Node API | `https://dev-swat.com/api/v1/node` |

## Authentication

All authenticated endpoints require a Bearer token:
```
Authorization: Bearer <jwt_token>
```

---

## Authentication Endpoints

### POST /auth/login
Authenticate user and receive JWT token.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "secure_password"
}
```

**Response:**
```json
{
  "access_token": "eyJ...",
  "token_type": "bearer",
  "expires_in": 3600,
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "trust_tier": 2
  }
}
```

### POST /auth/register
Create new user account.

### POST /auth/refresh
Refresh expired access token.

### POST /auth/logout
Invalidate current session.

---

## Identity Endpoints

### GET /identity/{dsid}
Get identity details by DSID.

**Response:**
```json
{
  "dsid": "did:rg:0x...",
  "owner": "0x...",
  "status": "active",
  "trust_tier": 3,
  "created_at": "2024-01-15T10:30:00Z"
}
```

### POST /identity/register
Register new decentralized identity.

### PATCH /identity/{dsid}/status
Update identity status (requires ownership).

---

## Agent Endpoints

### GET /agents
List all agents for authenticated user.

**Query Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| status | string | Filter by status (active, paused, deprecated) |
| category | string | Filter by category |
| limit | int | Max results (default: 20) |
| offset | int | Pagination offset |

**Response:**
```json
{
  "agents": [
    {
      "id": "uuid",
      "name": "Research Assistant",
      "description": "AI research helper",
      "status": "active",
      "category": "research",
      "execution_count": 150,
      "created_at": "2024-01-10T08:00:00Z"
    }
  ],
  "total": 5,
  "limit": 20,
  "offset": 0
}
```

### GET /agents/{id}
Get single agent details.

### POST /agents
Create new agent.

**Request:**
```json
{
  "name": "My Agent",
  "description": "Agent description",
  "category": "utility",
  "system_prompt": "You are a helpful assistant...",
  "capabilities": ["web_search", "code_execution"],
  "trust_requirements": 1
}
```

### PATCH /agents/{id}
Update agent configuration.

### DELETE /agents/{id}
Delete agent (soft delete).

### POST /agents/{id}/execute
Execute agent with input.

**Request:**
```json
{
  "input": "Research the latest AI papers",
  "context": {},
  "session_id": "optional-session-uuid"
}
```

**Response:**
```json
{
  "execution_id": "uuid",
  "status": "completed",
  "output": "Here are the latest AI papers...",
  "tokens_used": 1500,
  "duration_ms": 3200,
  "cost": 0.0015
}
```

### GET /agents/{id}/metrics
Get agent performance metrics.

**Response:**
```json
{
  "total_executions": 150,
  "successful": 145,
  "failed": 5,
  "avg_duration_ms": 2800,
  "total_tokens": 225000,
  "total_cost": 0.225
}
```

---

## Webhook Endpoints

### POST /webhooks/trigger
Generic webhook trigger.

**Request:**
```json
{
  "event_type": "custom_event",
  "payload": {},
  "signature": "hmac-sha256-signature"
}
```

### POST /webhooks/agent/{agent_id}
Agent-specific webhook trigger.

### POST /webhooks/github
GitHub webhook handler.

**Headers:**
```
X-Hub-Signature-256: sha256=...
X-GitHub-Event: push
```

---

## Node API Endpoints

### GET /node/status
Get blockchain node status.

**Response:**
```json
{
  "running": true,
  "mode": "full",
  "identity": "did:rg:0x...",
  "chain_connected": true,
  "runtime_active": true,
  "indexer_synced": true,
  "peer_count": 12,
  "block_height": 1234567
}
```

### GET /node/health
Health check endpoint.

### GET /node/agents
Search agents on the network.

**Query Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| category | string | Filter by category |
| min_trust_tier | int | Minimum trust tier |
| max_price | float | Maximum price per execution |
| limit | int | Max results |

**Response:**
```json
{
  "agents": [
    {
      "manifest_hash": "0x...",
      "name": "Network Agent",
      "version": "1.0.0",
      "description": "Description",
      "category": "utility",
      "trust_tier": 3,
      "execution_count": 500,
      "price_per_execution": 0.001
    }
  ],
  "count": 25
}
```

### POST /node/agents/publish
Publish agent to network.

**Request:**
```json
{
  "name": "My Network Agent",
  "description": "Agent for the network",
  "category": "automation",
  "manifest": {
    "version": "1.0.0",
    "capabilities": ["web_search"],
    "trust_requirements": 2
  },
  "price_per_execution": 0.001
}
```

**Response:**
```json
{
  "success": true,
  "manifest_hash": "0x...",
  "tx_hash": "0x...",
  "published_at": "2024-01-15T12:00:00Z"
}
```

### POST /node/execute
Execute agent on the network.

**Request:**
```json
{
  "manifest_hash": "0x...",
  "input_data": {
    "query": "Execute this task"
  },
  "user_dsid": "did:rg:0x...",
  "trust_tier": 2
}
```

### GET /node/executions
Get execution history.

### GET /node/executions/{id}
Get execution status.

### POST /node/executions/{id}/cancel
Cancel running execution.

### GET /node/network/stats
Get network statistics.

**Response:**
```json
{
  "total_nodes": 50,
  "active_nodes": 45,
  "total_agents": 200,
  "total_executions": 15000,
  "network_tps": 12.5,
  "avg_latency_ms": 150
}
```

### GET /node/network/nodes
Discover network nodes.

### GET /node/network/ping/{node_id}
Ping specific node.

---

## Error Responses

All errors follow this format:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input provided",
    "details": {
      "field": "email",
      "reason": "Invalid email format"
    }
  }
}
```

### Error Codes
| Code | HTTP Status | Description |
|------|-------------|-------------|
| UNAUTHORIZED | 401 | Missing or invalid auth token |
| FORBIDDEN | 403 | Insufficient permissions |
| NOT_FOUND | 404 | Resource not found |
| VALIDATION_ERROR | 400 | Invalid request data |
| RATE_LIMITED | 429 | Too many requests |
| INTERNAL_ERROR | 500 | Server error |

---

## Rate Limits

| Tier | Requests/min | Executions/day |
|------|--------------|----------------|
| Free | 60 | 100 |
| Plus | 300 | 1000 |
| Enterprise | 1000 | Unlimited |

Rate limit headers:
```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1705312800
```

---

## WebSocket API

### Connection
```
wss://dev-swat.com/ws?token=<jwt_token>
```

### Events
| Event | Direction | Description |
|-------|-----------|-------------|
| execution.started | Server→Client | Execution began |
| execution.progress | Server→Client | Progress update |
| execution.completed | Server→Client | Execution finished |
| execution.failed | Server→Client | Execution error |

### Example
```javascript
const ws = new WebSocket('wss://dev-swat.com/ws?token=...');
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log(data.type, data.payload);
};
```
