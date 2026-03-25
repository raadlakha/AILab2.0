# 11 — Elastic MCP Server (Model Context Protocol Client)

> **Source:** [ServiceNow Zurich Documentation — Model Context Protocol Client](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/mcp-client.html) | [Add an MCP server in AI Agent Studio](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/add-mcp-client-on-ai-agent-studio.html)

## What It Is

The **Model Context Protocol (MCP) Client** in ServiceNow enables AI agents built in AI Agent Studio to access tools hosted on **external MCP servers**. This capability allows the Resolution Pathfinder AI Agent to query a live **Elastic** log server for server log entries related to the extracted error code — without ServiceNow needing to own or manage the Elastic infrastructure.

As described in the Zurich documentation:

> *"The ServiceNow® Model Context Protocol (MCP) client enables you to access the Model Context Protocol tools that are hosted externally and published using an MCP server in the ServiceNow® AI Agent Studio."*

In this lab, Elastic exposes a log query API as an MCP server. ServiceNow's MCP Client connects to it, making that tool available as `ElasticLogSearch` inside the Resolution Pathfinder AI Agent. This is Tool 2 in the resolution search sequence — invoked when internal KB has no answer, to check live production logs before falling back to internet search.

> **Important:** The **MCP Client** (for AI agents consuming external tools) is different from the **MCP Server Console** (for external applications consuming ServiceNow tools). This lab covers the **MCP Client** — ServiceNow as the consumer.

---

## When to Use

- When an AI agent needs to access tools hosted outside ServiceNow (Elastic, Splunk, Datadog, custom APIs)
- When live operational data (logs, metrics, traces) should inform AI agent resolution decisions
- When external systems expose MCP-compatible APIs and you want to avoid building custom integrations
- When you want to extend AI agent capabilities without writing ServiceNow integrations

---

## Customer Value

| Business Outcome | How MCP Client Delivers It |
|-----------------|----------------------------|
| Real-time log analysis | AI agent queries live Elastic logs, not static data — catches active issues |
| No custom integration code | MCP Client connects to any MCP-compliant server without bespoke development |
| Governance and security | Authentication (OAuth 2.1, API Key) is managed centrally in AI Agent Studio |
| Extensible architecture | Same pattern works for Splunk, Datadog, Prometheus, custom REST APIs |
| Root cause from logs | Log entries often contain the definitive root cause — faster resolution than KB alone |

---

## How It Works — Architecture

```
Resolution Pathfinder AI Agent
        │
        │ (Internal KB: PATH B — no resolution found)
        │
        ▼
Tool call: ElasticLogSearch (MCP Tool)
        │
        │ Parameters: { error_code: "HOST_UNREACHABLE", server: "prod-app-01" }
        │
        ▼
ServiceNow MCP Client
        │
        ├── Authentication: API Key (or OAuth 2.1)
        ├── Connection: Elastic MCP Server (external)
        │
        ▼
Elastic MCP Server
        │  Runs query: logs WHERE error_code=HOST_UNREACHABLE AND host=prod-app-01
        │  Returns: matching log entries with timestamps
        │
        ▼
MCP Client returns results to AI Agent
        │
        ▼
AI Agent analyses log entries, identifies root cause
Writes to Incident work_notes
```

---

## How to Build

### This Capability Covers: ServiceNow Side Only

This lab covers the **ServiceNow configuration** required to register and use the Elastic MCP server. It assumes Elastic has already been configured to expose an MCP-compatible endpoint. (Elastic configuration is outside the scope of this ServiceNow lab.)

---

### Prerequisites

- Now Assist AI Agents plugin (`sn_aia`) installed and active
- Elastic instance with an MCP server endpoint exposed
- API Key or OAuth 2.1 credentials for the Elastic MCP server
- Role: `sn_aia.admin` or `admin`

---

## Lab Exercise — Steps to Build

### Step 1: Install the MCP Client Plugin

1. Navigate to **All** > search **Plugins**
2. Search for **Model Context Protocol Client** (`sn_mcp_client`)
3. Click **Install / Activate**
4. Wait for installation to complete

> **Prerequisite check:** Confirm `sn_aia` (Now Assist AI Agents) is installed before installing the MCP Client plugin.

### Step 2: Register the Elastic MCP Server in AI Agent Studio

