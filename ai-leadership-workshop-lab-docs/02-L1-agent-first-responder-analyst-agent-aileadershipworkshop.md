# 02 — L1 First Responder Operations Analyst Agent

> **Release:** Zurich | **Flow:** Requestor Flow — Phase 1 (Step 3)

***

## What It Is

The **First Responder Operations Analyst Agent** is an AI Agent built in AI Agent Studio. It is the first automated intelligence that engages after NAVA routes a user message — handling the conversation from initial user identification through to Incident creation.

This section covers building the **AI Agent** only.

***

## Role in the Requestor Flow

![L1 Agent Flow Overview](../screenshots/flow-L1-agent.png)

***

## What the Agent Enables

| Capability                    | Tool                        | How                                                                 |
| ----------------------------- | --------------------------- | ------------------------------------------------------------------- |
| Silent user contextualisation | Tool 1 — Knowledge Graph    | Queries User Graph to get attributes related to the user            |
| Guided troubleshooting        | Tool 2 — File Upload        | L1/L2/L3 severity-tiered guide from attached PDF                    |
| Deflection                    | —                           | Issue resolved in chat → conversation ends, no Incident created     |
| In-chat image upload          | Tool 3 — Conversation Topic | OOTB topic renders native in-chat image upload picker               |
| Enriched Incident creation    | Tool 4 — Subflow            | Creates Incident after images captured; images attached             |

***

## Prerequisites

| Requirement                         | Detail         |
| ----------------------------------- | -------------- |
| New Virtual Agent assistant created | Complete steps |
| Duplicate 'Upload Image' topic      | Complete steps |

***

## Lab Exercise — Steps to Build

### Step 1 — Define the Specialty

Navigate to **All → AI Agent Studio → Create and manage → AI Agents → Add → Chat**.

![Create a new AI Agent (Chat type)](<../screenshots/flow-L1-agent-create-new.png>)

The wizard opens on **Define the specialty**.

![Define the specialty — agent name and description](<../screenshots/L1-Agent.png>)

The page instructs: _"Using clear, precise language, write the name, description, role, and list of steps this AI agent completes. Writing a clear, distinct name and description is critical because the LLM relies on the wording to correctly identify and use this agent."_

Configure the following fields:

| Field                                            | Value                                      |
| ------------------------------------------------ | ------------------------------------------ |
| **AI agent name**                                | `First Responder Operations Analyst Agent` |
| **AI agent description** _(Description for LLM)_ | Definition in **AI Agent definition**      |
| **AI agent role**                                | Definition in **AI Agent definition**      |
| **List of Steps**                                | Definition in **AI Agent definition**      |

