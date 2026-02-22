# Smart Contracts Documentation

Complete documentation for ResonantGenesis blockchain smart contracts.

## Overview

ResonantGenesis uses a hybrid blockchain architecture:

- **Ethereum/Base (External)**: Used **ONLY** for identity verification via DSIDs
- **Internal Chain**: Used for agent logging, memory anchoring, and operational data

| Contract | Purpose | Network |
|----------|---------|---------|
| IdentityRegistry | Decentralized Semantic Identities (DSIDs) | Base Mainnet (External) |
| AgentRegistry | On-chain agent registration | Internal Chain |
| MemoryAnchors | Immutable memory timestamps | Internal Chain |

> **Note**: Only the IdentityRegistry contract is deployed on the public Ethereum L2 (Base). AgentRegistry and MemoryAnchors operate on ResonantGenesis's internal blockchain for cost efficiency and privacy.

## Contract Addresses

### External Chain (Base Mainnet)

| Contract | Address | Purpose |
|----------|---------|---------|
| IdentityRegistry | `0x...` (TBD) | DSID identity verification |

### External Chain (Base Sepolia Testnet)

| Contract | Address | Purpose |
|----------|---------|---------|
| IdentityRegistry | `0x...` (TBD) | DSID identity verification (testnet) |

### Internal Chain

AgentRegistry and MemoryAnchors are deployed on ResonantGenesis's internal blockchain. These contracts are not publicly accessible but follow the same interface documented below.

---

## IdentityRegistry

Registry for Decentralized Semantic Identities (DSIDs).

### Purpose

- Register unique identities on-chain
- Associate public keys with identities
- Manage identity status (Active, Suspended, Revoked)
- Transfer ownership of identities

### Data Structures

```solidity
enum Status { Active, Suspended, Revoked }

struct Identity {
    address owner;      // Wallet that owns this identity
    bytes publicKey;    // Public key for verification
    Status status;      // Current status
    uint256 createdAt;  // Registration timestamp
}
```

### Functions

#### registerIdentity

Register a new identity with a public key.

```solidity
function registerIdentity(bytes32 dsid, bytes calldata publicKey) external
```

**Parameters:**
- `dsid`: The distributed semantic ID (keccak256 hash)
- `publicKey`: The public key associated with this identity

**Events:**
```solidity
event IdentityRegistered(bytes32 indexed dsid, address indexed owner, uint256 createdAt)
```

**Example:**
```javascript
const dsid = ethers.keccak256(ethers.toUtf8Bytes("my-unique-identity"));
const publicKey = "0x04..."; // Your public key bytes

await identityRegistry.registerIdentity(dsid, publicKey);
```

#### getIdentity

Get identity details.

```solidity
function getIdentity(bytes32 dsid) external view returns (
    address owner,
    bytes memory publicKey,
    uint8 status,
    uint256 createdAt
)
```

**Returns:**
- `owner`: The identity owner address
- `publicKey`: The public key bytes
- `status`: The identity status (0=Active, 1=Suspended, 2=Revoked)
- `createdAt`: The registration timestamp

#### isRegistered

Check if an identity is registered.

```solidity
function isRegistered(bytes32 dsid) external view returns (bool)
```

#### updateStatus

Update identity status (owner only).

```solidity
function updateStatus(bytes32 dsid, Status newStatus) external
```

**Status Values:**
- `0` = Active
- `1` = Suspended
- `2` = Revoked

#### transferOwnership

Transfer identity ownership to a new address.

```solidity
function transferOwnership(bytes32 dsid, address newOwner) external
```

### Errors

| Error | Description |
|-------|-------------|
| `AlreadyRegistered()` | DSID is already registered |
| `NotRegistered()` | DSID does not exist |
| `NotOwner()` | Caller is not the owner |
| `InvalidPublicKey()` | Public key is empty |

---

## AgentRegistry

Registry for AI agents with on-chain identity.

> **Deployment**: This contract is deployed on ResonantGenesis's **internal blockchain**, not on public Ethereum. The API abstracts blockchain interactions.

### Purpose

- Register agents with manifest hashes
- Store metadata URIs (IPFS, HTTP)
- Track agent ownership
- Manage agent status (Active, Paused, Deprecated)

### Data Structures

```solidity
enum Status { Active, Paused, Deprecated }

struct Agent {
    address owner;        // Wallet that owns this agent
    string metadataUri;   // URI to agent metadata (IPFS, HTTP)
    Status status;        // Current status
    uint256 registeredAt; // Registration timestamp
}
```

### Functions

#### registerAgent

Register a new agent.

```solidity
function registerAgent(bytes32 manifestHash, string calldata metadataUri) external
```

