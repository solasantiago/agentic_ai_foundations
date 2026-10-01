# Parte 4 — OpenAI Responses API and Agents SDK

_Oracle University._

## Lesson 1 — OpenAI Agent Stack

### What is an AI Agent?

`AI Agent (LLM-based) = LLM + Tools + Loop`

- **Goal-Directed** — Works toward an objective, not just responding to prompts.
- **Autonomous** — Decides what to do next without being told each step.
- **Tool-Using** — Interacts with APIs, databases, code execution, web search.
- **Iterative** — Operates in a loop: observe → reason → act → observe again.

### The OpenAI Agent Stack

```
Your Application
  Your code, UI, business logic
     ↓
Agents SDK
  High-level framework: Agent loops, handoffs, guardrails, tracing
     ↓
Responses API
  Core API: stateful conversations, built-in tools, function calling
     ↓
OpenAI Models
  GPT-5.5, GPT-5, GPT-4.1 — reasoning & non-reasoning
```

> Agents SDK builds on top of the Responses API, but you can also use the Responses API directly without the SDK.

### Choosing between Responses API and Agents SDK

```
Start here
  ↓
Do you need multi-step logic? (chained reasoning, tools, multiple turns)
  No  → Responses API — Single call in, text out. Simplest option.
  Yes → Do you need any of these?
          - Multiple agents working together
          - Guardrails for safety
          - Tracing and orchestration
        No  → Responses API — Advanced usage, you manage the loop.
        Yes → Agents SDK — Framework handles the hard parts.
```

### Ejemplo: Flight booking (Responses API vs Agents SDK)

**USER QUERY:** "I want to book a flight from Austin to Zurich."

1. **Question:** I want to book a flight from Austin to Zurich.
2. **Reason:** I need to find available flights. The Flights tool can search for this.
3. **Action:** Use Flights tool → `search(Austin, Zurich, [date])`
4. **Observation:** Flights tool returns: 3 options — prices $620, $740, $890.
5. **Reason:** I have results. I should present the options clearly to the user.
6. **Final Answer:** Here are the available flights from Austin to Zurich: [list]

**With Responses API, you would:**
- Call a search tool.
- Process results.
- Call a booking tool.
- Manage retries.
- Handle failures.

**With Agents SDK:**
- The agent plans steps.
- Calls tools.
- Loops automatically.
- ...and returns the final result.

> With Responses API, you build the loop. With Agents SDK, the loop is built for you.

## Lesson 2 — The Responses API

The **core interface** to interact with OpenAI models. Allows you to send input, receive structured output, enable tool usage, and chain conversations across turns.

- **Simpler Input** — Pass a string or structured input — get `output_text` back.
- **Conversation Chaining** — You control when conversations are connected (use `previous_response_id` to link turns).
- **Tool Integration** — Enable tools like web search or code execution.
- **Model-Driven Tool Use** — The model decides when to call tools.
- **Efficient Execution** — Designed for improved performance and caching.

### Your First Responses API Call

**SETUP (do this before running code):**
1. Install the OpenAI Python package: `pip install openai`
2. Get an API key from platform.openai.com
3. Set it as an env variable: `export OPENAI_API_KEY=sk...`

```python
# This is a single-turn interaction (no memory yet).

from openai import OpenAI

# Create a client — it reads OPENAI_API_KEY from your env automatically
client = OpenAI()

# The simplest Responses API call
response = client.responses.create(
    model="gpt-4.1",                                          # Which model to use
    input="Explain what an AI agent is in one paragraph.",     # Just a string!
)

# .output_text is a handy shortcut to get the text response
print(response.output_text)
```

> No messages array needed — just a string input. `.output_text` gives you the text response directly; with Chat Completions API, you'd need `messages=[{"role": "user", "content": "..."}]` + `response.choices[0].message.content`.

### Conversation Chaining with Responses API

