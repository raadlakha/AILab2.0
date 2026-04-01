# 00 — Use Case Summary: NAVA Agentic Incident Resolution

> **Use case:** Veritas Resolution Agentic Workflow
> **Release:** Zurich | **Team:** ServiceNow APJ Intelligent Automation SC Team · March 2026

---

## 1. Executive Summary

**NAVA (Now Assist Virtual Agent) with Agentic Incident Resolution** is an end-to-end, multi-agent pipeline built on ServiceNow that handles IT incidents autonomously — from first user message to resolution — with no mandatory human intervention.

When an issue cannot be deflected in chat, the platform summarises all of the chat interactions and uses it to creates an Incident case, extracts key information from user-uploaded images via Now Assist Document Intelligence, and triggers the **Veritas Resolution Agentic Workflow**. The workflow searches multiple knowledge sources and — on finding a resolution — dispatches a remediation plan to Azure AI Foundry via the **A2A (Agent-to-Agent) protocol** for execution.

> **Key outcome:** Zero-touch incident resolution — user message → autonomous diagnosis → external remediation → incident closed, with a complete audit trail written at every step.

---

## 2. Business Context & Problem Statement

Traditional ITSM flows suffer from three compounding inefficiencies:

**High L1 ticket volume** for errors that have known resolutions — solved by structured KB deflection before any incident is created.

**Slow triage** because agents lack structured error data — solved by Now Assist Doc Intel auto-extracting fields from uploaded images.

**Resolution bottleneck at L2** because search is manual and cross-system — solved by the Veritas agentic search cascade and A2A-driven automated remediation.

This use case is designed for regulated-industry account (banking, healthcare, infrastructure) where ticket volume is high, resolution SLAs are tight, and there is a need to demonstrate autonomous AI value without removing human oversight from critical paths.

---

## 3. Solution Architecture Overview

The solution is structured across three swimlanes, each owned by a distinct set of ServiceNow components and agents.

| Swimlane | Components & Agents |
|----------|-------------------|
| **Requestor flow** | Now Assist Virtual Agent (NAVA), L1 / Requestor Agent, Knowledge Graph + User Graph, Troubleshooting Guide (file upload tool), Conversation Topic, Flow Action (Create Incident Case), Now Assist Document Intelligence |
| **Fulfiller flow (Veritas Workflow)** | ResolutionFinderInternalData Now Assist Custom Skill, CreateOptimalSearchQuery Now Assist Custom Skill, GenerateWebSearchQnsForResolutionPlan Now Assist Custom Skill, Flow Action (Retrieve Incident Case), Web Search tool, Elastic MCP server connectivity (Log entries) |
| **External integration (Veritas Workflow)** | Observability & Action Agent, A2A Protocol, Azure AI Foundry (remote execution agent) |

### Full Pipeline — Decision Tree

```
User Message (NAVA Chat)
        │
        ▼
Phase 1 — Requestor Flow — Deflection & Incident Creation
        │
        ├── KB deflection succeeds → conversation ends, NO ticket created ✅
        │
        └── No deflection → chat conversation is summarised and gathering of additional details → image upload → Incident created → Doc Intel extraction
                │
                ▼
Phase 2 — Fulfiller Flow — Veritas Resolution Finder Agent
        │
        ├── Trigger: state=In Progress AND channel=Chat AND error_code≠empty
        │
        ├── Path A: Internal Knowledge Base + Predictive Intelligence (Similarity) → possible resolution next steps found
                  → Checked against Elastic server's log record entries to validate on root cause analysis → In concurrence with information found internally
        │         → Resolution Plan generated → Phase 3 triggered ✅
        │
        └── Path B: No internal solution → Consider Privacy-safe Web Search to gather more information for resolution
                  → Checked against Elastic server's log record entries to validate on root cause analysis; nothing conclusive found → Privacy-safe Web Search
                  ├── Web search finds actionable next steps for resolution → Resolution Plan generated to be written to Incident work notes for Support Agent (human) to takeover
                  └── Web search fails → Work notes generated to be appended to the Incident case → L2 escalation 🔴
                          │
                          ▼
Phase 3 — External Integration — Observability & Action Agent via A2A
        │  (Triggered only on Path A success — NOT on Path B)
        │
        ├── Path 3A: Azure AI Foundry executes → incident auto-resolved ✅
        └── Path 3B: Partial / failure → full log written → L2 handoff 🔴
```