**Retrieve AI Agent Description, role and List of steps:** [AI Agent definition](https://github.com/raadlakha/AILab2.0/blob/main/AI%20Agents%20definition%20and%20prompts/First-responder-operations-analyst-agent-definition)

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

| Field                      | Value                                     |
| -------------------------- | ----------------------------------------- |
| **Select knowledge graph** | `User Graph`                              |
| **Name**                   | `User related attributes Knowledge Graph` |
| **Execution mode**         | `Autonomous`                              |
| **Display Output**         | `No`                                      |

**Tool description** _(Description for LLM)_

```
Use this Knowledge Graph to retrieve relevant information about the user who you are currently engaged with.
```

> **Why this tool:** Fires silently at the start of every conversation. The User Graph gives the AI Agent the caller's related user attributes — so it can personalise the response and pre-populate Incident fields without asking the user a single identity question. **Autonomous** mode means the user never sees the tool call happen.

Click **Add**.

***

#### Tool 2 — File Upload (Troubleshooting Resolution Guide)

From **Add tool ▼** select **File upload**.

![Add file upload dialog — fields](<../.gitbook/assets/L1-agent-tool4-file-upload (1).png>)

![Add file upload dialog — attachment](<../.gitbook/assets/L1-agent-tool4-file-upload2 (1).png>)

Configure the following fields:

| Field              | Value                                  |
| ------------------ | -------------------------------------- |
| **Name**           | `Troubleshooting Resolution Guide`     |
| **Execution mode** | `Autonomous`                           |
| **Display output** | `No`                                   |
| **Attachments**    | `troubleshooting-resolution-guide.pdf` |

**Tool description** _(Description for LLM)_

```
This guide covers common enterprise backup appliance problems categorised into three severity tiers (L1, L2, L3), providing symptoms, likely causes, and diagnostic steps for each to help operations agents quickly triage issues and determine whether to resolve, escalate, or raise an Incident.
```

> **Why this tool:** This is the deflection gate. If the user confirms the steps resolved their issue, the conversation ends — no Incident is created. Only unresolved issues continue on in the flow.

Click **Add**.

***

#### Tool 3 — Conversational Topic (Image Upload)

From **Add tool ▼** select **Conversational topic**.

![Add tool dropdown — Conversational topic highlighted](<../.gitbook/assets/L1-Agent-Tool1 (1).png>)

![Add a conversational topic dialog](<../.gitbook/assets/L1-agent-tool-conv-topic (1).png>)

Configure the following fields:

| Field              | Value                                                                                                                                                                             |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Select topic**   | `Upload image x_nava_agentic_lab`                                                                                                                                                 |
| **Name**           | `Upload image x_nava_agentic_lab`                                                                                                                                                 |
| **Execution mode** | `Autonomous`                                                                                                                                                                      |
| **Display output** | `No` |

**Tool description** _(Description for LLM)_

```
This tool allows user to upload image to Now Assist Virtual Agent
```

> **Why this tool:** Renders the native in-chat file picker. The agent invokes it only after the user confirms the Troubleshooting Guide did not resolve their issue.
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

| Field              | Value                                                            |
| ------------------ | ---------------------------------------------------------------- |
| **Select subflow** | `Create and submit Incident record with image upload(s) subflow` |
| **Name**           | `Create Incident Case`                                           |
| **Execution mode** | `Autonomous`                                                     |
| **Display output** | `No`                                                             |

**Mandatory subflow inputs:**

| Input field             | Internal name             | Data type | Source                                         |
| ----------------------- | ------------------------- | --------- | -----------------------------------------------|
| Category type           | `category_type`           | string    | Derived from conversation context              |
| Chat user name          | `chat_user_name`          | string    | Retrieved by Tool 1 (Knowledge Graph)          |
| Short description input | `short_description_input` | string    | User-reported issue description (LLM summarise)|
| Work notes details      | `work_notes_details`      | string    | Diagnostic notes from troubleshooting session  |

**Tool description** _(Description for LLM)_

```
The Create Incident Case Tool is a subflow that takes in four inputs in order to raise the Incident case. All of the four inputs are mandatory parameters.
```

> **Why this tool:** Creates and submits the Incident record — but only after Image Upload tool has completed invocation. The Incident is created with images already attached (optional).

Click **Add**.

***

### Step 3 — Define User Access

The wizard advances to **Define security controls → Define user access**.

![Define user access — Any authenticated user](<../screenshots/L1-agent-user-access-2.png>)

| Field           | Value                       |
| --------------- | --------------------------- |
| **User access** | `Any authenticated user`    |

Click **Save and continue**.

***

### Step 4 — Define Data Access

The wizard advances to **Define data access**.

![Define data access — user identity](<../screenshots/L1-agent-data-access-2.png>)

| Field                  | Value          |
| ---------------------- | -------------- |
| **User identity type** | `Dynamic user` |
| **Approved role(s)**   | `admin`, `x_snc_apacaienable.incident_extend_user`  |

Click **Save and continue**.

***

### Step 5 — Add Triggers

Navigate to **Add triggers** — leave empty, no triggers required for this agent.

Click **Save and continue**.

***

### Step 6 — Select Channels and Status

The wizard advances to **Select channels and status**.

![Select channels and status](<../screenshots/L1-agent-channel-status.png>)

| Field                                   | Value                               |
| --------------------------------------- | ----------------------------------- |
| **Engage via the Now Assist panel**     | `OFF`                               |
| **Engage via Virtual Agent assistants** | `ON`                                |
| **Chat assistants**                     | `Now Assist in Virtual Agent - AI Lab` |

> Select `Now Assist in Virtual Agent - AI Lab` — the assistant created in Capability 01. The Now Assist panel toggle stays **OFF** deliberately — this agent is triggered through NAVA, not the panel.

Click **Save and continue** to complete the agent configuration.

***

### Step 7 — Test the Agent

#### 7.1 — Impersonate Alex Rai

1. In the ServiceNow platform UI, click on your **user avatar / profile icon** in the top-right banner
2. From the dropdown menu, select **Impersonate user**

![User Menu — Impersonate user option](../.gitbook/assets/L1-agent-testing-1.png)

3. In the **Impersonate user** dialog, search for or select **alex rai** (`alex.rai`) from the user list
4. Click **Impersonate user** to confirm

![Impersonate user dialog — Alex Rai selected](../.gitbook/assets/L1-agent-testing-2.png)

> **Why impersonate Alex Rai?** Running this evaluation as a non System Administrator user allows us to see the Knowledge Graph in action, as well as AI Agent discovery (only authenticated users can see this AI Agent).

***

#### 7.2 — Open the Service Portal and Start a Chat

5. Navigate to your instance's **Service Portal** (`/sp`)
6. Click the **chat icon** in the bottom-right corner to open the **Now Support** chat widget
7. Verify that the widget loads and greets you with a message such as _"Hi alex, how can I help you?"_

![Service Portal — Now Support chat widget open](../.gitbook/assets/L1-agent-testing-3.png)

> **Checkpoint:** Confirm the top-right corner of the Service Portal shows **alex rai** as the logged-in user. If it still shows System Administrator or another user, the impersonation did not take effect — go back and re-impersonate.

***

#### 7.3 — Initiate the Conversation and Verify Agent Routing

8. In the chat input field, type: **`Hi, I can't reach my backup server`** and press Enter
9. Observe the agent response — you should see:
   * A brief acknowledgement of the issue (e.g., _"Let me initiate diagnostics to help you with your backup server connectivity issue."_)
   * A system message confirming **Started AI Agent "First Responder Operations Analyst Agent"**
   * The agent addresses the user by name (_"Hello Alex Rai"_) — confirming **Tool 1 (Knowledge Graph)** fired silently and retrieved the user's identity
   * The agent asks a clarification question to categorise the issue, presenting **Hardware** and **Software** as selectable options

![Chat — Agent started, user identified, category options presented](../screenshots/testing-agent1-hw-sw.png)

> **What to verify:**

| Check                    | Expected Behaviour                                                                   |
| ------------------------ | ------------------------------------------------------------------------------------ |
| Agent routing            | System message shows "Started AI Agent **First Responder Operations Analyst Agent**" |
| Tool 1 — Knowledge Graph | Agent greets user by name without being told — user context retrieved silently       |
| Category prompt          | Hardware / Software buttons rendered as selectable options                           |

***

#### 7.3a — Agent Gathers Structured Context

After the category selection, the agent continues collecting the structured context it needs before invoking the Troubleshooting Guide. The screenshots below show this information-gathering sequence:

10. Select **Hardware** — the agent then asks which specific product or system is affected. Reply **`Veritas NetBackup`**.
11. The agent asks for the hostname of the affected device. Reply with the any hostname (e.g., **`vnb-01-sn1`**, there is no fixed hostname).

![Chat — Hardware selected, product identified as Veritas NetBackup, hostname provided](../screenshots/testing-agent1-pn-hn.png)

12. The agent then asks when the issue first occurred — select the date and time using the calendar (e.g., **Saturday, April 14, 2026 2:44 PM**).
13. The agent asks whether this is affecting a single device or multiple devices. Reply either **`Single device`** or **`Multiple device`**.

![Chat — Date and time selected, device scope confirmed as single device](../screenshots/testing-agent1-datetime-devicetype.png)

> **Note:** The exact questions the agent asks and their order may vary depending on how you have authored the agent's description, role, and list of steps. The agent uses its LLM reasoning to determine which context fields it still needs before proceeding — these screenshots reflect one representative flow.

***

#### 7.4 — Describe the Issue and Verify Troubleshooting Guide

14. Provide additional detail about the issue, **for example: `I'm seeing that the server seems to be overheating in temperature, and the LED lights are turning from green to red colour`**
15. Observe the agent response — you should see:
    * A system message confirming the agent **Used the tool "Troubleshooting Resolution Guide"** — this confirms **Tool 2 (File Upload)** was invoked
    * The agent attempts to match the symptoms against the L1/L2/L3 troubleshooting guide
    * If no matching resolution steps are found, the agent informs the user and offers to raise an Incident (e.g., _"Unfortunately, I could not find any documented troubleshooting steps for the Veritas NetBackup server overheating and LED lights turning red."_)
    * The agent presents **Yes** and **No** buttons asking whether to proceed with Incident creation

![Chat — Troubleshooting Guide used, no resolution found, Incident offer](../.gitbook/assets/L1-agent-testing-5.png)

> **What to verify:**

| Check                          | Expected Behaviour                                                           |
| ------------------------------ | ---------------------------------------------------------------------------- |
| Tool 2 — Troubleshooting Guide | System message confirms "Used the tool **Troubleshooting Resolution Guide**" |
| Deflection logic               | Agent evaluates the guide and reports whether steps were found               |
| Escalation prompt              | Agent asks the user whether to raise an Incident with Yes / No options       |

> **Deflection test (optional):** To test the deflection path, start a new conversation and describe a symptom that _does_ match the troubleshooting guide (e.g., a known L1 backup failure error). Confirm the issue is resolved → the conversation should end without creating an Incident.

***

#### 7.5 — Trigger Image Upload

16. Click **Yes** to confirm you want to raise an Incident
17. The agent invokes **Tool 3 (Conversational Topic — Upload Image)** and the chat displays:
    * _"Please upload an image"_
    * A **"Click here to upload an image."** link/button rendered by the Virtual Agent topic

![Chat — Image upload prompt rendered](../.gitbook/assets/L1-agent-testing-6.png)

> **What to verify:**

| Check                       | Expected Behaviour                                        |
| --------------------------- | --------------------------------------------------------- |
| Tool 3 — Upload Image topic | Chat renders the native file upload prompt                |
| Upload link                 | "Click here to upload an image." is visible and clickable |

> **If the upload prompt does not appear:** Check that (a) the duplicated Upload Image topic is **Active** and scoped to `x_nava_agentic_lab`, (b) **File uploads** is enabled as one of the tools within the AI Agent, and (c) the topic is registered as a tool on this agent.

***

#### 7.6 — Upload the Image

18. Click **"Click here to upload an image."** to open the file picker
19. Select an error screenshot image (e.g., a Veritas NetBackup error screen capture showing an error code) and upload it
20. Observe the chat — you should see:
    * The uploaded image rendered as a thumbnail in the conversation
    * A confirmation message: _"The attachment is available in this link"_
    * The agent shows a **"Processing..."** indicator as it processes the uploaded image

![Chat — Image uploaded, attachment confirmed, processing](../.gitbook/assets/L1-agent-testing-7.png)

> **What to verify:**

| Check                   | Expected Behaviour                                                       |
| ----------------------- | ------------------------------------------------------------------------ |
| Image upload            | Image thumbnail appears in the chat conversation                         |
| Attachment confirmation | "The attachment is available in **this link**" message displayed         |
| Processing              | "Processing..." indicator visible — agent is preparing the issue summary |

***

#### 7.7 — Review the Issue Summary

21. Once processing completes, the agent presents a **summary of all information collected** during the conversation — including the uploaded screenshot — and asks for confirmation before raising the Incident. You should see a structured summary that includes:
    * **Issue type** — the category selected earlier (e.g., `Software`)
    * **Affected product / system** — identified from the conversation context (e.g., `Veritas NetBackup`)
    * **Hostname / IP address** — retrieved from the conversation (e.g., `veritas-backup-01`)
    * **Date/Time of occurrence** — when the issue was reported (e.g., `04-03-2026 11:11:00`)
    * **Screenshot** — upload status (e.g., `Uploaded`)
    * **Issue description** — the user's reported symptoms (e.g., `Server overheating, LED lights turning from green to red`)
22. The agent asks: _"Is all of the above information correct? Shall I go ahead and raise an incident on your behalf? Please reply 'yes' to confirm or 'no' to make changes."_

![Chat — Issue summary presented with confirmation prompt](../.gitbook/assets/L1-agent-testing-9.png)

> **What to verify:**

| Check                | Expected Behaviour                                                                                          |
| -------------------- | ----------------------------------------------------------------------------------------------------------- |
| Summary completeness | Agent displays all key fields — issue type, product, hostname, time, screenshot status, description         |
| Data accuracy        | Summary reflects the information provided during the conversation and retrieved by Tool 1 (Knowledge Graph) |
| Screenshot confirmed (if uploaded) | Summary shows `Screenshot: Uploaded` or `image file names` — confirming the image(s) was captured |
| Confirmation prompt  | Agent asks user to confirm with yes/no before proceeding to Incident creation                               |

> **If the summary is incorrect:** Reply **'no'** — the agent should allow you to correct specific fields before re-confirming. This is a safeguard to ensure the Incident record is created with accurate data.

23. Reply **'yes'** to confirm the summary and proceed with Incident creation

***

#### 7.8 — Verify Incident Creation

24. The agent invokes **Tool 4 (Subflow — Create Incident Case)** and after processing completes, you should see:
    * A confirmation that the Incident has been successfully raised
    * The **Incident reference number** (e.g., `INCE0012003`)
    * A message indicating the support team will review the issue (e.g., _"Your incident has been successfully raised. Your reference number is INCE0012003. Our support team will review this and be in touch shortly."_)
    * The agent may also flag that the issue has been escalated for priority review
    * A closing message: _"All done! Is there anything else I can help you with?"_

![Chat — Incident created with reference number INCE0012003](../.gitbook/assets/L1-agent-testing-8.png)

> **What to verify:**

| Check                 | Expected Behaviour                                                                           |
| --------------------- | -------------------------------------------------------------------------------------------- |
| Incident created      | Agent confirms Incident raised with a reference number (e.g., `INCE0012003`)                 |
| Number prefix         | Reference number starts with `INCE` — confirming it was created on the Incident Extend table |
| Conversation complete | Agent asks if anything else is needed — flow is complete                                     |

***

#### 7.9 — Verify the Incident Record in the Platform

25. End the impersonation session (or open a new browser tab as System Administrator)
26. Navigate to the **incident extend** table: type `x_snc_apacaienable_incident_extend.list` in the Filter navigator
27. Locate the newly created Incident by the reference number from Step 24 (e.g., `INCE0012003`)
28. Open the Incident record and verify the following fields:

| Field             | Expected Value                                   |
| ----------------- | ------------------------------------------------ |
| Number            | `INCE0012003` (or the number returned in chat)   |
| State             | `In Progress`                                    |
| Channel           | `chat`                                           |
| Category          | `Software` (or the category selected in Step 10) |
| Short description | Contains the user-reported issue description     |
| Caller            | `Alex Rai`                                       |
| Attachments       | Uploaded image(s) present on the record          |

> **If the Incident was not created:** Check the following:
>
> * Subflow **Run As** is set to **System User** (Pre-Requisite 5)
> * All four mandatory subflow inputs are being passed correctly by the agent
> * The `x_nava_agentic_lab` application scope is active
> * Alex Rai has the `x_snc_apacaienable.incident_extend_user` role (Pre-Requisite 6)

***

### Test Summary

| Step      | Tool Verified                  | What to Confirm                                                                                  |
| --------- | ------------------------------ | ------------------------------------------------------------------------------------------------ |
| 7.3       | Tool 1 — Knowledge Graph       | Agent greets user by name without asking; "Started AI Agent" system message                      |
| 7.3a      | Context gathering              | Agent collects category, product, hostname, date/time, and device scope before troubleshooting   |
| 7.4       | Tool 2 — Troubleshooting Guide | "Used the tool Troubleshooting Resolution Guide" system message; deflection or escalation prompt |
| 7.5       | Issue Summary confirmation     | Agent presents structured summary of all collected information; user confirms before proceeding  |
| 7.6       | Tool 3 — Upload Image          | Native file upload prompt rendered in chat                                                       |
| 7.7–7.8   | Tool 4 — Create Incident Case  | Image uploaded, Incident reference number returned (INCE prefix)                                 |
| 7.9       | Post-creation verification     | Incident record fields correct; `channel = chat`; attachments present                            |

***

## Key Configuration Summary

| Field            | Value                                                                    |
| ---------------- | ------------------------------------------------------------------------ |
| Agent name       | `First Responder Operations Analyst Agent`                               |
| Type             | Chat                                                                     |
| Tool 1           | Knowledge Graph — `User related attributes Knowledge Graph` — User Graph |
| Tool 2           | File Upload — `Troubleshooting Resolution Guide`                         |
| Tool 3           | Conversational Topic — `Upload image x_nava_agentic_lab`                 |
| Tool 4           | Subflow — `Create Incident Case`                                         |
| User access      | Users with specific roles → `snc_internal`                               |
| Data access      | Dynamic user → approved role: `snc_internal`                             |
| Channel          | Virtual Agent — `Now Assist in Virtual Agent - AlLab`                    |
| Now Assist panel | OFF                                                                      |

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

→ [03 — Now Assist Document Intelligence](03-now-assist-document-intelligence.md) — configure NADI, which auto-triggers on Incident attachments created by Tool 4 and populates `error_code`
