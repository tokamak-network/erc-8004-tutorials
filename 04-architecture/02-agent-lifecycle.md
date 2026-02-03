# Agent Lifecycle

## Overview

An ERC-8004 agent goes through these stages:

```
1. REGISTER    → Agent gets on-chain identity
2. ADVERTISE   → Agent makes itself discoverable
3. WORK        → Agent performs tasks
4. GET PAID    → Agent receives payment
5. BUILD TRUST → Agent earns reputation
```

Let's follow an agent through its entire lifecycle!

## Stage 1: Registration (One-Time)

Your agent must register on-chain to get a unique `agentId`.

### The Code

```python
from web3 import Web3

# Connect to Ethereum
w3 = Web3(Web3.HTTPProvider("https://sepolia.infura.io/v3/YOUR_KEY"))

# Load Identity Registry contract
identity_registry = w3.eth.contract(
    address="0x8004A818BFB912233c491871b3d84c89A494BD9e",  # Sepolia
    abi=IDENTITY_REGISTRY_ABI
)

# Prepare agent metadata
agent_uri = "ipfs://Qm.../my-agent.json"  # Points to agent info
metadata = b""  # Optional extra data

# Register (sends transaction)
tx_hash = identity_registry.functions.register(
    agent_uri,
    metadata
).transact({
    "from": YOUR_WALLET_ADDRESS,
    "gas": 200000
})

# Wait for transaction to be mined
receipt = w3.eth.wait_for_transaction_receipt(tx_hash)

# Extract agent ID from event logs
agent_id = receipt['logs'][0]['topics'][1].hex()  # Your new agent ID!

print(f"Agent registered! ID: {agent_id}")
```

### What Happens On-Chain

```
Before: No agent exists
        ↓
Transaction sent to IdentityRegistry.register()
        ↓
After:  - ERC-721 NFT minted to your wallet
        - Token ID = your agentId
        - tokenURI = your agent_uri
        - Event emitted: Registered(agentId, owner, agentURI)
```

You now **own** this agent identity as an NFT!

## Stage 2: Advertise (Continuous)

Your agent must be discoverable. The `agentURI` points to a JSON file describing your agent.

### Agent Registration File

Host this JSON file on IPFS, your server, or Arweave:

```json
{
  "name": "CryptoAnalyzer Pro",
  "description": "I analyze crypto markets using on-chain + off-chain data",
  "version": "1.0.0",
  "capabilities": [
    "market-analysis",
    "trend-prediction",
    "portfolio-optimization"
  ],
  "endpoints": {
    "a2a": "https://myagent.com/api/a2a",
    "web": "https://myagent.com"
  },
  "wallet": "0x1234...5678",
  "pricing": {
    "analysis": "0.01 ETH",
    "portfolio_review": "0.05 ETH"
  },
  "contact": {
    "telegram": "@cryptoanalyzer",
    "email": "support@myagent.com"
  }
}
```

### How Clients Find You

```python
# Client searches for agents
identity_registry = w3.eth.contract(...)

# Get agent URI
agent_uri = identity_registry.functions.tokenURI(42).call()
# Returns: "ipfs://Qm.../my-agent.json"

# Fetch and parse JSON
agent_info = fetch_json(agent_uri)
print(agent_info['capabilities'])  # ["market-analysis", ...]

# Check reputation
reputation_registry = w3.eth.contract(...)
summary = reputation_registry.functions.getSummary(42, ...).call()
print(f"Score: {summary.averageScore}/100")

# Decide to hire!
```

## Stage 3: Work (The Core Loop)

Your agent receives requests and performs work.

### The Work Flow

```
Client Request
      ↓
┌─────────────────────────────────────┐
│        YOUR AGENT (Python)          │
│                                     │
│  1. Validate Request                │
│     - Check client authorization    │
│     - Verify payment (if required)  │
│                                     │
│  2. Process with LLM                │
│     - Parse request                 │
│     - Use tools to gather data      │
│     - Generate analysis             │
│                                     │
│  3. Format Response                 │
│     - Structured output             │
│     - Attach proof/evidence         │
│                                     │
│  4. Return to Client                │
│     - Send response                 │
│     - Request feedback auth         │
└─────────────────────────────────────┘
      ↓
Client receives result
```

### Example: Market Analysis Request

```python
class MarketAnalyzer:
    def handle_request(self, request):
        # 1. Parse request
        symbol = request['symbol']  # "BTC"
        analysis_type = request['type']  # "technical"

        # 2. Check agent is registered
        agent_exists = self.identity_registry.functions.agentExists(
            self.agent_id
        ).call()

        if not agent_exists:
            raise Exception("Agent not registered!")

        # 3. Do the actual work (LLM + tools)
        price = self.get_price(symbol)
        sentiment = self.get_sentiment(symbol)

        analysis = self.llm.analyze(
            f"Analyze {symbol}. Price: ${price}, Sentiment: {sentiment}"
        )

        # 4. Return structured result
        return {
            "symbol": symbol,
            "analysis": analysis,
            "timestamp": int(time.time()),
            "agent_id": self.agent_id
        }
```

## Stage 4: Get Paid

Payment happens **off-chain** (fast, cheap) or **on-chain** (trustless).

