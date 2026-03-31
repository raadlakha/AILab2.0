# 02 — L1 / First Responder Analyst Agent

> **Release:** Zurich | **Flow:** Requestor Flow — Phase 1 (Steps 2–5)
> **Source:** [ServiceNow Zurich — Now Assist AI agents](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/na-ai-agents.html) | [Create an AI agent](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/configure-next-best-action-agent.html) | [Add tools and information](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/add-tool-aia.html)

---

## What It Is

The **L1 / First Responder Analyst Agent** is a **Chat AI Agent** built in **AI Agent Studio**. It is the first automated intelligence that engages after NAVA receives a user message — taking over the conversation and orchestrating the full Requestor Flow from user intake through to Incident creation.

In Zurich, AI agents are created and managed entirely within **AI Agent Studio** (`All → Now Assist → AI Agent Studio`). The agent type used for handling chat conversations is **Chat**, which allows the agent to operate within the NAVA session, use tools, and take actions on the user's behalf.

The L1 Agent's responsibility in the Requestor Flow:

1. **Query the Knowledge Graph** — identify and contextualise the user (role, department, affected CI)
2. **Present a Troubleshooting Guide** — sourced via the Conversation Topic tool, delivered through the Conversation Topic interface
3. **Decision point** — if troubleshooting resolves the issue → deflect in chat (no ticket created). If not → continue.
4. **Request screenshots and device images** — prompt the user to upload via the file upload tool
5. **Create the Incident** — state = `New`, uploaded images attached to the record

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
   ┌────┴─────────────────────────────┐
   │  Decision: Issue resolved?        │
   └──────────────┬───────────────────┘
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

> **Deflection is the optimal outcome.** If the Troubleshooting Guide resolves the issue, no Incident is created and no downstream automation fires. Only when the user confirms the steps have not helped does the flow proceed to image upload and Incident creation.

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
| Role | `sn_aia.admin` or `admin` |
| NAVA | Capability 01 complete — NAVA must be configured and active |
| Knowledge Graph | Capability 03 complete — KG schema must be configured with User Graph |
| Conversation Topic | The Troubleshooting Guide Conversation Topic must exist (see below) |

---

## Lab Exercise — Steps to Build the L1 Agent

### Step 1: Open AI Agent Studio

Navigate to **All** → **Now Assist** → **AI Agent Studio** → **Create and manage** → **AI agents**

![L1 Agent — Main configuration view](../screenshots/L1-Agent.png)

> AI Agent Studio is the central surface for creating, configuring, and testing all AI agents. The L1 Agent is of type **Chat** — visible in the agent list alongside any other agents you have built.

---

### Step 2: Create the Agent — Basic Details

Click **Add** → **Chat** to create a new Chat AI Agent.

![L1 Agent — Configuration detail view](../screenshots/L1-Agent2.png)

Configure the basic identity:

| Field | Value |
|-------|-------|
| Name | `L1 First Responder Analyst` |
| Description | `Handles user-reported IT infrastructure issues via NAVA. Identifies the user via Knowledge Graph, delivers troubleshooting steps, collects device images, and creates an enriched Incident record.` |
| Agent type | **Chat** |

---

### Step 3: Write the Agent Role and Instructions

In the **Role** field (max 2,000 characters):

```
You are the L1 First Responder Analyst, an AI assistant embedded in the IT support chat. Your role is to help users experiencing IT infrastructure issues — such as servers being unreachable, applications being inaccessible, or device failures.
```

In the **Instructions** field (max 8,000 characters):

```xml
<instructions>
  <role>
    You are the L1 First Responder Analyst. Your goal is to resolve the user's issue before creating a ticket. If resolution is not possible in chat, create an enriched Incident record with supporting images attached.
  </role>

  <steps>
    <step order="1">
      Use the KnowledgeGraphSearch tool to identify the user's role, department, and the affected CI based on their message. Do not ask the user to provide this information if it can be retrieved from the Knowledge Graph.
    </step>
    <step order="2">
      Use the TroubleshootingGuide tool to retrieve the relevant troubleshooting steps for the reported issue type. Present the steps clearly and conversationally to the user.
    </step>
    <step order="3">
      Ask the user: "Have these steps resolved your issue?" Wait for their response.
      - If YES: Thank the user and close the conversation. Do not create an Incident.
      - If NO: Proceed to Step 4.
    </step>
    <step order="4">
      Ask the user to upload a screenshot of the error they are seeing AND a photo of the device label (if applicable). Use the UploadImage tool to collect the files.
    </step>
    <step order="5">
      Create an Incident record with:
        - state = New
        - contact_type = chat
        - caller_id = logged-in user
        - cmdb_ci = affected CI identified in Step 1
        - short_description = user-reported description
        - Attach all uploaded images to the Incident record
      Inform the user: "I have created Incident [number]. Our systems will investigate automatically."
    </step>
  </steps>

  <constraints>
    - Always attempt deflection before creating a ticket.
    - Never ask for information that can be retrieved from the Knowledge Graph.
    - Keep responses concise and jargon-free.
    - If the user does not upload images, still create the Incident — NADI will not auto-trigger without attachments, but the Incident should not be blocked.
  </constraints>
</instructions>
```

