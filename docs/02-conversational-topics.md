# 02 — Conversational Topics

> **Source:** [ServiceNow Zurich Documentation — Natural Language Understanding](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/nlu-service/nlu-landing.html)

## What It Is

Conversational Topics are the intent-detection layer inside Now Assist for Virtual Agent (NAVA). When a user sends a message, the Natural Language Understanding (NLU) engine classifies their intent and maps it to a topic. In Zurich, a topic configured as **Now Assist** type acts as the bridge between what the user said and which **AI agent** should handle it.

In this lab, the Conversational Topic detects that the user is reporting a server accessibility issue and routes the conversation to the **First Responder Operations Analyst AI Agent**.

Topics in Zurich NAVA can be one of two types:
- **Classic Topics** — scripted conversation trees (legacy approach, still supported)
- **Now Assist Topics** — topics that hand off directly to an AI agent using LLM-powered conversation (this lab's approach)

---

## When to Use

- Any time you want NAVA to understand user intent and route to a specific AI agent
- When a specific business scenario (e.g., "server down", "can't access application", "VPN issue") needs a dedicated AI agent handler
- When you want to preserve the ability to escalate to a live agent if the AI agent cannot resolve
- When NLU confidence thresholds should control whether the AI agent or a fallback topic handles the request

---

## Customer Value

| Business Outcome | How Conversational Topics Deliver It |
|-----------------|--------------------------------------|
| Accurate intent routing | NLU maps natural language to the right AI agent handler — no mis-routes |
| Reduced configuration overhead | One topic handles all variations of "I can't reach the server" |
| Seamless AI agent handoff | User experience is uninterrupted — same chat window, same conversation |
| Measurable deflection | Topic-level analytics show how many issues were resolved vs. escalated |
| No rigid scripting | Now Assist topic type delegates all conversation handling to the AI agent |

---

## How It Works — Architecture

```
User message: "I can't access the production server"
        │
        ▼
  NLU Engine
  (classifies intent → "IT Infrastructure Issue")
  Confidence: 0.87 > threshold 0.70 → match
        │
        ▼
  Conversational Topic: IT Infrastructure Issue
  (Type: Now Assist)
        │
        ▼
  AI Agent: First Responder Operations Analyst
```

---

## How to Build

### Prerequisites

- Now Assist for Virtual Agent activated (Capability 01 complete)
- NLU model active and published
- Role: `sn_va_designer` or `admin`

---

## Lab Exercise — Steps to Build

### Step 1: Navigate to Virtual Agent Topics

1. Navigate to **All** > search **Virtual Agent** > **Designer**
2. Click **New Topic**

### Step 2: Create the Topic

Fill in the following:

| Field | Value |
|-------|-------|
| Name | `IT Infrastructure Issue` |
| Description | `Handles server, network, and infrastructure accessibility issues` |
| Topic type | **Now Assist** (not Classic) |
| AI Agent | *(leave blank — set after building Capability 05)* |

Click **Submit**.

### Step 3: Add Training Utterances

Training utterances teach the NLU to recognise this intent. Add at least 8–10 variations:

```
"I can't access the server"
"Server is down"
"Production server is unreachable"
"I'm getting a connection timeout to the server"
"Server inaccessible"
"Can't connect to the database server"
"Server not responding"
"Network issue with server"
"Application server is unavailable"
"I'm unable to reach the file server"
```

To add utterances:
1. In the topic designer, click the **Training** tab
2. Click **Add utterance** for each phrase above
3. Click **Train Model** when done
4. Wait for training to complete (status: **Published**)

### Step 4: Configure the Now Assist Handoff

1. In the topic, click the **Now Assist** tab
2. Under **AI Agent**, select **First Responder Operations Analyst** *(configure after Capability 05)*
3. Under **Handoff message**, set: `"Let me look into that for you."`
4. Under **Escalation**, configure live agent fallback if desired

### Step 5: Set Topic Priority

1. Navigate to **Virtual Agent** > **NLU** > **Intent Ranking**
2. Ensure `IT Infrastructure Issue` is ranked above any generic fallback topics
3. Set **Minimum confidence threshold** to `0.70` (recommended)

### Step 6: Publish and Test

1. Click **Publish** on the topic
2. Open Service Portal chat (`/sp`)
3. Type: *"The production server is not responding"*
4. Verify: NLU correctly identifies the intent (check **Conversation Logs** for confidence score)

---

## Key Configuration Fields

| Field | Value for This Lab |
|-------|--------------------|
| Topic Type | Now Assist |
| NLU Model | Default Now Assist NLU |
| AI Agent | First Responder Operations Analyst *(after Cap. 05)* |
| Min. confidence threshold | 0.70 |
| Fallback | Live Agent or classic VA escalation topic |

---

## Tip: Viewing NLU Confidence

After testing, navigate to **Virtual Agent** > **Logs** > **Conversation Logs** to see which intent was matched and the confidence score for each message.

---

## Reference

- [ServiceNow Zurich — Natural Language Understanding](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/nlu-service/nlu-landing.html)

---

## Next Step

Continue to [03 — Knowledge Graph](03-knowledge-graph.md) to configure the semantic data layer that AI agents use to find resolution steps and personalise responses.
