# 02 — L1 First Responder Operations Analyst Agent

> **Release:** Zurich | **Flow:** Requestor Flow — Phase 1 (Step 3)

***

## What It Is

The **First Responder Operations Analyst Agent** is an AI Agent built in AI Agent Studio. It is the first automated intelligence that engages after NAVA routes a user message — handling the conversation from initial user identification through to Incident creation.

This section covers building the **AI Agent** only.

***

## Role in the Requestor Flow

```
NAVA receives user message (contact_type = chat stamped)
        │
        ▼
Tool 1 — Knowledge Graph (User Graph)
        Silently identifies who the user is and related information about the user
        No questions asked to the user
        │
        ▼
Tool 2 — File Upload (Troubleshooting Resolution Guide)
        L1/L2/L3 severity-tiered troubleshooting guide provided to the AI Agent to accurately categorise,
        diagnose and recommend suitable troubleshooting and resolution steps
        User works through diagnostic steps
        │
        ▼
   Issue resolved?
   YES → Deflect. Conversation ends. No Incident created.
   NO  → Continue
        │
        ▼
Tool 3 — Conversation Topic (Upload image Topic)
        OOTB topic prompts user to upload error screenshot + device image
        In-chat file picker rendered via Virtual Agent
        │
        ▼
Tool 4 — Subflow (Create and submit Incident record with image upload(s))
        Gets triggered after images are uploaded
        Creates Incident record with all mandatory inputs
        Images attached at creation time
        │
        ▼
Now Assist in Document Intelligence (NADI) auto-triggers on attachments
→ extracts: error code, model details, product bar code, product name, serial number
```

> **Tool 4 fires only after Tool 3 completes.** The Incident creation subflow is sequenced after image upload so the record is created with images already attached — making it immediately NADI-ready.

***

## What the Agent Enables

| Capability | Tool | How |
| --- | --- | --- |
| Silent user contextualisation | Tool 1 — Knowledge Graph | Queries User Graph to get attributes related to the user |
| Guided troubleshooting | Tool 2 — File Upload | L1/L2/L3 severity-tiered guide from attached PDF |
| Deflection | — | Issue resolved in chat → conversation ends, no Incident created |
| In-chat image upload | Tool 3 — Conversation Topic | OOTB topic renders native in-chat image upload picker |
| Enriched Incident creation | Tool 4 — Subflow | Creates Incident after images captured; images attached; NADI-ready |

***

## Prerequisites

| Requirement | Detail |
| --- | --- |
| New Virtual Agent assistant created | Complete steps |
| Duplicate 'Upload Image' topic | Complete steps |

***

## Lab Exercise — Steps to Build

### Step 1 — Define the Specialty

Navigate to **All → AI Agent Studio → Create and manage → AI Agents → New**.

The wizard opens on **Define the specialty**.

![Define the specialty — agent name and description](<../.gitbook/assets/L1-Agent (1).png>)

The page instructs: _"Using clear, precise language, write the name, description, role, and list of steps this AI agent completes. Writing a clear, distinct name and description is critical because the LLM relies on the wording to correctly identify and use this agent."_

Configure the following fields:

| Field | Value |
| --- | --- |
| **AI agent name** | `First Responder Operations Analyst Agent` |
| **AI agent description** _(Description for LLM)_ | See full prompt below |
| **AI agent role** | See full prompt below |
| **List of Steps** | See full prompt below |

**AI agent description** _(Description for LLM)_ — Expectation: SC to build the prompt for the description

**AI agent role** — Expectation: SC to build the prompt for the role

**List of Steps** — Expectation: SC to build the prompt for the list of steps

***

### Step 2 — Add Tools and Information

The wizard advances to **Add tools and information**.

![Add tools and information — tool list](<../.gitbook/assets/L1-Agent2 (1).png>)

The page reads: _"Tools provide the essential functionality and data an AI agent needs to carry out its role. An AI agent selects a tool based on the tool's name and description, which need to be clearly written."_

Four tools must be added. Use the **Add tool ▼** dropdown to select the tool type for each.

> You can click **+ Recommend Tools** to get AI-suggested tools based on your agent description, or add each manually.

***

#### Tool 1 — Knowledge Graph

From **Add tool ▼** select **Knowledge graph**.

![Add a Knowledge graph dialog](<../.gitbook/assets/L1-agent-tool-kg1 (1).png>)

Configure the following fields:

| Field | Value |
| --- | --- |
| **Select knowledge graph** | `User Graph` |
| **Name** | `User related attributes Knowledge Graph` |
| **Execution mode** | `Autonomous` |
| **Display Output** | `No` |

