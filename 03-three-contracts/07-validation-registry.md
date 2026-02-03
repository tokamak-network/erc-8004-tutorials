# 2.3 Validation Registry - The Proof System

## What It Does

The Validation Registry is like **hiring an auditor** to verify your work:
- Agent does work → Requests validation
- Validator checks → Gives a response (0-100 score)
- Everything recorded on-chain (proof!)

## Simple Analogy

```
Real World:                    ERC-8004 Validation:
───────────                    ────────────────────
Accountant does taxes    →     Agent does work
Hires auditor to verify  →     validationRequest()
Auditor signs off        →     validationResponse(score=95)
Audit report filed       →     On-chain record forever
```

## Reputation vs Validation

| Aspect | Reputation | Validation |
|--------|------------|------------|
| Who gives it | **Client** (hired the agent) | **Validator** (independent checker) |
| Purpose | Feedback on experience | Proof of correctness |
| Nature | Subjective ("I liked it") | Objective ("Work is correct") |
| When | After job is done | During or after |
| Required | Optional | For high-stakes tasks |

## Key Data Structures

### Validation Request
```solidity
struct Request {
    address validatorAddress;   // Who should validate
    uint256 agentId;            // Which agent's work
    string requestUri;          // Link to work (IPFS, etc.)
    bytes32 requestHash;        // Hash for integrity
    uint256 timestamp;          // When requested
}
```

### Validation Response
```solidity
struct Response {
    address validatorAddress;   // Who validated
    uint256 agentId;            // Which agent
    uint8 response;             // Score 0-100
    bytes32 tag;                // Category tag
    uint256 lastUpdate;         // When responded
}
```

## Key Functions

```solidity
// Request validation for your agent's work
function validationRequest(
    address validatorAddress,  // Who should validate
    uint256 agentId,           // Your agent ID
    string requestUri,         // Link to work
    bytes32 requestHash        // Optional: hash for integrity
) external;

// Validator responds with their assessment
function validationResponse(
    bytes32 requestHash,       // Which request
    uint8 response,            // Score 0-100
    string responseUri,        // Optional: detailed response
    bytes32 responseHash,      // Optional: hash
    bytes32 tag                // Category tag
) external;

// Get validation status
function getValidationStatus(
    bytes32 requestHash
) external view returns (uint8 response, bytes32 tag, uint256 lastUpdate);

// Get all validations for an agent
function getAgentValidations(
    uint256 agentId
) external view returns (bytes32[] memory requestHashes);

// Get summary with optional filters
function getSummary(
    uint256 agentId,
    address validatorFilter,   // Filter by validator
    bytes32 tagFilter          // Filter by tag
) external view returns (Summary memory);
```

## The Complete Flow

```
Agent #42                     Validator #99                ValidationRegistry
    │                              │                              │
    │  1. validationRequest(                                      │
    │     validator=0x99...,                                      │
    │     agentId=42,                                             │
    │     uri="ipfs://my-analysis"                                │
    │  )                                                          │
    │ ───────────────────────────────────────────────────────────>│
    │                              │                              │
    │                              │     Request stored           │
    │                              │     requestHash generated    │
    │                              │                              │
    │                              │  2. Validator fetches URI    │
    │                              │     Reviews the work         │
    │                              │                              │
    │                              │  3. validationResponse(      │
    │                              │     requestHash,             │
    │                              │     response=92,             │
    │                              │     tag="quality"            │
    │                              │  )                           │
    │                              │ ─────────────────────────────>│
    │                              │                              │
    │                              │     Response stored on-chain │
    │                              │                              │
    │  4. Anyone can now verify:                                  │
    │     getValidationStatus(requestHash) → 92, "quality"        │
    │ <───────────────────────────────────────────────────────────│
```

## Security Features

### 1. No Self-Validation
```solidity
require(validatorAddress != agentOwner, "Self-validation not allowed");
require(validatorAddress != msg.sender, "Self-validation not allowed");
```
You cannot validate your own work - defeats the purpose!

### 2. Only Agent Owner Can Request
```solidity
require(
    msg.sender == agentOwner ||
    identityRegistry.isApprovedForAll(agentOwner, msg.sender) ||
    identityRegistry.getApproved(agentId) == msg.sender,
    "Not authorized"
);
```

### 3. Agent Must Exist
```solidity
require(identityRegistry.agentExists(agentId), "Agent does not exist");
```

### 4. Unique Request Hashes
Each validation request gets a unique hash - no duplicates.

## Types of Validators

ERC-8004 supports different validation methods:

| Method | How It Works | Trust Level |
|--------|--------------|-------------|
| **Human Validator** | Person reviews work manually | Medium |
| **Stake-Secured** | Validators stake ETH, lose if wrong | High |
| **zkML (Zero-Knowledge ML)** | Cryptographic proof of correct AI execution | Very High |
| **TEE (Trusted Execution Environment)** | Hardware-secured computation | Very High |

## When to Use Validation

```
Task Value/Risk              Recommended Approach
───────────────              ────────────────────
Low ($0-$100)         →      Reputation only
Medium ($100-$10k)    →      Reputation + Validation
High ($10k+)          →      Validation required (maybe zkML/TEE)
Critical (any amount) →      Multiple validators + cryptographic proof
```

## Test Example

From `test_ValidationRequest_Success`:
```solidity
// Agent requests validation
validationRegistry.validationRequest(
    validatorAddress,           // Who should validate
    agentId,                    // Agent #1
    "ipfs://QmValidation123",   // Link to work
    bytes32(0)                  // Auto-generate hash
);

// Validator responds
validationRegistry.validationResponse(
    requestHash,                // The request hash
    85,                         // Score: 85/100
    "ipfs://QmResponse456",     // Detailed response
    bytes32(0),                 // No hash needed
    bytes32("quality")          // Tag
);
```

## Connection to Other Registries

```
┌─────────────────┐
│IdentityRegistry │ ◄─── ValidationRegistry checks agent exists
└─────────────────┘
         │
         │ agentId
         ▼
┌─────────────────┐     ┌─────────────────────┐
│ReputationRegistry│     │ ValidationRegistry   │
│ (client reviews) │     │ (verifier proofs)    │
└─────────────────┘     └─────────────────────┘
         │                        │
         └────────────┬───────────┘
                      │
              Both build trust
              for the same agent
```

## Key Takeaways

1. **Validation is for proof** - Not just opinion, but verification
2. **Independent validators** - Can't validate your own work
3. **On-chain record** - Permanent proof of validation
4. **Flexible methods** - Human, stake-secured, zkML, TEE
5. **Use for high-stakes** - When reputation alone isn't enough

---

## Quick Quiz (Self-Check)

1. Can an agent validate their own work?
2. What's the score range for validation responses?
3. When would you use Validation instead of just Reputation?
4. Who can submit a validation response?

**Answers:**
1. No - self-validation is blocked
2. 0-100
3. For high-stakes tasks where you need proof, not just opinion
4. Only the designated validator address
