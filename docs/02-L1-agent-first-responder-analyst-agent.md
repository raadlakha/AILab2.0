# 02 — L1 / First Responder Analyst Agent

> **Release:** Zurich | **Flow:** Requestor Flow — Phase 1 (Steps 2–5)
> **Source:** [ServiceNow Zurich — Now Assist AI agents](https://www.servicenow.com/docs/bundle/zurich-intelligent-experiences/page/administer/now-assist-ai-agents/concept/ai-agent-studio.html) | [Create an AI agent](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/configure-next-best-action-agent.html) | [Add tools and information](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/add-tool-aia.html)

---

## What It Is

The **L1 / First Responder Analyst Agent** is an **AI Agent** built in **AI Agent Studio** and connected to an **Agentic Workflow** (Use Case). It is the first automated intelligence that engages after NAVA routes a user message — taking over the conversation and orchestrating the Requestor Flow from user intake through to Incident creation.

### Platform Structure — Zurich

In Zurich, the correct hierarchy is:

```
Agentic Workflow (Use Case)   ← the business problem / why
        │
        └── AI Agent           ← the worker / who
                │
                └── Tools      ← the actions / how
```

The **Agentic Workflow** defines the overall business objective (handle IT infrastructure issues reported via chat). The **AI Agent** is the specialist attached to that workflow. **Tools** are the platform capabilities the agent uses to execute its tasks.

Navigation: **All → AI Agent Studio → Create and manage → Use Cases → New**

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

### Step 1: Open AI Agent Studio

Navigate to **All → AI Agent Studio → Overview**

![L1 Agent — AI Agent Studio overview](../screenshots/L1-Agent.png)

> This is the central surface for all agentic AI configuration. From here you access **Agentic Workflows**, **AI Agents**, and the **Testing** console. The **Create and manage** tab gives you the list views.

---

### Step 2: Create the Agentic Workflow (Use Case)

Navigate to **All → AI Agent Studio → Create and manage → Use Cases → New**

Configure the agentic workflow:

| Field | Value |
|-------|-------|
| Name | `IT Infrastructure Issue — Requestor Flow` |
| Description | `Handles user-reported IT infrastructure issues via NAVA chat. Delivers troubleshooting guidance, collects supporting images, and creates an enriched Incident record for downstream agentic resolution.` |
| Goal | `Deflect the issue in chat if possible. If not, create an Incident with uploaded images attached so that Now Assist Document Intelligence can extract the error code and arm the Resolution Pathfinder workflow.` |

Under **Select channel and status**, set:

| Setting | Value |
|---------|-------|
| Display | **Engage via Now Assist Panel** — enables the agentic workflow to surface in NAVA |
| Status | Active |

Click **Save and continue**.

---

### Step 3: Create the AI Agent — Describe and Instruct

Within the Agentic Workflow, scroll to **Connect AI Agents** → click **Add AI Agent** → **New**.

![L1 Agent — Describe and instruct section](../screenshots/L1-Agent2.png)

Under the **Describe and instruct** section, configure:

| Field | Value |
|-------|-------|
| Name | `L1 First Responder Analyst` |
| Description | `Identifies the user via Knowledge Graph, delivers troubleshooting steps, collects device images, and creates an enriched Incident record.` |
| Proficiency | `This agent handles IT infrastructure issues reported via the NAVA chat interface. It searches the Knowledge Graph to identify the affected user and CI, presents troubleshooting steps, collects error screenshots and device label images, and creates Incident records when the issue cannot be deflected in chat.` |

> **Proficiency is critical.** The AI Agent Orchestrator uses this field to decide which agent to invoke for a given user query. If the proficiency is missing or vague, you will get "No agents available at the moment" errors in NAVA.

In the **AI agent role** field (max 2,000 characters):

```
You are the L1 First Responder Analyst, an AI assistant embedded in the IT support chat. Your role is to help users experiencing IT infrastructure issues — such as servers being unreachable, applications being inaccessible, or device failures. Always attempt to resolve the issue in chat before creating a ticket.
```

In the **Instructions** field (max 8,000 characters):

```
You are the L1 First Responder Analyst. Follow these steps in order:

Step 1: Use the KnowledgeGraphSearch tool to identify the user's role, department, and the affected CI based on their message. Do not ask the user for information that can be retrieved from the Knowledge Graph.

Step 2: Use the TroubleshootingGuide tool to retrieve the relevant troubleshooting steps for the reported issue type. Present the steps clearly and conversationally to the user.

Step 3: Ask the user: "Have these steps resolved your issue?" Wait for their response.
- If YES: Thank the user and close the conversation. Do not create an Incident.
- If NO: Proceed to Step 4.

Step 4: Ask the user to upload a screenshot of the error they are seeing AND a photo of the device label if applicable. Use the UploadImage tool to collect the files.

Step 5: Create an Incident record with:
  - state = New
  - contact_type = chat
  - caller_id = logged-in user
  - cmdb_ci = affected CI identified in Step 1
  - short_description = user-reported description
  - Attach all uploaded images to the Incident record
  Inform the user: "I have created Incident [number]. Our systems will investigate automatically."

Constraints:
- Always attempt deflection before creating a ticket.
- Never ask for information that can be retrieved from the Knowledge Graph.
- Keep responses concise and jargon-free.
- If the user does not upload images, still create the Incident. Note: NADI will not auto-trigger without attachments, but the Incident must not be blocked.
```

---

### Step 4: Add Tools — "Add tools and information" Tab

Click the **Add tools and information** tab to configure all tools for this agent.

#### Tool 1 — Knowledge Graph Search

![L1 Agent — Tool 1 overview](../screenshots/L1-Agent-Tool1.png)

![L1 Agent — Knowledge Graph tool config](../screenshots/L1-agent-tool-kg1.png)

| Field | Value |
|-------|-------|
| Tool name | `KnowledgeGraphSearch` |
| Tool type | **Knowledge Graph** |
| Schema | IT Infrastructure schema *(configured in Capability 03)* |
| Description | `Search the Knowledge Graph to identify the user's role, department, and affected CI. Also retrieves relevant KB articles and troubleshooting runbooks.` |

> The Knowledge Graph query at Step 2 of the Requestor Flow provides the user context that pre-populates Incident fields — reducing clarifying questions and improving deflection accuracy.

---

#### Tool 2 — Troubleshooting Guide (Subflow)

![L1 Agent — Tool 2 (Subflow)](../screenshots/L1-agent-tool2.png)

![L1 Agent — Subflow tool config](../screenshots/L1-agent-tool2-subflow.png)

![L1 Agent — Subflow tool detail](../screenshots/L1-agent-tool2-subflow2.png)

| Field | Value |
|-------|-------|
| Tool name | `TroubleshootingGuide` |
| Tool type | **Subflow** |
| Subflow | *(select the Troubleshooting Guide subflow from Flow Designer)* |
| Description | `Retrieve the troubleshooting guide steps relevant to the user's reported issue type. Returns a structured list of steps to deliver conversationally.` |

---

#### Tool 3 — Conversation Topic

![L1 Agent — Conversation Topic tool](../screenshots/L1-agent-tool-conv-topic.png)

| Field | Value |
|-------|-------|
| Tool name | `ConversationTopicGuide` |
| Tool type | **Conversation Topic** |
| Topic | *(select the Troubleshooting Guide Conversation Topic)* |
| Description | `Deliver structured troubleshooting guide steps through the Conversation Topic interface.` |

---

#### Tool 4 — File Upload (Image)

![L1 Agent — File Upload tool](../screenshots/L1-agent-tool4-file-upload.png)

![L1 Agent — File Upload tool detail](../screenshots/L1-agent-tool4-file-upload2.png)

| Field | Value |
|-------|-------|
| Tool name | `UploadImage` |
| Tool type | **File Upload** |
| Accepted types | `image/png, image/jpeg, image/jpg` |
| Description | `Allows the user to upload error screenshots and device label images in chat. Uploaded files are attached to the Incident record and trigger Now Assist Document Intelligence extraction.` |

> **This tool is the bridge to NADI.** Uploaded images attach to the Incident record. NADI (Capability 04) auto-triggers on those attachments and extracts `u_extracted_error_code`, which arms the Resolution Pathfinder Agentic Workflow.

---

### Step 5: Configure Channel and Status

Back in the Agentic Workflow settings, under **Select channel and status**:

![L1 Agent — Channel and status config](../screenshots/L1-agent-channel-status.png)

| Setting | Value |
|---------|-------|
| Engage via Now Assist Panel | ✅ Enabled |
| Virtual Agent experience | ✅ Toggled on — required for NAVA to invoke this agent |
| Status | **Active** |

> From the FAQ: *"Be sure the AI Agents skill is activated in the Assistants → Skills setup page, and that the Virtual Agent experience is toggled on for the AI Agent."* Without this, the agent will not be discoverable from NAVA even if all other settings are correct.

---

### Step 6: Configure Data Access

Navigate to the agent's **Data Access** settings.

![L1 Agent — Data access settings](../screenshots/L1-agent-data-access.png)

Ensure the agent has access to:

| Table | Purpose |
|-------|---------|
| `incident` | Create Incident records |
| `cmdb_ci` | Resolve affected CI from Knowledge Graph |
| `sys_user` | Identify caller from logged-in session |
| Extended Incident table | Write fields populated downstream by NADI |

---

### Step 7: Configure User Access

![L1 Agent — User access settings](../screenshots/L1-agent-user-access.png)

| Setting | Value |
|---------|-------|
| User roles | `now_assist_panel_user` — required to interact with the agent via NAVA |

> Per the official getting started guide: *"Click the down arrow, then add `now_assist_panel_user` in the User roles field."* This is the minimum role required for a user to invoke an AI Agent from the Now Assist Panel / NAVA chat.

---

### Step 8: Test the Agent

Navigate to **AI Agent Studio → Testing**.

1. Enter test scenario: *"I can't access the server prod-app-01"*
2. Verify the agent:
   - Queries the Knowledge Graph for user context
   - Presents troubleshooting guide steps
   - Asks if the steps resolved the issue
   - If "No" → prompts for image upload
   - Creates an Incident with `state = New` and `contact_type = chat`
3. Check the created Incident for:
   - `contact_type = chat` ✅
   - `state = New` ✅
   - Uploaded images attached ✅
4. Verify NADI triggers on the attachments and populates `u_extracted_error_code`

> If you get **"No agents available at the moment"**: check that AI Search is enabled, the Agentic Workflow and AI Agent are Active, the agent's **Proficiency** is filled in and descriptive, and the **Virtual Agent experience** is toggled on.

---

## Key Configuration Fields

| Field | Value for This Lab |
|-------|--------------------|
| Agentic Workflow name | IT Infrastructure Issue — Requestor Flow |
| AI Agent name | L1 First Responder Analyst |
| Role required to configure | `sn_aia.admin` |
| User role required to invoke | `now_assist_panel_user` |
| Tool 1 | KnowledgeGraphSearch (Knowledge Graph) |
| Tool 2 | TroubleshootingGuide (Subflow) |
| Tool 3 | ConversationTopicGuide (Conversation Topic) |
| Tool 4 | UploadImage (File Upload) |
| Virtual Agent experience | Toggled on |
| Incident state on creation | `New` |
| Incident contact_type | `chat` |
| Images attached | Yes — triggers NADI |

---

## Technical Notes

### Agentic Workflow vs AI Agent — When You Use Each

| Concept | What it is | Configured where |
|---------|-----------|-----------------|
| **Agentic Workflow** | The business problem / goal | AI Agent Studio → Create and manage → Use Cases |
| **AI Agent** | The specialist worker attached to the workflow | Connected within the Agentic Workflow |
| **Tools** | The platform actions the agent can take | "Add tools and information" tab on the AI Agent |

Think of the Agentic Workflow as the *why* and the AI Agent as the *who*. Tools are the *how*.

### Proficiency — Why It Matters

The AI Agent Orchestrator uses the agent's **Proficiency** field to match user queries to the right agent. This is separate from the **Role** (which gives the LLM its persona) and the **Instructions** (which govern step-by-step behavior). Without a detailed, specific Proficiency, the Orchestrator cannot route queries to this agent and will return "No agents available."

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
