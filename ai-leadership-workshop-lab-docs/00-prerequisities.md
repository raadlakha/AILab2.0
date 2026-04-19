# Prerequisites

> **Release:** Zurich | **Flow:** Pre-Build Setup **Complete these steps before starting any lab exercise.**

***

## What This Is

Before building the Agentic Workflow and AI Agents in this lab, there are a few instance-level prerequisites that must be in place. Skipping these steps can result in cross-scope errors, missing components, or silent failures during the build exercises.

Complete every step below in order before proceeding to [01 — Now Assist for Virtual Agent (NAVA)](01-now-assist-virtual-agent.md).

Lab resources that you will need in order to complete the lab: [Lab resources folder](../Lab%20resources/)

***

### Pre-Requisite 1: Switch to the `x_nava_agentic_lab` Application Scope

All lab artefacts — topics, agents, agentic workflows, flow actions, and tables — live in the **x\_nava\_agentic\_lab** scoped application. If you build or configure components in the wrong scope (e.g., Global), they will not be visible to the AI Agent at runtime and cross-scope privilege errors will occur.

### Steps

1. In the ServiceNow banner frame, click the **globe icon** (Application scope picker) in the top-right navigation bar
2. In the **Application scope** dropdown, type `x_nava` in the filter field
3. Select **x\_nava\_agentic\_lab** from the results

![Change Application Scope to x\_nava\_agentic\_lab](<../.gitbook/assets/change-scope-to-x_nava_agentic_lab (1).png>)

4. Confirm the scope picker now displays **x\_nava\_agentic\_lab** as the active scope

> **This must remain your active scope for the entire lab.** If you navigate away and the scope resets to Global, switch back before making any changes. A common symptom of being in the wrong scope is seeing _"Records not found"_ or _"Insufficient privileges"_ errors when saving flow actions or agent configurations.

***

## Pre-Requisite 2: Change Subflow Run As to System User

The **Create and submit Incident record with image upload(s) subflow** is invoked by the L1 First Responder Operations Analyst Agent to create Incident records. By default, the subflow's **Run As** property is set to **User who initiates session** — meaning it executes with the permissions of the chat user. This can cause permission failures when the subflow attempts to create records on tables that the chat user does not have write access to.

You need to change the **Run As** property to **System User** so the subflow executes with elevated permissions regardless of who initiated the chat session.

### Steps

1. In the **Filter navigator**, type `workflow studio`
2. Under **Process Automation**, click **Workflow Studio**

![Filter Navigator — Workflow Studio](../.gitbook/assets/flowaction-check1.png)

3. In Workflow Studio, click the **Subflows** tab
4. Filter the subflow list by **Name contains `Create and submit`**

![Workflow Studio — Subflows Tab with Filter](../.gitbook/assets/flowaction-check2.png)

5. Confirm the subflow **Create and submit Incident record with image upload(s) subflow** appears — it should show **Published**, **Active: true**, and belong to the `x_nava_agentic_lab` application

![Subflows — Filtered Result](../.gitbook/assets/flowaction-check3.png)

6. Click on the subflow name to open it
7. Click on Edit subflow to begin editing the subflow

![Subflow — Inputs and Structure](../.gitbook/assets/flowaction-check4.png)

8. Click the **kebab menu** (three-dot icon, top-right) and select **Properties**

![Subflow — Kebab Menu with Properties](../.gitbook/assets/flowaction-check5.png)

9. In the **Subflow properties** dialog, expand **Advanced Options**
10. Change the **Run As** dropdown from `User who initiates session` to **`System User`**

![Subflow Properties — Run As Changed to System User](../.gitbook/assets/flowaction-check6.png)

11. Click **Update** to save the change and finally, publish the subflow again

> **Why System User?** When the AI Agent fires this subflow during a Virtual Agent chat session, the session runs as the end user. If Run As is set to "User who initiates session", the subflow inherits that user's ACLs — which typically do not include read/write access to the Incident Extend custom table. Setting Run As to **System User** ensures the subflow has the necessary permissions to create the Incident record, attach uploaded images, and set all required fields — regardless of who is chatting with the agent.

***

## Pre-Requisite 3: Provide sn_aia.admin role to Admin user

### Steps

1. In the **Filter navigator**, type `user admin`
2. Under **User Administration**, click **Users**

![Filter Navigator — User Administration > Users](../.gitbook/assets/check-user-permissions-1.png)

***

### Navigate to Admin user

Alex Rai is our requestor personna!

3. In the **Users** list, filter by **User ID starts with `admin`**

![Users List — Filtered by admin](../screenshots/check-user-permissions-8.png)

4. Click on **admin** to open the User record
5. Scroll down to the **Roles** section and click on **Edit**

![User Record — Roles of admin user](../screenshots/check-user-permissions-9.png)

6. Add (move to the right) the **sn_aia.admin** role to the Admin user. Once done, click on the **Save** button.

![Add sn_aia.admin role to Admin user)](../screenshots/check-user-permissions-10.png)

6. Finally, click on the **Update** button on the top right hand corner to ensure that the sn_aia.admin role addition is done successfully.

![Save role changes to Admin user)](../screenshots/check-user-permissions-11.png)

***

## Checklist

| # | Pre-Requisite                                                         |
| - | --------------------------------------------------------------------- |
| 1 | Application scope set to `x_nava_agentic_lab`                         |
| 2 | Subflow Run As changed to System User                                 |
| 3 | Add sn_aia.admin role to Admin user                                   |

***

## Next Step

Continue to [01 — Now Assist for Virtual Agent (NAVA)](01-now-assist-virtual-agent.md) to configure the conversational entry point for the lab.
