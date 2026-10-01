# 🚀 Resumen Repaso — Parte 5: Agentic AI for Enterprises

_Términos técnicos en inglés, explicaciones en español. Hecho para repasar rápido antes del examen._

---

## 🏗️ Agent Stack & Runtime Architecture

> 💡 Escribiste tu primer agent, funciona en tu laptop... ¿y ahora? ¿Dónde corre cuando cerrás la laptop? ¿Qué pasa con 1.000 usuarios a la vez? ¿Dónde vive el historial de conversación? ¿Qué pasa si falla una tool call?

**🚗 La analogía del auto:**
- **THE ENGINE — Agent logic** (cómo piensa el cerebro): reasoning, tool calling logic, prompt orchestration. Framework: LangChain, OpenAI Agents SDK.
- **EVERYTHING ELSE — Runtime architecture** (el cuerpo que hace funcionar al cerebro): fuel system → execution environment, cooling → scaling & reliability, roads → request handling & APIs, traffic control → observability & logs.

**🧱 Agent Stack (3 capas, de demo a enterprise):**
1. **User / App** — cómo los usuarios llegan al agent (UI, API call, chatbot).
2. **Runtime Architecture** — lo que hay que construir vos (o contratar como managed service).
3. **Agent Logic** — LangChain / OpenAI Agents SDK/Responses API — el razonamiento.

**🔧 Inside the runtime — cosas que no escribiste (7 piezas):**
Request Handling (auth, API gateway) · Execution Environment (server/container/serverless) · Orchestration & State (memoria, retries) · Tool Sandbox (aislamiento) · Integration layer (conectores) · Scaling & reliability (auto-scaling, failover) · Observability (logs, traces, metrics).

**🔀 Dos caminos a producción:**
| Path A — Hacerlo vos mismo | Path B — Managed runtime (OCI Enterprise AI Agents) |
|---|---|
| ❌ Provisionar servers, auth, state storage, scaling, logs — todo desde cero | ✅ Execution environment, request handling, state & sessions, tool sandboxing, scaling, logs — todo manejado por OCI |

> 🎯 **Key Takeaway:** Agent frameworks te dan la inteligencia. La runtime architecture es lo que permite que esa inteligencia corra en el mundo real.

---

## 🤖 OCI Enterprise AI Models

**Chat · Embed · Rerank:**
- **Chat Models** — el más versátil: QA, summarization, content generation, code, reasoning, agentic workflows. Stateless por default. Providers: Cohere Command A, Meta Llama 4, xAI Grok 4, Google Gemini 2.5, OpenAI gpt-oss.
- **Embed Models** — convierten texto/imágenes en vectores numéricos (semantic representation). Vectores de dimensión fija, comparados por cosine similarity. Providers: Cohere Embed 4, Embed English/Multilingual v3.
- **Rerank Models** — toman una query + documentos candidatos y los devuelven ordenados por relevancia (0 a 1). **Más preciso que la similitud vectorial sola.** Provider: Cohere Rerank 3.5.

**⚙️ Serving Mode: On-Demand vs Dedicated Cluster**

| On-Demand | Dedicated Cluster |
|---|---|
| GPU compartida entre tenants, pago por llamada, sin compromiso inicial | GPU dedicada a tu tenancy — aislamiento total, latencia predecible |
| NO disponible para modelos fine-tuned o importados | REQUERIDO para modelos fine-tuned/importados |
| Ideal para experimentos, PoCs, carga variable | Ideal para SLAs de producción, compliance, alto volumen sostenido |

---

## ☁️ OCI Enterprise AI Agents

_End-to-end Lifecycle + Runtime, Simplified._

**🧩 Las tres capas:**
- **Agent Management** — Agents se construyen, despliegan y observan: Authoring, Observability, Testing & evaluation, Hosted deployment.
- **Agent Orchestration** — Los agents planean cómo ejecutar la tarea: Human-in-the-loop, Protocols (MCP, A2A), Open-source Frameworks.
- **Agent Runtime** — Cómo los agents hacen las cosas: Responses API, Tools, Vector Stores, Memory.

**🔒 OCI Enterprise AI Governance:**
- **Network Security** — Private Endpoints, ZPR (identity-based comms), sin exposición pública de endpoints.
- **Identity Control** — IAM Policies granulares, least-privilege por default.
- **AI Behavior** — Guardrails en runtime, content moderation, defensa contra prompt injection, detección de PII.

