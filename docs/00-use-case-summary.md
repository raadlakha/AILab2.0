# Use Case Summary — AI-Powered IT Incident Resolution (End-to-End)

> **Release:** Zurich | **Lab:** ServiceNow Native AI Enablement Lab

---

## Scenario

> A user reports that a production server is inaccessible. This document traces every step the platform takes — from the user's first message to the execution of a remediation action — including every decision point and alternate path.

---

## Overview — Full Pipeline

```
User Message (NAVA Chat)
        │
        ▼
[STAGE 1] First Responder AI Agent — Intake & Triage
        │
        ├── PATH 1A: KB answer found → respond in chat → DONE (deflected)
        └── PATH 1B: No KB answer → enrich and create Incident
                │
                ▼
        [STAGE 2] Resolution Pathfinder — Background Investigation
                │
                ├── PATH 2A: Internal KB resolves → write plan → continue
                ├── PATH 2B: Elastic logs resolve → write plan → continue
                └── PATH 2C: Web search resolves → write plan → continue
                        │
                        ▼
                [STAGE 3] Observability and Action Agent — Remediation
                        │
                        ├── PATH 3A: Remediation succeeds → write result → DONE
                        └── PATH 3B: Remediation fails / partial → write result → manual handoff
```

---

## Stage 1 — User Intake (Capabilities 01 – 05, 07)

### Step 1: User opens NAVA and types a message
**Capability 01 — Now Assist for Virtual Agent (NAVA)**

- The user navigates to the Service Portal and opens the Now Assist chat widget
- NAVA receives the message: *"The production server prod-app-01 is unreachable"*
- The Now Assist Panel powers the LLM-backed conversation layer

---

### Step 2: Intent is identified and routed
**Capability 02 — Conversational Topics**

- The NLU model evaluates the message against trained intents
- Minimum confidence threshold: **0.70**
- If the intent **"Server Unreachable"** is matched → topic activates and hands off to the First Responder AI Agent
- If confidence < 0.70 → the topic falls back to the default VA flow (out of scope for this lab)

---

### Step 3: First Responder AI Agent takes over the conversation
**Capability 05 — First Responder Operations Analyst (Chat AI Agent)**

The First Responder is a **Chat AI Agent** running in AI Agent Studio. It takes control of the NAVA conversation and begins gathering context.

#### Step 3a — Knowledge Graph search (immediate deflection attempt)
**Capability 03 — Knowledge Graph**

- Tool call: `KnowledgeGraphSearch`
- The agent queries the semantic data layer using the user's description
- **PATH 1A — KB answer found:**
  - Agent responds with resolution steps directly in the chat
  - Offers the user confirmation that the issue is resolved
  - **Conversation ends. No Incident created. Deflected.**
- **PATH 1B — No KB answer found:**
  - Agent continues to gather context
  - Proceeds to Step 3b

#### Step 3b — Error code extraction from uploaded screenshot
**Capability 04 — Now Assist in Document Intelligence**

- Agent prompts user: *"Could you upload a screenshot of the error you're seeing?"*
- User uploads a screenshot via the chat
- Tool call: `UploadSupportingDocument` (File Upload tool type)
- Now Assist in Document Intelligence processes the image using GenAI
- Extracts the error code (e.g., `HOST_UNREACHABLE`) and populates `u_extracted_error_code`
- **If no document uploaded:** The agent may attempt to infer the error code from the conversation text, or proceed without it (the Agentic Workflow requires `u_extracted_error_code` to be non-empty — so if not extracted, the Incident is still created but the Resolution Pathfinder will not trigger automatically)

#### Step 3c — Structured data extraction
**Capability 07 — Flow Action Tool (ExtractIncidentDetails)**

- Tool call: `ExtractIncidentDetails` (Flow Action tool type)
- The Flow Designer action runs a script that parses the conversation and extracts:
  - `cmdb_ci` — the affected server (e.g., `prod-app-01`)
  - `u_extracted_error_code` — the error code (e.g., `HOST_UNREACHABLE`)
  - `category` — derived from error type
  - `subcategory`
  - `urgency` — derived from error severity (e.g., `HOST_UNREACHABLE` → urgency = 1 - Critical)

