# 01 — Now Assist for Virtual Agent (NAVA)

> **Source:** [ServiceNow Zurich Documentation — Enable AI experiences](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/ai-products.html)

## What It Is

Now Assist for Virtual Agent (NAVA) is the AI-powered conversational interface that users interact with across ServiceNow portals, mobile, and workspace experiences. In Zurich, NAVA is powered by large language models (LLMs) and connected to the Now Assist AI Agent ecosystem — meaning conversations are not scripted trees but are handled dynamically by AI agents that reason, use tools, and take action.

NAVA serves as the **entry point** for the end-to-end lab scenario. When a user types "I can't access the server", NAVA receives that message, runs it through the Natural Language Understanding (NLU) engine to classify intent, and routes the conversation to the appropriate **Now Assist AI agent**.

NAVA is distinct from the Virtual Agent platform it runs on:
- **Virtual Agent** — the underlying conversational platform with topics, flows, and scripted responses
- **Now Assist for Virtual Agent** — the generative AI layer that enables LLM-powered responses, AI agent handoffs, and intelligent slot-filling via Knowledge Graph

---

## When to Use

- When you want users to interact with AI agents via chat rather than filling out forms
- When you need dynamic, reasoning-based conversations rather than static conversation trees
- When Knowledge Graph slot-filling should reduce how many questions users need to answer
- When AI agents need a conversational front door that routes to the right handler

---

## Customer Value

| Business Outcome | How NAVA Delivers It |
|-----------------|----------------------|
| Reduced form abandonment | Conversational intake replaces structured forms — users describe the issue in their own words |
| Higher intent accuracy | LLM + NLU classifies intent more accurately than keyword matching |
| Faster triage | AI agent receives conversation context immediately; no re-collection needed |
| Consistent experience | Same chat interface whether the agent resolves the issue or escalates it |
| Slot-filling via Knowledge Graph | Known user data (name, department, CI) pre-populates fields — fewer questions asked |

---

## How It Works — Architecture

```
User opens Service Portal / Employee Center / Mobile
        │
        ▼
Now Assist for Virtual Agent (NAVA)
        │
        ▼
NLU Engine — classifies intent
        │
        ▼
Conversational Topic: IT Infrastructure Issue
        │
        ▼
AI Agent handoff: First Responder Operations Analyst
        │
        ▼
AI Agent responds, uses tools, creates Incident
```

---

## How to Build

### Prerequisites

- Now Assist Pro+ or ITSM Pro+ licence with Now Assist for Virtual Agent activated
- Virtual Agent plugin active on the instance
- Role: `sn_va_designer` or `admin`
- NLU model active (included with Now Assist)

---

## Lab Exercise — Steps to Build

### Step 1: Verify Now Assist for Virtual Agent is Activated

1. Navigate to **All** > search **Now Assist** > **Administration** > **Setup**
2. Confirm **Now Assist for Virtual Agent** is listed as **Active**
3. If not active, request activation through **Now Assist Setup Assistant**

### Step 2: Verify the NLU Model is Active

1. Navigate to **All** > search **Natural Language Understanding** > **NLU Models**
2. Confirm the default **Now Assist NLU Model** status is **Published**
3. If unpublished, click the model > **Publish**

### Step 3: Configure the Virtual Agent for Now Assist

1. Navigate to **All** > search **Virtual Agent** > **Administration** > **Settings**
2. Under **Now Assist**, ensure:

| Setting | Value |
|---------|-------|
| Enable Now Assist for Virtual Agent | Checked |
| Default AI model | Now Assist LLM (default) |
| Conversation memory | Enabled |

3. **Save**

### Step 4: Configure the Now Assist Panel (for fulfillers)

The Now Assist Panel surfaces AI agent activity to fulfiller-side users (e.g., L2 agents viewing Incidents).

1. Navigate to **All** > search **Now Assist** > **Administration** > **Now Assist Panel**
2. Configure panel visibility for the **Incident** table
3. Enable **AI agent activity visibility** so fulfillers can see what the Resolution Pathfinder has done

### Step 5: Open the Service Portal Chat to Test

1. Navigate to your instance's **Service Portal** (`/sp`)
2. Click the chat icon (bottom right)
3. Verify the Virtual Agent widget loads and responds
4. Type: *"Hello"* — confirm NAVA responds with a greeting

> **Note:** The AI agent connection will be tested after building Capability 02 (Conversational Topics) and Capability 05 (AI Agent).

---

## Key Configuration Fields

| Field | Value for This Lab |
|-------|--------------------|
| Product | Now Assist for Virtual Agent |
| NLU model | Now Assist NLU (default) |
| AI model | Now Assist LLM |
| Chat channel | Service Portal / Employee Center |
| Role to configure | `sn_va_designer` or `admin` |

---

## Reference

- [ServiceNow Zurich — Enable AI experiences](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/ai-products.html)
- [Now Assist AI agents overview](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/na-ai-agents.html)

---

## Next Step

Continue to [02 — Conversational Topics](02-conversational-topics.md) to configure the NLU intent that routes server issue reports to the First Responder AI Agent.