---

## 4. End-to-End Flow — Phase by Phase

### Phase 1 — Requestor Flow — Deflection & Incident Creation

**Step 1** — User sends a message to NAVA via chat. The Now Assist Virtual Agent receives the message and stamps `contact_type = chat` on the session.

**Step 2** — The L1 / Requestor Agent is triggered. Under the agent, the Knowledge Graph queries the User Graph to identify and contextualise the user (role, device, past tickets). The NLU model evaluates intent (minimum confidence threshold: 0.70).

**Step 3** — The agent presents a Troubleshooting Guide sourced via the file upload tool. Steps are delivered through the Conversation Topic interface.

**Decision point:** If the troubleshooting steps resolve the issue → the conversation is deflected in chat with no ticket created (optimal outcome). If unhelpful, the flow continues.

**Step 4** — The agent requests the user to upload error screenshots and device detail images via the Conversation Topic (upload image to VA).

**Step 5** — The Incident record is created first with `state = New` and the uploaded images attached to the record.

**Step 6** — Now Assist Document Intelligence auto-triggers on each image. It extracts structured fields per the Doc Intel use case configuration. The key output field `u_extracted_error_code` is populated on the incident record — arming the downstream agentic workflow trigger.

| Incident field | Value |
|---------------|-------|
| `short_description` | User-reported description |
| `cmdb_ci` | Affected CI (e.g., Veritas server) |
| `u_extracted_error_code` | Extracted error code (e.g., `HOST_UNREACHABLE`) |
| `category` | Network / Infrastructure |
| `urgency` | 1 — Critical |
| `state` | 2 — In Progress |
| `contact_type` | chat |

→ See [01 — Now Assist Virtual Agent](01-now-assist-virtual-agent.md) | [02 — L1 First Responder Agent](02-L1-agent-first-responder-analyst-agent.md) | [04 — Now Assist Document Intelligence](04-now-assist-document-intelligence.md)

---

### Phase 2 — Fulfiller Flow — Veritas Resolution Finder Agent

**Trigger:** The Veritas Resolution Agentic Workflow fires when:
- `state = In Progress`
- `contact_type = chat`
- `u_extracted_error_code` is not empty

These conditions are evaluated on the **`incident extend`** table (`x_nava_agentic_lab_incident_extend`), which holds the Doc Intel extracted fields.

**Path A — Step 1 (parallel fire):** Two Now Assist skills fire in parallel from the workflow Start node:

1. `GenerateSearchQueryAgainstAISearch` skill — constructs an optimised AI search query from the error code and incident context
2. `FindSimilarIncidents` skill (Predictive Intelligence) — surfaces historically similar incidents using ML-based similarity matching

**Path A — Step 2:** The optimised AI search query is passed to the `RetrieveRelevantKBContent` Retriever skill, which fetches ranked relevant results from the ServiceNow Knowledge Base using AI Search. Results from FindSimilarIncidents merge at this stage.

**Path A — Step 3:** The `Assess if solution exists` Skill Prompt evaluates the combined retrieval output (KB content + similar incidents) and determines whether a viable resolution exists. This is a reasoning step — the agent does not proceed to remediation unless a solution is confirmed.

**Path A — Result:** If a solution exists → structured Resolution Plan built and written to `work_notes` → Phase 3 triggered. If no solution confirmed → fall through to Path B.

**Path B:** If no result from KB or similar incidents, a privacy-safe web search is triggered. PII and internal identifiers are stripped before the query is sent externally (enforced by the `GenerateWebSearchQnsForResolutionPlan` skill prompt).

**Path B — Fallback:** If web search also yields no actionable result → the incident is escalated to L2. The external agent (Phase 3) is NOT triggered — no automated remediation executes on an unresolved search path.

