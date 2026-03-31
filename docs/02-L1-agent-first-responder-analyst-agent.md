# 02 — L1 / First Responder Analyst Agent

> **Release:** Zurich | **Flow:** Requestor Flow — Phase 1 (Steps 2–5)
> **Source:** [ServiceNow Zurich — AI Agent Studio](https://www.servicenow.com/docs/bundle/zurich-intelligent-experiences/page/administer/now-assist-ai-agents/concept/ai-agent-studio.html) | [Create an AI agent](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/configure-next-best-action-agent.html) | [Add tools and information](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/add-tool-aia.html)

---

## What It Is

The **First Responder Operations Analyst Agent** is an **AI Agent** built in **AI Agent Studio**. It is the first automated intelligence that engages after NAVA routes a user message — taking over the conversation and orchestrating the Requestor Flow from initial user identification through to Incident creation.

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
Step 2: Agent triggered → Tool 1 (Knowledge Graph — User Graph)
        → silently identifies user role, department, affected CI
        → no questions asked to the user
        │
        ▼
Step 3: Tool 2 (File Upload — Troubleshooting Resolution Guide)
        → agent presents L1/L2/L3 severity-tiered troubleshooting guide
        → user works through diagnostic steps
        │
        ▼
   ┌────┴──────────────────────────────┐
   │  Decision: Issue resolved?         │
   └───────────────┬───────────────────┘
                   │
        YES → Deflect. No Incident created.
                   │
        NO  → Continue
                   │
                   ▼
Step 4: Tool 3 (Conversation Topic — Upload image x_nava_agentic_lab)
        → OOTB topic prompts user to upload error screenshot + device image
        → in-chat file picker rendered via Virtual Agent
        │
        ▼
Step 5: Tool 4 (Subflow — Create and submit Incident record with image upload(s))
        → fires after images are uploaded
        → creates Incident record with all four mandatory inputs
        → images attached at creation time
        │
        ▼
NADI auto-triggers on attachments → extracts u_extracted_error_code
(see 04-now-assist-document-intelligence.md)
```

> **Deflection is the optimal outcome.** If the Troubleshooting Guide resolves the issue, no Incident is created and no downstream automation fires.
>
> **Tool 4 fires only after Tool 3 completes.** The Incident creation subflow is deliberately sequenced after image upload — the record is created with images already attached, making it immediately NADI-ready.

---

## What the L1 Agent Enables

| Capability | Tool | How the Agent Delivers It |
|-----------|------|--------------------------|
| Silent user contextualisation | Tool 1 — Knowledge Graph | Queries User Graph to identify caller role, department, and affected CI with no user prompting |
| Guided initial troubleshooting | Tool 2 — File Upload (Troubleshooting Resolution Guide) | Severity-tiered guide (L1/L2/L3) presented in chat; deflects before escalation |
| Deflection | — | If guide resolves the issue, conversation ends — no Incident created |
| In-chat image upload | Tool 3 — Conversation Topic (OOTB) | OOTB Virtual Agent topic renders in-chat file picker for screenshots and device images |
| Enriched Incident creation | Tool 4 — Subflow | Creates and submits Incident after images captured — `state = New`, images attached, NADI-ready |

---

## Prerequisites

| Requirement | Detail |
|-------------|--------|
| Licence | Now Assist Pro+ or ITSM Pro+ |
| Plugin — Now Assist AI Agents | `sn_aia` — must be Active |
| Plugin — GAIC | `sn_gaic` — v11.2+ |
| AI Search | Must be enabled — agents return "No agents available" without it |
| Now Assist Panel | Must be enabled |
| Role | `sn_aia.admin` or `admin` |
| NAVA | Capability 01 complete — NAVA configured, AI Agents skill enabled in Assistant Designer |

---

## Lab Exercise — Steps to Build

### Step 1: Navigate to AI Agent Studio → Define the Specialty

Navigate to **All → AI Agent Studio → Create and manage → AI Agents** → click **New**

The wizard opens on **Step 1: Define the specialty**.

![L1 Agent — Define the specialty](../screenshots/L1-Agent2.png)

Fill in the agent identity:

| Field | Value |
|-------|-------|
| AI agent name | `First Responder Operations Analyst Agent` |
| AI agent description | Detailed role description for the LLM — covers the agent's scope (IT Operations issues: servers, appliances, network equipment, storage devices, OS, backup software, monitoring tools), behaviour modes (Diagnose, Classify & Act, Raise a Case, Communication), and escalation logic per severity tier. **This field is marked "Description for LLM"** — the LLM uses it to understand the agent's purpose and match incoming queries. |

> **Write a clear, distinct description.** The platform states: *"Writing a clear, distinct name and description is critical because the LLM relies on the wording to correctly identify and use this agent."* The description also drives the auto-generated **Proficiency** field which the Orchestrator uses to route queries to this agent.

Click **Generate details** to let Now Assist draft role and instructions from your description, or write them manually. Click **Save and continue** when done.

---

### Step 2: Add Tools and Information

The wizard advances to **Step 2: Add tools and information**.

![L1 Agent — Add tools and information](../screenshots/L1-Agent.png)

> This step is where all four tools are configured. The platform notes: *"Tools provide the essential functionality and data an AI agent needs to carry out its role. An AI agent selects a tool based on the tool's name and description, which need to be clearly written."*
>
> Use **+ Recommend Tools** to get AI-suggested tools based on your agent description, or add each tool manually via **Add tool →**.

---

#### Tool 1 — Knowledge Graph (User Graph)

From **Add tool →** select **Knowledge graph**.

![L1 Agent — Add tool dropdown (Knowledge graph)](../screenshots/L1-Agent-Tool1.png)

> Wait — `L1-Agent-Tool1.png` shows the dropdown with **Conversational topic** highlighted. This is the screenshot taken when adding Tool 3. See Tool 3 below for where this screenshot belongs.

The Knowledge Graph config dialog opens:

![L1 Agent — Knowledge Graph config](../screenshots/L1-agent-tool-kg1.png)

Configure the following fields:

| Field | Value |
|-------|-------|
| Select knowledge graph | `User Graph` |
| Name | `User related attributes Knowledge Graph` |
| Tool description (Description for LLM) | `Use this Knowledge Graph to retrieve relevant information about the user who you are currently engaged with.` |
| Query instruction | `Query to the knowledge graph. It should be a direct translation of request into a search query.` |
| Execution mode | **Autonomous** |

> **Why this tool first:** The agent silently queries the User Graph at the start of every conversation. This gives it the caller's role, department, manager, and related CI — so it can personalise its response and pre-populate the Incident record without asking the user any identity questions.

Click **Add**.

---

#### Tool 2 — File Upload (Troubleshooting Resolution Guide)

From **Add tool →** select **File upload**.

The Add file upload dialog opens:

![L1 Agent — Add file upload (top)](../screenshots/L1-agent-tool4-file-upload.png)

![L1 Agent — Add file upload (scrolled)](../screenshots/L1-agent-tool4-file-upload2.png)

Configure the following fields:

| Field | Value |
|-------|-------|
| Name | `Troubleshooting Resolution Guide` |
| Tool description (Description for LLM) | `This guide covers common enterprise backup appliance problems categorised into three severity tiers (L1, L2, L3), providing symptoms, likely causes, and diagnostic steps for each to help operations agents quickly triage issues and determine whether to resolve, escalate, or raise an Incident.` |
| Execution mode | **Autonomous** |
| Display output | **No** |
| Attachments | Upload `troubleshooting-resolution-guide.pdf` (114.0 KB) — the PDF is the guide the agent will search when troubleshooting |

> **Why this tool second:** This is the deflection gate. The agent uses this tool to walk the user through L1/L2/L3 severity-tiered diagnostic steps. If the user confirms resolution, the conversation ends with no ticket created. Only unresolved issues proceed to image upload and Incident creation.
>
> The attachment (PDF) is the grounding document the LLM reads from when presenting troubleshooting steps. It must be uploaded here — not linked externally.

Click **Add**.

---

#### Tool 3 — Conversation Topic (OOTB Image Upload)

From **Add tool →** select **Conversational topic**.

![L1 Agent — Add tool dropdown (Conversational topic)](../screenshots/L1-Agent-Tool1.png)

The Add a conversational topic dialog opens:

![L1 Agent — Conversational topic config](../screenshots/L1-agent-tool-conv-topic.png)

Configure the following fields:

| Field | Value |
|-------|-------|
| Select topic | `Upload image x_nava_agentic_lab` |
| Selected topic description | `Use this topic to prompt the user to upload an image` *(auto-populated, read-only)* |
| Name | `Upload image x_nava_agentic_lab` |
| Tool description (Description for LLM) | `This tool allows user to upload image to Now Assist Virtual Agent` |
| Execution mode | **Autonomous** |

> **Why this tool third:** This is the OOTB Virtual Agent conversation topic that renders the native in-chat image upload experience. The agent invokes it only after the user confirms the Troubleshooting Guide did not resolve their issue. The topic `Upload image x_nava_agentic_lab` is a scoped topic (`x_nava_agentic_lab`) — ensure this scope is active in your instance.

Click **Add**.

---

#### Tool 4 — Subflow (Create and Submit Incident Record)

From **Add tool →** select **Subflow**.

![L1 Agent — Add tool dropdown (Subflow)](../screenshots/L1-agent-tool2.png)

The Add a subflow dialog opens:

![L1 Agent — Subflow config (select + inputs)](../screenshots/L1-agent-tool2-subflow.png)

![L1 Agent — Subflow config (name + description)](../screenshots/L1-agent-tool2-subflow2.png)

Configure the following fields:

**Subflow selection and inputs:**

| Field | Value |
|-------|-------|
| Select subflow | `Create and submit Incident record with image upload(s) subflow` |
| category_type | `string` — incident category passed by the agent |
| chat_user_name | `string` — caller identified via Knowledge Graph (Tool 1) |
| short_description_input | `string` — user-reported issue description |
| ci_name_input | `string` — affected CI identified via Knowledge Graph (Tool 1) |
| work_notes_details | `string` — diagnostic notes from the troubleshooting session |

**Tool settings:**

| Field | Value |
|-------|-------|
| Name | `Create Incident Case` |
| Tool description (Description for LLM) | `The Create Incident Case Tool is a subflow that takes in four inputs in order to raise the Incident case. All of the four inputs are mandatory parameters.` |
| Execution mode | **Autonomous** |
| Display output | **No** |

> **Why this tool fourth:** This subflow creates and submits the Incident record **only after** Tool 3 (image upload) has completed. All five inputs are mandatory — the agent collects them across the conversation (caller from Tool 1, description from the user, CI from Tool 1, category and work notes from the troubleshooting session). The images uploaded via Tool 3 are attached to the record at creation time, making it immediately eligible for NADI processing.

Click **Add**.

---

### Step 3: Define Security Controls → Define User Access

The wizard advances to **Define security controls → Define user access**.

![L1 Agent — Define user access](../screenshots/L1-agent-user-access.png)

| Field | Value |
|-------|-------|
| User access | `Users with specific roles` |
| Role(s) | `itil` |

> This ACL controls who can interact with this agent via NAVA. Setting `itil` ensures only authenticated IT service desk users can invoke the agent — preventing general users from accessing IT triage functionality.

Click **Save and continue**.

---

### Step 4: Define Security Controls → Define Data Access

The wizard advances to **Define data access**.

![L1 Agent — Define data access](../screenshots/L1-agent-data-access.png)

| Field | Value |
|-------|-------|
| User identity type | `Dynamic user` |
| Approved role(s) | `itil` |

> **Dynamic user** means the agent runs as the logged-in user's identity — it inherits their roles and ACLs when reading and writing records. Setting `itil` as the approved role ensures the agent can read CMDB, create Incident records, and access user data within the bounds of the `itil` role. This prevents privilege escalation — the agent cannot perform actions the user themselves could not perform.

Click **Save and continue**.

---

### Step 5: Select Channels and Status

The wizard advances to **Select channels and status**.

![L1 Agent — Select channels and status](../screenshots/L1-agent-channel-status.png)

| Setting | Value |
|---------|-------|
| Engage via the Now Assist panel | **Allow: OFF** — agent is not discoverable from the Now Assist panel directly |
| Engage via Virtual Agent assistants | **Allow: ON** — agent is invoked via the Virtual Agent (NAVA) |
| Chat assistants | `Now Assist in Virtual Agent AlLab` |

> **Virtual Agent is the correct channel for the Requestor Flow.** The Requestor Flow is triggered through NAVA (the Virtual Agent), not directly from the Now Assist Panel. Enabling the Virtual Agent channel and selecting `Now Assist in Virtual Agent AlLab` ensures this agent is discoverable when users interact via the NAVA chat interface.
>
> The warning shown — *"this AI agent is designed to be part of an agentic workflow that users can discover in the Now Assist panel"* — is informational. For this lab, Virtual Agent is the intended channel.

Click **Save and continue** to complete the agent configuration.

---

### Step 6: Test the Agent

Navigate to **AI Agent Studio → Testing**.

1. Enter a test scenario: describe an IT infrastructure issue (e.g. a server unreachable, backup failure)
2. Verify **Tool 1** fires silently — agent responds with context about the user's role and affected CI without asking
3. Verify **Tool 2** presents tiered troubleshooting steps from the PDF guide
4. Confirm the deflection path — if user says resolved, conversation ends with no Incident
5. If unresolved — verify **Tool 3** triggers the in-chat image upload experience
6. After images uploaded — verify **Tool 4** creates the Incident with `state = New`, `contact_type = chat`, all five inputs populated, and images attached
7. Verify NADI triggers on the attachments and populates `u_extracted_error_code`

> If you get **"No agents available at the moment"**: check that AI Search is enabled, the agent is Active, the Virtual Agent channel is enabled with the correct assistant selected, and the **Proficiency** field is populated.

---

## Key Configuration Fields

| Field | Value |
|-------|-------|
| Agent name | `First Responder Operations Analyst Agent` |
| Role required to configure | `sn_aia.admin` |
| Tool 1 | Knowledge Graph — `User related attributes Knowledge Graph` |
| Tool 2 | File Upload — `Troubleshooting Resolution Guide` (PDF attached) |
| Tool 3 | Conversation Topic — `Upload image x_nava_agentic_lab` |
| Tool 4 | Subflow — `Create Incident Case` (5 mandatory inputs) |
| User access (ACL) | `itil` role |
| Data access | Dynamic user, approved role `itil` |
| Channel | Virtual Agent — `Now Assist in Virtual Agent AlLab` |
| Incident state on creation | `New` |
| Incident contact_type | `chat` |
| Images attached at creation | Yes — NADI triggers immediately |

---

## Technical Notes

### Tool Execution Order

The agent's instructions drive the tool execution sequence:

1. **Tool 1 (Knowledge Graph)** — fires at conversation start; builds user context silently
2. **Tool 2 (File Upload / Guide)** — presents troubleshooting steps; deflection gate
3. **Tool 3 (Conversation Topic)** — fires only if user confirms issue unresolved; triggers image upload
4. **Tool 4 (Subflow)** — fires only after Tool 3 completes; creates Incident with images attached

All four subflow inputs are mandatory — the agent gathers them across the conversation before invoking Tool 4.

### Why `state = New`?

The Subflow creates the Incident with `state = New`. The state transitions to `In Progress` only after NADI extracts `u_extracted_error_code` from the attached images. This ensures the Resolution Pathfinder Agentic Workflow trigger conditions are only met once all enrichment is complete:

```
Agentic Workflow trigger conditions:
  ✓ state = In Progress (2)        ← set after NADI extraction
  ✓ contact_type = chat            ← stamped by NAVA (Capability 01)
  ✓ u_extracted_error_code ≠ empty ← populated by NADI (Capability 04)
```

### Proficiency Field

The Orchestrator uses Proficiency to match user queries to this agent. It is auto-generated from the agent name and description — inspect it by adding it as a column in the AI Agents list view. If blank or too generic, the Orchestrator cannot route queries here and returns "No agents available."

---

## Reference

- [ServiceNow Zurich — AI Agent Studio](https://www.servicenow.com/docs/bundle/zurich-intelligent-experiences/page/administer/now-assist-ai-agents/concept/ai-agent-studio.html)
- [ServiceNow Zurich — Create an AI agent](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/configure-next-best-action-agent.html)
- [ServiceNow Zurich — Add tools and information](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/add-tool-aia.html)
- [AI Agents FAQ and Troubleshooting — ServiceNow Community](https://www.servicenow.com/community/now-assist-articles/ai-agents-faq-and-troubleshooting/ta-p/3200454)

---

## Next Step

Continue to [03 — Knowledge Graph](03-knowledge-graph.md) to configure the User Graph schema.

After completing Capability 03, continue to [04 — Now Assist Document Intelligence](04-now-assist-document-intelligence.md) to configure NADI — which auto-triggers on images attached by this agent and populates `u_extracted_error_code`.
