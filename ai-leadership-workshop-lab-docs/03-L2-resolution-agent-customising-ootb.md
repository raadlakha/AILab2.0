# 03 — Customising an Out-of-the-Box Platform AI Agent as the Incident Investigation and Resolution AI Agent
 
> **Release:** Zurich | **Flow:** Fulfiller Flow — Phase 2 (Step 1)
 
***
 
## What It Is
 
The **Incident Investigation and Resolution AI Agent** is a customised copy of the OOTB **ITSM incident resolution investigation AI agent** — one of the platform AI agents that ships with the IT Service Management AI agent collection.
 
Instead of building an agent from scratch (as in Capability 02), this section demonstrates how to **duplicate an existing platform agent and adapt it** to work with the custom Incident Extend table (`x_snc_apacaienable_incident_extend`). The result is a fully functional resolution agent — complete with pre-built tools for incident lookup, knowledge retrieval, similar incident search, and work notes — ready in minutes with only a single script change.
 
***
 
## Why Customise an OOTB Agent?
 
ServiceNow ships dozens of platform AI agents purpose-built for ITSM, ITOM, HRSD, and CSM workflows. Each agent comes with pre-authored descriptions, roles, steps, and tools — all written to best-practice standards.
 
Rather than recreating this logic, customisation lets you:
 
- **Start from a proven baseline** — the agent's LLM instructions, tool descriptions, and execution modes are already optimised
- **Leverage best practices from each workflow group** — OOTB agents are authored by the teams who build each product (ITSM, ITOM, HRSD, CSM), embedding domain-specific expertise in the agent's role, steps, and tool logic that would be hard to replicate from scratch
- **Inherit OOTB tools** — scripted tools, search retrievals, and subflows are duplicated with the agent
- **Adapt with minimal changes** — in this lab, only one script line needs updating to redirect the agent to the custom table
- **Demonstrate platform depth** — in a demo context, this shows that ServiceNow's agentic capabilities are not just net-new builds but extend across the entire ITSM platform
***
 
## What the Agent Enables
 
| Capability                     | Tool                                              | How                                                                                     |
| ------------------------------ | ------------------------------------------------- | --------------------------------------------------------------------------------------- |
| Incident detail retrieval      | Tool 1 — Get details of Incident                  | Scripted tool queries the Incident Extend table by number and returns key fields         |
| Resolution plan generation     | Tool 2 — Add a resolution plan to work notes      | Adds a structured resolution plan to the incident's work notes (if a resolution plan can be generated) |
| Knowledge article retrieval    | Tool 3 — Get relevant knowledge articles (Search) | RAG-powered search retrieval fetches similar and relevant knowledge articles              |
| Similar incident lookup        | Tool 4 — Get similar incidents (Search)            | Retrieves similar incidents based on the short description to surface historical context |
 
***
 
## Prerequisites
 
| Requirement                                          | Detail                                                             |
| ---------------------------------------------------- | ------------------------------------------------------------------ |
| Capability 02 complete                               | L1 First Responder agent built and tested                          |
| Incident Extend table populated                      | At least one Incident record exists on `x_snc_apacaienable_incident_extend` (e.g., created during Capability 02 testing) |
| Now Assist Panel enabled                             | Required for AI Agent Studio Testing — configured in this exercise |
 
***
 
## Lab Exercise — Steps to Build
 
### Step 1 — Locate the OOTB Agent
 
Navigate to **All → AI Agent Studio → Create and manage → AI Agents**.
 
The AI Agents list displays all agents across the instance. To find the OOTB ITSM agents, filter the list by application:
 
1. In the **Application** column, right-click (or click the cell actions icon) on any row showing `IT Service Management AI agent collection`
2. Select **Show Matching** to filter the list to only ITSM platform agents
![AI Agent Studio — Show Matching on IT Service Management AI agent collection](../screenshots/customise-ootb-ai-agent-1.png)
 
3. Scroll through the filtered list to locate **ITSM incident resolution investigation AI agent**
![Filtered AI Agent list — ITSM incident resolution investigation AI agent visible](../screenshots/customise-ootb-ai-agent-2.png)
 
> **What you're looking at:** These are all the AI agents that ship with the IT Service Management AI agent collection — purpose-built for incident investigation, change management, CI suggestions, and more. Each one comes pre-configured with tools, descriptions, and roles.
 
***
 
### Step 2 — Duplicate the Agent
 
