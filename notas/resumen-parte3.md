# 🚀 Resumen Repaso — Parte 3: Introduction to MCP

_Términos técnicos en inglés, explicaciones en español. Hecho para repasar rápido antes del examen._

---

## 🔌 Introduction to MCP (Model Context Protocol)

### ❓ Qué es MCP

Estándar abierto que da una interfaz universal para que las apps de AI se conecten con tools, data sources y sistemas externos — de forma segura y consistente.

- 🚀 **Launched** — Noviembre 2024, por Anthropic
- 🌐 **Soportado por** — OpenAI, Oracle, Microsoft, AWS
- ⚙️ **Foundation** — JSON-RPC 2.0, open source
- 🏛️ **Governed by** — Agentic AI Foundation (Linux Foundation)

> 💡 Pensá a MCP como el **USB-C de la AI** — cualquier client MCP-compatible habla con cualquier server MCP-compatible, sin importar quién lo construyó.

### 🧮 El problema que resuelve: N×M → N+M

| Antes de MCP | Después de MCP |
|---|---|
| Cada AI app necesita un conector custom para cada tool → **N×M integraciones** (ej: 3 apps × 4 tools = 12) | Cada lado construye UNA integración MCP → **N+M integraciones** (3+4 = 7) |

### 🏗️ MCP Architecture

`MCP HOST (Claude Desktop, VS Code, etc.) = LLM + MCP Client 1/2/3 → cada client conecta a UN server (File System, GitHub API, Slack)`

- Host maneja la app y crea los clients.
- Cada client se conecta a exactamente un server.
- Los servers exponen tools, data y prompts.
- Comunicación vía JSON-RPC 2.0. Los servers pueden correr local o remoto.

### 🧩 Core Primitives

| Primitive | Control | Qué hace | Analogía |
|---|---|---|---|
| 🛠️ **Tools** | Model-Controlled | Funciones que la AI puede llamar para actuar (`create_issue()`, `send_message()`) | Como un POST — ejecuta código, produce side effects |
| 📄 **Resources** | Application-Controlled | Data estructurada que la AI puede leer (`file://`, `db://`, `api://`) | Como un GET — carga info al context window |
| 💬 **Prompts** | User-Controlled | Templates que estructuran la interacción (`bug_report_template`) | Como un slash command — el usuario los invoca |

### 🔄 The MCP Connection Lifecycle

```
1. Initialize → Client manda versión del protocolo & capabilities
2. Discover   → Client pide la lista de tools/resources/prompts
3. Operate    → LLM decide qué tool llamar, client ejecuta
4. Shutdown   → Client cierra el transport y termina la sesión
```

### 📡 JSON-RPC 2.0 y métodos clave

- **`tools/list`** — el client pregunta "¿qué tools tenés?" (sin parámetros). El server devuelve cada tool con su `inputSchema` (define tipos/estructura de los argumentos).
- **`tools/call`** — el client pide ejecutar una tool específica con argumentos (`{"name": "multiply", "arguments": {"a": 15, "b": 8}}`).

> En la mayoría de los casos, frameworks como FastMCP y LangChain manejan esto por vos — pero entender la estructura ayuda a debuggear.

### 🚚 Transport Mechanisms

| STDIO | Streamable HTTP |
|---|---|
| Host lanza el server como proceso hijo, stdin/stdout. Más rápido, sin overhead de red. Un client por server, mismo máquina. Ideal para tools locales (files, git, shell) | Server corre como servicio HTTP (POST + SSE). Soporta deploy remoto/cloud, múltiples clients, auth estándar (OAuth, tokens) |

### 🔀 Antes / Después de MCP

**Antes:** todo en un archivo (`first_agent.py`), tools hardcodeadas con `@tool` — solo ese agent las puede usar, copy-paste para reusar, no hay sharing.

**Después:** dos archivos separados — **Client** (`GPT-4o → ReAct agent → MCP client`) y **Server** (`FastMCP("Math") → add(), multiply(), divide()...` vía `@mcp.tool()`).