**Parameters:**
- `manifestHash`: The keccak256 hash of the agent manifest
- `metadataUri`: URI pointing to agent metadata (IPFS, HTTP, etc.)

**Events:**
```solidity
event AgentRegistered(bytes32 indexed manifestHash, address indexed owner, string metadataUri, uint256 registeredAt)
```

**Example:**
```javascript
const manifest = {
  name: "Research Assistant",
  version: "1.0.0",
  systemPrompt: "You are a helpful research assistant.",
  tools: ["web_search"]
};

const manifestHash = ethers.keccak256(ethers.toUtf8Bytes(JSON.stringify(manifest)));
const metadataUri = "ipfs://QmXxx...";

await agentRegistry.registerAgent(manifestHash, metadataUri);
```

#### getAgent

Get agent details.

```solidity
function getAgent(bytes32 manifestHash) external view returns (
    address owner,
    string memory metadataUri,
    uint8 status,
    uint256 registeredAt
)
```

#### isRegistered

Check if an agent is registered.

```solidity
function isRegistered(bytes32 manifestHash) external view returns (bool)
```

#### getAgentsByOwner

Get all agents owned by an address.

```solidity
function getAgentsByOwner(address owner) external view returns (bytes32[] memory)
```

#### updateStatus

Update agent status (owner only).

```solidity
function updateStatus(bytes32 manifestHash, Status newStatus) external
```

**Status Values:**
- `0` = Active
- `1` = Paused
- `2` = Deprecated

#### updateMetadata

Update agent metadata URI (owner only).

```solidity
function updateMetadata(bytes32 manifestHash, string calldata newMetadataUri) external
```

#### transferOwnership

Transfer agent ownership to a new address.

```solidity
function transferOwnership(bytes32 manifestHash, address newOwner) external
```

### Errors

| Error | Description |
|-------|-------------|
| `AlreadyRegistered()` | Agent is already registered |
| `NotRegistered()` | Agent does not exist |
| `NotOwner()` | Caller is not the owner |
| `InvalidMetadataUri()` | Metadata URI is empty |

---

## MemoryAnchors

Anchors content hashes on-chain for verifiable timestamps.

> **Deployment**: This contract is deployed on ResonantGenesis's **internal blockchain**, not on public Ethereum. The API abstracts blockchain interactions.

### Purpose

- Create immutable proof of existence
- Timestamp agent memories on-chain
- Verify content existed before a given time
- Track memory ownership

### Data Structures

```solidity
struct Anchor {
    address owner;       // Wallet that created the anchor
    uint256 timestamp;   // Block timestamp when anchored
    uint256 blockNumber; // Block number when anchored
}
```

### Functions

#### anchor

Anchor a content hash on-chain.

```solidity
function anchor(bytes32 contentHash) external
```

**Parameters:**
- `contentHash`: The keccak256 hash of the content

**Events:**
```solidity
event Anchored(bytes32 indexed contentHash, address indexed owner, uint256 timestamp, uint256 blockNumber)
```

**Example:**
```javascript
const memory = {
  sessionId: "session-123",
  context: "User asked about quantum computing",
  timestamp: Date.now()
};

const contentHash = ethers.keccak256(ethers.toUtf8Bytes(JSON.stringify(memory)));
await memoryAnchors.anchor(contentHash);
```

#### getAnchor

Get anchor details.

```solidity
function getAnchor(bytes32 contentHash) external view returns (
    address owner,
    uint256 timestamp,
    uint256 blockNumber
)
```

#### isAnchored

Check if content is anchored.

```solidity
function isAnchored(bytes32 contentHash) external view returns (bool)
```

#### getAnchorsByOwner

Get all anchors by owner.

```solidity
function getAnchorsByOwner(address owner) external view returns (bytes32[] memory)
```

#### verifyBefore

Verify content was anchored before a timestamp.

```solidity
function verifyBefore(bytes32 contentHash, uint256 beforeTimestamp) external view returns (bool)
```

**Example:**
```javascript
// Verify memory existed before a specific time
const wasAnchored = await memoryAnchors.verifyBefore(
  contentHash,
  Math.floor(Date.now() / 1000) - 86400 // 24 hours ago
);
```

### Errors

| Error | Description |
|-------|-------------|
| `AlreadyAnchored()` | Content is already anchored |
| `NotAnchored()` | Content is not anchored |

---

## Integration Examples

### JavaScript/TypeScript (ethers.js)