1. On the **ITSM incident resolution investigation AI agent** row, click the **Duplicate** icon (clipboard icon on the far right)
![Duplicate icon on the ITSM incident resolution investigation AI agent row](../screenshots/customise-ootb-ai-agent-3.png)
 
2. A confirmation dialog appears: _"You are duplicating an AI Agent. This will produce an identical AI agent. Fine-tune the copied AI agent with distinct instructions and tools by following the AI agent configuration guided setup."_
![Duplicate confirmation dialog](../screenshots/customise-ootb-ai-agent-4.png)
 
3. Click **Duplicate**
The wizard opens on **Define the specialty** with the duplicated agent — name suffixed with `(Copy)`.
 
***
 
### Step 3 — Review the Specialty
 
The duplicated agent inherits the original agent's name, description, role, and list of steps.
 
![Define the specialty — ITSM incident resolution investigation AI agent (Copy)](../screenshots/customise-ootb-ai-agent-5.png)
 
Review the pre-populated fields:
 
| Field                      | Value                                                   |
| -------------------------- | ------------------------------------------------------- |
| **AI agent name**          | `ITSM incident resolution investigation AI agent (Copy)` |
| **AI agent description**   | Pre-populated from the OOTB agent (Description for LLM) |
 
> **Note:** For this lab, keep the default name and description as-is. In a production scenario, you would rename the agent and refine the description to reflect the customised use case (e.g., `Meridian Corp Incident Resolution Agent`).
 
Click **Save and continue**.
 
***
 
### Step 4 — Review the Duplicated Tools
 
The wizard advances to **Add tools and information**. The duplicated agent inherits all four tools from the original.
 
![Add tools and information — four duplicated tools listed](../screenshots/customise-ootb-ai-agent-6.png)
 
The following tools are pre-configured:
 
| #   | Name                                             | Type              | Execution mode | Display output |
| --- | ------------------------------------------------ | ----------------- | -------------- | -------------- |
| 1   | Get details of Incident                          | Scripted tool     | Autonomous     | false          |
| 2   | Add a resolution plan to incident's work notes   | Scripted tool     | Autonomous     | true           |
| 3   | Get relevant knowledge articles                  | Search retrieval  | Autonomous     | false          |
| 4   | Get similar incidents                            | Search retrieval  | Autonomous     | false          |
 
> **Key takeaway:** All four tools — including the scripted tools with their full code and the RAG-powered search retrievals — are duplicated automatically. No manual tool creation required.
 
***
 
### Step 5 — Modify the Incident Lookup Script
 
The OOTB agent's **Get details of Incident** tool queries the standard `incident` table. Since the lab uses a custom extended table, the script must be updated to query `x_snc_apacaienable_incident_extend` instead.
 
1. Click on **Get details of Incident** to open the tool configuration dialog
![Get details of Incident — original script querying "incident" table](../screenshots/customise-ootb-ai-agent-7.png)
 
2. In the **Script** editor, locate the `GlideRecordSecure` initialisation line:
```javascript
var ctGR = new GlideRecordSecure("incident");
```
 
3. Change the table name to the custom Incident Extend table:
```javascript
var ctGR = new GlideRecordSecure("x_snc_apacaienable_incident_extend");
```
 
![Modified script — now querying "x_snc_apacaienable_incident_extend"](../screenshots/customise-ootb-ai-agent-8.png)
 
> **This is the only code change required.** The rest of the script — the encoded query, field retrieval (`short_description`, `description`, `state`, `priority`, `caller`, `assigned_to`), and output mapping — all remain the same because the Incident Extend table inherits these fields from its parent.
 
4. Click **Save**
Click **Save and continue** to proceed.
 
***
 
### Step 6 — Update User Access
 
The wizard advances to **Define security controls → Define user access**.
 
The duplicated agent inherits the OOTB ACL — allowing users with the `itil` and `sn_service_desk_agent` roles.
 
![Define user access — inherited ACL with itil and sn_service_desk_agent](../screenshots/customise-ootb-ai-agent-9.png)
 
For lab testing purposes, add the `admin` role so System Administrators can access the agent during testing:
 
1. Click the **Edit** (pencil) icon on the ACL row
2. In the **Edit ACL** dialog, add `admin` to the **Role** field alongside the existing `itil` and `sn_service_desk_agent` roles
![Edit ACL — adding admin role](../screenshots/customise-ootb-ai-agent-10.png)
 
3. Click **Update**
Click **Save and continue**.
 
