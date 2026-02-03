# Full Stack Overview

## The Key Understanding

**Your Python code calls smart contracts.**

That's it. That's the architecture.

```
Python Code → calls → Smart Contract
(your agent)         (on Ethereum)
```

Everything else is details.

## The Complete Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         YOUR AGENT                                   │
│                    (Python/TypeScript code)                          │
│                      RUNS OFF-CHAIN                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                     YOUR CODE (you write this)                │   │
│  ├──────────────────────────────────────────────────────────────┤   │
│  │                                                                  │
│  │   1. LLM Connection         2. Tools/Functions                │   │
│  │   ┌─────────────────┐      ┌─────────────────────┐           │   │
│  │   │ from anthropic  │      │ def get_price():    │           │   │
│  │   │ import Anthropic│      │ def analyze():      │           │   │
│  │   │                 │      │ def execute():      │           │   │
│  │   │ client = ...    │      │                     │           │   │
│  │   └─────────────────┘      └─────────────────────┘           │   │
│  │                                                               │   │
│  │   3. Web3 Connection (to smart contracts)                     │   │
│  │   ┌─────────────────────────────────────────────┐            │   │
│  │   │ from web3 import Web3                        │            │   │
│  │   │ w3 = Web3(Web3.HTTPProvider(...))            │            │   │
│  │   │                                              │            │   │
│  │   │ # Call smart contracts                       │            │   │
│  │   │ identity_registry.register(...)              │            │   │
│  │   │ reputation_registry.giveFeedback(...)        │            │   │
│  │   └─────────────────────────────────────────────┘            │   │
│  │                                                               │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
         │                    │                        │
         │                    │                        │
         ▼                    ▼                        ▼
   ┌──────────┐        ┌──────────┐           ┌───────────────┐
   │ LLM API  │        │ External │           │   Ethereum    │
   │ (Claude, │        │   APIs   │           │  Blockchain   │
   │  GPT-4)  │        │(prices,  │           │               │
   │          │        │ news)    │           │ ┌───────────┐ │
   │ OFF-CHAIN│        │ OFF-CHAIN│           │ │ Identity  │ │
   └──────────┘        └──────────┘           │ │ Registry  │ │
                                              │ ├───────────┤ │
                                              │ │Reputation │ │
                                              │ │ Registry  │ │
                                              │ ├───────────┤ │
                                              │ │Validation │ │
                                              │ │ Registry  │ │
                                              │ └───────────┘ │
                                              │   ON-CHAIN    │
                                              └───────────────┘
```

## Breaking It Down

### Layer 1: Your Agent (Off-Chain)

This is **your Python/TypeScript code** running on a server (AWS, Heroku, your laptop, etc.)

**Components:**
1. **LLM Integration** - Calls to OpenAI, Anthropic, etc.
2. **Business Logic** - Your tools, functions, workflows
3. **Web3 Integration** - Calls to Ethereum smart contracts

**Example Code:**
```python
from anthropic import Anthropic
from web3 import Web3

class MyAgent:
    def __init__(self):
        # LLM setup
        self.llm = Anthropic(api_key="...")

        # Web3 setup
        self.w3 = Web3(Web3.HTTPProvider("https://sepolia.infura.io/v3/..."))
        self.identity_registry = self.w3.eth.contract(
            address="0x8004A818BFB912233c491871b3d84c89A494BD9e",  # Sepolia
            abi=IDENTITY_ABI
        )

    def process_request(self, user_input):
        # Use LLM for reasoning
        response = self.llm.messages.create(
            model="claude-sonnet-4-20250514",
            messages=[{"role": "user", "content": user_input}]
        )

        # Use Web3 to read on-chain data
        agent_exists = self.identity_registry.functions.agentExists(42).call()

        return response.content[0].text
