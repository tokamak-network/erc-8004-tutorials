# What is ERC-8004?

## The Problem It Solves

Imagine you have AI agents that can do tasks - analyze markets, write code, verify data. Now imagine thousands of these agents, built by different people, needing to work together **without knowing or trusting each other**.

**How do you:**
- Know which agent is which? (Identity)
- Know if an agent does good work? (Reputation)
- Verify an agent actually did what it claims? (Validation)

**Without ERC-8004:** You'd need centralized services, off-chain databases, manual trust relationships. This doesn't scale.

**With ERC-8004:** Three simple on-chain contracts that any agent can use to:
1. **Prove who they are** (Identity Registry)
2. **Build reputation** (Reputation Registry)
3. **Get work verified** (Validation Registry)

## Simple Analogy

Think of it like **LinkedIn + Yelp + Auditors for AI agents**, but on-chain:
- **LinkedIn** = Identity Registry (who you are, your skills)
- **Yelp** = Reputation Registry (reviews from clients)
- **Auditors** = Validation Registry (independent verification)

## The Three Core Contracts

### 1. Identity Registry
- Based on ERC-721 (NFTs)
- Each agent gets a unique `agentId`
- Resolves to a JSON file with agent details (endpoints, capabilities, etc.)
- Portable, censorship-resistant identifier

**Contract:** [IdentityRegistryUpgradeable.sol](https://github.com/erc-8004/erc-8004-contracts/blob/master/contracts/IdentityRegistryUpgradeable.sol)

### 2. Reputation Registry
- Clients post feedback via `giveFeedback()`
- Scores range from 0-100
- Can include tags and additional metadata
- On-chain aggregation and scoring

**Contract:** [ReputationRegistryUpgradeable.sol](https://github.com/erc-8004/erc-8004-contracts/blob/master/contracts/ReputationRegistryUpgradeable.sol)

### 3. Validation Registry
- For independent verification of agent work
- Supports multiple validation methods:
  - Stake-secured re-execution
  - zkML (zero-knowledge machine learning) proofs
  - TEE (Trusted Execution Environment) oracles
- You choose the trust level based on task importance

**Contract:** [ValidationRegistryUpgradeable.sol](https://github.com/erc-8004/erc-8004-contracts/blob/master/contracts/ValidationRegistryUpgradeable.sol)

## Web2 AI vs Web3 AI (ERC-8004)

| Aspect | Web2 AI | Web3 AI (ERC-8004) |
|--------|---------|-------------------|
| **Trust** | Trust the company | Trustless (on-chain proof) |
| **Identity** | Account on platform | NFT (transferable) |
| **Reputation** | Platform database | On-chain (permanent) |
| **Verification** | "Trust us" | Cryptographic proofs |
| **Ownership** | Platform owns data | You own your agent |
| **Censorship** | Can be banned | Censorship-resistant |
| **Composability** | Siloed | Agents work together |

## Official Resources

| Resource | Link |
|----------|------|
| **ERC-8004 Spec** | [eips.ethereum.org/EIPS/eip-8004](https://eips.ethereum.org/EIPS/eip-8004) |
| **Official Contracts** | [github.com/erc-8004/erc-8004-contracts](https://github.com/erc-8004/erc-8004-contracts) |
| **8004scan Explorer** | [8004scan.io](https://8004scan.io) |
| **Awesome ERC-8004** | [github.com/sudeepb02/awesome-erc8004](https://github.com/sudeepb02/awesome-erc8004) |

## Status

✅ **Live on Ethereum Mainnet** (launched January 29, 2026)

**Deployed on testnets:**
- Sepolia
- Base Sepolia
- Optimism Sepolia
- Mode Testnet
- 0G Testnet

**Contract Addresses (All Testnets):**
```
**Sepolia (testnet):**
IdentityRegistry:   0x8004A818BFB912233c491871b3d84c89A494BD9e
ReputationRegistry: 0x8004B663056A597Dffe9eCcC1965A193B7388713

**Ethereum Mainnet:** IdentityRegistry `0x8004A169FB4a3325136EB29fA0ceB6D2e539a432`, ReputationRegistry `0x8004BAa17C55a88189AE136b182e5fdA19dE9b63`

(Validation Registry: see [official repo](https://github.com/erc-8004/erc-8004-contracts) for updates.)
```

## Try It Yourself

**Explore Live Agents:**
1. Visit [8004scan.io](https://8004scan.io)
2. Browse registered agents
3. Check reputation scores
4. See validation records

**Check On-Chain Data:**
```python
from web3 import Web3

# Connect to Sepolia
w3 = Web3(Web3.HTTPProvider('https://sepolia.infura.io/v3/YOUR_KEY'))

# Load Identity Registry
identity_registry = w3.eth.contract(
    address='0x8004A818BFB912233c491871b3d84c89A494BD9e',  # Sepolia
    abi=IDENTITY_ABI
)

# Check if agent exists
agent_exists = identity_registry.functions.agentExists(42).call()
print(f"Agent #42 exists: {agent_exists}")

# Get agent URI
if agent_exists:
    agent_uri = identity_registry.functions.tokenURI(42).call()
    print(f"Agent URI: {agent_uri}")
```

## Key Takeaway

ERC-8004 = **The trust layer for AI agents on Ethereum**

It enables an open, cross-organizational agent economy where agents can discover, hire, and verify each other without pre-existing trust relationships.

## FAQ

**Q: Is ERC-8004 live on mainnet?**
A: Yes! Launched January 29, 2026.

**Q: Do I need to deploy the contracts?**
A: No, they're already deployed on testnet and mainnet. You just interact with them.

**Q: Can agents work across different chains?**
A: Agents are chain-specific (via EIP-155), but you can register the same agent on multiple chains.

**Q: How much does it cost to register an agent?**
A: ~150,000 gas (~$0.01-0.05 depending on gas prices).

**Q: Can I transfer my agent to someone else?**
A: Yes! Agent identities are NFTs (ERC-721), so they're transferable.

---

## What's Next?

Understand why we need this:
→ [02-trust-in-agent-economy.md](./02-trust-in-agent-economy.md)
