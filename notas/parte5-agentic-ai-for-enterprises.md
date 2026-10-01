# Parte 5 — Agentic AI for Enterprises

_Oracle University. Build, Deploy & Govern Production-Grade AI Agents on Oracle Cloud Infrastructure._

## Lesson 1 — Agent Stack & Runtime Architecture

### The question every beginner hits

You wrote your first agent. It works on your laptop. Now what?

```python
from langchain import create_agent
from openai import OpenAI

agent = create_agent(
    model="gpt-4.1",
    tools=[search, calculator]
)

result = agent.invoke("What is 15 x 8 / 3?")
print(result)

# ✓ Works on your laptop.
# ✗ Now what?
```

**You think you're done — until reality arrives:**
- Where does it run when your laptop is closed?
- What happens when 1,000 users hit it at once?
- How do users call your agent?
- Where do conversation histories live?
- What if a tool call fails halfway through?
- How do you see what the agent did yesterday?

### The car analogy

Why designing the engine is not the same as driving down the road.

**THE ENGINE — Agent logic** _(how the brain thinks)_
- Reasoning and decision making
- Tool calling logic
- Prompt orchestration
- Framework: LangChain, OpenAI Agents SDK

**EVERYTHING ELSE — Runtime architecture** _(the body that lets the brain function)_
- Fuel system → execution environment
- Cooling system → scaling & reliability
- Roads → request handling & APIs
- Traffic control → observability & logs

### Agent Stack

Every agent system, from demos to enterprise, has these three layers:

1. **User / App** — UI · API call · chatbot interface — how users reach the agent.
2. **Runtime Architecture** — What you must build yourself — or get from a managed service: Request Handling, Execution Environment, Orchestration & State, Tool Sandbox, Integrations, Scaling & Reliability, Observability.
3. **Agent Logic** — LangChain · OpenAI Agents SDK/Responses API — reasoning, tool calling.

### Inside the runtime — things you didn't write

Each box is real code, real infrastructure; none of it comes from your framework.

1. **Request Handling** — Authentication, API Gateway Routing — converting user input into an agent invocation.
2. **Execution Environment** — Server, container, or serverless — the machine that runs your agent.
3. **Orchestration & State** — Memory, Multi-step workflows, Retries when a tool call fails.
4. **Tool Sandbox** — Isolation and resource limits when the agent runs code, browses, or hits APIs.
5. **Integration layer** — Connectors and credentials for databases, APIs, and enterprise systems.
6. **Scaling & reliability** — Auto-scaling, Load Balancing, Failover when traffic spikes or a component dies.
7. **Observability** — Logs, Traces, Metrics — so you can see step-by-step what the agent did.

### Two paths to production

Same agent code. Very different amount of work to ship it.

**Path A — Do it yourself** _(Pure LangChain / OpenAI / any framework)_
- ❌ Provision servers, containers, or serverless.
- ❌ Build request handling, auth, API gateway.
- ❌ Implement state storage & retries yourself.
- ❌ Secure tool execution & credentials.
- ❌ Set up scaling, load balancing, failover.
- ❌ Instrument logs, traces, metrics from scratch.

**Path B — Managed agent runtime** _(OCI Enterprise AI Agents service)_
- ✅ Execution environment — fully hosted.
- ✅ Request handling — built-in API endpoints.
- ✅ State & sessions — managed automatically.
- ✅ Tool execution — sandboxed by default.
- ✅ Scaling & reliability — handled by OCI.
- ✅ Logs & traces — OCI Logging, Monitoring, Audit.

### Key Takeaway

> Agent frameworks give you intelligence. Runtime architecture lets that intelligence run in the real world.

- **Agent logic** — What you wrote with LangChain, OpenAI, or the Responses API.
- **Runtime architecture** — The multi-layer system that keeps agent logic alive in production.
- **OCI Enterprise AI Agents** — OCI Enterprise AI Agents service gives you a managed runtime.

## Lesson 2 — OCI Enterprise AI Models

### Chat, Embed, Rerank

