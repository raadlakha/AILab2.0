# 00 — Use Case Summary: NAVA Agentic Incident Resolution

> **Use case:** Veritas Resolution Agentic Workflow
> **Team:** ServiceNow · Global Intelligent Automation SC Team · India
> **Release:** Zurich | **Document version:** v2.1 · March 2026

---

## Executive Summary

NAVA (Now Assist Virtual Agent) with Agentic Incident Resolution is an end-to-end, multi-agent pipeline built on ServiceNow that handles IT incidents autonomously — from first user message to resolution — with no mandatory human intervention.

When an issue cannot be deflected in chat, the platform creates an incident, extracts structured data from user-uploaded images via Now Assist Document Intelligence, and triggers the **Veritas Resolution Agentic Workflow**. The workflow searches multiple knowledge sources and — on finding a resolution — dispatches a remediation plan to Azure AI Foundry via the A2A (Agent-to-Agent) protocol for execution.

> **Key outcome:** Zero-touch incident resolution — user message → autonomous diagnosis → external remediation → incident closed, with a complete audit trail written at every step.

---

## Business Context & Problem Statement

Traditional ITSM flows suffer from three compounding inefficiencies:

**High L1 ticket volume** for errors that have known resolutions — solved by structured KB deflection via the L1 Requestor Agent before any ticket is created.

**Slow triage** because agents lack structured error data — solved by Now Assist Document Intelligence auto-extracting fields from uploaded images, including the key `u_extracted_error_code` field that gates the entire agentic workflow.

**Resolution bottleneck at L2** because search is manual and cross-system — solved by the Veritas agentic search cascade (KB → Elastic logs → internet) and A2A-driven automated remediation via Azure AI Foundry.

This use case is designed for **regulated-industry accounts** (banking, healthcare, infrastructure) where ticket volume is high, resolution SLAs are tight, and there is a need to demonstrate autonomous AI value without removing human oversight from critical paths.

---

## Architecture Overview

The solution is structured across three swimlanes, each owned by a distinct set of ServiceNow components and agents:

| Swimlane | Components & Agents |
|----------|-------------------|
| **Requestor Flow** | Now Assist Virtual Agent (NAVA), L1 / Requestor Agent, Knowledge Graph + User Graph, Troubleshooting Guide (file upload tool), Conversation Topic, Now Assist Document Intelligence |
| **Fulfiller Flow** (Veritas Workflow) | Resolution Finder Agent, GenerateSearchQueryAgainstAISearch Skill, FindSimilarIncidents Skill (Predictive Intelligence), RetrieveRelevantKBContent Retriever, Assess if solution exists Skill Prompt, Privacy-safe Web Search |
| **External Integration** (Veritas Workflow) | Observability & Action Agent, A2A Protocol, Azure AI Foundry (remote execution agent) |

### Full Pipeline

```
User Message via NAVA (Chat)
        │
        ▼
Phase 1 — Requestor Flow: Deflection & Incident Creation
        │
        ├── DEFLECTED → KB answer found in chat → conversation ends, no ticket
        │
        └── NOT DEFLECTED → Incident created with Doc Intel error extraction
                │
                ▼
        Phase 2 — Fulfiller Flow: Veritas Resolution Finder Agent
                │
                ├── Path A: KB + Predictive Intelligence → assess → resolution plan found
                │           └── → Phase 3 (External Integration)
                │
                └── Path B: No internal resolution → privacy-safe web search
                            ├── Web search resolves → resolution plan written → Phase 3
                            └── Web search fails → escalate to L2 (Phase 3 NOT triggered)
                                        │
                                        ▼
                            Phase 3 — External Integration: A2A → Azure AI Foundry
                                        │
                                        ├── Path 3A: Remediation succeeds → incident closed
                                        └── Path 3B: Fail/partial → full log → L2 handoff
```

---

## End-to-End Flow — Phase by Phase

### Phase 1 — Requestor Flow: Deflection & Incident Creation