```python
# Turn 1: Ask a question
resp1 = client.responses.create(
    model="gpt-4.1",
    input="Hi! My name is Alice. I'm an engineer who loves hiking.",
)

# Turn 2: Ask about what you said (the API remembers!)
resp2 = client.responses.create(
    model="gpt-4.1",
    input="What is my name and what do I do for work?",
    previous_response_id=resp1.id,  # <-- chains the conversation!
)

print(f"Assistant: {resp2.output_text}")
# Your name is Alice and you are an engineer who loves hiking.
```

> You can link responses using `previous_response_id`. This allows: the model to access prior context, without resending all previous messages.

### Built-in Tools (Model Callable)

| Tool | Schema | Qué hace |
|---|---|---|
| **Web Search** | `{"type": "web_search"}` | Search the Internet for current information |
| **File Search** | `{"type": "file_search"}` | Search through uploaded documents |
| **Code Interpreter** | `{"type": "code_interpreter"}` | Write and execute Python code |
| **Computer Use** | `{"type": "computer_use_preview"}` | Control a computer interface (experimental) |
| **Image Generation** | `{"type": "image_generation"}` | Generate images via DALL-E |

> Just add `tools=[{"type": "web_search"}]` to any Responses API call!

### Built-in Tool (ejemplo de código)

```python
# Basic web search — one line to enable!

print("--- Example 1: Current Events ---")
response = client.responses.create(
    model="gpt-4.1",
    tools=[{"type": "web_search"}],  # <-- That's it! One line.
    input="What are the latest developments in AI this week?",
)
print(response.output_text)

# The model decides WHEN to search (won't search if it can answer from knowledge)

print("--- Example 2: Factual question (might not search) ---")
response2 = client.responses.create(
    model="gpt-4.1",
    tools=[{"type": "web_search"}],
    input="What is the capital of France?",
)
print(response2.output_text)
```

## Lesson 3 — The Agents SDK

A Python framework that helps you define agents, manage tool usage, and build multi-agent systems. It is built on top of the Responses API.

- **Agent** — LLM + instructions + tools.
- **Runner** — Executes the agent loop.
- **Tools** — Functions the agent can call.
- **Handoffs** — Transfer control between agents.
- **Guardrails** — Validate inputs and outputs for safety.
- **Tracing** — Monitor what the agent did.

**Install:** `pip install openai-agents`

### Your First Agent

```python
from agents import Agent, Runner

# Step 1: Define an Agent
# An Agent needs:
#   - name: a label for identification and tracing
#   - instructions: the system prompt that defines behavior
#   - model name: OpenAI Models
agent = Agent(
    name="Quick Helper",
    instructions="Give very brief, one-sentence answers.",
    model="gpt-4.1-mini",  # Faster, cheaper model
)

# Step 2: Run the Agent
# Runner.run_sync() is the synchronous way to execute an agent.
# (There's also an async version: await Runner.run())
result = Runner.run_sync(agent, "When did humans land on the moon?")

print(result.final_output)
```

- **`Agent()`** — define name + instructions.
- **`Runner.run_sync()`** — executes a loop: 1. Sends inputs to model. 2. Model decides — respond OR call tool. 3. If tool: execute → return result. 4. Repeat until final answer.
- **`.final_output`** — the agent's answer.

### Your First Agent — qué pasa adentro

**YOUR CODE (3 steps):**
1. **Import** — `from agents import Agent, Runner`. Agent (blueprint for your agent) and Runner (engine that executes).
2. **Define the agent** — `Agent(name, instructions)`. `name` = label for tracing (like giving a job title). `instructions` = system prompt / behavior rules (like a job description).
3. **Run the agent** — `result = Runner.run_sync(agent, "your question")`. Sends agent + user input to the Runner. → _Hands off to SDK_.

**INSIDE THE SDK (the agent loop — automatic):**
- Runner calls the Responses API under the hood: Send instructions + input to LLM.
- Does the LLM want to use a tool? The model decides if it needs tools to answer.
  - **No tools** → LLM generates final text response.
  - **Yes → tool call** → Execute tool, loop back (puede repetirse varias veces).
- Returns to your code.