**🏗️ Dos formas de construir:**
- **Build Agents with OCI Responses API** (vía Managed APIs) — tu app llama la API, OCI maneja el orchestration loop. Sin infra que desplegar.
- **Deploy Hosted Agentic Apps in OCI** (vía Hosted Applications) — empaquetás tu agent (de cualquier framework) como container, lo subís a OCI Registry, y OCI lo convierte en un servicio de producción (endpoint, auto-scaling, auth incluidos).

> **Tu Agent Logic es todo lo que escribís vos:** instructions & persona, qué tools puede llamar, de qué knowledge bases se nutre, y el resultado de negocio que impulsa. **OCI se encarga de:** endpoints hosteados, scaling, memoria & sesiones (Conversations API), tools sandboxeadas, observability, e integraciones.

---

## 📡 OCI Enterprise Responses API

**The OCI Responses API:**
- **OpenAI Compatible APIs** — mismos SDKs/workflows de OpenAI, cambios mínimos de código.
- **Built-in Tool Reasoning** — el modelo razona a través de múltiples tool calls.
- **Multi-Model Routing** — rutea a OpenAI, Grok, Gemini, Llama, o Cohere.
- **State Management** — conversation history, memory, context compaction.

**🛠️ Agent Tools:** File Search (retrieval semántico) · Code Interpreter (Python sandboxeado) · Function Calling (tus funciones de negocio) · MCP Calling (conexión a MCP servers remotos).

**🧠 Agent Memory:**
- **Short-Term Memory** — Responses API + Conversations API. _Analogía: notepad en una reunión._
- **Long-Term Memory** — memoria durable entre conversaciones vía `subject_id` único por proyecto. _Analogía: tu diario personal._
- **Context Optimization** — condensa el historial para reducir latencia/tokens. _Analogía: resumir una reunión de 3hs._

### 🔑 Setting Up Authentication

```python
# Método 1: API Keys (Beginner)
client = OpenAI(api_key=os.getenv("OCI_API_KEY"), base_url="https://inference.generativeai...")

# Método 2: OCI IAM (Production) — auth=OciSessionAuth()
```

### 🆚 Native OpenAI vs OCI OpenAI-compatible Responses API

> Mismo código en ambos lados — lo único que cambia es `client = OpenAI()` (default) vs. `client = OpenAI(base_url=OCI_BASE_URL, default_headers={...})` (apunta a OCI). `.responses.create(...)` y `.output_text` son idénticos.

---

## 🧱 Agent Lower-Level Building Blocks

- **Vector Stores API** — bases de datos vectoriales para semantic search: upload de documentos, chunking y embedding automático.
- **Files API** — subir y gestionar archivos que los agents pueden referenciar.
- **Containers API** — correr containers custom en sandboxes seguros, para procesamiento especializado o entornos de ejecución aislados.

---

## ✅ Preguntas tipo examen (con respuesta marcada)

**1. What is the purpose of the Containers API in OCI Enterprise AI Agents?**
- **✅ Running custom workloads inside isolated execution environments**
- Managing OCI IAM policies for hosted AI applications
- Routing requests between multiple foundation model providers
- Converting uploaded documents into semantic embeddings

**2. What is the benefit of using sandboxed tool execution in OCI Enterprise AI Agents?**
- It increases model training throughput across clusters
- **✅ It isolates tool execution from the host environment.**
- It converts OCI APIs into OpenAI-compatible endpoints.
- It automatically fine-tunes models using runtime logs.

**3. In OCI Enterprise AI Agents, what is the purpose of the Conversations API?**
- It provisions GPU clusters for inference workloads.
- **✅ It manages multi-turn conversational session history.**
- It deploys hosted containerized agent applications.
- It performs semantic ranking of uploaded documents.

**4. Why might a team choose Dedicated AI Clusters instead of On-Demand inference?**
- Dedicated AI Clusters eliminate the need for authentication policies
- **✅ Dedicated AI Clusters provide isolated GPU capacity and predictable latency**
- Dedicated AI Clusters only support embedding and reranking models.
- Dedicated AI Clusters automatically generate orchestration workflows.

**5. Why is reranking used after vector similarity search in a RAG workflow?**
- It reduces GPU memory usage during embedding generation.
- **✅ It improves relevance scoring among retrieved documents.**
- It converts prompts into vector database records.
- It stores conversation history across agent sessions.

---

📖 Detalle completo con slides y código: `parte5-agentic-ai-for-enterprises.md`
