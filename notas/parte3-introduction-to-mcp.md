# Parte 3 — Introduction to MCP

_Florencia Díaz · Oracle University. Model Context Protocol._

## M3: Introduction to MCP — Agenda del módulo

- What is Model Context Protocol (MCP)?
- Core components — Tools, Resources, Prompts
- Add MCP Server to your first Agent
- Real-world MCP Walkthrough

## What is MCP?

An open standard that provides a universal interface for AI applications to connect with external tools, data sources, and systems — securely and consistently.

- **Launched** — November 2024 by Anthropic
- **Supported across the ecosystem** — OpenAI, Oracle, Microsoft, AWS
- **Foundation** — JSON-RPC 2.0, Open source
- **Governed By** — Agentic AI Foundation (Linux Foundation)

> Think of MCP as USB-C for AI — any MCP-compatible client can talk to any MCP-compatible server, regardless of who built them.

## The Problem MCP Solves

**BEFORE MCP: N × M Problem**
`Claude, ChatGPT, Gemini` × `GitHub, Slack, Drive, Database` — every AI app needs a custom connector for every tool = 12 integrations (3×4).

**AFTER MCP: N + M Solution**
`Claude, ChatGPT, Gemini → MCP → GitHub, Slack, Drive, Database` — each side builds ONE MCP integration = only 7 total integrations (3+4).

## MCP Architecture

**MCP HOST** (Claude Desktop, VS Code, or any MCP-enabled AI application)
- Contains: `LLM (AI Model)` + `MCP Client 1 / 2 / 3`
- Cada client se conecta a un server distinto: `MCP Server File System`, `MCP Server GitHub API`, `MCP Server Slack`

**Key Points**
- Host manages the AI app and creates clients.
- Each client connects to exactly one server.
- Servers expose tools, data, and prompts.
- Communication uses JSON-RPC 2.0.
- Servers can run locally or remotely.

## Core Primitives

| Primitive | Control | Qué hace | Ejemplos | Analogía |
|---|---|---|---|---|
| **Tools** | Model-Controlled | Functions the AI can call to perform actions ("Do something") | `create_issue()`, `send_message()`, `query_database()`, `run_test()` | Como POST endpoints — ejecutan código y producen side effects |
| **Resources** | Application-Controlled | Structured data the AI can read for context ("Read Something") | `file://project/readme.md`, `db://users/schema`, `api://config/settings` | Como GET endpoints — cargan info en el context window del LLM |
| **Prompts** | User-Controlled | Templates that structure LLM interactions ("Structure something") | `bug_report_template`, `code_review_prompt`, `summarize_doc_template` | Como slash commands — los usuarios los invocan a través de elementos de UI |

## The MCP Connection Lifecycle

1. **Initialize** — Client sends protocol version & capabilities to server.
2. **Discover** — Client requests list of tools, resources, prompts.
3. **Operate** — LLM decides tools to call, client executes those calls.
4. **Shutdown** — Client closes transport and ends session.

## MCP uses JSON-RPC 2.0

JSON-RPC messages include:
- **jsonrpc** (always "2.0").
- **id** (used to match requests and responses, when applicable).
- One of: **method** (request), **result** (success), or **error** (failure).

> In most cases, frameworks like FastMCP and LangChain handle this for you — but understanding the structure helps with debugging.

**Request (client → server):**
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "multiply",
    "arguments": { "a": 15, "b": 8 }
  }
}
```

**Response (server → client):**
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "content": [
      { "type": "text", "text": "120.0" }
    ]
  }
}
```

## MCP Method #1: tools/list

- Typically called when the client connects to discover available tools.
- The server returns every tool it offers.
- Agent discovers tools without you hardcoding them.

**Request (client → server):** that's it — no parameters needed. Just "tell me what you have."
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list"
}
```

**Response (server → client):**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": { "tools": [
    {
      "name": "multiply",
      "description": "Multiply two numbers",
      "inputSchema": {
        "type": "object",
        "properties": {
          "a": {"type": "number"},
          "b": {"type": "number"}
        }
      }
    },
    "... add, divide, square_root ..."
  ]}
}
```