> **Lab doc:** [01 — Now Assist Virtual Agent](01-now-assist-virtual-agent.md) | [02 — L1 First Responder Agent](02-L1-agent-first-responder-analyst-agent.md) | [04 — Now Assist Document Intelligence](04-now-assist-document-intelligence.md)

**Step 1 — User sends a message to NAVA via chat.**
The Now Assist Virtual Agent receives the message and stamps `contact_type = chat` on the session. The Now Assist Panel powers the LLM-backed conversation layer.

**Step 2 — The L1 / Requestor Agent is triggered.**
Under the agent, the Knowledge Graph queries the User Graph to identify and contextualise the user — role, device, past tickets — enabling personalised troubleshooting and pre-populated incident fields.

**Step 3 — Troubleshooting Guide is presented.**
The agent presents a Troubleshooting Guide sourced via the file upload tool, delivered through the Conversation Topic interface.

> **Decision point:** If the troubleshooting steps resolve the issue → the conversation is deflected in chat with no ticket created (optimal outcome — no incident volume generated). If unhelpful, the flow continues.

**Step 4 — User uploads error screenshots and device detail images.**
The agent requests images via the Conversation Topic (upload image to Virtual Agent).

**Step 5 — Incident record created with state = New.**
Uploaded images are attached to the incident record.

**Step 6 — Now Assist Document Intelligence auto-triggers on each attached image.**
Doc Intel extracts structured fields per the configured use case — no manual data entry. The critical output is `u_extracted_error_code`, populated on the incident record and used as the gate for the downstream agentic workflow trigger.

---

### Phase 2 — Fulfiller Flow: Veritas Resolution Finder Agent

> **Lab doc:** [06 — Fulfiller AI Agent](06-fulfiller-ai-agent.md) | [03 — NASK CreateOptimalSearchQuery](03-now-assist-skill-kit.md) | [03 — NASK GenerateWebSearchQns](03-now-assist-skill-kit-web-search.md) | [08 — Agentic Workflow](08-wrapping-in-agentic-workflow.md)

**Trigger conditions** (all three must be true simultaneously):

| Field | Required value |
|-------|---------------|
| `state` | In Progress |
| `contact_type` (Channel) | Chat |
| `u_extracted_error_code` | is not empty |

The trigger fires on the **`incident extend`** table — because `u_extracted_error_code` is a custom field that only exists there (populated by NADI).

---

**Path A — Step 1 (parallel fire from workflow Start node):**

Two Now Assist skills fire in parallel:

1. **GenerateSearchQueryAgainstAISearch Skill** — constructs an optimised AI search query from the error code and incident context. This skill calls `CreateOptimalSearchQuery` as an upstream tool via skill chaining.
2. **FindSimilarIncidents Skill (Predictive Intelligence)** — searches for historically similar incidents using ML-based similarity matching.

**Path A — Step 2:**
The optimised AI search query is passed to the **RetrieveRelevantKBContent Retriever** skill, which fetches ranked relevant results from the ServiceNow Knowledge Base using AI Search RAG. Results from FindSimilarIncidents also merge at this stage.

**Path A — Step 3:**
The **Assess if solution exists Skill Prompt** evaluates the combined retrieval output (KB content + similar incidents) and determines whether a viable resolution exists. This is a reasoning step — the agent does not proceed to remediation unless a solution is confirmed.

**Path A — Result:**
- If the Assess skill confirms a solution exists → the agent builds a structured Resolution Plan and writes it to the incident work notes → flow continues to **Phase 3**.
- If no solution is confirmed → flow falls through to **Path B**.

---

**Path B — No internal resolution found:**
If no result is found from KB or similar incidents, a **privacy-safe web search** is triggered. PII and internal identifiers are stripped before the query is sent externally (enforced by the `GenerateWebSearchQnsForResolutionPlan` NASK skill).

> **Lab doc:** [03 — NASK Web Search Skill](03-now-assist-skill-kit-web-search.md) | [09 — Web Search Query Generator](09-web-search-query-generator.md) | [10 — Web Search Tool](10-web-search-tool.md)

**Path B — Fallback:**
If the web search also yields no actionable result → the incident is escalated to L2 manual pickup. The external agent (Phase 3) is **NOT triggered** — user validation is required before any automated remediation executes on an unresolved search path.

