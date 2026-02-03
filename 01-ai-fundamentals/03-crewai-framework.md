# CrewAI Framework

## What is CrewAI?

**CrewAI** is a Python framework for building multi-agent systems. It makes it easy to create specialized agents that work together as a team (crew).

Think of it as **Foundry for AI agents** - a framework that handles the complex parts so you can focus on what your agents do.

## The 4 Core Building Blocks

```
┌─────────────┬─────────────┬─────────────┬──────────────┐
│   AGENT     │    TASK     │    TOOL     │    CREW      │
│   "Who"     │   "What"    │   "How"     │   "Team"     │
├─────────────┼─────────────┼─────────────┼──────────────┤
│  The worker │  The job    │ The skills  │ Orchestrates │
│  Has a role │  Has a goal │ Functions   │ Runs tasks   │
└─────────────┴─────────────┴─────────────┴──────────────┘
```

### 1. **Agent** - Who does the work
```python
from crewai import Agent

analyst = Agent(
    role="Crypto Market Analyst",
    goal="Analyze crypto markets accurately",
    backstory="Expert trader with 10 years experience",
    tools=[price_tool, news_tool]
)
```

### 2. **Task** - What needs to be done
```python
from crewai import Task

analysis_task = Task(
    description="Analyze Bitcoin price trends for the next week",
    agent=analyst,
    expected_output="Detailed analysis with buy/sell recommendation"
)
```

### 3. **Tool** - How agents interact with the world
```python
from crewai.tools import BaseTool

class PriceTool(BaseTool):
    name = "get_crypto_price"
    description = "Get current price of a cryptocurrency"

    def _run(self, symbol: str) -> str:
        # Your code to fetch price
        price = fetch_price_from_api(symbol)
        return f"{symbol} is currently ${price}"
```

### 4. **Crew** - The team that runs everything
```python
from crewai import Crew

crew = Crew(
    agents=[analyst, researcher],
    tasks=[analysis_task, research_task],
    verbose=True  # See what agents are doing
)

result = crew.kickoff()
```

## Complete Working Example

```python
from crewai import Agent, Task, Crew
from crewai.tools import BaseTool

# Step 1: Define Tools
class PriceTool(BaseTool):
    name = "get_price"
    description = "Get current crypto price"

    def _run(self, symbol: str) -> str:
        # Simplified - in reality, call an API
        prices = {"BTC": 45000, "ETH": 2500}
        return f"{symbol}: ${prices.get(symbol, 'Unknown')}"

class SentimentTool(BaseTool):
    name = "get_sentiment"
    description = "Get market sentiment (0-100)"

    def _run(self, symbol: str) -> str:
        # Simplified
        return f"{symbol} sentiment: 75/100 (Bullish)"

# Step 2: Create Agents
analyst = Agent(
    role="Crypto Analyst",
    goal="Provide accurate market analysis",
    backstory="Expert in technical analysis",
    tools=[PriceTool(), SentimentTool()],
    verbose=True
)

writer = Agent(
    role="Report Writer",
    goal="Write clear, actionable reports",
    backstory="Financial journalist",
    tools=[],
    verbose=True
)

# Step 3: Define Tasks
analyze_task = Task(
    description="Analyze Bitcoin. Check price and sentiment.",
    agent=analyst,
    expected_output="Analysis with recommendation"
)

write_task = Task(
    description="Write a concise report based on the analysis",
    agent=writer,
    expected_output="2-paragraph report",
    context=[analyze_task]  # Uses output from analyze_task
)

# Step 4: Create Crew
crew = Crew(
    agents=[analyst, writer],
    tasks=[analyze_task, write_task],
    verbose=True
)

# Step 5: Run!
result = crew.kickoff()
print(result)
```

### What Happens When You Run This?

```
1. Analyst agent starts
   └─ "I need to check BTC price"
   └─ Calls PriceTool("BTC") → "BTC: $45,000"
   └─ "I need sentiment"
   └─ Calls SentimentTool("BTC") → "Sentiment: 75/100"
   └─ Thinks: "Price is high, sentiment is bullish..."
   └─ Returns: "BTC shows strength, consider buying"

2. Writer agent starts
   └─ Receives analyst's output
   └─ Thinks: "How should I structure this?"
   └─ Returns: "Bitcoin Analysis Report..."

3. Final result printed to console
```