***
 
### Step 7 — Configure Data Access
 
The wizard advances to **Define data access**.
 
![Define data access — Dynamic user with approved roles](../screenshots/customise-ootb-ai-agent-11.png)
 
Configure the following:
 
| Field                  | Value          |
| ---------------------- | -------------- |
| **User identity type** | `Dynamic user` |
| **Approved role(s)**   | `itil`, `sn_service_desk_agent`, `x_snc_apacaienable.incident_extend_user` |
 
> **Why add `x_snc_apacaienable.incident_extend_user`?** The agent now queries the custom Incident Extend table. This role ensures the dynamic user identity has the ACL permissions required to read and write records on that table — without it, the scripted tool will fail with a security rule check error.
 
Click **Save and continue**.
 
***
 
### Step 8 — Skip Triggers
 
Navigate to **Add triggers** — leave empty, no triggers required for this agent.
 
Click **Save and continue**.
 
***
 
### Step 9 — Select Channels and Status
 
For this lab, no channel configuration changes are needed. The agent will be tested directly from the AI Agent Studio **Testing** tab (via the Now Assist panel), not through Virtual Agent.
 
Click **Save and continue** to complete the agent configuration.
 
***
 
## Testing the Customised Agent
 
### Step 10 — Open the Testing Tab
 
Navigate to **AI Agent Studio → Testing** and configure a new test:
 
![Testing tab — test configuration](../screenshots/customise-ootb-ai-agent-12.png)
 
| Field                                     | Value                                                     |
| ----------------------------------------- | --------------------------------------------------------- |
| **Choose a test type**                    | `AI agent or workflow`                                    |
| **Name of the AI agent or agentic workflow** | `ITSM incident resolution investigation AI agent (Copy)` |
| **Version**                               | `1 - V1 (Active)`                                        |
 
> **If you see a "Now Assist Panel Disabled" banner:** The AI Agent Studio Testing tab requires the Now Assist panel to be enabled. Follow Steps 11–13 below to resolve this before proceeding.
 
***
 
### Step 11 — Enable the Now Assist Panel (If Required)
 
If the Testing tab displays a banner reading _"Now Assist Panel Disabled. To start testing, you need to turn on the Now Assist panel by going to Now Assist Admin."_ — follow these steps:
 
1. Click **Edit this setting** from the Testing page
![Now Assist Admin — Now Assist panel with Turn on button](../screenshots/customise-ootb-ai-agent-13.png)
 
2. Click **Turn on**
3. A success dialog confirms: _"Now Assist panel is now on"_
![Success — Now Assist panel is now on](../screenshots/customise-ootb-ai-agent-14.png)
 
4. Close the dialog
***
 
### Step 12 — Activate the Now Assist Panel Assistant (If Required)
 
After enabling the Now Assist panel, refresh the Testing tab again and it will display a second banner: _"Now Assist Panel is not configured. To start testing, you need to set up the default assistant for Now Assist Panel."_
 
![Testing tab — Now Assist Panel is not configured banner](../screenshots/customise-ootb-ai-agent-15.png)
 
1. Click **Set up**
2. Open the **Now Assist Panel - Platform (default)** assistant — it will show as **Inactive**
![Assistant Designer — Now Assist Panel - Platform (default) Inactive](../screenshots/customise-ootb-ai-agent-16.png)
 
3. Click **Activate** in the top-right corner
4. Confirm the activation — a banner confirms _"Now Assist Panel - Platform (default) was activated"_
![Assistant Designer — Now Assist Panel - Platform (default) Activated](../screenshots/customise-ootb-ai-agent-17.png)

5. Return back to the Testing tab in AI Agent Studio to test your AI Agent.
 
***
 
### Step 13 — Run the Test
 
Return to **AI Agent Studio → Testing** and configure the test:
 
![Testing tab — task entered with incident number](../screenshots/customise-ootb-ai-agent-18.png)
 
| Field    | Value                                                     |
| -------- | --------------------------------------------------------- |
| **Task** | `Help me resolve INCE0013001`                             |
 
> Replace `INCE0013001` with a valid Incident Extend record number from your instance — for example, the Incident created during Capability 02 testing.
 
Click **Continue to Test Chat Response** to launch the test conversation.
 
***

### Step 14 — Review the Test Results
 
The Testing tab displays three panels: the **chat transcript** (left), the **agent execution map** (centre), and the **AI agent decision logs** (right).
 
#### 14.1 — Resolution Plan Generated Successfully
 
