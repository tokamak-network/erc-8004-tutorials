# Deployment Model: What You Actually Deploy

## The Big Question

**"What do I need to deploy to run an ERC-8004 agent?"**

Answer: **Just your Python/TypeScript backend!**

Let's break down what exists vs what you build.

## What You Deploy vs What's Already There

```
┌──────────────────────────────────────────────────────────────┐
│                    WHAT YOU DEPLOY                            │
└──────────────────────────────────────────────────────────────┘

     ┌─────────────┐         ┌─────────────────┐
     │   FRONTEND  │         │  PYTHON BACKEND │
     │   (React)   │ ──────> │   (Your Agent)  │
     │   Optional  │   API   │   ** THIS! **   │
     └─────────────┘         └────────┬────────┘
                                      │
                    ┌─────────────────┼─────────────────┐
                    │                 │                 │
                    ▼                 ▼                 ▼
              ┌──────────┐    ┌─────────────┐   ┌────────────┐
              │ LLM API  │    │  Price APIs │   │  Ethereum  │
              │ (Claude) │    │  (external) │   │ (testnet)  │
              └──────────┘    └─────────────┘   └────────────┘

              You DON'T       You DON'T        Already deployed
              deploy this     deploy this      (nuwa-8004)
```

## What YOU Deploy

### 1. Your Agent Backend (Required)

This is your **Python/TypeScript application**.

**What it contains:**
- LLM integration (OpenAI, Anthropic)
- Business logic (your tools, workflows)
- Web3 integration (calls to smart contracts)
- API endpoints (for clients to reach you)

**Example structure:**
```
my-agent/
├── agent.py              # Main agent logic
├── tools/
│   ├── price_tool.py     # Get crypto prices
│   ├── sentiment_tool.py # Analyze sentiment
│   └── onchain_tool.py   # Read blockchain data
├── contracts/
│   ├── identity_abi.json # Contract interfaces
│   └── reputation_abi.json
├── config.py             # API keys, addresses
└── requirements.txt      # Dependencies
```

**Where to deploy:**
- AWS EC2 / Lambda
- Google Cloud Run
- Heroku
- DigitalOcean
- Your own server
- Even your laptop (for testing!)

**Cost:** $5-50/month depending on usage

### 2. Frontend (Optional)

A web interface for users to interact with your agent.

**Technologies:**
- React / Next.js / Vue
- Connects to your backend via API
- Can integrate WalletConnect for payments

**Example:** `https://myagent.com` → User asks question → Your backend processes

**Where to deploy:**
- Vercel (free for small projects)
- Netlify (free tier)
- AWS S3 + CloudFront
- Same server as backend

**Cost:** Free - $10/month

### 3. Agent Registration File (Required)

JSON file describing your agent (the `agentURI`).

**Example:**
```json
{
  "name": "CryptoAnalyzer Pro",
  "description": "AI-powered crypto market analysis",
  "version": "1.0.0",
  "capabilities": ["market-analysis", "trend-prediction"],
  "endpoints": {
    "a2a": "https://api.myagent.com/a2a",
    "web": "https://myagent.com"
  },
  "wallet": "0x1234...5678"
}
```

**Where to host:**
- IPFS (recommended - decentralized)
- Your server (centralized but easy)
- Arweave (permanent storage)

## What You DON'T Deploy

### 1. Smart Contracts (Already on testnet!)

The three registries are **already deployed** by the ERC-8004 team:

**Testnet Addresses (Sepolia, Base Sepolia, etc.):**
```
IdentityRegistry:   0x8004A818BFB912233c491871b3d84c89A494BD9e   # Sepolia
ReputationRegistry: 0x8004B663056A597Dffe9eCcC1965A193B7388713   # Sepolia
ValidationRegistry: (see official erc-8004-contracts repo for deployed address)
```

You just **call** these contracts - you don't deploy them!

```python
# You do this (call existing contract):
identity_registry = w3.eth.contract(
    address="0x8004A818BFB912233c491871b3d84c89A494BD9e",  # Sepolia (official)
    abi=IDENTITY_ABI
)

# You DON'T do this (deploy new contract):
# identity_registry = w3.eth.contract(...).deploy()  ❌
```

### 2. LLM Models

You **call** LLM APIs - you don't host the models!

```python
# You do this (call API):
client = Anthropic(api_key="...")
response = client.messages.create(...)

# You DON'T do this (run model):
# model = load_model("claude-sonnet-4")  ❌
```

**Cost:** Pay per API call (~$0.01-0.03 per request)

### 3. Ethereum Nodes

You **connect** to existing nodes via RPC providers.

**Options:**
- Infura (free tier)
- Alchemy (free tier)
- QuickNode
- Public RPCs

```python
# You do this (connect to node):
w3 = Web3(Web3.HTTPProvider("https://sepolia.infura.io/v3/YOUR_KEY"))

# You DON'T do this (run node):
# geth --syncmode full ...  ❌ (unnecessary!)
```

## Deployment Checklist

When you're ready to deploy, you need:

### Environment Setup
- [ ] Python/Node.js installed
- [ ] Dependencies installed (`pip install web3 anthropic`)
- [ ] API keys configured:
  - [ ] LLM API key (OpenAI/Anthropic)
  - [ ] Ethereum RPC key (Infura/Alchemy)
  - [ ] Wallet private key (for signing transactions)

### Agent Registration
- [ ] Create agent registration JSON
- [ ] Upload to IPFS/server
- [ ] Call `IdentityRegistry.register(agentURI, metadata)`
- [ ] Save your `agentId`

