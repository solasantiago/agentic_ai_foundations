# 🚀 Resumen Repaso — Parte 1: Introduction to AI Agents & LangChain

_Términos técnicos en inglés, explicaciones en español. Hecho para repasar rápido antes del examen._

---

## 🧠 Introduction to AI Agents

### 🔺 Los tres componentes de todo Agent

| Componente | Rol | Qué hace |
|---|---|---|
| 🧠 **LLM** | El cerebro | Entiende la intención, decide qué Tool llamar y con qué argumentos, interpreta resultados, genera la respuesta final |
| 🛠️ **Tools** | Las manos | Conectan al Agent con el mundo exterior (APIs, DBs, código). Habilitan RAG |
| 🔄 **Loop (Orchestration)** | El sistema nervioso | Decide cuándo razonar y qué Tool usar. Gestiona memoria y estado |

> 💡 Un mismo LLM puede potenciar distintos Agents solo cambiando sus Tools — **no hace falta reentrenar**.

### 🔁 The Agent Execution Loop

```
PERCEIVE → REASON → ACT → OBSERVE → (repite)
```

**Termina cuando:** el Agent tiene la respuesta final 🏁 / se alcanza el máximo de iteraciones 🔢 / hay error o timeout ⏱️

**⚠️ Qué puede salir mal:** loops infinitos · tool calls alucinadas (Tools que no existen) · explosión de costos (demasiadas llamadas al LLM)

> 🔒 **Seguridad clave:** el LLM **NUNCA** ejecuta Tools directamente. Solo pide un tool call estructurado; la app lo valida y ejecuta. **MCP** estandariza cómo los LLMs acceden a Tools, prompts y recursos.

### 🧩 Patrones de razonamiento (Reasoning Frameworks)

| Framework | Idea | Mejor para |
|---|---|---|
| 🪜 **CoT** (Chain-of-Thought) | Razona paso a paso antes de responder. Solo usa conocimiento interno — **no puede actuar ni buscar afuera** | Matemática, lógica, análisis paso a paso |
| 🔂 **ReAct** (Reasoning + Acting) | CoT + uso de Tools en loop (Reason → Action → Observation). Reduce alucinaciones al fundamentar en datos reales | Selección de Tools, tareas multi-paso, llamadas a APIs |
| 🌳 **ToT** (Tree-of-Thoughts) | Explora **varios** caminos de razonamiento en paralelo, como un árbol, y evalúa cada rama | Tareas creativas, planificación estratégica, exploración |

> 💡 CoT no hace al modelo más inteligente — lo ayuda a usar su capacidad de razonamiento de forma más confiable. **ReAct = CoT + acción** → por eso ReAct es necesario para que un Agent realmente actúe.

### ⚙️ Agent Frameworks

| Framework | Para qué sirve |
|---|---|
| 🦜 **LangChain / LangGraph** | Ecosistema más grande, workflows con estado (grafos) → ideal producción |
| 🤖 **OpenAI Agents SDK** | Orquestar multi-agente, tracing y guardrails integrados |
| 👥 **CrewAI** | Equipos de Agents basados en roles |
| 🤗 **Hugging Face SmolAgents** | Mínimo (~1K líneas) → ideal para aprender |

> 🐍 **Python Essentials:** `@tool` (decorator que registra la función como Tool) · type hints `a: float` (le dicen al LLM qué tipo espera cada parámetro) · docstrings `"""..."""` (el LLM las lee para decidir cuándo usar la Tool) · return type `-> float` (indica qué devuelve).

### 🛡️ Safety and Guardrails

**⚠️ Threat Model — qué puede salir mal:**
- 💉 **Prompt Injection** — atacante secuestra al Agent con input/contenido envenenado. **La amenaza #1.**
- 🔧 **Tool Misuse** — el Agent llama Tools con argumentos erróneos o peligrosos.
- 🧪 **Memory Poisoning** — contenido envenenado en memoria afecta comportamiento futuro.
- 📤 **Data Exfiltration** — el Agent es engañado para filtrar datos sensibles.
- 💸 **Runaway Execution** — loops infinitos / exceso de llamadas → costos disparados.

**🧱 Defense in Depth (capas, ninguna alcanza sola):**
1. Input Validation — contenido externo = no confiable
2. LLM Guardrails — system prompts de seguridad, revisión humana si hay baja confianza
3. Tool Boundaries — mínimo privilegio, sandboxing
4. Output Filtering — filtrar PII, validar contenido
5. Observability — loguear todo: inputs, tool calls, outputs, errores, costos

---

## 🦜 LangChain for AI Agents

### ❓ Qué es LangChain

Framework open-source que conecta LLMs con el mundo exterior. 3 funciones: **Connect** (Tools/APIs/DBs) · **Orchestrate** (encadenar pasos) · **Build** (Agents, chatbots, RAG).

### 🧱 Models, Prompts y Chains