| Chat Models (Conversational AI) | Embed Models (Semantic representation) | Rerank Models (Relevance Scoring) |
|---|---|---|
| Most versatile model type — handles question answering, summarization, content generation, code, reasoning, and agentic workflows. | Convert text or images into dense numerical vectors that capture semantic meaning. | Take a query and a list of candidate documents; return them sorted by a relevance score from 0 to 1. |
| **Key traits:** stateless by default, temperature-controlled creativity, tool-calling support for agentic use. | **Key traits:** vectors are fixed-dimension (e.g., 1024-dim), compared via cosine similarity or other measures. | **Key traits:** more accurate than vector similarity alone. |
| **Providers:** Cohere Command A · Meta Llama 4 · xAI Grok 4 · Google Gemini 2.5 · OpenAI gpt-oss | **Providers:** Cohere Embed 4 · Embed English v3 · Embed Multilingual v3 (100+ languages) | **Provider:** Cohere Rerank 3.5 |

### Serving Mode

| On-Demand | Dedicated Cluster |
|---|---|
| Shared GPU infrastructure across tenants. | GPUs physically dedicated to your tenancy only — full isolation. |
| Pay per inference call — zero upfront commitment. | Commit to cluster hours — billed regardless of call volume. |
| Dynamic throttling — implement exponential back-off in your code. | Predictable latency — suitable for production SLA commitments. |
| Retirement ends on-demand access immediately. | REQUIRED for fine-tuned and imported custom models. |
| NOT available for fine-tuned or imported custom models. | Up to 50 endpoints served from a single hosting cluster. |
| Start in minutes — no provisioning required. | Retired model clusters continue running — no production disruption. |
| Best for experiments, PoCs, and variable-load workloads. | Break-even with on-demand at sustained high call volume. |

**Decision Guide:**
- **On-Demand** — exploring a new model, building a proof-of-concept, or running variable workloads where sharing GPU is acceptable.
- **Dedicated** — need production SLAs, predictable latency, GPU isolation for compliance, or deploying a custom/imported model.

## Lesson 3 — OCI Enterprise AI Agents

_End-to-end Lifecycle + Runtime, Simplified._

### OCI Enterprise AI Agents — las tres capas

- **Agent Management** — Agents are built, deployed, and observed. Authoring · Observability · Testing & evaluation · Hosted deployment.
- **Agent Orchestration** — Agents plan how to execute the task given. Human-in-the-loop · Protocols (MCP, A2A) · Open-source Frameworks.
- **Agent Runtime** — How the agents get things done. Responses API · Tools · Vector Stores · Memory.

### OCI Enterprise AI Governance

| Network Security | Identity Control | AI Behavior |
|---|---|---|
| Private Endpoints keep model access inside a secure network boundary. | IAM Policies define who can access, use, or manage AI resources. | Guardrails apply runtime controls on model inputs and outputs. |
| ZPR enforces identity-based service communication. | Fine-grained control at individual resource-type level. | Content moderation detects harmful or policy-violating content. |
| Prevents public internet exposure of AI model endpoints. | Supports compartment-level and tenancy-level policies. | Prompt injection defense prevents malicious instruction override. |
| Applies to dedicated AI cluster endpoints and private API access. | Least-privilege access recommended for all non-admin groups. | PII detection masks or blocks sensitive personal data in responses. |

### Two approaches to building with OCI Enterprise AI Agents

**Build Agents with OCI Responses API** _(Via Managed APIs)_
- Use OCI Responses API to build and run agentic workloads directly from your application — no infrastructure to deploy.
- Your application calls the API; OCI handles the orchestration loop.

**Deploy Hosted Agentic Apps in OCI** _(Via Hosted Applications)_
- Package your agent (built with any framework) as a container image, upload to OCI Registry, and deploy it as a managed application.
- OCI provisions an HTTP endpoint, handles auto-scaling, networking, storage, and auth — your agent becomes a production service.

### OCI Enterprise AI Agents — Lifecycle + Runtime, delivered

`Your User / App → Managed by OCI (Agent Runtime/Orchestration/Management) → Your Agent Logic`

**Managed by OCI:** Hosted endpoints · Auto scaling · Memory & sessions · Sandboxed tools · Logging & Audit (OCI) · Integrations

