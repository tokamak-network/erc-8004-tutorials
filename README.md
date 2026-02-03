# ERC-8004 Learning Documentation

Complete guide to understanding and building **ERC-8004 Trustless Agents** on Ethereum.

## 🎯 What You'll Learn

This documentation teaches you everything from AI fundamentals to deploying your own trustless agent on Ethereum. Designed for developers with **blockchain experience but no AI background**.

**End Goal:** Build and deploy a working ERC-8004 agent on testnet (or mainnet when ready).

---

## 📚 Learning Path

Follow these sections in order:

### [1. AI Fundamentals](./01-ai-fundamentals/)
**Start here if you're new to AI agents**

Learn what AI agents are and how they work:
- What are LLMs? (Claude, GPT-4)
- Multi-agent systems
- CrewAI framework
- Tools and structured output

**Time:** ~45 minutes | **Prerequisites:** Basic Python

---

### [2. ERC-8004 Basics](./02-erc8004-basics/)
**Understand the standard**

Learn what ERC-8004 is and why it matters:
- What is ERC-8004?
- Trust in the agent economy
- Relationship to Google's A2A protocol
- Required EIP standards (155, 712, 721, 1271)

**Time:** ~25 minutes | **Prerequisites:** Blockchain basics

**📌 Official Resources:**
- [ERC-8004 Spec](https://eips.ethereum.org/EIPS/eip-8004)
- [Official Contracts](https://github.com/erc-8004/erc-8004-contracts)
- [8004scan Explorer](https://8004scan.io)

---

### [3. The Three Contracts](./03-three-contracts/)
**Deep dive into the smart contracts**

Understand the three core registries:
- **Identity Registry** - Agent discovery (NFT-based)
- **Reputation Registry** - Feedback and scoring
- **Validation Registry** - Work verification

**Time:** ~30 minutes | **Prerequisites:** Solidity basics

**📌 Contract Source Code:**
- [IdentityRegistry.sol](https://github.com/erc-8004/erc-8004-contracts/blob/master/contracts/IdentityRegistryUpgradeable.sol)
- [ReputationRegistry.sol](https://github.com/erc-8004/erc-8004-contracts/blob/master/contracts/ReputationRegistryUpgradeable.sol)
- [ValidationRegistry.sol](https://github.com/erc-8004/erc-8004-contracts/blob/master/contracts/ValidationRegistryUpgradeable.sol)

**📌 Deployed Addresses (official [erc-8004-contracts](https://github.com/erc-8004/erc-8004-contracts)):**

Addresses differ by **mainnet** vs **testnet**. The official repo only lists Identity and Reputation registries; Validation Registry is under active spec update.

| Registry | Mainnet | Testnet (Sepolia, Base Sepolia, Polygon Amoy, Monad Testnet, BSC Testnet) |
|----------|---------|----------------------------------------------------------------------------|
| IdentityRegistry | [`0x8004A169FB4a3325136EB29fA0ceB6D2e539a432`](https://etherscan.io/address/0x8004A169FB4a3325136EB29fA0ceB6D2e539a432) | [`0x8004A818BFB912233c491871b3d84c89A494BD9e`](https://sepolia.etherscan.io/address/0x8004A818BFB912233c491871b3d84c89A494BD9e) |
| ReputationRegistry | [`0x8004BAa17C55a88189AE136b182e5fdA19dE9b63`](https://etherscan.io/address/0x8004BAa17C55a88189AE136b182e5fdA19dE9b63) | [`0x8004B663056A597Dffe9eCcC1965A193B7388713`](https://sepolia.etherscan.io/address/0x8004B663056A597Dffe9eCcC1965A193B7388713) |

**Mainnet networks:** Ethereum, Base, Polygon, Monad, BSC (same mainnet addresses). **Testnet networks:** Ethereum Sepolia, Base Sepolia, Polygon Amoy, Monad Testnet, BSC Testnet (same testnet addresses).  
**Source:** [erc-8004-contracts README](https://github.com/erc-8004/erc-8004-contracts/blob/master/README.md) — check there for additional chains and Validation Registry updates.

---

### [4. Architecture](./04-architecture/)
**See how everything connects**

Understand the full stack:
- How Python AI code calls smart contracts
- Agent lifecycle (register → work → reputation)
- What you deploy vs what exists
- Deployment model and costs

**Time:** ~30 minutes | **Prerequisites:** Sections 1-3

---

### [5. Ecosystem](./05-ecosystem/)
**Explore and build**

Discover the ecosystem and build your agent:
- Live agents and success patterns
- Tools and SDKs (CrewAI, web3.py, etc.)
- Step-by-step building guide

**Time:** ~25 minutes | **Prerequisites:** Ready to build!

---

## 🚀 Quick Start

**Brand new to ERC-8004?**
1. Start with [01-ai-fundamentals](./01-ai-fundamentals/)
2. Read sequentially through section 5
3. Build your agent!

**Already know AI agents?**
1. Skip to [02-erc8004-basics](./02-erc8004-basics/)
2. Continue from there

**Just want to build?**
1. Read [03-three-contracts](./03-three-contracts/) (understand the contracts)
2. Jump to [05-ecosystem/03-building-your-agent.md](./05-ecosystem/03-building-your-agent.md)

---

## 📊 Documentation Stats

```
Total Sections:        5
Total Files:          21
Total Content:       ~120KB
Estimated Time:      ~2.5 hours
Code Examples:       50+
Diagrams:            30+
```

---

## 🎓 For Different Audiences

### Beginners (No AI Experience)
✅ Start with AI Fundamentals
✅ Follow the sequential path
✅ Take your time with examples
✅ Estimated time: 4-6 hours total

### Blockchain Developers
✅ Skim AI Fundamentals
✅ Focus on ERC-8004 Basics and Contracts
✅ Study Architecture section carefully
✅ Estimated time: 2-3 hours

### AI Developers (New to Blockchain)
✅ Skim AI Fundamentals
✅ Study ERC-8004 Basics thoroughly
✅ Focus on Three Contracts section
✅ Estimated time: 3-4 hours

### Experienced (Want to Build Fast)
✅ Review ERC-8004 Basics (10 min)
✅ Study Three Contracts (20 min)
✅ Jump to Building Guide (30 min)
✅ Estimated time: 1 hour

---

## 🛠️ Tech Stack Covered

**Smart Contracts:**
- Solidity
- Foundry
- ERC-721, EIP-712, ERC-1271

**Backend:**
- Python or TypeScript
- web3.py or ethers.js
- CrewAI or LangChain

**AI:**
- OpenAI (GPT-4)
- Anthropic (Claude)
- Ollama (local)

**Infrastructure:**
- Infura/Alchemy (RPC)
- IPFS/Pinata (storage)
- Railway/Heroku (hosting)

---

## 📖 Key Concepts

### The Three Contracts

```
┌─────────────────┐    ┌──────────────────┐    ┌───────────────────┐
│    IDENTITY     │    │   REPUTATION     │    │    VALIDATION     │
│    REGISTRY     │    │    REGISTRY      │    │     REGISTRY      │
├─────────────────┤    ├──────────────────┤    ├───────────────────┤
│ "Who are you?"  │    │ "Are you good?"  │    │ "Prove it!"       │
├─────────────────┤    ├──────────────────┤    ├───────────────────┤
│ • ERC-721 NFT   │    │ • Score 0-100    │    │ • Request check   │
│ • Agent URI     │    │ • Tags           │    │ • Validator reply │
│ • Metadata      │    │ • Authorization  │    │ • On-chain proof  │
└─────────────────┘    └──────────────────┘    └───────────────────┘
```

### How It Works

```
1. Agent registers on IdentityRegistry (gets NFT with agentId)
2. Agent does work for clients (off-chain AI processing)
3. Client posts feedback on ReputationRegistry (on-chain review)
4. (Optional) Request validation on ValidationRegistry (proof)
5. Agent builds reputation over time
```

### On-Chain vs Off-Chain

| Aspect | On-Chain | Off-Chain |
|--------|----------|-----------|
| **What** | Identity, reputation, proofs | AI logic, data processing |
| **Where** | Ethereum blockchain | Your server |
| **Speed** | Slow (12 sec blocks) | Fast (milliseconds) |
| **Cost** | Gas fees ($$) | Server costs ($) |
| **Trust** | Trustless | Requires trust |

---

## 🔗 Essential Links

| Resource | Description | Link |
|----------|-------------|------|
| **ERC-8004 Spec** | Official standard | [eips.ethereum.org/EIPS/eip-8004](https://eips.ethereum.org/EIPS/eip-8004) |
| **Official Contracts** | Reference implementation | [github.com/erc-8004/erc-8004-contracts](https://github.com/erc-8004/erc-8004-contracts) |
| **8004scan** | Agent explorer | [8004scan.io](https://8004scan.io) |
| **Awesome ERC-8004** | Curated resources | [github.com/sudeepb02/awesome-erc8004](https://github.com/sudeepb02/awesome-erc8004) |
| **Implementers Guide** | How to build | [IMPLEMENTERS_GUIDE.md](https://github.com/erc-8004/erc-8004-contracts/blob/master/IMPLEMENTERS_GUIDE.md) |

---

## ⏱️ Timeline

- **January 29, 2026**: ERC-8004 launched on Ethereum Mainnet (same registry addresses as testnets)
- **Early 2026**: Growing ecosystem (500+ agents on testnets; mainnet live)
- **Today**: You're learning to build one!

---

## 💡 What You'll Build

By the end of this documentation, you'll be able to:

✅ Understand AI agents and multi-agent systems
✅ Explain ERC-8004 and its three registries
✅ Read and interact with the smart contracts
✅ Build an agent with Python + AI + Web3
✅ Register your agent on testnet
✅ Earn reputation and build trust
✅ Deploy to mainnet (when ready)

---

## 🤝 Contributing

Found an error? Have a suggestion?
- Open an issue on GitHub
- Submit a pull request
- Join the community discussion

---

## 📝 License

This documentation is open source and available for the community.

---

## 🚦 Status

- ✅ Section 1: AI Fundamentals - **Complete**
- ✅ Section 2: ERC-8004 Basics - **Complete**
- ✅ Section 3: Three Contracts - **Complete**
- ✅ Section 4: Architecture - **Complete**
- ✅ Section 5: Ecosystem - **Complete**

**All documentation ready for community use!**

---

## 🎯 Next Steps

1. **Start Learning:** Begin with [01-ai-fundamentals](./01-ai-fundamentals/)
2. **Join Community:** Telegram, Discord (coming soon)
3. **Build Something:** Follow [05-ecosystem/03-building-your-agent.md](./05-ecosystem/03-building-your-agent.md)
4. **Share Your Agent:** Register on [8004scan.io](https://8004scan.io)

---

**Ready to build trustless agents? Let's go!** 🚀
