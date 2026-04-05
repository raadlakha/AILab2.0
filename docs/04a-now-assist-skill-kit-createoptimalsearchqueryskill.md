# 04a — NASK: CreateOptimalSearchQuery

> **Release:** Zurich | **Flow:** Fulfiller Flow — Phase 2, Path A, Step 1 **Source:** [Now Assist Skill Kit — ServiceNow Community](https://www.servicenow.com/community/now-assist-articles/now-assist-skill-kit-tool-and-deployment-options/ta-p/3284803) | [Create a custom skill — ServiceNow Docs](https://www.servicenow.com/docs/bundle/zurich-intelligent-experiences/page/administer/now-assist-skill-kit/concept/now-assist-skill-kit-landing.html)

***

## What It Is

**Now Assist Skill Kit (NASK)** is the custom skill authoring tool built into the ServiceNow platform. It lets you build generative AI skills that are not available in the standard out-of-the-box Now Assist catalogue — combining LLM prompts with pre-processing tools (Flow Actions, Subflows, Retrievers, Scripts) into a single deployable unit.

Each NASK skill follows a four-step authoring workflow:

```
1. Edit prompt      → author the LLM prompt and skill inputs
2. Add tools        → attach pre-processing tools to augment context
3. Optimize         → test and evaluate prompt performance
4. Deploy           → configure workflow, product, and publish
```

This section covers building the **`CreateOptimalSearchQuery`** skill — the first skill to fire in the Fulfiller Flow (Path A, Step 1).

***

## Role in the Fulfiller Flow

```
Fulfiller Flow — Phase 2 (triggered when state = In Progress AND
                          channel = chat AND
                          u_extracted_error_code ≠ empty)
        │
        ▼
Path A — Step 1 (parallel fire from workflow Start node):

  ┌────────────────────────────────────────────────────┐
  │  CreateOptimalSearchQuery skill fires              │
  │  ↓                                                 │
  │  GetIncidentExtendDetail (Flow Action)             │
  │  → reads Incident extend table fields              │
  │  → outputs: short_description, error_code, CI,     │
  │    description, product_name, serial_number, etc.  │
  │  ↓                                                 │
  │  GenerateOptimalPromptForRAG (Skill Prompt)        │
  │  → LLM constructs an optimised AI Search query     │
  │    from the incident context                       │
  └────────────────────────────────────────────────────┘
        │
        ▼  (output: optimised search query string ready for AI Search)
Path A — Step 2: RetrieveRelevantKBContent skill
        (receives the query → fetches ranked KB results via AI Search RAG)
```

> **Why this skill exists:** Raw error codes and messy incident case records make poor search queries. This skill transforms the structured incident data (error code, CI name, product, description) into an LLM-optimised query string that AI Search can rank meaningfully — improving KB retrieval quality significantly.

***

## Skill Architecture

The skill has two nodes on its canvas, executed in sequence:

| Node                          | Type         | Purpose                                                                                                                            |
| ----------------------------- | ------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| `GetIncidentExtendDetail`     | Flow Action  | Pre-processing tool — reads all relevant fields from the Incident extend record and outputs them as structured data for the prompt |
| `GenerateOptimalPromptForRAG` | Skill Prompt | LLM prompt — takes the Flow Action outputs as context and generates an optimised AI Search query string                            |

The skill takes one input — `incidentextendrecord` (the Incident record identifier) — and outputs a query string ready for the AI Search retriever in the next step.

***

## Prerequisites

| Requirement                           | Detail                                                                                               |
| ------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Now LLM Service or Azure OpenAI (We are using Azure OpenAI to build this skill in this lab document)       | LLM provider configured in the instance                                                              |
| `GetIncidentExtendDetail` Flow Action | Must exist — this is the Flow Action resource referenced by the tool                                 |
| Incident Extend table                 | `x_nava_agentic_lab_incident_extend` (or equivalent) — must be populated by the time the skill fires |

***

## Lab Exercise — Steps to Build

### Step 1: Navigate to Now Assist Skill Kit

Navigate to **All → Now Assist Skill Kit → Home** → click **Create skill**.

The **New skill** wizard opens on **General info**.

***

### Step 2: Configure General Info

Fill in the skill identity:

| Field       | Value                                                                                                           |
| ----------- | --------------------------------------------------------------------------------------------------------------- |
| Skill name  | `CreateOptimalSearchQuery`                                                                                      |
| Description | `This skill is created to generate the optimal search query for AI Search to be returned with the best results` |

> Skill names must use letters, numbers, dashes, and underscores only — no spaces or symbols.

**Choose default provider:**

The page presents a model picker. **Now LLM generic** is the recommended default (specialised in enterprise tasks, data analysis, and complex reasoning).

For this lab, the following provider is selected:

| Field                | Value                                                          |
| -------------------- | -------------------------------------------------------------- |
| Default provider     | `Azure OpenAI`                                                 |
| Default provider API | `Chat Completions`                                             |
| Available models     | gpt\_small, gpt-4-turbo, gpt-4o-mini, gpt-4o, gpt\_large, gpt4 |

![NASK — General Info: Skill Name, Description, and Provider](<../.gitbook/assets/NASKCreateOptimalSearchQuery1-1 (1).png>)

> The provider selection determines which LLM processes the prompt. Azure OpenAI is used here for its GPT model range. If your instance uses Now LLM Service natively, select that instead — the skill logic is provider-agnostic.

***

### Step 3: Configure Security Controls

Scroll down on the General info page to reach **Configure security controls**.

**Define user access with an Access Control List (ACL):**

| Field       | Value          |
| ----------- | -------------- |
| User access | `Select roles` |
| Roles       | `itil`         |

**Apply role restrictions to skill:**

| Field | Value  |
| ----- | ------ |
| Roles | `itil` |

![NASK — Security Controls: User Access and Role Restrictions](<../.gitbook/assets/NASKCreateOptimalSearchQuery1-2 (1).png>)

> **User access** controls who can invoke this skill. **Role restrictions** set the maximum privilege level the skill can inherit when it executes — even if the invoking user has broader roles, the skill operates within `itil` limits.

Click **Continue** to proceed to prompt creation.

***

### Step 4: Add Skill Input

Before authoring the prompt, define the skill input that will be passed in at runtime.

| Field                | Value                          |
| -------------------- | ------------------------------ |
| Datatype             | `String`                       |
| Name                 | `incidentextendrecord`         |
| Description          | `Table for extended incidents` |
| Make input mandatory | Unchecked                      |
| Allow truncation     | Unchecked                      |

The 'Make input mandatory' is currently unchecked, but as you progressively firm up your AI Agent / Agentic Workflow build, it can eventually be turned on (checked) to ensure more robustness in overall solutioning, as making the input mandatory will ensure that the flow runs smoothly.

![NASK — Skill Input: incidentextendrecord](<../.gitbook/assets/NASKCreateOptimalSearchQuery1-3 (1).png>)

> `incidentextendrecord` is the identifier passed in by the Fulfiller Flow when the skill is invoked. It references the Incident extend record that `GetIncidentExtendDetail` will query. This input is threaded through to the Flow Action tool as `{{incidentextendrecord}}`.

***

### Step 5: Rename the Prompt

When the skill is created, NASK auto-generates a default prompt named **Incident Summarization**. This needs to be renamed to reflect the skill's actual purpose.

1. In the **Skill contents** panel (left side), expand **Prompts** → **Azure OpenAI** → **Chat Completions**
2. Click on **Incident Summarization** to select it

![NASK — Edit Prompt: Default Name (Incident Summarization)](<../.gitbook/assets/NASKCreateOptimalSearchQuery1-15 (1).png>)

3. Click the **pencil icon** next to the prompt name to open the **Edit prompt name** dialog
4. Change the **Name** from `Incident Summarization` to `GenerateOptimalPromptForRAG`

![NASK — Edit Prompt Name Dialog](<../.gitbook/assets/NASKCreateOptimalSearchQuery1-16 (1).png>)

5. Click **Save changes**
6. Confirm the prompt name now displays as **GenerateOptimalPromptForRAG** in the Skill contents panel

![NASK — Prompt Renamed to GenerateOptimalPromptForRAG](<../.gitbook/assets/NASKCreateOptimalSearchQuery1-17 (1).png>)

> The prompt name is referenced throughout the skill — on the canvas (as the Skill prompt node label), in the Publish dialog, and in the finalized prompt selector. Renaming it now ensures consistency across all surfaces. This is also the prompt name that appears when you publish the skill in Step 14.

***

### Step 6: Add Tools — Canvas

After saving the skill input, navigate to the **Add tools** tab (Step 2 of the NASK wizard).

The canvas shows the execution flow: **Start → \[tool nodes] → End**. Click the **+** connector between Start and End to add a node.

Select **Tool node** and click **Add**.

![NASK — Add Node Dialog: Tool Node Selected](<../.gitbook/assets/NASKCreateOptimalSearchQuery1-4 (1).png>)

The tool type picker appears.

Select **Flow Action** as the tool type and click **Configure tool**.

![NASK — Tool Type: Flow Action](<../.gitbook/assets/NASKCreateOptimalSearchQuery1-5 (1).png>)

***

### Step 7: Configure the Flow Action Tool — General Info

The **Add flow action as a tool** wizard opens (5-step wizard: General info → Tool inputs → Tool outputs → Tool conditions → Summary).

**Step 1 — General info:**

| Field    | Value                                                      |
| -------- | ---------------------------------------------------------- |
| Name     | `GetIncidentExtendDetail`                                  |
| Resource | `Retrieval of Relevant Fields from Incident Extract table` |

![NASK — Flow Action Tool: General Info](<../.gitbook/assets/NASKCreateOptimalSearchQuery1-6 (1).png>)

> The **Resource** field references the Flow Action that reads the Incident extend table. Ensure ACLs are correctly configured on this resource — the platform shows a warning: _"Ensure that Access Control Lists (ACLs) are correctly configured for the resource so that the skill can access it."_

Click **Continue**.

***

### Step 8: Configure the Flow Action Tool — Tool Inputs

**Step 2 — Tool inputs:**

| Field    | Value                      |
| -------- | -------------------------- |
| Name     | `Incident Number`          |
| Datatype | `String`                   |
| Value    | `{{incidentextendrecord}}` |

![NASK — Flow Action Tool: Tool Inputs](<../.gitbook/assets/NASKCreateOptimalSearchQuery1-7 (1).png>)

> `{{incidentextendrecord}}` is the skill input defined in Step 4 — it is passed directly into the Flow Action here. This is how the runtime Incident record is threaded through the skill into the pre-processing tool.

Click **Continue**.

***

### Step 9: Configure the Flow Action Tool — Tool Outputs

**Step 3 — Tool outputs:**

The Flow Action returns the following outputs from the Incident extend table. All outputs flow into the prompt template as context variables:

| Output field         | Type       | Used for                                  |
| -------------------- | ---------- | ----------------------------------------- |
| Action Status        | Object     | Execution status of the Flow Action       |
| Short Description    | String     | Incident short description                |
| Don't Treat as Error | True/False | Error handling flag                       |
| Description          | String     | Full incident description                 |
| Configuration Item   | String     | Affected CI name                          |
| Error Code           | String     | Extracted error code (from NADI)          |
| Product Bar Code     | String     | Device barcode                            |
| Product Name         | String     | Device product name                       |
| Serial Number        | String     | Device serial number                      |
| Category             | String     | Incident category                         |
| Work Notes           | String     | Diagnostic notes from the L1 conversation |

![NASK — Flow Action Tool: Tool Outputs](<../.gitbook/assets/NASKCreateOptimalSearchQuery1-8 (1).png>)

> Truncation is available per output field — enabling truncation trims the field value if it exceeds the token limit, keeping the prompt intact. Leave all truncation unchecked unless you hit token limit issues in testing.

Click **Continue**.

***

### Step 10: Configure the Flow Action Tool — Tool Conditions

**Step 4 — Tool conditions (optional):**

| Field | Value                 |
| ----- | --------------------- |
| Type  | **None (Always run)** |

![NASK — Flow Action Tool: Tool Conditions](<../.gitbook/assets/NASKCreateOptimalSearchQuery1-9 (1).png>)

> Tool conditions let you specify whether the tool should run or be skipped based on a script or filter. **None (Always run)** means the `GetIncidentExtendDetail` Flow Action executes unconditionally every time the skill is invoked — the correct setting here since the prompt always needs the Incident context.

Click **Continue** → review the Summary → click **Add tool**.

***

### Step 11: Review Tool Summary

**Step 5 — Summary:**

Verify all fields before adding:

| Section         | Field           | Value                                                      |
| --------------- | --------------- | ---------------------------------------------------------- |
| Type            | —               | Flow Action                                                |
| General info    | Name            | `GetIncidentExtendDetail`                                  |
| General info    | Resource        | `Retrieval of Relevant Fields from Incident Extract table` |
| Inputs          | Incident Number | `{{incidentextendrecord}}`                                 |
| Outputs         | (all 11 fields) | String / Object / True/False as defined                    |
| Tool conditions | Type            | none                                                       |

![NASK — Flow Action Tool: Summary](<../.gitbook/assets/NASKCreateOptimalSearchQuery1-10 (1).png>)

Click **Add tool**.

***

### Step 12: Verify Canvas

After adding the tool, the canvas updates to show the full skill execution flow:

```
Start
  │
  ▼
GetIncidentExtendDetail   ← Flow Action (Tool node)
  │
  ▼
GenerateOptimalProm...    ← Skill Prompt node (auto-created)
  │
  ▼
End
```

![NASK — Canvas: Full Skill Flow](<../.gitbook/assets/NASKCreateOptimalSearchQuery1-11 (1).png>)

The left panel shows **Tools** → `GetIncidentExtendDetail` / Flow Action.

> The **Skill prompt** node (`GenerateOptimalProm...` = `GenerateOptimalPromptForRAG`) is the LLM prompt step. Navigate to **Step 1: Edit prompt** to author the prompt template — reference the Flow Action outputs using `{{GetIncidentExtendDetail.field_name}}` syntax (e.g. `{{GetIncidentExtendDetail.short_description}}`, `{{GetIncidentExtendDetail.error_code}}`).

***

### Step 13: Author the Prompt

Navigate back to **Step 1: Edit prompt** in the NASK wizard tab bar. Click on the **GenerateOptimalPromptForRAG** prompt in the Skill contents panel to open the prompt editor.

The prompt for this skill is provided in the lab repository. Copy the full prompt text from the following link listed in step 1 and paste it into the **Prompt** text area:


1. Open the file at [`../NASKprompts/CreateOptimalSearchQuery-Prompt`](https://raw.githubusercontent.com/raadlakha/AILab2.0/main/NASKprompts/CreateOptimalSearchQuery-Prompt) in the lab repository
2. Copy the **entire contents** of the file and read through the contents of the prompt. Remember that GenAI based systems are intelligent systems who adapt to their environment, and that every environment can differ. So read the prompt and see if there are things you want to include/exclude from it to best suit the environment that you are building. Do not copy blindly!
3. In the NASK prompt editor, paste the copied text into the **Prompt** field
4. Verify that the prompt references the Flow Action output variables using `{{GetIncidentExtendDetail.<field_name>}}` syntax — for example:
   * `{{GetIncidentExtendDetail.short_description}}`
   * `{{GetIncidentExtendDetail.description}}`
   * `{{GetIncidentExtendDetail.configuration_item}}`
   * `{{GetIncidentExtendDetail.error_code}}`
   * `{{GetIncidentExtendDetail.product_name}}`
   * `{{GetIncidentExtendDetail.serial_number}}`
   * `{{GetIncidentExtendDetail.category}}`
   * `{{GetIncidentExtendDetail.work_notes}}`
5. Click **Save** to save the prompt
6. Click **Manage prompt** → **Finalize prompt** to lock the prompt as version 1

> **Why finalize?** The prompt must be finalized before it can be selected in the Publish dialog (Step 15). Finalizing creates an immutable version (`GenerateOptimalPromptForRAG (v1)`) — you can continue editing the draft and finalize again to create v2, v3, etc. Only finalized versions are available for publishing.

> **Prompt design note:** The prompt instructs the LLM to take the structured incident fields (error code, CI name, product details, work notes, description) and synthesise them into a single, keyword-optimised search query string suitable for AI Search. It is not a summarisation task — the output is a search query, not a summary. This distinction is critical for downstream KB retrieval quality.

***

### Step 14: Configure Deployment Settings

Navigate to **Step 4: Deployment and skill settings** → select **Deployment settings** from the left nav.

| Field    | Value                          |
| -------- | ------------------------------ |
| Workflow | `Other`                        |
| Product  | (leave blank — not applicable) |

![NASK — Deployment Settings: Workflow](<../.gitbook/assets/NASKCreateOptimalSearchQuery1-13 (1).png>)

> The **Workflow** field determines where this skill appears in the Now Assist Admin → Skills catalogue. Setting it to **Other** places the skill under the **Other** category — making it findable when activating it for use in Flow Designer.

***

### Step 15: Publish the Skill

Click **Publish skill** (top right of the skill editor).

The **Publish Skill** dialog opens:

Review the deployment settings summary:

| Field           | Value          |
| --------------- | -------------- |
| Workflow        | Other          |
| Product         | Not Applicable |
| Feature         | Not Applicable |
| Display Options | None           |

Under **Select which finalized prompts to include in the Published skill:**

| Provider     | Prompt                                              | Action    |
| ------------ | --------------------------------------------------- | --------- |
| Azure OpenAI | `GenerateOptimalPromptForRAG (v1)` — Default prompt | ✅ Checked |

![NASK — Publish Skill Dialog](<../.gitbook/assets/NASKCreateOptimalSearchQuery1-12 (1).png>)

Click **Publish**.

> A prompt must be **finalized** before it can be selected here. If no finalized prompts appear, return to Step 1 (Edit prompt), complete the prompt, and click **Finalize prompt** before returning to publish.

***

### Step 16: Activate the Skill

After publishing, navigate to **All → Admin Center → Now Assist Admin → Now Assist Skills** tab.

Locate `CreateOptimalSearchQuery` under the **Other** workflow (it will show **Custom | Not started | Now LLM Service**). Click **Turn on** → set role restrictions as required → confirm activation.

![Now Assist Admin — Skills: CreateOptimalSearchQuery](<../.gitbook/assets/NASKCreateOptimalSearchQuery1-14 (1).png>)

> The skill must be **Active** in Now Assist Admin for it to be callable as a Flow Action from the Fulfiller Flow in Flow Designer. Publishing alone is not sufficient — activation is a separate step.

***

## Key Configuration Summary

| Field                   | Value                                                      |
| ----------------------- | ---------------------------------------------------------- |
| Skill name              | `CreateOptimalSearchQuery`                                 |
| Skill type              | Custom skill                                               |
| Default provider        | Azure OpenAI / Now LLM generic                             |
| Skill input             | `incidentextendrecord` (String)                            |
| Tool                    | `GetIncidentExtendDetail` — Flow Action                    |
| Tool resource           | `Retrieval of Relevant Fields from Incident Extract table` |
| Tool input              | `Incident Number` → `{{incidentextendrecord}}`             |
| Tool outputs            | 11 fields from Incident extend table.                      |
| Tool condition          | None (Always run)                                          |
| Prompt                  | `GenerateOptimalPromptForRAG (v1)`                         |
| Workflow (deployment)   | Other                                                      |
| User access             | Select roles → `itil`                                      |
| Role restrictions       | `itil`                                                     |
| Status after activation | Active                                                     |

***

## Technical Notes

### Flow Action Outputs as Prompt Context

All 11 outputs from `GetIncidentExtendDetail` are available as template variables in the skill prompt using the syntax `{{GetIncidentExtendDetail.<output_name>}}`. The prompt uses these to construct a semantically rich query — combining the error code, CI name, product details, and work notes into a single optimised string that AI Search can rank accurately.

### Why "Retrieval of Relevant Fields from Incident Extract table"?

The standard Incident table does not contain the custom fields added by NADI (`error_code`, product barcode, serial number, etc.). The `GetIncidentExtendDetail` Flow Action reads from the **Incident extend table** (`x_nava_agentic_lab_incident_extend`) which stores these enriched fields — giving the LLM the full device and error context it needs to generate a precise query.

### Tool Condition: None (Always run)

Tool conditions can be used to skip a tool based on a script evaluation or filter — for example, only running a retriever if a field is non-empty. For this skill, `None (Always run)` is correct: the Flow Action must always fire because the prompt has no fallback context without it.

### Finalize vs Publish

In NASK, prompts go through a two-stage lifecycle: **Finalize** locks the prompt version (creating v1, v2, etc.) and **Publish** deploys the skill with selected finalized prompts to the Now Assist ecosystem. You can have multiple finalized versions and choose which to publish. The published version is what runs in production.

***

## Reference

* [Now Assist Skill Kit — Tool and Deployment Options](https://www.servicenow.com/community/now-assist-articles/now-assist-skill-kit-tool-and-deployment-options/ta-p/3284803)
* [Now Assist Skill Kit FAQ](https://www.servicenow.com/community/now-assist-articles/now-assist-skill-kit-nask-faq/ta-p/3007953)
* [Creating a Custom Skill with NASK — Part 1](https://www.servicenow.com/community/developer-blog/creating-a-custom-skill-with-now-assist-skill-kit-part-1/ba-p/3448719)
* [Trigger a Custom Skill via Flow — Part 1](https://www.servicenow.com/community/developer-blog/trigger-a-custom-now-assist-skill-via-flow-part-1/ba-p/3453557)

***

## Next Steps

→ The published `CreateOptimalSearchQuery` skill outputs an optimised query string that feeds directly into the next step of the Fulfiller Flow.

→ Continue to the next section to configure the [04b - ResolutionFinderUsingInternalData](04b-now-assist-skill-kit-part2-resolutionfinderinternaldataskill.md) skill (Path A — Step 2), which takes this query and fetches ranked Knowledge Base results using the AI Search RAG retriever and PI.
