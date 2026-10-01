# Agentic AI Foundations — Oracle

Material de preparación para la certificación **Agentic AI Foundations**, cursada en las oficinas de Oracle.

## 📖 Para repasar antes del examen

Esta es la referencia principal del repo. Cada parte tiene su propio resumen de repaso rápido (📖, con preguntas tipo examen y respuestas marcadas) y sus notas completas (con slides y código). El resto del contenido (agenda, recursos) es soporte/detalle.

| Horario | Docente | Temas | Resumen | Notas completas |
|---|---|---|---|---|
| 9:30 - 10:00 | Florencia Díaz | Introduction to AI Agents | [📖 resumen](notas/resumen-parte1.md) | [notas](notas/parte1-introduction-to-ai-agents.md) |
| 10:00 - 10:30 | Florencia Díaz | LangChain for AI Agents | [📖 resumen](notas/resumen-parte2.md) | [notas](notas/parte2-langchain-for-ai-agents.md) |
| 10:30 - 11:30 | Johannes Segura Campos | Introduction to MCP | [📖 resumen](notas/resumen-parte3.md) | [notas](notas/parte3-introduction-to-mcp.md) |
| 11:30 - 12:00 | Johannes Segura Campos | OpenAI Responses API and Agents SDK | [📖 resumen](notas/resumen-parte4.md) | [notas](notas/parte4-openai-responses-api-and-agents-sdk.md) |
| 12:00 - en curso | A determinar | Por confirmar | _pendiente_ | _pendiente_ |

_Nota: horarios redondeados a :00/:30 en base al horario real de la clase (cada corte coincide con el quiz de práctica al final de cada tema, salvo la última fila que sigue en curso). El resto de las partes del día (ver `notas/agenda.md`) todavía no tienen apuntes cargados en este repo._

## Estructura

```
notas/
├── resumen-parte1.md                      # 📖 Resumen de repaso — Introduction to AI Agents
├── resumen-parte2.md                      # 📖 Resumen de repaso — LangChain for AI Agents
├── resumen-parte3.md                      # 📖 Resumen de repaso — Introduction to MCP
├── resumen-parte4.md                      # 📖 Resumen de repaso — OpenAI Responses API and Agents SDK
├── parte1-introduction-to-ai-agents.md    # Notas completas: Introduction to AI Agents
├── parte2-langchain-for-ai-agents.md      # Notas completas: LangChain for AI Agents
├── parte3-introduction-to-mcp.md          # Notas completas: Introduction to MCP
├── parte4-openai-responses-api-and-agents-sdk.md  # Notas completas: OpenAI Responses API and Agents SDK
└── agenda.md                              # Agenda del día de certificación

recursos/
└── material-estudio-externo.md            # Referencias a material de estudio externo
```

- **`notas/`** — Apuntes tomados en vivo a partir de las slides de la capacitación, uno por parte/tema, con ejemplos de código y preguntas de práctica tipo examen al final de cada uno. Cada parte tiene su resumen de repaso (`resumen-parteN.md`) y sus notas completas (`parteN-*.md`).
- **`recursos/`** — Enlaces y referencias a material externo (no se usa directamente en este repo, solo se cita desde las notas si hace falta).

## Contenido cubierto

### Parte 1 — Introduction to AI Agents

Componentes de un Agent (LLM, Tools, Loop), Agent Execution Loop, reasoning frameworks (CoT, ReAct, ToT), Agent Frameworks, Safety & Guardrails.

### Parte 2 — LangChain for AI Agents

Models, Prompt Templates, Chains (LCEL), Tools, y un walkthrough paso a paso de qué pasa internamente al llamar `agent.invoke()`.

### Parte 3 — Introduction to MCP

Qué es el Model Context Protocol, arquitectura, Core Primitives (Tools/Resources/Prompts), connection lifecycle, JSON-RPC 2.0, transport mechanisms, y cómo sumar un MCP Server a un Agent.

### Parte 4 — OpenAI Responses API and Agents SDK

Qué es un AI Agent, el OpenAI Agent Stack (Application → Agents SDK → Responses API → Models), y cómo elegir entre Responses API y Agents SDK según la complejidad del caso de uso.

## Notas

Las fotos originales de las slides (carpeta `fotos/`) no se versionan — están en `.gitignore`.
