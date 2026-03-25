# 03 — Knowledge Graph

> **Source:** [ServiceNow Zurich Documentation — Knowledge Graph](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/knowledge-graph/knowledge-graph-landing.html)

## What It Is

The **Knowledge Graph** application uses the structured and unstructured data from ServiceNow records, knowledge bases, and external sources to enhance the performance of Now Assist Virtual Agent, AI agents, and generative AI skills.

As described in the Zurich docs:

> *"ServiceNow Knowledge Graph application enhances the Now Platform by creating a semantic layer that connects data, relationships, and context across the enterprise. It structures information as a graph of entities and connections, bringing context and meaning to the available raw data. By leveraging AI, it powers personalized and intelligent experiences across the Now Assist ecosystem including NAVA, AI Agents, and AI Search to deliver more relevant insights and automation."*

In this lab, Knowledge Graph is used as a **tool** in both:
- The **First Responder AI Agent** — to search for KB articles and provide immediate resolution guidance
- The **Resolution Pathfinder AI Agent** — as part of the internal data resolution search

Knowledge Graph enables **natural language queries** over your ServiceNow data. Instead of keyword search, it understands meaning — so "server unreachable" surfaces content about "HOST_UNREACHABLE", "network connectivity failure", and "host not responding".

---

## When to Use

- When your organisation has Knowledge Base articles containing troubleshooting procedures
- When you want AI agents to search KB content as part of their resolution workflow
- When you want users to ask natural language questions like "Who is my manager?" or "What's assigned to me?"
- When slot-filling should pre-populate forms using known user context (role, department, location)
- When AI agents need to retrieve facts and relationships directly from the graph to automate actions

---

## Customer Value

| Business Outcome | How Knowledge Graph Delivers It |
|-----------------|----------------------------------|
| Higher first-contact resolution | AI finds relevant KB articles even when user describes the issue in their own words |
| Personalised AI responses | User context (role, department, location) shapes AI agent responses |
| Slot-filling at scale | Known data pre-populates forms — users answer fewer questions |
| Smarter AI agents | Agents retrieve structured facts (CI details, relationships) without additional API calls |
| Continuous improvement | Schema grows over time to capture new entity relationships |

---

## How It Works — Technical Overview

```
ServiceNow records (KB articles, CIs, Users, Incidents)
        │
        ▼
Knowledge Graph schema (nodes = entities, edges = relationships)
        │
        ▼
AI Agent or NAVA sends natural language query
        │
        ▼
Knowledge Graph returns structured answer
  • "What KB articles mention HOST_UNREACHABLE?"
  • "Who is the caller's manager?"
  • "What CI is assigned to this server?"
        │
        ▼
AI Agent uses context to personalise response or fill slots
```

**Key components:**
- **Knowledge Graph Designer** — no-code UI for `kg_admin` users to create and manage schemas
- **Nodes** — entities (e.g., `incident`, `cmdb_ci`, `sys_user`, `kb_article`)
- **Edges** — relationships between entities (e.g., User *has* Incident, CI *belongs to* Department)
- **Prebuilt integrations** — out-of-the-box connections to NAVA, AI agents, Now Assist Panel

**Knowledge Graph user roles:**

| User role | KG Functionality | Description |
|-----------|-----------------|-------------|
| `kg_admin` | Knowledge Graph Designer | Create and manage Knowledge Graph schemas |
| Requester | Prebuilt integration with NAVA and Agentic AI | Personalised answers, natural language queries, slot-filling |
| Fulfiller | Prebuilt integration with Now Assist Panel and Agentic AI | Personalised answers in fulfilment workspace |

---

## How to Build

### Prerequisites

- Knowledge Management plugin active
- Now Assist for ITSM or AI Search activated
- At least one published Knowledge Base with articles
- Role: `kg_admin` or `admin`

---

## Lab Exercise — Steps to Build

### Step 1: Access Knowledge Graph Designer

1. Navigate to **All** > search **Knowledge Graph**
2. Click **Knowledge Graph Designer**
3. Review the available prebuilt schemas (employee schema, user NLQ graph are provided out of the box)

### Step 2: Create a Knowledge Base for IT Infrastructure

1. Navigate to **Knowledge** > **Knowledge Bases** > **New**
2. Fill in:

