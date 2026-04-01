# 04c — NASK: GenerateWebSearchQnsForResolutionPlan

> **Release:** Zurich | **NASK Plugin Version:** 3.1.3+ (Xanadu Patch 7 / Yokohama Patch 1 minimum for Web Search tool) **Flow:** Veritas Resolution Agentic Workflow — Fallback Path (after internal KB and Elastic log analysis) **Sources:** [NASK Tool & Deployment Options — ServiceNow Community](https://www.servicenow.com/community/now-assist-articles/now-assist-skill-kit-tool-and-deployment-options/ta-p/3284803) | [Now Assist Skill Kit Docs — Zurich](https://www.servicenow.com/docs/bundle/zurich-intelligent-experiences/page/administer/now-assist-skill-kit/concept/now-assist-skill-kit-landing.html)

***

## What This Doc Covers

This document walks through building the **`GenerateWebSearchQnsForResolutionPlan`** skill — a custom Now Assist skill that bridges internal incident data and external internet search. It is the **fallback resolution path** in the Veritas workflow: invoked when AI Search (RAG over KB) has not produced a resolution.

The skill does two things in sequence:

1. Calls a **pre-processing tool** — another published NASK skill (`CreateOptimalSearchQuery`) — to retrieve a privacy-safe, LLM-optimised search query from the incident
2. Passes that query to the **skill prompt** which generates a structured set of targeted web search questions to drive an actionable Resolution Plan

This is a **skill-chaining** pattern: a skill calling another published skill as a tool — one of the most powerful composition patterns available in NASK from version 3.0.1 onward.

***

## Where This Fits in the Veritas Architecture

```
Veritas Resolution Agentic Workflow
        │
        ▼
Phase 1: AI Search (KB RAG) — CreateOptimalSearchQuery → RetrieveRelevantKBContent
        │
        ▼ (no resolution found)
Phase 2: Elastic Log Analysis — ElasticLogAnalysis skill (Capability 11)
        │
        ▼ (no resolution found)
Phase 3: Internet Fallback ← YOU ARE HERE
        │
  ┌─────────────────────────────────────────────────────────────┐
  │  GenerateWebSearchQnsForResolutionPlan skill fires          │
  │                                                             │
  │  Tool node: RetrieveGeneratedSearchQuerythatwasforAI        │
  │  → calls CreateOptimalSearchQuery skill (published)         │
  │  → input: {{incidentextendrecord}}                          │
  │  → output: response (the optimised query string)            │
  │                                                             │
  │  Skill prompt: Generate Web Search Questions for            │
  │  Resolution Plan                                            │
  │  → takes tool output + incident context                     │
  │  → generates structured web search question set             │
  └─────────────────────────────────────────────────────────────┘
        │
        ▼
  Output: set of web search questions → Resolution Plan
```

***

## Skill Overview

| Field                 | Value                                                                                                                                                            |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Skill name            | `GenerateWebSearchQnsForResolutionPlan`                                                                                                                          |
| Skill type            | Custom skill                                                                                                                                                     |
| Default provider      | Now LLM Service — Now LLM Generic                                                                                                                                |
| Available models      | llm\_generic\_small\_v2, llm\_generic\_small\_v1, llm\_generic\_large, llm\_generic\_small, pii\_anonymize\_v2, llm\_generic\_large\_v1, llm\_generic\_large\_v2 |
| Input                 | `incidentextendrecord` (String) — Record Number from Incident Extend table                                                                                       |
| Outputs               | provider, response, error, errorCode, status (all String)                                                                                                        |
| Tool                  | `RetrieveGeneratedSearchQuerythatwasforAI` → calls `CreateOptimalSearchQuery` skill                                                                              |
| Tool condition        | None (Always run)                                                                                                                                                |
| Prompt                | `Generate Web Search Questions for Resolution Plan (v2)`                                                                                                         |
| Workflow (deployment) | Other                                                                                                                                                            |
| User access           | Any authenticated user                                                                                                                                           |
| Role restrictions     | `itil`                                                                                                                                                           |
| Status                | Active                                                                                                                                                           |

***

## Understanding Web Search in NASK

Before building this skill it's important to understand what the **Web Search tool type** is — and why it is _not_ used directly here.

### The NASK Web Search Tool (Native)

NASK introduced a native **Web Search** tool type in version **3.1.3** (Xanadu Patch 7 / Yokohama Patch 1). It allows a custom skill to directly call an external internet search API and inject results into the prompt context. It supports two modes:

| Mode                  | How it works                                                                                          |
| --------------------- | ----------------------------------------------------------------------------------------------------- |
| **Search and Scrape** | Queries a search engine, scrapes the top results, and injects summarised content into the prompt      |
| **AI Answers**        | Queries a search engine and generates an AI-synthesised answer from results, injected into the prompt |

**Important constraint:** The native Web Search tool requires your organisation to **provide and maintain its own API keys** for the external search service (e.g. Bing Search API, Google Custom Search API, Brave Search, etc.). ServiceNow does not provide search API credentials.

> **Why this matters for the lab:** In this AILab2.0 setup, instead of using the native Web Search tool directly (which requires a paid external API key), the skill uses a **skill-chaining approach** — calling `CreateOptimalSearchQuery` as a tool to retrieve the pre-generated query, then constructing targeted web search questions via the LLM prompt. The actual web search execution is handled downstream in the Veritas agent workflow (Capability 10: Web Search Tool).

### Skill as a Tool (Skill Chaining)

Available from NASK version **3.0.1** (Xanadu Patch 3). This allows a published skill to be called as a tool node inside another skill. Key rules:

* The target skill **must be published** to appear as an option
* Any inputs required by the target skill must be mapped in the tool configuration
* Skills can use different LLMs — no restriction on chaining across providers
* The tool node exposes the called skill's **outputs** (provider, response, error, errorCode, status) for use in the parent skill's prompt

***

## Lab Exercise — Step-by-Step Build

### Prerequisites

| Requirement                      | Detail                                                               |
| -------------------------------- | -------------------------------------------------------------------- |
| NASK plugin                      | `sn_now_assist_skill_kit` v3.0.1+ — Active                           |
| `CreateOptimalSearchQuery` skill | Must be **published** (not just saved) — it is called as a tool here |
| Extended Incident table          | `x_nava_agentic_lab_incident_extend` populated                       |
| Role                             | `sn_skill_kit.admin` or `admin`                                      |
| Now LLM Service                  | Configured and active on instance                                    |

***

### Step 1: Create the Skill — General Info

Navigate to **All → Now Assist Skill Kit → Home → Create skill**.

The **New skill** wizard opens on **General info**.

![General info — skill name and description](../.gitbook/assets/NASK-generatewebsearchquestion-step1.png)

Configure the skill identity:

| Field       | Value                                                                                                                                                                                                                          |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Skill name  | `GenerateWebSearchQnsForResolutionPlan`                                                                                                                                                                                        |
| Description | `This skill is created to take in the 'Optimal Search Query' that was created to search against ServiceNow's AI Search and then create a set of web search questions in order to drive towards an actionable Resolution Plan.` |

> Skill names use letters, numbers, dashes, and underscores — no spaces or symbols. The description surfaces in Now Assist Admin and the Skill catalogue.

**Choose default provider:**

| Field                | Value             |
| -------------------- | ----------------- |
| Default provider     | `Now LLM Service` |
| Default provider API | `Now LLM Generic` |

The available models shown under Now LLM Generic include the full NowLLM model family: `llm_generic_small_v2`, `llm_generic_small_v1`, `llm_generic_large`, `llm_generic_small`, `pii_anonymize_v2`, `llm_generic_large_v1`, `llm_generic_large_v2`.

> **Now LLM Generic** is flagged as "Recommended" — it is specialised for enterprise tasks, data analysis, and complex reasoning. For India-region instances, NowLLM routes to the appropriate regional cluster. See the [Now LLM availability guide](08-now-assist-ai-search-deep-dive.md) for region-specific LLM routing details.

***

### Step 2: Configure Security Controls

Scroll down on the General info page to reach **Configure security controls**.

![Security controls — ACL and role restrictions](../.gitbook/assets/NASK-generatewebsearchquestion-step2.png)

**Define user access with an ACL:**

| Field       | Value                    |
| ----------- | ------------------------ |
| User access | `Any authenticated user` |

> "As long as a user is logged in, they can access this skill." This is appropriate for an internet-facing fallback skill used by fulfiller agents — any authenticated ITIL user in the Veritas workflow can invoke it.

**Apply role restrictions to skill:**

| Field | Value  |
| ----- | ------ |
| Roles | `itil` |

> Role restrictions cap the maximum privilege level the skill inherits during execution — even if the invoking user has admin, the skill runs within `itil` bounds. This is a security best practice: skills should operate with least-privilege.

Click **Continue**.

***

### Step 3: Skill Published — Prompt Canvas

After creation the skill lands on **Step 1: Edit prompt**.

![Skill editor — prompt canvas showing published status](../.gitbook/assets/NASK-generatewebsearchquestion-step3.png)

Note the **Published** badge in the header — this skill was published as part of a prior iteration. For a new build, status will show **Draft** at this point. The skill editor has four tabs:

| Tab                              | Purpose                                                  |
| -------------------------------- | -------------------------------------------------------- |
| 1. Edit prompt                   | Author the LLM prompt, define inputs and outputs         |
| 2. Add tools                     | Add pre-processing tool nodes to the execution canvas    |
| 3. Optimize and evaluate         | Test the prompt, review grounded prompt and tool outputs |
| 4. Deployment and skill settings | Configure workflow placement, security, and publish      |

The **Skill contents** panel on the left shows:

* **inputs (1):** `incidentextendrecord` (String)
* **Outputs (5):** provider, response, error, errorCode, status
* **Prompts (1):** Now LLM Service → Now LLM Generic → `Generate Web Search Questions for Resolution Plan`

The **input modal** (shown in this screenshot) confirms:

| Field                | Value                                      |
| -------------------- | ------------------------------------------ |
| Datatype             | String                                     |
| Name                 | `incidentextendrecord`                     |
| Description          | `Record Number from Incident Extend table` |
| Make input mandatory | Unchecked                                  |
| Allow truncation     | Unchecked                                  |
| Test value           | `INCE0011002`                              |

> `incidentextendrecord` is the runtime identifier — the Incident record number — passed in when the Veritas agent invokes this skill. It threads through to the tool as `{{incidentextendrecord}}`.

***

### Step 4: Add Tools — Open Tool Canvas

Navigate to **Step 2: Add tools**.

The canvas shows **Start → Skill prompt → End**. Click the **+** connector between Start and the Skill prompt node to insert a tool node before the prompt fires.

![Add node dialog — Tool node selection](../.gitbook/assets/NASK-generatewebsearchquestion-step4.png)

Select **Tool node** and click **Add**.

The tool type picker opens.

![Add skill as a tool — Tool type picker](../.gitbook/assets/NASK-generatewebsearchquestion-step5.png)

Select **Skill** as the tool type and click **Configure tool**.

> This activates the **skill-chaining** pattern — a published NASK skill used as a pre-processing tool to inject its output into the parent skill's prompt context.

***

### Step 5: Configure Skill Tool — General Info

The **Add skill as a tool** wizard opens (5-step: General info → Tool inputs → Tool outputs → Tool conditions → Summary).

![Add skill as a tool — General info](../.gitbook/assets/NASK-generatewebsearchquestion-step6.png)

**Step 1 — General info:**

| Field        | Value                                                                                                           |
| ------------ | --------------------------------------------------------------------------------------------------------------- |
| Name         | `RetrieveGeneratedSearchQuerythatwasforAI`                                                                      |
| Description  | `This skill is created to generate the optimal search query for AI Search to be returned with the best results` |
| Resource     | `CreateOptimalSearchQuery`                                                                                      |
| Provider API | `Now LLM Generic` (auto-populated from the resource skill's provider)                                           |

> The **Resource** field references the published `CreateOptimalSearchQuery` skill (built in [doc 03-now-assist-skill-kit.md](03-now-assist-skill-kit.md)). A platform warning appears: _"Ensure that Access Control Lists (ACLs) are correctly configured for the resource so that the skill can access it."_ — verify that `CreateOptimalSearchQuery` has `itil` as its role restriction and `Any authenticated user` access, matching the parent skill.

Click **Continue**.

***

### Step 6: Configure Skill Tool — Tool Inputs

![Add skill as a tool — Tool inputs](../.gitbook/assets/NASK-generatewebsearchquestion-step7.png)

**Step 2 — Tool inputs:**

| Name                   | Datatype | Value                      |
| ---------------------- | -------- | -------------------------- |
| `incidentextendrecord` | string   | `{{incidentextendrecord}}` |

> This maps the parent skill's input directly through to the child skill. `{{incidentextendrecord}}` is the Mustache-style variable syntax NASK uses to reference skill inputs at runtime. The child skill (`CreateOptimalSearchQuery`) receives the same incident record identifier and uses it to retrieve the incident extend fields and generate the optimal query.

Click **Continue**.

***

### Step 7: Configure Skill Tool — Tool Outputs

![Add skill as a tool — Tool outputs](../.gitbook/assets/NASK-generatewebsearchquestion-step8.png)

**Step 3 — Tool outputs:**

The called skill (`CreateOptimalSearchQuery`) returns the standard NASK skill output set:

| Name        | Type   | Description                                                                              |
| ----------- | ------ | ---------------------------------------------------------------------------------------- |
| `provider`  | string | LLM provider used by the child skill                                                     |
| `response`  | string | **The generated optimal search query string** — primary output used by the parent prompt |
| `error`     | string | Error message if the child skill failed                                                  |
| `errorCode` | string | Error code if applicable                                                                 |
| `status`    | string | Execution status (success / error)                                                       |

> `response` is the key output. It contains the privacy-safe, LLM-generated search query produced by `CreateOptimalSearchQuery`. In the parent skill's prompt template, this is referenced as `{{RetrieveGeneratedSearchQuerythatwasforAI.response}}`. Truncation is available per field — leave unchecked unless token limits are hit in testing.

Click **Continue**.

***

### Step 8: Configure Skill Tool — Tool Conditions

![Add skill as a tool — Tool conditions](../.gitbook/assets/NASK-generatewebsearchquestion-step9.png)

**Step 4 — Tool conditions (optional):**

| Field | Value                 |
| ----- | --------------------- |
| Type  | **None (Always run)** |

> Tool conditions allow you to skip a tool based on a script or filter evaluation — for example, skipping the web search if a retriever already found results (a common pattern using Decision Nodes). For this skill, `None (Always run)` is correct: the child skill must always fire because the parent prompt has no search query context without it.

Click **Continue**.

***

### Step 9: Review Tool Summary

![Add skill as a tool — Summary](../.gitbook/assets/NASK-generatewebsearchquestion-step10.png)

**Step 5 — Summary:**

| Section         | Field                | Value                                                                                                           |
| --------------- | -------------------- | --------------------------------------------------------------------------------------------------------------- |
| Type            | —                    | Skill                                                                                                           |
| General info    | Name                 | `RetrieveGeneratedSearchQuerythatwasforAI`                                                                      |
| General info    | Description          | `This skill is created to generate the optimal search query for AI Search to be returned with the best results` |
| General info    | Resource             | `CreateOptimalSearchQuery`                                                                                      |
| General info    | Provider API         | Now LLM Generic                                                                                                 |
| Inputs          | incidentextendrecord | `{{incidentextendrecord}}`                                                                                      |
| Outputs         | provider             | string                                                                                                          |
| Outputs         | response             | string                                                                                                          |
| Outputs         | error                | string                                                                                                          |
| Outputs         | errorCode            | string                                                                                                          |
| Outputs         | status               | string                                                                                                          |
| Tool conditions | Type                 | none                                                                                                            |

Click **Add tool**.

***

### Step 10: Verify the Execution Canvas

After adding the tool, the canvas updates to show the full skill execution flow:

![Tool canvas with skill tool and skill prompt nodes](../.gitbook/assets/NASK-generatewebsearchquestion-step11.png)

```
Start
  │
  ▼
RetrieveGeneratedSearchQuerythatwas...   ← Skill tool node (calls CreateOptimalSearchQuery)
  │
  ▼
Generate Web Search ...                  ← Skill prompt node
  │
  ▼
End
```

The left panel shows **Tools** → `RetrieveGeneratedSearchQuerythaswas` (truncated display name).

> The canvas execution order matters: the tool node fires **before** the prompt node. This ensures `CreateOptimalSearchQuery` has run and its `response` output is available for the prompt template to reference via `{{RetrieveGeneratedSearchQuerythatwasforAI.response}}` before the LLM processes the prompt.

***

### Step 11: Author the Skill Prompt

Navigate back to **Step 1: Edit prompt** to view and edit the prompt template.

![Prompt editor — Generate Web Search Questions for Resolution Plan](../.gitbook/assets/NASK-generatewebsearchquestion-step12.png)

The prompt is visible in the **Skill contents** panel:

```
Prompt name: Generate Web Search Questions for Resolution Plan
Provider: Now LLM Service → Now LLM Generic
Word count: 1026 words (as configured)
```

**Prompt preamble (visible in screenshot):**

```
You are an AI agent that excels at generating web-search-optimised queries for
customer-reported issues. Given a structured incident report, you produce a
single, high-impact search query designed to be executed against an internet
search engine to retrieve the troubleshooting articles, guides, and resolution
procedures needed to build an actionable Resolution Plan for the customer's issue.

---

## INPUT FORMAT

You will receive a structured incident report containing the following fixed fields:

- **ISSUE**: A concise summary of the problem.
- **SYMPTOMS**: Observable behaviours, error patterns, and any remediation already attempted.
- **ERROR**: The specific error code or status message reported by the system.
- **SYSTEM**: The product, appliance model, or platform affected.
- **CATEGORY**: The classification of the problem domain (e.g., Hardware, Software, Network).
- **DESCRIPTION**: A detailed narrative of the problem, including environmental context and actions taken so far.
```

> The full prompt (1026 words) structures the LLM's task in detail — defining the input schema, output format, privacy rules (no internal hostnames/IPs/user data), and examples. The prompt references `{{RetrieveGeneratedSearchQuerythatwasforAI.response}}` to inject the query generated by the tool node.

**Key prompt design decisions:**

| Decision                                   | Rationale                                                                                                             |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------- |
| Structured INPUT FORMAT with fixed fields  | Forces consistent context injection regardless of incident data quality                                               |
| Privacy constraints in prompt rules        | Prevents internal system details leaking into externally-visible search queries                                       |
| Output: single optimised query string      | Downstream Web Search tool (Capability 10) expects a single query, not a list                                         |
| `web-search-optimised` framing in preamble | Guides the LLM to produce queries with search-engine semantics, not NLP questions                                     |
| 1026-word prompt                           | Reflects the complexity of the task — privacy rules, output format, and multiple examples require verbose instruction |

***

### Step 12: Publish the Skill

Navigate to **Step 4: Deployment and skill settings** → click **Publish skill** (top right).

The **Publish Skill** dialog opens:

![Publish Skill dialog](../.gitbook/assets/NASK-generatewebsearchquestion-step13.png)

Review deployment settings:

| Field           | Value          |
| --------------- | -------------- |
| Workflow        | Other          |
| Product         | Not Applicable |
| Feature         | Not Applicable |
| Display Options | None           |

Under **Select which finalized prompts to include in the Published skill:**

| Provider                                             | Prompt                                                   | Action    |
| ---------------------------------------------------- | -------------------------------------------------------- | --------- |
| Now LLM Service (Now LLM Generic) — Default provider | `Generate Web Search Questions for Resolution Plan (v2)` | ✅ Checked |

Click **Publish**.

> A prompt must be **finalized** before it appears here. Finalization locks a prompt version (v1, v2, etc.). Publishing deploys the skill to the Now Assist ecosystem. These are separate actions — you can have multiple finalized versions and choose which to publish.

***

### Step 13: Configure Deployment Settings

In **Step 4: Deployment and skill settings**, select **Deployment settings** from the left nav.

![Deployment settings — Workflow = Other](../.gitbook/assets/NASK-generatewebsearchquestion-step14.png)

| Field    | Value         |
| -------- | ------------- |
| Workflow | `Other`       |
| Product  | (leave blank) |

> **Workflow = Other** places the skill in the **Now Assist Skills → Other** category in Now Assist Admin. This is the correct placement for custom skills that are used programmatically (by agents or flows) rather than being deployed to a specific product surface (ITSM, HRSD, etc.).

***

### Step 14: Activate the Skill

Navigate to **All → Admin Center → Now Assist Admin → Now Assist Skills → Other**.

![Now Assist Admin — Skills — Other — GenerateWebSearchQnsForResolutionPlan](../.gitbook/assets/NASK-generatewebsearchquestion-step15.png)

Locate `GenerateWebSearchQnsForResolutionPlan` under the **Available** tab in the **Other** workflow. Status shows **Custom | Inactive | Now LLM Service**.

Click **Turn on**.

![Successfully activated confirmation](../.gitbook/assets/NASK-generatewebsearchquestion-step16.png)

The **"Successfully activated"** confirmation dialog appears:

> **GenerateWebSearchQnsForResolutionPlan is now active**

Click **Done**.

> **Publishing alone is not sufficient.** The skill must be **Active** in Now Assist Admin for it to be callable by agents, flows, or as a tool by other skills. Activation creates the runtime link between the published skill and the Now Assist execution engine.

***

## Key Configuration Summary

| Field                 | Value                                                       |
| --------------------- | ----------------------------------------------------------- |
| Skill name            | `GenerateWebSearchQnsForResolutionPlan`                     |
| Skill type            | Custom skill                                                |
| Default provider      | Now LLM Service → Now LLM Generic                           |
| Skill input           | `incidentextendrecord` (String) — test value: `INCE0011002` |
| Tool type             | Skill (skill-chaining)                                      |
| Tool name             | `RetrieveGeneratedSearchQuerythatwasforAI`                  |
| Tool resource         | `CreateOptimalSearchQuery` (published skill)                |
| Tool input            | `incidentextendrecord` → `{{incidentextendrecord}}`         |
| Tool key output       | `response` — the optimised query string                     |
| Tool condition        | None (Always run)                                           |
| Prompt                | `Generate Web Search Questions for Resolution Plan (v2)`    |
| Prompt size           | \~1026 words                                                |
| Workflow (deployment) | Other                                                       |
| User access           | Any authenticated user                                      |
| Role restrictions     | `itil`                                                      |
| Final status          | Active                                                      |

***

## Technical Deep Dive

### Skill Chaining: How Output Variables Flow

When NASK executes a skill-as-tool, the child skill's outputs become available as template variables in the parent prompt using dot notation:

```
{{<tool_name>.<output_field>}}

Example:
{{RetrieveGeneratedSearchQuerythatwasforAI.response}}
```

This is how the parent prompt receives the optimal query string generated by `CreateOptimalSearchQuery`. The LLM processing the parent prompt sees the query inline within the prompt context — not as a separate API call.

### Execution Order and Token Budget

The tool node fires **before** the LLM processes the prompt. The grounded prompt (what the LLM actually receives) contains the fully resolved tool outputs substituted into the template. This means:

* Tool execution failures surface before the LLM is called
* Token budget must accommodate both the prompt template **and** all tool output values
* For this skill: the `CreateOptimalSearchQuery` response is a short query string (typically 5–15 words) — minimal token overhead

### Why Not Use the Native Web Search Tool?

The native NASK Web Search tool (v3.1.3+) directly calls an external search API. In the Veritas lab context:

| Factor                    | Native Web Search                  | Skill-Chaining Approach                                           |
| ------------------------- | ---------------------------------- | ----------------------------------------------------------------- |
| External API key required | Yes — org must provide and manage  | No — no external dependency                                       |
| Search execution          | Happens inside the skill           | Happens in the downstream Veritas agent (Capability 10)           |
| Query optimisation        | Passes raw input to search API     | LLM-generated, privacy-safe query from `CreateOptimalSearchQuery` |
| Privacy control           | Must configure domain restrictions | LLM strips internal identifiers in the upstream skill             |
| Suitable for              | Standalone web-augmented skills    | Multi-agent workflows where search is a separate agent action     |

> In a production agentic workflow, the two approaches are complementary: the native Web Search tool is ideal for self-contained skills that need real-time web grounding; the skill-chaining pattern is ideal when web search is one step in a larger orchestrated resolution workflow.

### Decision Nodes — Advanced Pattern

For production hardening of this skill, consider adding a **Decision Node** (available from NASK v3.0.1) between the tool node and the prompt node:

```
Start
  │
  ▼
RetrieveGeneratedSearchQuery (tool)
  │
  ▼
Decision Node: did tool return a non-empty response?
  ├── TRUE → Skill prompt (generate web search questions)
  └── FALSE → End (or fallback skill)
```

This prevents the LLM prompt from firing if `CreateOptimalSearchQuery` failed or returned an empty response — saving LLM calls and producing a cleaner error state.

### Troubleshooting

Use the **Test prompt** section in Step 3 (Optimize and evaluate) to debug:

| Tab                 | What it shows                                                                                                                                                              |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Grounded prompt** | The full prompt delivered to the LLM, with all tool outputs substituted in — use this to verify `{{RetrieveGeneratedSearchQuerythatwasforAI.response}}` resolved correctly |
| **Tools**           | Input sent to and response received from each tool individually — use this to verify `CreateOptimalSearchQuery` is returning a valid query                                 |

Common issues:

| Symptom                                 | Likely cause                                                | Fix                                                               |
| --------------------------------------- | ----------------------------------------------------------- | ----------------------------------------------------------------- |
| Tool output is empty                    | `CreateOptimalSearchQuery` not published or inactive        | Publish and activate the upstream skill                           |
| ACL warning on Resource field           | Skill ACL mismatch                                          | Ensure both skills have matching role restrictions (`itil`)       |
| Skill not appearing in Now Assist Admin | Not activated                                               | Run Turn on in Now Assist Admin → Other                           |
| Empty response from skill               | `incidentextendrecord` test value not found in extend table | Verify INCE0011002 exists in `x_nava_agentic_lab_incident_extend` |

***

## Reference

* [NASK Tool & Deployment Options — ServiceNow Community](https://www.servicenow.com/community/now-assist-articles/now-assist-skill-kit-tool-and-deployment-options/ta-p/3284803)
* [NASK FAQ — ServiceNow Community](https://www.servicenow.com/community/now-assist-articles/now-assist-skill-kit-nask-faq/ta-p/3007953)
* [Add a tool — ServiceNow Docs](https://www.servicenow.com/docs/csh?topicname=add-a-tool.html\&version=latest)
* [Add Web Search tool — ServiceNow Docs](https://www.servicenow.com/docs/csh?topicname=add-web-search.html\&version=latest)
* [03 — CreateOptimalSearchQuery (upstream skill)](03-now-assist-skill-kit.md)
* [09 — Web Search Query Generator (agent-level)](09-web-search-query-generator.md)
* [10 — Web Search Tool (Veritas agent)](10-web-search-tool.md)

***

## Next Steps

→ This skill outputs a set of structured web search questions targeting the incident's unresolved error.

→ Continue to [10 — Web Search Tool](10-web-search-tool.md) to configure the Web Search execution step in the Veritas agent that takes these questions, executes them against the internet, and synthesises a Resolution Plan.

→ If you want to understand the upstream query generation: see [03 — Now Assist Skill Kit: CreateOptimalSearchQuery](03-now-assist-skill-kit.md).