**Tool description** _(Description for LLM)_

```
Use this Knowledge Graph to retrieve relevant information about the user who you are currently engaged with.
```

**Query instruction**

```
Query to the knowledge graph. It should be a direct translation of request into a search query.
```

> **Why this tool:** Fires silently at the start of every conversation. The User Graph gives the agent the caller's related user attributes — so it can personalise the response and pre-populate Incident fields without asking the user a single identity question. **Autonomous** mode means the user never sees the tool call happen.

Click **Add**.

***

#### Tool 2 — File Upload (Troubleshooting Resolution Guide)

From **Add tool ▼** select **File upload**.

![Add file upload dialog — fields](<../.gitbook/assets/L1-agent-tool4-file-upload (1).png>)

![Add file upload dialog — attachment](<../.gitbook/assets/L1-agent-tool4-file-upload2 (1).png>)

Configure the following fields:

| Field | Value |
| --- | --- |
| **Name** | `Troubleshooting Resolution Guide` |
| **Execution mode** | `Autonomous` |
| **Display output** | `No` |
| **Attachments** | `troubleshooting-resolution-guide.pdf` |

**Tool description** _(Description for LLM)_

```
This guide covers common enterprise backup appliance problems categorised into three severity tiers (L1, L2, L3), providing symptoms, likely causes, and diagnostic steps for each to help operations agents quickly triage issues and determine whether to resolve, escalate, or raise an Incident.
```

> **Why this tool:** This is the deflection gate. If the user confirms the steps resolved their issue, the conversation ends — no Incident is created. Only unresolved issues continue to image collection.

Click **Add**.

***

#### Tool 3 — Conversational Topic (Image Upload)

From **Add tool ▼** select **Conversational topic**.

![Add tool dropdown — Conversational topic highlighted](<../.gitbook/assets/L1-Agent-Tool1 (1).png>)

![Add a conversational topic dialog](<../.gitbook/assets/L1-agent-tool-conv-topic (1).png>)

Configure the following fields:

| Field | Value |
| --- | --- |
| **Select topic** | `Upload image x_nava_agentic_lab` |
| **Name** | `Upload image x_nava_agentic_lab` |
| **Execution mode** | `Autonomous` |
| **Display output** | `Yes` |

**Tool description** _(Description for LLM)_

```
This tool allows user to upload image to Now Assist Virtual Agent
```

> **Why this tool:** Renders the native in-chat file picker. The agent invokes it only after the user confirms the Troubleshooting Guide did not resolve their issue. The images uploaded here are what NADI processes to extract the error code.
>
> Ensure you have duplicated the OOTB Topic 'Upload Image' within Virtual Agent Designer and scoped it to `x_nava_agentic_lab` before testing.

Click **Add**.

***

#### Tool 4 — Subflow (Incident Creation)

From **Add tool ▼** select **Subflow**.

![Add tool dropdown — Subflow highlighted](<../.gitbook/assets/L1-agent-tool2 (1).png>)

![Add a subflow — subflow selection and inputs](<../.gitbook/assets/L1-agent-tool2-subflow (1).png>)

![Add a subflow — tool name and description](<../.gitbook/assets/L1-agent-tool2-subflow2 (1).png>)

Configure the following fields:

| Field | Value |
| --- | --- |
| **Select subflow** | `Create and submit Incident record with image upload(s) subflow` |
| **Name** | `Create Incident Case` |
| **Execution mode** | `Autonomous` |
| **Display output** | `No` |

**Mandatory subflow inputs:**

| Input field | Internal name | Data type | Source |
| --- | --- | --- | --- |
| Category type | `category_type` | string | Derived from conversation context |
| Chat user name | `chat_user_name` | string | Retrieved by Tool 1 (Knowledge Graph) |
| Short description input | `short_description_input` | string | User-reported issue description |
| CI name input | `ci_name_input` | string | Retrieved by Tool 1 (Knowledge Graph) |
| Work notes details | `work_notes_details` | string | Diagnostic notes from troubleshooting session |

**Tool description** _(Description for LLM)_

```
The Create Incident Case Tool is a subflow that takes in four inputs in order to raise the Incident case. All of the four inputs are mandatory parameters.
```

> **Why this tool:** Creates and submits the Incident record — but only after Tool 3 (image upload) has completed. The Incident is created with images already attached so NADI triggers immediately and can extract `u_extracted_error_code`.

Click **Add**.

***

