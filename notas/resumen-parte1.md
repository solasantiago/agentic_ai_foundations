# 🚀 Resumen Repaso — Parte 1: Introduction to AI Agents

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

## ✅ Preguntas tipo examen (con respuesta marcada)

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

---

📖 Detalle completo con slides y código: `parte1-introduction-to-ai-agents.md`
