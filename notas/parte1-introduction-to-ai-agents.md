# Parte 1 — Introduction to AI Agents

## Three Core Components of Every Agent

**LLM**
- Language model (LM) at the centro of decision-making.
- Can be one or multiple models of any size.
- Can be prompted with reasoning strategies like CoT y ReAct.
- Same LLM can power different agents with different tool configurations — no retraining needed.

**Tools**
- Bridge between the agent and the external world.
- Enable real-time data access and real-world actions.
- Can wrap any function: API calls, database queries, file operations.
- Unlock RAG and many other specialized capabilities.

**Loop (Orchestration)**
- The 'thinking loop' — cyclical process governing decisions.
- Manages memory (short-term and long-term).
- Maintains state across multi-turn interactions.
- Continues looping until goal is achieved or stopped.

## 1: LLM — Agent's Brain

**What LLM does inside an Agent**
- Understands user intent from natural language.
- Plans multi-step sequences to achieve the goal.
- Decides which tool to call and with what arguments.
- Interprets tool results and determines next action.
- Generates the final response to the user.

**Capabilities, Cost, Latency Tradeoffs**
- Model must reliably follow structured instructions.
- Large context window is preferred.
- Strong reasoning capabilities help complex tasks.
- Each loop iteration = 1+ LLM API call.
- Two-tier pattern: cheap model for routing, capable model for reasoning.
- Newer reasoning models have built-in chain-of-thought training, which can improve agent reasoning.

> The LLM is the reasoning core — but selecting the right one is more nuanced than picking the highest benchmark score.

## 2: Tools — Agent's Hands

1. **Agent Defines Tools** — JSON schemas.
2. **Agent Sends Tools to LLM** — Tools + user query.
3. **LLM Decides** — Text or tool call.
4. **Agent Executes Tool** — Your code runs it.
5. **Agent Returns Results to LLM** — Back to LLM.

**Critical Security Boundary**
- LLM NEVER executes tools directly.
- It emits a structured tool-call request, and your application validates and executes it.
- MCP provides a standardized way to connect LLMs with the context they need, including resources, prompts, and tools.

> Tools transform a language model from a text generator into an actor — capable of reading information and executing actions.

## 3. Loop (Orchestration) — Agent's Nervous System

- **Loop Management** — Runs the Think-Act-Observe cycle. Knows when to continue, pause, or stop.
- **Reasoning Strategy** — Applies Chain-of-Thought, ReAct, or other prompting frameworks to break goals into steps.
- **Memory Management** — Short-term scratchpad for the current session; query stored history, documents, or KBs for longer-term context.
- **Tool Routing** — Selects which tool to call, formats the call, handles errors, and feeds results back.
- **State Machine** — Tracks where in the plan the agent is. Detects completion, loops, and failures.
- **Safety & Guardrails** — Hard-coded policy rules that override model reasoning when needed.

> The orchestration layer is the conductor of the entire agentic symphony — deciding when to reason, which tool to call, and how results inform the next step.

## The Agent Execution Loop

`PERCEIVE → REASON → ACT → OBSERVE → (loop)`

- **PERCEIVE** — Receive input or observation.
- **REASON** — Selects next step.
- **ACT** — Call tool or respond to user.
- **OBSERVE** — Receive tool result or feedback.

Loop continues until goal achieved or max iterations reached.

**Termination conditions**
- Agent decides it has the final answer.
- Maximum iteration count reached.
- Error or timeout triggers fallback.

