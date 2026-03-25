# 07 — Flow Action Tool — Extract Incident Details

> **Source:** [ServiceNow Zurich Documentation — Add tools and information to an AI agent](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/add-tool-aia.html)

## What It Is

A **Flow Action** is one of the tool types available to AI agents in AI Agent Studio. It allows an AI agent to invoke a ServiceNow Flow Designer action — a structured, parameterised operation — and receive back structured data.

In this lab, the **Extract Incident Details** Flow Action is the tool that transforms a user's free-text description into structured fields. When the First Responder AI Agent calls this tool, it receives back:
- The name of the affected server/CI
- The extracted error code (if identifiable from conversation)
- The incident category and subcategory
- The urgency level

This structured output is then used to populate the Incident record accurately, ensuring downstream processes (including the Resolution Pathfinder) receive complete data.

---

## When to Use

- When an AI agent needs to invoke a defined, governed business process as a step in its reasoning
- When structured output is required from unstructured conversational input
- When you want to centralise data extraction logic in Flow Designer rather than in the LLM prompt
- When the same extraction logic is reused across multiple agents or workflows

---

## Customer Value

| Business Outcome | How the Flow Action Delivers It |
|-----------------|----------------------------------|
| Consistent data extraction | Every incident goes through the same structured extraction logic — no variation |
| Governed processing | Flow Designer provides audit trail, version control, and admin oversight |
| Decoupled AI from business logic | AI agent focuses on conversation; Flow Action handles data transformation |
| Reusable across agents | Same flow action works for both the First Responder and other future agents |

---

## How It Works — Architecture

```
First Responder AI Agent
        │
        │ "The server prod-app-01 is unreachable with error HOST_UNREACHABLE"
        ▼
Tool call: ExtractIncidentDetails (Flow Action)
        │
        ▼
Flow Designer — Extract Incident Details Action
        │
        ├── Input: conversation summary, user description
        │
        ├── Step 1: NLU/LLM extraction of server name → cmdb_ci lookup
        ├── Step 2: Error code identification from text → u_extracted_error_code
        ├── Step 3: Category classification → category / subcategory
        └── Step 4: Urgency derivation from context
        │
        ▼
Output returned to AI Agent:
{
  "cmdb_ci": "prod-app-01",
  "u_extracted_error_code": "HOST_UNREACHABLE",
  "category": "Infrastructure",
  "subcategory": "Server",
  "urgency": "2"
}
```

---

## How to Build

### Prerequisites

- Flow Designer active on the instance
- Role: `flow_designer` or `admin`
- CMDB populated with CI records for testing

---

## Lab Exercise — Steps to Build

### Step 1: Create the Flow Action in Flow Designer

1. Navigate to **All** > search **Flow Designer**
2. Click **New** > **Action**
3. Fill in:

| Field | Value |
|-------|-------|
| Action name | `Extract Incident Details` |
| Description | `Extracts structured incident information from a user's conversational description. Returns CI name, error code, category, and urgency.` |
| Category | `IT Service Management` |

### Step 2: Define the Input Variables

Click **Inputs** and add the following:

| Name | Type | Description |
|------|------|-------------|
| `conversation_summary` | String | The summary of the user's reported issue |
| `user_description` | String | The full user-provided description |

### Step 3: Add Action Steps

#### Step 3a: Extract CI Reference

1. Click **Add Step** > **Utilities** > **Look Up Record**
2. Configure:
   - Table: `cmdb_ci`
   - Condition: Name contains the server name identified from `conversation_summary`
3. Output: CI sys_id and display name

#### Step 3b: Script Step — Extract Fields Using LLM

1. Click **Add Step** > **Utilities** > **Script**
2. Add a script that parses the conversation summary for:
   - Error code patterns (e.g., `HOST_UNREACHABLE`, `ERR_NETWORK_TIMEOUT`)
   - Urgency keywords (e.g., "production", "critical" → urgency 1; "non-prod" → urgency 3)
   - Category classification based on keywords

```javascript
(function execute(inputs, outputs) {
    var description = inputs.conversation_summary;

    // Extract error code
    var errorCodePattern = /\b(HOST_UNREACHABLE|ERR_NETWORK_TIMEOUT|CONNECTION_REFUSED|DISK_FULL)\b/i;
    var errorMatch = description.match(errorCodePattern);
    outputs.error_code = errorMatch ? errorMatch[0].toUpperCase() : '';

    // Derive urgency
    outputs.urgency = '3'; // Default: Low
    if (/production|prod|p1|critical/i.test(description)) {
        outputs.urgency = '1';
    } else if (/staging|uat|p2|high/i.test(description)) {
        outputs.urgency = '2';
    }

    // Set category
    outputs.category = 'Infrastructure';
    outputs.subcategory = 'Server';

})(inputs, outputs);
```

### Step 4: Define the Output Variables

Click **Outputs** and add:

| Name | Type | Description |
|------|------|-------------|
| `cmdb_ci` | Reference | The identified CI record |
| `u_extracted_error_code` | String | Extracted error code |
| `category` | String | Incident category |
| `subcategory` | String | Incident subcategory |
| `urgency` | String | Urgency level (1=Critical, 2=High, 3=Medium) |

### Step 5: Save and Publish

1. Click **Save**
2. Click **Publish** — this makes the action available as a tool in AI Agent Studio

### Step 6: Register as a Tool in the First Responder AI Agent

1. Navigate to **AI Agent Studio** > **Create and manage** > **AI agents**
2. Open **First Responder Operations Analyst**
3. Under **Tools**, click **Add tool**
4. Select type: **Flow Action**
5. Configure:

| Field | Value |
|-------|-------|
| Tool name | `ExtractIncidentDetails` |
| Flow Action | `Extract Incident Details` *(published in Step 5)* |
| Description | `Extracts structured incident information: affected CI, error code, category, urgency` |

### Step 7: Test the Flow Action

1. In Flow Designer, open `Extract Incident Details`
2. Click **Test**
3. Enter test inputs:
   - `conversation_summary`: `"The production server prod-app-01 is showing HOST_UNREACHABLE error"`
4. Verify outputs:
   - `cmdb_ci` resolves to `prod-app-01`
   - `u_extracted_error_code` = `HOST_UNREACHABLE`
   - `urgency` = `1` (production keyword detected)

---

## Key Configuration Fields

| Field | Value for This Lab |
|-------|--------------------|
| Action name | Extract Incident Details |
| Input | conversation_summary, user_description |
| Output | cmdb_ci, u_extracted_error_code, category, urgency |
| Tool type in AI Agent Studio | Flow Action |
| Used by agents | First Responder Operations Analyst |

---

## Reference

- [ServiceNow Zurich — Add tools to an AI agent](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/add-tool-aia.html)

---

## Next Step

Continue to [08 — Resolution Finder Internal Data](08-resolution-finder-internal-data.md) to build the Now Assist Skill that searches internal knowledge for resolution steps.