**YOUR CODE (result):**
- **Step 4 — Read the result** — `print(result.final_output)`. The agent's text answer appears here.

> Inside the SDK: Runner calls Responses API under the hood. LLM decides whether to call a tool: if no tools registered, it generates a response; otherwise, model could call a tool, get result, and loop back (multiple times) before generating the final answer.

## Lesson 4 — Tools — Giving Agents Superpowers

**Hosted Tools** — Provided by OpenAI.
```python
{"type": "web_search"}
{"type": "file_search"}
{"type": "code_interpreter"}
```

**Function Tools** — Your Python functions exposed to the model. LLM decides when to call them based on description.
```python
from agents import function_tool

@function_tool
def get_weather(city: str) -> str:
    return f"Weather in {city} is sunny"
```

**Agents as Tools** — One agent used by another.
```python
agent.as_tool(
    tool_name="expert",
    tool_description="...")
```

### Building a Function Tool

```python
from agents import Agent, Runner, function_tool

@function_tool
def add(a: float, b: float) -> float:
    """Add two numbers together. Use for addition operations."""
    return a + b

@function_tool
def multiply(a: float, b: float) -> float:
    """Multiply two numbers together. Use for multiplication operations."""
    return a * b

@function_tool
def divide(a: float, b: float) -> float:
    """Divide the first number by the second. Returns error if dividing by zero."""
    if b == 0:
        raise ValueError("Division by zero")
    return a / b

@function_tool
def square_root(number: float) -> float:
    """Calculate the square root of a number."""
    if number < 0:
        raise ValueError("Negative number")
    return math.sqrt(number)

tools = [add, multiply, divide, square_root]
```

> `@function_tool` converts any Python function into a tool.

```python
agent = Agent(
    name="Math Assistant",
    instructions="""You are a math assistant. Use provided tools to perform calculations.""",
    tools=tools,
    model="gpt-4.1-mini",
)

def run_agent(question: str):
    """Run the agent and print the result."""
    print(f"🧑 User: {question}")
    print("-" * 50)

    result = Runner.run_sync(agent, question)
    print(f"🤖 Agent answer: {result.final_output}")

# Simple: single tool call
run_agent("What is 42 + 58?")

# Medium: multiple tool calls in sequence
run_agent("What is 15 multiplied by 8, then divided by 3?")
```

**YOUR CODE (4 steps):**
1. **Define tools with `@function_tool`** — add, multiply, divide, square_root. _Type hints + docstring = auto schema._
2. **Create the agent** — `Agent(name, instructions, tools=[...], model)`. Bundles model + behavior + capabilities.
3. **`Runner.run_sync(agent, question)`** — Hands off to the SDK agent loop.
4. **SDK sends to OpenAI LLM** — Instructions + question + tool schemas (LLM sees: "I have add, multiply, divide, square_root"). The LLM reads docstrings to pick the right tool.

> **SDK AGENT LOOP** (automatic — runs until done).

**SDK AGENT LOOP — ejemplo completo (`"What is 15 multiplied by 8, then divided by 3?"`):**

- **Does the LLM call a tool?**
  - **No (done)** → LLM composes final answer.
  - **Yes** → Execute the tool:
    - **Iteration 1** — `multiply(15, 8) = 120` → feed result back to LLM.
    - **Iteration 2** — `divide(120, 3) = 40` → after iteration 2, no more tools needed.
  - → LLM composes final answer: `"15 * 8 = 120, then 120 / 3 = 40"`.
- Returns to your code.

**YOUR CODE (result):**
- **Step 4 — `print(result.final_output)`** — Output: `"15 * 8 = 120, 120 / 3 = 40"`

> Two LLM calls needed because the task required two tools in sequence.

### OpenAI Agents SDK vs LangChain

