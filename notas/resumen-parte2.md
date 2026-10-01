# 🚀 Resumen Repaso — Parte 2: LangChain for AI Agents

_Términos técnicos en inglés, explicaciones en español. Hecho para repasar rápido antes del examen._

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

**1. What information does LangChain send to the model API during the first agent request?**
- **✅ The user message and the available tool definitions**
- The final answer and the complete execution trace
- The Python source files and environment variables
- The vector database contents and cached embeddings

**2. Why does the model return a tool call instead of executing Python code directly?**
- **✅ The model can only reason and request actions.**
- The model runs inside a restricted Python sandbox.
- The model requires manual approval before execution.
- The model can execute only built-in LangChain tools.

**3. Why is understanding LangChain internals important for developers?**
- It allows models to train themselves during execution.
- **✅ It helps with debugging and production optimization.**
- It removes the need for tool descriptions and schemas.
- It guarantees that every agent produces correct answers.

**4. What is the main purpose of LCEL in LangChain?**
- **✅ It connects multiple chain components using the pipe operator.**
- It stores vector embeddings for retrieval applications.
- It converts tools into external REST API services.
- It encrypts prompts before sending them to the model.

**5. Why does LangChain provide a unified model interface?**
- It guarantees identical behavior across all LLM providers.
- **✅ It allows developers to switch providers with minimal code changes.**
- It automatically fine-tunes models for specific business tasks.
- It removes the need for provider-specific API credentials.

---

📖 Detalle completo con slides y código: `parte2-langchain-for-ai-agents.md`
