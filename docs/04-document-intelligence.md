# 04 — Now Assist in Document Intelligence

> **Source:** [ServiceNow Zurich Documentation — Now Assist in Document Intelligence](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/now-assist-in-document-intelligence/docintel-nowassist-landing.html)

## What It Is

**Now Assist in Document Intelligence** is the Zurich-era product for extracting structured data from documents using generative AI. As described in the official documentation:

> *"With ServiceNow® Now Assist in Document Intelligence, you can use generative AI to get key information from digital documents into your automation workflows."*

In this lab, the user can upload a screenshot or log file when reporting the server issue. Now Assist in Document Intelligence extracts the **error code** from that document and populates the `u_extracted_error_code` field on the Incident record. This field is the trigger condition for the Resolution Pathfinder AI Agent — making document intelligence a critical part of the end-to-end pipeline.

> **Important (Zurich):** The legacy **Document Intelligence** (DocIntel) application is being prepared for future deprecation in the Zurich release. It will no longer be activated on new instances. **Now Assist in Document Intelligence** is the current, supported product and should be used for all new implementations.

---

## When to Use

- When users submit supporting documents (error screenshots, log exports) that contain data you need in structured form
- When you want to auto-populate Incident fields from email bodies, attachments, or images
- When replacing manual data entry with AI-powered extraction
- When error codes, server names, or timestamps need to be captured without asking the user to type them

---

## Customer Value

| Business Outcome | How Now Assist in Document Intelligence Delivers It |
|-----------------|------------------------------------------------------|
| Faster incident creation | Error codes, system names, timestamps extracted automatically — no manual copy-paste |
| Higher data quality | GenAI extracts exact values from source documents, reducing human transcription errors |
| Richer AI agent context | Extracted error code feeds directly into AI agent's resolution search |
| Seamless user experience | User uploads a screenshot; the system does the rest |
| End-to-end automation | Extracted error code triggers the Resolution Pathfinder AI Agent automatically |

---

## How It Works — Technical Overview

```
User uploads screenshot / log file in NAVA chat
        │
        ▼
Now Assist in Document Intelligence triggered
        │
        ▼
Generative AI reads document, extracts key fields
        │
        ▼
Extracted fields mapped to Incident record fields:
  • u_extracted_error_code  ← key trigger field
  • cmdb_ci (if server name found)
  • short_description (enriched from document)
        │
        ▼
Resolution Pathfinder Agentic Workflow trigger condition now met
```

---

## How to Build

### Prerequisites

- Now Assist in Document Intelligence activated on the instance
- Role: `admin` or `sn_doc_intel.admin`
- An active Incident table with custom field `u_extracted_error_code` (String)

### Step 0: Create the Custom Field (if not already present)

1. Navigate to **System Definition** > **Tables**
2. Search for `Incident` → click to open
3. Click **New** under **Columns**
4. Add:

| Field | Value |
|-------|-------|
| Column label | `Extracted Error Code` |
| Column name | `u_extracted_error_code` |
| Type | `String` |
| Max length | `100` |

5. **Save**

---

## Lab Exercise — Steps to Build

### Step 1: Activate Now Assist in Document Intelligence

1. Navigate to **All** > search **Now Assist** > **Administration** > **Setup**
2. Find **Now Assist in Document Intelligence** in the product list
3. Click **Activate** if not already active
4. Confirm the activation in the **Now Assist Skill** configuration

### Step 2: Configure the Extraction Skill

Now Assist in Document Intelligence uses generative AI skills. Configure a skill for IT Incident error extraction:

1. Navigate to **All** > search **Now Assist** > **Skills**
2. Find or create the extraction skill for Incident documents
3. Configure the extraction fields:

| Field Label | Target Field | Extraction Prompt |
|-------------|-------------|-------------------|
| Error Code | `u_extracted_error_code` | `Extract the error code, status code, or error identifier from the document. Format: alphanumeric string (e.g., HOST_UNREACHABLE, ERR_NETWORK_TIMEOUT).` |
| Server / Host Name | `cmdb_ci` | `Extract the server name, hostname, or IP address mentioned in the document.` |
| Error Description | `description` | `Extract a brief description of the error or issue shown in the document.` |

4. Set the **Target table** to `Incident [incident]`
5. **Save and Activate** the skill

### Step 3: Configure the File Upload in NAVA

To allow users to upload files in the NAVA chat:

1. Navigate to **All** > **AI Agent Studio** > **Create and manage** > **AI agents**
2. Open the **First Responder Operations Analyst** AI agent *(built in Capability 05)*
3. Under **Tools**, confirm the **File Upload** tool is configured:

| Field | Value |
|-------|-------|
| Tool name | `UploadSupportingDocument` |
| Type | File Upload |
| Description | `Allows the user to upload screenshots or log files for error code extraction` |
| Accepted types | `image/*, application/pdf` |

### Step 4: Trigger Document Intelligence from the AI Agent

The AI agent instructions (system prompt) should instruct the agent to:
1. Ask the user if they have a screenshot or log file
2. Accept the file upload via the `UploadSupportingDocument` tool
3. The document intelligence skill processes the attachment automatically
4. The extracted `u_extracted_error_code` is written to the Incident record

Add to the agent's system prompt (in Capability 05):
```
After collecting the initial description of the issue, ask:
"Do you have a screenshot or error log you can upload? This helps me identify the exact error code."
If the user uploads a file, use the UploadSupportingDocument tool to accept it.
The error code will be extracted automatically.
```

### Step 5: Test the Extraction

1. Take a screenshot of any error message, or create a sample image with text:
   `ERROR: HOST_UNREACHABLE — server01.prod — 2024-03-15 14:32:07`
2. Open the NAVA chat in Service Portal
3. Report the server issue and upload the screenshot when prompted
4. Navigate to the created Incident record
5. Verify `u_extracted_error_code` is populated with `HOST_UNREACHABLE`

---

## Why `u_extracted_error_code` Matters

This field is the **trigger condition** for the Resolution Pathfinder Agentic Workflow (Capability 06):

```
Trigger: Incident state = "In Progress"
         AND channel = "Chat"
         AND u_extracted_error_code IS NOT EMPTY
```

Without this field populated, the Resolution Pathfinder will not activate. Document Intelligence is what makes the automation fully end-to-end.

---

## Key Configuration Fields

| Field | Value for This Lab |
|-------|--------------------|
| Product | Now Assist in Document Intelligence |
| Target field | `u_extracted_error_code` |
| Trigger | Attachment processed on Incident |
| AI model | Now Assist LLM (managed by ServiceNow) |

---

## Reference

- [ServiceNow Zurich — Now Assist in Document Intelligence](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/now-assist-in-document-intelligence/docintel-nowassist-landing.html)
- [Explore Now Assist in Document Intelligence](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/now-assist-in-document-intelligence/docintel-exploring-now-assist.html)

---

## Next Step

Continue to [05 — First Responder AI Agent](05-first-responder-ai-agent.md) to build the AI agent that handles the user's initial report and creates the Incident.