#### Step 3d — Incident creation
- The agent creates an Incident record with all enriched context:

| Field | Value |
|-------|-------|
| `short_description` | User-reported description |
| `cmdb_ci` | Affected server CI |
| `u_extracted_error_code` | `HOST_UNREACHABLE` |
| `category` | Network / Infrastructure |
| `urgency` | 1 — Critical |
| `state` | 2 — In Progress |
| `contact_type` | chat |

- The agent informs the user: *"I've created Incident INC0012345. Our systems are investigating automatically."*

---

## Stage 2 — Background Resolution Investigation (Capabilities 06, 08, 09, 10, 11)

The Incident record save triggers the **Resolution Pathfinder Agentic Workflow**.

### Step 4: Agentic Workflow trigger evaluation
**Capability 06 — Resolution Pathfinder (Agentic Workflow)**

The workflow evaluates the trigger conditions on the Incident record:

| Condition | Required Value | Outcome |
|-----------|---------------|---------|
| `state` | = `2` (In Progress) | Must be true |
| `contact_type` | = `chat` | Must be true |
| `u_extracted_error_code` | not empty | Must be true |

- **All conditions met** → Agentic Workflow fires → Resolution Pathfinder AI Agent is invoked in the background
- **Any condition not met** → Workflow does not trigger → Incident sits in the queue for manual L2 pickup

---

### Step 5: Resolution Pathfinder investigates — Internal Data
**Capability 08 — Resolution Finder (Internal Data) — Now Assist Skill**

- Tool call: `ResolutionFinderInternalData` (Now Assist Skill — RAG-based)
- The Skill uses Retrieval-Augmented Generation to search:
  - Knowledge Graph KB articles tagged to the error code
  - Previously resolved Incidents with a similar `u_extracted_error_code`
- Returns a JSON payload:
  ```json
  {
    "resolution_found": true | false,
    "steps": ["step 1", "step 2", ...],
    "source": "KB article title or Incident number",
    "confidence": 0.0–1.0
  }
  ```

**PATH 2A — Internal resolution found (confidence ≥ threshold):**
- Agent formats the resolution plan:
  ```
  ---
  RESOLUTION PLAN
  Source: Internal KB
  Steps:
  1. Restart network interface on prod-app-01
  2. Clear ARP cache
  Citation: KB0045231 — Network Interface Recovery Procedures
  ---
  ```
- Plan is written to the Incident `work_notes`
- Agent proceeds directly to **Stage 3** (Observability and Action Agent)

**PATH 2B — No internal resolution found:**
- Agent proceeds to Step 6

---

### Step 6: Resolution Pathfinder investigates — Live Log Data
**Capability 11 — Elastic MCP Server Integration**

- Tool call: `ElasticLogSearch` (MCP Tool via MCP Client)
- Parameters passed: `{ "error_code": "HOST_UNREACHABLE", "server": "prod-app-01" }`
- ServiceNow MCP Client calls the Elastic MCP Server over the registered connection
- Elastic runs: `logs WHERE error_code=HOST_UNREACHABLE AND host=prod-app-01`
- Returns matching log entries with timestamps

**PATH 2B-i — Root cause found in logs:**
- Agent analyses log entries and identifies root cause (e.g., *"NIC failure at 14:22:05 UTC, repeated LINK_DOWN events"*)
- Formats resolution plan:
  ```
  ---
  RESOLUTION PLAN
  Source: Log Analysis
  Steps:
  1. Investigate NIC failure on prod-app-01 (LINK_DOWN events at 14:22)
  2. Replace or reset the network interface card
  3. Verify link state after reset
  Citation: Elastic logs — prod-app-01 — 2024-03-15 14:22:05 UTC
  ---
  ```
- Plan written to Incident `work_notes`
- Agent proceeds to **Stage 3** (Observability and Action Agent)

**PATH 2B-ii — No conclusive finding in logs:**
- Agent proceeds to Step 7

---

### Step 7: Resolution Pathfinder investigates — Internet Search
**Capability 09 — Web Search Query Generator — Now Assist Skill**
**Capability 10 — Web Search Tool**

