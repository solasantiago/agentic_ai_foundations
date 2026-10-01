# Agentic AI Foundations — Oracle

Material de preparación para la certificación **Agentic AI Foundations**, cursada en las oficinas de Oracle.

## 📖 Para repasar antes del examen

👉 **[`notas/resumen-parte1.md`](notas/resumen-parte1.md)** — resumen de repaso rápido, con preguntas tipo examen y respuestas marcadas.

Esta es la referencia principal del repo. El resto del contenido (apuntes completos, agenda, recursos) es soporte/detalle.

## Estructura

```
notas/
├── resumen-parte1.md                    # 📖 Resumen de repaso rápido (con quiz) — EMPEZAR ACÁ
├── parte1-introduction-to-ai-agents.md  # Notas completas: AI Agents + LangChain + MCP
└── agenda.md                            # Agenda del día de certificación

recursos/
└── material-estudio-externo.md          # Referencias a material de estudio externo
```

- **`notas/`** — Apuntes tomados en vivo a partir de las slides de la capacitación, organizados por lesson/tema, con ejemplos de código y preguntas de práctica tipo examen.
- **`recursos/`** — Enlaces y referencias a material externo (no se usa directamente en este repo, solo se cita desde las notas si hace falta).

## Contenido cubierto

- **Introduction to AI Agents** — componentes de un Agent (LLM, Tools, Loop), Agent Execution Loop, reasoning frameworks (CoT, ReAct, ToT), Agent Frameworks, Safety & Guardrails.
- **LangChain for AI Agents** — Models, Prompt Templates, Chains (LCEL), Tools, y un walkthrough paso a paso de qué pasa internamente al llamar `agent.invoke()`.
- **Introduction to MCP** — qué es el Model Context Protocol, arquitectura, Core Primitives (Tools/Resources/Prompts), connection lifecycle, JSON-RPC 2.0, transport mechanisms, y cómo sumar un MCP Server a un Agent.

## Notas

Las fotos originales de las slides (carpeta `fotos/`) no se versionan — están en `.gitignore`.