**What can go wrong**
- Infinite loops (agent never decides it's done).
- Hallucinated tool calls (calling tools that don't exist).
- Cost explosion (too many LLM API calls).

## Lesson 3 — Patrones de razonamiento

### Major Reasoning Frameworks

**CoT — Chain-of-Thought**
- Breaks problems into a sequential chain of intermediate reasoning steps before arriving at a conclusion.
- Best for: Math, logic, step-by-step analysis.

**ReAct — Reasoning + Acting**
- Interleaves reasoning (Thought) with actions (Action) and observations. The agent uses intermediate reasoning to decide when and how to act.
- Best for: Tool selection, multi-step tasks, external API calls.

**ToT — Tree-of-Thoughts**
- Explores multiple reasoning branches simultaneously, like a search tree. Each 'thought' is evaluated before the agent decides which branch to pursue.
- Best for: Creative tasks, strategic planning, exploration.

### Chain of Thought (CoT): Thinking Step by Step

**The Core Idea**
- Instruct the LLM to reason step-by-step before answering.
- Dramatically improves accuracy on complex tasks.
- Transparency (we can see how the answer was derived).
- Debuggability (we can spot where reasoning goes wrong).
- Zero-shot: Add "Let's think step by step" to the prompt.
- Few-shot: Provide worked examples showing reasoning.

> Chain-of-Thought does not make the model smarter — it helps the model use its reasoning ability more reliably.

**Example: Math Reasoning**

Problem: If a store has 15 apples and sells 3 batches of 4, how many are left?

Without CoT:
- Answer: 3 (may guess or make mistakes without clear reasoning)

With CoT:
- Let me work through this.
- 3 batches of 4 = 12.
- 15 - 12 = 3.
- Answer: 3 (correct, with traceable reasoning)

### CoT limitations for Agents

**Limitations of Standalone CoT**
- Uses only internal knowledge — cannot look things up.
- If knowledge is wrong, reasoning is confidently wrong.
- Cannot self-correct against external reality.
- Cannot take actions in the real world.

**Why this matters for Agents**
- CoT gives agents the ability to "think".
- But thinking alone isn't enough — agents also need to act.
- ReAct pattern = CoT + Tool use in a loop.
- Modern reasoning models (o1, R1) do CoT internally (may not expose 'thinking' block).

### ReAct: Reasoning + Acting in a Loop

**ReAct Trace Example**
```
User: Who directed Inception and what year?
Reason: I need to search for the director of Inception.
Action: search("Inception film director and year")
Observation: Christopher Nolan directed Inception (2010).
Reason (Again): I now have both pieces of information.
Final Answer: Inception was directed by Christopher Nolan, released in 2010.
```

**Why ReAct Works**
- Reasoning traces make agent's logic transparent and debuggable.
- External tool use grounds the reasoning in real-world data, reducing hallucinations.
- Interleaved format prevents the model from "making up" answers when it should look things up.
- Outperforms both CoT-only and action-only approaches on QA and decision-making benchmarks.

## Lesson 4 — Agent Frameworks

| Framework | Descripción | Mejor para |
|---|---|---|
| **LangChain / LangGraph** | Largest ecosystem, graph-based stateful workflows | Production systems needing flexibility |
| **OpenAI Agents SDK** | Define agents and orchestrate multi-agent workflows | OpenAI Agent Stack, Built-in Tracing, Guardrails, multi-agent systems |
| **CrewAI** | Role-based multi-agent teams, intuitive design | Workflows that mirror team structures |
| **Hugging Face SmolAgents** | Minimal (~1K lines), great for learning | Education, research, prototyping |

**Practical Guidance**
- For learning: Start with a simple example to understand the internals.
- For production: LangChain/LangGraph gives you the most flexibility and ecosystem support.
- Frameworks help when complexity grows.

> **Nota al pie — Python Essentials You'll Need**
> _A quick refresher on the Python features used in the agent code that follows._
> - **Decorators (`@tool`)** — `@tool` wraps a function so the agent framework can register it as a callable tool. Think of `@` as a label you stick on a function to give it special powers — the function itself doesn't change.
> - **Type Hints (`a: float`)** — Tells the LLM what data type each parameter expects (number, string, etc.). The agent reads these hints to construct valid tool calls — wrong types = broken tools.
> - **Docstrings (`"""..."""`)** — The triple-quoted string right after a function definition describes what it does. The LLM reads docstrings to decide **when and how** to use each tool. Vague docstring = wrong tool choice.
> - **Return Types (`-> float`)** — Tells the agent what kind of data the tool sends back. Helps the agent plan how to use one tool's output as another tool's input.

## Lesson 5 — Safety and Guardrails

### AI Agent Threat Model: What Can Go Wrong

- **Prompt Injection** — Attacker hijacks AI Agent via crafted input or poisoned retrieved content. One of the most important threats for AI Agents.
- **Tool Misuse** — Agent calls tools with wrong or dangerous arguments (unauthorized emails, destructive DB queries).
- **Memory Poisoning** — Poisoned content stored in memory can affect future behavior for affected users and potentially others if memory is shared or broadly retrieved.
- **Data Exfiltration** — AI Agent tricked into leaking sensitive internal data through tool calls or responses.
- **Runaway Execution** — Infinite loops or excessive API calls cause massive cost overruns and system strain.

### Defense in Depth: Layered Guardrails Architecture

1. **Input Validation** — Treat all external/retrieved content as untrusted data, PII detection, rate limiting.
2. **LLM Guardrails** — Safety system prompts, tool access controls, low-confidence outputs get routed to human review.
3. **Tool Boundaries** — Least-privilege, input validation, sandboxing, human-in-the-loop.
4. **Output Filtering** — PII screening, content policy, relevance verification.
5. **Observability** — Log traces: inputs, tool calls, outputs, errors, and costs.

> No single layer is sufficient. Defense in depth catches what individual layers miss. Route by risk level.

## Preguntas de práctica / quiz

_Preguntas mostradas durante la sesión (presumiblemente similares a las del examen de certificación)._

**1. Why are type hints important when defining agent tools in Python?**
- They improve GPU utilization speed.
- They help generate valid tool schemas. ✅ (ver nota al pie "Python Essentials" en Lesson 4: type hints ayudan al LLM/agente a construir tool calls válidos)
- They reduce the size of prompts.
- They automatically train the agent.

**2. Which statement describes how an AI agent differs from a fixed workflow?**
- It follows predefined steps only.
- It generates responses from memory.
- It decides actions based on context. ✅ (ver Lesson 1: REASON — selects next step, el loop decide dinámicamente según el contexto/observación)
- It stores prompts in a database.

**3. Which defense layer focuses on restricting dangerous tool behavior?**
- Output filtering
- Tool boundaries ✅ (ver Lesson 5 — Defense in Depth: Least-privilege, input validation, sandboxing, human-in-the-loop)
- Input validation
- Observability logging

**4. What is the purpose of the "Observe" step in the agent loop?**
- Receive results from previous actions. ✅ (ver Lesson 1 — Agent Execution Loop: OBSERVE — Receive tool result or feedback)
- Store long-term model parameters.
- Convert prompts into embeddings.
- Train the model on new examples.

**5. Which reasoning framework explores multiple possible solution paths simultaneously?**
- Chain-of-Thought
- ReAct reasoning
- Tree-of-Thoughts ✅ (ver Lesson 3 — ToT: explores multiple reasoning branches simultaneously, like a search tree)
- Sequential prompting

---

# LangChain for AI Agents

_Oracle University. Material de estudio: `floridioracle/AI-AGENT-CERTIFICACION` (repo con QR en la slide)._

## What is LangChain?

An open-source framework that connects Large Language Models with the outside world — tools, data, APIs, and each other.

- **Connect** — Link LLMs to external tools, APIs, & databases.
- **Orchestrate** — Chain multiple steps into smart workflows.
- **Build** — Create agents, chatbots, & RAG applications.

## Models — Reasoning Engine

LangChain provides a unified interface — but model capabilities may still differ across providers.

```python
# Install: pip install langchain-openai langchain-anthropic

from langchain.chat_models import init_chat_model

# Initialize model
model = init_chat_model("openai:gpt-4o")

# Switch provider (example)
model = init_chat_model("anthropic:claude-3.5")
```

- **OpenAI** — GPT-5.5, GPT-4o
- **Anthropic** — Claude 3.7 / 3.5 / 4
- **Google** — Gemini 2.5
- **Open Source** — Llama, Mistral

## Prompt Template — Steering the Model

Prompts are reusable templates with placeholders. They let you build dynamic instructions for the LLM.

```python
from langchain_core.prompts import PromptTemplate

template = PromptTemplate(
    input_variables=["topic"],
    template="Explain {topic} to a beginner"
)

# Fill in the placeholder
prompt = template.format(topic="LangChain")

# Result: "Explain LangChain to a beginner"
```

- **PromptTemplate** — For simple text input/output tasks with variable placeholders.
- **ChatPromptTemplate** — For chat models with System, Human, and AI message roles.
- **Few-Shot Prompts** — Include examples in the prompt to improve output quality.

## Chains — Connecting the Pieces

The "Chain" in LangChain. Pipe data through a sequence of steps.

`Prompt Template → LLM Model → Output Parser → Final Result`

**LangChain Expression Language (LCEL)**

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_template("Explain {topic}")

# The pipe "|" operator chains steps together
chain = prompt | model | StrOutputParser()

# Run it!
result = chain.invoke({"topic": "AI agents"})
```

> Nota: `StrOutputParser()` toma la respuesta del modelo y extrae su texto plano.

## Tools — Giving LLMs Superpowers

Tools are functions that an LLM can call during its reasoning process. They extend the model beyond text generation into real-world actions.

- **Web Search** — Look up real-time information online.
- **Database Query** — Read/write data in SQL or NoSQL.
- **Code Execution** — Run Python code for calculations.
- **Custom APIs** — Call any external service or endpoint.

> In LangChain, tools are defined as Python functions with a name, description, and type hints. The LLM reads the tool description to decide when and how to use it. You can create your own custom ones using the `@tool` decorator.

## Anatomy of an Agent Tool

```python
@tool
def add(a: float, b: float) -> float:
    """Add two numbers together. Use for addition operations."""
    return a + b
```

- **`@tool` Decorator** — registers this function as a tool the agent can discover and call.
- **`a: float` Type hints** — tells the LLM what data types to pass (auto-generates JSON schema).
- **`-> float` Return type** — tells the agent what kind of data the tool sends back.
- **`"""..."""` Docstring** — the LLM reads this to decide **when** to use this tool. Clear description = accurate tool selection.

> What the LLM sees: The framework converts your decorated function into a JSON schema — `{ "name": "add", "description": "Add two numbers...", "parameters": { "a": { "type": "number" }, "b": { "type": "number" } } }`

## The ReAct Pattern

`REASON (Think about what to do next) → ACT (Call a tool or respond to user) → OBSERVE (Examine Tool's results) → (loop)`

Loop continues until goal achieved (or max iterations reached).

**Example Walkthrough**
```
User: "What's the weather in Seattle?"
Agent Reasons: I need current weather data. I'll use the weather tool.
Agent Acts: Call search_weather with city='Seattle'
Agent Observes: Got result: '55°F, cloudy'
Reason (Again): I have the answer. Return to user.
```

## Ejemplo completo: crear un Agent con LangChain

```python
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent
from langchain_core.tools import tool

# 1. Initialize the model
model = init_chat_model("openai:gpt-4o")

# 2. Define your tools
@tool
def multiply(a: float, b: float) -> float:
    """Multiply two numbers together. Use for multiplication operations."""
    return a * b

@tool
def divide(a: float, b: float) -> float:
    """Divide the first number by the second. Returns error if dividing by zero."""
    if b == 0:
        return "Error: Cannot divide by zero"
    return a / b

# 3. Create the LangChain Agent — that's it!
agent = create_agent(model, [multiply, divide])

# 4. Run it
result = agent.invoke({"messages": [
    ("user", "What is 15 multiplied by 8, then divided by 3?")
]})
```

## Lesson 3 — LangChain Agent under the hood

_A step-by-step walkthrough of what happens when you call `agent.invoke()`._

### 1. User runs the Python code

Your Python code triggers the agent:
```python
run_agent("What is 15 multiplied by 8, then divided by 3?")
```

Inside `run_agent` this happens:
```python
result = agent.invoke({
    "messages": [("user", question)]
})
```

`Your Python Code → LangChain Agent → Model API (LLM)`

**Chain of Control: User talks to your code**
- Your Python code calls LangChain.
- LangChain prepares the Prompt and Tool definitions.
- LangChain sends a request to the Model API.
- User never directly communicates with the LLM.

### 2. LangChain Agent sends the request to the Model API _(estimado, sin foto)_

_No se capturó la foto de este paso. Por la secuencia del diagrama (`Your Python Code → LangChain Agent → Model API (LLM)`) y lo adelantado en el paso 1 ("LangChain prepares the Prompt and Tool definitions" / "sends a request to the Model API"), probablemente este paso mostraba a LangChain armando el prompt + definiciones de tools y enviándolos al LLM. No confirmado — completar si aparece la slide real._

### 3. Model Returns a Tool Call #1

The model sees the question and decides:
- First, I need to multiply 15 by 8.
- Then I need to divide that result by 3.
- But the model cannot execute Python itself. So, it returns a Tool Call request.
- Conceptually, the first response might look like this:

```json
{
  "role": "assistant",
  "content": "",
  "tool_calls": [
    {
      "id": "call_001",
      "type": "function",
      "function": {
        "name": "multiply",
        "arguments": "{\"a\": 15, \"b\": 8}"
      }
    }
  ]
}
```

- **content: ""** — No text answer yet, model wants an action.
- **tool_calls** — Structured request for a function call.
- **name: "multiply"** — Which tool to invoke.
- **Arguments: {"a": 15, "b": 8}** — parsed by your app.

### 4. LangChain interprets the response

LangChain reads that response and notices:
- LLM did not give a final natural-language answer yet. Instead, it asked for a tool call.
- **Is there a text answer?** No → content is empty.
- **Are there tool_calls?** Yes → tool call detected!

LangChain extracts three pieces of info. Conceptually, LangChain does something like this:
```python
tool_name = "multiply"
tool_args = {"a": 15, "b": 8}
id        = "call_001"
```

> LangChain needs to figure out which real Python function corresponds to the tool name "multiply".

### 5. LangChain executes the Python Function (Multiply)

Real code runs on your machine — this is where reasoning becomes action.

```python
@tool
def multiply(a: float, b: float) -> float:
    return a * b

# LangChain calls it
Result = multiply(a=15, b=8)
#result = 120
```

`▲ Model Reasoning` / `▼ Code Execution`

**In Real Systems, Tools Can Do Much More:**
- Query a database · Call an external API
- Send an email · Create a support ticket
- Read/write files · Provision cloud resources

```python
def lookup_order(order_id):
    return database.query(...)
```

### 6-8. _(pasos intermedios — no se capturaron fotos)_

_Por la numeración y lo que se ve en el paso 9, estos pasos probablemente repiten el ciclo ReAct para la segunda operación: LangChain returns Tool #1 result → Model Returns a Tool Call #2 (`divide`) → LangChain executes the Python Function (Divide). No confirmado — completar si aparecen las slides reales._

### 9. LangChain returns Tool #2 result

Mensajes acumulados en la conversación (`messages`):

```json
{
  "messages": [
    {"role": "user", "content": "What is 15 multiplied by 8, then divided by 3?"},
    {"role": "assistant", "content": "", "tool_calls": [
      {"id": "call_001", "type": "function", "function": {"name": "multiply", "arguments": "{\"a\": 15, \"b\": 8}"}}
    ]},
    {"role": "tool", "tool_call_id": "call_001", "content": "120"},
    {"role": "assistant", "content": "", "tool_calls": [
      {"id": "call_002", "type": "function", "function": {"name": "divide", "arguments": "{\"a\": 120, \"b\": 3}"}}
    ]},
    {"role": "tool", "tool_call_id": "call_002", "content": "40.0"}
  ],
  "tools": ["..."]
}
```

### 10. Model generates final answer

No more tools needed — the model produces a natural-language response.

```json
{
  "role": "assistant",
  "content": "15 multiplied by 8 is 120, and 120 divided by 3 is 40."
}
```

**Exit Signal!**
- No `tool_calls` present this time.
- Content has actual text.
- LangChain sees this and knows the loop is done — time to return the result.

**The Three Assistant (LLM) Responses:**
1. Tool call → `multiply(15, 8)`
2. Tool call → `divide(120, 3)`
3. Final answer → "15 multiplied by 8 is 120..."

### 11. Python prints the result

LangChain returns the result object, and your code prints the final answer.

```python
result = agent.invoke({
    "messages": [("user", question)]
})

print("🤖 Agent:", result)
```

**Terminal Output (with trace enabled):**
```
User: What is 15 multiplied by 8, then divided by 3?
--------------------------------------------------
Agent thinks → calling: multiply({'a': 15, 'b': 8})
  Tool result: 120
Agent thinks → calling: divide({'a': 120, 'b': 3})
  Tool result: 40.0
Agent answer: 15 multiplied by 8 is 120, and dividing by 3 gives 40.
====================================================
```

### What LangChain hides from you

`agent.invoke()` is one line — but there is a LOT of machinery behind it.

```python
agent = create_agent(model=model, tools=tools,)

result = agent.invoke({"messages": [("user", question)] })
```

**Why This Matters**

Frameworks help beginners by hiding machinery. Understanding what is underneath is essential for:
- Debugging agent failures.
- Building custom agent loops.
- Optimizing performance.
- Moving to production.

**What LangChain Does For You**
- Building tool schemas from `@tool` decorators.
- Formatting JSON messages for the model API.
- Parsing tool call responses from JSON.
- Looking up tools in the internal registry.
- Executing Python functions with parsed args.
- Appending tool results to conversation.
- Deciding if another LLM call is needed.
- Assembling the final result object.

## Preguntas de práctica / quiz — LangChain for AI Agents

**1. What information does LangChain send to the model API during the first agent request?**
- The user message and the available tool definitions ✅ (ver Lesson 3, paso 2: LangChain prepara prompt + tool definitions y las envía al modelo)
- The final answer and the complete execution trace
- The Python source files and environment variables
- The vector database contents and cached embeddings

**2. Why does the model return a tool call instead of executing Python code directly?**
- The model can only reason and request actions. ✅ (ver Lesson 3, paso 3: el modelo no puede ejecutar Python, devuelve un Tool Call request)
- The model runs inside a restricted Python sandbox.
- The model requires manual approval before execution.
- The model can execute only built-in LangChain tools.

**3. Why is understanding LangChain internals important for developers?**
- It allows models to train themselves during execution.
- It helps with debugging and production optimization. ✅ (ver "What LangChain hides from you" — Why This Matters: debugging, custom agent loops, optimizing performance, moving to production)
- It removes the need for tool descriptions and schemas.
- It guarantees that every agent produces correct answers.

**4. What is the main purpose of LCEL in LangChain?**
- It connects multiple chain components using the pipe operator. ✅ (ver "Chains — Connecting the Pieces": el operador `|` encadena prompt | model | parser)
- It stores vector embeddings for retrieval applications.
- It converts tools into external REST API services.
- It encrypts prompts before sending them to the model.

**5. Why does LangChain provide a unified model interface?**
- It guarantees identical behavior across all LLM providers.
- It allows developers to switch providers with minimal code changes. ✅ (ver "Models — Reasoning Engine": `init_chat_model("openai:gpt-4o")` → cambiar a `init_chat_model("anthropic:claude-3.5")` con una línea)
- It automatically fine-tunes models for specific business tasks.
- It removes the need for provider-specific API credentials.

---

# Introduction to MCP

_Oracle University. Model Context Protocol._

## M3: Introduction to MCP — Agenda del módulo

- What is Model Context Protocol (MCP)?
- Core components — Tools, Resources, Prompts
- Add MCP Server to your first Agent
- Real-world MCP Walkthrough

## What is MCP?

An open standard that provides a universal interface for AI applications to connect with external tools, data sources, and systems — securely and consistently.

- **Launched** — November 2024 by Anthropic
- **Supported across the ecosystem** — OpenAI, Oracle, Microsoft, AWS
- **Foundation** — JSON-RPC 2.0, Open source
- **Governed By** — Agentic AI Foundation (Linux Foundation)

> Think of MCP as USB-C for AI — any MCP-compatible client can talk to any MCP-compatible server, regardless of who built them.

## The Problem MCP Solves

**BEFORE MCP: N × M Problem**
`Claude, ChatGPT, Gemini` × `GitHub, Slack, Drive, Database` — every AI app needs a custom connector for every tool = 12 integrations (3×4).

**AFTER MCP: N + M Solution**
`Claude, ChatGPT, Gemini → MCP → GitHub, Slack, Drive, Database` — each side builds ONE MCP integration = only 7 total integrations (3+4).

## MCP Architecture

**MCP HOST** (Claude Desktop, VS Code, or any MCP-enabled AI application)
- Contains: `LLM (AI Model)` + `MCP Client 1 / 2 / 3`
- Cada client se conecta a un server distinto: `MCP Server File System`, `MCP Server GitHub API`, `MCP Server Slack`

**Key Points**
- Host manages the AI app and creates clients.
- Each client connects to exactly one server.
- Servers expose tools, data, and prompts.
- Communication uses JSON-RPC 2.0.
- Servers can run locally or remotely.

### Different Boundaries — Math vs Oracle Usage MCP server

- **MCP Client Code (we wrote)** `first_agent_mcp.py` → **MCP Math Server (we wrote)** `mcp_math_server.py` → **Local Python tools (we wrote)** `Add(), subtract(), multiply(py)` — vía stdio, JSON-RPC 2.0, Python function call.
- **MCP Client code (we wrote)** `mcp_usage_mcp_client.py` → **Oracle Usage MCP server (Oracle wrote)** `we made an usage mcp-server` → **OCI Usage REST API (Oracle hosts)** `Report/Cummarized/usage` — vía HTTPS, OCI SDK.

> Same: Client code shape, JSON-RPC protocol, Stdio transport, list_tools/call_tool.
> Different: who owns the server and what the server ultimately wraps.

## Core Primitives

| Primitive | Control | Qué hace | Ejemplos | Analogía |
|---|---|---|---|---|
| **Tools** | Model-Controlled | Functions the AI can call to perform actions ("Do something") | `create_issue()`, `send_message()`, `query_database()`, `run_test()` | Como POST endpoints — ejecutan código y producen side effects |
| **Resources** | Application-Controlled | Structured data the AI can read for context ("Read Something") | `file://project/readme.md`, `db://users/schema`, `api://config/settings` | Como GET endpoints — cargan info en el context window del LLM |
| **Prompts** | User-Controlled | Templates that structure LLM interactions ("Structure something") | `bug_report_template`, `code_review_prompt`, `summarize_doc_template` | Como slash commands — los usuarios los invocan a través de elementos de UI |

## The MCP Connection Lifecycle

1. **Initialize** — Client sends protocol version & capabilities to server.
2. **Discover** — Client requests list of tools, resources, prompts.
3. **Operate** — LLM decides tools to call, client executes those calls.
4. **Shutdown** — Client closes transport and ends session.
