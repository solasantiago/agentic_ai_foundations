# Parte 1 — Introduction to AI Agents

_Florencia Díaz · Oracle University._

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
