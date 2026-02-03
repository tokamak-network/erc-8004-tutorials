# 2.2 Reputation Registry - The Review System

## What It Does

The Reputation Registry is like **leaving a review on Amazon**, but on-chain:
- Reviews are permanent (on-chain)
- Agent must authorize you to leave a review (prevents spam)
- Scores are 0-100
- Can add tags for categorization
- Reviews can be revoked

## Simple Analogy

```
Amazon Review:          ERC-8004 Reputation:
─────────────          ─────────────────────
★★★★☆ (4/5)     →      Score: 80/100
"Great product"  →      feedbackURI: "ipfs://..."
Category: Electronics → tag1: "quality", tag2: "speed"
Can delete review →     isRevoked: true/false
```

## Key Data Structure

```solidity
struct Feedback {
    uint8 score;      // 0-100 rating
    bytes32 tag1;     // Category tag (e.g., "quality")
    bytes32 tag2;     // Another tag (e.g., "speed")
    bool isRevoked;   // Can be taken back
}
```

## Why Authorization is Required

**Problem:** Without authorization, anyone could spam fake reviews.

**Solution:** Agent must SIGN a message allowing you to leave feedback.

```
Flow:
1. Client does work with Agent #42
2. Agent #42 signs: "I authorize 0xClientAddress to give me feedback"
3. Client calls giveFeedback() with that signature
4. Contract verifies signature → Feedback accepted!
```

This uses EIP-191 (personal signatures) or ERC-1271 (contract signatures).

## Key Functions

```solidity
// Give feedback to an agent (requires authorization signature)
function giveFeedback(
    uint256 agentId,
    uint8 score,           // 0-100
    bytes32 tag1,          // Optional category
    bytes32 tag2,          // Optional category
    string fileuri,        // Optional: link to detailed feedback
    bytes32 filehash,      // Optional: hash for integrity
    bytes feedbackAuth     // Required: signed authorization
) external;

// Revoke feedback you previously gave
function revokeFeedback(
    uint256 agentId,
    uint64 feedbackIndex
) external;

// Append a response to feedback (anyone can respond)
function appendResponse(
    uint256 agentId,
    address clientAddress,
    uint64 feedbackIndex,
    string responseURI
) external;

// Get summary of feedback for an agent
function getSummary(
    uint256 agentId,
    address clientFilter,  // Optional: filter by client
    bytes32 tagFilter      // Optional: filter by tag
) external view returns (Summary memory);

// Get all feedback for an agent
function readAllFeedback(
    uint256 agentId
) external view returns (FeedbackData[] memory);
```

## Connection to Identity Registry

```solidity
IdentityRegistry public immutable identityRegistry;
```

The Reputation Registry checks the Identity Registry to:
- Verify agent exists before accepting feedback
- Prevent self-feedback (agent can't review themselves)

## Feedback Authorization Structure

```solidity
struct FeedbackAuth {
    uint256 agentId;           // Which agent
    address clientAddress;     // Who can give feedback
    uint64 indexLimit;         // Max number of feedbacks
    uint256 expiry;            // When authorization expires
    uint256 chainId;           // Network (prevents replay)
    address identityRegistry;  // Which registry
    address signerAddress;     // Who signed
}
```

## Important Security Features

1. **No Self-Feedback**: Agent cannot give feedback to themselves
2. **Expiry**: Authorization can have time limits
3. **Index Limits**: Can limit how many feedbacks one client gives
4. **Revocation**: Feedback can be revoked if needed
5. **Chain-bound**: Signatures are tied to specific network

## The Feedback Flow (Visual)

```
Client                          Agent #42                    ReputationRegistry
  │                                │                                │
  │  1. "I want to give feedback"  │                                │
  │ ──────────────────────────────>│                                │
  │                                │                                │
  │  2. Signs authorization        │                                │
  │ <──────────────────────────────│                                │
  │                                │                                │
  │  3. giveFeedback(agentId=42, score=95, signature)               │
  │ ───────────────────────────────────────────────────────────────>│
  │                                │                                │
  │                                │  4. Verify signature           │
  │                                │  5. Check agent exists         │
  │                                │  6. Store feedback on-chain    │
  │                                │                                │
  │  7. Feedback recorded!         │                                │
  │ <───────────────────────────────────────────────────────────────│
```

## Test Example

From `test_GiveFeedback_Success`:
```solidity
reputationRegistry.giveFeedback(
    agentId,           // Agent receiving feedback
    95,                // Score: 95/100
    TAG1,              // First tag
    TAG2,              // Second tag
    "ipfs://feedback", // Link to details
    bytes32(0),        // No hash needed for IPFS
    feedbackAuth       // Signed authorization
);
```

## Key Takeaways

1. **Authorization required** - Prevents spam and fake reviews
2. **Scores are 0-100** - Standardized rating scale
3. **Tags enable filtering** - Find feedback by category
4. **On-chain aggregation** - Smart contracts can read reputation
5. **Linked to Identity** - Only real agents can receive feedback

---

## Quick Quiz (Self-Check)

1. Why can't you give feedback without authorization?
2. What's the score range?
3. Can an agent give feedback to themselves?
4. Can feedback be deleted?

**Answers:**
1. Prevents spam, requires agent's signature
2. 0-100
3. No, blocked by contract
4. Not deleted, but can be revoked (isRevoked = true)