→ See [03 — CreateOptimalSearchQuery Skill](03-now-assist-skill-kit.md) | [03 — GenerateWebSearchQns Skill](03-now-assist-skill-kit-web-search.md) | [03 — Skill Kit Part 2](03-now-assist-skill-kit-part2.md) | [06 — Fulfiller AI Agent](06-fulfiller-ai-agent.md) | [08 — Agentic Workflow](08-wrapping-in-agentic-workflow.md)

---

### Phase 3 — External Integration — Observability & Action Agent via A2A

**Trigger:** Phase 3 fires only when Path A of the Fulfiller Flow produces a resolution plan. It does **not** trigger on Path B.

**Step 1** — The Observability & Action Agent (running in ServiceNow as the primary orchestrator) receives the resolution plan and constructs a structured remediation action set.

**Step 2** — The remediation plan is serialised and dispatched to Azure AI Foundry via the **A2A (Agent-to-Agent) protocol**. ServiceNow remains the orchestrator; Azure AI Foundry acts as the remote execution agent.

**Step 3** — Azure AI Foundry executes the prescribed remediation actions in the target environment (e.g., service restart, config patch, queue flush). Execution telemetry streams back to ServiceNow.

**Path 3A — Success:** Incident auto-resolved, resolution notes and execution outcome written to the incident record, ticket closed. No human intervention required.

**Path 3B — Fail / Partial:** Full execution log, attempted steps, and failure reason written to incident work notes. Incident escalated to L2 with complete context for investigation.

→ See [07 — External Agent Integration (A2A OAuth + Registration)](07-external-agent-integration.md) | [12 — Observability and Action Agent](12-observability-action-agent.md)

---

## 5. Key ServiceNow Components & Capabilities

| Component | Role in this use case |
|-----------|-----------------------|
| **Now Assist Virtual Agent (NAVA)** | Front-door for all user interactions. Handles multi-turn conversation, intent routing, troubleshooting delivery, and image upload orchestration. |
| **L1 / Requestor AI Agent** | Orchestrates the requestor-side flow: KG lookup, troubleshooting, user prompting for images, and handoff to incident creation. |
| **Knowledge Graph (User Graph)** | Identifies and contextualises the user at session start, enabling personalised troubleshooting and pre-populated incident fields. |
| **Troubleshooting Guide (file upload tool)** | Surfaces step-by-step resolution guides via the file upload tool, delivered through the Conversation Topic interface. |
| **Now Assist Document Intelligence** | Auto-triggers on images attached to an incident. Extracts structured fields (including `u_extracted_error_code`) per use case config — no manual data entry. |
| **Veritas Resolution Agentic Workflow** | The named agentic workflow spanning the Fulfiller and External Integration flows. Owned by the Resolution Finder Agent. Contains search cascade and A2A dispatch logic. |
| **GenerateSearchQueryAgainstAISearch Skill** | Generates an optimised AI search query from the raw error code + incident context. Runs in parallel with FindSimilarIncidents from the workflow Start node. |
| **FindSimilarIncidents (Predictive Intelligence) + RetrieveRelevantKBContent Retriever** | FindSimilarIncidents uses Predictive Intelligence ML to surface historically similar incidents. RetrieveRelevantKBContent Retriever fetches ranked KB results using the AI-generated query. Both feed the Assess if solution exists Skill Prompt. |
| **GenerateWebSearchQnsForResolutionPlan Skill** | Fallback when internal sources find no resolution. Generates privacy-safe web search questions by chaining the CreateOptimalSearchQuery skill output. Strips all internal identifiers before external query. |
| **Observability & Action Agent** | Builds the remediation plan from the resolution found in Phase 2 and dispatches it to Azure AI Foundry via A2A. |
| **A2A Protocol** | Agent-to-Agent protocol enabling cross-platform agent communication. ServiceNow acts as primary orchestrator; Azure AI Foundry as remote executor. |
| **Azure AI Foundry (ObsAgent)** | Remote execution agent that carries out remediation actions in the target environment and streams telemetry back to ServiceNow. |