---

### Step 4: Configure Tool 1 — Knowledge Graph Search

Navigate to the **Tools** tab and add the first tool.

![L1 Agent — Tool 1 (Knowledge Graph)](../screenshots/L1-Agent-Tool1.png)

![L1 Agent — KG Tool detail](../screenshots/L1-agent-tool-kg1.png)

| Field | Value |
|-------|-------|
| Tool name | `KnowledgeGraphSearch` |
| Tool type | **Knowledge Graph** |
| Schema | IT Infrastructure schema *(configured in Capability 03)* |
| Description | `Search the Knowledge Graph to identify the user's role, department, and affected CI. Also used to find relevant KB articles and troubleshooting runbooks.` |

> The Knowledge Graph query at Step 2 of the Requestor Flow provides the user context that pre-populates the Incident fields — reducing the number of clarifying questions the agent needs to ask.

---

### Step 5: Configure Tool 2 — Troubleshooting Guide (Subflow)

Add the second tool.

![L1 Agent — Tool 2 (Subflow)](../screenshots/L1-agent-tool2.png)

![L1 Agent — Subflow tool config](../screenshots/L1-agent-tool2-subflow.png)

![L1 Agent — Subflow tool detail](../screenshots/L1-agent-tool2-subflow2.png)

| Field | Value |
|-------|-------|
| Tool name | `TroubleshootingGuide` |
| Tool type | **Subflow** |
| Subflow | *(select your Troubleshooting Guide subflow from Flow Designer)* |
| Description | `Retrieve the troubleshooting guide steps relevant to the user's reported issue type. Returns a structured list of steps to deliver to the user.` |

> The Troubleshooting Guide is delivered through this subflow rather than directly from a KB article — this allows the steps to be pre-formatted for conversational delivery and sourced from the file upload tool in the Conversation Topic.

---

### Step 6: Configure Tool 3 — Conversation Topic

Add the third tool.

![L1 Agent — Conversation Topic tool](../screenshots/L1-agent-tool-conv-topic.png)

| Field | Value |
|-------|-------|
| Tool name | `ConversationTopicGuide` |
| Tool type | **Conversation Topic** |
| Topic | *(select the Troubleshooting Guide Conversation Topic)* |
| Description | `Deliver structured troubleshooting guide steps through the Conversation Topic interface. Use when presenting the guide to the user.` |

---

### Step 7: Configure Tool 4 — File Upload (Image)

Add the fourth tool.

![L1 Agent — File Upload tool](../screenshots/L1-agent-tool4-file-upload.png)

![L1 Agent — File Upload tool detail](../screenshots/L1-agent-tool4-file-upload2.png)

| Field | Value |
|-------|-------|
| Tool name | `UploadImage` |
| Tool type | **File Upload** |
| Accepted types | `image/png, image/jpeg, image/jpg` |
| Description | `Allows the user to upload error screenshots and device label images in chat. Uploaded files are attached to the Incident record and trigger Now Assist Document Intelligence extraction.` |

> **This tool is the bridge to NADI.** When the user uploads images via this tool, the files are attached to the Incident record. Now Assist Document Intelligence (Capability 04 — NADI) auto-triggers on those attachments and extracts `u_extracted_error_code`, which arms the Resolution Pathfinder Agentic Workflow.

---

### Step 8: Configure Channel & Status

Navigate to the **Channels** tab.

![L1 Agent — Channel and status config](../screenshots/L1-agent-channel-status.png)

| Setting | Value |
|---------|-------|
| Channel | Virtual Agent (NAVA) |
| Status | **Active** |
| Display in Virtual Agent | ✅ Enabled |

> The agent must be set to **Active** and have **Display in Virtual Agent** enabled for the AI Agent Planner to route user messages to it from NAVA.

---

### Step 9: Configure Data Access

