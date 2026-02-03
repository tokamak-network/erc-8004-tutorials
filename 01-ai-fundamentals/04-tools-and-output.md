# Tools and Structured Output

## What are Tools?

**Tools are functions that AI agents can call** to interact with the real world.

Without tools, an LLM can only generate text. With tools, an LLM can:
- Fetch live data
- Execute code
- Send emails
- Make trades
- Call APIs
- Interact with databases

## Simple Analogy

```
Human Worker                    AI Agent
─────────────                   ────────

Brain + Tools:                  LLM + Tools:
├─ Hands (grab things)          ├─ get_price() [fetch data]
├─ Calculator (math)            ├─ calculate() [compute]
├─ Phone (communicate)          ├─ send_message() [send data]
└─ Computer (access info)       └─ search_web() [get info]

Without tools → Just thinking    Without tools → Just text
With tools → Can take action     With tools → Can take action
```

## How Tool Calling Works

### The Flow

```
1. User: "What's the Bitcoin price?"
         ↓
2. LLM thinks: "I need to call get_price tool"
         ↓
3. YOUR CODE: Executes get_price("BTC") → Returns "$45,000"
         ↓
4. LLM sees result: "$45,000"
         ↓
5. LLM responds: "Bitcoin is currently $45,000"
```

### Example: Price Tool

```python
from crewai.tools import BaseTool

class CryptoPriceTool(BaseTool):
    name = "get_crypto_price"
    description = "Get the current price of a cryptocurrency"

    def _run(self, symbol: str) -> str:
        """
        Args:
            symbol: The crypto symbol (e.g., "BTC", "ETH")
        Returns:
            Current price as string
        """
        # Your code to fetch price
        import requests

        response = requests.get(
            f"https://api.coinbase.com/v2/prices/{symbol}-USD/spot"
        )
        data = response.json()
        price = data['data']['amount']

        return f"{symbol} is currently ${price}"

# Agent uses this tool
agent = Agent(
    role="Crypto Analyst",
    tools=[CryptoPriceTool()]
)
```

### The LLM's Perspective

When you give tools to an LLM, it sees something like this:

```
You have access to these tools:

1. get_crypto_price(symbol: str) -> str
   Description: Get the current price of a cryptocurrency
   Args:
       - symbol (string): Crypto symbol like "BTC" or "ETH"

2. get_sentiment(symbol: str) -> str
   Description: Get market sentiment score (0-100)
   Args:
       - symbol (string): Crypto symbol

Use tools when needed. Call them in this format:
Action: get_crypto_price
Action Input: {"symbol": "BTC"}
```

The LLM decides WHEN and HOW to use tools based on the task.

## Multiple Tool Calls

Agents can call multiple tools in sequence:

```python
User: "Should I buy Bitcoin?"
      ↓
Agent: "I need price and sentiment"
      ├─ Calls get_crypto_price("BTC") → "$45,000"
      └─ Calls get_sentiment("BTC") → "75/100 (Bullish)"
      ↓
Agent: "BTC is $45k with 75% bullish sentiment. Consider buying."
```

## Common Tool Types

| Tool Type | Example | Use Case |
|-----------|---------|----------|
| **Data Fetching** | `get_price()`, `get_news()` | Get external data |
| **Calculation** | `calculate_rsi()`, `compute_risk()` | Complex math |
| **Execution** | `place_trade()`, `send_email()` | Take actions |
| **Search** | `search_web()`, `query_db()` | Find information |
| **Validation** | `verify_signature()`, `check_balance()` | Verify data |

## Structured Output

Sometimes you don't want a text response - you want **structured data** (JSON, objects, etc.).

### Without Structured Output (Raw Text)

```python
response = llm.ask("Analyze Bitcoin")
# Returns: "Bitcoin looks bullish with price at..."
# Problem: Hard to parse, inconsistent format
```

### With Structured Output (JSON)

```python
from pydantic import BaseModel

class Analysis(BaseModel):
    symbol: str
    price: float
    sentiment: str  # "bullish", "bearish", "neutral"
    recommendation: str  # "buy", "sell", "hold"
    confidence: int  # 0-100

response = llm.ask("Analyze Bitcoin", response_format=Analysis)
# Returns:
# {
#     "symbol": "BTC",
#     "price": 45000,
#     "sentiment": "bullish",
#     "recommendation": "buy",
#     "confidence": 85
# }

# Now you can use it programmatically!
if response.recommendation == "buy" and response.confidence > 80:
    execute_trade()
```

### Benefits of Structured Output

✅ **Consistent format** - Always get the same structure
✅ **Easy parsing** - No string manipulation needed
✅ **Type safety** - Know what fields exist
✅ **Validation** - Pydantic validates data automatically
✅ **Integration** - Easily pass to other functions/contracts