---

## 6. Decision Path Summary

| Phase | Condition | Path | Outcome |
|-------|-----------|------|---------|
| Phase 1 | KB deflection succeeds | **1A** | Deflected in chat. No Incident created. ✅ |
| Phase 1 | No KB deflection | **1B** | Incident created. Pipeline continues. |
| Phase 2 | Internal KB + similar incidents resolve | **Path A** | Resolution Plan written. Phase 3 triggered. ✅ |
| Phase 2 | Internal fails → web search resolves | **Path B** | Resolution Plan written. Phase 3 triggered. ✅ |
| Phase 2 | All sources fail | **Path B fallback** | Work notes + L2 escalation. Phase 3 NOT triggered. 🔴 |
| Phase 2 | `u_extracted_error_code` empty | *(no trigger)* | Agentic Workflow does not fire. Manual queue. |
| Phase 3 | Azure AI Foundry executes successfully | **Path 3A** | Result written. Incident auto-resolved. ✅ |
| Phase 3 | Azure AI Foundry fails / partial | **Path 3B** | Partial log written. L2 handoff with full context. 🔴 |

---

## 7. Complete Incident Record at End of Pipeline

By the end of a successful run (Path A → Path 3A), the Incident `work_notes` contains:

```
[Entry 1 — Resolution Pathfinder Agent]
---
RESOLUTION PLAN
Source: Internal KB / AI Search
Steps:
1. Restart network interface on affected server
2. Clear ARP cache
3. Verify connectivity
Citation: KB0045231 — Network Interface Recovery Procedures
---

[Entry 2 — Observability & Action Agent via A2A]
---
REMEDIATION RESULT
Agent: Azure AI Foundry ObsAgent
Status: Success
Actions taken:
1. Restarted network interface
2. Cleared ARP cache
Completed: 2026-03-30 14:38:22 UTC
---
```

The Incident carries a complete, audit-ready record of every AI decision and every action taken — with no human involvement required for known error types.

---

## 8. Lab Capabilities Index

| # | Capability | Phase | Lab Doc |
|---|-----------|-------|--------|
| 01 | Now Assist for Virtual Agent (NAVA) | 1 | [01-now-assist-virtual-agent.md](01-now-assist-virtual-agent.md) |
| 02 | L1 First Responder Operations Analyst Agent | 1 | [02-L1-agent-first-responder-analyst-agent.md](02-L1-agent-first-responder-analyst-agent.md) |
| 03a | Now Assist Skill Kit — CreateOptimalSearchQuery | 2 | [03-now-assist-skill-kit.md](03-now-assist-skill-kit.md) |
| 03b | Now Assist Skill Kit — GenerateWebSearchQns | 2 | [03-now-assist-skill-kit-web-search.md](03-now-assist-skill-kit-web-search.md) |
| 03c | Now Assist Skill Kit — Part 2 (Resolution Finder) | 2 | [03-now-assist-skill-kit-part2.md](03-now-assist-skill-kit-part2.md) |
| 04 | Now Assist Document Intelligence | 1 | [04-now-assist-document-intelligence.md](04-now-assist-document-intelligence.md) |
| 05 | Elastic MCP Integration | 2 | [05-mcp-elastic-integration.md](05-mcp-elastic-integration.md) |
| 06 | Fulfiller AI Agent (Resolution Pathfinder) | 2 | [06-fulfiller-ai-agent.md](06-fulfiller-ai-agent.md) |
| 07 | External Agent Integration — A2A OAuth + Registration | 3 | [07-external-agent-integration.md](07-external-agent-integration.md) |
| 08 | Wrapping in Agentic Workflow (Veritas) | 2+3 | [08-wrapping-in-agentic-workflow.md](08-wrapping-in-agentic-workflow.md) |
| 09 | Web Search Query Generator Skill | 2 | [09-web-search-query-generator.md](09-web-search-query-generator.md) |
| 10 | Web Search Tool | 2 | [10-web-search-tool.md](10-web-search-tool.md) |
| 11 | Elastic MCP Server | 2 | [11-elastic-mcp-server.md](11-elastic-mcp-server.md) |
| 12 | Observability & Action Agent (A2A — Azure AI Foundry) | 3 | [12-observability-action-agent.md](12-observability-action-agent.md) |

