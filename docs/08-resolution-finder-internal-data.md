# 08 — Resolution Finder Using Internal Data (Now Assist Skill)

> **Source:** [ServiceNow Zurich Documentation — Now Assist Skill Kit](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/now-assist-skill-kit/now-assist-skill-kit-landing.html)

## What It Is

**ResolutionFinderUsingInternalData** is a **Now Assist Skill** — a custom generative AI skill built using the Now Assist Skill Kit that provides the Resolution Pathfinder AI Agent with the ability to search internal knowledge sources and return structured resolution recommendations.

The skill performs **Retrieval-Augmented Generation (RAG)**: it retrieves relevant content from the Knowledge Graph and previously resolved incidents, then synthesises that content using an LLM to produce actionable resolution steps — rather than just returning raw search results.

This is Tool 1 in the Resolution Pathfinder's search sequence. If this skill finds a viable resolution, the agent stops here (PATH A) and writes the result to the Incident. If not, it continues to the Elastic log search (PATH B).

---

## When to Use

- When an AI agent needs to reason over internal KB content (not just retrieve it)
- When resolved Incident history should inform new resolution recommendations
- When RAG is required to synthesise across multiple knowledge sources in a single response
- When you want AI-generated resolution steps with cited sources (not just matched articles)

---

## Customer Value

| Business Outcome | How This Skill Delivers It |
|-----------------|----------------------------|
| Accurate first-pass resolution | Combines vector search with LLM synthesis — more relevant than keyword search alone |
| Cited, audit-ready output | Skill outputs include source article references |
| Scales with knowledge base | More resolved incidents = better recommendations over time |
| Zero additional tools for common errors | Common errors resolved without ever reaching Elastic or internet |

---

## How It Works — Technical Overview

```
AI Agent calls ResolutionFinderInternalData skill
        │
        Input: incident description, u_extracted_error_code
        │
        ▼
Skill Step 1: Knowledge Graph search
        │  Vector similarity against IT Infrastructure Runbooks + CMDB
        │
        ▼
Skill Step 2: Similar resolved incidents search
        │  Query incident table: resolved incidents with same error code
        │
        ▼
Skill Step 3: LLM synthesis
        │  "Given this error code and these KB articles/past incidents,
        │   generate actionable resolution steps."
        │
        ▼
Output to AI Agent:
{
  "resolution_found": true,
  "steps": ["1. Check routing tables...", "2. Verify VPN..."],
  "source": "KB Article: Infrastructure Error Code Reference",
  "confidence": "high"
}
```

---

## How to Build

### Prerequisites

- Now Assist Skill Kit activated
- Knowledge Graph configured with IT Infrastructure KB (Capability 03)
- Role: `sn_aia.admin` or `admin`

---

## Lab Exercise — Steps to Build

### Step 1: Open Now Assist Skill Kit

1. Navigate to **All** > search **Now Assist**
2. Click **Now Assist Skill Kit** > **Skills** > **New**

### Step 2: Configure the Skill

| Field | Value |
|-------|-------|
| Name | `ResolutionFinderUsingInternalData` |
| Description | `Searches internal Knowledge Graph and resolved incidents using RAG to generate resolution steps for IT incidents` |
| Skill type | Custom |
| Target table | `incident` |

### Step 3: Configure Knowledge Sources

Under **Knowledge sources**, add:

| Source type | Value |
|------------|-------|
| Knowledge Graph schema | IT Infrastructure Runbooks |
| Table (resolved incidents) | `incident` — filter: `state=6 AND u_extracted_error_code IS NOT EMPTY` |

### Step 4: Write the Skill Prompt

In the **Prompt** field, enter the following prompt:

```
You are a resolution analyst for IT infrastructure incidents. Your task is to search the provided knowledge sources and return actionable resolution steps for the incident.

INCIDENT CONTEXT:
Error code: {{u_extracted_error_code}}
Short description: {{short_description}}
Affected CI: {{cmdb_ci}}

INSTRUCTIONS:
1. Search the knowledge base for articles related to the error code and description.
2. Search previously resolved incidents for the same error code pattern.
3. If you find a viable resolution:
   - List the resolution steps clearly and concisely (numbered list)
   - Cite the source (KB article title or resolved incident number)
   - Set resolution_found = true
4. If you cannot find a viable resolution after searching all available sources:
   - Set resolution_found = false
   - Explain briefly what was searched

RESPONSE FORMAT:
Return a JSON object:
{
  "resolution_found": true/false,
  "steps": ["step 1", "step 2", ...],
  "source": "KB Article title or INC number",
  "confidence": "high/medium/low",
  "reason_if_not_found": "explanation if resolution_found is false"
}

Focus only on actionable, technically accurate steps. Do not include speculative or generic advice.
Only return resolution_found = true if you have high confidence in the steps.
```

### Step 5: Configure Output Mapping

Under **Output**, map the skill response to variables the AI agent can consume:

| Variable | Source |
|----------|--------|
| `resolution_found` | JSON response field: `resolution_found` |
| `resolution_steps` | JSON response field: `steps` |
| `resolution_source` | JSON response field: `source` |
| `confidence` | JSON response field: `confidence` |

### Step 6: Save and Activate the Skill

1. Click **Save**
2. Click **Activate** — the skill is now available as a Now Assist Skill tool type in AI Agent Studio

### Step 7: Register as a Tool in the Resolution Pathfinder AI Agent

1. Navigate to **AI Agent Studio** > **Create and manage** > **AI agents**
2. Open **Resolution Pathfinder**
3. Under **Tools**, click **Add tool**
4. Select type: **Now Assist Skill**
5. Configure:

| Field | Value |
|-------|-------|
| Tool name | `ResolutionFinderInternalData` |
| Skill | `ResolutionFinderUsingInternalData` |
| Description | `Searches internal Knowledge Graph and resolved incidents using RAG to generate resolution steps` |
| Input | incident number, error code, description |

### Step 8: Test the Skill

1. Navigate to **Now Assist Skill Kit** > **Skills** > open `ResolutionFinderUsingInternalData`
2. Click **Test**
3. Select a test Incident with `u_extracted_error_code = HOST_UNREACHABLE`
4. Run the skill
5. Verify the output:
   - `resolution_found = true`
   - `steps` contains numbered troubleshooting steps
   - `source` cites an article from IT Infrastructure Runbooks

---

## Key Configuration Fields

| Field | Value for This Lab |
|-------|--------------------|
| Skill name | ResolutionFinderUsingInternalData |
| Knowledge sources | IT Infrastructure Runbooks, resolved incidents |
| Output | resolution_found, steps, source, confidence |
| Tool type in AI Agent | Now Assist Skill |
| Used by agent | Resolution Pathfinder |

---

## Reference

- [ServiceNow Zurich — Now Assist Skill Kit](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/now-assist-skill-kit/now-assist-skill-kit-landing.html)

---

## Next Step

Continue to [09 — Web Search Query Generator](09-web-search-query-generator.md) to build the Now Assist Skill that generates privacy-safe web search queries when internal sources don't have an answer.