## AI Agents vs Bots/Scripts

Understanding the difference is crucial:

### Traditional Bot (Fixed Rules)

```python
def trading_bot(price):
    if price < 44000:
        return "BUY"
    elif price > 50000:
        return "SELL"
    else:
        return "HOLD"

# Behavior:
trading_bot(43000)  # Always "BUY"
trading_bot(43000)  # Always "BUY"
trading_bot(43000)  # Always "BUY"
```

### AI Agent (Reasoning)

```python
def trading_agent(price):
    # LLM analyzes context
    analysis = llm.analyze(
        price=price,
        tools=[get_news, get_sentiment, get_volume]
    )
    return analysis.recommendation

# Behavior:
trading_agent(43000)  # "BUY - price low, sentiment positive"
trading_agent(43000)  # "HOLD - negative news about regulation"
trading_agent(43000)  # "BUY SMALL - uncertain market conditions"
```

### Key Differences

```
BOT:                              AI AGENT:
────                              ─────────

Fixed logic                       Adaptive reasoning
if/else statements                LLM decides based on context

No context awareness              Considers full situation
price < 44000 → BUY               "Price is low BUT news is bad..."

Deterministic                     Non-deterministic
Same input → Same output          Same input → Considers context

Cannot explain                    Can explain reasoning
return "BUY"                      "Buy because X, Y, Z..."

Fast & cheap                      Slower & costs money
Milliseconds, $0                  Seconds, ~$0.01-0.03

Predictable                       Unpredictable (by design)
Always behaves same way           Adapts to new information
```

## When to Use Which?

| Use Bot/Script | Use AI Agent |
|----------------|--------------|
| Fixed business rules | Complex decision making |
| High-frequency operations | Quality over speed |
| Deterministic behavior needed | Adaptation to context needed |
| Cost-sensitive | Value accuracy/reasoning |
| Simple if/else logic | Multi-factor analysis |

### Hybrid Approach (Best!)

```python
# Use BOTH together!

# Bot: Fast, fixed logic for obvious cases
if price < 40000:
    return "DEFINITELY BUY"  # No-brainer
elif price > 60000:
    return "DEFINITELY SELL"  # No-brainer

# Agent: Complex reasoning for unclear cases
else:
    return ai_agent.analyze(price)  # Needs thinking
```

## Real Example: ERC-8004 Agent

```python
class TrustlessAgent:
    def __init__(self):
        self.llm = Anthropic()
        self.tools = [
            PriceTool(),
            NewsTool(),
            OnChainTool(),  # Check blockchain data
            ReputationTool()  # Check agent reputation
        ]

    def analyze_market(self, symbol):
        # LLM uses tools to gather data
        prompt = f"Analyze {symbol}. Use tools to get data."

        response = self.llm.messages.create(
            model="claude-sonnet-4-20250514",
            messages=[{"role": "user", "content": prompt}],
            tools=self.tools
        )

        # LLM called tools, analyzed data, returned recommendation
        return response

# This agent can:
# ✅ Fetch live prices
# ✅ Read news sentiment
# ✅ Check on-chain metrics
# ✅ Reason about what to do
# ✅ Explain its reasoning
```

## Key Takeaways

1. **Tools = Functions** that agents can call
2. **LLM decides when** to use tools based on task
3. **Structured output** = JSON/objects instead of text
4. **Agents ≠ Bots** - agents reason, bots follow rules
5. **Hybrid approach** - use both bots and agents together

---

## Section Complete! 🎉

You now understand AI fundamentals. Next, learn what ERC-8004 is:
→ **[../02-erc8004-basics/README.md](../02-erc8004-basics/README.md)**

## Try It Yourself

**Exercise**: Create a tool that fetches Ethereum gas prices

```python
from crewai.tools import BaseTool
import requests

class GasPriceTool(BaseTool):
    name = "get_gas_price"
    description = "Get current Ethereum gas price in Gwei"

    def _run(self) -> str:
        # TODO: Implement this!
        # Hint: Use etherscan.io API or web3.py
        pass

# Test it
tool = GasPriceTool()
print(tool._run())  # Should print current gas price
```

<details>
<summary>Solution</summary>

```python
from crewai.tools import BaseTool
import requests

class GasPriceTool(BaseTool):
    name = "get_gas_price"
    description = "Get current Ethereum gas price in Gwei"

    def _run(self) -> str:
        response = requests.get(
            "https://api.etherscan.io/api"
            "?module=gastracker"
            "&action=gasoracle"
            "&apikey=YourApiKey"
        )
        data = response.json()
        gas_price = data['result']['SafeGasPrice']
        return f"Current gas price: {gas_price} Gwei"
```
</details>
