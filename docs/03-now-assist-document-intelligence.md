# 03 — Now Assist Document Intelligence

> **Release:** Zurich | **Flow:** Requestor Flow — Phase 1 (Steps 4)

***

## What It Is

**Now Assist in Document Intelligence (NADI)** uses generative AI to extract structured data from images and documents and map it directly into ServiceNow table fields — with no manual data entry required.

In Zurich, the legacy Document Intelligence (DocIntel) application is **no longer activated on new instances** and is being prepared for future deprecation. **Now Assist in Document Intelligence is the current, supported product** for all new implementations.

In this lab, NADI is configured with a use case called **Veritas Extract**. When a user uploads an error screenshot or device label image (via the Conversation Topic upload in Step 4 of the Requestor Flow), NADI auto-triggers on the Incident attachment and extracts structured fields — specifically `error_code` — which arms the downstream Agentic Workflow trigger.

***

## Role in the Requestor Flow

![NADI Flow Overview](../screenshots/flow-NADI.png)

***

## What NADI Enables in This Lab

| Capability              | How NADI Delivers It                                                               |
| ----------------------- | ---------------------------------------------------------------------------------- |
| Auto-field population   | Error code and device details extracted from uploaded image — no manual copy-paste |
| Agentic Workflow arming | `error_code` populated on the Incident, enabling downstream resolution             |
| Full automation mode    | No agent review required — GenAI writes directly to record fields                  |
| Richer AI agent context | Extracted error code used by Resolution Pathfinder to search KB, logs, and web     |
| Higher data quality     | AI reads directly from source image — eliminates transcription errors              |

***

## Lab Exercise — Steps to Configure NADI

### Step 1: Open Now Assist in Document Intelligence

Navigate to **All** → search **Now Assist Admin** → **Skills** → **Platform** → **Search for 'Document'** → **Edit 'Extract information from documents'**

![NADI — Now Assist Features screen](<../.gitbook/assets/NADI-1 (1).png>)

> This is the entry point for all NADI configuration. The skill is OOB — you do not create it. You create **use cases** within it.

***

### Step 2: Create the Use Case

1. Click **Edit** on the **Extract Information from documents** skill
2. Click **New use case**

![NADI — Create Use Case](<../.gitbook/assets/NADI-2 (1).png>)

| Field             | Value                                                        |
| ----------------- | ------------------------------------------------------------ |
| **Use case name** | `Veritas Extract`                                            |
| **Target table**  | `Incident Extend table (x_snc_apacaienable_incident_extend)` |
| **LLM**           | `Azure OpenAI - GPT Large` (or your configured provider)     |
| **Image Mode**    | Yes                                                          |

3. Click **Next**

![NADI — Use Case Details](<../.gitbook/assets/NADI-3 (1).png>)

***

### Step 3: Add Extraction Fields

On the **Fields** step, click **Add a field** — select **Field** (not question or table).

![NADI — Add Fields](<../.gitbook/assets/NADI-4 (1).png>)

#### Field 1 — Error Code

![NADI — Error Code Field](<../.gitbook/assets/NADI-5-error-code-field (1).png>)

| Setting                     | Value        |
| --------------------------- | ------------ |
| **Field name**              | `Error Code` |
| **Field type**              | `Text`       |
| **Target field**            | `error_code` |
| **Required for extraction** | Yes          |

**Details**

```
This is the error code mentioned in the image, example of error code text is "0xE00052", however, we only want to extract 52 from this.
```

> **This is the critical field.** `error_code` gates the downstream Agentic Workflow as it is the most reliable indicator for resolution troubleshooting.

#### Field 2 — Model Details

![NADI — Model Details Field](<../.gitbook/assets/NADI-5-model-details-field (1).png>)

| Setting                     | Value           |
| --------------------------- | --------------- |
| **Field name**              | `Model Details` |
| **Field type**              | `Text`          |
| **Target field**            | `model details` |
| **Required for extraction** | Optional        |

**Details**

```
This is Reg. Model Version, Alpha numeric field on the image
```

#### Field 3 — Product Name

![NADI — Product Name Field](<../.gitbook/assets/NADI-5-product-name-field (1).png>)

| Setting                     | Value          |
| --------------------------- | -------------- |
| **Field name**              | `product name` |
| **Field type**              | `Text`         |
| **Target field**            | `product name` |
| **Required for extraction** | Optional       |

**Details**

