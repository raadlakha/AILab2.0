# ServiceNow Native AI Enablement Lab

> **Release:** Zurich | **Source:** [ServiceNow Zurich Documentation — Enable AI experiences](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/ai-products.html)

An enablement lab for Solution Consultants to learn, build, and demonstrate AI solutions natively on ServiceNow — using the Zurich release capabilities.

## Lab Scenario

> A user reports that a server is inaccessible. Through this lab, you will build the complete end-to-end AI-powered resolution pipeline — from the moment the user types their first message in Now Assist Virtual Agent (NAVA), all the way through to an automatically generated resolution plan written back to the Incident record.

---

## Use Case Summary

For a complete step-by-step walkthrough of the full pipeline — including all conditional paths — see **[00 — Use Case Summary](docs/00-use-case-summary.md)**.

---

## Capabilities Covered

| # | Capability | What It Enables |
|---|-----------|-----------------|
| 01 | [Now Assist for Virtual Agent (NAVA)](docs/01-now-assist-virtual-agent.md) | Conversational AI entry point for users |
| 02 | [Conversational Topics](docs/02-conversational-topics.md) | NLU-based intent routing to AI agents |
| 03 | [Knowledge Graph](docs/03-knowledge-graph.md) | Semantic data layer powering AI agent search and slot-filling |
| 04 | [Now Assist in Document Intelligence](docs/04-document-intelligence.md) | GenAI extraction of error codes from uploaded documents |
| 05 | [First Responder AI Agent (Chat)](docs/05-requestor-ai-agent.md) | Chat AI agent that gathers context and creates enriched Incidents |
| 06 | [Resolution Pathfinder (Agentic Workflow)](docs/06-fulfiller-ai-agent.md) | Background AI agent + workflow that researches and resolves Incidents |
| 07 | [Flow Action Tool — Extract Incident Details](docs/07-flow-action-tool.md) | Flow Designer action that structures incident data for AI agents |
| 08 | [Resolution Finder — Internal Data](docs/08-resolution-finder-internal-data.md) | Now Assist Skill using RAG over Knowledge Graph + resolved incidents |
| 09 | [Web Search Query Generator](docs/09-web-search-query-generator.md) | Now Assist Skill that generates privacy-safe internet search queries |
| 10 | [Web Search Tool](docs/10-web-search-tool.md) | Executes external web searches as an AI agent tool |
| 11 | [Elastic MCP Server Integration](docs/11-elastic-mcp-server.md) | MCP Client connecting AI agents to external Elastic log data |
| 12 | [Observability and Action Agent (A2A)](docs/12-observability-action-agent.md) | External AI Agent executing remediation via Azure AI Foundry over A2A protocol |

---

## Solution Architecture

```
User (NAVA Chat — Service Portal)
        │
        ▼
┌──────────────────────────────────┐
│  Now Assist for Virtual Agent    │  ← Capability 01
│  + Conversational Topic (NLU)    │  ← Capability 02
└─────────────┬────────────────────┘
              │
              ▼
┌──────────────────────────────────┐
│  First Responder AI Agent        │  ← Capability 05
│  (AI Agent type — Chat)          │
│  Tools:                          │
│   • Knowledge Graph search       │  ← Capability 03
│   • Now Assist Document Intel    │  ← Capability 04
│   • Flow Action (Raise INC)      │  ← Capability 07
└─────────────┬────────────────────┘
              │ Incident Created
              │ state=In Progress, contact_type=chat, u_extracted_error_code≠empty
              ▼
┌──────────────────────────────────┐
│  Resolution Pathfinder           │  ← Capability 06
│  (Agentic Workflow + Chat Agent) │
│  Tools:                          |
    • Flow Action (Retrieve INC)   │  ← Capability 07
│   • ResolutionFinderInternalData │  ← Capability 08
│     (Now Assist Skill — RAG & PI)│
│   • ElasticLogSearch             │  ← Capability 11
│     (MCP Tool)                   │
│   • GenerateWebSearchQuery       │  ← Capability 09
│     (Now Assist Skill)           │
│   • WebSearch                    │  ← Capability 10
└─────────────┬────────────────────┘
              │
              ▼
  Resolution Plan generated to be written to Incident work_notes
              │
              ▼
┌──────────────────────────────────┐
│  Observability and Action Agent  │  ← Capability 12
│  (External type — A2A Protocol)  │
│  AI Agent Fabric →               │
│  Azure AI Foundry Agent          │
│   • Executes remediation steps   │
│   • Returns outcome              │
└─────────────┬────────────────────┘
              │
              ▼
  Remediation result generated to be written to Incident work_notes
```

---

## How to Use This Lab

Each capability file follows the same structure:

- **What It Is** — Concept and role in the solution
- **When to Use** — Trigger conditions and fit
- **Customer Value** — Business outcomes it delivers
- **How to Build** — Configuration steps in ServiceNow
- **Lab Exercise** — Hands-on steps to complete

Work through the capabilities in order — each builds on the previous.

---

## Prerequisites

| Requirement | Detail |
|-------------|--------|
| ServiceNow release | Zurich |
| Licence | Now Assist Pro+ or ITSM Pro+ |
| Plugins activated | Now Assist for ITSM, Virtual Agent, AI Agent Studio (`sn_aia`), MCP Client (`sn_mcp_client`), Now Assist Skill Kit, Now Assist in Document Intelligence |
| Admin roles | `sn_aia.admin` (AI agents), `kg_admin` (Knowledge Graph), `sn_va_designer` (Virtual Agent), `admin` |
| External systems | Elastic instance with MCP-compatible endpoint (for Capability 11) |
| External systems | Azure AI Foundry agent with A2A-compliant Agent Card at `/.well-known/agent.json` (for Capability 12) |
| Zurich minimum | Patch 6 Hot Fix 2 (required for External AI Agent / A2A protocol — Capability 12) |

---

## Terminology — Zurich Edition

| Zurich Term | What It Means |
|-------------|---------------|
| AI Agent Studio | Where you create and manage AI agents |
| Agent types: Chat / Voice / External | Chat = user-facing via NAVA; Voice = telephony; External = third-party AI platforms |
| Agentic Workflow | Defines how and when AI agents are triggered; orchestrates multi-agent flows |
| Now Assist Skill | Custom GenAI skill built with the Now Assist Skill Kit |
| MCP Client | ServiceNow consuming tools from external MCP servers |
| MCP Server Console | External applications consuming ServiceNow tools (not used in this lab) |
| Knowledge Graph | Semantic data layer connecting entities, relationships, and context across the platform |
| Role: `sn_aia.admin` | Required to create and manage AI agents in AI Agent Studio |
| Agent2Agent (A2A) Protocol | Open standard for agent-to-agent interoperability; used to connect ServiceNow to external AI platforms |
| AI Agent Fabric | ServiceNow framework enabling integration with external AI agents via A2A protocol |
| Agent Card | JSON document at `/.well-known/agent.json` that external agents expose for ServiceNow to discover |
