# What are LLMs?

## Simple Definition

**LLM = Large Language Model**

Think of it as a super smart text predictor trained on vast amounts of internet data. You give it text (prompt), it returns intelligent text (response).

## Real-World Analogy

```
LLM is like a very knowledgeable assistant:
- You ask a question → It thinks → Gives answer
- You give instructions → It understands → Takes action
- You provide context → It remembers → Uses it
```

## How to Call an LLM

It's surprisingly simple - just an API call!

### Example 1: Using Claude (Anthropic)

```python
from anthropic import Anthropic

client = Anthropic(api_key="your-api-key")

response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "What is Bitcoin?"}
    ]
)

print(response.content[0].text)
# Output: "Bitcoin is a decentralized digital currency..."
```

### Example 2: Using GPT-4 (OpenAI)

```python
from openai import OpenAI

client = OpenAI(api_key="your-api-key")

response = client.chat.completions.create(
    model="gpt-4",
    messages=[
        {"role": "user", "content": "What is Bitcoin?"}
    ]
)

print(response.choices[0].message.content)
```

## Understanding Message Roles

When talking to an LLM, messages have specific roles:

| Role | Who | Example |
|------|-----|---------|
| `system` | Instructions for the LLM | "You are a crypto trading expert" |
| `user` | You (or your user) | "Should I buy Bitcoin now?" |
| `assistant` | The LLM's response | "Based on current market conditions..." |

### Conversation Example

```python
messages = [
    {
        "role": "system",
        "content": "You are a helpful crypto analyst"
    },
    {
        "role": "user",
        "content": "What's the trend for BTC?"
    },
    {
        "role": "assistant",
        "content": "BTC is currently in an uptrend..."
    },
    {
        "role": "user",
        "content": "Should I buy now?"
    }
]

response = client.messages.create(
    model="claude-sonnet-4-20250514",
    messages=messages
)
```

The LLM sees the **entire conversation history** and responds accordingly.

## Key Properties of LLMs

### 1. **Stateless**
Each API call is independent. You must send full conversation history every time.

```python
# WRONG - LLM won't remember previous call
response1 = llm.ask("What's BTC price?")
response2 = llm.ask("Should I buy?")  # ❌ No context!

# RIGHT - Include conversation history
messages = [
    {"role": "user", "content": "What's BTC price?"},
    {"role": "assistant", "content": response1},
    {"role": "user", "content": "Should I buy?"}
]
response2 = llm.ask(messages)  # ✅ Has context!
```

### 2. **Non-Deterministic**
Same prompt can give different responses each time (by design).

```python
# Call 1: "Bitcoin is bullish"
# Call 2: "Bitcoin shows positive momentum"
# Call 3: "Bitcoin is trending upward"
# All mean similar things but phrased differently
```

### 3. **Token-Based**
LLMs process text as "tokens" (roughly 4 characters = 1 token).
- More tokens = higher cost
- Each model has max token limit (e.g., 200K tokens)

## Getting API Keys

| Provider | Where to Get | Free Tier | Cost (approx) |
|----------|-------------|-----------|---------------|
| **Anthropic** | console.anthropic.com | $5 free credit | $0.01-0.03 per call |
| **OpenAI** | platform.openai.com | $5 free credit | $0.01-0.03 per call |

Both require credit card after free tier expires.

## Popular LLM Models (Jan 2026)

| Model | Provider | Best For |
|-------|----------|----------|
| **GPT-4** | OpenAI | General purpose, reasoning |
| **Claude Opus 4.5** | Anthropic | Complex tasks, long context |
| **Claude Sonnet 4** | Anthropic | Fast, balanced (recommended) |
| **Gemini Pro** | Google | Multimodal (text + images) |

## Key Takeaways

1. **LLM = API call** that returns intelligent text
2. **Messages have roles**: system, user, assistant
3. **Conversations need history** - LLMs are stateless
4. **Non-deterministic** - same input can give different outputs
5. **Cost per token** - longer conversations cost more

---

## What's Next?

Now that you understand LLMs, learn how to use **multiple agents working together**:
→ [02-multi-agent-systems.md](./02-multi-agent-systems.md)

## Try It Yourself

**Exercise**: Get an API key and make your first LLM call.

```python
# TODO: Your first LLM call
from anthropic import Anthropic

client = Anthropic(api_key="YOUR_KEY_HERE")

response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=100,
    messages=[
        {"role": "user", "content": "Explain ERC-8004 in one sentence"}
    ]
)

print(response.content[0].text)
```
