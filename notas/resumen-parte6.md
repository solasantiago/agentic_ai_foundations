# 🚀 Resumen Repaso — Parte 6: Agentic AI for Oracle AI Database

_Términos técnicos en inglés, explicaciones en español. Hecho para repasar rápido antes del examen._

---

## 🧩 Agentic AI Innovations for Oracle AI Database

**4 pilares del módulo:**
- **Oracle AI Vector Search** — construir apps potenciadas por vectores.
- **Oracle AI Database Private Agent Factory** — crear agents enterprise sin escribir código.
- **Select AI Agent** — agents autónomos que viven adentro de la base de datos.
- **Oracle Autonomous AI Database MCP Server** — MCP server integrado en la base.

> 🔑 **El problema: por qué el keyword search no alcanza.** `LIKE '%canine%'` no encuentra documentos sobre "dog" — búsqueda exacta, 0 resultados. El **semantic/vector search** entiende el significado: "canine" y "dog" están cerca en el espacio vectorial → 5 resultados encontrados con `VECTOR_DISTANCE()`.

### 🧮 Vectors & Embeddings

- **Vector** — lista ordenada de números que representa la "ubicación" de algo en un espacio de alta dimensión. `"dog" → [0.82, -0.15, 0.67, ...]`
- **Embedding** — vector creado por una red neuronal que captura el significado semántico. Modelos: Hugging Face all-MiniLM-L6-v2, OpenAI text-embedding-3-large.
- **Analogía del mapa** — cosas similares quedan cerca (`dog, puppy, canine` juntos), cosas distintas quedan lejos (`car, vehicle` aparte).

---

## 🔍 Oracle AI Vector Search Workflow

**¿Qué es?** Feature integrado en Oracle AI Database que permite guardar, indexar y buscar vector embeddings junto a tus datos de negocio — todo en un solo sistema, con SQL.

- **No Separate Vector Database** — los vectores viven junto a los datos relacionales, sin fragmentación.
- **SQL-Native** — `VECTOR_DISTANCE()` en queries SQL estándar.
- **Enterprise Grade** — seguridad, ACID, escalabilidad de Oracle.

> 💡 **Key Insight:** combinás búsqueda vectorial semántica con predicados relacionales en una sola query — ej. "casas que se vean como esta foto" + `WHERE price < 500 AND city = 'NYC'`.

**🔄 Los 5 pasos del workflow:**
1. **Generate Embeddings** — modelos ONNX in-DB o APIs externas (OpenAI, Cohere).
2. **Store Vectors** — tipo de dato `VECTOR` en tablas Oracle.
3. **Create Indexes** — HNSW, IVF, o Hybrid Vector Indexes.
4. **Search & Query** — `VECTOR_DISTANCE()` combinado con `WHERE` relacional.
5. **RAG & LLM** — los resultados alimentan a un LLM para generar la respuesta.

**⚙️ Generando embeddings — dos caminos:**
- **In-Database (ONNX Runtime)** — `DBMS_VECTOR.LOAD_ONNX_MODEL(...)` + `VECTOR_EMBEDDING(...)`. Sin llamadas externas.
- **External (REST APIs)** — `DBMS_VECTOR_CHAIN.UTL_TO_EMBEDDING(...)` llamando OpenAI/Cohere/Hugging Face vía PL/SQL. Más flexibilidad de modelo.

**✂️ Chunking** — los embedding models tienen límite de tokens, hay que partir documentos grandes antes de embeddearlos: `PDF/DOC → Plain Text → Chunks → Vectors` (`UTL_TO_TEXT()` → `UTL_TO_CHUNKS()` → `UTL_TO_EMBEDDINGS()`).

**📦 El tipo `VECTOR`:**
```sql
CREATE TABLE my_vectors (
    id        NUMBER,
    embedding VECTOR(768, INT8)   -- dims, formato
);
```
Formatos: **FLOAT32** (default) · **FLOAT64** (precisión) · **INT8** (compacto) · **BINARY** (1 bit/dimensión). DENSE vs. SPARSE (solo valores no-cero) — hasta 65.535 dimensiones.

**📏 Distance Functions:**
| Función | SQL | Uso |
|---|---|---|
| **COSINE** | `COSINE_DISTANCE()` | Ángulo entre vectores. **Default de Oracle, el más usado para text embeddings** |
| **EUCLIDEAN** | `L2_DISTANCE()` | Distancia en línea recta. Bueno para datos espaciales |
| **DOT PRODUCT** | `INNER_PRODUCT()` | Magnitud + dirección. Cómputo rápido |
| **MANHATTAN** | `L1_DISTANCE()` | Suma de diferencias absolutas. Robusto a outliers |

---

## 🏭 Oracle AI Database Private Agent Factory