- **Antes (ejecución de tool):** `LLM pide Tool → App llama función Python local → App obtiene resultado → App se lo manda al LLM`
- **Después (con MCP):** `LLM pide Tool → App crea MCP client → MCP client → MCP server → server ejecuta → server devuelve resultado → App se lo manda al LLM`

> 🎭 **Desde el punto de vista del LLM no cambia nada** — sigue viendo nombres y descripciones de tools, elige cuál llamar, manda argumentos como JSON y recibe resultados. Nunca sabe si vino de una función local o de un MCP server.

### ✅ Qué deja de hacer tu app con MCP

| ❌ Sin MCP, tu app hace | ✅ Con MCP, el server provee |
|---|---|
| Definir schemas (`@tool`) | Tool discovery (`tools/list`) |
| Registrar tools localmente | Schemas (`inputSchema`) |
| Mapear nombres a código | Invocación estandarizada (`tools/call`) |
| Ejecutar tools directamente | Resultados estandarizados (`content[]`) |
| Manejar errores | Error handling del lado del server |
| Mantener implementaciones | Implementación y mantenimiento |

> Tu app pasa a ser un **orquestador de tools**, no la dueña de cada integración custom.

### 🌟 Por qué MCP mejora tu Agent

- **Escribís las tools una vez** — un server, todas las apps las descubren (LangChain, Claude Desktop, Cursor, ChatGPT).
- **Actualizás en un solo lugar** — arreglás un bug en `divide()`, todos los agents conectados lo reciben al instante.
- **Mix and match** — sumás un server de GitHub + uno de Slack solo agregando entradas al config.
- **Código del agent más limpio** — el agent se enfoca en razonar, la lógica de tools vive aparte.

> ⚠️ **Cuándo realmente conviene:** Para ejemplos simples (sumar/dividir) MCP es overkill — una función Python local es más simple y rápida. MCP brilla con tools **externas o reusables**: Cloud (OCI/AWS/Azure), Databases, Dev Tools (GitHub/Jira), Comunicación (Slack/Email), File Systems remotos.

### 🏭 Real-world MCP: Math Server vs Oracle Usage Server

| | Math MCP Server (sandbox) | OCI Usage MCP Server (real) |
|---|---|---|
| ¿Quién escribe el server? | Nosotros | Oracle (vendor) |
| Tools | add, multiply, divide, square_root | `get_summarized_usage` (wrapea una REST API de OCI) |
| Objetivo | Aprender la mecánica del protocolo | Integrar con el servicio de otro |

> 💡 MCP brilla cuando **NO** tenés que escribir el server vos mismo.

> **Igual en ambos:** forma del client code, protocolo JSON-RPC, transport stdio, `list_tools`/`call_tool`.
> **Diferente:** quién es el dueño del server y qué termina wrapeando.

---

## ✅ Preguntas tipo examen (con respuesta marcada)

**1. In an MCP workflow, which component is responsible for executing the actual tool implementation?**
- **✅ The MCP server**
- The LLM directly
- The MCP host application
- The JSON-RPC transport layer

**2. In the example using MultiServerMCPClient, why can one agent access tools from multiple servers?**
- Because MCP merges all servers into one physical server process
- **✅ Because the client internally manages multiple dedicated client-server connections**
- Because JSON-RPC allows one request to execute simultaneously on all servers
- Because MCP requires all servers to expose identical tool names

**3. What is the role of the inputSchema returned during tools/list discovery?**
- It specifies the authentication method required by the server.
- **✅ It defines the expected structure and types of tool arguments.**
- It controls which transport mechanism the client must use.
- It determines whether a tool runs locally or remotely.

**4. Why is MCP described as a "standardized tool execution layer" in the course?**
- Because MCP replaces the LLM reasoning loop with server-side execution
- **✅ Because MCP standardizes how applications communicate with external tools and services**
- Because MCP converts all tools into REST APIs automatically
- Because MCP requires every tool to run inside the same programming language runtime

**5. What is the purpose of the tools/call method in MCP?**
- To allow the server to advertise all available tools and schemas
- **✅ To allow the client to request execution of a specific tool with arguments**
- To synchronize protocol versions between the client and server
- To stream resource updates continuously to connected clients

---

📖 Detalle completo con slides y código: `parte3-introduction-to-mcp.md`
