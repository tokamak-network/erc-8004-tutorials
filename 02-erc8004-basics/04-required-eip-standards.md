# Required EIP Standards

ERC-8004 doesn't reinvent the wheel - it builds on existing Ethereum standards. Here's what each one does and why ERC-8004 needs it.

## EIP-155: Chain ID

### What it does
Prevents replay attacks across chains by including a chain identifier in transactions.

### Why ERC-8004 needs it
- An agent registered on Ethereum mainnet shouldn't accidentally work on Arbitrum
- Chain ID ensures transactions are network-specific
- Each chain has its own set of registries

### You already use this
Every time you set `chainId` in your Foundry config or MetaMask:
- Ethereum Mainnet: 1
- Sepolia: 11155111
- Arbitrum: 42161

**In Practice:**
```python
from web3 import Web3

# Chain ID is included in every transaction
tx = {
    'from': your_address,
    'to': contract_address,
    'value': 0,
    'gas': 100000,
    'gasPrice': w3.eth.gas_price,
    'nonce': w3.eth.get_transaction_count(your_address),
    'chainId': 11155111  # Sepolia - prevents replay on mainnet
}

signed_tx = w3.eth.account.sign_transaction(tx, private_key)
tx_hash = w3.eth.send_raw_transaction(signed_tx.rawTransaction)
```

---

## EIP-712: Typed Structured Data Signing

### What it does
A standard way to sign human-readable data (not just raw hashes).

### Why ERC-8004 needs it
- Agents need to sign messages like "I authorize feedback for Agent #42"
- EIP-712 makes these signatures readable and verifiable
- Enables gasless operations (sign off-chain, verify on-chain)

### Example
```
Instead of signing: 0x7f83b1657ff1fc53b92dc18...

You sign something readable:
┌─────────────────────────────────┐
│ Give Feedback                   │
│ ─────────────────────────────── │
│ Agent ID: 42                    │
│ Score: 85                       │
│ Comment: "Great market analysis"│
│ Timestamp: 1706621234           │
└─────────────────────────────────┘
```

### Practical Example: Signing Feedback Authorization

**Step 1: Define EIP-712 Domain**
```python
from eth_account.messages import encode_structured_data

# EIP-712 Domain
domain = {
    'name': 'ERC8004ReputationRegistry',
    'version': '1',
    'chainId': 11155111,  # Sepolia
    'verifyingContract': '0x8004B663056A597Dffe9eCcC1965A193B7388713'  # Sepolia ReputationRegistry
}
```

**Step 2: Define Message Types**
```python
types = {
    'EIP712Domain': [
        {'name': 'name', 'type': 'string'},
        {'name': 'version', 'type': 'string'},
        {'name': 'chainId', 'type': 'uint256'},
        {'name': 'verifyingContract', 'type': 'address'}
    ],
    'FeedbackAuth': [
        {'name': 'agentId', 'type': 'uint256'},
        {'name': 'clientAddress', 'type': 'address'},
        {'name': 'indexLimit', 'type': 'uint64'},
        {'name': 'expiry', 'type': 'uint256'},
        {'name': 'identityRegistry', 'type': 'address'}
    ]
}
```

**Step 3: Create Message**
```python
import time

message = {
    'agentId': 42,
    'clientAddress': '0xClientAddress...',
    'indexLimit': 0,  # No limit
    'expiry': int(time.time()) + 86400,  # 24 hours
    'identityRegistry': '0x8004A818BFB912233c491871b3d84c89A494BD9e'  # Sepolia
}
```

**Step 4: Sign with EIP-712**
```python
from eth_account import Account

# Create structured data
structured_data = {
    'types': types,
    'primaryType': 'FeedbackAuth',
    'domain': domain,
    'message': message
}

# Sign
encoded = encode_structured_data(structured_data)
signed = Account.sign_message(encoded, private_key)

signature = signed.signature.hex()
print(f"Signature: {signature}")
```

**Step 5: Client Uses Signature to Give Feedback**
```python
# Client now calls giveFeedback() with the signature
reputation_registry.functions.giveFeedback(
    agentId=42,
    score=95,
    tag1=b"quality",
    tag2=b"fast",
    fileuri="ipfs://feedback-details",
    filehash=b"",
    feedbackAuth=bytes.fromhex(signature[2:])  # Remove 0x prefix
).transact({'from': client_address})
```

### Complete Working Example

```python
from web3 import Web3
from eth_account import Account
from eth_account.messages import encode_structured_data
import time

# Setup
w3 = Web3(Web3.HTTPProvider('https://sepolia.infura.io/v3/YOUR_KEY'))
agent_private_key = 'YOUR_PRIVATE_KEY'
agent_account = Account.from_key(agent_private_key)

def sign_feedback_authorization(agent_id, client_address, expiry_hours=24):
    """Agent signs authorization for client to give feedback"""

    domain = {
        'name': 'ERC8004ReputationRegistry',
        'version': '1',
        'chainId': 11155111,
        'verifyingContract': '0x8004B663056A597Dffe9eCcC1965A193B7388713'  # Sepolia ReputationRegistry
    }

    types = {
        'EIP712Domain': [
            {'name': 'name', 'type': 'string'},
            {'name': 'version', 'type': 'string'},
            {'name': 'chainId', 'type': 'uint256'},
            {'name': 'verifyingContract', 'type': 'address'}
        ],
        'FeedbackAuth': [
            {'name': 'agentId', 'type': 'uint256'},
            {'name': 'clientAddress', 'type': 'address'},
            {'name': 'indexLimit', 'type': 'uint64'},
            {'name': 'expiry', 'type': 'uint256'},
            {'name': 'identityRegistry', 'type': 'address'}
        ]
    }

    message = {
        'agentId': agent_id,
        'clientAddress': client_address,
        'indexLimit': 0,
        'expiry': int(time.time()) + (expiry_hours * 3600),
        'identityRegistry': '0x8004A818BFB912233c491871b3d84c89A494BD9e'  # Sepolia
    }

    structured_data = {
        'types': types,
        'primaryType': 'FeedbackAuth',
        'domain': domain,
        'message': message
    }

    encoded = encode_structured_data(structured_data)
    signed = agent_account.sign_message(encoded)

    return signed.signature.hex()

# Usage
signature = sign_feedback_authorization(
    agent_id=42,
    client_address='0xClient...',
    expiry_hours=24
)

print(f"✅ Signature ready for client: {signature}")
```