### Backend Deployment
- [ ] Deploy Python/TS code to server
- [ ] Configure environment variables
- [ ] Test API endpoints
- [ ] Set up monitoring/logging

### Optional
- [ ] Deploy frontend
- [ ] Set up custom domain
- [ ] Configure SSL certificate
- [ ] Add analytics

## Hosting Your Agent Registration File

### Option 1: IPFS (Recommended)

**Pros:** Decentralized, permanent, censorship-resistant
**Cons:** Need to keep it pinned

```bash
# Install IPFS CLI
brew install ipfs  # macOS
# or download from https://ipfs.tech

# Initialize
ipfs init

# Start daemon
ipfs daemon

# Add your file
ipfs add agent.json
# Returns: QmX1234...  (this is your hash)

# Your URI: ipfs://QmX1234.../agent.json
```

**Keep it alive:**
- Pin on Pinata (https://pinata.cloud - free tier)
- Pin on Web3.Storage (https://web3.storage - free)
- Run your own IPFS node

### Option 2: Your Server (Easy)

**Pros:** Simple, full control
**Cons:** Centralized, depends on your uptime

```bash
# Just upload to your server
scp agent.json user@myserver.com:/var/www/html/

# Your URI: https://myserver.com/agent.json
```

### Option 3: Arweave (Permanent)

**Pros:** Permanent storage (pay once, stored forever)
**Cons:** Costs money (~$0.001 per KB)

```bash
# Use Arweave CLI or web uploader
arweave deploy agent.json

# Your URI: ar://abc123...
```

### Comparison Table

| Method | Cost | Permanence | Speed | Decentralized |
|--------|------|------------|-------|---------------|
| **IPFS** | Free* | High (if pinned) | Fast | Yes ✅ |
| **Your Server** | ~$5/mo | Depends on you | Fastest | No |
| **Arweave** | ~$0.001/KB | Forever | Fast | Yes ✅ |

*Free with pinning services' free tiers

## Typical Costs

Let's estimate monthly costs for running an agent:

### Small Agent (100 requests/month)
```
Backend hosting:        $5    (Heroku Hobby)
LLM API calls:          $3    (100 × $0.03)
Ethereum transactions:  $2    (gas fees)
Domain name:            $1    (/month)
─────────────────────────────
Total:                 ~$11/month
```

### Medium Agent (1,000 requests/month)
```
Backend hosting:       $20    (AWS EC2 t3.small)
LLM API calls:         $30    (1000 × $0.03)
Ethereum transactions:  $5    (gas fees)
IPFS pinning:          $0     (Pinata free tier)
Domain + SSL:          $1
─────────────────────────────
Total:                ~$56/month
```

### Large Agent (10,000 requests/month)
```
Backend hosting:      $100    (AWS EC2 t3.medium)
LLM API calls:        $300    (10000 × $0.03)
Ethereum transactions: $20    (gas fees)
Monitoring:           $10     (Datadog/Sentry)
Domain + SSL:         $2
─────────────────────────────
Total:               ~$432/month
```

**Note:** Most cost is LLM calls. Use caching to reduce!

## Scaling Your Agent

### Stage 1: MVP (You are here)
- Single server
- Manual deployment
- Test on Sepolia testnet
- **Cost:** $10-50/month

### Stage 2: Production
- Proper CI/CD (GitHub Actions)
- Multiple environments (dev, staging, prod)
- Monitoring and alerts
- **Cost:** $100-500/month

### Stage 3: Scale
- Load balancer
- Auto-scaling
- Redis caching (reduce LLM calls)
- Database for analytics
- **Cost:** $500-2000/month

## Development Workflow

```
1. Local Development
   └─ Run on localhost
   └─ Use testnet (Sepolia)
   └─ Test with small amounts

2. Deploy to Staging
   └─ Deploy to test server
   └─ Still use testnet
   └─ Invite beta users

3. Deploy to Production
   └─ Deploy to production server
   └─ Use mainnet (real money!)
   └─ Monitor carefully

4. Iterate
   └─ Add features
   └─ Improve based on feedback
   └─ Scale as needed
```

## Simple Deployment Example

```bash
# 1. Prepare your code
cd my-agent
pip install -r requirements.txt

# 2. Test locally
python agent.py
# ✓ Agent running on http://localhost:5000

# 3. Deploy to Heroku (example)
heroku create my-trustless-agent
git push heroku main
# ✓ Deployed to https://my-trustless-agent.herokuapp.com

# 4. Register on-chain
python scripts/register_agent.py
# ✓ Agent registered! ID: 42

# 5. You're live!
```

## Key Takeaways

1. **You only deploy your backend** - Python/TS application
2. **Smart contracts are already deployed** - Just call them
3. **LLMs are API calls** - Don't host models yourself
4. **Start small** - Deploy on free/cheap tier first
5. **IPFS is recommended** - For agent registration file
6. **Total cost: $10-100/month** for most agents

---

## Section Complete! 🎉

You now understand the full architecture! Next, explore the ecosystem:
→ **[../05-ecosystem/README.md](../05-ecosystem/README.md)**

## Deployment Resources

- **Heroku**: https://heroku.com (easy Python hosting)
- **Railway**: https://railway.app (modern alternative)
- **Infura**: https://infura.io (Ethereum RPC provider)
- **Pinata**: https://pinata.cloud (IPFS pinning)
- **Sepolia Faucet**: https://sepoliafaucet.com (free testnet ETH)

## Try It Yourself

**Exercise**: Deploy a simple "Hello World" agent

1. Create a simple Flask app
2. Add Web3 integration
3. Deploy to Heroku/Railway
4. Call `IdentityRegistry` from your deployed app

This will give you the full deployment experience!
