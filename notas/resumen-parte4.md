# 🚀 Resumen Repaso — Parte 4: OpenAI Responses API and Agents SDK

_Términos técnicos en inglés, explicaciones en español. Hecho para repasar rápido antes del examen._

---

## 🤖 OpenAI Agent Stack

`AI Agent (LLM-based) = LLM + Tools + Loop`

- 🎯 **Goal-Directed** — trabaja hacia un objetivo, no solo responde prompts.
- 🧭 **Autonomous** — decide qué hacer después sin que le digan cada paso.
- 🛠️ **Tool-Using** — interactúa con APIs, DBs, code execution, web search.
- 🔁 **Iterative** — opera en loop: observe → reason → act → observe again.

**El stack completo:**
```
Your Application (tu código, UI, lógica de negocio)
     ↓
Agents SDK (framework: agent loops, handoffs, guardrails, tracing)
     ↓
Responses API (API core: conversaciones con estado, built-in tools, function calling)
     ↓
OpenAI Models (GPT-5.5, GPT-5, GPT-4.1)
```

> 💡 Agents SDK se construye sobre la Responses API — pero podés usar la Responses API directamente, sin el SDK.

### 🔀 Cuándo usar cada una

```
¿Necesitás lógica multi-step (razonamiento encadenado, tools, múltiples turnos)?
  No  → Responses API (single call in, text out — la más simple)
  Sí  → ¿Necesitás múltiples agents, guardrails, o tracing/orchestration?
          No  → Responses API (avanzado — vos manejás el loop)
          Sí  → Agents SDK (el framework maneja todo lo difícil)
```

> 💡 Con Responses API, **vos construís el loop**. Con Agents SDK, **el loop ya está construido**.

---

## 📡 The Responses API

Interfaz **core** para interactuar con modelos OpenAI: input simple, output estructurado, tool usage, y chaining de conversaciones entre turnos.

```python
from openai import OpenAI
client = OpenAI()  # lee OPENAI_API_KEY del env automáticamente

response = client.responses.create(
    model="gpt-4.1",
    input="Explain what an AI agent is in one paragraph.",  # ¡solo un string!
)
print(response.output_text)
```

> 🆚 A diferencia de Chat Completions (`messages=[...]` + `response.choices[0].message.content`), acá es `input` como string + `.output_text`.

**🔗 Conversation Chaining** — `previous_response_id` encadena turnos sin reenviar todo el historial:
```python
resp2 = client.responses.create(
    model="gpt-4.1",
    input="What is my name?",
    previous_response_id=resp1.id,  # <-- encadena la conversación
)
```

**🧰 Built-in Tools (Model Callable):** `web_search` · `file_search` · `code_interpreter` · `computer_use_preview` (experimental) · `image_generation`

> Alcanza con `tools=[{"type": "web_search"}]` en la llamada — **el modelo decide** cuándo usarlo (no busca si ya sabe la respuesta).

---

## 🏗️ The Agents SDK

Framework de Python para definir agents, manejar tools y construir sistemas multi-agente. Se construye sobre la Responses API.

| Componente | Qué hace |
|---|---|
| **Agent** | LLM + instructions + tools |
| **Runner** | Ejecuta el agent loop |
| **Tools** | Funciones que el agent puede llamar |
| **Handoffs** | Transfieren control entre agents |
| **Guardrails** | Validan inputs/outputs por seguridad |
| **Tracing** | Monitorea qué hizo el agent |

**Install:** `pip install openai-agents`

```python
from agents import Agent, Runner

agent = Agent(
    name="Quick Helper",
    instructions="Give very brief, one-sentence answers.",
    model="gpt-4.1-mini",
)
result = Runner.run_sync(agent, "When did humans land on the moon?")
print(result.final_output)
```

**🔬 Qué pasa adentro:**
1. Tu código: `Agent(name, instructions)` → `Runner.run_sync(agent, pregunta)` → hands off al SDK.
2. SDK (loop automático): manda instructions + input al LLM → ¿quiere usar una tool? No → respuesta final. Sí → ejecuta tool, vuelve a loopear (puede repetirse varias veces).
3. Tu código: `print(result.final_output)`.

### 🛠️ Tools en el Agents SDK

- **Hosted Tools** — provistas por OpenAI (`web_search`, `file_search`, `code_interpreter`).
- **Function Tools** — tus funciones Python con `@function_tool`. El LLM decide cuándo llamarlas según la descripción (docstring).
- **Agents as Tools** — un agent usado por otro: `agent.as_tool(tool_name=..., tool_description=...)`.

```python
@function_tool
def add(a: float, b: float) -> float:
    """Add two numbers together."""
    return a + b

tools = [add, multiply, divide, square_root]
agent = Agent(name="Math Assistant", instructions="...", tools=tools, model="gpt-4.1-mini")
```

> `@function_tool` convierte cualquier función Python en una tool — type hints + docstring = schema automático.

