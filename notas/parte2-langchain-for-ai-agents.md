# Parte 2 — LangChain for AI Agents

_Florencia Díaz · Oracle University. Material de estudio: `floridioracle/AI-AGENT-CERTIFICACION` (repo con QR en la slide)._

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