### Step 3 — Define User Access

The wizard advances to **Define security controls → Define user access**.

![Define user access — role assignment](<../.gitbook/assets/L1-agent-user-access (1).png>)

| Field | Value |
| --- | --- |
| **User access** | `Users with specific roles` |
| **Role(s)** | `snc_internal` |

> Restricts agent access to authenticated IT service desk users. General platform users without this role cannot invoke the agent via NAVA.

Click **Save and continue**.

***

### Step 4 — Define Data Access

The wizard advances to **Define data access**.

![Define data access — user identity](<../.gitbook/assets/L1-agent-data-access (1).png>)

| Field | Value |
| --- | --- |
| **User identity type** | `Dynamic user` |
| **Approved role(s)** | `snc_internal` |

> **Dynamic user** means the agent runs as the logged-in user's identity — it inherits their ACLs for every read and write operation. The `snc_internal` approved role sets the ceiling — the agent cannot exceed those permissions regardless of who is logged in.

Click **Save and continue**.

***

### Step 5 — Add Triggers

Navigate to **Add triggers** — leave empty, no triggers required for this agent.

Click **Save and continue**.

***

### Step 6 — Select Channels and Status

The wizard advances to **Select channels and status**.

![Select channels and status](<../.gitbook/assets/L1-agent-channel-status (1).png>)

| Field | Value |
| --- | --- |
| **Engage via the Now Assist panel** | `OFF` |
| **Engage via Virtual Agent assistants** | `ON` |
| **Chat assistants** | `Now Assist in Virtual Agent AlLab` |

> Select `Now Assist in Virtual Agent AlLab` — the assistant created in Capability 01. The Now Assist panel toggle stays **OFF** deliberately — this agent is triggered through NAVA, not the panel.

Click **Save and continue** to complete the agent configuration.

***

### Step 7 — Test the Agent

**Impersonate as user Alex Rai → Service Portal → Chat Widget**

1. Enter: _"I can't reach the backup server"_
2. **Tool 1 check** — agent responds with caller context without asking for it
3. **Tool 2 check** — agent presents L1/L2/L3 diagnostic steps
4. Reply _"resolved"_ → conversation ends, no Incident created (**deflection confirmed**)
5. Re-run → reply _"not resolved"_ → **Tool 3 check** — image upload prompt appears
6. Upload a test image → **Tool 4 check** — Incident created with `state = New`, `channel = chat`, image attached
7. Open the Incident — confirm NADI triggered and `u_extracted_error_code` is populated

***

## Key Configuration Summary

| Field | Value |
| --- | --- |
| Agent name | `First Responder Operations Analyst Agent` |
| Type | Chat |
| Tool 1 | Knowledge Graph — `User related attributes Knowledge Graph` — User Graph |
| Tool 2 | File Upload — `Troubleshooting Resolution Guide` |
| Tool 3 | Conversational Topic — `Upload image x_nava_agentic_lab` |
| Tool 4 | Subflow — `Create Incident Case` |
| User access | Users with specific roles → `snc_internal` |
| Data access | Dynamic user → approved role: `snc_internal` |
| Channel | Virtual Agent — `Now Assist in Virtual Agent AlLab` |
| Now Assist panel | OFF |

***

## Technical Notes

### Tool Execution Order

1. **Tool 1 (Knowledge Graph)** — conversation start; silently builds user context
2. **Tool 2 (File Upload)** — presents troubleshooting guide; deflection gate
3. **Tool 3 (Conversational Topic)** — fires only if issue unresolved; triggers image upload
4. **Tool 4 (Subflow)** — fires only after Tool 3 completes; creates Incident with images already attached

All subflow inputs are mandatory — the agent accumulates them across the conversation before invoking Tool 4.

***

## Reference

* [ServiceNow Zurich — AI Agent Studio](https://www.servicenow.com/docs/bundle/zurich-intelligent-experiences/page/administer/now-assist-ai-agents/concept/ai-agent-studio.html)
* [ServiceNow Zurich — Create an AI agent](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/configure-next-best-action-agent.html)
* [ServiceNow Zurich — Add tools and information](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/add-tool-aia.html)
* [AI Agents FAQ and Troubleshooting](https://www.servicenow.com/community/now-assist-articles/ai-agents-faq-and-troubleshooting/ta-p/3200454)

***

## Next Steps

→ [03 — Now Assist Document Intelligence](03-now-assist-document-intelligence.md) — configure NADI, which auto-triggers on Incident attachments created by Tool 4 and populates `u_extracted_error_code`