## MCP Transport Mechanisms

| STDIO Transport | Streamable HTTP Transport |
|---|---|
| Host spawns server as a child process | Server runs as an HTTP service |
| Messages flow through stdin / stdout | Client → Server via HTTP POST |
| No network overhead — fastest option | Server → Client via SSE streaming |
| Server runs on the same machine | Supports remote / cloud deployment |
| One client per server instance | Multiple clients can connect |
| Best for local tools (files, git, shell) | Standard HTTP auth (OAuth, tokens) |

`Client ←stdin/stdout→ Server` vs. `Client ←POST/SSE→ Server`

## Add MCP Server to your First Agent

### Before MCP: First Agent

Everything lives in one file (`first_agent.py`) — tightly coupled.

`GPT-4o (The AI brain) → ReAct agent (Reason → Act → Observe) → calls: add(), multiply(), divide(), square_root()`

**The problem:**
- Tools are hardcoded with `@tool` decorators.
- Only THIS agent can use them.
- Want the same tools in Claude? Copy-paste the code.
- Change a tool? Must update every agent that has a copy.
- No sharing, no reuse.

### After MCP: Two separate files (Client, Server)

Tools live on a server — any AI app can discover and use them.

**`first_agent_with_mcp.py` — The agent (consumer)**
`GPT-4o (The AI brain) → ReAct agent (Reason → Act → Observe) → MCP client (langchain-mcp-adapters)`

**`first_agent_with_mcp.py` — The MCP server (provider)**
`FastMCP("Math") (Server framework) → add(), multiply(), divide(), square_root()` — vía `@mcp.tool()` decorators.

### MCP Server (código)

```python
from mcp.server.fastmcp import FastMCP
import math

# 1. Create the MCP server
mcp = FastMCP("Math")

# 2. These are the SAME tools from first_agent.py but now exposed via MCP instead of @tool
@mcp.tool()
def add(a: float, b: float) -> float:
    """Add two numbers together. Use for addition operations."""
    return a + b

@mcp.tool()
def multiply(a: float, b: float) -> float:
    """Multiply two numbers together. Use for multiplication operations."""
    return a * b

# 3. Run the server
# transport="stdio" means the server communicates via stdin/stdout — the agent launches it as a subprocess.
if __name__ == "__main__":
    mcp.run(transport="stdio")
```

### Agent (MCP Client) connects to the MCP Server

```python
from langchain_mcp_adapters.client import MultiServerMCPClient
from langchain.agents import create_agent

async def main():
    """
    Main async function — MCP connections are async because they involve I/O (spawning processes, network calls).
    """
    # Connect to the MCP Server
    # Instead of defining tools locally, we point to an MCP server and let agent discover tools automatically.
    # Get the absolute path to the MCP server script
    current_dir = os.path.dirname(os.path.abspath(__file__))
    server_path = os.path.join(current_dir, "mcp_math_server.py")

    client = MultiServerMCPClient(
        {
            "math": {
                # The MCP server to connect to
                "command": "python",
                "args": [server_path],
                "transport": "stdio",  # Local process (stdin/stdout)
            },
        }
    )
```

### Agent (MCP Client) discovers (and calls) tools from MCP Server

```python
# Discover Tools from the MCP Server
# This is where the magic happens!
# The client connects to the server, performs the MCP
# handshake, and auto-discovers all available tools.
# Each MCP tool is converted into a LangChain tool.

tools = await client.get_tools()

print("=" * 55)
print("  MCP Agent — Tools discovered from MCP Server")
print("=" * 55)
print(f"\n Found {len(tools)} tools from MCP server:\n")
for t in tools:
    print(f"  {t.name}: {t.description[:60]}...")
print()
```

**Output:**
```
MCP Agent — Tools discovered from MCP Server
Found 4 tools from MCP server:
  add: Add two numbers together. Use for addition operations....
  multiply: Multiply two numbers together. Use for multiplication operat...
  divide: Divide the first number by the second. Returns error if divi...
  square_root: Calculate the square root of a number....
```