- **Models** — `init_chat_model("openai:gpt-4o")` → interfaz unificada, cambiar de proveedor (OpenAI/Anthropic/Google/open source) con mínimo cambio de código.
- **Prompt Templates** — `PromptTemplate` (texto simple) · `ChatPromptTemplate` (roles de chat) · Few-Shot Prompts (ejemplos en el prompt).
- **Chains (LCEL)** — el operador `|` encadena: `Prompt → Model → Output Parser → Resultado`. `StrOutputParser()` extrae el texto plano.

### 🛠️ Tools en LangChain

Funciones Python con `@tool`: Web Search · Database Query · Code Execution · Custom APIs. El LLM lee el **docstring** para decidir cuándo/cómo usarla.

**Anatomy of a Tool:**
```python
@tool
def add(a: float, b: float) -> float:
    """Add two numbers together."""
    return a + b
```
`@tool` registra → type hints generan el JSON schema → `-> float` indica qué devuelve → docstring guía la selección.

### 🏗️ Crear un Agent (ejemplo)

```python
agent = create_agent(model, [multiply, divide])
result = agent.invoke({"messages": [("user", "pregunta")]})
```

### 🔬 Qué pasa realmente al llamar `agent.invoke()`

| # | Paso |
|---|---|
| 1 | Tu código Python dispara el Agent. El usuario **nunca** habla directo con el LLM |
| 2 | *(estimado)* LangChain arma prompt + Tools y los envía al modelo |
| 3 | El modelo **no puede ejecutar Python** → devuelve un **Tool Call** (JSON con `tool_calls`, nombre, argumentos) |
| 4 | LangChain interpreta: ¿hay texto? No. ¿Hay `tool_calls`? Sí → extrae nombre, args, id |
| 5 | LangChain ejecuta la función Python real — acá el razonamiento se vuelve acción |
| 6-8 | *(sin confirmar)* probablemente se repite el ciclo para la 2ª operación |
| 9 | LangChain devuelve el resultado de la Tool #2, mensajes acumulados en `messages` |
| 10 | El modelo ya no pide más Tools → genera respuesta final en texto. **Señal de salida:** `tool_calls` vacío + `content` con texto |
| 11 | Python imprime el resultado de `agent.invoke()` |

> 🎩 **Qué esconde LangChain:** construye schemas desde los decorators · formatea JSON para la API · parsea tool calls · busca Tools en su registro · ejecuta funciones con args parseados · agrega resultados a la conversación · decide si hace falta otra llamada al LLM · arma el resultado final.
>
> Entenderlo sirve para: 🐛 debuggear · 🔧 construir loops custom · ⚡ optimizar performance · 🚀 ir a producción.

---

## ✅ Preguntas tipo examen (con respuesta marcada)

### Introduction to AI Agents

**1. Why are type hints important when defining agent tools in Python?**
- They improve GPU utilization speed.
- **✅ They help generate valid tool schemas.**
- They reduce the size of prompts.
- They automatically train the agent.

**2. Which statement describes how an AI agent differs from a fixed workflow?**
- It follows predefined steps only.
- It generates responses from memory.
- **✅ It decides actions based on context.**
- It stores prompts in a database.

**3. Which defense layer focuses on restricting dangerous tool behavior?**
- Output filtering
- **✅ Tool boundaries**
- Input validation
- Observability logging

**4. What is the purpose of the "Observe" step in the agent loop?**
- **✅ Receive results from previous actions.**
- Store long-term model parameters.
- Convert prompts into embeddings.
- Train the model on new examples.

**5. Which reasoning framework explores multiple possible solution paths simultaneously?**
- Chain-of-Thought
- ReAct reasoning
- **✅ Tree-of-Thoughts**
- Sequential prompting

### LangChain for AI Agents

**6. What information does LangChain send to the model API during the first agent request?**
- **✅ The user message and the available tool definitions**
- The final answer and the complete execution trace
- The Python source files and environment variables
- The vector database contents and cached embeddings

**7. Why does the model return a tool call instead of executing Python code directly?**
- **✅ The model can only reason and request actions.**
- The model runs inside a restricted Python sandbox.
- The model requires manual approval before execution.
- The model can execute only built-in LangChain tools.

**8. Why is understanding LangChain internals important for developers?**
- It allows models to train themselves during execution.
- **✅ It helps with debugging and production optimization.**
- It removes the need for tool descriptions and schemas.
- It guarantees that every agent produces correct answers.

**9. What is the main purpose of LCEL in LangChain?**
- **✅ It connects multiple chain components using the pipe operator.**
- It stores vector embeddings for retrieval applications.
- It converts tools into external REST API services.
- It encrypts prompts before sending them to the model.

**10. Why does LangChain provide a unified model interface?**
- It guarantees identical behavior across all LLM providers.
- **✅ It allows developers to switch providers with minimal code changes.**
- It automatically fine-tunes models for specific business tasks.
- It removes the need for provider-specific API credentials.

---

📖 Detalle completo con slides y código: `parte1-introduction-to-ai-agents.md` · `parte2-langchain-for-ai-agents.md`