#### Step 7a — Generate a privacy-safe search query
- Tool call: `GenerateWebSearchQuery` (Now Assist Skill)
- The Skill strips all internal identifiers (hostnames, IPs, internal names) and generates a clean, public-safe query
- Example output: `"HOST_UNREACHABLE error network interface Linux server restart steps"`
- **Privacy rule:** Internal values like `prod-app-01` or `192.168.1.50` are never sent to the internet

#### Step 7b — Execute the web search
- Tool call: `WebSearch` (built-in Web Search tool type — Google Grounding)
- Top results are returned and synthesised by the LLM
- Agent formats the resolution plan:
  ```
  ---
  RESOLUTION PLAN
  Source: Web Search
  Steps:
  1. Check if the network interface is up: ip link show
  2. Restart the interface: ifup eth0
  3. Flush routing tables: ip route flush cache
  4. Test connectivity: ping -c 4 <gateway>
  Citation: https://example.com/host-unreachable-linux-fix
  ---
  ```
- Plan written to Incident `work_notes`

**PATH 2C — Web search also fails to produce a resolution:**
- Agent writes to `work_notes`:
  ```
  No automated resolution found. Manual investigation required.
  Searched: Internal KB, Elastic Log System, Web.
  ```
- Incident remains In Progress for L2 manual pickup
- **Pipeline ends here. Stage 3 is not invoked.**

---

## Stage 3 — Autonomous Remediation (Capability 12)

Triggered automatically by the Agentic Workflow after the resolution plan is written (Paths 2A, 2B-i, and 2C excluded).

### Step 8: Agentic Workflow passes control to the Observability and Action Agent
**Capability 12 — Observability and Action Agent (External AI Agent — A2A Protocol)**

- The Agentic Workflow's next stage activates the **Observability and Action Agent**
- This is an **External AI Agent** type registered in AI Agent Studio via the **Agent2Agent (A2A) protocol**
- ServiceNow's **AI Agent Fabric** invokes the agent using the A2A open standard

---

### Step 9: ServiceNow discovers and calls the Azure AI Foundry agent

- ServiceNow has previously discovered the Azure AI Foundry agent via its **Agent Card** at:
  `https://<azure-foundry-endpoint>/.well-known/agent.json`
- The Agent Card declares the external agent's skills:
  - `run_remediation` — executes structured remediation steps against a CI
  - `check_server_status` — verifies current server state
- ServiceNow sends the resolution plan steps and incident context to Azure AI Foundry:
  - Resolution steps (from `work_notes`)
  - Affected CI (`cmdb_ci` = `prod-app-01`)
  - Error code (`u_extracted_error_code` = `HOST_UNREACHABLE`)

---

### Step 10: Azure AI Foundry executes the remediation

The Azure AI Foundry agent runs the actual actions against the infrastructure:

**Example remediation sequence for `HOST_UNREACHABLE`:**
1. Restart network interface on `prod-app-01`
2. Clear ARP cache
3. Verify connectivity (ping test)

**PATH 3A — Remediation succeeds:**
- Azure AI Foundry returns:
  ```json
  {
    "status": "success",
    "actions_taken": [
      "Restarted network interface on prod-app-01",
      "Cleared ARP cache"
    ],
    "timestamp": "2024-03-15T14:38:22Z"
  }
  ```
- ServiceNow writes to Incident `work_notes`:
  ```
  ---
  REMEDIATION RESULT
  Agent: Azure AI Foundry — Observability and Action Agent
  Status: Success
  Actions taken:
  1. Restarted network interface on prod-app-01
  2. Cleared ARP cache
  Completed: 2024-03-15 14:38:22 UTC
  ---
  ```
- Incident can be auto-resolved or flagged for closure review

**PATH 3B — Remediation fails or partial:**
- Azure AI Foundry returns:
  ```json
  {
    "status": "partial",
    "actions_taken": ["Restarted network interface on prod-app-01"],
    "failed_actions": ["ARP cache clear failed — permission denied"],
    "timestamp": "2024-03-15T14:38:45Z"
  }
  ```
- ServiceNow writes the partial result to `work_notes`
- Incident remains In Progress with full audit trail of what was attempted
- L2 engineer picks up the Incident with complete context: original description, resolution plan, and remediation attempt log