### After MCP: Agent Architecture

**Before MCP:** `User ↔ Your Application ↔ LLM ↔ Local @tool Functions`

**After MCP:** `User ↔ Your Application ↔ LLM ↔ MCP Client ↔ MCP Server ↔ Tool Implementation` (MCP Client + MCP Server = "MCP Layer")

> MCP inserts a standardized tool execution layer — it does not replace the LLM or the agent loop.

### After MCP: What Changed (Tool Execution)

**Before MCP:**
`LLM asks for multiply Tool → Agent (App) calls local Python function → Agent (App) gets result → Agent (App) sends result to LLM`

**After MCP:**
`LLM asks for multiply Tool → Agent (App) creates MCP client → MCP client → MCP server → MCP server executes tool → MCP server returns result → Agent (App) sends result to LLM`

### What Your Agent App No Longer Needs To Do

MCP shifts responsibilities from your application to the MCP server.

| ❌ Without MCP: Your App Does | ✅ With MCP: Server Provides |
|---|---|
| Define tool schemas (`@tool` decorators) | Tool discovery (`tools/list`) |
| Register tools in a local registry | Tool schemas (`inputSchema`) |
| Map tool names to code | Standardized invocation (`tools/call`) |
| Execute tools directly | Standardized results (`content[]`) |
| Handle errors and edge cases | Server-side error handling |
| Maintain all tool implementations | Tool implementation & maintenance |

> Your app becomes a tool orchestrator, not a custom integration owner.

### From the LLM's Point of View

**What the LLM Sees:**
```json
{"role": "tool", "tool_call_id": "call_001", "content": "120"}
{"role": "tool", "tool_call_id": "call_002", "content": "40.0"}
```
> The LLM never knows if this came from a local function or an MCP server.

**The LLM still:**
- Sees tool names and descriptions
- Chooses which tool to call
- Provides arguments as JSON
- Receives tool results back
- Reasons step by step (ReAct)
- Produces the final answer

> The LLM sees absolutely no difference between MCP and non-MCP.

### Why MCP makes your Agent better

`MCP Math Server` → usado por las 4 apps al mismo tiempo: **Your agent** (LangChain), **Claude** (Desktop), **Cursor** (IDE), **ChatGPT** (desktop) — all 4 apps use the same tools, build once, use everywhere.

- **Write tools once** — Define `add()`, `multiply()` etc. in one server file. Every AI app discovers them automatically.
- **Update in one place** — Fix a bug in `divide()`? Change the server — all connected agents get the fix instantly.
- **Mix and match servers** — Add a GitHub server + Slack server to the same agent: just add entries to config dictionary.
- **Agent code stays clean** — Your agent focuses on reasoning. Tool logic lives separately. Easier to test and maintain.

### When Does MCP Actually Help?

> For our math example (multiply, divide), MCP is overkill. A local Python function is simpler and faster.

**MCP Shines When Tools Are External or Reusable:**
- **Cloud Resources** — OCI, AWS, Azure — manage infra without custom SDKs.
- **Databases** — Oracle DB, PostgreSQL — query without embedding drivers.
- **Dev Tools** — GitHub, Jira — create issues, review PRs from any agent.
- **Communication** — Slack, Email — send messages without per-app integrations.
- **File Systems** — Read/write files on remote machines securely.

## Real-world MCP Walkthrough

### From Sandbox to Real World MCP

**What we Built So Far — Math MCP Server**
- We created both the client AND the server.
- Tools: add, multiply, divide, square root (Python functions).
- Controlled, easy to debug, fast to iterate.
- Goal: learn the MCP protocol mechanics.

