# 05 — First Responder Operations Analyst (Chat AI Agent)

> **Source:** [ServiceNow Zurich Documentation — Now Assist AI agents](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/na-ai-agents.html) | [Create an AI agent](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/configure-next-best-action-agent.html)

## What It Is

The **First Responder Operations Analyst** is a **Chat AI Agent** — an AI agent that operates on behalf of the user within the NAVA conversation. In Zurich, AI agents are created in **AI Agent Studio** and the agent type that handles chat conversations is **Chat**.

It is the first automated intelligence that engages when someone reports a server issue. Its job is to:
1. Understand the user's problem through natural conversation
2. Search the Knowledge Graph for an immediate answer
3. Ask the user to upload a screenshot and extract the error code via Document Intelligence
4. Use the **ExtractIncidentDetails** Flow Action to structure incident information
5. Create an Incident record with all enriched context
6. If it can resolve the issue immediately, provide the answer — otherwise, create the Incident (which triggers the Resolution Pathfinder Agentic Workflow)

---

## When to Use

- When the user-facing intake process needs AI reasoning — not just a form
- When you want the agent to decide dynamically what questions to ask based on what the user said
- When structured data (error codes, server names, CI references) needs to be collected before creating a ticket
- When deflection is possible — the agent should try to resolve before escalating

---

## Customer Value

| Business Outcome | How the First Responder Agent Delivers It |
|-----------------|------------------------------------------|
| Structured incident creation | Agent extracts all required fields before creating the Incident — no incomplete tickets |
| Immediate deflection attempt | Agent searches Knowledge Graph first; only creates ticket if no resolution found |
| User-friendly intake | Conversational, not form-based |
| Consistent triage | Every server issue follows the same enrichment and escalation logic |
| Faster MTTR | Incident arrives at the resolution team with error code, CI, and KB search already done |

---

## How It Works — Architecture

```
NAVA Chat
        │  "The production server is unreachable"
        ▼
First Responder AI Agent (Chat type)
        │
        ├──► Tool: KnowledgeGraphSearch
        │         Query: user's description
        │         If result found → respond with KB steps → offer to resolve
        │
        ├──► Tool: UploadSupportingDocument (File Upload)
        │         User uploads screenshot
        │         Now Assist in Document Intelligence extracts error code
        │
        ├──► Tool: ExtractIncidentDetails (Flow Action)
        │         Returns: server name, error code, category, urgency
        │
        └──► Action: Create Incident record
                  Fields: short_description, category, CI, u_extracted_error_code
                  State: In Progress
                  Channel: Chat
                  → Triggers Resolution Pathfinder Agentic Workflow
```

---

## How to Build

### Prerequisites

- AI Agent Studio enabled (Now Assist Pro+)
- Capabilities 01–04 complete
- Role: `sn_aia.admin` or `admin`

---

## Lab Exercise — Steps to Build

### Step 1: Open AI Agent Studio

1. Navigate to **All** > search **AI Agent Studio**
2. Click **Create and manage** > **AI agents**
3. Click the **Add** drop-down

### Step 2: Select Agent Type

In Zurich, the available agent types are:
- **Chat** — handles conversations in NAVA and messaging channels
- **Voice** — handles voice interactions
- **External** — integrates with external AI platforms

Select **Chat**.

### Step 3: Configure Agent Identity

| Field | Value |
|-------|-------|
| Name | `First Responder Operations Analyst` |
| Description | `Handles user-reported IT infrastructure issues via NAVA. Collects context, searches Knowledge Graph, extracts error codes, and creates enriched Incident records.` |
| Agent type | **Chat** |

### Step 4: Define the Specialty

In the **Specialty** section, configure how the agent behaves:

| Field | Value |
|-------|-------|
| Name | `IT Infrastructure Issue Handler` |
| Description | `Handles server inaccessibility issues. Searches KB, collects error codes, creates Incidents.` |
| AI agent role | `You are the First Responder Operations Analyst, an AI assistant embedded in the IT support chat. Your role is to help users experiencing IT infrastructure issues.` |
| Long-term memory | Disabled (per-conversation only) |
| AI agent learning | Enabled |

