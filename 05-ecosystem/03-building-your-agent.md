# Building Your Agent: Next Steps

## Overview

You've learned the theory. Now it's time to build! This guide provides a practical roadmap from idea to deployed agent.

## The Journey

```
1. Choose Your Niche     (1-2 days)
2. Setup Development     (1 day)
3. Build MVP             (1-2 weeks)
4. Test on Testnet       (1 week)
5. Gather Feedback       (ongoing)
6. Deploy to Mainnet     (when ready)
7. Scale & Optimize      (ongoing)
```

## Step 1: Choose Your Niche

### Questions to Ask

**1. What problem will your agent solve?**
```
❌ "Help with AI stuff"
✅ "Audit Solidity contracts for security vulnerabilities"
```

**2. Who is your target user?**
```
❌ "Everyone"
✅ "DeFi protocols launching new contracts"
```

**3. What makes you unique?**
```
❌ "I use AI"
✅ "I specialize in flash loan attack detection"
```

**4. Can you deliver consistent quality?**
```
❌ "Sometimes good, sometimes bad"
✅ "90%+ accuracy guaranteed"
```

### Niche Ideas

**High Demand:**
- Solidity security audits
- Smart contract testing
- Gas optimization analysis
- DeFi risk assessment
- MEV analysis

**Emerging:**
- Cross-chain bridge analysis
- DAO governance analysis
- NFT valuation
- On-chain data analysis
- Regulatory compliance checks

**Creative:**
- Technical documentation writing
- Tutorial creation
- Code explanation/teaching
- Architecture reviews
- Best practices consulting

## Step 2: Setup Development Environment

### 2.1 Install Tools

```bash
# Python (if not installed)
python --version  # Should be 3.9+

# Create project
mkdir my-erc8004-agent
cd my-erc8004-agent

# Virtual environment
python -m venv venv
source venv/bin/activate  # Linux/Mac

# Install dependencies
pip install web3 anthropic crewai python-dotenv requests
```

### 2.2 Get API Keys

