# Agentic AI Foundations — Oracle

Material de preparación para la certificación **Agentic AI Foundations**, cursada en las oficinas de Oracle.

## 📖 Para repasar antes del examen

👉 **[`notas/resumen-parte1.md`](notas/resumen-parte1.md)** — resumen de repaso rápido, con preguntas tipo examen y respuestas marcadas.

Esta es la referencia principal del repo. El resto del contenido (apuntes completos, agenda, recursos) es soporte/detalle.

**¿Quién dictó qué, y cuándo?**

| Parte | Horario | Docente | Temas | Notas |
|---|---|---|---|---|
| Parte 1 | 9:30 - 10:00 | Florencia Díaz | Introduction to AI Agents | [`parte1-introduction-to-ai-agents.md`](notas/parte1-introduction-to-ai-agents.md) |
| Parte 2 | 10:00 - 10:30 | Florencia Díaz | LangChain for AI Agents | [`parte2-langchain-for-ai-agents.md`](notas/parte2-langchain-for-ai-agents.md) |
| Parte 3 | 10:30 - en curso | Florencia Díaz | Introduction to MCP | [`parte3-introduction-to-mcp.md`](notas/parte3-introduction-to-mcp.md) |

_Nota: horarios redondeados a :00/:30 en base al horario real de la clase (cada corte coincide con el quiz de práctica al final de cada tema). El resto de las partes del día (ver `notas/agenda.md`) todavía no tienen apuntes cargados en este repo._

## Estructura

```
notas/
├── resumen-parte1.md                      # 📖 Resumen de repaso rápido (con quiz) — EMPEZAR ACÁ
├── parte1-introduction-to-ai-agents.md    # Notas completas: Introduction to AI Agents
├── parte2-langchain-for-ai-agents.md      # Notas completas: LangChain for AI Agents
├── parte3-introduction-to-mcp.md          # Notas completas: Introduction to MCP
└── agenda.md                              # Agenda del día de certificación

recursos/
└── material-estudio-externo.md            # Referencias a material de estudio externo
```

- **`notas/`** — Apuntes tomados en vivo a partir de las slides de la capacitación, uno por parte/tema, con ejemplos de código y preguntas de práctica tipo examen al final de cada uno.
- **`recursos/`** — Enlaces y referencias a material externo (no se usa directamente en este repo, solo se cita desde las notas si hace falta).

## Contenido cubierto

### Parte 1 — Introduction to AI Agents

Componentes de un Agent (LLM, Tools, Loop), Agent Execution Loop, reasoning frameworks (CoT, ReAct, ToT), Agent Frameworks, Safety & Guardrails.

### Parte 2 — LangChain for AI Agents

Models, Prompt Templates, Chains (LCEL), Tools, y un walkthrough paso a paso de qué pasa internamente al llamar `agent.invoke()`.

### Parte 3 — Introduction to MCP

Qué es el Model Context Protocol, arquitectura, Core Primitives (Tools/Resources/Prompts), connection lifecycle, JSON-RPC 2.0, transport mechanisms, y cómo sumar un MCP Server a un Agent.

## Notas

Las fotos originales de las slides (carpeta `fotos/`) no se versionan — están en `.gitignore`.