Navigate to the **Data Access** tab.

![L1 Agent — Data access settings](../screenshots/L1-agent-data-access.png)

Ensure the agent has read access to:

| Table | Purpose |
|-------|---------|
| `incident` | Create Incident records |
| `cmdb_ci` | Resolve affected CI from Knowledge Graph |
| `sys_user` | Identify caller from logged-in session |
| Extended Incident table | Write to `u_extracted_error_code` field (populated by NADI downstream) |

---

### Step 10: Configure User Access

Navigate to the **User Access** tab.

![L1 Agent — User access settings](../screenshots/L1-agent-user-access.png)

| Setting | Value |
|---------|-------|
| User access | All authenticated users |
| Required role | *(leave empty for all users, or restrict to specific roles)* |

> In this lab, the L1 Agent is accessible to all authenticated users via the Service Portal chat. In production, you may restrict by role (e.g., `itil_requester`).

---

### Step 11: Test the Agent

1. Navigate to **AI Agent Studio** → **Test**
2. Type: *"I can't access the server prod-app-01"*
3. Verify the agent:
   - Queries the Knowledge Graph for user context
   - Presents troubleshooting guide steps
   - Asks if the steps resolved the issue
   - If "No" → prompts for image upload
   - Creates an Incident with `state = New` and `contact_type = chat`
4. Check the created Incident for:
   - `contact_type = chat` ✅
   - `state = New` ✅
   - Uploaded images attached ✅
5. Verify NADI triggers on the attachments and populates `u_extracted_error_code`

---

## Key Configuration Fields

| Field | Value for This Lab |
|-------|--------------------|
| Agent type | Chat |
| Role required to configure | `sn_aia.admin` |
| Tool 1 | KnowledgeGraphSearch (Knowledge Graph) |
| Tool 2 | TroubleshootingGuide (Subflow) |
| Tool 3 | ConversationTopicGuide (Conversation Topic) |
| Tool 4 | UploadImage (File Upload) |
| Channel | Virtual Agent (NAVA) |
| Incident state on creation | `New` |
| Incident contact_type | `chat` |
| Images attached | Yes — triggers NADI |

---

## Technical Notes

### Agent Instructions — Why XML?

The agent instructions use Anthropic-style XML prompt conventions (`<instructions>`, `<steps>`, `<constraints>`) to impose a clear hierarchical structure on the LLM's reasoning. This:
- Prevents step-skipping (the agent will not jump from Step 1 to Step 5)
- Makes the deflection decision explicit and condition-gated
- Separates behavioral constraints from operational steps

In Zurich, the AI Agent instructions field supports up to 8,000 characters and accepts free-form text. XML structuring is a prompt engineering best practice — not a platform requirement.

### Why `state = New` (Not `In Progress`)?

The L1 Agent creates the Incident with `state = New` intentionally. The state transitions to `In Progress` only after NADI successfully extracts `u_extracted_error_code` from the uploaded images. This ensures the Resolution Pathfinder Agentic Workflow trigger conditions are only satisfied once all required enrichment is complete:

```
Agentic Workflow trigger conditions:
  ✓ state = In Progress (2)       ← set after NADI extraction
  ✓ contact_type = chat           ← stamped by NAVA (Capability 01)
  ✓ u_extracted_error_code ≠ empty ← populated by NADI (Capability 04)
```

### Deflection Path — No Agentic Workflow Fires

If the Troubleshooting Guide resolves the issue in chat, no Incident is created and the Agentic Workflow trigger is never evaluated. This is the **optimal outcome** — full deflection with zero ticket creation.

---

## Reference

- [ServiceNow Zurich — Now Assist AI agents](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/na-ai-agents.html)
- [ServiceNow Zurich — Create an AI agent](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/configure-next-best-action-agent.html)
- [ServiceNow Zurich — Add tools and information](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/add-tool-aia.html)
- [AI Agents FAQ and Troubleshooting — ServiceNow Community](https://www.servicenow.com/community/now-assist-articles/ai-agents-faq-and-troubleshooting/ta-p/3200454)

---

## Next Step

Continue to [03 — Knowledge Graph](03-knowledge-graph.md) to configure the Knowledge Graph schema and User Graph that the L1 Agent queries in Step 2 of the Requestor Flow.

After completing Capabilities 03 and the extended Incident table setup, continue to [04 — Now Assist Document Intelligence](04-now-assist-document-intelligence.md) to configure NADI — which auto-triggers on the images attached by the L1 Agent and populates `u_extracted_error_code`.