1. Navigate to **All** > **AI Agent Studio** > **Create and manage** > **AI agents**
2. Open **Resolution Pathfinder** agent
3. Under **Tools**, click **Add tool**
4. Select type: **MCP Tool**
5. Click **Add MCP Server**

This launches the MCP server registration flow.

### Step 3: Add the MCP Server

Select the authentication method for your Elastic MCP server:

#### Option A: API Key Authentication (recommended for lab)

1. Select **API Key**
2. Fill in:

| Field | Value |
|-------|-------|
| Server name | `Elastic Log Server` |
| MCP server URL | `https://your-elastic-instance.example.com/mcp` |
| API Key | *(your Elastic API key)* |
| Description | `Elastic log server for IT infrastructure log analysis` |

3. Click **Connect** — ServiceNow will validate the connection
4. Once connected, the available MCP tools from the Elastic server are listed

#### Option B: OAuth 2.1 Authentication

1. Select **OAuth 2.1**
2. Fill in:

| Field | Value |
|-------|-------|
| Server name | `Elastic Log Server` |
| MCP server URL | `https://your-elastic-instance.example.com/mcp` |
| Client ID | *(OAuth client ID from Elastic)* |
| Authorization URL | *(Elastic OAuth authorization endpoint)* |
| Token URL | *(Elastic OAuth token endpoint)* |

3. Click **Authenticate** — completes OAuth 2.1 flow
4. Once authenticated, available tools are listed

### Step 4: Add the MCP Tool to the Agent

After registering the server:

1. From the tool list returned by the Elastic MCP server, select the log query tool
2. Configure:

| Field | Value |
|-------|-------|
| Tool name | `ElasticLogSearch` |
| MCP Server | Elastic Log Server *(registered in Step 3)* |
| Description | `Queries Elastic for log entries related to the extracted error code and affected server` |
| Input parameters | `{ "error_code": "{{u_extracted_error_code}}", "server": "{{cmdb_ci}}" }` |

3. **Save**

### Step 5: Verify the Tool in the Agent Instructions

Confirm the Resolution Pathfinder's system prompt (Capability 06) includes instructions for `ElasticLogSearch`:

```
2. Use the ElasticLogSearch tool to query the Elastic log server for entries
   related to the extracted error code and affected server.
   - If log analysis reveals a root cause and resolution steps, write them
     to the Incident work notes and stop.
   - If no conclusive finding, proceed to step 3.
```

### Step 6: Test the MCP Connection

1. Navigate to **AI Agent Studio** > **Test**
2. Select a test Incident with:
   - `u_extracted_error_code` = `HOST_UNREACHABLE`
   - `cmdb_ci` = a server CI that exists in Elastic logs
3. Run the Resolution Pathfinder agent
4. Verify:
   - The agent calls `ElasticLogSearch`
   - Log entries are returned
   - If root cause is found in logs: `work_notes` shows `Source: Log Analysis`
   - If no root cause in logs: agent proceeds to web search (Cap. 10)

---

## Key Configuration Fields

| Field | Value for This Lab |
|-------|--------------------|
| Plugin | Model Context Protocol Client (`sn_mcp_client`) |
| Prerequisite | `sn_aia` (Now Assist AI Agents) |
| Authentication | API Key or OAuth 2.1 |
| MCP Server | Elastic Log Server |
| Tool name in agent | ElasticLogSearch |
| Tool type | MCP Tool |
| Used by agent | Resolution Pathfinder (Tool 2) |

---

## MCP Client vs MCP Server Console — Clarification

| Component | Purpose | Direction |
|-----------|---------|-----------|
| **MCP Client** *(this capability)* | ServiceNow AI agent consumes external tools | ServiceNow → External |
| **MCP Server Console** | External AI applications consume ServiceNow tools | External → ServiceNow |

This lab builds the **MCP Client** configuration — ServiceNow calls out to Elastic.

---

## Reference

- [ServiceNow Zurich — Model Context Protocol Client](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/mcp-client.html)
- [Configure MCP Client](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/configuring-mcp-client.html)
- [Add an MCP server in AI Agent Studio](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/add-mcp-client-on-ai-agent-studio.html)
- [Add MCP tool](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/add-mcp-server-tool.html)

---

## Lab Complete

You have now built all 11 capabilities of the AI-Powered IT Incident Resolution lab. Return to the [README](../README.md) for the end-to-end testing guide.
