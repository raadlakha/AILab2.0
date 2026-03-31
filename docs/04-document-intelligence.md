# 04 — Now Assist in Document Intelligence

> **Source:** [ServiceNow Zurich Documentation — Now Assist in Document Intelligence](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/now-assist-in-document-intelligence/docintel-nowassist-landing.html)

## What It Is

**Now Assist in Document Intelligence** uses generative AI to extract structured data from documents and map it directly into ServiceNow table fields. As described in the official documentation:

> *"With ServiceNow® Now Assist in Document Intelligence, you can use generative AI to get key information from digital documents into your automation workflows."*

In this lab, the **"Extract Information from documents"** Now Assist skill is configured with a use case called **Veritas Extract**. When a user uploads a VERITAS device label image, the skill extracts two fields — **product name** and **error code** — and writes them into the **incident extend** table. An auto-generated Flow then processes the extracted data downstream.

> **Important (Zurich):** The legacy **Document Intelligence** (DocIntel) application is being prepared for future deprecation in the Zurich release. It will no longer be activated on new instances. **Now Assist in Document Intelligence** is the current, supported product and should be used for all new implementations.

---

## When to Use

- When users submit supporting documents (device labels, error screenshots, log exports) that contain data you need in structured form
- When you want to auto-populate table fields from images or attachments using AI extraction
- When replacing manual data entry with GenAI-powered field population
- When a downstream Flow or workflow depends on extracted values being present on a record

---

## Customer Value

| Business Outcome | How Now Assist in Document Intelligence Delivers It |
| --- | --- |
| Faster incident creation | Product name and error code extracted automatically — no manual copy-paste |
| Higher data quality | GenAI extracts exact values from source documents, reducing human transcription errors |
| Richer AI agent context | Extracted fields feed directly into downstream automation and AI agent workflows |
| Seamless user experience | User uploads a device label image; the system extracts and writes the data |
| End-to-end automation | Full automation mode bypasses agent review and triggers the next step immediately |

---

## How It Works — In This Lab

```text
User uploads VERITAS device label image (via NAVA upload image topic)
        │
        ▼
"Extract Information from documents" Now Assist skill
  Use Case: Veritas Extract
  Target table: incident extend
        │
        ▼
GenAI reads image, extracts two fields:
  • product name  → writes to `product` on incident extend
  • error_code    → writes to `error_code` on incident extend
        │
        ▼
Document task status changes to Done
        │
        ▼
Auto-generated Flow fires:
  "DocIntel Extract Values Flow - Veritas Extract"
  Trigger: Document Task Updated (Status = Done, Use Case = Veritas Extract)
        │
        ▼
Extracted data available on incident extend record for downstream processing
```

---

## Lab Exercise — Steps to Build

### Step 1: Open the Extract Information from Documents Skill

1. Navigate to **All** > **Now Assist** > **Skills**
2. In the **Now Assist skills for Platform** screen, filter by **Document**
3. Locate the **Extract Information from documents** skill
4. Click **Edit**

### Step 2: Create the Use Case

1. Inside the skill, click **New use case**
2. Fill in:

| Field | Value |
| --- | --- |
| Use case name | `Veritas Extract` |
| Target table | `incident extend` (`x_snc_apacaienable_incident_extend`) |
| LLM | `Azure OpenAI - GPT Large` |

1. Click **Next**

### Step 3: Add Extraction Fields

On the **Fields** step, click **Add a field** → select **Field** (not question or table).

#### Field 1 — Product Name

| Setting | Value |
| --- | --- |
| Field name | `product name` |
| Details | `This is the product name which is mentioned on the device details image, generally mentioned below "Product."` |
| Field Type | `Text` |
| Target table | `incident extend` |
| Target field | `product` |
| Required for extraction | ✅ Yes |

#### Field 2 — Error Code

| Setting | Value |
| --- | --- |
| Field name | `Error Code` |
| Details | `This is the error code mentioned in the image, example of error code text is "0xE00052", however, we only want to extract 52 from this.` |
| Field Type | `Text` |
| Target table | `incident extend` |
| Target field | `error_code` |
| Required for extraction | ✅ Yes |

After adding both fields, the extraction template shows:

| Target field | Type | Required |
| --- | --- | --- |
| product | Text | Yes |
| error_code | Text | Yes |

### Step 4: Enable Full Automation Mode

1. Click **Settings** (gear icon) on the use case
2. Select **Extraction mode**
3. Toggle **Full automation mode (no agent review required)** → **On**

> This bypasses the agent review step. GenAI auto-fills all fields and submits the document task without waiting for a human reviewer. If any required field is missing, you can optionally toggle the fallback setting to require agent review.

### Step 5: Test the Extraction

1. On the use case, click **Test**
2. In the test dialog, choose:
   - **Upload from record** — select an existing incident (e.g., `INCE0011003`), or
   - **Upload from this device** — upload a VERITAS device label image directly
3. Click **Continue**
4. The Document Q&A panel displays the extracted values — verify:
   - **product name** → e.g., `NETBACKUP APPLIANCE 5240 4TB 4 1GB ETHERNET - 2 10GBT CU ETHERNET STANDARD APPLIANCE`
   - **Error Code** → numeric value extracted from the label

### Step 6: Add the Integration

1. In the use case, navigate to the **Integrations** step
2. Click **Add Integration**
3. Fill in:

| Field | Value |
| --- | --- |
| Integration Name | `Veritas_Process` |
| Target Table | `incident extend` |
| Integration type | `Process task` |
| Create Flow | ✅ Checked |

1. Click **Save**

This auto-generates a Flow in Workflow Studio:

Flow name: `DocIntel Extract Values Flow - Veritas Extract - Veritas Extract`

Trigger — Document Task Updated where:

- Status changes to **Done**
- Source Table is `x_snc_apacaienable_incident_extend`
- Use Case is **Veritas Extract**

### Step 7: Activate the Flow

1. Click **Open in Flow Designer** from the integration panel, or navigate to the flow in Workflow Studio
2. The flow opens with status **Inactive**
3. Click **Activate**
4. Confirm — status updates to **Active**

---

## Reference

- [ServiceNow Zurich — Now Assist in Document Intelligence](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/now-assist-in-document-intelligence/docintel-nowassist-landing.html)
- [Explore Now Assist in Document Intelligence](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/now-assist-in-document-intelligence/docintel-exploring-now-assist.html)

---

## Next Step

Continue to [05 — First Responder AI Agent](05-first-responder-ai-agent.md) to build the AI agent that handles the user's initial report and creates the Incident.
