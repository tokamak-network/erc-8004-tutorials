# 2.1 Identity Registry - Agent Discovery

## What is the Identity Registry?

It's an **ERC-721 (NFT) contract** where each agent is a token. When you register an agent:

1. You call `register(agentURI, metadata)`
2. An NFT is minted to your wallet
3. The token ID becomes your `agentId`
4. The `agentURI` points to your agent's registration file

## Simple Analogy

Think of it like **getting a passport**:
- Your passport has a unique number (agentId)
- It contains your details (agentURI → registration file)
- It's yours - you own it (NFT ownership)
- You can update your address but the passport number stays same

## The Agent Registration File

Your `agentURI` points to a JSON file (hosted anywhere - IPFS, your server, etc.):

```json
{
  "name": "Market Analysis Agent",
  "description": "I analyze crypto markets and provide insights",
  "version": "1.0.0",
  "capabilities": ["market-analysis", "trend-prediction"],
  "endpoints": {
    "a2a": "https://myagent.com/a2a",
    "mcp": "https://myagent.com/mcp",
    "web": "https://myagent.com"
  },
  "wallet": "0x1234...5678"
}
```

### What Each Field Means

| Field | Purpose |
|-------|---------|
| `name` | Human-readable name |
| `description` | What your agent does |
| `version` | Track updates to your agent |
| `capabilities` | Tags for discovery (searchable) |
| `endpoints` | Where to communicate with your agent |
| `wallet` | Address for receiving payments |

## Key Functions

```solidity
// Register a new agent (mints NFT to your wallet)
function register(string agentURI, bytes metadata) returns (uint256 agentId)

// Update your agent's URI (new capabilities, endpoints, etc.)
function setAgentURI(uint256 agentId, string newURI)

// Update agent's wallet address (for payments)
function setAgentWallet(uint256 agentId, address newWallet)

// Read agent metadata
function getMetadata(uint256 agentId) returns (bytes)

// Standard ERC-721: Who owns this agent?
function ownerOf(uint256 agentId) returns (address)
```

## Key Events

```solidity
// Emitted when new agent is registered
event Registered(uint256 indexed agentId, address indexed owner, string agentURI);

// Emitted when agent URI is updated
event AgentURIUpdated(uint256 indexed agentId, string newURI);

// Emitted when agent wallet is updated
event AgentWalletUpdated(uint256 indexed agentId, address newWallet);
```

## Why NFT-Based Identity?

| Benefit | Explanation |
|---------|-------------|
| **Ownership** | You cryptographically own your agent |
| **Transferable** | Can sell/transfer agent WITH its reputation intact |
| **Interoperable** | Works with wallets, marketplaces, any NFT tool |
| **Permanent** | Can't be deleted by anyone (censorship resistant) |
| **Composable** | Other contracts can check ownership, build on top |

## Workflow: Registering an Agent

```
You (wallet: 0xABC...)
    │
    │ 1. Call register("ipfs://Qm.../agent.json", metadata)
    ▼
┌─────────────────────────┐
│   Identity Registry     │
│                         │
│ 2. Mint NFT #42 to you  │
│ 3. Store agentURI       │
│ 4. Emit Registered()    │
└─────────────────────────┘
    │
    ▼
You now own Agent #42!
- agentId = 42
- ownerOf(42) = 0xABC...
- tokenURI(42) = "ipfs://Qm.../agent.json"
```

## Key Takeaways

1. **One agent = One NFT** - Your agent ID is a token you own
2. **agentURI is crucial** - It tells the world how to interact with your agent
3. **You control updates** - Only owner can change URI or wallet
4. **Transferable** - Agent ownership (and reputation) can be sold

---

## Quick Quiz

1. What standard is the Identity Registry based on?
2. What happens when you call `register()`?
3. Where is the agent's capability information stored?
4. Can you transfer your agent to someone else? Why would you?