**Your Agent Logic — this is all you write.**

**What you focus on:**
- The agent's instructions & persona.
- Which tools it can call.
- Which knowledge bases to ground on.
- The business outcome it drives.

**What OCI handles for you:**
- Hosted endpoints — no servers to manage.
- Scaling for hosted apps — handled by OCI.
- Memory & sessions — built in (Conversations API).
- Sandboxed tools (Code Interpreter, Containers).
- Observability via OCI Logging, Monitoring, Audit.
- Integrations with OCI data & services.

## Lesson 4 — OCI Enterprise Responses API

### The OCI Responses API

- **OpenAI Compatible APIs** — Use familiar OpenAI SDKs and workflows with minimal code changes.
- **Built-in Tool Reasoning** — Model can reason across multiple tool calls during response generation.
- **Multi-Model Routing** — Route requests to OpenAI, Grok, Gemini, Llama, or Cohere models.
- **State Management** — Supports conversation history, memory, and context compaction.

### Agent Tools — Your Agent's Superpowers

- **File Search** — Semantic retrieval from uploaded documents.
- **Code Interpreter** — Execute Python in a sandboxed environment.
- **Function Calling** — Invoke your custom business functions.
- **MCP Calling** — Connect to remote MCP servers.

### Agent Memory — Remembering Context

- **Short-Term Memory** — Responses API + Conversations API simplify conversation state across multiple turns. _Analogy: Notepad at a meeting._
- **Long-Term Memory** — Durable memory across conversations via a unique `subject_id` within a project. _Analogy: Your personal journal._
- **Context Optimization** — Condenses chat history into retained memory — reduces latency and token usage in long conversations. _Analogy: Summarizing a 3-hr meeting._

## Lesson 5 — Agent Lower-Level Building Blocks

- **Vector Stores API** — Manage vector databases for semantic search. Upload documents, automatic chunking and embedding, semantic retrieval with metadata filtering.
- **Files API** — Upload and manage files. Supports various formats. Files can be referenced by agents for processing, analysis, or retrieval.
- **Containers API** — Run custom containers in secure sandboxes. Use for specialized processing, custom tools, or isolated execution environments.

### Setting Up Authentication

**Method 1: API Keys (Beginner)**
```python
from openai import OpenAI
import os

client = OpenAI(
    api_key=os.getenv("OCI_API_KEY"),
    base_url=
      "https://inference.generativeai.us-chicago-1.oci.oraclecloud.com/20231130/actions/v1"
)
```

**Method 2: OCI IAM (Production)**
```python
import httpx
from openai import OpenAI
from oci_genai_auth import \
  OciSessionAuth

client = OpenAI(
    base_url="https://inference.generativeai...com/openai/v1",
    api_key="not-used",
    http_client=httpx.Client(
        auth=OciSessionAuth()))
```

> **IAM Policy:** `allow any-user to use generative-ai-family in compartment <your-compartment> where ALL { request.principal.type='generativeaiapikey' }`

### Your First Agent — Simple Chat

```python
from openai import OpenAI
import os

OCI_REGION = "us-chicago-1"  # <-- CHANGE THIS to your region
OCI_BASE_URL = (
    f"https://inference.generativeai.{OCI_REGION}"
    f".oci.oraclecloud.com/openai/v1"
)
MODEL = "openai.gpt-oss-120b"
OCI_PROJECT_ID = "ocid1.generativeaiproject.oc1.us-chicago-1.xxxxxxxx"

# --- Create Client pointing to OCI
client = OpenAI(
    api_key=os.getenv("OCI_GENAI_API_Key"),
    base_url=OCI_BASE_URL,
    default_headers={
        "OpenAI-Project": OCI_PROJECT_ID,
    },
)

# Call the Responses API
response = client.responses.create(
    model=MODEL,
    input="Explain what an AI agent is in one paragraph.",
)

print(f"Response:\n{response.output_text}\n")
```