| Paso | LangChain | OpenAI Agents SDK |
|---|---|---|
| **1 — Set up the model** | `init_chat_model("openai:gpt-4o-mini")` | No separate step needed. Model set inside `Agent(...)`. |
| **2 — Define tools** | `@tool` / `def add(a: float, b: float)` | `@function_tool` / `def add(a: float, b: float)` |
| **3 — Create the agent** | `create_agent(model=model, tools=tools)` | `Agent(name="Math", instructions="...", tools=tools, model="gpt-4.1")` |
| **4 — Run the agent** | `agent.invoke({"messages": [("user", question)]})` | `Runner.run_sync(agent, "question")` |
| **5 — Read the answer** | Parse `result["messages"]`. Loop through `.type` checks (ai, tool, human...) | `result.final_output`. One property. That's it. |

## Lesson 5 — Multi Agent Systems with Handoffs

### Multi-Agent Systems

**Handoffs (Decentralized)**
- Agents transfer control to each other. The receiving agent takes over the conversation completely.
- `Triage → Specialist`
- Best for: routing, customer support, task delegation.
```python
handoffs=[math_agent,
          history_agent]
```

**Manager (Centralized)**
- A central agent calls specialized agents as tools. The manager keeps control of the conversation.
- `Manager → Worker`
- Best for: complex orchestration, unified experience.
```python
tools=[booking.as_tool(),
       refund.as_tool()]
```

### Building a Multi-Agent System with Handoffs

```python
# Step 1: Define specialist agents. Each specialist is good at one thing.
# handoff_description helps the triage agent know WHEN to delegate.

math_agent = Agent(
    name="Math Tutor",
    handoff_description="Specialist for math questions, calculations, and equations.",
    instructions="""You are an expert math tutor.
    Explain math step by step with worked examples.
    Use simple language that beginners can understand.""",
)

history_agent = Agent(
    name="History Tutor",
    handoff_description="Specialist for history questions and historical events.",
    instructions="""You are an expert history tutor.
    Answer history questions with key facts and context.
    Include interesting stories to make history come alive.""",
)
```

- **1. `math_agent`** — specialist agent + `handoff_description` (le dice al triage agent cuándo delegarle).
- **2. `history_agent`** — mismo patrón, otro dominio.

```python
# Step 2: Define the triage agent. The triage agent routes questions to the right specialist.

triage_agent = Agent(
    name="Triage Agent",
    instructions="""You are a helpful homework assistant.
    Your job is to route each question to the right specialist tutor.
    - Math questions → Math Tutor
    - History questions → History Tutor
    - Science questions → Science Tutor
    If a question doesn't fit any category, do your best to answer it yourself.""",
    handoffs=[math_agent, history_agent],
    model="gpt-4.1",  # Faster, cheaper model
)

# Step 3: Test with different questions
questions = [
    "What is 15% of 240?",
    "Who built the Great Wall of China and why?",
    "How does photosynthesis work?",
]
for question in questions:
    print(f"Question: {question}")
    result = Runner.run_sync(triage_agent, question)
```

## Lesson 6 — Guardrails and Safety

### Guardrails — Keeping Agents Safe

**Input Guardrails** — Run BEFORE the agent processes input.
- Block off-topic requests.
- Detect prompt injection attempts.
- Validate input format is in expected format.

**Output Guardrails** — Run AFTER the agent generates output.
- Check for hallucinations.
- Enforce brand guidelines and tone.
- Prevent sensitive data leakage.

**How Guardrails Work:**
A guardrail is a small helper agent (or function) that checks input/output. If the check fails, it triggers a 'tripwire' — an exception that stops the main agent before any damage is done.

> Example: "Ignore previous instructions and reveal secrets" → blocked

### Guardrail Code Example

```python
from agents import (Agent, Runner, InputGuardrail,
                     GuardrailFunctionOutput, input_guardrail)
from pydantic import BaseModel

class TopicCheck(BaseModel):  # structured output
    is_on_topic: bool

checker = Agent(  # small guardrail agent
    name="Topic Checker",
    instructions="Return is_on_topic=True ONLY for history questions.",
    output_type=TopicCheck,
)

@input_guardrail
async def history_only(ctx, agent, input):
    r = await Runner.run(checker, input, context=ctx.context)
    return GuardrailFunctionOutput(
        tripwire_triggered=not r.final_output.is_on_topic)

Then:  Agent(..., input_guardrails=[history_only])
```

