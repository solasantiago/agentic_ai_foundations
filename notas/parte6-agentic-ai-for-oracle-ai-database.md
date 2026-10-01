# Parte 6 — Agentic AI for Oracle AI Database

_Oracle University. Agentic AI Innovations for Oracle AI Database._

## Lesson 1 — Agentic AI for Oracle AI Database

### Agenda del módulo

- **Oracle AI Vector Search** (Oracle Autonomous AI Vector Database — LA) — Build vector powered applications.
- **Oracle AI Database Private Agent Factory** — Create enterprise AI agents without writing code.
- **Select AI Agent** — Build autonomous agents that live inside the Oracle AI Database.
- **Oracle Autonomous AI Database MCP Server** — Leverage MCP server built-in the database.

> Oracle Deep Data Security, Oracle Trusted Answer Search, Oracle Vectors on Ice, Oracle Private AI Services Container..

### The Problem: Why Keywords Aren't Enough

**Keyword Search**
Traditional databases use exact keyword matching. Searching for "canine" won't find documents about "dogs", even though they mean the same thing.
```sql
SELECT * FROM docs
WHERE text LIKE '%canine%';

-- Result: 0 rows (missed "dog" docs!)
```

**Semantic / Vector Search**
Vector search understands meaning. "Canine" and "dog" are close in vector space, so semantic search finds relevant results regardless of exact words.
```sql
SELECT * FROM docs
ORDER BY VECTOR_DISTANCE(
    embedding, query_vec)
FETCH FIRST 5 ROWS ONLY;

-- Result: 5 rows (found them!)
```

### Vectors & Embeddings: Turning Meaning into Math

**What Is a Vector?**
A vector is simply an ordered list of numbers that represents the "location" of something in a high-dimensional space.
`"dog" → [0.82, -0.15, 0.67, ...]`

**What Is an Embedding?**
An embedding is a vector created by a neural network (embedding model) that captures the semantic meaning of input data.
Embedding Models: Hugging Face all-MiniLM-L6-v2, OpenAI text-embedding-3-large.

**Think of It Like a Map**
Imagine a map where every word, sentence, or document has a location. Similar things are plotted close together, different things are far apart.

`{dog, puppy, canine}` cerca entre sí · `{cat, kitten, feline}` cerca entre sí pero lejos del grupo anterior · `{car, vehicle}` lejos de ambos grupos.

## Lesson 3 — Oracle AI Vector Search Workflow

### What Is Oracle AI Vector Search?

Oracle AI Vector Search is a feature built directly into Oracle AI Database that lets you store, index, and search vector embeddings alongside your traditional business data — all in one system, using SQL.

- **No Separate Vector Database** — Vectors live alongside relational data. No data fragmentation.
- **SQL-Native Vector Search** — Use `VECTOR_DISTANCE()` in standard SQL queries.
- **Enterprise Grade** — Full Oracle security, ACID transactions, and scalability.

> **Key Insight:** Combine semantic vector search with relational predicates, e.g., find houses that look like this picture and `WHERE price < 500 AND city = 'NYC'` in a single SQL query.

### Oracle AI Vector Search Workflow

Five primary steps from raw data to intelligent search:

1. **Generate Embeddings** — Use ONNX models in-DB or external APIs (OpenAI, Cohere) to convert data to vectors.
2. **Store Vectors** — Store vectors alongside business data in Oracle tables using the `VECTOR` data type.
3. **Create Indexes** — Build HNSW or IVF vector indexes, or Hybrid Vector Indexes for combined search.
4. **Search & Query** — Run similarity searches with `VECTOR_DISTANCE()`, combine with relational `WHERE` clauses.
5. **RAG & LLM** — Feed search results into LLMs for Retrieval Augmented Generation responses.

### Generating Vector Embeddings

Two approaches: generate embeddings inside or outside Oracle Database.

**In-Database (ONNX Runtime)** — Import ONNX-format models directly into Oracle. Generate embeddings using SQL — no external calls needed.
```sql
-- Load ONNX model
EXEC DBMS_VECTOR.LOAD_ONNX_MODEL(
    'VEC_DUMP',
    'my_model.onnx',
    'doc_model');

-- Generate embedding
SELECT VECTOR_EMBEDDING(
    doc_model USING 'hello' AS data
) AS embedding;
```

**External (REST APIs)** — Call third-party APIs (OpenAI, Cohere, Hugging Face) via PL/SQL. More model flexibility.
```sql
-- Using DBMS_VECTOR_CHAIN
SELECT DBMS_VECTOR_CHAIN
    .UTL_TO_EMBEDDING(
    'Oracle AI Vector Search',
    JSON('{
        "provider":"openai",
        "model":"text-embedding-3-small"
    }')
) AS embedding
FROM dual;
```

### Chunking: Splitting Documents for Embedding

Embedding models have token limits. Large documents must be split into chunks before embedding.