## LLM Integration

CrewAI supports multiple LLMs. You configure which one to use:

### Option 1: OpenAI (GPT-4)

```python
import os
from crewai import Agent

os.environ["OPENAI_API_KEY"] = "sk-..."

# CrewAI uses OpenAI by default
agent = Agent(
    role="Analyst",
    goal="Analyze markets",
    llm="gpt-4"  # or "gpt-3.5-turbo" for cheaper
)
```

### Option 2: Anthropic (Claude)

```python
import os
from crewai import Agent, LLM

os.environ["ANTHROPIC_API_KEY"] = "sk-ant-..."

# Use Claude
llm = LLM(
    model="claude-sonnet-4-20250514",
    api_key=os.environ["ANTHROPIC_API_KEY"]
)

agent = Agent(
    role="Analyst",
    goal="Analyze markets",
    llm=llm
)
```

### Option 3: Local LLM (Ollama - Free!)

```python
from crewai import Agent, LLM

# No API key needed!
llm = LLM(
    model="ollama/llama2",
    base_url="http://localhost:11434"
)

agent = Agent(
    role="Analyst",
    goal="Analyze markets",
    llm=llm
)
```

## Cost Comparison

| LLM Provider | Cost per Call | Quality | Speed |
|-------------|---------------|---------|-------|
| **GPT-4** | ~$0.03 | Excellent | Fast |
| **GPT-3.5-turbo** | ~$0.002 | Good | Very Fast |
| **Claude Sonnet 4** | ~$0.015 | Excellent | Fast |
| **Claude Haiku** | ~$0.001 | Good | Very Fast |
| **Ollama (local)** | $0 (free!) | Good | Depends on hardware |

**Recommendation for learning**: Start with GPT-3.5-turbo or local Ollama (both cheap/free).

## Installation

```bash
# Install CrewAI
pip install crewai

# Install LLM SDKs (choose what you need)
pip install openai          # For GPT
pip install anthropic       # For Claude
```

## Common Patterns

### Pattern 1: Sequential Tasks

```python
task1 = Task(description="Get data", agent=data_agent)
task2 = Task(description="Analyze", agent=analyst, context=[task1])
task3 = Task(description="Report", agent=writer, context=[task2])

# Runs in order: task1 → task2 → task3
```

### Pattern 2: Parallel + Combine

```python
# These run in parallel
tech_task = Task(description="Technical analysis", agent=tech_agent)
news_task = Task(description="News analysis", agent=news_agent)

# This waits for both
final_task = Task(
    description="Combine analyses",
    agent=decision_agent,
    context=[tech_task, news_task]
)
```

## CrewAI vs Raw LLM Calls

| Aspect | Raw LLM Calls | CrewAI |
|--------|--------------|--------|
| **Setup** | Manual everything | Framework handles it |
| **Multi-Agent** | You code coordination | Built-in orchestration |
| **Tools** | You code tool calling | Simple decorators |
| **History** | You manage conversations | Automatic |
| **Error Handling** | You code retries | Built-in |

**When to use CrewAI**: Multi-agent systems, complex workflows
**When to use raw LLM**: Simple single-agent tasks, full control needed

## Key Takeaways

1. **CrewAI = Framework** for multi-agent systems (like Foundry for AI)
2. **4 components**: Agent, Task, Tool, Crew
3. **Supports multiple LLMs**: OpenAI, Anthropic, local
4. **Tools = Functions** agents can call
5. **Context = Task dependencies** (sequential workflows)

---

## What's Next?

Learn how agents use tools and produce structured output:
→ [04-tools-and-output.md](./04-tools-and-output.md)

## Try It Yourself

```bash
# Install CrewAI
pip install crewai openai

# Create test_crew.py with the complete example above
# Set your OpenAI key
export OPENAI_API_KEY="sk-..."

# Run it!
python test_crew.py
```

**Challenge**: Modify the example to analyze Ethereum instead of Bitcoin!