```

### Layer 2: External Services (Off-Chain)

Your agent calls external APIs:
- **LLM APIs** - OpenAI, Anthropic, etc.
- **Data APIs** - CoinGecko, Etherscan, news feeds
- **Other Services** - Databases, email, webhooks

These are **off-chain** - fast, cheap, but not trustless.

### Layer 3: Ethereum Blockchain (On-Chain)

The three registries live here:
- **Identity Registry** - Who is this agent?
- **Reputation Registry** - Is this agent good?
- **Validation Registry** - Can we prove it?

This is **on-chain** - slower, costs gas, but trustless and permanent.

## What Each Component Does

| Component | Location | Purpose | Example |
|-----------|----------|---------|---------|
| **Agent Code** | Off-chain | AI logic, orchestration | Python with CrewAI |
| **LLM** | Off-chain (API) | Reasoning "brain" | Claude, GPT-4 |
| **Tools** | Off-chain | Get data, execute actions | `get_price()`, `send_tx()` |
| **Smart Contracts** | On-chain | Store trust data | Identity, Reputation, Validation |
| **Web3 Library** | Off-chain | Bridge to blockchain | web3.py, ethers.js |

## On-Chain vs Off-Chain

Understanding this distinction is **crucial**:

| Aspect | On-Chain | Off-Chain |
|--------|----------|-----------|
| **Location** | Ethereum blockchain | Your server / APIs |
| **Speed** | Slow (12 sec blocks) | Fast (milliseconds) |
| **Cost** | Expensive (gas fees) | Cheap (API costs) |
| **Trust** | Trustless, verifiable | Requires trust |
| **Mutability** | Immutable | Can be changed |
| **Privacy** | Public | Can be private |
| **Best For** | Identity, reputation, proofs | AI logic, data processing |

### The Golden Rule

```
Put TRUST on-chain.
Put WORK off-chain.
```

**Examples:**
- Agent identity → On-chain ✅
- Agent's LLM reasoning → Off-chain ✅
- Client feedback/reviews → On-chain ✅
- Real-time price data → Off-chain ✅

## The Web3 Bridge

**web3.py** (Python) or **ethers.js** (JavaScript) connects your code to Ethereum:

```python
from web3 import Web3

# Connect to Ethereum node
w3 = Web3(Web3.HTTPProvider("https://sepolia.infura.io/v3/YOUR_KEY"))

# Load contract
identity_registry = w3.eth.contract(
    address="0x8004A818BFB912233c491871b3d84c89A494BD9e",  # Sepolia
    abi=IDENTITY_ABI  # Contract interface
)

# READ from contract (free, no gas)
agent_exists = identity_registry.functions.agentExists(42).call()
agent_uri = identity_registry.functions.tokenURI(42).call()

# WRITE to contract (costs gas)
tx = identity_registry.functions.register(
    "ipfs://Qm.../agent.json",
    b""  # metadata
).transact({"from": your_wallet_address})
```

## How Data Flows

### Example: User asks agent a question

```
1. User → "Should I buy Bitcoin?"
         ↓
2. Your Agent (Python)
         ├─ Check agent reputation (Web3 → IdentityRegistry)
         ├─ Analyze with LLM (API → Claude)
         ├─ Get price data (API → CoinGecko)
         └─ Return answer to user
         ↓
3. User → Satisfied with answer
         ↓
4. Your Agent → Record feedback (Web3 → ReputationRegistry)
         ↓
5. Feedback permanently stored on-chain
```

## Regular Function vs AI Agent (Revisited)

Now that you see the architecture:

### Traditional Bot
```python
def trading_bot(price):
    if price < 44000:
        return "BUY"
    # Fixed rules, no learning
```

### AI Agent (ERC-8004)
```python
class TradingAgent:
    def analyze(self, user_query):
        # 1. Get on-chain data (reputation, validation)
        reputation = self.get_reputation_score()

        # 2. Use LLM for reasoning
        analysis = self.llm.analyze(
            query=user_query,
            tools=[get_price, get_news, get_on_chain_data]
        )

        # 3. Return structured recommendation
        return analysis

    # After work done:
    def record_feedback(self, score):
        # Write to blockchain
        self.reputation_registry.giveFeedback(...)
```

The agent **thinks** (off-chain with LLM) but **proves trust** (on-chain with contracts).

## Key Takeaways

1. **Python/TS code runs off-chain** - Your agent is a regular server app
2. **Smart contracts live on-chain** - Trust data is on Ethereum
3. **web3.py bridges the gap** - Your code calls contracts
4. **LLMs are just APIs** - Claude, GPT-4 are external services
5. **On-chain = Trust, Off-chain = Work** - Use each for what it's best at

---

## What's Next?

Now see how an agent's lifecycle actually works:
→ [02-agent-lifecycle.md](./02-agent-lifecycle.md)

## Try It Yourself

**Exercise**: Install web3.py and read from a contract

```bash
pip install web3
```

```python
from web3 import Web3

# Connect to Sepolia testnet
w3 = Web3(Web3.HTTPProvider("https://sepolia.infura.io/v3/YOUR_KEY"))

# Check connection
print(f"Connected: {w3.is_connected()}")
print(f"Latest block: {w3.eth.block_number}")

# TODO: Read from IdentityRegistry contract
# Address: 0x8004A818BFB912233c491871b3d84c89A494BD9e (Sepolia)
```

<details>
<summary>Need an Infura key?</summary>

1. Go to https://infura.io
2. Sign up (free)
3. Create new project
4. Copy API key
5. Use in URL: `https://sepolia.infura.io/v3/YOUR_KEY`
</details>
