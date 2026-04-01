# 00 — Prerequisites
 
> **Release:** Zurich | **Flow:** Pre-Build Setup
> **Complete these steps before starting any lab exercise.**
 
---
 
## What This Is
 
Before building the Agentic Workflow and AI Agents in this lab, there are a number of instance-level prerequisites that must be in place. Skipping these steps will result in cross-scope errors, missing components, or silent failures during the build exercises.
 
Complete every step below in order before proceeding to [01 — Now Assist for Virtual Agent (NAVA)](01-now-assist-virtual-agent.md).
 
---
 
## Pre-Requisite 1: Switch to the `x_nava_agentic_lab` Application Scope
 
All lab artefacts — topics, agents, agentic workflows, flow actions, and tables — live in the **x_nava_agentic_lab** scoped application. If you build or configure components in the wrong scope (e.g., Global), they will not be visible to the AI Agent at runtime and cross-scope privilege errors will occur.
 
### Steps
 
1. In the ServiceNow banner frame, click the **globe icon** (Application scope picker) in the top-right navigation bar
2. In the **Application scope** dropdown, type `x_nava` in the filter field
3. Select **x_nava_agentic_lab** from the results
 
![Change Application Scope to x_nava_agentic_lab](../screenshots/change-scope-to-x_nava_agentic_lab.png)
 
4. Confirm the scope picker now displays **x_nava_agentic_lab** as the active scope
 
> **This must remain your active scope for the entire lab.** If you navigate away and the scope resets to Global, switch back before making any changes. A common symptom of being in the wrong scope is seeing *"Record not found"* or *"Insufficient privileges"* errors when saving flow actions or agent configurations.
 
---
 
## Checklist
 
| # | Pre-Requisite |
|---|---------------|
| 1 | Application scope set to `x_nava_agentic_lab` |
 
---
 
## Next Step
 
Continue to [01 — Now Assist for Virtual Agent (NAVA)](01-now-assist-virtual-agent.md) to configure the conversational entry point for the lab.
