# 12 — Observability and Action Agent (External Agent via A2A Protocol)

> **Source:** [ServiceNow Zurich Documentation — Create an external AI agent with the Agent2Agent protocol](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/create-a2a-agent.html)

## What It Is

The **Observability and Action Agent** is an **External AI Agent** registered in ServiceNow AI Agent Studio using the **Agent2Agent (A2A) protocol** from the ServiceNow AI Agent Fabric. It is the final stage of the incident resolution pipeline.

Once the Resolution Pathfinder AI Agent has produced a resolution plan, the Observability and Action Agent:
1. Receives the resolution plan from the Resolution Pathfinder via the Agentic Workflow
2. Delegates execution of the remediation steps to an **Azure AI Foundry** agent
3. The Azure AI Foundry agent runs the actual remediation actions against the target infrastructure (e.g., restart a server, flush routing tables, clear disk space)
4. Returns the remediation outcome — success, failure, or partial — back to ServiceNow
5. The result is written to the Incident record as the final work note

The A2A protocol is an open standard for agent-to-agent interoperability. ServiceNow implements it as an **External** agent type in AI Agent Studio. The Azure AI Foundry agent exposes an **Agent Card** (`/.well-known/agent.json`) that ServiceNow discovers and activates.

> **Prerequisite:** Your ServiceNow instance must be on **Zurich patch 4** or later.

---

## When to Use

- When remediation actions need to run on infrastructure that lives outside ServiceNow (Azure, AWS, GCP, on-prem)
- When a third-party agentic AI platform (Azure AI Foundry, LangChain, Bedrock Agents) handles the execution layer
- When you want closed-loop automation — ServiceNow diagnoses, external agent acts, ServiceNow records the result
- When the external agent already exists and you want to integrate it without rebuilding it inside ServiceNow

---

## Customer Value

| Business Outcome | How the Observability and Action Agent Delivers It |
|-----------------|---------------------------------------------------|
| Fully autonomous resolution | AI diagnoses the issue and executes the fix — no human intervention required for known errors |
| Closed-loop ITSM | The remediation result is written back to the Incident automatically — complete audit trail |
| Leverage existing investments | Azure AI Foundry agents already built for ops teams are reused via A2A — no duplication |
| Faster MTTR | Remediation starts seconds after the resolution plan is generated — no waiting for L2 to act |
| Interoperability | A2A protocol is open-standard — works with any A2A-compliant external agent, not just Azure |

---

## How It Works — Architecture

```
Resolution Pathfinder AI Agent
        │
        │ Resolution plan generated, written to Incident work_notes
        │
        ▼
Agentic Workflow (Observability & Action stage)
        │
        ▼
Observability and Action Agent (External type — A2A)
        │
        ├── ServiceNow AI Agent Fabric
        │   A2A Protocol call →
        │
        ▼
Azure AI Foundry Agent
        │
        ├── Receives: resolution plan steps, incident context (CI, error code)
        │
        ├── Executes remediation:
        │   e.g., restart service on prod-app-01
        │   e.g., flush routing tables for VLAN
        │   e.g., clear disk space on affected server
        │
        └── Returns: { status: "success", actions_taken: [...], timestamp: ... }
        │
        ▼
ServiceNow — result written to Incident work_notes:
---
REMEDIATION RESULT
Agent: Azure AI Foundry — Observability and Action Agent
Status: Success
Actions taken:
1. Restarted network interface on prod-app-01
2. Cleared ARP cache
Completed: 2024-03-15 14:38:22 UTC
---
```

---

## How to Build

### Prerequisites

- ServiceNow instance on **Zurich patch 4** or later
- AI Agent Studio enabled with external agent access: **AI Agent Studio** > **Settings** > **External AI Agents** > **Discoverability** > Enable "Allow ServiceNow to access External AI Agents"
- Azure AI Foundry agent deployed and A2A-compliant, exposing `/.well-known/agent.json`
- Connection & Credential alias configured with credentials to access the Azure AI Foundry agent endpoint
- Role: `sn_aia.admin` or `admin`

---

## Lab Exercise — Steps to Build

### Step 1: Enable External Agent Discoverability

1. Navigate to **All** > **AI Agent Studio** > **Settings**
2. Under **External AI Agents** > **Discoverability**, enable:

| Setting | Value |
|---------|-------|
| Allow ServiceNow to access External AI Agents | Enabled |

3. **Save**

### Step 2: Create the External AI Agent

1. Navigate to **All** > **AI Agent Studio** > **Create and manage** > **AI agents**
2. Click **New** > **External**
3. Select protocol: **Agent2Agent (A2A)**

This launches a guided 3-step wizard.

---

### Step 3: Discover and Activate (Step 1 of wizard)

#### Add the Azure AI Foundry Provider

1. Click **Add new provider**
2. Fill in:

| Field | Value |
|-------|-------|
| Name | `Azure AI Foundry — Observability Agent` |
| Agent card URL | `https://<your-azure-foundry-endpoint>/.well-known/agent.json` |
| Under Advanced settings: Connection & Credential alias | *(select or create alias for Azure credentials)* |
| Under Advanced settings: Select subflow | *(leave as default — handles majority of A2A cases)* |

3. Click **Save**

#### Discover the Agent

1. Click **Discover external AI agent**
2. ServiceNow calls the Agent Card URL and validates the connection
3. On success, the agent **Name** and **Version** from the Agent Card appear below the button
4. Click the agent name to verify:
   - **Name**: Observability and Action Agent
   - **Version**: *(from Agent Card)*
   - **Description**: *(from Agent Card)*
   - **Skills**: listed from Agent Card (e.g., `run_remediation`, `check_server_status`)
5. Click **Activate**
6. Click **Save and continue**

---

