# 02 — L1 / First Responder Analyst Agent

> **Release:** Zurich | **Flow:** Requestor Flow — Phase 1 (Steps 2–5)
> **Source:** [ServiceNow Zurich — AI Agent Studio](https://www.servicenow.com/docs/bundle/zurich-intelligent-experiences/page/administer/now-assist-ai-agents/concept/ai-agent-studio.html) | [Create an AI agent](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/configure-next-best-action-agent.html) | [Add tools and information](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/add-tool-aia.html)

---

## What It Is

The **L1 / First Responder Analyst Agent** is an **AI Agent** built in **AI Agent Studio**. It is the first automated intelligence that engages after NAVA routes a user message — taking over the conversation and orchestrating the Requestor Flow from user intake through to Incident creation.

In Zurich, the correct platform hierarchy is:

```
Agentic Workflow (Use Case)   ← the business problem / why
        │
        └── AI Agent           ← the worker / who
                │
                └── Tools      ← the actions / how
```

This section covers building the **AI Agent** only. The Agentic Workflow that connects this agent to the NAVA trigger is configured separately.

---

## Role in the Requestor Flow

```
[Steps 2–5 — Requestor Flow]

NAVA receives user message (contact_type = chat stamped)
        │
        ▼
Step 2: L1 Agent triggered by AI Agent Planner
        │  Queries User Graph → identifies role, department, CI
        ▼
Step 3: Agent presents Troubleshooting Guide
        │  via Conversation Topic tool → steps delivered in chat
        ▼
   ┌────┴──────────────────────────────┐
   │  Decision: Issue resolved?         │
   └───────────────┬───────────────────┘
                   │
        YES → Deflect in chat. No Incident created.
                   │
        NO  → Continue to Step 4
                   │
                   ▼
Step 4: Agent requests user to upload error screenshots
        │  and device detail images via File Upload tool
        ▼
Step 5: Incident created (state = New)
        │  uploaded images attached to Incident record
        ▼
Now Assist Document Intelligence auto-triggers on attachments
(see 04-now-assist-document-intelligence.md)
```

> **Deflection is the optimal outcome.** If the Troubleshooting Guide resolves the issue, no Incident is created and no downstream automation fires.

---

## What the L1 Agent Enables

| Capability | How the L1 Agent Delivers It |
|-----------|------------------------------|
| User contextualisation | Knowledge Graph query identifies the user's role and affected CI before any conversation step |
| Guided troubleshooting | Troubleshooting Guide steps delivered conversationally through the Conversation Topic tool |
| Deflection | If resolved in chat, no Incident created — reduces ticket volume |
| Structured image collection | File Upload tool prompts the user to upload error screenshots and device labels in-chat |
| Enriched Incident creation | Incident created with `state = New`, images attached — arming NADI for downstream extraction |

---

## Prerequisites

| Requirement | Detail |
|-------------|--------|
| Licence | Now Assist Pro+ or ITSM Pro+ |
| Plugin — Now Assist AI Agents | `sn_aia` — must be Active (v5.2+ recommended for Zurich) |
| Plugin — GAIC | `sn_gaic` — v11.2+ |
| AI Search | Must be enabled — agents return "No agents available" without it |
| Now Assist Panel | Must be enabled |
| Role | `sn_aia.admin` or `admin` |
| NAVA | Capability 01 complete — NAVA configured, AI Agents skill enabled in Assistant Designer |

---

## Lab Exercise — Steps to Build

### Step 1: Open AI Agent Studio → Create and Manage → AI Agents

Navigate to **All → AI Agent Studio → Create and manage → AI Agents** → click **New**

![L1 Agent — New AI Agent form](../screenshots/L1-Agent.png)

> This lands you on the new AI Agent form. The **Create and manage** tab is the entry point for building both Agentic Workflows and AI Agents. Select the **AI Agents** sub-tab, then click **New** on the right-hand side.

---

### Step 2: Configure Describe and Instruct

The new agent form opens on the **Describe and instruct** section.

![L1 Agent — Describe and instruct section](../screenshots/L1-Agent2.png)

Fill in the agent's identity fields:

| Field | Value |
|-------|-------|
| Name | `L1 First Responder Analyst` |
| Description | `Handles user-reported IT infrastructure issues via NAVA. Identifies the user via Knowledge Graph, delivers troubleshooting steps, collects device images, and creates an enriched Incident record.` |

The **AI agent role** and **Instructions** fields are prompt fields for SCs to author based on their specific use case design. Max lengths: Role = 2,000 chars, Instructions = 8,000 chars.

> **Proficiency is auto-generated** from the agent's description and role. It determines which user queries the Orchestrator routes to this agent. If the Proficiency is missing or vague, you will get "No agents available at the moment" errors in NAVA — add it as a column in the AI Agents list view to inspect it.

Click **Save and continue**.

---

### Step 3: Add Tools — "Add tools and information" Tab

Click the **Add tools and information** tab to configure tools for this agent.

#### Tool 1 — Knowledge Graph

![L1 Agent — Tool 1 list view](../screenshots/L1-Agent-Tool1.png)

![L1 Agent — Knowledge Graph tool config](../screenshots/L1-agent-tool-kg1.png)

| Field | Value |
|-------|-------|
| Tool type | **Knowledge Graph** |
| Name | *(name the tool)* |
| Schema | IT Infrastructure schema *(configured in Capability 03)* |
| Description | *(describe what this tool does for the agent — used by the LLM to decide when to call it)* |

> The Knowledge Graph tool gives the agent access to the User Graph — allowing it to identify the caller's role, department, and affected CI without asking the user.

---

#### Tool 2 — Subflow

![L1 Agent — Tool 2 (Subflow)](../screenshots/L1-agent-tool2.png)

![L1 Agent — Subflow tool config](../screenshots/L1-agent-tool2-subflow.png)

![L1 Agent — Subflow tool detail](../screenshots/L1-agent-tool2-subflow2.png)

| Field | Value |
|-------|-------|
| Tool type | **Subflow** |
| Subflow | *(select the Troubleshooting Guide subflow from Flow Designer)* |
| Description | *(describe what this tool retrieves — guides the LLM on when to invoke it)* |

---

#### Tool 3 — Conversation Topic

![L1 Agent — Conversation Topic tool](../screenshots/L1-agent-tool-conv-topic.png)

| Field | Value |
|-------|-------|
| Tool type | **Conversation Topic** |
| Topic | *(select the Troubleshooting Guide Conversation Topic)* |
| Description | *(describe the conversational delivery purpose)* |

---

#### Tool 4 — File Upload

![L1 Agent — File Upload tool](../screenshots/L1-agent-tool4-file-upload.png)

![L1 Agent — File Upload tool detail](../screenshots/L1-agent-tool4-file-upload2.png)

| Field | Value |
|-------|-------|
| Tool type | **File Upload** |
| Description | *(describe the purpose — the agent will use this to collect error screenshots and device images)* |

> **This tool is the bridge to NADI.** Files uploaded by the user via this tool are attached to the Incident record. NADI (Capability 04) auto-triggers on those attachments and extracts `u_extracted_error_code`.

---

### Step 4: Configure Channel and Status

Navigate to the **Channel and status** section on the AI Agent.

![L1 Agent — Channel and status config](../screenshots/L1-agent-channel-status.png)

| Setting | Value |
|---------|-------|
| Virtual Agent experience | ✅ Toggled on — required for NAVA to invoke this agent |
| Status | **Active** |

> The Virtual Agent experience toggle must be on for the AI Agent Orchestrator to route NAVA queries to this agent. Without it, the agent will not be discoverable from the Now Assist Panel.

---

### Step 5: Configure Data Access

Navigate to the **Data Access** section.

![L1 Agent — Data access settings](../screenshots/L1-agent-data-access.png)

Ensure the agent has access to:

| Table | Purpose |
|-------|---------|
| `incident` | Create Incident records |
| `cmdb_ci` | Resolve affected CI from Knowledge Graph |
| `sys_user` | Identify caller from logged-in session |
| Extended Incident table | Write fields populated downstream by NADI |

---

### Step 6: Configure User Access

Navigate to the **User Access** section.

![L1 Agent — User access settings](../screenshots/L1-agent-user-access.png)

| Setting | Value |
|---------|-------|
| User roles | `now_assist_panel_user` |

> `now_assist_panel_user` is the minimum role required for a user to invoke an AI Agent from the Now Assist Panel / NAVA chat.

---

### Step 7: Test the Agent

Navigate to **AI Agent Studio → Testing**.

1. Enter a test scenario describing an IT infrastructure issue
2. Verify the agent invokes the correct tools in sequence
3. Confirm an Incident is created with `state = New` and `contact_type = chat`
4. Confirm uploaded images are attached to the Incident record
5. Verify NADI triggers on the attachments and populates `u_extracted_error_code`

> If you get **"No agents available at the moment"**: check that AI Search is enabled, the AI Agent is Active, the **Proficiency** is populated, and **Virtual Agent experience** is toggled on.

---

## Key Configuration Fields

| Field | Value for This Lab |
|-------|-------------------|
| AI Agent name | L1 First Responder Analyst |
| Role required to configure | `sn_aia.admin` |
| User role required to invoke | `now_assist_panel_user` |
| Tool 1 | Knowledge Graph — User Graph |
| Tool 2 | Subflow — Troubleshooting Guide |
| Tool 3 | Conversation Topic — Troubleshooting Guide |
| Tool 4 | File Upload — error screenshots and device images |
| Virtual Agent experience | Toggled on |
| Incident state on creation | `New` |
| Incident contact_type | `chat` |
| Images attached | Yes — triggers NADI |

---

## Technical Notes

### Proficiency Field

The Orchestrator uses Proficiency to match incoming user queries to the right agent. It is generated from the agent's description and role content — check it by adding it as a column in the AI Agents list view. If blank or too generic, the Orchestrator will not route queries here and will return "No agents available."

### Why `state = New` (Not `In Progress`)?

The L1 Agent creates the Incident with `state = New` deliberately. The state transitions to `In Progress` only after NADI successfully extracts `u_extracted_error_code`. This ensures the Resolution Pathfinder Agentic Workflow trigger conditions are only met once all enrichment is complete:

```
Agentic Workflow trigger conditions:
  ✓ state = In Progress (2)        ← set after NADI extraction
  ✓ contact_type = chat            ← stamped by NAVA (Capability 01)
  ✓ u_extracted_error_code ≠ empty ← populated by NADI (Capability 04)
```

---

## Reference

- [ServiceNow Zurich — AI Agent Studio](https://www.servicenow.com/docs/bundle/zurich-intelligent-experiences/page/administer/now-assist-ai-agents/concept/ai-agent-studio.html)
- [ServiceNow Zurich — Create an AI agent](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/configure-next-best-action-agent.html)
- [ServiceNow Zurich — Add tools and information](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/add-tool-aia.html)
- [AI Agents FAQ and Troubleshooting — ServiceNow Community](https://www.servicenow.com/community/now-assist-articles/ai-agents-faq-and-troubleshooting/ta-p/3200454)
- [Getting Started with AI Agents — ServiceNow Community](https://www.servicenow.com/community/developer-articles/get-familiar-with-agentic-workflows-amp-ai-agent/ta-p/3326559)

---

## Next Step

Continue to [03 — Knowledge Graph](03-knowledge-graph.md) to configure the Knowledge Graph schema and User Graph that the L1 Agent queries in Step 2 of the Requestor Flow.

After completing Capability 03 and the extended Incident table setup, continue to [04 — Now Assist Document Intelligence](04-now-assist-document-intelligence.md) to configure NADI — which auto-triggers on images attached by the L1 Agent and populates `u_extracted_error_code`.
