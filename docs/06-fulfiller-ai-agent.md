# 06 — Resolution Pathfinder (AI Agent + Agentic Workflow)

> **Source:** [ServiceNow Zurich Documentation — Now Assist AI agents](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/na-ai-agents.html) | [Create an agentic workflow](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/configure-use-case-ai-agents.html)

## What It Is

The **Resolution Pathfinder** is an AI agent operating in the background, on behalf of the IT team (fulfiller side). In Zurich, this is implemented as a **Chat AI Agent** participating in an **Agentic Workflow** that is triggered automatically when an Incident meets specific conditions.

Two components are built here:
1. **Resolution Pathfinder AI Agent** — the AI entity with tools and instructions
2. **Resolution Pathfinder Agentic Workflow** — the workflow that defines the trigger and orchestrates the agent

The agent's goal is to build a complete, actionable resolution plan by searching across three data sources in sequence:

**Search sequence:**
1. **Internal systems** — Knowledge Graph + Now Assist Skill `ResolutionFinderUsingInternalData`
2. **External log data** — Elastic MCP Server for log entries related to the error code
3. **Internet** — Web search as a last resort

Once a resolution plan is built, the agent writes it — along with source citations — directly to the Incident work notes. The Agentic Workflow then continues to the **Observability and Action Agent** (Capability 12), which executes the remediation steps via Azure AI Foundry.

**Agentic Workflow trigger conditions:**
- Incident state = `In Progress`
- `contact_type` = `chat`
- `u_extracted_error_code` is not empty

---

## When to Use

- When you want AI to attempt resolution before a human even looks at the ticket
- When resolution data exists across multiple sources (KB, logs, internet) and you want all of it searched automatically
- When MTTR reduction is a key metric — this pattern cuts hours of manual investigation to seconds
- When you want audit-ready resolution documentation written to the ticket automatically

---

## Customer Value

| Business Outcome | How the Resolution Pathfinder Delivers It |
|-----------------|------------------------------------------|
| Dramatic MTTR reduction | AI searches all sources in seconds vs. engineers spending hours investigating |
| Auto-documented resolution | Resolution plan + citations written to Incident — no manual note-taking |
| Intelligent escalation | If no resolution found, agent documents what was searched and why it failed |
| Cross-source intelligence | Combines internal KB, live log data, and internet knowledge in one pass |
| Consistent L2 handoff | L2 engineers receive tickets with full investigation context already done |

---

## How It Works — Architecture

```
Agentic Workflow Trigger: Incident (In Progress, contact_type=chat, u_extracted_error_code≠empty)
        │
        ▼
Resolution Pathfinder AI Agent (Chat type)
        │
        ├──► Tool 1: ResolutionFinderInternalData (Now Assist Skill)
        │         Knowledge Graph + RAG search
        │         PATH A: resolution found → write to Incident work_notes → stop
        │         PATH B: not found → continue to Tool 2
        │
        ├──► Tool 2: ElasticLogSearch (MCP Tool)
        │         Query Elastic log server for error_code + server
        │         If root cause found → write to Incident work_notes → stop
        │         If no conclusive finding → continue to Tool 3
        │
        └──► Tool 3: GenerateWebSearchQuery (Now Assist Skill)
                  Generate privacy-safe search query
                  → Tool 4: WebSearch
                        Synthesise top results
                        Write resolution plan + URL to Incident work_notes
        │
        ▼
Agentic Workflow continues →
Observability and Action Agent (Cap. 12)
        │ A2A Protocol → Azure AI Foundry
        ▼
Remediation executed, result written to Incident work_notes
```

---

## How to Build

### Prerequisites

- AI Agent Studio enabled (Now Assist Pro+)
- Capabilities 03, 07, 08, 09, 10, 11 configured
- Role: `sn_aia.admin` or `admin`
- Elastic MCP Server registered (Capability 11)

---

## Lab Exercise — Steps to Build

### Step 1: Create the AI Agent

1. Navigate to **All** > **AI Agent Studio** > **Create and manage** > **AI agents**
2. Click **Add** > **Chat**
3. Configure:

| Field | Value |
|-------|-------|
| Name | `Resolution Pathfinder` |
| Description | `Automatically investigates IT Incidents by searching internal KB, log systems, and the internet. Writes a resolution plan to the Incident work notes.` |
| Agent type | **Chat** |

### Step 2: Define the Specialty

| Field | Value |
|-------|-------|
| Name | `Incident Resolution Investigator` |
| Description | `Searches Knowledge Graph, Elastic logs, and the internet for resolution steps for IT incidents` |
| AI agent role | `You are the Resolution Pathfinder. You investigate IT incidents and build resolution plans.` |

### Step 3: Write the Agent Instructions (System Prompt)