**🔄 SDK Agent Loop — ejemplo con 2 tools en secuencia** (`"What is 15 multiplied by 8, then divided by 3?"`):
`multiply(15, 8) = 120` → feed result back to LLM → `divide(120, 3) = 40` → no more tools needed → LLM compone la respuesta final. **2 LLM calls** porque se necesitaron 2 tools en secuencia.

### ⚖️ Agents SDK vs LangChain

| Paso | LangChain | Agents SDK |
|---|---|---|
| Modelo | `init_chat_model(...)` | Se define dentro de `Agent(...)` |
| Tools | `@tool` | `@function_tool` |
| Crear agent | `create_agent(model, tools)` | `Agent(name, instructions, tools, model)` |
| Correr | `agent.invoke({"messages": [...]})` | `Runner.run_sync(agent, "question")` |
| Leer respuesta | Parsear `result["messages"]`, loop por `.type` | `result.final_output` — una sola propiedad |

---

## 👥 Multi-Agent Systems

| | Handoffs (Decentralized) | Manager (Centralized) |
|---|---|---|
| Patrón | `Triage → Specialist`. El agent que recibe toma control completo de la conversación | `Manager → Worker`. Un agent central llama a agents especializados como tools, y mantiene el control |
| Mejor para | Routing, customer support, task delegation | Orquestación compleja, experiencia unificada |
| Código | `handoffs=[math_agent, history_agent]` | `tools=[booking.as_tool(), refund.as_tool()]` |

```python
math_agent = Agent(
    name="Math Tutor",
    handoff_description="Specialist for math questions...",  # le dice al triage CUÁNDO delegarle
    instructions="...",
)

triage_agent = Agent(
    name="Triage Agent",
    instructions="Route each question to the right specialist tutor...",
    handoffs=[math_agent, history_agent],
    model="gpt-4.1",
)
```

---

## 🛡️ Guardrails and Safety

- **Input Guardrails** — corren ANTES de procesar el input: bloquear requests off-topic, detectar prompt injection, validar formato.
- **Output Guardrails** — corren DESPUÉS de generar el output: chequear alucinaciones, enforce brand guidelines, prevenir leaks de data sensible.

> 🧩 Un guardrail es un **mini-agent** (o función) que chequea input/output. Si falla, dispara un **tripwire** (excepción que detiene al agent principal antes de que haga daño).

```python
class TopicCheck(BaseModel):
    is_on_topic: bool

checker = Agent(name="Topic Checker", instructions="...", output_type=TopicCheck)

@input_guardrail
async def history_only(ctx, agent, input):
    r = await Runner.run(checker, input, context=ctx.context)
    return GuardrailFunctionOutput(tripwire_triggered=not r.final_output.is_on_topic)

# Then: Agent(..., input_guardrails=[history_only])
```

### 📊 Tracing

Cada corrida de un agent se traza automáticamente en `platform.openai.com → Dashboard → Traces`: qué agents corrieron y en qué orden, qué tools se llamaron, cuándo/por qué hubo handoffs, si los guardrails pasaron o se dispararon, y cuántos tokens consumió cada paso.

---

## 🧩 Putting It All Together

**Ejemplo: Customer Support Agent System** — `Triage Agent → Order Status / Refunds / FAQ`

✅ Triage Agent enruta requests · ✅ Specialized Agents manejan las tareas · ✅ Tools ejecutan las acciones.

> En el trace real, el handoff entre agents es **instantáneo** (0 ms — "just a pointer"). El tiempo lo consumen las llamadas al LLM (el guardrail corre en paralelo con el triage).

---

## ✅ Preguntas tipo examen (con respuesta marcada)

**1. What is the purpose of Python type hints in @function_tool definitions?**
- They determine whether tracing is enabled for the tool.
- **✅ They help generate the tool schema automatically.**
- They restrict tools to synchronous execution only.
- They control which OpenAI model can call the tool.

**2. According to the course decision framework, when is the Agents SDK the better choice over using only the Responses API?**
- **✅ When the application needs orchestration, guardrails, or multiple agents**
- When the application needs only a single text response
- When the application avoids all tool usage entirely
- When the application requires no conversational state management

**3. In the Responses API, what is the benefit of using previous_response_id when continuing a conversation?**
- **✅ It allows the API to continue a prior conversation context across turns.**
- It automatically selects the fastest available OpenAI model.
- It converts tool outputs into structured database records.
- It encrypts earlier prompts before sending new requests.

**4. In the Manager multi-agent pattern, what is the role of central manager agent?**
- It transfers full conversational control to another agent.
- **✅ It coordinates specialized agents while remaining in control.**
- It disables tool usage for all worker agents.
- It converts all handoffs into synchronous operations.

**5. What is the purpose of tracing in the OpenAI Agents SDK?**
- To permanently store model weights after fine-tuning
- **✅ To monitor agent execution steps and debugging details**
- To automatically optimize prompts before each API call
- To convert tool results into structured training datasets

---

📖 Detalle completo con slides y código: `parte4-openai-responses-api-and-agents-sdk.md`