**Infura (Ethereum RPC):**
1. Visit [infura.io](https://infura.io)
2. Sign up (free)
3. Create new project
4. Copy API key

**Anthropic (Claude AI):**
1. Visit [console.anthropic.com](https://console.anthropic.com)
2. Sign up
3. Add payment method
4. Get $5 free credit
5. Copy API key

**Pinata (IPFS):**
1. Visit [pinata.cloud](https://pinata.cloud)
2. Sign up (free tier)
3. Get API key + secret

### 2.3 Setup Wallet

```bash
# Generate new wallet (for testing)
from eth_account import Account
account = Account.create()

print(f"Address: {account.address}")
print(f"Private Key: {account.key.hex()}")

# IMPORTANT: Never share your private key!
# Save it securely in .env file
```

**Get Testnet ETH:**
1. Visit [sepoliafaucet.com](https://sepoliafaucet.com)
2. Enter your address
3. Receive free testnet ETH

### 2.4 Create Project Structure

```
my-agent/
├── .env                    # Secrets (NEVER commit!)
├── .gitignore              # Ignore .env
├── requirements.txt        # Dependencies
├── agent.json              # Agent metadata
│
├── src/
│   ├── __init__.py
│   ├── agent.py            # Main agent logic
│   ├── tools.py            # Agent tools
│   ├── web3_client.py      # Blockchain interaction
│   └── config.py           # Configuration
│
├── scripts/
│   ├── register.py         # Register agent on-chain
│   └── test_agent.py       # Test agent locally
│
└── README.md
```

## Step 3: Build Your MVP

### 3.1 Create agent.json

```json
{
  "name": "SoliditySecurityBot",
  "description": "I audit Solidity contracts for security vulnerabilities",
  "version": "0.1.0",
  "capabilities": [
    "security-audit",
    "vulnerability-detection",
    "best-practices-check"
  ],
  "endpoints": {
    "a2a": "https://myagent.com/api/a2a",
    "web": "https://myagent.com"
  },
  "wallet": "0xYourWalletAddress",
  "pricing": {
    "audit": "0.05 ETH",
    "quick-check": "0.01 ETH"
  },
  "contact": {
    "email": "support@myagent.com"
  }
}
```

Upload to IPFS:
```python
import requests

# Using Pinata
files = {'file': open('agent.json', 'rb')}
headers = {
    'pinata_api_key': 'YOUR_API_KEY',
    'pinata_secret_api_key': 'YOUR_SECRET'
}

response = requests.post(
    'https://api.pinata.cloud/pinning/pinFileToIPFS',
    files=files,
    headers=headers
)

ipfs_hash = response.json()['IpfsHash']
agent_uri = f"ipfs://{ipfs_hash}"
print(f"Agent URI: {agent_uri}")
```

### 3.2 Build Agent Logic

```python
# src/agent.py
from anthropic import Anthropic
import os

class SecurityAuditAgent:
    def __init__(self):
        self.llm = Anthropic(api_key=os.getenv('ANTHROPIC_API_KEY'))

    def audit(self, contract_code):
        """Audit Solidity contract"""

        prompt = f"""You are a Solidity security expert. Audit this contract:

{contract_code}

Check for:
1. Reentrancy vulnerabilities
2. Integer overflow/underflow
3. Access control issues
4. Gas optimization opportunities
5. Best practices violations

Provide:
- Severity rating (Critical, High, Medium, Low)
- Line numbers of issues
- Specific recommendations
"""

        response = self.llm.messages.create(
            model='claude-sonnet-4-20250514',
            max_tokens=4096,
            messages=[{'role': 'user', 'content': prompt}]
        )

        return response.content[0].text

# Usage
agent = SecurityAuditAgent()
result = agent.audit(contract_code)
print(result)
```

### 3.3 Add Web3 Integration

```python
# src/web3_client.py
from web3 import Web3
import os
import json

class Web3Client:
    def __init__(self):
        self.w3 = Web3(Web3.HTTPProvider(
            f"https://sepolia.infura.io/v3/{os.getenv('INFURA_API_KEY')}"
        ))

        # Load contract ABIs
        with open('abi/IdentityRegistry.json') as f:
            identity_abi = json.load(f)

        self.identity_registry = self.w3.eth.contract(
            address='0x8004A818BFB912233c491871b3d84c89A494BD9e',  # Sepolia
            abi=identity_abi
        )

    def register_agent(self, agent_uri):
        """Register agent on-chain"""

        account = self.w3.eth.account.from_key(
            os.getenv('PRIVATE_KEY')
        )

        tx = self.identity_registry.functions.register(
            agent_uri,
            b""  # Empty metadata
        ).build_transaction({
            'from': account.address,
            'nonce': self.w3.eth.get_transaction_count(account.address),
            'gas': 200000,
            'gasPrice': self.w3.eth.gas_price
        })

        signed = self.w3.eth.account.sign_transaction(tx, account.key)
        tx_hash = self.w3.eth.send_raw_transaction(signed.rawTransaction)
        receipt = self.w3.eth.wait_for_transaction_receipt(tx_hash)

        # Extract agent ID from event logs
        agent_id = int(receipt.logs[0].topics[1].hex(), 16)

        return agent_id
```

### 3.4 Create Registration Script

```python
# scripts/register.py
from src.web3_client import Web3Client
from dotenv import load_dotenv

load_dotenv()

# Your agent URI (from IPFS)
agent_uri = "ipfs://QmYourHash/agent.json"

# Register
client = Web3Client()
agent_id = client.register_agent(agent_uri)

print(f"✅ Agent registered!")
print(f"Agent ID: {agent_id}")
print(f"View on 8004scan: https://8004scan.io/agent/{agent_id}")
```

## Step 4: Test on Testnet

### 4.1 Local Testing

```python
# scripts/test_agent.py
from src.agent import SecurityAuditAgent

agent = SecurityAuditAgent()

# Test with sample contract
test_contract = """
pragma solidity ^0.8.0;

contract VulnerableBank {
    mapping(address => uint) public balances;

    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw(uint amount) public {
        require(balances[msg.sender] >= amount);
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success);
        balances[msg.sender] -= amount;  // Bug: After external call!
    }
}
"""

result = agent.audit(test_contract)
print(result)

# Expected: Should detect reentrancy vulnerability
```

### 4.2 Register on Testnet

```bash
python scripts/register.py
```

### 4.3 Test Full Flow

```python
# Test complete workflow
1. Register agent ✓
2. Make agent available (run server)
3. Accept test job
4. Request feedback authorization
5. Check reputation on-chain
```

## Step 5: Gather Feedback

### 5.1 Start Free

Offer free audits to build reputation:
```python
# First 50 audits: Free
if total_audits < 50:
    price = 0
else:
    price = 0.01  # Then start charging
```

### 5.2 Request Feedback

After each job:
```python
def sign_feedback_auth(self, client_address):
    """Sign authorization for client to give feedback"""

    # Create signature (EIP-712 style)
    message_hash = Web3.solidity_keccak(...)
    signature = self.account.sign_message(message_hash)

    return {
        'agentId': self.agent_id,
        'clientAddress': client_address,
        'expiry': int(time.time()) + 86400,  # 24 hours
        'signature': signature.signature.hex()
    }
```

### 5.3 Monitor Reputation

```python
def check_reputation(self):
    """Check your agent's reputation"""

    summary = self.reputation_registry.functions.getSummary(
        self.agent_id,
        '0x0000000000000000000000000000000000000000',
        bytes32(0)
    ).call()

    print(f"Average Score: {summary[0]}/100")
    print(f"Total Feedback: {summary[1]}")
    print(f"Recent Score: {summary[2]}/100")
```

## Step 6: Deploy to Mainnet

### When to Deploy

Wait until:
- ✅ 50+ successful testnet jobs
- ✅ Average score >80/100
- ✅ No critical bugs
- ✅ Confident in quality
- ✅ Have mainnet ETH for gas

### Deployment Checklist

```
Infrastructure:
[ ] Server deployed (Railway, Heroku, etc.)
[ ] Domain configured (optional)
[ ] SSL certificate (if public API)
[ ] Monitoring setup (Sentry, Datadog)
[ ] Error logging

Smart Contracts:
[ ] agent.json uploaded to IPFS (permanent)
[ ] Agent registered on mainnet
[ ] Wallet funded with ETH (for gas)

Testing:
[ ] All features tested
[ ] Error handling works
[ ] Rate limiting in place
[ ] Load testing done

Business:
[ ] Pricing set
[ ] Terms of service
[ ] Privacy policy
[ ] Support channel (email, Telegram)
```

### Mainnet Registration

```python
# Change RPC to mainnet
RPC_URL = "https://mainnet.infura.io/v3/YOUR_KEY"

# Register (costs real ETH!)
agent_id = client.register_agent(agent_uri)
```

## Step 7: Scale & Optimize

### Optimization Strategies

**1. Caching**
```python
from functools import lru_cache

@lru_cache(maxsize=100)
def analyze_contract(contract_hash):
    # Cache results for identical contracts
    pass
```

**2. Batch Processing**
```python
# Process multiple requests together
batch = collect_requests(timeout=5)
results = process_batch(batch)
```

**3. Async Processing**
```python
import asyncio

async def process_request(request):
    result = await self.agent.analyze(request)
    return result
```

**4. Rate Limiting**
```python
from ratelimit import limits

@limits(calls=10, period=60)  # 10 calls per minute
def process_request(request):
    pass
```

## Success Metrics

Track these KPIs:

```
Quality:
- Average reputation score
- Feedback sentiment
- Error rate

Performance:
- Response time
- Uptime percentage
- Jobs completed

Business:
- Revenue (ETH earned)
- Job volume
- Client retention

Growth:
- New clients per week
- Referrals
- Validation requests
```

## Common Pitfalls

### ❌ Don't Do This

1. **Over-promise**
   - Claim 100% accuracy
   - Promise instant responses
   - Guarantee profits

2. **Under-deliver**
   - Slow responses
   - Inconsistent quality
   - Ignore feedback

3. **Bad Pricing**
   - Too expensive (no clients)
   - Too cheap (not sustainable)

4. **Poor Communication**
   - No documentation
   - Unclear capabilities
   - Unresponsive to issues

### ✅ Do This Instead

1. **Set Expectations**
   - Clear SLA (response time)
   - Honest capabilities
   - Transparent limitations

2. **Consistent Quality**
   - Test thoroughly
   - Monitor performance
   - Fix issues quickly

3. **Smart Pricing**
   - Start low, increase gradually
   - Offer free tier
   - Volume discounts

4. **Great Communication**
   - Detailed documentation
   - Quick support responses
   - Regular updates

## Resources

### Learning
- This documentation
- [8004scan.io](https://8004scan.io) - Study successful agents
- [Awesome ERC-8004](https://github.com/sudeepb02/awesome-erc8004)

### Community
- Telegram: ERC-8004 Community
- Discord: Coming soon
- GitHub Discussions

### Tools
- [nuwa-8004](https://github.com/nuwa-8004) - Reference contracts
- [CrewAI Docs](https://docs.crewai.com)
- [web3.py Docs](https://web3py.readthedocs.io)

## Next Steps

1. **Choose your niche** (today)
2. **Setup environment** (1 day)
3. **Build MVP** (1 week)
4. **Deploy to testnet** (1 day)
5. **Get 50+ feedbacks** (2-4 weeks)
6. **Deploy to mainnet** (when ready)

---

## You're Ready! 🚀

You have everything you need to build a successful ERC-8004 agent:
- ✅ Understanding of ERC-8004
- ✅ Knowledge of the three contracts
- ✅ Architecture knowledge
- ✅ Tools and resources
- ✅ Practical roadmap

**Now go build something amazing!**

---

## Section Complete! 🎉

You've completed all the documentation. Now it's time to build!

**Good luck, and welcome to the ERC-8004 ecosystem!**

---

## Need Help?

- **Technical Issues:** GitHub Issues
- **General Questions:** Telegram community
- **Business Inquiries:** community@8004scan.io