`PDF/DOC (UTL_TO_TEXT()) → Plain Text (UTL_TO_CHUNKS()) → Chunks (UTL_TO_EMBEDDINGS()) → Vectors (Store & Index)`

**One-Statement Pipeline: PDF → Text → Chunks → Vectors**
```sql
INSERT INTO doc_chunks
SELECT dt.id doc_id, et.embed_id chunk_id,
       et.embed_data chunk_data, to_vector(et.embed_vector) chunk_embedding
FROM documentation_tab dt,
     dbms_vector_chain.utl_to_embeddings(
         dbms_vector_chain.utl_to_chunks(
             dbms_vector_chain.utl_to_text(dt.data),
             json('{"normalize":"all"}')),
         json('{"provider":"database", "model":"doc_model"}')) t,
     JSON_TABLE(t.column_value, '$[*]'
         COLUMNS (embed_id NUMBER PATH '$.embed_id',
                  embed_data VARCHAR2(4000) PATH '$.embed_data',
                  embed_vector CLOB PATH '$.embed_vector')) et;
```

### The VECTOR Data Type

Oracle introduced a native `VECTOR` column type. You declare it like any other column:

```sql
-- Flexible: any dimensions, any format
CREATE TABLE my_vectors (
    id         NUMBER,
    embedding  VECTOR
);

-- Strict: exactly 768 dims, INT8 format
CREATE TABLE my_vectors (
    id         NUMBER,
    embedding  VECTOR(768, INT8)
);

-- Insert a vector
INSERT INTO my_vectors
VALUES (1, '[10, 20, 30]');
```

**Dimension Formats**
- **FLOAT32** — 32-bit floats (default).
- **FLOAT64** — 64-bit floats (precision).
- **INT8** — 8-bit integers (compact).
- **BINARY** — 1-bit per dimension.

> **DENSE vs. SPARSE:** Use `VECTOR(dims, format, SPARSE)` for vectors with mostly zero values — only non-zero values are stored. Max 65,535 dimensions.

### Distance Functions & Similarity Search

Finding similar vectors = measuring distance between them.

- **COSINE** — `COSINE_DISTANCE()`. Measures the angle between vectors. Most popular for text embeddings. Default in Oracle.
- **EUCLIDEAN** — `L2_DISTANCE()`. Measures straight-line distance between points. Good for spatial data.
- **DOT PRODUCT** — `INNER_PRODUCT()`. Measures both magnitude and direction. Fast computation.
- **MANHATTAN** — `L1_DISTANCE()`. Sum of absolute differences along each dimension. Robust to outliers.

```sql
-- Find top 5 most similar documents to a query vector
SELECT doc_id, chunk_data
FROM   doc_chunks
ORDER BY VECTOR_DISTANCE(chunk_embedding, :query_vector, COSINE)
FETCH FIRST 5 ROWS ONLY;
```

## Lesson 5 — Oracle AI Database Private Agent Factory

### What Is Oracle AI Database Private Agent Factory?

> Agent Factory is a no-code platform that lets both business users and engineers rapidly build, test, and deploy intelligent AI agents — without writing any code.

- **Pre-built Agents** — Ready-to-use Knowledge and Data Analysis agents out of the box.
- **No-Code Builder** — Visual drag-and-drop interface for designing custom agents.
- **Enterprise Data** — Connect to Oracle DB, SharePoint, OCI Storage, and websites.
- **Secure & Governed** — SSO, role-based access, prompt guardrails, grounded responses.

### How Agent Factory Works

`Business User (Asks a question in plain English) → Agent Factory (Routes request through agent logic) → LLM Provider (LLM generates a response) → Oracle Database (Grounded with real data)`

**Deployment Flexibility**
- Agent Factory runs as a containerized service (Podman).
- It can be deployed on-premises, OCI, and across multi-cloud — wherever your Oracle AI Database 26ai runs.

### Knowledge Agent

_Your intelligent enterprise search assistant._

**Data Sources:** File Systems & Documents · Websites & Web Pages · OCI Object Storage · Microsoft SharePoint.

- **Connect Your Sources** — Link SharePoint, file folders, websites, or OCI buckets as knowledge sources.
- **Automatic Vectorization** — Uses Oracle AI Vector Search to index your content for semantic search.
- **Ask in Plain English** — Users ask questions naturally. Agent retrieves only relevant, approved content.
- **Traceable Answers** — Every response is source-linked, so users can verify answer sources.

### Data Analysis Agent

_Your intelligent database exploration assistant._

**Compatible with:** Oracle Database 19c and above · Oracle AI Database 26ai · Autonomous Database (ADB) · Self-managed & Cloud instances.

- **Natural Language to SQL** — Ask business questions in plain English — agent writes and executes the database query for you.
- **Table & View Understanding** — Automatically explores table structures and extracts semantic meaning using advanced LLM capabilities.
- **Auto Visualization** — Generates charts and graphs automatically to make insights visually clear and instantly actionable.
- **Context-Aware Question Suggestions** — Suggests smart follow-up questions based on data patterns discovered through variation analysis.

