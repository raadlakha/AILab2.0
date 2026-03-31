# 03 — Knowledge Graph

> **Source:** [ServiceNow Zurich Documentation — Knowledge Graph](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/knowledge-graph/knowledge-graph-landing.html)

## What It Is

The **Knowledge Graph** application uses the structured and unstructured data from ServiceNow records, knowledge bases, and external sources to enhance the performance of Now Assist Virtual Agent, AI agents, and generative AI skills.

As described in the Zurich docs:

> *"ServiceNow Knowledge Graph application enhances the Now Platform by creating a semantic layer that connects data, relationships, and context across the enterprise. It structures information as a graph of entities and connections, bringing context and meaning to the available raw data. By leveraging AI, it powers personalized and intelligent experiences across the Now Assist ecosystem including NAVA, AI Agents, and AI Search to deliver more relevant insights and automation."*

**In This Lab:** The out-of-box **User Graph** is used by the **Requestor AI Agent** to look up a user's full name based on their login. That full name is then written into the incident record as part of the requestor identification flow. No custom Knowledge Graph configuration is required. The out-of-box **User Graph** is used by the **Requestor AI Agent** to look up a user's full name based on their login. That full name is then written into the incident record as part of the requestor identification flow. No custom Knowledge Graph configuration is required.

---

## When to Use

- When AI agents need to look up user attributes (full name, department, manager, location) based on a login or sys_id
- When slot-filling should pre-populate incident or request fields using known user context
- When you want AI agents to query structured relationships across ServiceNow records without additional API calls
- When your organisation has Knowledge Base articles and you want AI agents to search them via natural language queries

---

## Customer Value

| Business Outcome | How Knowledge Graph Delivers It |
| --- | --- |
| Accurate caller identification | Agent resolves full name from login automatically — no manual lookup |
| Fewer questions asked of the user | Known user data pre-populates incident fields |
| Personalised AI responses | User context (role, department, location) shapes AI agent behaviour |
| Smarter AI agents | Agents retrieve structured facts without additional API calls |
| Continuous improvement | Schema grows over time to capture new entity relationships |

---

## How It Works — In This Lab

```text
User initiates conversation in NAVA
        │
        ▼
  Requestor AI Agent receives session
        │
        ▼
  Agent queries OOTB User Graph
  (input: user login from session context)
        │
        ▼
  User Graph returns: full name
        │
        ▼
  Agent writes caller full name → Incident record
```

The **User Graph** is a prebuilt Knowledge Graph schema that models `sys_user` entities and their attributes. The Requestor AI Agent uses it as a tool to resolve the caller's identity at the start of the conversation, then populates the incident's caller field accordingly.

---

## Key Components

- **Knowledge Graph Designer** — no-code UI for `kg_admin` users to create and manage schemas
- **User Graph (OOTB)** — prebuilt schema over `sys_user`; exposes user attributes (full name, email, department, manager) queryable by login or sys_id
- **Nodes** — entities (e.g., `sys_user`, `incident`, `cmdb_ci`, `kb_article`)
- **Edges** — relationships between entities (e.g., User *reported* Incident, User *belongs to* Department)
- **Prebuilt integrations** — out-of-the-box connections to NAVA, AI agents, Now Assist Panel

**Knowledge Graph user roles:**

| User role | KG Functionality | Description |
| --- | --- | --- |
| `kg_admin` | Knowledge Graph Designer | Create and manage Knowledge Graph schemas |
| Requester | Prebuilt integration with NAVA and Agentic AI | Personalised answers, natural language queries, slot-filling |
| Fulfiller | Prebuilt integration with Now Assist Panel and Agentic AI | Personalised answers in fulfilment workspace |

---

## Reference

- [ServiceNow Zurich — Knowledge Graph](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/knowledge-graph/knowledge-graph-landing.html)
- [Exploring Knowledge Graph](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/knowledge-graph/exploring-knowledge-graph.html)
- [Configuring Knowledge Graph](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/knowledge-graph/configuring-knowledge-graph.html)

---

## Next Step

Continue to [04 — Now Assist in Document Intelligence](04-now-assist-document-intelligence.md) to configure AI-powered extraction of error codes from files users upload during the conversation.