```
This is the product name which is mentioned on the device details image, generally mentioned below "Product:"
```

#### Field 4 — Serial Number

![NADI — Serial Number Field](<../.gitbook/assets/NADI-5-serial-number-field (1).png>)

| Setting                     | Value           |
| --------------------------- | --------------- |
| **Field name**              | `Serial Number` |
| **Field type**              | `Text`          |
| **Target field**            | `serial number` |
| **Required for extraction** | Optional        |

**Details**

```
This is the SN number mentioned at the end of the image on the left side, mentioned after the text "SN:"
```

#### Field 5 — PN Bar Code

![NADI — PN Bar Code Field](<../.gitbook/assets/NADI-5-pn-bar-code-field (1).png>)

| Setting                     | Value              |
| --------------------------- | ------------------ |
| **Field name**              | `PN Bar Code`      |
| **Field type**              | `Text`             |
| **Target field**            | `product bar code` |
| **Required for extraction** | Optional           |

**Details**

```
This is the pn number above the bar code, which is alpha numeric also it is hyphenated
```

***

### Step 4: Test the Extraction

This step validates that NADI can correctly read your lab images and extract the configured fields before activating for production use.

#### Step 4a — Upload the Test Images

In the **Test output** step, an upload dialog appears.

![NADI — Test upload dialog](../.gitbook/assets/NADI-testing1.png)

Select **Upload from this device** and click **+ Add file**.

> **Add both lab images:**
>
> * The **Error 37 screenshot** (error screen from the Veritas appliance)
> * The **Veritas server details label** (device label with model, serial, product details)
>
> Both images are required to test all 5 extraction fields. If you missed downloading the lab resources, they are available in the Prerequisites section.

Click **Continue**.

#### Step 4b — Review Extracted Fields

NADI processes both images and opens the **Document Q\&A** panel.

![NADI — Extracted fields pending review](../.gitbook/assets/NADI-testing-mark-review1.png)

The panel shows:

* **All (5)** tab — all 5 configured fields
* **To review (5)** tab — fields pending your review
* Each field shows its extracted value below the field name
* Fields not yet reviewed appear with a grey spinner icon

Example extracted values: `error code` = `37`, `model detais` = `VER5000W`

#### Step 4c — Mark All Fields as Reviewed

For each field, click **Mark As Reviewed** to confirm the extracted value is correct.

![NADI — Mark As Reviewed button](../.gitbook/assets/NADI-testing-mark-review2.png)

> Mark all 5 extracted fields as reviewed. Each field card turns green and the **To review** count decrements.

#### Step 4d — Verify All Fields are Green

![NADI — All fields reviewed and green](../.gitbook/assets/NADI-testing-mark-review-validate.png)

* **All fields should be green** — each field card shows a green checkmark
* **To review count should become 0** — confirmed you have reviewed all fields

> If any field shows an unexpected value, click the edit (pencil) icon to correct it before submitting.

#### Step 4e — Confirm and Submit

Click **Submit** (top right). The **Confirm field predictions** dialog appears.

![NADI — Confirm and submit dialog](../.gitbook/assets/NADI-confirm-submit-field.png)

The dialog reads: _"Confirm that you've reviewed all fields for accuracy before you submit."_

Click **Confirm and submit**.

> The error code field (`37` in this example) is the key value — this populates `u_extracted_error_code` on the Incident and triggers the Resolution Pathfinder Agentic Workflow.

***

### Step 5: Add the Integration

1. Navigate to the **Integrations** step in the use case
2. Click **Add Integration**

![NADI — Integration config 1](<../.gitbook/assets/NADI-8-integration (1).png>)

| Field                | Value             |
| -------------------- | ----------------- |
| **Integration Name** | `Veritas_Process` |
| **Target Table**     | `incident extend` |
| **Integration type** | `Process task`    |
| **Create Flow**      | Checked           |

**Condition** — configure when the flow should fire:

![NADI — Process integration condition](../.gitbook/assets/NADI-process-integrate-condition.png)

| Field    | Operator       |
| -------- | -------------- |
| `Number` | `is not empty` |

> This ensures the integration only fires on records with a valid incident number.

3. Ensure **Create Flow** is checked, then click **Save**

![NADI — Integration config 2](<../.gitbook/assets/NADI-8-integration2 (1).png>)

This auto-generates:

```
Flow: DocIntel Extract Values Flow — Veritas Extract
Trigger: Document Task Updated WHERE status = Done
```

