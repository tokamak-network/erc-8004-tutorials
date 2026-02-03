# Tools & SDKs for Building Agents

## Overview

Building an ERC-8004 agent requires combining several technologies. This guide covers all the tools you'll need.

## Recommended Stack

```
┌─────────────────────────────────────────┐
│           YOUR AGENT                     │
├─────────────────────────────────────────┤
│ AI Framework:    CrewAI or LangChain    │
│ Language:        Python or TypeScript   │
│ Web3 Library:    web3.py or ethers.js   │
│ LLM Provider:    OpenAI or Anthropic    │
│ Contract Tool:   Foundry (optional)     │
└─────────────────────────────────────────┘
```

## 1. Smart Contract Tools

### Foundry (Recommended for Contract Development)

**What:** Fast, portable Ethereum development toolkit

**Use when:** Building custom contracts or testing

```bash
# Install
curl -L https://foundry.paradigm.xyz | bash
foundryup

# Initialize project
forge init my-agent-contracts

# Install ERC-8004 contracts
forge install erc-8004/erc-8004-contracts

# Test
forge test
```

**Resources:**
- Website: [getfoundry.sh](https://getfoundry.sh)
- Book: [book.getfoundry.sh](https://book.getfoundry.sh)

### Hardhat (Alternative)

**What:** Ethereum development environment

**Use when:** You prefer JavaScript/TypeScript

```bash
npm install --save-dev hardhat
npx hardhat init
```

## 2. Web3 Libraries

### web3.py (Python - Recommended)

**What:** Python library for Ethereum interaction

```bash
pip install web3
```

**Example:**
```python
from web3 import Web3

# Connect to Sepolia
w3 = Web3(Web3.HTTPProvider('https://sepolia.infura.io/v3/YOUR_KEY'))

# Load Identity Registry
identity_registry = w3.eth.contract(
    address='0x8004A818BFB912233c491871b3d84c89A494BD9e',  # Sepolia
    abi=IDENTITY_ABI
)

# Register agent
tx_hash = identity_registry.functions.register(
    "ipfs://Qm.../agent.json",
    b""
).transact({'from': your_address})

receipt = w3.eth.wait_for_transaction_receipt(tx_hash)
print(f"Agent registered! Gas used: {receipt.gasUsed}")
```

**Resources:**
- Docs: [web3py.readthedocs.io](https://web3py.readthedocs.io)
- GitHub: [github.com/ethereum/web3.py](https://github.com/ethereum/web3.py)

### ethers.js (JavaScript/TypeScript)

**What:** Complete Ethereum library for JS/TS

```bash
npm install ethers
```

**Example:**
```typescript
import { ethers } from 'ethers';

// Connect to Sepolia
const provider = new ethers.JsonRpcProvider(
  'https://sepolia.infura.io/v3/YOUR_KEY'
);

// Load Identity Registry
const identityRegistry = new ethers.Contract(
  '0x8004A818BFB912233c491871b3d84c89A494BD9e',  # Sepolia
  IDENTITY_ABI,
  signer
);

// Register agent
const tx = await identityRegistry.register(
  'ipfs://Qm.../agent.json',
  '0x'
);
await tx.wait();
console.log('Agent registered!');
```

**Resources:**
- Docs: [docs.ethers.org](https://docs.ethers.org)
- GitHub: [github.com/ethers-io/ethers.js](https://github.com/ethers-io/ethers.js)

### viem (TypeScript - Modern Alternative)

**What:** Lightweight, type-safe Ethereum library

```bash
npm install viem
```

**Resources:**
- Docs: [viem.sh](https://viem.sh)

## 3. AI Agent Frameworks

### CrewAI (Recommended for Python)

**What:** Multi-agent orchestration framework

```bash
pip install crewai crewai-tools
```

**Example:**
```python
from crewai import Agent, Task, Crew
from anthropic import Anthropic

# Create agent
analyst = Agent(
    role='Crypto Analyst',
    goal='Provide accurate market analysis',
    backstory='Expert trader with 10 years experience',
    llm=Anthropic(api_key='...')
)

# Create task
task = Task(
    description='Analyze Bitcoin price trends',
    agent=analyst,
    expected_output='Detailed analysis with recommendation'
)

# Run
crew = Crew(agents=[analyst], tasks=[task])
result = crew.kickoff()
```

**Resources:**
- Website: [crewai.com](https://www.crewai.com)
- Docs: [docs.crewai.com](https://docs.crewai.com)
- GitHub: [github.com/joaomdmoura/crewAI](https://github.com/joaomdmoura/crewAI)

### LangChain (Alternative)

**What:** Framework for LLM applications

```bash
pip install langchain langchain-anthropic
```

**Example:**
```python
from langchain.agents import AgentExecutor, create_openai_tools_agent
from langchain_anthropic import ChatAnthropic

llm = ChatAnthropic(model='claude-sonnet-4-20250514')
agent = create_openai_tools_agent(llm, tools, prompt)
executor = AgentExecutor(agent=agent, tools=tools)

result = executor.invoke({"input": "Analyze Bitcoin"})
```

**Resources:**
- Docs: [python.langchain.com](https://python.langchain.com)
- GitHub: [github.com/langchain-ai/langchain](https://github.com/langchain-ai/langchain)

## 4. LLM Providers

### OpenAI (GPT-4)

**What:** Leading LLM provider

```bash
pip install openai
```

**Example:**
```python
from openai import OpenAI

client = OpenAI(api_key='sk-...')

response = client.chat.completions.create(
    model='gpt-4',
    messages=[
        {'role': 'user', 'content': 'Analyze BTC price'}
    ]
)

print(response.choices[0].message.content)
```

**Pricing:**
- GPT-4: ~$0.03 per 1K tokens
- GPT-3.5: ~$0.002 per 1K tokens

**Resources:**
- Platform: [platform.openai.com](https://platform.openai.com)
- Docs: [platform.openai.com/docs](https://platform.openai.com/docs)

### Anthropic (Claude)

**What:** Advanced AI with long context

```bash
pip install anthropic
```

**Example:**
```python
from anthropic import Anthropic

client = Anthropic(api_key='sk-ant-...')

response = client.messages.create(
    model='claude-sonnet-4-20250514',
    max_tokens=1024,
    messages=[
        {'role': 'user', 'content': 'Analyze BTC price'}
    ]
)

print(response.content[0].text)
```

**Pricing:**
- Claude Opus: ~$0.03 per 1K tokens
- Claude Sonnet: ~$0.015 per 1K tokens
- Claude Haiku: ~$0.001 per 1K tokens

**Resources:**
- Console: [console.anthropic.com](https://console.anthropic.com)
- Docs: [docs.anthropic.com](https://docs.anthropic.com)

### Ollama (Local - Free!)

**What:** Run LLMs locally

```bash
# Install
curl https://ollama.ai/install.sh | sh

# Run model
ollama run llama2
```

**Example:**
```python
import requests

response = requests.post('http://localhost:11434/api/generate', json={
    'model': 'llama2',
    'prompt': 'Analyze Bitcoin price trends'
})

print(response.json()['response'])
```

**Pros:** Free, private, no API limits
**Cons:** Requires good GPU, lower quality than GPT-4/Claude

**Resources:**
- Website: [ollama.ai](https://ollama.ai)
- Models: [ollama.ai/library](https://ollama.ai/library)

## 5. ERC-8004 Specific Tools

### Official Contracts

**nuwa-8004** (Latest v1.0 implementation)

```bash
# Clone
git clone https://github.com/nuwa-8004/nuwa-8004.git

# Contents:
contracts/
├── IdentityRegistryUpgradeable.sol
├── ReputationRegistryUpgradeable.sol
└── ValidationRegistryUpgradeable.sol

test/          # 79 tests (100% spec compliant)
abi/           # Contract ABIs
examples/      # Example agent JSON files
```

**Resources:**
- GitHub: [github.com/nuwa-8004](https://github.com/nuwa-8004)
- Deployment Guide: [DEPLOYMENT_GUIDE.md](https://github.com/nuwa-8004/nuwa-8004/blob/main/DEPLOYMENT_GUIDE.md)
- Implementers Guide: [IMPLEMENTERS_GUIDE.md](https://github.com/nuwa-8004/nuwa-8004/blob/main/IMPLEMENTERS_GUIDE.md)

### ChaosChain SDK (Coming Soon)

**What:** Complete ERC-8004 development kit

**Features:**
- Agent deployment wizard
- Reputation tracking dashboard
- Validation infrastructure
- Payment handling

**Status:** Beta access available

**Resources:**
- Website: [chaoschain.io](https://chaoschain.io)

### Praxis SDK (In Development)

**What:** Agent development SDK (Python/Go)

**Features:**
- Easy ERC-8004 integration
- Built-in reputation management
- Validation helpers

**Status:** Coming Q1 2026

## 6. Infrastructure Services

### Infura (Ethereum Node Provider)

**What:** Ethereum RPC endpoints

```
Free Tier:
- 100,000 requests/day
- Multiple networks

Paid:
- Unlimited requests
- $50/month+
```

**Sign up:** [infura.io](https://infura.io)

### Alchemy (Alternative)

**What:** Blockchain developer platform

```
Free Tier:
- 300M compute units/month
- Enhanced APIs

Paid:
- More compute units
- $49/month+
```

**Sign up:** [alchemy.com](https://www.alchemy.com)

### IPFS Pinning Services

**Pinata (Recommended)**

```
Free Tier:
- 1 GB storage
- 100 GB bandwidth

Paid:
- More storage
- $20/month+
```

**Sign up:** [pinata.cloud](https://pinata.cloud)

**Web3.Storage (Alternative)**

```
Free:
- Unlimited storage
- No bandwidth limits
- Backed by Filecoin
```

**Sign up:** [web3.storage](https://web3.storage)

## 7. Development Tools

### Python Environment

```bash
# Virtual environment
python -m venv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate     # Windows

# Install dependencies
pip install web3 anthropic crewai python-dotenv
```

### TypeScript Environment

```bash
# Initialize project
npm init -y
npm install typescript @types/node
npx tsc --init

# Install dependencies
npm install ethers dotenv
```

### Environment Variables

Create `.env` file:
```bash
# Ethereum
INFURA_API_KEY=your_infura_key
PRIVATE_KEY=your_private_key
SEPOLIA_RPC=https://sepolia.infura.io/v3/YOUR_KEY

# LLM
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...

# Contracts
IDENTITY_REGISTRY=0x8004A818BFB912233c491871b3d84c89A494BD9e
REPUTATION_REGISTRY=0x8004B663056A597Dffe9eCcC1965A193B7388713
# Validation Registry: see https://github.com/erc-8004/erc-8004-contracts for deployed address
```

## 8. Testing Tools

### Foundry (Contract Testing)

```bash
forge test -vvv
forge coverage
```

### Pytest (Python Testing)

```bash
pip install pytest pytest-asyncio

# Run tests
pytest tests/
```

### Jest (JavaScript Testing)

```bash
npm install --save-dev jest @types/jest
npm test
```

## 9. Deployment Tools

### Railway (Recommended for Backend)

**What:** Modern hosting platform

```
Features:
- Auto-deploy from GitHub
- Environment variables
- Logs and monitoring

Pricing:
- Free tier: $5 credit/month
- Paid: Usage-based
```

**Website:** [railway.app](https://railway.app)

### Heroku (Alternative)

**What:** Classic PaaS

```
Pricing:
- Free tier: Limited hours
- Hobby: $7/month
```

**Website:** [heroku.com](https://heroku.com)

### Vercel (For Frontends)

**What:** Frontend hosting

```
Free Tier:
- Unlimited sites
- Serverless functions
- Edge network
```

**Website:** [vercel.com](https://vercel.com)

## Complete Project Template

```
my-erc8004-agent/
├── .env                    # Environment variables
├── .gitignore              # Ignore .env, node_modules, etc.
├── requirements.txt        # Python dependencies
├── package.json            # Node dependencies (if using TS)
│
├── contracts/              # Smart contracts (optional)
│   ├── MyCustomContract.sol
│   └── test/
│
├── abi/                    # Contract ABIs
│   ├── IdentityRegistry.json
│   ├── ReputationRegistry.json
│   └── ValidationRegistry.json
│
├── src/
│   ├── agent.py            # Main agent logic
│   ├── tools/              # Agent tools
│   │   ├── price_tool.py
│   │   └── sentiment_tool.py
│   ├── web3_client.py      # Web3 integration
│   └── config.py           # Configuration
│
├── tests/
│   ├── test_agent.py
│   └── test_tools.py
│
├── scripts/
│   ├── register_agent.py   # Registration script
│   └── deploy.py           # Deployment script
│
├── agent.json              # Agent metadata
└── README.md
```

## Quick Start Guide

### 1. Install Dependencies

```bash
# Python
pip install web3 anthropic crewai python-dotenv

# Get API keys
# - Infura: infura.io
# - Anthropic: console.anthropic.com
# - Pinata: pinata.cloud (for IPFS)
```

### 2. Setup Environment

Create `.env`:
```
INFURA_API_KEY=...
ANTHROPIC_API_KEY=...
PRIVATE_KEY=...
```

### 3. Create Agent

```python
# agent.py
from anthropic import Anthropic
from web3 import Web3

class MyAgent:
    def __init__(self):
        self.llm = Anthropic(api_key=os.getenv('ANTHROPIC_API_KEY'))
        self.w3 = Web3(Web3.HTTPProvider(...))

    def analyze(self, query):
        response = self.llm.messages.create(
            model='claude-sonnet-4-20250514',
            messages=[{'role': 'user', 'content': query}]
        )
        return response.content[0].text
```

### 4. Register On-Chain

```python
# scripts/register_agent.py
from web3 import Web3

# Upload agent.json to IPFS
agent_uri = "ipfs://Qm.../agent.json"

# Register
tx = identity_registry.functions.register(
    agent_uri, b""
).transact({'from': your_address})

receipt = w3.eth.wait_for_transaction_receipt(tx)
print(f"Agent registered! ID: {receipt.logs[0].topics[1]}")
```

### 5. Deploy

```bash
# Deploy to Railway
railway init
railway up
```

## Resources Summary

| Category | Tool | Link |
|----------|------|------|
| **Contracts** | Foundry | [getfoundry.sh](https://getfoundry.sh) |
| **Web3** | web3.py | [web3py.readthedocs.io](https://web3py.readthedocs.io) |
| **AI Framework** | CrewAI | [crewai.com](https://www.crewai.com) |
| **LLM** | Anthropic | [console.anthropic.com](https://console.anthropic.com) |
| **Node Provider** | Infura | [infura.io](https://infura.io) |
| **IPFS** | Pinata | [pinata.cloud](https://pinata.cloud) |
| **Hosting** | Railway | [railway.app](https://railway.app) |

---

## What's Next?

Ready to build? Follow the practical guide:
→ [03-building-your-agent.md](./03-building-your-agent.md)

## Get Help

- **Documentation:** This guide + official docs
- **Community:** Telegram, Discord
- **Examples:** [nuwa-8004/examples](https://github.com/nuwa-8004/nuwa-8004/tree/main/examples)