```
You are the Resolution Pathfinder, an AI agent responsible for building resolution plans for IT Incidents. You are triggered automatically when an Incident is In Progress and contains an extracted error code.

Your investigation follows this strict sequence:

1. Use the ResolutionFinderInternalData tool to search the internal knowledge base and similar resolved incidents using the incident's extracted error code and description.
   - If a viable resolution is found (PATH A), write the resolution steps and source to the Incident work notes and stop.
   - If no resolution is found (PATH B), proceed to step 2.

2. Use the ElasticLogSearch tool to query the Elastic log server for entries related to the extracted error code and affected server.
   - If log analysis reveals a root cause and resolution steps, write them to the Incident work notes and stop.
   - If no conclusive finding, proceed to step 3.

3. Use the GenerateWebSearchQuery tool to create an optimised search query, then use the WebSearch tool to search the internet.
   - Synthesise the top results into actionable resolution steps.
   - Write the resolution plan and source URL to the Incident work notes.

Always write your findings to the Incident work notes in this format:
---
RESOLUTION PLAN
Source: [Internal KB / Log Analysis / Web Search]
Steps:
1. [Step]
2. [Step]
...
Citation: [Article title or URL]
---

If all three sources fail to produce a resolution, write:
"No automated resolution found. Manual investigation required. Searched: Internal KB, Log System, Web."
```

### Step 4: Add Tools to the Agent

#### Tool 1: Resolution Finder — Internal Data

| Field | Value |
|-------|-------|
| Tool name | `ResolutionFinderInternalData` |
| Type | Now Assist Skill |
| Skill | `ResolutionFinderUsingInternalData` *(configured in Cap. 08)* |
| Description | `Searches internal Knowledge Graph and similar resolved incidents using RAG` |

#### Tool 2: Elastic Log Search

| Field | Value |
|-------|-------|
| Tool name | `ElasticLogSearch` |
| Type | MCP Tool |
| MCP Server | Elastic MCP Server *(configured in Cap. 11)* |
| Description | `Queries Elastic for log entries related to the extracted error code and affected server` |

#### Tool 3: Web Search Query Generator

| Field | Value |
|-------|-------|
| Tool name | `GenerateWebSearchQuery` |
| Type | Now Assist Skill |
| Skill | `GenerateWebSearchQnsForResolutionPlan` *(configured in Cap. 09)* |
| Description | `Generates a single optimised, privacy-safe web search query from the incident details` |

#### Tool 4: Web Search

| Field | Value |
|-------|-------|
| Tool name | `WebSearch` |
| Type | Web Search *(configured in Cap. 10)* |
| Description | `Executes the generated search query against the internet and returns top results` |

### Step 5: Configure the Agentic Workflow

1. Navigate to **All** > **AI Agent Studio** > **Create and manage** > **Agentic workflows**
2. Click **New**
3. Configure:

| Field | Value |
|-------|-------|
| Name | `Resolution Pathfinder Workflow` |
| Description | `Triggers the Resolution Pathfinder agent when an Incident is In Progress with an extracted error code` |
| Trigger type | Record-based |
| Table | `incident` |
| Condition | `state = 2 AND contact_type = chat AND u_extracted_error_code != ""` |
| Run as | System (background) |

4. Under **Agents**, add the `Resolution Pathfinder` agent
5. Set the agent's **role** in the workflow to handle the resolution investigation

### Step 6: Configure Output — Write to Incident

1. Under the agentic workflow **Output Actions**, add **Record operation**
2. Configure:

| Field | Mapping |
|-------|---------|
| Operation | Update |
| Table | `incident` |
| Field | `work_notes` |
| Value | Agent's resolution plan output |

### Step 7: Test the Agent

1. Manually create a test Incident with:
   - `state` = In Progress
   - `contact_type` = chat
   - `u_extracted_error_code` = `HOST_UNREACHABLE`
   - `cmdb_ci` = any server CI
2. Save the record
3. Navigate to **AI Agent Studio** > **Test** > select the Incident
4. Run the agentic workflow manually
5. Verify `work_notes` is updated with a resolution plan
6. Check the source cited (Internal KB / Log Analysis / Web Search)

---

## Key Configuration Fields

| Field | Value for This Lab |
|-------|--------------------|
| Agent type | Chat |
| Role required | `sn_aia.admin` |
| Agentic Workflow trigger | state=In Progress, contact_type=chat, u_extracted_error_code≠empty |
| Tools | ResolutionFinderInternalData, ElasticLogSearch, GenerateWebSearchQuery, WebSearch |
| Output | work_notes on Incident record |

---

## Reference

- [ServiceNow Zurich — Now Assist AI agents](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/na-ai-agents.html)
- [Create an agentic workflow](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/configure-use-case-ai-agents.html)
- [Add tools and information](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/add-tool-aia.html)

---

## Next Step

Continue to [07 — Flow Action Tool](07-flow-action-tool.md) to build the structured tool that extracts incident details and feeds them into both AI Agents.

After completing capabilities 07–11 (individual tools and integrations), continue to [12 — Observability and Action Agent](12-observability-action-agent.md) to add autonomous remediation execution via Azure AI Foundry.