> Plataforma **no-code** que deja a usuarios de negocio e ingenieros construir, testear y desplegar AI agents inteligentes sin escribir código.

- **Pre-built Agents** — Knowledge y Data Analysis agents listos para usar.
- **No-Code Builder** — interfaz visual drag-and-drop.
- **Enterprise Data** — conecta a Oracle DB, SharePoint, OCI Storage, websites.
- **Secure & Governed** — SSO, acceso basado en roles, guardrails, respuestas fundamentadas.

**Cómo funciona:** `Business User (pregunta en inglés plano) → Agent Factory (rutea vía agent logic) → LLM Provider (genera respuesta) → Oracle Database (fundamentada con datos reales)`

> Corre como servicio containerizado (Podman) — desplegable on-premises, OCI, o multi-cloud, donde sea que corra tu Oracle AI Database 26ai.

**🔎 Knowledge Agent** — tu asistente de búsqueda enterprise. Conecta fuentes (SharePoint, OCI buckets, websites), vectoriza automáticamente con Oracle AI Vector Search, responde en lenguaje natural y **cada respuesta está linkeada a su fuente** (trazable).

**📊 Data Analysis Agent** — tu asistente de exploración de BD. Compatible con Oracle 19c+/26ai/ADB. **Natural Language to SQL** (escribe y ejecuta la query por vos), entiende tablas/vistas automáticamente, genera visualizaciones automáticas, y sugiere preguntas de follow-up.

**🧱 Agent Builder** — interfaz visual no-code: Simple Workflows (FAQ bots) · Enhanced Agents with Tools (built-in tools o APIs externas vía MCP) · Python Custom Components (lógica custom) · Multi-Agent Orchestrations (un agent principal delega a sub-agents).

---

## 🔌 Oracle Autonomous AI Database MCP Server

> Feature multi-tenant integrada en Autonomous AI Database Serverless que expone endpoints MCP, dejando que AI agents invoquen tools definidas con Select AI Agent.

- **Built into the database** — corre como servicio administrado adentro de la DB. Nada que instalar, desplegar, escalar o parchear.
- **Speaks the MCP standard** — cualquier client MCP funciona: Claude Desktop, OCI AI Agent, VS Code + Cline, apps custom. Sin adaptadores a medida.

**🆚 Por qué un MCP server nativo de la base:**

| Third-Party DB MCP Server (❌) | Autonomous AI Database MCP Server (✅) |
|---|---|
| Corre AFUERA de la DB — hay que desplegar, escalar y parchear un proceso separado | Corre ADENTRO de la DB — multi-tenant, administrado por Oracle, se activa con un tag |
| Schema discovery en cada query — más round-trips, más tokens, más lento | Se saltea el schema discovery — el AI profile ya conoce el schema |
| Boundary de confianza extra — un segundo layer de autorización para coordinar | La DB aplica todas las políticas (RBAC, ACLs, VPD, auditoría) — una sola fuente de verdad |

**🏗️ Arquitectura:** `MCP client → HTTPS endpoint → Autonomous AI DB → Select AI Agent tools`

- **MCP Clients** — Claude Desktop, OCI AI Agents, VS Code + Cline, apps custom (LangChain, OpenAI Agents SDK) — vía MCP over HTTPS (streamable-http).
- **Select AI Agent tools** (instantáneamente disponibles vía MCP): **NL2SQL** (lenguaje natural → SQL) · **RAG** (retrieval-augmented generation) · **Custom PL/SQL** (tus funciones envueltas como MCP tools).

> El AI client se autentica con usuario/contraseña de la base. Los roles, ACLs y políticas VPD del usuario deciden qué tools y filas puede ver la IA.

---

## ✅ Preguntas tipo examen (con respuesta marcada)

**1. MCP Connectivity: How do MCP clients communicate with the Autonomous AI Database MCP Server?**
- **✅ HTTPS using the MCP protocol**
- FTP using database file transfers
- SMTP using email-based requests
- Bluetooth using local pairing sessions

**3. Distance Functions: Which distance metric is the Oracle default for text embedding searches?**
- **✅ COSINE distance**
- Manhattan distance
- Euclidean distance
- Hamming distance

**4. Select AI Agent Architecture: Where does Select AI Agent run and orchestrate workflows?**
- **✅ Inside Oracle AI Database**
- Inside Oracle Linux kernel modules
- Inside browser-based JavaScript clients
- Inside external GPU rendering clusters

**5. VECTOR Data Type: Which Oracle data type is designed for storing embeddings?**
- **✅ VECTOR**
- CLOB
- JSON
- XMLTYPE

_Nota: no se capturó la foto de la pregunta 2 del quiz._

---

📖 Detalle completo con slides y código: `parte6-agentic-ai-for-oracle-ai-database.md`