---

## 9. Value Outcomes

| Outcome | How the architecture delivers it |
|---------|----------------------------------|
| **L1 ticket deflection** | Troubleshooting guide delivered in chat before any incident is created. Deflected conversations produce no ticket, reducing incident volume. |
| **Structured incident data without manual entry** | Now Assist Doc Intel extracts all key fields from user images automatically — no form filling, no agent interpretation required. |
| **Faster resolution via optimised search** | GenerateSearchQueryAgainstAISearch + FindSimilarIncidents (Predictive Intelligence) + RetrieveRelevantKBContent + Assess if solution exists cascade finds the best available resolution — KB content and similar incidents evaluated together before escalating. |
| **Zero-touch remediation for known paths** | Path A + Phase 3 delivers end-to-end resolution with no human involvement: from error code to closed incident. |
| **Context-rich L2 handoffs** | Every escalation path (Path B fallback, Path 3B) writes a complete trail — attempted steps, search results, execution logs — so L2 engineers never start from zero. |
| **Cross-platform AI orchestration** | A2A protocol integration with Azure AI Foundry demonstrates ServiceNow as the orchestration layer for multi-cloud AI — not a walled garden. |

---

## 10. Positioning & SC Notes

This use case is positioned for the following customer conversations:

**"How do I connect my existing Azure AI investment to ServiceNow?"** — The A2A integration with Azure AI Foundry is a direct answer: ServiceNow orchestrates, Azure executes, and the result is written back to the system of record.

**"Can ServiceNow actually fix things, or just raise tickets?"** — Path A + Phase 3 demonstrates end-to-end autonomous remediation with a full audit trail.

**"How does your AI handle cases it can't solve?"** — Path B and Path 3B show deliberate, context-rich escalation — not silent failure.

**"Is there human oversight?"** — Path B explicitly requires no external execution without a confirmed resolution. Path 3B writes a complete log before L2 handoff. The system is designed for regulated environments.

> **Relevant product capabilities to highlight:** Now Assist Virtual Agent (Agentic Reasoning) · AI Agent Studio · Now Assist Document Intelligence · Agentic Workflow · A2A Protocol · GenerateSearchQueryAgainstAISearch Skill · FindSimilarIncidents (Predictive Intelligence) · RetrieveRelevantKBContent Retriever · Knowledge Graph

---

## 11. Key Technical Constraints

| Constraint | Detail |
|-----------|--------|
| Agentic Workflow trigger requires | `state = In Progress`, `contact_type = chat`, `u_extracted_error_code` non-empty — evaluated on `incident extend` table |
| A2A / External Agent minimum release | ServiceNow Zurich Patch 4 |
| OAuth for A2A | Client Credentials flow — `Default Grant = Client Credentials`, `Send Credentials = In Request Body` — Azure requires this |
| Privacy — web search | Internal hostnames, IPs, user names stripped before internet search (enforced by GenerateWebSearchQnsForResolutionPlan skill prompt) |
| Communication mode — ObsAgent | Synchronous for remediations < 30s; Asynchronous + callback registry for longer-running operations |
| ACL on External Agent | Required — without it, the Observability & Action Agent cannot be invoked from the Agentic Workflow |
| Auth — Azure AI Foundry | OAuth 2.0 Client Credentials via `FoundryConnectCreds` Connection & Credential Alias |
| Agent routing in Agentic Workflow | LLM-governed — workflow description `<steps>` XML must align with agent names and descriptions for correct step-to-agent matching |
| Dynamic user data access | Workflow runs as `Assigned to [task]` — inherits ITIL user permissions, capped at `itil` approved role |

---

*Document prepared by ServiceNow APJ Intelligent Automation SC Team · March 2026*