---

### Phase 3 — External Integration: Observability & Action Agent via A2A

> **Lab doc:** [07 — External Agent Integration (A2A OAuth + Registration)](07-external-agent-integration.md) | [12 — Observability and Action Agent](12-observability-action-agent.md)

> **Trigger:** Phase 3 fires only when Path A of the Fulfiller Flow produces a resolution plan. It does NOT trigger on Path B fallback.

**Step 1 — Observability & Action Agent constructs remediation action set.**
The agent (running in ServiceNow as the primary A2A orchestrator) receives the resolution plan and structures it into a remediation action set.

**Step 2 — Remediation plan dispatched to Azure AI Foundry via A2A.**
The plan is serialised and sent to Azure AI Foundry via the Agent-to-Agent protocol. ServiceNow remains the orchestrator; Azure AI Foundry acts as the remote execution agent. Authentication uses OAuth 2.0 Client Credentials via the `FoundryConnectCreds` connection alias.

**Step 3 — Azure AI Foundry executes remediation actions.**
The ObsAgent executes prescribed actions in the target environment (service restart, config patch, queue flush, etc.). Execution telemetry streams back to ServiceNow.

**Path 3A — Success:**
Incident auto-resolved, resolution notes and execution outcome written to the incident record, ticket closed. No human intervention required.

**Path 3B — Fail / Partial:**
Full execution log, attempted steps, and failure reason written to incident work notes. Incident escalated to L2 with complete context for investigation.

---

## Key ServiceNow Components & Capabilities

| Component | Role in this use case | Lab doc |
|-----------|----------------------|---------|
| **Now Assist Virtual Agent (NAVA)** | Front-door for all user interactions. Handles multi-turn conversation, intent routing, troubleshooting delivery, and image upload orchestration. | [01](01-now-assist-virtual-agent.md) |
| **L1 / Requestor AI Agent** | Orchestrates the requestor-side flow: KG lookup, troubleshooting, user prompting for images, and handoff to incident creation. | [02](02-L1-agent-first-responder-analyst-agent.md) |
| **Knowledge Graph (User Graph)** | Identifies and contextualises the user at session start, enabling personalised troubleshooting and pre-populated incident fields. | [02](02-L1-agent-first-responder-analyst-agent.md) |
| **Troubleshooting Guide (file upload tool)** | Surfaces step-by-step resolution guides via the file upload tool, delivered through the Conversation Topic interface. | [01](01-now-assist-virtual-agent.md) |
| **Now Assist Document Intelligence** | Auto-triggers on images attached to an incident. Extracts structured fields (including `u_extracted_error_code`) per use case config — no manual data entry. | [04](04-now-assist-document-intelligence.md) |
| **Veritas Resolution Agentic Workflow** | The named agentic workflow spanning the Fulfiller and External Integration flows. Contains search cascade logic, agent bindings, trigger conditions, and A2A dispatch. | [08](08-wrapping-in-agentic-workflow.md) |
| **GenerateSearchQueryAgainstAISearch Skill** | Generates an optimised AI search query from the raw error code + incident context. Runs in parallel with FindSimilarIncidents from the workflow Start node. | [03](03-now-assist-skill-kit.md) |
| **GenerateWebSearchQnsForResolutionPlan Skill** | Generates privacy-safe web search questions when internal KB and logs fail. Uses skill chaining — calls `CreateOptimalSearchQuery` as an upstream tool. | [03-web](03-now-assist-skill-kit-web-search.md) |
| **FindSimilarIncidents + RetrieveRelevantKBContent** | FindSimilarIncidents uses Predictive Intelligence ML to surface historically similar incidents. RetrieveRelevantKBContent fetches ranked KB results via AI Search RAG. Both feed the Assess skill. | [06](06-fulfiller-ai-agent.md) |
| **Elastic MCP Server** | Live log querying for root cause analysis when KB search is inconclusive. ServiceNow acts as MCP client. | [05](05-mcp-elastic-integration.md) / [11](11-elastic-mcp-server.md) |
| **Observability & Action Agent** | Builds the remediation plan from the Phase 2 resolution and dispatches it to Azure AI Foundry via A2A. External agent type registered in AI Agent Studio. | [12](12-observability-action-agent.md) |
| **A2A Protocol** | Agent-to-Agent protocol enabling cross-platform agent communication. ServiceNow acts as primary orchestrator; Azure AI Foundry as remote executor. OAuth 2.0 Client Credentials via `FoundryConnectCreds`. | [07](07-external-agent-integration.md) |
| **Azure AI Foundry (ObsAgent)** | Remote execution agent that carries out remediation actions in the target environment and streams telemetry back to ServiceNow. Hosted on Azure Container Apps. | [07](07-external-agent-integration.md) / [12](12-observability-action-agent.md) |

