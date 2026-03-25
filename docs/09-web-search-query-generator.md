# 09 — Web Search Query Generator (Now Assist Skill)

> **Source:** [ServiceNow Zurich Documentation — Now Assist Skill Kit](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/now-assist-skill-kit/now-assist-skill-kit-landing.html)

## What It Is

**GenerateWebSearchQnsForResolutionPlan** is a **Now Assist Skill** that generates a single, optimised, privacy-safe web search query from incident details. It is Tool 3 in the Resolution Pathfinder's search sequence — invoked only when both internal knowledge (Capability 08) and Elastic log analysis (Capability 11) have failed to produce a resolution.

The skill's purpose is to translate internal, potentially sensitive incident data into a safe, effective internet search query. It removes:
- Server hostnames and internal IP addresses
- User names and personal identifiers
- Internal application names or proprietary system names

And replaces them with generic, searchable terms that will return useful public documentation.

---

## When to Use

- When internal KB and log analysis have not found a resolution
- When the error code or pattern is a known industry-standard error that public documentation covers
- When you want to search the internet without leaking internal system details
- When the AI agent's resolution search needs a final fallback before declaring "no resolution found"

---

## Customer Value

| Business Outcome | How This Skill Delivers It |
|-----------------|----------------------------|
| Privacy-safe internet search | Internal hostnames, IPs, and user data are stripped before any external query |
| Higher relevance results | Optimised query phrasing returns better results than raw incident text |
| Reduces data leakage risk | LLM generates the query — no raw incident data leaves ServiceNow directly |
| Automated query formulation | No human needs to write the search query; AI handles it |

---

## How It Works — Technical Overview

```
AI Agent calls GenerateWebSearchQnsForResolutionPlan skill
        │
        Input: u_extracted_error_code, short_description
        │
        ▼
LLM generates privacy-safe search query
        │
        Examples:
        Input:  "HOST_UNREACHABLE on prod-app-01.internal.company.com"
        Output: "HOST_UNREACHABLE Linux server troubleshooting resolution steps"
        │
        Input:  "ERR_NETWORK_TIMEOUT connecting to database via 10.0.0.45"
        Output: "ERR_NETWORK_TIMEOUT database connection timeout fix"
        │
        ▼
Returns: single search query string
        │
        ▼
AI Agent passes query to WebSearch tool (Capability 10)
```

---

## How to Build

### Prerequisites

- Now Assist Skill Kit activated
- Role: `sn_aia.admin` or `admin`

---

## Lab Exercise — Steps to Build

### Step 1: Create the Skill

1. Navigate to **All** > search **Now Assist** > **Now Assist Skill Kit** > **Skills** > **New**
2. Configure:

| Field | Value |
|-------|-------|
| Name | `GenerateWebSearchQnsForResolutionPlan` |
| Description | `Generates a single, optimised, privacy-safe web search query from IT incident details for external resolution research` |
| Skill type | Custom |
| Target table | `incident` |

### Step 2: Write the Skill Prompt

In the **Prompt** field, enter:

```
You are a security-conscious IT search query specialist. Your task is to create a single, effective web search query from incident details that can be used to find resolution documentation on the internet.

INCIDENT DETAILS:
Error code: {{u_extracted_error_code}}
Short description: {{short_description}}

STRICT RULES — you MUST follow these:
1. DO NOT include any internal hostnames, server names, or IP addresses in the query
2. DO NOT include any user names, employee IDs, or personal identifiers
3. DO NOT include company-specific application names or internal product names
4. Replace all internal references with generic technical terms
5. The query must be suitable for a public search engine (Google, Bing)

QUERY GUIDELINES:
- Include the error code (it is a standard technical term, safe to search)
- Include the operating system or platform if identifiable (e.g., "Linux", "Windows Server")
- Include "troubleshooting", "fix", or "resolution steps" to get actionable results
- Keep the query concise — 5 to 10 words maximum
- Return ONLY the search query string, nothing else

EXAMPLES:
Input: Error code = HOST_UNREACHABLE, Description = "prod-app-01 server not responding"
Output: HOST_UNREACHABLE Linux server troubleshooting resolution steps

Input: Error code = ERR_NETWORK_TIMEOUT, Description = "database 10.0.0.45 timeout"
Output: ERR_NETWORK_TIMEOUT database connection timeout fix

Input: Error code = DISK_FULL, Description = "Application server running out of space"
Output: DISK_FULL Linux server disk space resolution

Return ONLY the search query. No explanation. No punctuation other than spaces.
```

### Step 3: Configure Output

Under **Output**, map:

| Variable | Source | Description |
|----------|--------|-------------|
| `web_search_query` | LLM response (full text) | The generated search query string |

### Step 4: Save and Activate

1. Click **Save**
2. Click **Activate**

### Step 5: Register as a Tool in the Resolution Pathfinder AI Agent

1. Navigate to **AI Agent Studio** > **Create and manage** > **AI agents**
2. Open **Resolution Pathfinder**
3. Under **Tools**, click **Add tool**
4. Select type: **Now Assist Skill**
5. Configure:

| Field | Value |
|-------|-------|
| Tool name | `GenerateWebSearchQuery` |
| Skill | `GenerateWebSearchQnsForResolutionPlan` |
| Description | `Generates a single optimised, privacy-safe web search query from the incident details` |

### Step 6: Test the Skill

1. Navigate to **Now Assist Skill Kit** > **Skills** > open `GenerateWebSearchQnsForResolutionPlan`
2. Click **Test**
3. Select a test Incident with `u_extracted_error_code = HOST_UNREACHABLE`
4. Run the skill
5. Verify the output:
   - Returns a single search query string
   - Contains no internal hostnames or IPs
   - Contains the error code and resolution-oriented terms

---

## Key Configuration Fields

| Field | Value for This Lab |
|-------|--------------------|
| Skill name | GenerateWebSearchQnsForResolutionPlan |
| Input | u_extracted_error_code, short_description |
| Output | web_search_query (single string) |
| Privacy rule | No internal hostnames, IPs, or user data in output |
| Tool type in AI Agent | Now Assist Skill |
| Used by agent | Resolution Pathfinder (Tool 3) |

---

## Reference

- [ServiceNow Zurich — Now Assist Skill Kit](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/now-assist-skill-kit/now-assist-skill-kit-landing.html)

---

## Next Step

Continue to [10 — Web Search Tool](10-web-search-tool.md) to configure the Web Search tool that executes the generated query against the internet.
