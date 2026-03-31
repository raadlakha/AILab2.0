# 02 — Conversational Topics

> **Source:** [ServiceNow Zurich Documentation — Natural Language Understanding](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/nlu-service/nlu-landing.html)

## What It Is

Conversational Topics are the intent-detection layer inside Now Assist for Virtual Agent (NAVA). When a user sends a message, the Natural Language Understanding (NLU) engine classifies their intent and maps it to a topic. A topic configured as **Now Assist** type acts as the bridge between what the user said and which **AI agent** should handle it.

Topics in NAVA can be one of two types:

- **Classic Topics** — scripted conversation trees (legacy approach, still supported)
- **Now Assist Topics** — topics that hand off directly to an AI agent using LLM-powered conversation

> **In This Lab:** The **Upload Image** conversational topic is out-of-box and already integrated into the First Responder Agent — no configuration is required. This document provides reference context on what conversational topics are and how they can be applied in broader implementations.

---

## When to Use

- Any time you want NAVA to understand user intent and route to a specific AI agent
- When a specific business scenario needs a dedicated AI agent handler
- When you want to preserve the ability to escalate to a live agent if the AI agent cannot resolve
- When NLU confidence thresholds should control whether the AI agent or a fallback topic handles the request

---

## Customer Value

| Business Outcome | How Conversational Topics Deliver It |
| --- | --- |
| Accurate intent routing | NLU maps natural language to the right AI agent handler — no mis-routes |
| Reduced configuration overhead | One topic handles all variations of a user's intent |
| Seamless AI agent handoff | User experience is uninterrupted — same chat window, same conversation |
| Measurable deflection | Topic-level analytics show how many issues were resolved vs. escalated |
| No rigid scripting | Now Assist topic type delegates all conversation handling to the AI agent |

---

## How It Works — Architecture

```text
User message
        │
        ▼
  NLU Engine
  (classifies intent → matched topic)
  Confidence score > threshold → match
        │
        ▼
  Conversational Topic
  (Type: Now Assist)
        │
        ▼
  AI Agent (handles the conversation)
```

---

## Reference

- [ServiceNow Zurich — Natural Language Understanding](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/nlu-service/nlu-landing.html)

---

## Next Step

Continue to [03 — Knowledge Graph](03-knowledge-graph.md) to configure the semantic data layer that AI agents use to find resolution steps and personalise responses.
