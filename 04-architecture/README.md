# Architecture: How Everything Connects

## Overview

Now that you understand AI fundamentals, it's time to see **how it all fits together** with blockchain.

This section answers:
- Where does AI code run? (Off-chain)
- Where does trust data live? (On-chain)
- How do they communicate? (Web3 calls)
- What do you actually deploy? (Your Python backend)

## The Key Insight

**Your Python code (with AI) calls smart contracts on Ethereum.**

```
Python Agent → web3.py → Smart Contracts
  (AI brain)   (bridge)   (trust data)
```

It's simpler than you think!

## What You'll Learn

1. **Full Stack Overview** - See the complete architecture diagram
2. **Agent Lifecycle** - Follow an agent from registration to earning reputation
3. **Deployment Model** - Understand what you deploy vs what exists

## Prerequisites

Before reading this section, you should understand:
- ✅ What AI agents are (from 01-ai-fundamentals)
- ✅ What ERC-8004 is (from 02-erc8004-basics)
- ✅ The three contracts (from 03-three-contracts)
- ✅ Basic Python or TypeScript

## Reading Order

```
1. 01-full-stack-overview.md    (10 min) - The big picture
2. 02-agent-lifecycle.md         (8 min) - How agents work
3. 03-deployment-model.md        (10 min) - What you deploy
```

**Total time: ~30 minutes**

## After This Section

Once you understand the architecture, explore what's out there:
- **05-ecosystem/** - See live agents, tools, and examples

---

## Key Concepts

| Concept | Simple Explanation |
|---------|-------------------|
| **Off-chain** | Your Python code running on a server (not on blockchain) |
| **On-chain** | Smart contracts on Ethereum (immutable, permanent) |
| **Web3** | Library that lets Python talk to blockchain |
| **Agent Backend** | Your Python server with LLM integration |
| **Registry** | Smart contract that stores agent data |

## Common Questions

**Q: Does my agent code run on the blockchain?**
A: No! Your Python/AI code runs on your server. Only trust data (identity, reputation) lives on-chain.

**Q: Do I need to deploy smart contracts?**
A: No! The three registries (Identity, Reputation, Validation) are already deployed on testnet.

**Q: Where does the LLM run?**
A: Off-chain, via API calls to OpenAI/Anthropic/etc.

**Q: What's the difference between on-chain and off-chain?**
A: On-chain = permanent, public, expensive. Off-chain = fast, private, cheap. Use each for what they're best at!

---

Ready? Start with [01-full-stack-overview.md](./01-full-stack-overview.md)