---

## Complete Incident Record at End of Pipeline

By the end of a successful run, the Incident `work_notes` contains:

```
[Entry 1 — from Resolution Pathfinder]
---
RESOLUTION PLAN
Source: Internal KB
Steps:
1. Restart network interface on prod-app-01
2. Clear ARP cache
Citation: KB0045231
---

[Entry 2 — from Observability and Action Agent]
---
REMEDIATION RESULT
Agent: Azure AI Foundry — Observability and Action Agent
Status: Success
Actions taken:
1. Restarted network interface on prod-app-01
2. Cleared ARP cache
Completed: 2024-03-15 14:38:22 UTC
---
```

The Incident carries a complete, audit-ready record of every AI decision and every action taken — with no human involvement required for known error types.

---

## Decision Path Summary

| Stage | Condition | Path | Outcome |
|-------|-----------|------|---------|
| Stage 1 — Intake | KB answer found immediately | **1A** | Deflected in chat. No Incident. |
| Stage 1 — Intake | No KB answer | **1B** | Incident created. Pipeline continues. |
| Stage 2 — Investigation | Internal KB resolves | **2A** | Plan written. Stage 3 invoked. |
| Stage 2 — Investigation | Internal KB fails, logs resolve | **2B-i** | Plan written. Stage 3 invoked. |
| Stage 2 — Investigation | Internal KB + logs fail, web resolves | **2C** | Plan written. Stage 3 invoked. |
| Stage 2 — Investigation | All three sources fail | **2D** | Manual note written. No Stage 3. L2 pickup. |
| Stage 2 — Investigation | `u_extracted_error_code` empty | *(no trigger)* | Agentic Workflow does not fire. Manual queue. |
| Stage 3 — Remediation | Azure Foundry succeeds | **3A** | Result written. Incident resolved. |
| Stage 3 — Remediation | Azure Foundry fails or partial | **3B** | Partial result written. L2 handoff with full context. |

---

## Capabilities Reference

| # | Capability | Stage | Role in Pipeline |
|---|-----------|-------|-----------------|
| 01 | Now Assist for Virtual Agent (NAVA) | 1 | Chat entry point |
| 02 | Conversational Topics (NLU) | 1 | Intent routing to AI agent |
| 03 | Knowledge Graph | 1 | Immediate deflection search |
| 04 | Now Assist in Document Intelligence | 1 | Error code extraction from screenshots |
| 05 | First Responder AI Agent (Chat) | 1 | Conversation orchestration, Incident creation |
| 06 | Resolution Pathfinder (Agentic Workflow) | 2 | Background investigation + orchestration trigger |
| 07 | Flow Action Tool — Extract Incident Details | 1 | Structured data extraction from conversation |
| 08 | Resolution Finder — Internal Data (Now Assist Skill) | 2 | RAG search over KB + resolved incidents |
| 09 | Web Search Query Generator (Now Assist Skill) | 2 | Privacy-safe internet query generation |
| 10 | Web Search Tool | 2 | Internet resolution search |
| 11 | Elastic MCP Server (MCP Client) | 2 | Live log querying for root cause |
| 12 | Observability and Action Agent (A2A — Azure AI Foundry) | 3 | Autonomous remediation execution |

---

## Key Technical Constraints

| Constraint | Detail |
|-----------|--------|
| Agentic Workflow requires | `state=In Progress`, `contact_type=chat`, `u_extracted_error_code` non-empty |
| A2A / External Agent requires | ServiceNow Zurich **patch 4** minimum |
| Privacy — web search | Internal hostnames, IPs, user names must be stripped before internet search (enforced by Cap. 09 prompt) |
| Communication mode — Cap. 12 | Synchronous if remediation < 30 seconds; Asynchronous if longer-running |
| ACL on External Agent | Required — without it, the Observability and Action Agent cannot be invoked from the Agentic Workflow |
| Auth — Elastic MCP | API Key or OAuth 2.1 (managed in AI Agent Studio) |
| Auth — Azure AI Foundry | OAuth 2.0 or API Key via Connection & Credential alias |