- **1.** `OCI_REGION` / `OCI_BASE_URL` / `MODEL` / `OCI_PROJECT_ID` — config apuntando a la región e infraestructura de OCI.
- **2.** `client = OpenAI(...)` — crea el client apuntando a OCI (mismo SDK de OpenAI, distinto `base_url` + headers).
- **3.** `client.responses.create(...)` — misma Responses API vista antes, ahora corriendo sobre OCI.

### Native OpenAI vs OCI OpenAI-compatible Responses API

**Native OpenAI Responses API:**
```python
from openai import OpenAI

# Create a client — reads OPENAI_API_KEY from your env automatically
client = OpenAI()

# The simplest Responses API call
response = client.responses.create(
    model="gpt-4.1",
    input="Explain what an AI agent is in one paragraph.",  # Just a string!
)

# .output_text is a handy shortcut to get the text response
print(response.output_text)
```
> Set `OCI_GENAI_API_Key=sk-xx`

**OCI OpenAI compatible Responses API:**
```python
from openai import OpenAI
import os

OCI_REGION = "us-chicago-1"  # <-- CHANGE to your region
OCI_BASE_URL = (
    f"https://inference.generativeai.{OCI_REGION}"
    f".oci.oraclecloud.com/openai/v1"
)
MODEL = "openai.gpt-oss-120b"
OCI_PROJECT_ID = "ocid1.generativeaiproject.oc1.us-chicago-1.xxxx"

# --- Create Client pointing to OCI
client = OpenAI(
    api_key=os.getenv("OCI_GENAI_API_Key"),
    base_url=OCI_BASE_URL,
    default_headers={
        "OpenAI-Project": OCI_PROJECT_ID,
    },
)

# Call the Responses API
response = client.responses.create(
    model=MODEL,
    input="Explain what an AI agent is in one paragraph.",
)
```

> Misma forma de código en ambos lados — lo único que cambia es `client = OpenAI()` (default, apunta a OpenAI) vs. `client = OpenAI(base_url=OCI_BASE_URL, default_headers={...})` (apunta a OCI). El resto del código (`.responses.create(...)`, `.output_text`) es idéntico.

## Preguntas de práctica / quiz — Agentic AI for Enterprises

**1. What is the purpose of the Containers API in OCI Enterprise AI Agents?**
- Running custom workloads inside isolated execution environments ✅ (ver Lesson 5 — Agent Lower-Level Building Blocks: "Containers API — Run custom containers in secure sandboxes")
- Managing OCI IAM policies for hosted AI applications
- Routing requests between multiple foundation model providers
- Converting uploaded documents into semantic embeddings

**2. What is the benefit of using sandboxed tool execution in OCI Enterprise AI Agents?**
- It increases model training throughput across clusters
- It isolates tool execution from the host environment. ✅ (ver Lesson 3 — "What OCI handles for you": Sandboxed tools (Code Interpreter, Containers))
- It converts OCI APIs into OpenAI-compatible endpoints.
- It automatically fine-tunes models using runtime logs.

**3. In OCI Enterprise AI Agents, what is the purpose of the Conversations API?**
- It provisions GPU clusters for inference workloads.
- It manages multi-turn conversational session history. ✅ (ver Lesson 4 — Agent Memory: "Short-Term Memory — Responses API + Conversations API simplify conversation state across multiple turns")
- It deploys hosted containerized agent applications.
- It performs semantic ranking of uploaded documents.

**4. Why might a team choose Dedicated AI Clusters instead of On-Demand inference?**
- Dedicated AI Clusters eliminate the need for authentication policies
- Dedicated AI Clusters provide isolated GPU capacity and predictable latency ✅ (ver Lesson 2 — Serving Mode: Dedicated Cluster → "GPUs physically dedicated to your tenancy only — full isolation" + "Predictable latency — suitable for production SLA commitments")
- Dedicated AI Clusters only support embedding and reranking models.
- Dedicated AI Clusters automatically generate orchestration workflows.

**5. Why is reranking used after vector similarity search in a RAG workflow?**
- It reduces GPU memory usage during embedding generation.
- It improves relevance scoring among retrieved documents. ✅ (ver Lesson 2 — OCI Enterprise AI Models: Rerank Models — "more accurate than vector similarity alone")
- It converts prompts into vector database records.
- It stores conversation history across agent sessions.
