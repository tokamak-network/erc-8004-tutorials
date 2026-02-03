# Relationship to Google's A2A Protocol

## What is A2A (Agent-to-Agent)?

Google released a protocol called **A2A** - a way for AI agents to communicate with each other. Think of it like HTTP but for agents:

- **HTTP**: Browser talks to server
- **A2A**: Agent talks to agent

A2A defines:
- How agents describe their capabilities ("I can analyze markets")
- How agents send tasks to each other
- How agents return results

## The A2A Protocol Basics

### Agent Cards
Each agent has an "Agent Card" - a JSON file describing:
```json
{
  "name": "Market Analyst Agent",
  "description": "I analyze crypto markets",
  "capabilities": ["market-analysis", "trend-prediction"],
  "endpoints": {
    "task": "https://agent.example.com/task",
    "status": "https://agent.example.com/status"
  }
}
```

### Task Flow
1. Agent A discovers Agent B's capabilities
2. Agent A sends a task request
3. Agent B processes and returns results
4. Agent A uses the results

## Actual A2A Message Format

Here's what a real A2A task request looks like:

```json
{
  "type": "task",
  "id": "task-123",
  "from": {
    "agentId": "agent-a",
    "endpoint": "https://agent-a.com/callback"
  },
  "to": {
    "agentId": "agent-b",
    "endpoint": "https://agent-b.com/task"
  },
  "task": {
    "type": "market-analysis",
    "parameters": {
      "symbol": "BTC",
      "timeframe": "1 week",
      "analysisType": "technical"
    }
  },
  "timestamp": "2026-02-03T10:30:00Z"
}
```

**Response:**
```json
{
  "type": "result",
  "id": "task-123",
  "status": "completed",
  "result": {
    "recommendation": "BUY",
    "confidence": 0.85,
    "reasoning": "Strong uptrend with bullish indicators...",
    "data": {
      "price": 45000,
      "rsi": 65,
      "sentiment": 0.78
    }
  },
  "timestamp": "2026-02-03T10:32:15Z"
}
```

## The Problem with A2A Alone

A2A is great, but it's **off-chain and trust-based**. You still need to:
- Trust that Agent B is who they say they are
- Trust that reviews are real
- Trust a central authority to resolve disputes

## How ERC-8004 Extends A2A

**ERC-8004 adds Ethereum's superpowers:**

| A2A (Google) | + ERC-8004 (Ethereum) |
|--------------|----------------------|
| Agent discovery | **On-chain identity** (can't be faked) |
| Reviews | **On-chain reputation** (permanent, auditable) |
| "Trust me" | **Cryptographic validation** (zkML proofs) |
| Centralized registry | **Decentralized registries** (censorship-resistant) |

## How They Work Together

```
┌─────────────────────────────────────────────────────────┐
│                    COMMUNICATION                         │
│                   (A2A Protocol)                         │
│  "Here's my task" → "Here's my result"                  │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                      TRUST                               │
│                  (ERC-8004)                              │
│  Identity Registry │ Reputation Registry │ Validation   │
└─────────────────────────────────────────────────────────┘
```

## Simple Summary

```
A2A = How agents talk to each other (the language)
ERC-8004 = How agents trust each other (the trust layer)
```

They work together - A2A for communication, ERC-8004 for trust.

## Practical Example with Code

### Without ERC-8004 (A2A only)
```
1. Find agent at some-website.com/agent-card.json
2. Hope the website isn't lying
3. Send task, get result
4. Hope the result is correct
5. If scammed, complain on Twitter?
```

### With ERC-8004
```python
from web3 import Web3
import requests

# Step 1: Query Identity Registry for agents
identity_registry = w3.eth.contract(...)
agent_id = 42

# Step 2: Check reputation on-chain
reputation_registry = w3.eth.contract(...)
summary = reputation_registry.functions.getSummary(agent_id, ...).call()
print(f"Agent #42 score: {summary.averageScore}/100")

if summary.averageScore < 80:
    print("Low reputation, skip!")
    exit()

# Step 3: Get agent endpoint from on-chain URI
agent_uri = identity_registry.functions.tokenURI(agent_id).call()
agent_info = fetch_ipfs(agent_uri)
endpoint = agent_info['endpoints']['a2a']

# Step 4: Send task via A2A protocol
response = requests.post(endpoint, json={
    "type": "task",
    "task": {
        "type": "market-analysis",
        "parameters": {"symbol": "BTC"}
    }
})

result = response.json()

# Step 5: (Optional) Request validation
validation_registry = w3.eth.contract(...)
validation_registry.functions.validationRequest(
    validatorAddress="0xValidator...",
    agentId=agent_id,
    requestUri="ipfs://work-proof",
    requestHash=b""
).transact(...)

# Step 6: Give feedback on-chain
# (Requires feedback authorization from agent)
reputation_registry.functions.giveFeedback(
    agent_id,
    95,  # score
    "quality",  # tag
    "speed",    # tag
    "ipfs://feedback",
    b"",
    feedback_signature
).transact(...)

print("✅ Feedback recorded on-chain permanently!")
```

## Comparison Table

| Feature | A2A Only | A2A + ERC-8004 |
|---------|----------|---------------|
| **Agent Discovery** | Search websites, directories | Query on-chain Identity Registry |
| **Identity Verification** | Trust domain/certificate | Verify NFT ownership on-chain |
| **Reputation** | Off-chain reviews (can be faked) | On-chain feedback (permanent) |
| **Dispute Resolution** | Contact support, legal action | On-chain evidence |
| **Censorship Resistance** | Can be shut down | Decentralized, unstoppable |
| **Composability** | Limited to A2A-compatible agents | Any agent can interact |
| **Cost** | Free (centralized hosting) | Gas fees (decentralization cost) |

## Integration Example

Most ERC-8004 agents use **both** protocols:

```python
class ERC8004Agent:
    def __init__(self):
        # A2A: For communication
        self.a2a_endpoint = "https://myagent.com/a2a"

        # ERC-8004: For trust
        self.agent_id = 42
        self.identity_registry = w3.eth.contract(...)
        self.reputation_registry = w3.eth.contract(...)

    def handle_a2a_request(self, request):
        """Handle incoming A2A task"""
        # Process via A2A protocol
        result = self.process_task(request['task'])

        # Return A2A response
        return {
            "type": "result",
            "status": "completed",
            "result": result
        }

    def sign_feedback_auth(self, client_address):
        """ERC-8004: Authorize client to give feedback"""
        # Sign authorization (EIP-712)
        signature = self.sign_message(...)
        return signature

    def check_client_reputation(self, client_agent_id):
        """ERC-8004: Check if client is trustworthy"""
        summary = self.reputation_registry.functions.getSummary(
            client_agent_id, ...
        ).call()
        return summary.averageScore > 70
```

## Key Takeaway

A2A and ERC-8004 are complementary:
- **A2A** = Communication protocol (how agents talk)
- **ERC-8004** = Trust protocol (how agents verify each other)

You need both for a functional, trustless agent economy.

## FAQ

**Q: Can I use ERC-8004 without A2A?**
A: Yes! A2A is optional. You can use any communication method (REST API, GraphQL, WebSockets).

**Q: Is A2A an open standard?**
A: Yes, published by Google for anyone to implement.

**Q: Do all ERC-8004 agents use A2A?**
A: No, but it's recommended for interoperability.

**Q: Can agents have multiple communication protocols?**
A: Yes! An agent can support A2A, REST API, GraphQL, etc. List them all in your agentURI.

---

## What's Next?

Learn about the EIP standards ERC-8004 builds on:
→ [04-required-eip-standards.md](./04-required-eip-standards.md)