```typescript
import { ethers } from 'ethers';

// Connect to Base
const provider = new ethers.JsonRpcProvider('https://mainnet.base.org');
const signer = new ethers.Wallet(privateKey, provider);

// Contract ABIs (simplified)
const identityRegistryAbi = [
  "function registerIdentity(bytes32 dsid, bytes publicKey) external",
  "function getIdentity(bytes32 dsid) view returns (address, bytes, uint8, uint256)",
  "function isRegistered(bytes32 dsid) view returns (bool)"
];

// Connect to contract
const identityRegistry = new ethers.Contract(
  IDENTITY_REGISTRY_ADDRESS,
  identityRegistryAbi,
  signer
);

// Register an identity
async function registerIdentity(name: string, publicKey: string) {
  const dsid = ethers.keccak256(ethers.toUtf8Bytes(name));
  const tx = await identityRegistry.registerIdentity(dsid, publicKey);
  await tx.wait();
  return dsid;
}

// Check if registered
async function checkIdentity(dsid: string) {
  const isRegistered = await identityRegistry.isRegistered(dsid);
  if (isRegistered) {
    const [owner, publicKey, status, createdAt] = await identityRegistry.getIdentity(dsid);
    return { owner, publicKey, status, createdAt };
  }
  return null;
}
```

### Python (web3.py)

```python
from web3 import Web3

# Connect to Base
w3 = Web3(Web3.HTTPProvider('https://mainnet.base.org'))

# Contract ABI (simplified)
agent_registry_abi = [...]

# Connect to contract
agent_registry = w3.eth.contract(
    address=AGENT_REGISTRY_ADDRESS,
    abi=agent_registry_abi
)

# Register an agent
def register_agent(manifest: dict, metadata_uri: str, private_key: str):
    manifest_json = json.dumps(manifest, sort_keys=True)
    manifest_hash = w3.keccak(text=manifest_json)
    
    account = w3.eth.account.from_key(private_key)
    
    tx = agent_registry.functions.registerAgent(
        manifest_hash,
        metadata_uri
    ).build_transaction({
        'from': account.address,
        'nonce': w3.eth.get_transaction_count(account.address),
        'gas': 200000,
        'gasPrice': w3.eth.gas_price
    })
    
    signed = account.sign_transaction(tx)
    tx_hash = w3.eth.send_raw_transaction(signed.rawTransaction)
    receipt = w3.eth.wait_for_transaction_receipt(tx_hash)
    
    return manifest_hash.hex()
```

---

## Security Considerations

### Access Control

- Only owners can modify their identities/agents
- Ownership transfer is explicit and logged
- Status changes emit events for monitoring

### Gas Optimization

- Mappings used for O(1) lookups
- Minimal storage per record
- Events for off-chain indexing

### Best Practices

1. **Verify before trusting**: Always check `isRegistered()` before using data
2. **Monitor events**: Index events for efficient queries
3. **Use checksums**: Verify addresses are checksummed
4. **Test on Sepolia**: Always test on testnet first

---

## Deployment

### Prerequisites

```bash
npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox
```

### Deploy Script

```javascript
const { ethers } = require("hardhat");

async function main() {
  // Deploy IdentityRegistry
  const IdentityRegistry = await ethers.getContractFactory("IdentityRegistry");
  const identityRegistry = await IdentityRegistry.deploy();
  await identityRegistry.waitForDeployment();
  console.log("IdentityRegistry:", await identityRegistry.getAddress());

  // Deploy AgentRegistry
  const AgentRegistry = await ethers.getContractFactory("AgentRegistry");
  const agentRegistry = await AgentRegistry.deploy();
  await agentRegistry.waitForDeployment();
  console.log("AgentRegistry:", await agentRegistry.getAddress());

  // Deploy MemoryAnchors
  const MemoryAnchors = await ethers.getContractFactory("MemoryAnchors");
  const memoryAnchors = await MemoryAnchors.deploy();
  await memoryAnchors.waitForDeployment();
  console.log("MemoryAnchors:", await memoryAnchors.getAddress());
}

main().catch(console.error);
```

### Hardhat Config

```javascript
require("@nomicfoundation/hardhat-toolbox");

module.exports = {
  solidity: "0.8.19",
  networks: {
    "base-sepolia": {
      url: "https://sepolia.base.org",
      accounts: [process.env.PRIVATE_KEY]
    },
    "base-mainnet": {
      url: "https://mainnet.base.org",
      accounts: [process.env.PRIVATE_KEY]
    }
  }
};
```

---

## Audit Status

| Contract | Status | Auditor |
|----------|--------|---------|
| IdentityRegistry | Pending | - |
| AgentRegistry | Pending | - |
| MemoryAnchors | Pending | - |

---

## Resources

- **Source Code**: [resonant-contracts](https://github.com/louienemesh/ResonantGenesis/tree/main/contracts)
- **Base Documentation**: [docs.base.org](https://docs.base.org)
- **Ethers.js**: [docs.ethers.org](https://docs.ethers.org)
- **Hardhat**: [hardhat.org](https://hardhat.org)

---

**Questions?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
