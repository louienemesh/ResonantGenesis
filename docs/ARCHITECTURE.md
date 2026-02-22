# ResonantGenesis Architecture

## Overview

ResonantGenesis is a decentralized AI agent platform built on blockchain technology. This document describes the system architecture, component interactions, and data flows.

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              USER LAYER                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│  Web Frontend (React)  │  Mobile App  │  CLI Tools  │  SDK/API Clients      │
└────────────┬────────────────────┬────────────────────┬──────────────────────┘
             │                    │                    │
             ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           API GATEWAY LAYER                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│  Nginx Reverse Proxy  │  Rate Limiting  │  SSL/TLS  │  Load Balancing       │
└────────────┬────────────────────────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          APPLICATION LAYER                                   │
├──────────────────────┬──────────────────────┬───────────────────────────────┤
│   FastAPI Backend    │   Agent Engine       │   Blockchain Node             │
│   - Auth/Identity    │   - Execution        │   - P2P Network               │
│   - User Management  │   - Orchestration    │   - Agent Registry            │
│   - Webhooks         │   - Memory System    │   - Execution Proofs          │
│   - Analytics        │   - Tool Integration │   - Trust Scoring             │
└──────────┬───────────┴──────────┬───────────┴───────────┬───────────────────┘
           │                      │                       │
           ▼                      ▼                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            DATA LAYER                                        │
├──────────────────────┬──────────────────────┬───────────────────────────────┤
│   PostgreSQL         │   Redis              │   IPFS / Arweave              │
│   - User Data        │   - Session Cache    │   - Agent Manifests           │
│   - Agent Configs    │   - Rate Limits      │   - Execution Logs            │
│   - Execution Logs   │   - Pub/Sub          │   - Memory Anchors            │
└──────────────────────┴──────────────────────┴───────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         BLOCKCHAIN LAYER                                     │
├──────────────────────┬──────────────────────┬───────────────────────────────┤
│   Base Mainnet       │   Base Sepolia       │   Ethereum Mainnet            │
│   - IdentityRegistry │   (Testnet)          │   - Root Identity Anchors     │
│   - AgentRegistry    │   - Testing          │   - Cross-chain Proofs        │
│   - MemoryAnchors    │   - Development      │                               │
└──────────────────────┴──────────────────────┴───────────────────────────────┘
```

## Core Components

### 1. Frontend (React + TypeScript)

The web frontend provides the user interface for:
- **AgentOS**: Agent creation, configuration, and management
- **Network Browser**: Discover and execute agents on the decentralized network
- **Workflow Designer**: Create multi-agent workflows
- **Dashboard**: Monitor executions, metrics, and costs

**Key Technologies:**
- React 18 with TypeScript
- Vite for build tooling
- TailwindCSS for styling
- Zustand for state management

### 2. Backend API (FastAPI + Python)

The backend handles:
- **Authentication**: JWT-based auth with OAuth2 support
- **User Management**: Profiles, API keys, subscriptions
- **Webhooks**: External integrations (GitHub, Slack, etc.)
- **Agent Lifecycle**: CRUD operations for agent configurations

**Key Endpoints:**
```
POST /api/v1/auth/login
POST /api/v1/auth/register
GET  /api/v1/agents
POST /api/v1/agents/{id}/execute
POST /api/v1/webhooks/trigger
```

### 3. Agent Engine

The execution runtime for AI agents:
- **Orchestrator**: Manages agent execution flow
- **Memory System**: Persistent context across sessions
- **Tool Integration**: External API calls, code execution
- **Governance**: Trust-based execution policies

**Execution Flow:**
```
1. Request received → Validate trust tier
2. Load agent manifest → Initialize context
3. Execute steps → Apply governance rules
4. Store results → Anchor to blockchain (optional)
5. Return response → Update metrics
```

### 4. Blockchain Node

Decentralized network node providing:
- **P2P Discovery**: Find other nodes and agents
- **Agent Registry**: On-chain agent metadata
- **Execution Proofs**: Verifiable computation records
- **Trust Scoring**: Reputation-based agent ranking

**API Endpoints:**
```
GET  /api/v1/node/status
GET  /api/v1/node/agents
POST /api/v1/node/agents/publish
POST /api/v1/node/execute
GET  /api/v1/node/network/stats
```

## Smart Contracts

### IdentityRegistry.sol
Manages decentralized semantic identities (DSIDs):
- Register identity with public key
- Update status (Active, Suspended, Revoked)
- Transfer ownership

### AgentRegistry.sol
On-chain agent registration:
- Register agent with metadata URI
- Update status (Active, Paused, Deprecated)
- Track execution counts

### MemoryAnchors.sol
Immutable memory anchoring:
- Anchor content hashes with timestamps
- Verify existence before a given time
- Prove agent memory integrity

## Data Flow

### Agent Execution
```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Client  │───▶│  API GW  │───▶│  Engine  │───▶│  Agent   │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
                                      │               │
                                      ▼               ▼
                               ┌──────────┐    ┌──────────┐
                               │  Memory  │    │  Tools   │
                               └──────────┘    └──────────┘
                                      │
                                      ▼
                               ┌──────────┐
                               │Blockchain│
                               └──────────┘
```

### Network Discovery
```
┌──────────┐    ┌──────────┐    ┌──────────┐
│  Node A  │◀──▶│  Node B  │◀──▶│  Node C  │
└──────────┘    └──────────┘    └──────────┘
     │               │               │
     └───────────────┼───────────────┘
                     ▼
              ┌──────────────┐
              │ Agent Index  │
              └──────────────┘
```

## Security Model

### Trust Tiers
| Tier | Description | Capabilities |
|------|-------------|--------------|
| T0 | Unverified | Read-only, limited API |
| T1 | Email verified | Basic agent execution |
| T2 | Identity verified | Full agent access |
| T3 | Staked identity | Network publishing |
| T4 | Governance member | Protocol voting |

### Authentication Flow
1. User authenticates via OAuth2 or email/password
2. JWT issued with trust tier claims
3. API validates token on each request
4. Blockchain operations require wallet signature

## Deployment Architecture

### Production (dev-swat.com)
```
┌─────────────────────────────────────────┐
│           DigitalOcean Droplet          │
├─────────────────────────────────────────┤
│  Nginx (SSL termination, routing)       │
│    ├── /api/* → FastAPI (port 8000)     │
│    ├── /api/v1/node/* → Node (8081)     │
│    └── /* → Static frontend             │
├─────────────────────────────────────────┤
│  Docker Containers:                     │
│    - fastapi-backend                    │
│    - blockchain-node                    │
│    - postgres                           │
│    - redis                              │
└─────────────────────────────────────────┘
```

### CI/CD Pipeline
```
GitHub Push → GitHub Actions → Build → Test → Deploy to Droplet
```

## API Reference

See [API_ENDPOINTS.md](./API_ENDPOINTS.md) for complete API documentation.

## Contributing

See [CONTRIBUTING.md](../CONTRIBUTING.md) for development setup and guidelines.

## License

MIT License - see [LICENSE](../LICENSE) for details.
