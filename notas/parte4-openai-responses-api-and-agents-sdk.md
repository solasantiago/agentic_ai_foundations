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
