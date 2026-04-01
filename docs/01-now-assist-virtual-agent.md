# 01 — Now Assist for Virtual Agent (NAVA)

> **Release:** Zurich | **Flow:** Requestor Flow — Phase 1 (Step 1 & 2) **Source:** [ServiceNow Zurich — Using Now Assist in Virtual Agent](https://www.servicenow.com/docs/bundle/zurich-conversational-interfaces/page/administer/now-assist-in-va/concept/using-now-assist-in-va.html) | [Configuring Now Assist in Virtual Agent](https://www.servicenow.com/docs/bundle/zurich-conversational-interfaces/page/administer/now-assist-in-va/task/configure-now-assist-va.html)

***

## What It Is

**Now Assist for Virtual Agent (NAVA)** is the AI-powered conversational interface that serves as the **entry point for the entire Requestor Flow**. It is the surface through which users interact with the platform — via Service Portal, Employee Center, or Mobile — using natural language rather than structured forms.

In Zurich, NAVA is no longer a scripted conversation tree. It is powered by large language models (LLMs) and the **AI Agent Fabric**, which means conversations are handled dynamically by AI agents that reason, use tools, and take action. When a user types _"I can't access the server"_, NAVA receives the message, stamps `contact_type = chat` on the session, and routes the conversation to the appropriate AI agent.

### NAVA vs. Virtual Agent — Key Distinction

| Component                               | Role                                                                                                                |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **Virtual Agent**                       | The underlying conversational platform — topics, flows, channel routing, NLU model                                  |
| **Now Assist for Virtual Agent (NAVA)** | The generative AI layer — LLM-powered responses, AI agent handoffs, Knowledge Graph slot-filling, agentic reasoning |

NAVA is built on top of Virtual Agent. Both must be active for this lab to function.

***

## Role in the Requestor Flow

```
[Step 1 — Requestor Flow]

User types message in Service Portal / Employee Center / Mobile
        │
        ▼
Now Assist for Virtual Agent (NAVA)
        │  stamps contact_type = chat on the session
        ▼
NLU + AI Agent Planner evaluate the message
        │
        ▼
L1 / Requestor AI Agent is triggered
        │
        ▼
... Phase 1 continues (KB deflection → troubleshooting guide → incident creation)
```

> **Why `contact_type = chat` matters:** This field is the trigger condition for the downstream Agentic Workflow. If `contact_type` is not stamped as `chat`, the First Responder Operations Analyst Agent will not fire — even if all other conditions are met.

***

## What NAVA enables in this lab

| Capability            | How NAVA Enables It                                                                                                          |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Conversational intake | User describes the issue in natural language — no form filling                                                               |
| Session stamping      | `contact_type = chat` is automatically applied to the session and propagates to the Incident record                          |
| AI Agent routing      | NAVA's AI Agent Planner identifies the right agent (First Responder Operations Analyst Agent) and hands off the conversation |
| Agentic support       | AI Agents are prioritised in VA responses by default via the **Agentic Support** setting                                     |

***

## Architecture — How NAVA is configured

In Zurich, NAVA is configured through the **Assistant Designer**, a unified configuration surface. The wizard steps through the following sections in order:

```
Assistant Designer — Setup Wizard Order
  1. Settings: Basic details
  2. Settings: Agentic support
  3. Settings: Display experience
  4. Settings: Branding
  5. Settings: Chat experience
  6. Settings: Additional Chat features
  7. Information Sources: Search sources
  8. Information Sources: Knowledge Graphs
  9. Information Sources: Assets
  10. Review
```

***

## Lab Exercise — Steps to Configure NAVA

### Step 1: Open the Assistant Designer

Navigate to **Conversational Interfaces** → **Assistant Designer**

![Assistant Designer — Create Chat-Based Assistant](../.gitbook/assets/nava-create-chat-based-assistant.png)

> The Assistant Designer shows the **Assistants** tab. The default assistant is **Now Assist for Virtual Agent**. Click **Edit** to open the setup wizard.

***

### Step 2: Basic Details

The first section in the wizard is **Basic details**.

![NAVA Basic Details](../.gitbook/assets/nava-basic-detail.png)

Review and confirm:

| Field       | Value for This Lab                                   |
| ----------- | ---------------------------------------------------- |
| Name        | `Now Assist in Virtual Agent - AI Lab`               |
| Description | Description for Now Assist in Virtual Agent - AI Lab |

***

### Step 3: Agentic Support

The second section is **Agentic Support**.

![NAVA Agentic Support](../.gitbook/assets/nava-agentic-support.png)

| Setting                          | Value                                                            |
| -------------------------------- | ---------------------------------------------------------------- |
| Enable Agentic Support           | On                                                               |
| AI Agents prioritised in planner | Enabled (system property: `sn_aia.use_agents_in_planner = true`) |

> This ensures the AI Agent Planner evaluates registered agents **before** falling back to scripted topics. The First Responder Operations Analyst Agent will be evaluated first for every IT infrastructure message. More information on how this works is available here: https://www.servicenow.com/community/now-assist-articles/virtual-agent-with-agentic-reasoning-in-the-october-release-and/ta-p/3406077

***

### Step 4: Search Sources

The third section is **Search Sources**.

![NAVA Search Sources](../.gitbook/assets/nava-search-source.png)

## No configuration needed on this page - Click "Save and Continue"

### Step 5: Knowledge Graphs

The fourth section is **Knowledge Graphs**.

![NAVA Knowledge Graph](../.gitbook/assets/nava-kg.png)

No configuration needed on this page - Click "Save and Continue"

> To note: The First Responder Operations Analyst Agent uses the Knowledge Graph as **Tool** to query user context and identify who they are currently speaking with. The important distinction to note here is that we are not configuring the Knowledge Graphs available within Virtual Agent, but instead adding Knowledge Graph as a tool within the AI agent build.

***

### Step 6: Assets

The fifth section is **Assets**.

![NAVA Assets](../.gitbook/assets/nava-assets.png)

No configuration needed on this page - Click "Save and Continue"

***

### Step 7: Display Experience

The sixth section is **Display Experience** — where NAVA is surfaced.

![NAVA Display Experience](../.gitbook/assets/nava-display.png)

1. Add the **Service Portal** to the Chat experience for the new Assistant that you are creating. You have the option of deciding and configuring if you would like for it to be a Standard or Enhanced Chat experience. This will not affect the lab as much, but more of a conversational experience for users when interacting with the Virtual Agent.

![NAVA Display — Portal](../.gitbook/assets/nava-display-portal1.png)

> **Display experiences** determine which portal or workspace the assistant is embedded in. For this lab: **Service Portal** (`/sp`) chat widget.

***

### Step 8: Chat Experience & Chat Features

The eighth through tenth sections are **Branding**, **Chat experience**, and **Chat features**.

For this lab, **Branding** can be left at defaults, or you can choose a specific Branding theme that you like. The critical setting is in **Chat features**:

![NAVA Chat Features](../.gitbook/assets/nava-chat-features.png)

| Setting            | Value       |
| ------------------ | ----------- |
| Web search mode    | NOT Checked |
| Response Streaming | ✅ Checked   |
| Document uploads   | ✅ Checked   |
| Closed chats       | ✅ Checked   |

> **File upload in chat** must be enabled here. Without it, the file upload prompt in the Conversation Topic (Capability 02) will not function, and the user cannot submit error screenshots to trigger Document Intelligence Task.

***

### Step 9: Review and Save

The final section is **Review**. Confirm all settings look correct across each section, then click **Save**. Once all saved, check and make sure that the Assistant is in **Activated** mode.

***

### Step 10: Verify the Chat Widget in Service Portal

1. Navigate to your instance's **Service Portal** (`/sp`)
2. Click the chat icon (bottom-right corner)
3. Verify the Virtual Agent widget loads and NAVA responds
4. Check and confirm that the Chat Widget populated in Service Portal is the Assistant that you had just configured in Virtual Agent Designer.

***

Once the Virtual Agent Assistant has been setup, we need to move on to duplicate the 'Upload Image' topic that we will require as part of the AI Agent build.

### Step 11: Navigate to the Upload Image Topic

The **Upload image** topic is an out-of-the-box (OOB) topic provided by the Platform AI Agents and Skills application. It handles image upload within Virtual Agent conversations. For this lab, you will duplicate the topic into your scoped application so it can be used by the First Responder Operations Analyst Agent.

1. In Assistant Designer, click the Asset library tab
2. Select the Topics sub-tab
3. In the search bar, type Upload image
4. Confirm the OOB topic appears — it should show as Published and Active

![NAVA Chat Features](../.gitbook/assets/va-topic-tool-uploadimage-1.png)

> The OOB topic is owned by the **Platform AI Agents and Skills** scope. You cannot edit it directly — you must duplicate it into your lab scope (`x_nava_agentic_lab`). Make sure you are on the right scope before performing the duplication step!

***

### Step 12: Open the Topic and Duplicate It

1. Click on the **Upload image** topic name to open it in the **Flow** view
2. On the canvas, review the existing flow structure:

```
Upload image  →  Process image  →  Decision
                                      │
                              ┌───────┴───────┐
                        File Supported    File Not Supported
                              │                   │
                            End           File Error Message
```

3. Click the **kebab menu** (three-dot icon, top-right) and select **Duplicate**

![Upload Image Topic — Duplicate Option](../.gitbook/assets/va-topic-tool-uploadimage-2.png)

> Notice the left-hand **Components** palette — it shows the available node types (Input Collector, Text, Static Choice, File Picker, etc.) and the **Bot response** nodes. You do not need to modify the flow for this lab; the duplicate will inherit the full flow as-is.

***

### Step 13: Name the Duplicate Topic

In the **Duplicate Topic** dialog:

| Field | Value                             |
| ----- | --------------------------------- |
| Name  | `Upload image x_nava_agentic_lab` |

Click **Save**.

![Duplicate Topic Dialog](../.gitbook/assets/va-topic-tool-uploadimage-3.png)

> The naming convention appends the scoped application name to distinguish the duplicate from the OOB topic. This is important — the AI Agent will reference this topic by name when invoking the tool.

***

### Step 14: Verify the Duplicate in the Topics List

After saving, check on the 'Variables' tab and ensure that the variables have been copied over

You should now see **two** topics:

| Variable Name                    | Variable Type |
| -------------------------------- | ------------- |
| upload\_image                    | Input         |
| user                             | Input         |
| doc\_intel\_max\_file\_size      | Script        |
| doc\_intel\_supported\_file\_... | Script        |
| is\_file\_size\_supported        | Script        |
| is\_file\_type\_supported        | Script        |

![Topics List — Duplicate Confirmed](../.gitbook/assets/va-topic-tool-uploadimage-4.png)

***

### Step 15: Configure Topic Properties — System Roles

1. Switch to the **Properties** tab
2. Scroll down to **Advanced properties (optional)** → expand **Who can access this topic**
3. Next to **System Roles**, click the **edit** (pencil) icon

![Topic Properties — Advanced Properties](../.gitbook/assets/va-topic-tool-uploadimage-5.png)

4. In the **System Roles** dialog, move the following two roles from **Selected** to **Available** (essentially removing it from the System Roles):

| Role                                        | Purpose                                                                      |
| ------------------------------------------- | ---------------------------------------------------------------------------- |
| `wm_agent`                                  | Allows Workspace agents to invoke this topic                                 |
| `sn_uxc_gen_ai.platform_ai_image_processor` | Grants the AI image processing capability required by the Process image node |

![System Roles — Selection Dialog](../.gitbook/assets/va-topic-tool-uploadimage-6.png)

5. Click **Save** on the System Roles dialog
6. Confirm that the roles have been removed

![Topic Properties — Roles Configured](../.gitbook/assets/va-topic-tool-uploadimage-7.png)

***

### Step 16: Save, Publish, and Verify Active State

1. Click **Save** in the top-right corner
2. If the topic status shows **Inactive**, click **Publish** to make it active and available in the chat window
3. Switch to the **Flow** tab and confirm the status indicator shows **Active** (green dot)

![Duplicated Topic — Active and Published](../.gitbook/assets/va-topic-tool-uploadimage-9.png)

> The topic is now live in the `x_nava_agentic_lab` scope. It will be referenced as a **Tool** by the First Responder Operations Analyst Agent when it needs to prompt the user to upload an error screenshot.

***

## Key Configuration Fields

| Field                 | Value for This Lab                                 |
| --------------------- | -------------------------------------------------- |
| Assistant Name        | Now Assist in Virtual Agent - AI Lab               |
| Display experience    | Service Portal (`/sp`)                             |
| Agentic support       | Enabled                                            |
| Chat Features Enabled | Response Streaming, Document Uploads, Closed Chats |

***

## Technical Notes

### `contact_type = chat` — How It Gets Stamped

When a user interacts with NAVA via the Service Portal chat widget, ServiceNow automatically stamps `contact_type = chat` on the virtual agent session.

This is a **platform behaviour** — no manual configuration needed. However it is critical to verify post-incident creation, as it is one of three conditions that gate the Resolution Pathfinder Agentic Workflow:

```
Agentic Workflow trigger conditions:
  ✓ state = In Progress (2)
  ✓ channel = chat
  ✓ u_extracted_error_code ≠ empty
```

If `channel` is not `chat` (e.g., incident created via email or form), the Agentic Workflow will not trigger.

### Agentic Reasoning

From Zurich Patch 2, the following default behaviours change:

* AI Agents are **prioritised** over scripted topics in every response
* Multi-intent is supported and appears in responses
* Disambiguation follow-up questions are only asked for very unclear/one-word queries

> Source: [Virtual Agent with agentic reasoning — ServiceNow Community](https://www.servicenow.com/community/now-assist-articles/fully-agentic-virtual-agent-in-the-october-release-and-what-it/ta-p/3406077)

***

## Reference

* [ServiceNow Zurich — Using Now Assist in Virtual Agent](https://www.servicenow.com/docs/bundle/zurich-conversational-interfaces/page/administer/now-assist-in-va/concept/using-now-assist-in-va.html)
* [ServiceNow Zurich — Configuring Now Assist in Virtual Agent](https://www.servicenow.com/docs/bundle/zurich-conversational-interfaces/page/administer/now-assist-in-va/task/configure-now-assist-va.html)
* [Now Assist in Virtual Agent — Resources Guide](https://www.servicenow.com/community/now-assist-articles/now-assist-in-virtual-agent-resources-guide/ta-p/3052139)
* [Virtual Agent with agentic reasoning in Zurich](https://www.servicenow.com/community/now-assist-articles/fully-agentic-virtual-agent-in-the-october-release-and-what-it/ta-p/3406077)

***

## Next Step

Continue to [02 — First Responder Operations Analyst Agent](02-L1-agent-first-responder-analyst-agent.md) to configure the Requestor AI Agent: First Responder Operations Analyst Agent.