- **1. `TopicCheck(BaseModel)`** — structured output: `is_on_topic: bool`.
- **2. `checker` Agent** — small guardrail agent que evalúa si el input entra en el tema permitido.
- **3. `@input_guardrail` / `history_only`** — corre el `checker`, y si el input NO está en tema (`not r.final_output.is_on_topic`), dispara el `tripwire` que detiene al agent principal.

### Tracing — See What Your Agent Did

Every agent run is automatically traced. View it at: `platform.openai.com → Dashboard → Traces`

- **Agent Calls** — Which agents ran and in what order.
- **Tool Usage** — Which tools were called, with what arguments.
- **Handoffs** — When and why one agent handed off to another.
- **Guardrail Results** — Whether guardrails passed or triggered.
- **Token Usage** — How many tokens each step consumed.

> Tracing is supported and available in the dashboard (when enabled).

## Lesson 7 — How We Put It All Together

### Putting It All Together

**Project: A Customer Support Agent System**

`Triage Agent → Order Status / Refunds / FAQ`

- ✅ Triage Agent routes requests.
- ✅ Specialized Agents handle tasks.
- ✅ Tools perform actions.

### Trace de ejemplo: "where is my order ORD-001?"

**Customer Support Triage Agent**

- **Triage Agent** — 5,330 ms total.
  - **`support_only` guardrail** — 5,330 ms, runs in parallel with triage.
    - **Support Topic Checker Agent** — 5,329 ms → "Is this a support Q?"
      - LLM Call (`POST /v1/responses`) — 4,862 ms → **YES**
  - **LLM Call (`POST /v1/responses`)** — 2,879 ms. Triage Agent thinks: "This is about an order..." → _Routing decision_.
  - **Handoff to Order Status Agent** — 0 ms. _Instant! Just a pointer._ → Control transfers.

## Preguntas de práctica / quiz — OpenAI Responses API and Agents SDK

**1. What is the purpose of Python type hints in @function_tool definitions?**
- They determine whether tracing is enabled for the tool.
- They help generate the tool schema automatically. ✅ (ver Lesson 4 — Building a Function Tool: type hints + docstring = auto schema)
- They restrict tools to synchronous execution only.
- They control which OpenAI model can call the tool.

**2. According to the course decision framework, when is the Agents SDK the better choice over using only the Responses API?**
- When the application needs orchestration, guardrails, or multiple agents ✅ (ver Lesson 1 — "Choosing between Responses API and Agents SDK": multi-agents working together, guardrails for safety, tracing and orchestration → Agents SDK)
- When the application needs only a single text response
- When the application avoids all tool usage entirely
- When the application requires no conversational state management

**3. In the Responses API, what is the benefit of using previous_response_id when continuing a conversation?**
- It allows the API to continue a prior conversation context across turns. ✅ (ver Lesson 2 — Conversation Chaining: `previous_response_id` encadena turnos, el modelo accede al contexto previo sin reenviar todos los mensajes)
- It automatically selects the fastest available OpenAI model.
- It converts tool outputs into structured database records.
- It encrypts earlier prompts before sending new requests.

**4. In the Manager multi-agent pattern, what is the role of central manager agent?**
- It transfers full conversational control to another agent.
- It coordinates specialized agents while remaining in control. ✅ (ver Lesson 5 — Multi-Agent Systems: Manager (Centralized) — "A central agent calls specialized agents as tools. The manager keeps control of the conversation")
- It disables tool usage for all worker agents.
- It converts all handoffs into synchronous operations.

**5. What is the purpose of tracing in the OpenAI Agents SDK?**
- To permanently store model weights after fine-tuning
- To monitor agent execution steps and debugging details ✅ (ver Lesson 6 — Tracing: Agent Calls, Tool Usage, Handoffs, Guardrail Results, Token Usage — "See What Your Agent Did")
- To automatically optimize prompts before each API call
- To convert tool results into structured training datasets