### Off-Chain Payment (Simple)
```python
# Client pays via Stripe, PayPal, etc.
payment = stripe.charge(
    amount=100,  # $1.00
    customer=client_id
)

if payment.successful:
    result = agent.process_request(request)
    return result
```

### On-Chain Payment (Trustless)
```python
# Client deposits ETH to smart contract
# Agent delivers work
# Contract releases payment if work is validated

# This requires a custom escrow contract
# (Not part of ERC-8004 spec, but common pattern)
```

## Stage 5: Build Trust

After completing work, earn reputation!

### The Reputation Flow

```
1. Agent completes work
        ↓
2. Agent signs feedback authorization
        ↓
   FeedbackAuth {
       agentId: 42,
       clientAddress: 0xClient...,
       expiry: timestamp + 1 day
   }
        ↓
3. Agent sends signature to client
        ↓
4. Client gives feedback (on-chain)
        ↓
   reputation_registry.giveFeedback(
       agentId: 42,
       score: 95,  # 0-100
       tag1: "quality",
       tag2: "fast",
       fileuri: "ipfs://...",  # Optional detailed review
       feedbackAuth: signature
   )
        ↓
5. Feedback stored permanently on-chain!
```

### Code: Sign Feedback Authorization

```python
from eth_account.messages import encode_defunct

def sign_feedback_auth(agent_id, client_address, expiry):
    # Create message
    message_hash = Web3.solidity_keccak(
        ['uint256', 'address', 'uint256', 'uint256', 'address'],
        [agent_id, client_address, 0, expiry, IDENTITY_REGISTRY_ADDR]
    )

    # Sign with your private key
    signable_message = encode_defunct(primitive=message_hash)
    signed = w3.eth.account.sign_message(
        signable_message,
        private_key=YOUR_PRIVATE_KEY
    )

    return signed.signature.hex()

# After work is done:
signature = sign_feedback_auth(
    agent_id=42,
    client_address="0xClient...",
    expiry=int(time.time()) + 86400  # 24 hours
)

# Send signature to client
return {
    "result": analysis_result,
    "feedback_auth": signature
}
```

## Complete Example: User to Agent to Blockchain

Let's trace a real request through the entire system:

```
User: "Should I buy Bitcoin?"
         │
         ▼
┌─────────────────────────────────────┐
│    YOUR AGENT (Python + LLM)       │
│                                     │
│  Step 1: Check agent registered    │
│  └─ Web3 → IdentityRegistry        │
│  └─ agentExists(42) → true ✓       │
│                                     │
│  Step 2: Check reputation           │
│  └─ Web3 → ReputationRegistry       │
│  └─ getSummary(42) → 87/100 ✓      │
│                                     │
│  Step 3: Process with LLM           │
│  └─ API → Claude Sonnet 4           │
│  └─ LLM calls tools:                │
│      ├─ get_price("BTC") → $45,000  │
│      ├─ get_sentiment() → Bullish   │
│      └─ get_on_chain() → High vol   │
│  └─ LLM reasons and decides         │
│                                     │
│  Step 4: Return analysis            │
│  └─ "Based on analysis, consider    │
│      buying. Price is favorable..." │
│                                     │
│  Step 5: Sign feedback auth         │
│  └─ Return signature to client      │
└─────────────────────────────────────┘
         │
         ▼
User receives: Analysis + Auth
         │
         ▼
User satisfied? Gives feedback:
         │
         ▼
┌─────────────────────────────────────┐
│   Client calls on-chain:            │
│   reputation_registry.giveFeedback( │
│      agentId: 42,                   │
│      score: 95,                     │
│      feedbackAuth: signature        │
│   )                                 │
└─────────────────────────────────────┘
         │
         ▼
Feedback permanently on-chain!
Agent reputation increases! 🎉
```

## Optional: Validation (High-Stakes Work)

For important work, request independent validation:

```python
# After completing work
validation_registry.validationRequest(
    validatorAddress="0xValidator...",
    agentId=42,
    requestUri="ipfs://my-work-proof",
    requestHash=b""
)

# Validator reviews work and responds on-chain
# Now you have cryptographic proof of quality!
```

## The Reputation Flywheel

```
Good Work → High Reputation → More Clients → More Work → Better Skills
    ↑                                                            │
    └────────────────────────────────────────────────────────────┘
```

## Key Takeaways

1. **Registration is one-time** - Get your agentId (NFT)
2. **agentURI is your profile** - How clients find you
3. **Work happens off-chain** - Fast, cheap, private
4. **Trust is built on-chain** - Permanent, verifiable
5. **Reputation compounds** - Good agents get more work

---

## What's Next?

Understand exactly what you need to deploy:
→ [03-deployment-model.md](./03-deployment-model.md)

## Think About It

**Question**: Why does reputation need to be on-chain, but the actual analysis work can be off-chain?

<details>
<summary>Answer</summary>

**Reputation on-chain:**
- Needs to be trustless (can't be faked)
- Must be permanent (can't be deleted)
- Should be public (others can verify)
- Worth the gas cost (it's your long-term value)

**Work off-chain:**
- Too expensive on-chain (would cost $$$ in gas)
- Needs to be fast (instant responses)
- May contain sensitive data (privacy)
- Can use powerful LLMs (not possible on-chain)

Each layer does what it's best at!
</details>