---

## Decision Path Summary

| Phase | Condition | Path | Outcome |
|-------|-----------|------|---------|
| Phase 1 — Requestor | KB answer found in chat | **Deflected** | Conversation ends. No incident created. |
| Phase 1 — Requestor | No KB answer | **1B** | Incident created. Pipeline continues. |
| Phase 2 — Fulfiller | KB + PI resolves, solution confirmed | **Path A** | Resolution plan written. Phase 3 triggered. |
| Phase 2 — Fulfiller | Internal fails, web search resolves | **Path B** | Privacy-safe web resolution plan written. Phase 3 triggered. |
| Phase 2 — Fulfiller | All sources fail | **Path B Fallback** | Manual note written. Phase 3 NOT triggered. L2 pickup. |
| Phase 2 — Fulfiller | `u_extracted_error_code` empty | *(No trigger)* | Agentic Workflow does not fire. Manual queue. |
| Phase 3 — External | Azure AI Foundry succeeds | **Path 3A** | Result written. Incident auto-resolved. |
| Phase 3 — External | Azure AI Foundry fails / partial | **Path 3B** | Partial result written. Full context to L2. |

---

## Value Outcomes

| Outcome | How the architecture delivers it |
|---------|----------------------------------|
| **L1 ticket deflection** | Troubleshooting guide delivered in chat before any incident is created. Deflected conversations produce no ticket, reducing incident volume. |
| **Structured incident data without manual entry** | Now Assist Doc Intel extracts all key fields from user images automatically — no form filling, no agent interpretation required. |
| **Faster resolution via optimised search** | GenerateSearchQueryAgainstAISearch + FindSimilarIncidents (Predictive Intelligence) + RetrieveRelevantKBContent + Assess skill cascade finds the best available resolution — KB content and similar incidents evaluated together before escalating. |
| **Zero-touch remediation for known paths** | Path A + Phase 3 delivers end-to-end resolution with no human involvement — from error code to closed incident. |
| **Context-rich L2 handoffs** | Every escalation path (Path B Fallback, Path 3B) writes a complete trail — attempted steps, search results, execution logs — so L2 engineers never start from zero. |
| **Cross-platform AI orchestration** | A2A protocol integration with Azure AI Foundry demonstrates ServiceNow as the orchestration layer for multi-cloud AI — not a walled garden. |

---

## Positioning & SC Notes

This use case is positioned for the following customer conversations:

**"How do I connect my existing Azure AI investment to ServiceNow?"**
The A2A integration with Azure AI Foundry is a direct answer: ServiceNow orchestrates, Azure executes, and the result is written back to the system of record.

**"Can ServiceNow actually fix things, or just raise tickets?"**
Path A + Phase 3 demonstrates end-to-end autonomous remediation with a full audit trail — from chat message to closed incident with execution log.

**"How does your AI handle cases it can't solve?"**
Path B and Path 3B show deliberate, context-rich escalation — not silent failure. Every escalation writes a complete diagnostic trail before the L2 handoff.

**"Is there human oversight?"**
Path B explicitly requires user validation before external execution. Path 3B writes a complete log before L2 handoff. The system is designed for regulated environments — humans steer, agents execute within defined boundaries.

