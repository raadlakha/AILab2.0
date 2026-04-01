# 02 — Now Assist in Document Intelligence (NADI)

> **Release:** Zurich | **Flow:** Requestor Flow — Phase 1 (Steps 5 & 6)
> **Source:** [ServiceNow Zurich — Now Assist in Document Intelligence](https://www.servicenow.com/docs/bundle/zurich-intelligent-experiences/page/administer/document-intelligence/concept/docintel-nowassist-landing.html) | [Use cases for Now Assist in Document Intelligence](https://www.servicenow.com/docs/r/intelligent-experiences/now-assist-in-document-intelligence/use-cases-now-assist-document-intelligence.html)

---

## What It Is

**Now Assist in Document Intelligence (NADI)** uses generative AI to extract structured data from images and documents and map it directly into ServiceNow table fields — with no manual data entry required.

In Zurich, the legacy Document Intelligence (DocIntel) application is **no longer activated on new instances** and is being prepared for future deprecation. **Now Assist in Document Intelligence is the current, supported product** for all new implementations.

In this lab, NADI is configured with a use case called **Veritas Extract**. When a user uploads an error screenshot or device label image (via the Conversation Topic upload in Step 4 of the Requestor Flow), NADI auto-triggers on the Incident attachment and extracts structured fields — specifically `u_extracted_error_code` — which arms the downstream Agentic Workflow trigger.

---

## Role in the Requestor Flow

```
[Steps 5 & 6 — Requestor Flow]

Step 5: Incident created (state = New)
        │  uploaded images attached to the record
        ▼
Step 6: Now Assist in Document Intelligence auto-triggers
        │  on each image attachment present on the Incident
        ▼
  Use Case: Veritas Extract
  Target table: incident (extended)
        │
        ▼
  GenAI reads image, extracts:
    • u_extracted_error_code  ← KEY FIELD — gates the Agentic Workflow
    • Additional fields (model, product name, serial number, barcode) as configured
        │
        ▼
  Document task status → Done
        │
        ▼
  Auto-generated Flow fires:
    "DocIntel Extract Values Flow — Veritas Extract"
    Writes extracted values to Incident record fields
        │
        ▼
Agentic Workflow can now evaluate trigger conditions:
  ✓ state = In Progress
  ✓ contact_type = chat
  ✓ u_extracted_error_code ≠ empty   ← populated by NADI
```

> **Why this matters:** `u_extracted_error_code` is a custom field on the extended Incident table. It is populated **exclusively** by NADI. If NADI is not configured or fails to extract, this field remains empty and the Resolution Pathfinder Agentic Workflow will not fire.

---

## What NADI Enables in This Lab

| Capability | How NADI Delivers It |
|-----------|---------------------|
| Auto-field population | Error code and device details extracted from uploaded image — no manual copy-paste |
| Agentic Workflow arming | `u_extracted_error_code` populated on the Incident, enabling the downstream trigger |
| Full automation mode | No agent review required — GenAI writes directly to record fields |
| Richer AI agent context | Extracted error code used by Resolution Pathfinder to search KB, logs, and web |
| Higher data quality | AI reads directly from source image — eliminates transcription errors |

---

## Prerequisites

| Requirement | Detail |
|-------------|--------|
| Licence | Now Assist Pro+ or ITSM Pro+ with Now Assist in Document Intelligence |
| Plugin — Now Assist in Document Intelligence | `sn_now_assist_doc_intel` — must be Active |
| Custom field | `u_extracted_error_code` — must exist on the extended Incident table |
| Extended Incident table | `x_nava_agentic_lab_incident_extend` (or your scoped app equivalent) |
| Role | `sn_doc_intel.admin` or `admin` |
| Zurich patch | Base Zurich; Zurich Patch 4 recommended for NASK tool integration |

---

## Lab Exercise — Steps to Configure NADI

### Step 1: Open Now Assist in Document Intelligence

Navigate to **All** → search **Now Assist** → **Now Assist Features**

In the Now Assist Features screen, locate the **Extract Information from documents** skill.

![NADI — Now Assist Features screen](../screenshots/NADI-1.png)

> This is the entry point for all NADI configuration. The skill is OOB — you do not create it. You create **use cases** within it.

---

### Step 2: Create the Use Case

1. Click **Edit** on the **Extract Information from documents** skill
2. Click **New use case**

![NADI — Create Use Case](../screenshots/NADI-2.png)

Fill in the use case details:

| Field | Value |
|-------|-------|
| Use case name | `Veritas Extract` |
| Target table | Extended Incident table (`x_nava_agentic_lab_incident_extend`) |
| LLM | `Azure OpenAI - GPT Large` (or your configured provider) |

3. Click **Next**

![NADI — Use Case Details](../screenshots/NADI-3.png)

---

### Step 3: Add Extraction Fields

On the **Fields** step, click **Add a field** → select **Field** (not question or table).

![NADI — Add Fields](../screenshots/NADI-4.png)

Configure the following fields:

#### Field 1 — Error Code

![NADI — Error Code Field](../screenshots/NADI-5-error-code-field.png)

| Setting | Value |
|---------|-------|
| Field name | `Error Code` |
| Details | `This is the error code mentioned in the image. Example format: "0xE00052" — extract only the numeric suffix (e.g. 52).` |
| Field type | `Text` |
| Target table | Extended Incident table |
| Target field | `u_extracted_error_code` |
| Required for extraction | ✅ Yes |

> **This is the critical field.** `u_extracted_error_code` is the gate for the downstream Agentic Workflow. It must be mapped to the correct target field on the extended Incident table.

#### Field 2 — Model Details

![NADI — Model Details Field](../screenshots/NADI-5-model-details-field.png)

| Setting | Value |
|---------|-------|
| Field name | `Model Details` |
| Details | `Device model information from the label or image.` |
| Field type | `Text` |
| Target field | `u_model_details` |
| Required for extraction | Optional |

#### Field 3 — Product Name

![NADI — Product Name Field](../screenshots/NADI-5-product-name-field.png)

| Setting | Value |
|---------|-------|
| Field name | `product name` |
| Details | `The product name as shown on the device label, typically found below "Product."` |
| Field type | `Text` |
| Target field | `u_product_name` |
| Required for extraction | Optional |

#### Field 4 — Serial Number

![NADI — Serial Number Field](../screenshots/NADI-5-serial-number-field.png)

| Setting | Value |
|---------|-------|
| Field name | `Serial Number` |
| Details | `Device serial number from the label.` |
| Field type | `Text` |
| Target field | `u_serial_number` |
| Required for extraction | Optional |

#### Field 5 — PN / Bar Code

![NADI — PN Bar Code Field](../screenshots/NADI-5-pn-bar-code-field.png)

| Setting | Value |
|---------|-------|
| Field name | `PN / Bar Code` |
| Details | `Part number or barcode from the device label.` |
| Field type | `Text` |
| Target field | `u_pn_bar_code` |
| Required for extraction | Optional |

---

### Step 4: Enable Full Automation Mode

1. Click the **Settings** (gear) icon on the use case
2. Navigate to **Extraction mode**
3. Toggle **Full automation mode (no agent review required)** → **On**

> Full automation mode means the GenAI extracts fields and writes them directly to the Incident record without waiting for an agent to review or approve. This is required for the Requestor Flow — the Incident must be enriched immediately after image upload so the Agentic Workflow trigger can fire.

---

### Step 5: Test the Extraction

1. On the use case, click **Test**

![NADI — Test screen](../screenshots/NADI-6-test.png)

2. In the test dialog, choose:
   - **Upload from record** — select an existing Incident with an image attached, or
   - **Upload from this device** — upload a Veritas device label image directly
3. Click **Continue**

![NADI — Test Result 1](../screenshots/NADI-7-test1.png)

The Document Q&A panel displays the extracted values. Verify:
- `u_extracted_error_code` → numeric error value extracted from the image
- `u_product_name` → product name as printed on the label
- `u_serial_number` → serial number if present

![NADI — Test Result 2](../screenshots/NADI-7-test2.png)

> If extraction does not complete and the status stays **In Progress**, the document may still be processing. Click **Refresh**. If it stays stuck, check the execution logs.

---

### Step 6: Add the Integration

The integration tells NADI what to do once extraction is complete — specifically, it creates the Flow that writes extracted values back to the Incident record.

1. In the use case, navigate to the **Integrations** step
2. Click **Add Integration**

![NADI — Integration config 1](../screenshots/NADI-8-integration.png)

Fill in:

| Field | Value |
|-------|-------|
| Integration Name | `Veritas_Process` |
| Target Table | Extended Incident table |
| Integration type | `Process task` |
| Create Flow | ✅ Checked |

3. Click **Save**

![NADI — Integration config 2](../screenshots/NADI-8-integration2.png)

This auto-generates a Flow in Workflow Studio:

```
Flow name: DocIntel Extract Values Flow — Veritas Extract

Trigger: Document Task Updated
  WHERE status changes to Done
  AND source table = extended Incident table
  AND use case = Veritas Extract
```

---

### Step 7: Activate the Integration

1. In the **Integrations** panel, locate **Veritas_Process**

![NADI — Activate integration step 1](../screenshots/NADI-8-activate3.png)

2. Click **Open in Flow Designer** (or navigate to the auto-generated flow in Workflow Studio)
3. The flow opens with status **Inactive**
4. Click **Activate** → status updates to **Active**

---

### Step 8: Configure the Incident Integration Trigger

This step wires NADI to auto-trigger when images are attached to an Incident created via NAVA chat.

1. Navigate to the **Integrations** tab of the use case
2. Add a second integration — or configure the trigger on the existing flow:

![NADI — Integration trigger config 1](../screenshots/NADI-9-integration.png)

| Field | Value |
|-------|-------|
| Trigger type | Record-based |
| Table | Extended Incident table |
| Trigger condition | Attachment added AND `contact_type = chat` |

![NADI — Integration trigger config 2](../screenshots/NADI-9-integration2.png)

3. Verify the trigger activates correctly

![NADI — Activate trigger step 1](../screenshots/NADI-9-integration-activate3.png)

![NADI — Activate trigger step 2](../screenshots/NADI-9-integration-activate4.png)

> **Note:** The integration trigger ensures NADI runs automatically whenever an image is attached to a chat-originated Incident — no manual intervention required by the L1 Agent or user.

---

### Step 9: Verify Full Automation End-to-End

1. Create a test Incident via the NAVA chat (or manually with `contact_type = chat`)
2. Attach a Veritas device label image to the Incident
3. Verify the Document task is created and reaches **Done** status automatically
4. Check the Incident record for populated fields:
   - `u_extracted_error_code` → should contain the extracted error code
   - `u_product_name`, `u_serial_number` → populated if present in the image

![NADI — Full automation verification](../screenshots/NADI-10-usecase-full-auto.png)

![NADI — Full automation result](../screenshots/NADI-10-usecase-full-auto2.png)

> Once `u_extracted_error_code` is non-empty on an Incident with `state = In Progress` and `contact_type = chat`, the Resolution Pathfinder Agentic Workflow fires automatically.

---

## Key Configuration Fields

| Field | Value for This Lab |
|-------|--------------------|
| Skill | Extract Information from documents |
| Use case name | Veritas Extract |
| Target table | Extended Incident table |
| Critical output field | `u_extracted_error_code` |
| Extraction mode | Full automation (no agent review) |
| Integration name | Veritas_Process |
| Integration type | Process task |
| Auto-generated flow | DocIntel Extract Values Flow — Veritas Extract |
| Trigger condition | Document Task status = Done |
| Role required | `sn_doc_intel.admin` or `admin` |

---

## Technical Notes

### Why `u_extracted_error_code` is the Critical Field

This field is the **third and final condition** for the Resolution Pathfinder Agentic Workflow trigger:

```
Agentic Workflow trigger conditions:
  ✓ state = In Progress (2)       ← set when L1 Agent creates the Incident
  ✓ contact_type = chat           ← stamped by NAVA (Capability 01)
  ✓ u_extracted_error_code ≠ empty ← populated by NADI (this capability)
```

Without NADI running successfully, the workflow will never fire — regardless of how correctly the L1 Agent and NAVA are configured.

### Full Automation Mode vs. Agent Review Mode

| Mode | Behaviour | Use When |
|------|-----------|----------|
| **Full automation** | GenAI extracts and writes fields immediately, no review | Trusted document types with consistent layouts (device labels) |
| **Agent review** | GenAI extracts but a human reviews before values are written | Complex or variable documents where accuracy must be validated |

For this lab, **Full automation** is required — the Incident enrichment must happen immediately after image upload to enable the Agentic Workflow within the same session.

### Supported Document Types

Now Assist in Document Intelligence supports: PDF, PNG, JPEG, and other common image formats. For this lab, the primary input is a PNG/JPEG screenshot or device label photo uploaded by the user via the NAVA chat interface.

---

## Reference

- [ServiceNow Zurich — Now Assist in Document Intelligence](https://www.servicenow.com/docs/bundle/zurich-intelligent-experiences/page/administer/document-intelligence/concept/docintel-nowassist-landing.html)
- [Use cases for Now Assist in Document Intelligence](https://www.servicenow.com/docs/r/intelligent-experiences/now-assist-in-document-intelligence/use-cases-now-assist-document-intelligence.html)
- [Exploring Now Assist in Document Intelligence](https://www.servicenow.com/docs/bundle/zurich-intelligent-experiences/page/administer/document-intelligence/concept/docintel-exploring-now-assist.html)

---

## Next Step

Continue to [05 — L1 Requestor Agent](05-requestor-ai-agent.md) to build the Chat AI Agent that orchestrates the full Requestor Flow — from Knowledge Graph lookup through Troubleshooting Guide delivery, image upload prompting, and Incident creation.