### Agent Builder

Agent Builder is a drag-and-drop visual interface to design, prototype, and deploy AI agents with no coding required.

- **Simple Workflows** — Automate straightforward request-response flows. Great for FAQ bots and basic task automation.
- **Enhanced Agents with Tools** — Add built-in tools (web search, email, calculators) or connect external APIs via MCP servers.
- **Python Custom Components** — Advanced users can embed custom Python functions for specialized logic within the visual flow.
- **Multi-Agent Orchestrations** — Coordinate multiple specialized agents — a main agent delegates tasks to sub-agents and aggregates results.

## Lesson 6 — Oracle Autonomous AI Database MCP Server

### What Is the Autonomous AI Database MCP Server?

_A multi-tenant, built-in feature of Autonomous AI Database Serverless that exposes Model Context Protocol (MCP) endpoints, letting AI agents invoke tools you define with Select AI Agent._

- **Built into the database** — The MCP server runs as a managed service inside Autonomous AI DB itself. Nothing to install, deploy, scale, or patch.
- **Speaks the MCP standard** — Any MCP client works: Claude Desktop, OCI AI Agent, VS Code + Cline, custom apps. No bespoke adapters.

### Why a Database-Native MCP Server?

_Same MCP standard, but the server is part of the database — less to operate, tighter security, faster NL2SQL._

**❌ Typical Third-Party DB MCP Server**
- **Runs OUTSIDE the database** — You deploy, run, scale, patch, and secure a separate MCP process — usually next to the DB on a VM or container.
- **Schema discovery on every query** — Before an NL→SQL call, the LLM must list schemas, enumerate objects, and fetch metadata. Many round-trips, more tokens, slower.
- **Extra trust boundary** — Security policies (RBAC, ACLs, VPD) live in the DB. A third-party server adds a second authorization layer to coordinate.

**✅ Autonomous AI Database MCP Server**
- **Runs INSIDE the database** — Multi-tenant, Oracle-managed. No process to deploy or scale — turn it on with a tag and the endpoint appears.
- **Skips schema discovery** — Select AI Agent tools use an AI profile that already knows the schema, so NL2SQL queries skip the discovery round-trips.
- **DB enforces every policy** — Authentication uses DB credentials. RBAC, ACLs, VPD, and auditing are enforced by the database engine — one source of truth.

### How It Works — Architecture

`MCP client → HTTPS endpoint → Autonomous AI DB → Select AI Agent tools`

**MCP Clients:** Claude Desktop · OCI AI Agents · VS Code + Cline · Custom apps (LangChain, OpenAI Agents SDK, etc.) — conectan vía _MCP over HTTPS (streamable-http)_.

**Oracle Autonomous AI Database (19c / 26ai):**
- **MCP Server endpoint** — Multi-tenant, Oracle-managed, exposed per database OCID.
- **Select AI Agent tools** (instantly available via MCP):
  - **NL2SQL** — Natural language → SQL using an AI profile.
  - **RAG** — Retrieval-augmented generation over your data.
  - **Custom PL/SQL** — Your own functions wrapped as MCP tools.

**Endpoint:** `https://dataaccess.adb.<region>.oraclecloudapps.com/adb/mcp/v1/databases/<database-ocid>`

> The AI client signs in with a database username and password. The user's roles, ACLs, and VPD policies decide which tools and rows the AI is allowed to see.

## Preguntas de práctica / quiz — Agentic AI for Oracle AI Database

**1. MCP Connectivity: How do MCP clients communicate with the Autonomous AI Database MCP Server?**
- HTTPS using the MCP protocol ✅ (ver Lesson 6 — "How It Works — Architecture": MCP over HTTPS (streamable-http))
- FTP using database file transfers
- SMTP using email-based requests
- Bluetooth using local pairing sessions

**2.** _(no se capturó la foto de esta pregunta)_

**3. Distance Functions: Which distance metric is the Oracle default for text embedding searches?**
- COSINE distance ✅ (ver Lesson 3 — Distance Functions: "COSINE — Most popular for text embeddings. Default in Oracle")
- Manhattan distance
- Euclidean distance
- Hamming distance

**4. Select AI Agent Architecture: Where does Select AI Agent run and orchestrate workflows?**
- Inside Oracle AI Database ✅ (ver Lesson 6 — Autonomous AI Database MCP Server: "Select AI Agent tools use an AI profile that already knows the schema"; corre adentro de la base)
- Inside Oracle Linux kernel modules
- Inside browser-based JavaScript clients
- Inside external GPU rendering clusters

**5. VECTOR Data Type: Which Oracle data type is designed for storing embeddings?**
- VECTOR ✅ (ver Lesson 3 — "The VECTOR Data Type": Oracle introduced a native VECTOR column type)
- CLOB
- JSON
- XMLTYPE
