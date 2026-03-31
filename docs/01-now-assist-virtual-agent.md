# 01 — Now Assist for Virtual Agent (NAVA)

> **Release:** Zurich | **Flow:** Requestor Flow — Phase 1 (Step 1)
> **Source:** [ServiceNow Zurich — Using Now Assist in Virtual Agent](https://www.servicenow.com/docs/bundle/zurich-conversational-interfaces/page/administer/now-assist-in-va/concept/using-now-assist-in-va.html) | [Configuring Now Assist in Virtual Agent](https://www.servicenow.com/docs/bundle/zurich-conversational-interfaces/page/administer/now-assist-in-va/task/configure-now-assist-va.html)

---

## What It Is

**Now Assist for Virtual Agent (NAVA)** is the AI-powered conversational interface that serves as the **entry point for the entire Requestor Flow**. It is the surface through which users interact with the platform — via Service Portal, Employee Center, or Mobile — using natural language rather than structured forms.

In Zurich, NAVA is no longer a scripted conversation tree. It is powered by large language models (LLMs) and the **AI Agent Fabric**, which means conversations are handled dynamically by AI agents that reason, use tools, and take action. When a user types *"I can't access the server"*, NAVA receives the message, stamps `contact_type = chat` on the session, and routes the conversation to the appropriate AI agent.

### NAVA vs. Virtual Agent — Key Distinction

| Component | Role |
|-----------|------|
| **Virtual Agent** | The underlying conversational platform — topics, flows, channel routing, NLU model |
| **Now Assist for Virtual Agent (NAVA)** | The generative AI layer — LLM-powered responses, AI agent handoffs, Knowledge Graph slot-filling, agentic reasoning |

NAVA is built on top of Virtual Agent. Both must be active for this lab to function.

---

## Role in the Requestor Flow

![NAVA Requestor Flow — Step 1 Architecture](../screenshots/nava-requestor-flow.png)

> **Why `contact_type = chat` matters:** This field is the trigger condition for the downstream Agentic Workflow. If `contact_type` is not stamped as `chat`, the Resolution Pathfinder Agentic Workflow will not fire — even if all other conditions are met.

---

## What NAVA Enables in This Lab

| Capability | How NAVA Enables It |
|-----------|---------------------|
| Conversational intake | User describes the issue in natural language — no form filling |
| Session stamping | `contact_type = chat` is automatically applied to the session and propagates to the Incident record |
| AI Agent routing | NAVA's AI Agent Planner identifies the right agent (L1 Requestor Agent) and hands off the conversation |
| Knowledge Graph slot-filling | Known user context (name, department, CI) pre-populates fields — fewer questions asked |
| Agentic support | In Zurich Patch 2+, AI Agents are prioritised in VA responses by default via the **Agentic Support** setting |
| Search sources | NAVA queries configured search sources (Knowledge Base, AI Search) as part of response generation |

---

## Architecture — How NAVA Is Configured

In Zurich, NAVA is configured through the **Assistant Designer**, a unified configuration surface. The wizard steps through the following sections in order:

```
Assistant Designer — Setup Wizard Order
  1. Basic details
  2. Agentic support
  3. Search sources
  4. Knowledge Graphs
  5. Assets
  6. Display experience
  7. Branding
  8. Chat experience
  9. Chat features
 10. Review
```

---

## Prerequisites

| Requirement | Detail |
|-------------|--------|
| Licence | Now Assist Pro+ or ITSM Pro+ with Now Assist for Virtual Agent |
| Plugin — Virtual Agent | `com.glide.cs.chatbot` — must be Active |
| Plugin — Now Assist in VA | `sn_now_assist_va` — must be Active |
| Plugin — Now Assist AI Agents | `sn_aia` — must be Active (v5.2+ recommended for Zurich) |
| Plugin — GAIC | `sn_gaic` — v11.2+ for agentic reasoning |
| Role | `sn_va_designer` or `admin` |
| Zurich patch | Patch 2+ for AI Agents prioritisation; Patch 4+ for Agentic Support configuration in Settings UI |

---

## Lab Exercise — Steps to Configure NAVA

### Step 1: Verify Now Assist for Virtual Agent is Activated

1. Navigate to **All** → search **Now Assist** → **Administration** → **Setup**
2. Confirm **Now Assist for Virtual Agent** is listed as **Active**
3. If not active, use the **Now Assist Setup Assistant** to activate it

### Step 2: Open the Assistant Designer

Navigate to **Conversational Interfaces** → **Assistant Designer**

![Assistant Designer — Create Chat-Based Assistant](../screenshots/nava-create-chat-based-assistant.png)

> The Assistant Designer shows the **Assistants** tab. The default assistant is **Now Assist for Virtual Agent**. Click **Edit** to open the setup wizard.

---

### Step 3: Basic Details

The first section in the wizard is **Basic details**.

![NAVA Basic Details](../screenshots/nava-basic-detail.png)

Review and confirm:

| Field | Value for This Lab |
|-------|--------------------|
| Name | `Now Assist for Virtual Agent` |
| Description | AI-powered conversational assistant for IT support |
| Status | Active |
| Default language | English |

---

### Step 4: Agentic Support

The second section is **Agentic Support**. This is a Zurich Patch 2+ setting.

![NAVA Agentic Support](../screenshots/nava-agentic-support.png)

| Setting | Value |
|---------|-------|
| Enable Agentic Support | On |
| AI Agents prioritised in planner | Enabled (system property: `sn_aia.use_agents_in_planner = true`) |

> This ensures the AI Agent Planner evaluates registered agents **before** falling back to scripted topics. The L1 Requestor Agent will be evaluated first for every IT infrastructure message.

> **Important:** Once the **AI Agents** skill is enabled and you upgrade to Zurich Patch 2+, Virtual Agent with agentic reasoning is enabled. **This change cannot be reversed.** Uncheck before upgrading if you do not want it.

---

### Step 5: Search Sources

The third section is **Search Sources**.

![NAVA Search Sources](../screenshots/nava-search-source.png)

Add the following search source for this lab:

| Source | Type | Purpose |
|--------|------|---------|
| IT Infrastructure Runbooks | Knowledge Base | KB articles containing troubleshooting guides and resolution steps |

> The search profile controls what content NAVA searches when generating responses. In the Requestor Flow, this feeds the L1 Agent's KB lookup before creating an Incident.

---

### Step 6: Knowledge Graphs

The fourth section is **Knowledge Graphs**.

![NAVA Knowledge Graph](../screenshots/nava-kg.png)

Verify the Knowledge Graph is connected. This enables:
- User identification via the **User Graph** (role, department, previous interactions)
- Slot-filling is for pre-populating `caller_id`, `cmdb_ci`, `department` from the user's graph context - however empty/None for today's configuration.
- Semantic search across KB articles indexed in the Knowledge Graph

> The L1 Requestor Agent uses the Knowledge Graph as **Tool 1** to query user context and identify the affected CI at the start of every conversation.

---

### Step 7: Assets

The fifth section is **Assets**.

![NAVA Assets](../screenshots/nava-assets.png)

Enable the following Now Assist skills (assets) for this lab:

| Skill | Required | Purpose |
|-------|----------|---------|
| **AI Agents** | ✅ Yes | Routes conversation to the L1 Requestor Agent |
| **Knowledge** | ✅ Yes | Enables KB article search in responses |
| **AI Search** | Optional | Enhances search quality across multiple sources |

---

### Step 8: Display Experience

The sixth section is **Display Experience** — where NAVA is surfaced.

![NAVA Display Experience](../screenshots/nava-display.png)

1. Open or create the **Service Portal** display experience
2. Verify the **Now Assist Panel** is enabled for the fulfiller workspace

![NAVA Display — Portal](../screenshots/nava-display-portal1.png)

> **Display experiences** determine which portal or workspace the assistant is embedded in. For this lab: **Service Portal** (`/sp`) chat widget.

---

### Step 9: Chat Experience & Chat Features

The seventh through ninth sections are **Branding**, **Chat experience**, and **Chat features**.

For this lab, **Branding** can be left at defaults. The critical setting is in **Chat features**:

![NAVA Chat Features](../screenshots/nava-chat-features.png)

| Setting | Value |
|---------|-------|
| Enable Now Assist for Virtual Agent | Checked |
| Enable file upload in chat | ✅ Checked — required for Requestor Flow Step 4 (screenshot upload) |
| Default AI model | Now Assist LLM (default) or your configured provider |
| Conversation memory | Enabled |

> **File upload in chat** must be enabled here. Without it, the file upload prompt in the Conversation Topic (Capability 02) will not function, and the user cannot submit error screenshots to trigger Document Intelligence.

---

### Step 10: Review and Save

The final section is **Review**. Confirm all settings look correct across each section, then click **Save**.

---

### Step 11: Verify the Chat Widget in Service Portal

1. Navigate to your instance's **Service Portal** (`/sp`)
2. Click the chat icon (bottom-right corner)
3. Verify the Virtual Agent widget loads and NAVA responds
4. Type: *"I can't access the server"*
5. Confirm the L1 Requestor Agent activates (after completing Capability 05)

> The full end-to-end test — including agent handoff, Knowledge Graph search, file upload, and incident creation — is performed after completing Capabilities 02 through 07.

---

## Key Configuration Fields

| Field | Value for This Lab |
|-------|--------------------|
| Assistant | Now Assist for Virtual Agent |
| Display experience | Service Portal (`/sp`) |
| Skills (Assets) enabled | AI Agents, Knowledge |
| File upload | Enabled in Chat features |
| Agentic support | Enabled (Zurich Patch 2+) |
| Search source | IT Infrastructure Runbooks KB |
| Knowledge Graph | Connected |
| Session field stamped | `contact_type = chat` |
| Role to configure | `sn_va_designer` or `admin` |

---

## Technical Notes

### `contact_type = chat` — How It Gets Stamped

When a user interacts with NAVA via the Service Portal chat widget, ServiceNow automatically stamps `contact_type = chat` on the virtual agent session. When the L1 Agent subsequently creates an Incident, this value is carried forward to the `contact_type` field on the Incident record.

This is a **platform behaviour** — no manual configuration needed. However it is critical to verify post-incident creation, as it is one of three conditions that gate the Resolution Pathfinder Agentic Workflow:

```
Agentic Workflow trigger conditions:
  ✓ state = In Progress (2)
  ✓ contact_type = chat
  ✓ u_extracted_error_code ≠ empty
```

If `contact_type` is not `chat` (e.g., incident created via email or form), the Agentic Workflow will not trigger.

### Agentic Reasoning — Zurich Patch 2+ Behaviour Change

From Zurich Patch 2 (Now Assist in VA v12.x / GAIC v11.2), the following default behaviours change:

- Default LLM switches to **GPT-4.1** (NowLLM users switch to GPT-OSS)
- AI Agents are **prioritised** over scripted topics in every response
- Multi-intent is supported and appears in responses
- Disambiguation follow-up questions are only asked for very unclear/one-word queries

> Source: [Virtual Agent with agentic reasoning — ServiceNow Community](https://www.servicenow.com/community/now-assist-articles/fully-agentic-virtual-agent-in-the-october-release-and-what-it/ta-p/3406077)

---

## Reference

- [ServiceNow Zurich — Using Now Assist in Virtual Agent](https://www.servicenow.com/docs/bundle/zurich-conversational-interfaces/page/administer/now-assist-in-va/concept/using-now-assist-in-va.html)
- [ServiceNow Zurich — Configuring Now Assist in Virtual Agent](https://www.servicenow.com/docs/bundle/zurich-conversational-interfaces/page/administer/now-assist-in-va/task/configure-now-assist-va.html)
- [Now Assist in Virtual Agent — Resources Guide](https://www.servicenow.com/community/now-assist-articles/now-assist-in-virtual-agent-resources-guide/ta-p/3052139)
- [Virtual Agent with agentic reasoning in Zurich](https://www.servicenow.com/community/now-assist-articles/fully-agentic-virtual-agent-in-the-october-release-and-what-it/ta-p/3406077)

---

## Next Step

Continue to [02 — Conversational Topics](02-conversational-topics.md) to configure the NLU intent and Conversation Topic that routes server issue reports to the L1 Requestor Agent, and to set up the **file upload tool** used in Step 4 of the Requestor Flow.
