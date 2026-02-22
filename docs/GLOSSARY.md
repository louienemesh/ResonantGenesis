# Glossary

Key terms and definitions used throughout the ResonantGenesis platform.

## A

### Agent
An AI-powered autonomous entity that can perform tasks, make decisions, and interact with tools and other agents. Agents are configured with system prompts, models, and tool access.

### Agent Registry
A smart contract that stores on-chain records of registered agents, including their manifest hashes, metadata URIs, and ownership information.

### Agent Studio
The web interface for creating, configuring, and managing AI agents. Formerly called "Agents" in the navigation.

### Agent Teams
A feature allowing multiple agents to collaborate on complex tasks using various coordination strategies (hierarchical, sequential, parallel, collaborative).

### Anchor
See [Memory Anchor](#memory-anchor).

## B

### Base
An Ethereum Layer 2 blockchain built on the OP Stack. ResonantGenesis uses Base for low-cost, fast blockchain transactions.

### Blockchain Node
A service that connects to the Base blockchain network, enabling agent registration, memory anchoring, and decentralized identity verification.

## C

### Coordination Mode
The strategy used by Agent Teams to organize work. Options include:
- **Hierarchical**: Lead agent delegates to others
- **Sequential**: Agents work in order
- **Parallel**: Agents work simultaneously
- **Collaborative**: Agents work as peers

### Control Center
The dashboard for monitoring and managing platform operations. Includes monitoring, analytics, governance, and security tools.

## D

### DSID (Decentralized Semantic Identity)
A unique, blockchain-verified identity for agents and users. DSIDs provide:
- Immutable proof of ownership
- Verifiable execution history
- Trust scoring
- Marketplace listing capability

### Delegation
The process of assigning subtasks from one agent to another within a team.

## E

### Execution
A single run of an agent performing a task. Executions are tracked with metrics like duration, tokens used, and success status.

### Execution Proof
A cryptographic proof stored on-chain that verifies an agent execution occurred with specific inputs and outputs.

## G

### Goal
The objective given to an agent for a session. Goals define what the agent should accomplish.

### Governance
The system for managing platform rules, policies, and community decisions. T4 (Governance) tier users can vote on protocol changes.

## H

### Hash Sphere Memory
The AI memory infrastructure system that stores and retrieves agent context across sessions using semantic hashing.

## I

### Identity Registry
A smart contract that manages Decentralized Semantic Identities (DSIDs), storing owner addresses, public keys, and status information.

### Invariant
A condition that must always be true during agent execution. Used by State Physics for safety enforcement.

## K

### Knowledge Graph
A structured representation of information that agents can query and update, connecting concepts, entities, and relationships.

## M

### Manifest
A JSON document describing an agent's configuration, including name, system prompt, model, tools, and capabilities. The manifest hash is used for on-chain registration.

### Manifest Hash
A keccak256 hash of an agent's manifest, used as a unique identifier for blockchain registration.

### Marketplace
The platform section for discovering, purchasing, and renting agents, templates, and workflows. Formerly called "General Store."

### Memory Anchor
An on-chain record that timestamps content hashes, providing immutable proof that specific data existed at a given time.

### Memory System
The infrastructure for storing and retrieving agent context, including episodic memory (events), semantic memory (facts), and procedural memory (skills).

### Metadata URI
A URI (typically IPFS or HTTP) pointing to detailed agent metadata stored off-chain.

## N

### Network
The decentralized network of agents and nodes. Users can publish agents to the network and execute agents from other publishers.

### Node
A server running the ResonantGenesis blockchain node software, participating in the decentralized network.

## P

### Publisher
A T3 tier user who has staked ETH and can publish agents to the decentralized marketplace.

### Public Key
A cryptographic key associated with an identity, used for verification and signing.

## R

### Rate Limit
Restrictions on API request frequency based on user tier:
- Free: 60 requests/min
- Plus: 300 requests/min
- Enterprise: 1000 requests/min

### Resonant Chat
The conversational AI interface for interacting with agents.

### Resonant IDE
The in-browser integrated development environment for building and testing agents.

## S

### Sandbox
An isolated execution environment for running agent code safely, with resource limits and security restrictions.

### Session
A single interaction period with an agent, from start to completion. Sessions track goals, steps, and results.

### Staking
The process of locking ETH to become a T3 Publisher, demonstrating commitment to the platform.

### State Physics
The invariant enforcement API that ensures agents operate within defined safety boundaries.

### Step
A single action taken by an agent during a session, such as a tool call or decision.

### System Prompt
Instructions that define an agent's personality, capabilities, and behavior. The foundation of agent configuration.

## T

### Template
A pre-configured agent setup that can be instantiated to create new agents quickly.

### Token
A unit of text processed by AI models. Usage is measured in input and output tokens.

### Tool
A capability that extends what an agent can do, such as web search, code execution, or API calls.

### Trigger
An event that automatically starts an agent execution, such as a webhook, schedule, or external event.

### Trust Score
A numerical rating (0-100) indicating the trustworthiness of a user or agent, based on verification, activity, and reputation.

### Trust Tier
A level in the trust system (T0-T4) that determines user capabilities:
- **T0**: Unverified - Read-only access
- **T1**: Verified - Basic agent execution
- **T2**: Trusted - Full agent access
- **T3**: Publisher - Network publishing
- **T4**: Governance - Protocol voting

## W

### Webhook
An HTTP callback that triggers agent execution when external events occur (e.g., GitHub push, Slack message).

### Workflow
A sequence of agent actions or team collaborations designed to accomplish a complex task.

### Workspace
The file system area where agents can read and write files during execution.

## Abbreviations

| Abbreviation | Full Term |
|--------------|-----------|
| API | Application Programming Interface |
| CRUD | Create, Read, Update, Delete |
| DSID | Decentralized Semantic Identity |
| ETH | Ethereum (cryptocurrency) |
| HTTP | Hypertext Transfer Protocol |
| IPFS | InterPlanetary File System |
| JWT | JSON Web Token |
| KYC | Know Your Customer |
| L2 | Layer 2 (blockchain scaling) |
| SDK | Software Development Kit |
| SSL | Secure Sockets Layer |
| TLS | Transport Layer Security |
| URI | Uniform Resource Identifier |
| URL | Uniform Resource Locator |
| UX | User Experience |
| WS | WebSocket |

---

**Missing a term?** [Open an issue](https://github.com/louienemesh/ResonantGenesis/issues) or join our [Discord](https://discord.gg/resonantgenesis) to suggest additions.
