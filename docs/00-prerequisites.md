# 00 — Prerequisites
 
> **Release:** Zurich | **Flow:** Pre-Build Setup
> **Complete these steps before starting any lab exercise.**
 
---
 
## What This Is
 
Before building the Agentic Workflow and AI Agents in this lab, there are a number of instance-level prerequisites that must be in place. Skipping these steps will result in cross-scope errors, missing components, or silent failures during the build exercises.
 
Complete every step below in order before proceeding to [01 — Now Assist for Virtual Agent (NAVA)](01-now-assist-virtual-agent.md).
 
---
 
### Pre-Requisite 1: Switch to the `x_nava_agentic_lab` Application Scope

All lab artefacts — topics, agents, agentic workflows, flow actions, and tables — live in the **x_nava_agentic_lab** scoped application. If you build or configure components in the wrong scope (e.g., Global), they will not be visible to the AI Agent at runtime and cross-scope privilege errors will occur.
 
### Steps
 
1. In the ServiceNow banner frame, click the **globe icon** (Application scope picker) in the top-right navigation bar
2. In the **Application scope** dropdown, type `x_nava` in the filter field
3. Select **x_nava_agentic_lab** from the results
 
![Change Application Scope to x_nava_agentic_lab](../screenshots/change-scope-to-x_nava_agentic_lab.png)
 
4. Confirm the scope picker now displays **x_nava_agentic_lab** as the active scope
 
> **This must remain your active scope for the entire lab.** If you navigate away and the scope resets to Global, switch back before making any changes. A common symptom of being in the wrong scope is seeing *"Record not found"* or *"Insufficient privileges"* errors when saving flow actions or agent configurations.
 
---

## Pre-Requisite 2: Verify the Incident Extend Table and Sample Records
 
The lab scenario uses a **custom table called incident extend** (`x_snc_apacaienable_incident_extend`) that adds fields specific to the Veritas NetBackup triage use case — such as error codes, product, serial number, and barcode. This table must already exist on your instance and be populated with sample incident records before you begin the build.
 
### Steps
 
1. In the **Filter navigator** (left-hand sidebar), type `incident extend` in the search field
2. Under **All Results**, click **incident extends** to open the list view
 
![Filter Navigator — incident extends](../screenshots/inc-extend-tbl.png)
 
3. Confirm the **incident extends** list view loads and displays sample records
4. Verify that records are populated with Veritas NetBackup-related data — you should see incidents with short descriptions referencing hardware overheating, error codes (e.g., error 84, error 37, status code 2817), and categories such as **Hardware** and **Software**
 
![Incident Extends — Sample Records](../screenshots/inc-extend-tbl-main.png)
 
> **What to look for:** The records should include a mix of categories, assignment groups (e.g., IT Operations Support, IT Client Systems Engineering), and states. These sample records are used by Predictive Intelligence for training and by the AI Agent for pattern matching during triage. If the table is empty or missing, the downstream Agentic Workflow will have no historical data to reference when generating resolution plans.
 
| Field | Expected Value |
|-------|----------------|
| Table name | `x_snc_apacaienable_incident_extend` |
| List URL path | `x_snc_apacaienable_incident_extend_list.do` |
| Number prefix | `INCE` |
| Minimum sample records | 10+ |
| Record categories | Hardware, Software, Inquiry / Help |
 
> **If the table is missing or empty:** Contact your lab administrator to import the Update Set that creates the `x_snc_apacaienable` scoped application and its seed data. The table and records are delivered as part of the lab instance provisioning — they are not created during the build exercises.
 
---
 
## Checklist
 
| # | Pre-Requisite |
|---|---------------|
| 1 | Application scope set to `x_nava_agentic_lab` |
 
---
 
## Next Step
 
Continue to [01 — Now Assist for Virtual Agent (NAVA)](01-now-assist-virtual-agent.md) to configure the conversational entry point for the lab.
