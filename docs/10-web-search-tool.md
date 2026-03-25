# 10 — Web Search Tool

> **Source:** [ServiceNow Zurich Documentation — Add tools and information to an AI agent](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/add-tool-aia.html)

## What It Is

**Web Search** is a built-in tool type available to AI agents in AI Agent Studio. It allows an AI agent to execute a web search query against the internet and receive back results that it can reason over and synthesise into a response.

In Zurich, the Web Search tool supports **Google** as the search provider, powered by **Grounding with Google Search**. The tool is the final step in the Resolution Pathfinder's search sequence — used only when internal KB and Elastic log analysis have not produced a resolution.

The Resolution Pathfinder's workflow with this tool:
1. `GenerateWebSearchQuery` skill produces a privacy-safe query string (Capability 09)
2. `WebSearch` tool executes that query
3. The AI agent synthesises the top results into actionable resolution steps
4. The resolution plan + source URL is written to the Incident work notes

---

## When to Use

- As a last-resort fallback when internal knowledge sources have no answer
- When the error code or pattern is well-documented in public technical forums or vendor documentation
- When you want the AI agent to cite a public URL in the resolution plan for transparency
- When the internal KB is incomplete or does not yet cover the specific error

---

## Customer Value

| Business Outcome | How Web Search Delivers It |
|-----------------|----------------------------|
| Resolution when internal KB fails | Public docs (vendor guides, Stack Overflow, GitHub) often have answers internal KB doesn't |
| Cited sources | Public URLs are written to the Incident — L2 engineers can verify and follow the source |
| No manual googling | Agent does the search; engineers receive the synthesis |
| Graceful degradation | If even web search fails, the agent writes what it searched and flags for manual investigation |

---

## How It Works — Architecture

```
Resolution Pathfinder AI Agent
        │
        │ (Internal KB: no resolution found)
        │ (Elastic logs: no root cause found)
        │
        ▼
Tool call: GenerateWebSearchQuery → returns "HOST_UNREACHABLE Linux troubleshooting fix"
        │
        ▼
Tool call: WebSearch
        │ query: "HOST_UNREACHABLE Linux troubleshooting fix"
        │ provider: Google (Grounding with Google Search)
        │
        ▼
Returns: top N search results with titles, snippets, URLs
        │
        ▼
AI Agent synthesises results into resolution steps
Writes to Incident work_notes:
---
RESOLUTION PLAN
Source: Web Search
Steps:
1. ...
2. ...
Citation: https://example.com/host-unreachable-fix
---
```

---

## How to Build

### Prerequisites

- Now Assist AI Agents activated (Now Assist Pro+)
- Google Grounding with Google Search enabled on the instance (or configured via ServiceNow)
- Role: `sn_aia.admin` or `admin`

> **Data residency note:** The Web Search tool sends queries to Google's infrastructure. Ensure your organisation's data residency and privacy policies permit this for your use case. No incident-specific internal data should be in the query — this is enforced by the `GenerateWebSearchQuery` skill in Capability 09.

---

## Lab Exercise — Steps to Build

### Step 1: Register the Web Search Tool in the Resolution Pathfinder AI Agent

The Web Search tool is a built-in type — no separate configuration is required before adding it to an agent.

1. Navigate to **All** > **AI Agent Studio** > **Create and manage** > **AI agents**
2. Open **Resolution Pathfinder**
3. Under **Tools**, click **Add tool**
4. Select type: **Web Search**
5. Configure:

| Field | Value |
|-------|-------|
| Tool name | `WebSearch` |
| Description | `Executes the generated search query against the internet using Google and returns top results for the AI agent to synthesise into resolution steps` |
| Search provider | Google (Grounding with Google Search) |
| Max results | `5` |

6. **Save**

### Step 2: Verify the Tool Sequence in the Agent Instructions

Open the **Resolution Pathfinder** agent and confirm the system prompt (from Capability 06) instructs the agent to:
1. Call `GenerateWebSearchQuery` first to get the safe query string
2. Pass that query to `WebSearch`
3. Synthesise the results into numbered resolution steps
4. Include the source URL in the resolution plan

The relevant section of the agent instructions:
```
3. Use the GenerateWebSearchQuery tool to create an optimised search query,
   then use the WebSearch tool to search the internet.
   - Synthesise the top results into actionable resolution steps.
   - Write the resolution plan and source URL to the Incident work notes.
```

### Step 3: Test the Web Search Tool

1. Navigate to **AI Agent Studio** > **Test**
2. Select a test Incident where:
   - `u_extracted_error_code` = `HOST_UNREACHABLE`
   - Internal resolution is intentionally disabled (to force the web search path)
3. Run the Resolution Pathfinder agent
4. Verify the `work_notes` on the Incident include:
   - `Source: Web Search`
   - Numbered resolution steps
   - A citation URL from a public source

---

## Key Configuration Fields

| Field | Value for This Lab |
|-------|--------------------|
| Tool name | WebSearch |
| Tool type | Web Search |
| Search provider | Google (Grounding with Google Search) |
| Max results | 5 |
| Used by agent | Resolution Pathfinder (Tool 4, after GenerateWebSearchQuery) |
| Privacy protection | Query is pre-sanitised by GenerateWebSearchQuery skill |

---

## Reference

- [ServiceNow Zurich — Add tools to an AI agent](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/add-tool-aia.html)

---

## Next Step

Continue to [11 — Elastic MCP Server](11-elastic-mcp-server.md) to configure the Model Context Protocol Client connection that gives the Resolution Pathfinder access to live server log data.