### Step 5: Write the Agent Instructions (System Prompt)

In the **Instructions** field, enter:

```
You are the First Responder Operations Analyst, an AI assistant embedded in the IT support chat. Your role is to help users who are experiencing IT infrastructure issues — such as servers being unreachable, applications being inaccessible, or network connectivity failures.

When a user reports an issue:
1. Acknowledge the issue and ask one clarifying question if needed (e.g., "Which server or application is affected?")
2. Search the knowledge base using the KnowledgeGraphSearch tool with the user's description
3. If a relevant resolution is found, present it clearly and ask if it helped
4. Ask the user to upload a screenshot or error log if they have one, using the UploadSupportingDocument tool
5. Use the ExtractIncidentDetails flow action to structure the incident information
6. Create an Incident record with all collected information
7. Inform the user their ticket has been created and they will be updated

Keep responses concise and supportive. Do not use jargon. If you cannot find a resolution, tell the user you are escalating to the resolution team.
```

### Step 6: Add Tools to the Agent

Navigate to the **Tools** tab and add the following:

#### Tool 1: Knowledge Graph Search

| Field | Value |
|-------|-------|
| Tool name | `KnowledgeGraphSearch` |
| Type | Knowledge Graph |
| Schema | IT Infrastructure Runbooks schema *(configured in Cap. 03)* |
| Description | `Search the internal knowledge base and Knowledge Graph for troubleshooting steps and resolution guides` |

#### Tool 2: File Upload

| Field | Value |
|-------|-------|
| Tool name | `UploadSupportingDocument` |
| Type | File Upload |
| Description | `Allows the user to upload screenshots or log files. Triggers Now Assist in Document Intelligence to extract error codes.` |
| Accepted types | `image/*, application/pdf` |

#### Tool 3: Flow Action — Extract Incident Details

*(Built in Capability 07 — reference it here once created)*

| Field | Value |
|-------|-------|
| Tool name | `ExtractIncidentDetails` |
| Type | Flow Action |
| Flow Action | `Extract Incident Details` *(configured in Cap. 07)* |
| Description | `Extracts structured incident information: affected CI, error code, category, urgency` |

### Step 7: Configure Incident Creation Action

1. In the agent, under **Actions**, add **Record operation**
2. Configure:

| Field | Mapping |
|-------|---------|
| Operation | Create |
| Table | `incident` |
| short_description | Summarised from conversation |
| description | Full conversation context |
| category | `Infrastructure` |
| cmdb_ci | Extracted server name |
| u_extracted_error_code | From Document Intelligence |
| state | `2` (In Progress) |
| contact_type | `chat` |
| caller_id | Logged-in user |

### Step 8: Connect to Conversational Topic

1. Navigate back to **Virtual Agent** > **Designer** > open `IT Infrastructure Issue` topic
2. Under **Now Assist** tab > **AI Agent**, select `First Responder Operations Analyst`
3. **Publish** the topic

### Step 9: Test End-to-End

1. Open Service Portal chat
2. Type: *"I can't access server prod-app-01"*
3. Verify the agent:
   - Responds conversationally
   - Searches Knowledge Graph and returns relevant content
   - Asks if you want to upload a screenshot
   - Creates an Incident with populated fields
4. Check the Incident record for `u_extracted_error_code` and `contact_type = chat`

---

## Key Configuration Fields

| Field | Value for This Lab |
|-------|--------------------|
| Agent type | Chat |
| Role required | `sn_aia.admin` |
| Tools | KnowledgeGraphSearch, UploadSupportingDocument, ExtractIncidentDetails |
| Output action | Create Incident (state=In Progress, contact_type=chat) |
| Incident trigger fields | state=In Progress, contact_type=chat, u_extracted_error_code≠empty |

---

## Reference

- [ServiceNow Zurich — Now Assist AI agents](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/na-ai-agents.html)
- [Create an AI agent](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/configure-next-best-action-agent.html)
- [Add tools and information](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/add-tool-aia.html)

---

## Next Step

Continue to [06 — Resolution Pathfinder](06-resolution-pathfinder-ai-agent.md) to build the AI agent and Agentic Workflow that activates once the Incident is created.