***

### Step 6: Activate the Integration

1. Locate **Veritas\_Process** in the Integrations panel

![NADI — Activate integration](<../.gitbook/assets/NADI-8-activate3 (1).png>)

2. Click **Open in Flow Designer**
3. Click **Activate** — status updates to **Active**

***

### Step 7: Configure the Incident Integration Trigger

1. Navigate to **Integrations** tab of the use case
2. Add a second integration:

![NADI — Integration trigger config 1](<../.gitbook/assets/NADI-9-integration (1).png>)

| Field                | Value             |
| -------------------- | ----------------- |
| **Integration Name** | `Veritas_Extract` |
| **Target Table**     | `incident extend` |
| **Integration type** | `Extract Values`  |
| **Create Flow**      | Checked           |

![NADI — Integration trigger config 2](<../.gitbook/assets/NADI-9-integration2 (1).png>)

3. Verify the trigger activates correctly

![NADI — Activate trigger step 1](<../.gitbook/assets/NADI-9-integration-activate3 (1).png>)

![NADI — Activate trigger step 2](<../.gitbook/assets/NADI-9-integration-activate4 (1).png>)

***

### Step 8: Review and Activate the Use Case

Review the full configuration on the **Review and activate** screen (Step 5 of the wizard).

![NADI — Review and activate summary](../.gitbook/assets/NADI-setup-step5.png)

**Verify use case details:**

| Field         | Expected value    |
| ------------- | ----------------- |
| Use case name | `Veritas Extract` |
| Target table  | `incident extend` |

**Verify all 5 fields are present:**

| Field name    | Type | Required |
| ------------- | ---- | -------- |
| error code    | Text | Yes      |
| model details | Text | Yes      |
| product name  | Text | Yes      |
| serial number | Text | Yes      |
| pn bar code   | Text | Yes      |

**Verify both integrations are present:**

| Integration name               | Type           |
| ------------------------------ | -------------- |
| `Veritas_Extract Integrations` | Extract Values |
| `Veritas_Process`              | Process Task   |

Click **Complete setup**.

![NADI — Use case set successfully](../.gitbook/assets/NADI-step5-complete.png)

> **"Your use case has been set"** — Click **Return to use cases**.

***

### Step 9: Verify Full Automation End-to-End

1. Click the **Settings** (gear) icon on the use case
2. Navigate to **Extraction mode**
3. Toggle **Full automation mode (no agent review required)** to **On**

![NADI — Full automation verification](<../.gitbook/assets/NADI-10-usecase-full-auto (1).png>)

![NADI — Full automation result](<../.gitbook/assets/NADI-10-usecase-full-auto2 (1).png>)

> Full automation mode means GenAI extracts and writes fields immediately — no agent review required. This is required for the Requestor Flow so the Agentic Workflow trigger can fire within the same session.

***

## Key Configuration Fields

| Field             | Value for This Lab                 |
| ----------------- | ---------------------------------- |
| Skill             | Extract Information from documents |
| Use case name     | `Veritas Extract`                  |
| Target table      | Incident extend table              |
| Extraction mode   | Full automation (no agent review)  |
| Integration 1     | `Veritas_Extract` — Extract Values |
| Integration 2     | `Veritas_Process` — Process Task   |
| Process condition | Number is not empty                |

***

## Technical Notes

### Full Automation Mode vs. Agent Review Mode

| Mode                | Behaviour                                        | Use When                                    |
| ------------------- | ------------------------------------------------ | ------------------------------------------- |
| **Full automation** | GenAI extracts and writes immediately, no review | Consistent document layouts (device labels) |
| **Agent review**    | Human reviews before values are written          | Complex or variable documents               |

***

## Reference

* [ServiceNow Zurich — Now Assist in Document Intelligence](https://www.servicenow.com/docs/bundle/zurich-intelligent-experiences/page/administer/document-intelligence/concept/docintel-nowassist-landing.html)
* [Use cases for Now Assist in Document Intelligence](https://www.servicenow.com/docs/r/intelligent-experiences/now-assist-in-document-intelligence/use-cases-now-assist-document-intelligence.html)

***

## Next Step

Continue to [04a — Now Assist Skill Kit: CreateOptimalSearchQuery](04a-now-assist-skill-kit-createoptimalsearchqueryskill.md) to build the first of three Custom Now Assist Skills needed for the Resolution Pathfinder Agent.