> **Relevant product capabilities to highlight:** Now Assist Virtual Agent (Agentic Reasoning) · AI Agent Studio · Now Assist Document Intelligence · Agentic Workflow · A2A Protocol · GenerateSearchQueryAgainstAISearch Skill · FindSimilarIncidents (Predictive Intelligence) · RetrieveRelevantKBContent Retriever · Knowledge Graph

---

## Key Technical Constraints

| Constraint | Detail |
|-----------|--------|
| Agentic Workflow trigger gate | `state = In Progress` AND `contact_type = chat` AND `u_extracted_error_code` is not empty — all three required |
| Trigger table | `incident extend` (`x_nava_agentic_lab_incident_extend`) — not base `incident` table |
| A2A / External Agent minimum release | ServiceNow Zurich Patch 4 |
| Privacy — web search | Internal hostnames, IPs, user names stripped before internet query (enforced by `GenerateWebSearchQnsForResolutionPlan` NASK skill prompt) |
| Phase 3 trigger condition | Only fires when Path A produces a confirmed resolution plan — does NOT fire on Path B fallback |
| Communication mode — ObsAgent | Synchronous (remediations < 30s); Asynchronous for long-running operations via callback registry |
| OAuth — Azure AI Foundry | Client Credentials flow, `Send Credentials = In Request Body`, scope = `api://<client-id>/.default` |
| Auth — Elastic MCP | API Key or OAuth 2.0 managed in AI Agent Studio |
| ACL — External Agent | Required — without `itil` role ACL, Observability & Action Agent cannot be invoked from workflow |
| Data access — Agentic Workflow | Dynamic user → Assigned to [task] → approved role: `itil` |

---

## Lab Documentation Index

| # | Document | Phase | What it covers |
|---|----------|-------|---------------|
| 00 | **Use Case Summary** (this doc) | All | End-to-end architecture, flow, and SC positioning |
| 01 | [Now Assist Virtual Agent (NAVA)](01-now-assist-virtual-agent.md) | 1 | NAVA setup, conversation topic, chat channel |
| 02 | [L1 First Responder Agent](02-L1-agent-first-responder-analyst-agent.md) | 1 | L1 / Requestor Agent — KG lookup, troubleshooting, incident creation |
| 03 | [NASK — CreateOptimalSearchQuery](03-now-assist-skill-kit.md) | 2 | Now Assist Skill Kit — building the upstream query generation skill |
| 03p2 | [NASK — Part 2](03-now-assist-skill-kit-part2.md) | 2 | NASK extended configuration |
| 03ws | [NASK — GenerateWebSearchQnsForResolutionPlan](03-now-assist-skill-kit-web-search.md) | 2 | Web search skill chaining — skill-as-tool pattern, NASK web search tool |
| 04 | [Now Assist Document Intelligence](04-now-assist-document-intelligence.md) | 1 | NADI setup — error code extraction from uploaded images |
| 05 | [MCP Elastic Integration](05-mcp-elastic-integration.md) | 2 | MCP client configuration for Elastic log search |
| 06 | [Fulfiller AI Agent](06-fulfiller-ai-agent.md) | 2 | Resolution Finder Agent — tools, system prompt, KB retrieval cascade |
| 07 | [External Agent Integration (A2A)](07-external-agent-integration.md) | 3 | OAuth 2.0 setup + External Agent registration in AI Agent Studio |
| 08 | [Wrapping in Agentic Workflow](08-wrapping-in-agentic-workflow.md) | 2+3 | Veritas Agentic Workflow — LLM description, agents, trigger, channels |
| 09 | [Web Search Query Generator](09-web-search-query-generator.md) | 2 | Privacy-safe query generation skill |
| 10 | [Web Search Tool](10-web-search-tool.md) | 2 | Web Search tool execution |
| 11 | [Elastic MCP Server](11-elastic-mcp-server.md) | 2 | Elastic MCP server setup |
| 12 | [Observability & Action Agent](12-observability-action-agent.md) | 3 | ObsAgent — A2A external agent, remediation execution, Phase 3 paths |

---

*Document prepared by ServiceNow Global Intelligent Automation SC Team · India · March 2026*