| Field | Value |
|-------|-------|
| Name | `IT Infrastructure Runbooks` |
| Description | `Troubleshooting guides and runbooks for infrastructure issues` |
| Workflow | `Instant Publish` |

3. Click **Submit**

### Step 3: Create Sample Knowledge Articles

Create at least 3 articles to give the AI agent content to search against:

**Article 1: Server Inaccessibility Troubleshooting**
```
Title: Server Inaccessibility — Diagnosis and Resolution Steps

When a server becomes inaccessible, follow these steps:
1. Verify network connectivity using ping and traceroute from a jump host
2. Check firewall rules — ensure port 22 (SSH) or 3389 (RDP) is not blocked
3. Review server health in the CMDB for recent changes
4. Check CPU and memory utilisation via monitoring dashboard
5. If the server is a VM, verify the hypervisor host is healthy
6. Restart the network interface if connectivity is confirmed but access fails
Error codes: ERR_NETWORK_TIMEOUT, CONNECTION_REFUSED, HOST_UNREACHABLE
```

**Article 2: Infrastructure Error Code Reference**
```
Title: Infrastructure Error Code Reference

ERR_NETWORK_TIMEOUT: The server did not respond within the expected time.
  Resolution: Check network path, firewall, and server load.
CONNECTION_REFUSED: Server is reachable but not accepting connections on the port.
  Resolution: Check if the service is running. Restart if needed.
HOST_UNREACHABLE: No route to host.
  Resolution: Check routing tables, VPN, and VLAN configuration.
DISK_FULL: Server disk utilisation at 100%.
  Resolution: Archive logs, clear temp files, expand volume.
```

**Article 3: Escalation Runbook**
```
Title: Infrastructure Issue — Escalation Runbook

If initial troubleshooting steps do not resolve the issue:
1. Collect server logs from /var/log/syslog and /var/log/messages
2. Capture the exact error message and timestamp
3. Identify if the issue is isolated to one server or affects multiple
4. Escalate to L2 Infrastructure team with: server name, error code, logs, steps taken
5. If production system: invoke P1 bridge call immediately
```

To create each article:
1. Navigate to **Knowledge** > **Create New Article**
2. Select the `IT Infrastructure Runbooks` knowledge base
3. Paste the content above
4. Set **Workflow state** to **Published**

### Step 4: Connect Knowledge Graph to AI Agents

To use Knowledge Graph as a tool in an AI agent (Capability 05 and 06):

1. Navigate to **All** > **AI Agent Studio** > **Create and manage** > **AI agents**
2. Open the relevant AI agent
3. Under **Tools**, click **Add tool**
4. Select tool type: **Knowledge Graph**
5. Configure:

| Field | Value |
|-------|-------|
| Tool name | `KnowledgeGraphSearch` |
| Knowledge Graph schema | IT Infrastructure Runbooks (or enterprise employee schema) |
| Description | `Searches the IT Infrastructure Runbooks knowledge base for troubleshooting steps and resolution guides` |

### Step 5: Test Natural Language Queries

1. In **Knowledge Graph Designer**, open the schema
2. Use the **Query tester** to run: `"steps to resolve HOST_UNREACHABLE"`
3. Verify relevant KB article content is returned

---

## Key Configuration Fields

| Field | Value for This Lab |
|-------|--------------------|
| Knowledge Base | IT Infrastructure Runbooks |
| Knowledge Graph schema | Linked to IT Infrastructure KB |
| Admin role | `kg_admin` |
| AI agent tool type | Knowledge Graph |
| Prebuilt integration | NAVA slot-filling, AI agent search |

---

## Reference

- [ServiceNow Zurich — Knowledge Graph](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/knowledge-graph/knowledge-graph-landing.html)
- [Exploring Knowledge Graph](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/knowledge-graph/exploring-knowledge-graph.html)
- [Configuring Knowledge Graph](https://www.servicenow.com/docs/r/zurich/intelligent-experiences/knowledge-graph/configuring-knowledge-graph.html)

---

## Next Step

Continue to [04 — Now Assist in Document Intelligence](04-now-assist-document-intelligence.md) to configure AI-powered extraction of error codes from files users upload during the conversation.