**What Real World MCP Looks Like — OCI Usage MCP Server (Oracle's code)**
- Vendor publishes the server; you write only the client.
- Tool: `get_summarized_usage` (wraps a cloud REST API).
- Auth, retries, schemas — all handled by Oracle.
- Goal: integrate with someone else's service.

> MCP shines when you DON'T write the server.

### Different Boundaries — Math vs Oracle Usage MCP server

**Math (we wrote both sides):**
`MCP Client Code (we wrote) — first_agent_with_mcp.py` → _stdio · JSON-RPC 2.0_ → `MCP Math Server (we wrote) — mcp_math_server.py` → _Python function call_ → `Local Python tools (we wrote) — add(), subtract(), multiply()`

**Oracle Usage (vendor-provided server):**
`MCP Client code (we wrote) — oci_usage_mcp_client.py` → _stdio · JSON-RPC 2.0_ → `Oracle Usage MCP server (Oracle wrote) — uvx oracle.oci-usage-mcp-server` → _HTTPS · OCI SDK_ → `OCI Usage REST API (Oracle hosts) — RequestSummarizedUsages`

> **Same:** Client code shape, JSON-RPC protocol, Stdio transport, `list_tools`/`call_tool`.
> **Different:** who owns the server and what the server ultimately wraps.

### OCI Usage MCP Server Details

- **Who Publishes it?** — Oracle. Published on PyPI as `oracle.oci-usage-mcp-server`. Maintained alongside the rest of OCI's MCP Servers.
- **What does it Wrap?** — OCI Usage REST API. `RequestSummarizedUsages` — the same endpoint behind Console's Cost Analysis page.
- **What Tools it exposes?** — `get_summarized_usage`. One tool. Arguments include `tenant_id`, `start_time`, `end_time`, `group_by`, `granularity`, `query_type`.

**ARCHITECTURE**

`[this Python file (the MCP client)] ←stdio→ [oracle.oci-usage-mcp-server (Oracle's code)] ←HTTPS→ [OCI Usage API]`

```python
from mcp import StdioServerParameters

server_params = StdioServerParameters(
    command="uvx",
    args=["oracle.oci-usage-mcp-server"],
    env={**os.environ,
    "OCI_CONFIG_PROFILE": "DEFAULT"},
)
```

## Preguntas de práctica / quiz — Introduction to MCP

**1. In an MCP workflow, which component is responsible for executing the actual tool implementation?**
- The MCP server ✅ (ver "MCP Architecture"/"After MCP: What Changed": el MCP server es quien ejecuta el tool, el LLM solo pide la tool call)
- The LLM directly
- The MCP host application
- The JSON-RPC transport layer

**2. In the example using MultiServerMCPClient, why can one agent access tools from multiple servers?**
- Because MCP merges all servers into one physical server process
- Because the client internally manages multiple dedicated client-server connections ✅ (ver "Agent (MCP Client) connects to the MCP Server": `MultiServerMCPClient({"math": {...}, ...})` — un diccionario con una entrada/conexión por server)
- Because JSON-RPC allows one request to execute simultaneously on all servers
- Because MCP requires all servers to expose identical tool names

**3. What is the role of the inputSchema returned during tools/list discovery?**
- It specifies the authentication method required by the server.
- It defines the expected structure and types of tool arguments. ✅ (ver "MCP Method #1: tools/list": `inputSchema` describe `type`, `properties` como `a`/`b` con sus tipos)
- It controls which transport mechanism the client must use.
- It determines whether a tool runs locally or remotely.

**4. Why is MCP described as a "standardized tool execution layer" in the course?**
- Because MCP replaces the LLM reasoning loop with server-side execution
- Because MCP standardizes how applications communicate with external tools and services ✅ (ver "After MCP: Agent Architecture": "MCP inserts a standardized tool execution layer — it does not replace the LLM or the agent loop")
- Because MCP converts all tools into REST APIs automatically
- Because MCP requires every tool to run inside the same programming language runtime

**5. What is the purpose of the tools/call method in MCP?**
- To allow the server to advertise all available tools and schemas
- To allow the client to request execution of a specific tool with arguments ✅ (ver "MCP uses JSON-RPC 2.0": Request `"method": "tools/call", "params": {"name": "multiply", "arguments": {...}}`)
- To synchronize protocol versions between the client and server
- To stream resource updates continuously to connected clients