---

## EIP-721: NFTs (Non-Fungible Tokens)

### What it does
The standard for unique, non-fungible tokens (NFTs).

### Why ERC-8004 needs it
- Each agent gets a **unique identity as an NFT**
- Agent ID #42 is literally an ERC-721 token
- This means agent identities are:
  - **Transferable** - sell your agent's reputation
  - **Ownable** - prove you control this agent
  - **Interoperable** - works with any NFT tool/marketplace

### Key Functions Used
```solidity
// From ERC-721
ownerOf(tokenId)      // Who owns this agent?
transferFrom(...)     // Transfer agent ownership
tokenURI(tokenId)     // Get agent metadata URI
```

### Key Insight
**Your agent's identity IS an NFT.** When you register an agent, you mint an NFT to yourself. The token ID becomes your agent ID.

**Practical Example:**
```python
from web3 import Web3

# Check who owns agent #42
identity_registry = w3.eth.contract(...)
owner = identity_registry.functions.ownerOf(42).call()
print(f"Agent #42 owned by: {owner}")

# Get agent metadata
agent_uri = identity_registry.functions.tokenURI(42).call()
print(f"Agent URI: {agent_uri}")

# Transfer agent (sell with reputation!)
identity_registry.functions.transferFrom(
    current_owner,
    new_owner,
    42  # agent ID
).transact({'from': current_owner})
```

---

## ERC-1271: Smart Contract Signature Verification

### What it does
Allows smart contracts (not just EOA wallets) to verify signatures.

### Why ERC-8004 needs it
- An agent might be controlled by a **smart contract** (multisig, DAO, etc.)
- Not just an EOA (regular wallet with private key)
- ERC-1271 lets contracts validate signatures on their behalf

### Example Use Case
```
Agent owned by a 3-of-5 multisig:

1. Agent #42 is owned by MultisigContract
2. MultisigContract implements ERC-1271
3. When Agent #42 needs to sign something:
   - 3 of 5 signers approve
   - MultisigContract's isValidSignature() returns true
4. ERC-8004 registry accepts the signature
```

### The Interface
```solidity
interface IERC1271 {
    function isValidSignature(
        bytes32 hash,
        bytes memory signature
    ) external view returns (bytes4 magicValue);
}
```

**Magic Value:** `0x1626ba7e` (returned when signature is valid)

### Practical Example

```solidity
// Simple multisig that implements ERC-1271
contract MultisigAgent is IERC1271 {
    address[] public signers;
    uint256 public threshold;  // e.g., 3 of 5

    mapping(bytes32 => uint256) public approvals;

    function isValidSignature(
        bytes32 hash,
        bytes memory signature
    ) external view override returns (bytes4) {
        // Check if enough signers approved this hash
        if (approvals[hash] >= threshold) {
            return 0x1626ba7e;  // Magic value = valid
        }
        return 0xffffffff;  // Invalid
    }

    function approveMessage(bytes32 hash) external {
        require(isSigner(msg.sender), "Not a signer");
        approvals[hash]++;
    }
}
```

---

## Summary Table

| Standard | Purpose | ERC-8004 Usage |
|----------|---------|----------------|
| **EIP-155** | Chain ID | Network isolation - agents are chain-specific |
| **EIP-712** | Typed signing | Gasless authorizations, readable signatures |
| **EIP-721** | NFTs | Agent identity tokens - each agent is an NFT |
| **ERC-1271** | Contract signatures | Smart contract-owned agents (multisigs, DAOs) |

---

## How They All Connect

```
┌─────────────────────────────────────────────────────────┐
│                  Agent Registration                      │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  EIP-721: Mint agent NFT (agentId = tokenId)            │
│  EIP-155: Bound to specific chain                       │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                  Agent Operations                        │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  EIP-712: Sign authorizations (feedback, validation)    │
│  ERC-1271: Verify if owner is a contract                │
└─────────────────────────────────────────────────────────┘
```

## Key Takeaway

ERC-8004 is built on battle-tested Ethereum primitives:
- **Identity** = NFTs (EIP-721)
- **Security** = Chain isolation (EIP-155)
- **Usability** = Readable signatures (EIP-712)
- **Flexibility** = Contract ownership (ERC-1271)

You don't need to implement these yourself - they're already part of the ERC-8004 contracts.

## FAQ

**Q: Do I need to understand EIP-712 to use ERC-8004?**
A: Basic understanding helps, but most libraries (web3.py, ethers.js) handle it for you.

**Q: Can a DAO own an agent?**
A: Yes! Via ERC-1271, any contract can own and control an agent.

**Q: Why use EIP-712 instead of simple signatures?**
A: EIP-712 signatures are human-readable, preventing phishing attacks.

**Q: Can I test EIP-712 signing locally?**
A: Yes! Use the complete example above with your own test wallet.

---

## Section Complete! 🎉

You now understand what ERC-8004 is and why it exists. Next, dive into the three core contracts:
→ **[../03-three-contracts/README.md](../03-three-contracts/README.md)**
