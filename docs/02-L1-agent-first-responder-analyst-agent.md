# 02 — First Responder Operations Analyst Agent

> **Release:** Zurich | **Flow:** Requestor Flow — Phase 1
> **Source:** [AI Agent Studio](https://www.servicenow.com/docs/bundle/zurich-intelligent-experiences/page/administer/now-assist-ai-agents/concept/ai-agent-studio.html) | [Create an AI agent](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/configure-next-best-action-agent.html) | [Add tools and information](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/add-tool-aia.html)

---

## What It Is

The **First Responder Operations Analyst Agent** is an AI Agent built in AI Agent Studio. It is the first automated intelligence that engages after NAVA routes a user message — handling the conversation from initial user identification through to Incident creation.

This section covers building the **AI Agent** only. The Agentic Workflow that connects this agent to the NAVA trigger is configured separately.

---

## Role in the Requestor Flow

```
NAVA receives user message (contact_type = chat stamped)
        │
        ▼
Tool 1 — Knowledge Graph (User Graph)
        Silently identifies user role, department, affected CI
        No questions asked to the user
        │
        ▼
Tool 2 — File Upload (Troubleshooting Resolution Guide)
        L1/L2/L3 severity-tiered troubleshooting guide presented in chat
        User works through diagnostic steps
        │
        ▼
   Issue resolved?
   YES → Deflect. Conversation ends. No Incident created.
   NO  → Continue
        │
        ▼
Tool 3 — Conversation Topic (Upload image x_nava_agentic_lab)
        OOTB topic prompts user to upload error screenshot + device image
        In-chat file picker rendered via Virtual Agent
        │
        ▼
Tool 4 — Subflow (Create and submit Incident record with image upload(s))
        Fires after images are uploaded
        Creates Incident record with all mandatory inputs
        Images attached at creation time
        │
        ▼
NADI auto-triggers on attachments → extracts u_extracted_error_code
```

> **Tool 4 fires only after Tool 3 completes.** The Incident creation subflow is sequenced after image upload so the record is created with images already attached — making it immediately NADI-ready.

---

## What the Agent Enables

| Capability | Tool | How |
|-----------|------|-----|
| Silent user contextualisation | Tool 1 — Knowledge Graph | Queries User Graph to get caller role, department, CI — no prompting of the user |
| Guided troubleshooting | Tool 2 — File Upload | L1/L2/L3 severity-tiered guide from attached PDF; deflection gate |
| Deflection | — | Issue resolved in chat → conversation ends, no Incident created |
| In-chat image upload | Tool 3 — Conversation Topic | OOTB topic renders native in-chat file picker |
| Enriched Incident creation | Tool 4 — Subflow | Creates and submits Incident after images captured; `state = New`; images attached; NADI-ready |

---

## Prerequisites

| Requirement | Detail |
|-------------|--------|
| Licence | Now Assist Pro+ or ITSM Pro+ |
| Plugin `sn_aia` | Now Assist AI Agents — must be Active |
| Plugin `sn_gaic` | v11.2+ |
| AI Search | Must be enabled — agents return "No agents available" without it |
| Role | `sn_aia.admin` or `admin` |
| NAVA | Capability 01 complete |

---

## Lab Exercise — Steps to Build

### Wizard Step 1 — Define the Specialty

Navigate to **All → AI Agent Studio → Create and manage → AI Agents → New**.

The wizard opens on **Define the specialty**.

![Define the specialty — agent name and description](../screenshots/L1-Agent2.png)

The page header reads: *"Using clear, precise language, write the name, description, role, and list of steps this AI agent completes."*

Configure the following fields:

| Field | Value |
|-------|-------|
| AI agent name | `First Responder Operations Analyst Agent` |
| AI agent description *(Description for LLM)* | `You are the **First Responder Operations Analyst**, the first point of contact for IT Operations issues covering servers, appliances, network equipment, storage devices, operating systems, backup software, and monitoring tools.` `**Diagnose** — Ask focused clarifying questions to identify the issue. Use only the *Problem and Troubleshooting Guide* — never infer or fabricate solutions outside it.` `**Classify & Act** — Categorise every issue into a severity tier and respond accordingly: L1 — Walk the user through all documented resolution steps. If the issue persists after all steps are exhausted, raise a case. L2 & L3 — Inform the user that specialist support is required. Collect incident data and raise a case.` `**Raise a Case** — Collect all required incident fields before submission. Never raise a case with incomplete information.` `**Communication** — [continues below the fold]` |

> **This field is labelled "Description for LLM"** — the LLM reads it to understand the agent's scope, behaviour modes, and escalation logic. A clear description is critical: the platform warns *"Writing a clear, distinct name and description is critical because the LLM relies on the wording to correctly identify and use this agent."* It also auto-generates the **Proficiency** field that the Orchestrator uses to route user queries to this agent.

Click **Generate details** to let Now Assist draft the role and instructions from the description, or author them manually. Click **Save and continue**.

---

### Wizard Step 2 — Add Tools and Information

The wizard advances to **Add tools and information**.

![Add tools and information — empty tool list](../screenshots/L1-Agent.png)

The page reads: *"Tools provide the essential functionality and data an AI agent needs to carry out its role. An AI agent selects a tool based on the tool's name and description, which need to be clearly written."*

Four tools must be added in order. Use the **Add tool ▼** dropdown to select the tool type for each.

---

#### Tool 1 — Knowledge Graph

From **Add tool ▼** select **Knowledge graph**.

The **Add a Knowledge graph** dialog opens:

![Add a Knowledge graph dialog](../screenshots/L1-agent-tool-kg1.png)

Configure the following fields:

| Field | Value |
|-------|-------|
| Select knowledge graph | `User Graph` |
| Name | `User related attributes Knowledge Graph` |
| Tool description *(Description for LLM)* | `Use this Knowledge Graph to retrieve relevant information about the user who you are currently engaged with.` |
| Query instruction | `Query to the knowledge graph. It should be a direct translation of request into a search query.` |
| Execution mode | **Autonomous** |

> **Why:** This tool fires silently at the start of every conversation. The User Graph gives the agent the caller's role, department, manager, and affected CI — allowing it to personalise the conversation and pre-populate Incident fields without asking the user to identify themselves.
>
> Execution mode **Autonomous** means the agent calls this tool without asking the user for permission first.

Click **Add**.

---

#### Tool 2 — File Upload (Troubleshooting Resolution Guide)

From **Add tool ▼** select **File upload**.

The **Add file upload** dialog opens:

![Add file upload dialog — top](../screenshots/L1-agent-tool4-file-upload.png)

![Add file upload dialog — scrolled showing attachment](../screenshots/L1-agent-tool4-file-upload2.png)

Configure the following fields:

| Field | Value |
|-------|-------|
| Name | `Troubleshooting Resolution Guide` |
| Tool description *(Description for LLM)* | `This guide covers common enterprise backup appliance problems categorised into three severity tiers (L1, L2, L3), providing symptoms, likely causes, and diagnostic steps for each to help operations agents quickly triage issues and determine whether to resolve, escalate, or raise an Incident.` |
| Execution mode | **Autonomous** |
| Display output | **No** |
| Attachments | Upload `troubleshooting-resolution-guide.pdf` (114.0 KB) |

> **Why:** This is the deflection gate. The agent retrieves and presents the relevant troubleshooting steps from the attached PDF based on the user's reported issue. If the user confirms the steps resolved their issue, the conversation ends — no Incident is created. Only unresolved issues proceed to image collection and Incident creation.
>
> The PDF attachment is the grounding document for all troubleshooting responses. It must be uploaded here; the agent cannot reference it from an external URL. Supported formats: PDF, DOCX, TXT (max 5 files, 5 MB each).

Click **Add**.

---

#### Tool 3 — Conversational Topic (Image Upload)

From **Add tool ▼** select **Conversational topic**.

![Add tool dropdown — Conversational topic highlighted](../screenshots/L1-Agent-Tool1.png)

The **Add a conversational topic** dialog opens:

![Add a conversational topic dialog](../screenshots/L1-agent-tool-conv-topic.png)

Configure the following fields:

| Field | Value |
|-------|-------|
| Select topic | `Upload image x_nava_agentic_lab` |
| Selected topic description *(read-only, auto-populated)* | `Use this topic to prompt the user to upload an image` |
| Name | `Upload image x_nava_agentic_lab` |
| Tool description *(Description for LLM)* | `This tool allows user to upload image to Now Assist Virtual Agent` |
| Execution mode | **Autonomous** |

> **Why:** This OOTB Virtual Agent conversation topic renders the native in-chat file picker. The agent invokes it only after the user confirms the Troubleshooting Guide did not resolve their issue. The uploaded images are what NADI processes in the next capability.
>
> The topic `Upload image x_nava_agentic_lab` is scoped to `x_nava_agentic_lab` — ensure this scope is active and the Virtual Agent topic is published in your instance before testing.

Click **Add**.

---

#### Tool 4 — Subflow (Incident Creation)

From **Add tool ▼** select **Subflow**.

![Add tool dropdown — Subflow highlighted](../screenshots/L1-agent-tool2.png)

The **Add a subflow** dialog opens:

![Add a subflow — subflow selection and input fields](../screenshots/L1-agent-tool2-subflow.png)

![Add a subflow — tool name and description](../screenshots/L1-agent-tool2-subflow2.png)

**Subflow selection and mandatory inputs:**

| Input field | Internal name | Data type | What it carries |
|------------|--------------|-----------|----------------|
| Select subflow | — | — | `Create and submit Incident record with image upload(s) subflow` |
| Category type | `category_type` | string | Incident category derived from the conversation |
| Chat user name | `chat_user_name` | string | Caller name — retrieved by Tool 1 (Knowledge Graph) |
| Short description input | `short_description_input` | string | User-reported issue description |
| CI name input | `ci_name_input` | string | Affected CI — retrieved by Tool 1 (Knowledge Graph) |
| Work notes details | `work_notes_details` | string | Diagnostic notes from the troubleshooting session |

**Tool settings:**

| Field | Value |
|-------|-------|
| Name | `Create Incident Case` |
| Tool description *(Description for LLM)* | `The Create Incident Case Tool is a subflow that takes in four inputs in order to raise the Incident case. All of the four inputs are mandatory parameters.` |
| Execution mode | **Autonomous** |
| Display output | **No** |

> **Why:** This subflow creates and submits the Incident record — but only after Tool 3 (image upload) has completed. The sequencing is deliberate: the Incident is created with images already attached so NADI can trigger immediately on the attachments and extract `u_extracted_error_code`.
>
> All inputs are mandatory. The agent collects them across the conversation: `chat_user_name` and `ci_name_input` come from Tool 1 (Knowledge Graph); `short_description_input` comes from the user's message; `category_type` and `work_notes_details` come from the troubleshooting session. The subflow will not execute until all inputs are populated.

Click **Add**.

---

### Wizard Step 3 — Define User Access

The wizard advances to **Define security controls → Define user access**.

![Define user access](../screenshots/L1-agent-user-access.png)

| Field | Value |
|-------|-------|
| User access | `Users with specific roles` |
| Role(s) | `itil` |

> This ACL controls who can interact with this agent via NAVA. `itil` restricts access to authenticated IT service desk users — general platform users cannot invoke this agent.

Click **Save and continue**.

---

### Wizard Step 4 — Define Data Access

The wizard advances to **Define data access**.

![Define data access](../screenshots/L1-agent-data-access.png)

| Field | Value |
|-------|-------|
| User identity type | `Dynamic user` |
| Approved role(s) | `itil` |

> **Dynamic user** means the agent executes as the identity of the logged-in user — it inherits their roles and ACLs for every read and write operation. The `itil` approved role sets the upper bound: even if the logged-in user has more permissive roles, the agent operates within `itil` limits. This prevents privilege escalation — the agent cannot perform any action the user could not perform themselves.

Click **Save and continue**.

---

### Wizard Step 5 — Select Channels and Status

The wizard advances to **Select channels and status**.

![Select channels and status](../screenshots/L1-agent-channel-status.png)

| Setting | Value |
|---------|-------|
| Engage via the Now Assist panel | **Allow: OFF** |
| Engage via Virtual Agent assistants | **Allow: ON** |
| Chat assistants | `Now Assist in Virtual Agent AlLab` |

> The Requestor Flow is triggered through NAVA (Virtual Agent), not directly from the Now Assist Panel — so Virtual Agent is the correct channel here. The **Now Assist panel** toggle is left OFF deliberately.
>
> Under **Choose chat assistants**, select `Now Assist in Virtual Agent AlLab` — this is the Virtual Agent assistant configured in Capability 01. The agent will only be discoverable in the NAVA sessions linked to this assistant.
>
> The yellow warning — *"this AI agent is designed to be part of an agentic workflow that users can discover in the Now Assist panel"* — is informational. It does not require action for this lab.

Click **Save and continue** to complete the agent configuration.

---

### Wizard Step 6 — Test the Agent

Navigate to **AI Agent Studio → Testing**.

1. Enter a test scenario describing an IT infrastructure issue (e.g. *"I can't reach the backup server"*)
2. **Tool 1** — confirm the agent identifies the caller's role and affected CI silently, without asking
3. **Tool 2** — confirm the agent presents L1/L2/L3 troubleshooting steps from the PDF guide
4. Respond *"resolved"* — confirm the conversation ends with no Incident created (deflection path)
5. Re-run the test. Respond *"not resolved"* — confirm Tool 3 renders the in-chat image upload prompt
6. Upload a test image — confirm Tool 4 creates an Incident with `state = New`, `contact_type = chat`, all inputs populated, and the image attached
7. Navigate to the created Incident — confirm NADI has triggered and `u_extracted_error_code` is populated

> If you get **"No agents available at the moment"**: verify AI Search is enabled, the agent status is Active, the Proficiency field is populated, and the Virtual Agent channel is enabled with `Now Assist in Virtual Agent AlLab` selected.

---

## Key Configuration Summary

| Field | Value |
|-------|-------|
| Agent name | `First Responder Operations Analyst Agent` |
| Type | Chat |
| Tool 1 | Knowledge Graph — `User related attributes Knowledge Graph` — User Graph |
| Tool 2 | File Upload — `Troubleshooting Resolution Guide` — `troubleshooting-resolution-guide.pdf` |
| Tool 3 | Conversational Topic — `Upload image x_nava_agentic_lab` |
| Tool 4 | Subflow — `Create Incident Case` — `Create and submit Incident record with image upload(s) subflow` |
| User access | `Users with specific roles` → role: `itil` |
| Data access | `Dynamic user` → approved role: `itil` |
| Channel | Virtual Agent — `Now Assist in Virtual Agent AlLab` |
| Now Assist panel | OFF |
| Incident `state` on creation | `New` |
| Incident `contact_type` | `chat` |
| Images attached at creation | Yes — NADI triggers immediately |

---

## Technical Notes

### Tool Execution Order

The agent's instructions govern when each tool is called:

1. **Tool 1 (Knowledge Graph)** — fires at conversation start; silently builds user context
2. **Tool 2 (File Upload)** — presents troubleshooting guide; deflection gate before escalation
3. **Tool 3 (Conversational Topic)** — fires only if user confirms issue is unresolved; triggers image upload
4. **Tool 4 (Subflow)** — fires only after Tool 3 completes; creates Incident with images already attached

All subflow inputs are mandatory — the agent accumulates them across the conversation before invoking Tool 4.

### Why `state = New`?

The subflow creates the Incident with `state = New`. The state advances to `In Progress` only after NADI extracts `u_extracted_error_code` from the attached images. This gates the Resolution Pathfinder Agentic Workflow, which only triggers when all three conditions are met:

```
✓ state = In Progress (2)        ← set by NADI after extraction
✓ contact_type = chat            ← stamped by NAVA (Capability 01)
✓ u_extracted_error_code ≠ empty ← populated by NADI (Capability 04)
```

### Proficiency Field

Auto-generated from the agent name and description. The Orchestrator uses it to match incoming user messages to this agent. Add it as a column in the AI Agents list view to inspect it. If blank or too generic, the Orchestrator cannot route queries here.

---

## Reference

- [ServiceNow Zurich — AI Agent Studio](https://www.servicenow.com/docs/bundle/zurich-intelligent-experiences/page/administer/now-assist-ai-agents/concept/ai-agent-studio.html)
- [ServiceNow Zurich — Create an AI agent](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/configure-next-best-action-agent.html)
- [ServiceNow Zurich — Add tools and information](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/add-tool-aia.html)
- [AI Agents FAQ and Troubleshooting](https://www.servicenow.com/community/now-assist-articles/ai-agents-faq-and-troubleshooting/ta-p/3200454)

---

## Next Steps

→ [03 — Knowledge Graph](03-knowledge-graph.md) — configure the User Graph schema queried by Tool 1

→ [04 — Now Assist Document Intelligence](04-now-assist-document-intelligence.md) — configure NADI, which auto-triggers on Incident attachments created by Tool 4 and populates `u_extracted_error_code`