When the agent finds sufficient context — from similar incidents, knowledge articles, or both — it generates a resolution plan and adds it to the incident's work notes.
 
![Test result — resolution plan successfully added to work notes](../screenshots/customise-ootb-ai-agent-19.png)
 
> **What to verify:**
 
| Check                                     | Expected Behaviour                                                                                      |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Tool — Get details of Incident            | Completed — agent retrieved incident details from the Incident Extend table                              |
| Tool — Get similar incidents              | Completed — agent searched for similar historical incidents                                              |
| Tool — Get relevant knowledge articles    | Completed — agent retrieved matching knowledge articles via RAG                                          |
| Tool — Add a resolution plan to work notes| Completed — output confirms _"The resolution plan has been successfully added to the work notes"_        |
| Decision logs                             | All steps show green checkmarks; execution map shows all five tools invoked                              |
 
#### 14.2 — Resolution Plan Could Not Be Generated
 
When neither similar incidents nor relevant knowledge articles are found (or both), the agent informs the user that a resolution plan cannot be generated due to insufficient resources.
 
![Test result — resolution plan could not be generated due to insufficient resources](../screenshots/customise-ootb-ai-agent-20.png)
 
> **What to verify:**
 
| Check                                     | Expected Behaviour                                                                                      |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Tool — Get details of Incident            | Completed — agent retrieved incident details successfully                                                |
| Tool — Get similar incidents              | Completed — but returned no matching incidents                                                           |
| Tool — Get relevant knowledge articles    | Completed — but returned no matching articles                                                            |
| Agent reasoning                           | Decision log shows the agent's thought process: _"neither similar incidents nor relevant knowledge articles were found... a resolution plan cannot be generated due to insufficient resources"_ |
| Chat response                             | Agent informs the user: _"I couldn't come up with a incident resolution plan as I didn't have the required resources."_ |
 
> **This is expected behaviour.** The agent's ability to generate a resolution plan depends on the knowledge base and incident history available on the instance. In a production environment, a well-populated knowledge base and incident history will significantly improve resolution plan quality.
 
***

 
## Key Configuration Summary
 
| Field                | Value                                                                            |
| -------------------- | -------------------------------------------------------------------------------- |
| Source agent         | `ITSM incident resolution investigation AI agent` (Out Of The Box)               |
| Customised agent     | `ITSM incident resolution investigation AI agent (Copy)`                         |
| Type                 | Chat                                                                             |
| Tool 1               | Scripted tool — `Get details of Incident` — **modified to query Incident Extend**|
| Tool 2               | Scripted tool — `Add a resolution plan to incident's work notes`                 |
| Tool 3               | Search retrieval — `Get relevant knowledge articles`                             |
| Tool 4               | Search retrieval — `Get similar incidents`                                       |
| User access          | Users with specific roles → `itil`, `sn_service_desk_agent`, `admin`             |
| Data access          | Dynamic user → approved roles: `itil`, `sn_service_desk_agent`, `x_snc_apacaienable.incident_extend_user`|
| Channel              | Now Assist Panel (Testing)                                                       |
 
***
 
## Technical Notes
 
### What Was Changed vs. What Was Inherited
 
| Aspect                      | Changed | Inherited |
| --------------------------- | ------- | --------- |
| Agent name and description  |         | ✓         |
| Agent role and steps        |         | ✓         |
| Tool 1 script (table name)  | ✓       |           |
| Tool 1 inputs and outputs   |         | ✓         |
| Tool 2, 3, 4 (all config)   |         | ✓         |
| User access ACL (add admin) | ✓       |           |
| Data access (add role)      | ✓       |           |
 
> **One line of code. Three configuration fields. A fully functional resolution agent.** This is the power of customising OOTB platform agents — you inherit the engineering investment ServiceNow has made in tool descriptions, LLM instructions, and execution logic, and simply redirect it to your use case.
 
### Now Assist Panel Setup
 
The AI Agent Studio Testing tab uses the Now Assist panel as its chat surface. If this is the first time testing from AI Agent Studio on your instance, you may need to:
 
1. **Enable** the Now Assist panel (Now Assist Admin → Now Assist Experiences → Now Assist panel → Turn on)
2. **Activate** the default assistant (Assistant Designer → Now Assist Panel - Platform (default) → Activate)
Both steps are one-time instance configuration — once completed, all subsequent agent testing from AI Agent Studio will work without further setup.
 
***