### Step 4: Review Skills and Capabilities (Step 2 of wizard)

1. Verify the skill names match what the Azure AI Foundry agent declares in its Agent Card
2. Confirm the skill descriptions are accurate — these help the AI Agent Orchestrator decide when to delegate to this agent
3. Click **Save and continue**

---

### Step 5: Define the Specialty (Step 3 of wizard)

| Field | Value |
|-------|-------|
| AI agent description | `Receives a structured resolution plan from the Resolution Pathfinder and executes remediation actions against IT infrastructure via Azure AI Foundry. Returns the remediation outcome including actions taken and final status.` |
| Communication mode | **Synchronous** *(or Asynchronous if remediation may take >30 seconds)* |
| Connection & Credential alias | *(select alias for executing calls to the Azure AI Foundry agent endpoint)* |

#### Configure ACLs

| Setting | Value |
|---------|-------|
| User access | Users with specific roles |
| Roles | `sn_aia.admin`, `itil` |

> **Note:** ACL configuration is required. If this step is skipped, the agent cannot be invoked from an Agentic Workflow.

Click **Save and test** to complete setup and go to the Testing page.

---

### Step 6: Add to the Resolution Pathfinder Agentic Workflow

The Observability and Action Agent is added as the next stage in the existing Agentic Workflow built in Capability 06.

1. Navigate to **All** > **AI Agent Studio** > **Create and manage** > **Agentic workflows**
2. Open **Resolution Pathfinder Workflow**
3. Add a new stage after the Resolution Pathfinder agent stage:

| Field | Value |
|-------|-------|
| Stage name | `Remediation Execution` |
| Agent | `Azure AI Foundry — Observability Agent` |
| Trigger condition | Resolution Pathfinder has written a resolution plan (`work_notes` is not empty on the Incident) |
| Input | Pass the resolution plan steps and incident context (CI, error code) to the external agent |

4. Under **Output Actions**, configure the result to be written back to the Incident:

| Field | Mapping |
|-------|---------|
| Operation | Update |
| Table | `incident` |
| Field | `work_notes` |
| Value | Remediation result from Azure AI Foundry agent |

5. **Save** the workflow

---

### Step 7: Test the End-to-End Pipeline

1. Create a test Incident:
   - `state` = In Progress
   - `contact_type` = chat
   - `u_extracted_error_code` = `HOST_UNREACHABLE`
   - `cmdb_ci` = a CI that exists in your Azure environment
2. Save the record to trigger the Agentic Workflow
3. Verify the full pipeline executes:
   - Resolution Pathfinder finds a resolution plan (from KB, logs, or web)
   - Observability and Action Agent receives the plan via A2A
   - Azure AI Foundry agent executes the remediation steps
   - Result is written back to the Incident `work_notes`
4. Check the Incident for two work notes:
   - Resolution plan (from Resolution Pathfinder)
   - Remediation result (from Observability and Action Agent)

---

## Key Configuration Fields

| Field | Value for This Lab |
|-------|--------------------|
| Agent type | External |
| Protocol | Agent2Agent (A2A) |
| External platform | Azure AI Foundry |
| Agent Card URL | `https://<azure-foundry>/.well-known/agent.json` |
| Communication mode | Synchronous or Asynchronous |
| Authentication | OAuth 2.0 or API Key (via Connection & Credential alias) |
| Role required | `sn_aia.admin` |
| Zurich minimum | Patch 4 |
| Output | Remediation result written to Incident `work_notes` |

---

## Agent Card — What Azure AI Foundry Must Expose

For ServiceNow to discover and activate the external agent, the Azure AI Foundry agent must expose an A2A-compliant Agent Card at `/.well-known/agent.json`. Example structure:

```json
{
  "name": "Observability and Action Agent",
  "version": "1.0.0",
  "description": "Executes IT infrastructure remediation actions based on a provided resolution plan. Supports server restarts, network resets, and disk management operations.",
  "skills": [
    {
      "name": "run_remediation",
      "description": "Executes a structured list of remediation steps against a specified CI",
      "inputModes": ["text"],
      "outputModes": ["text"]
    },
    {
      "name": "check_server_status",
      "description": "Checks the current status of a server CI",
      "inputModes": ["text"],
      "outputModes": ["text"]
    }
  ],
  "capabilities": {
    "streaming": false,
    "pushNotifications": false
  }
}
```

> **Note:** The Azure AI Foundry team is responsible for building and maintaining this Agent Card and the execution endpoint. This ServiceNow lab covers only the ServiceNow-side registration.

---

## A2A Protocol vs MCP Client — Clarification

| Feature | A2A Protocol (Cap. 12) | MCP Client (Cap. 11) |
|---------|----------------------|---------------------|
| Purpose | Agent-to-agent delegation | Agent accessing external tools |
| Direction | ServiceNow delegates a task to an external AI agent | ServiceNow agent calls a specific tool on an external server |
| External side | Full AI agent (Azure AI Foundry, LangChain, etc.) | Tool server (Elastic, APIs) |
| Protocol | Agent2Agent (A2A) open standard | Model Context Protocol (MCP) |
| Discovery | Agent Card (`/.well-known/agent.json`) | MCP tool list endpoint |
| Use when | External agent handles the full reasoning + execution loop | External server provides a specific callable function |

---

## Reference

- [ServiceNow Zurich — Create an external AI agent with the Agent2Agent protocol](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/create-a2a-agent.html)
- [Create an agentic workflow](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/configure-use-case-ai-agents.html)
- [Now Assist AI agents overview](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/na-ai-agents.html)

---

## Lab Complete

You have now built all 12 capabilities of the AI-Powered IT Incident Resolution lab — from user intake through to autonomous remediation. Return to the [README](../README.md) for the complete end-to-end architecture.
