---
description: >-
  An employee raises the alert, AI takes cares of the rest. From troubleshooting
  to remediation
---

# ServiceNow-AI-Lab: Incident Resolution and Remediation

> **Platform:** ServiceNow · **Release:** Zurich **Team:** Global Intelligent Automation SC Team · APJ

***

## The Problem This Lab Solves

Enterprise IT operations teams face a fundamental tension: ticket volumes are growing faster than resolution capacity, yet the data and tools needed to resolve most incidents already exist inside the organisation.

Three compounding problems drive this gap:

**1. L1 agents spend most of their time on known, repeatable errors.** The majority of IT incidents involve error patterns with documented resolutions sitting in knowledge bases or resolved ticket history. Yet agents still handle each one manually — searching, reading, composing, and responding — because there is no mechanism to connect the error to the resolution without human effort.

**2. Incident records arrive without structured diagnostic data.** When users report errors, they describe symptoms in natural language or attach screenshots. The error code, affected system, and device details that would allow automated triage are buried in unstructured text or images. Without extraction, every ticket starts cold.

**3. Resolution requires crossing system boundaries that no single agent can navigate.** A complete resolution may require searching internal KB, querying live server logs, searching the internet for known issues, and then executing a fix on external infrastructure — a chain of actions that spans multiple platforms and today requires multiple human specialists to coordinate.

This lab demonstrates how ServiceNow's native AI capabilities — combined with the Agent-to-Agent (A2A) protocol — collapse all three of these gaps into a single autonomous pipeline.

***

## What This Lab Demonstrates

**NAVA Agentic Incident Resolution** is a fully documented, hands-on reference lab that builds the **Veritas Resolution Agentic Workflow** on ServiceNow Zurich.

The pipeline takes an IT incident from first user message to closed ticket — autonomously — without mandatory human intervention. It demonstrates:

* **Multi-agent orchestration** across a Requestor Agent, a Fulfiller Agent, and an External Azure AI Foundry agent
* **Cross-modal data extraction** — Now Assist Document Intelligence reading error codes from uploaded screenshots
* **Cascading search intelligence** — KB → live Elastic logs via MCP → privacy-safe internet search, in order of confidence
* **Cross-platform autonomous execution** — ServiceNow dispatching a remediation plan to Azure AI Foundry via the A2A protocol and receiving the execution result
* **Governed escalation paths** — every failure path writes a complete diagnostic trail before handing off to L2

***

## Why This Matters for Solution Consultants

ServiceNow's AI story in 2026 centres on three themes: **Agentic AI**, **AI Agent Fabric**, and **Now Assist as a platform** — not just a feature. Customers are asking:

* _"Can ServiceNow actually fix things, or just raise tickets?"_
* _"How does your AI work with my existing Azure investment?"_
* _"What happens when your AI can't solve something — does it just fail silently?"_
* _"Is there human oversight, or does AI act without guardrails?"_

This lab gives Solution Consultants a working, demo-ready answer to every one of these questions — built on a real instance, with real ServiceNow capabilities, using a realistic IT incident scenario.

By completing this lab, an SC will be able to:

* Explain and demonstrate the full NAVA-to-resolution flow end-to-end
* Configure Now Assist Document Intelligence to extract structured data from images
* Build custom Now Assist skills using the Skill Kit, including skill chaining
* Register and activate an external A2A agent (Azure AI Foundry) in AI Agent Studio
* Construct a governed Agentic Workflow with correct trigger conditions, security controls, and channel configuration
* Position ServiceNow as the AI orchestration layer for multi-cloud enterprise environments

***

## Lab Scenario

> A user reports that a Veritas server is throwing errors. They send a message to NAVA and upload screenshot(s) of the error screen. The platform extracts the error code from the image, creates an incident, searches the KB and live logs for a resolution, generates a remediation plan, and dispatches it to Azure AI Foundry for execution — all without a human touching the ticket.

***

## Solution Architecture

```
User Message → NAVA Chat (Service Portal)
        │
        ▼
Phase 1 — Requestor Flow
┌──────────────────────────────────────────────┐
│  L1 First Responder Operations Analyst Agent │
│  • Troubleshooting guide delivered in chat   │
│  • User uploads error screenshot(s)          │
│  • Now Assist Document Intelligence extracts │
│    key fields from image(s)                  │
│  • Incident created: state = In Progress,    │
│    channel = chat, error_code populated      │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
Phase 2 — Fulfiller Flow (Veritas Resolution Agentic Workflow)
┌──────────────────────────────────────────────┐
│  Resolution Pathfinder Agent                 │
│                                              │
│  Path A — Internal resolution:               │
│  • CreateOptimalSearchQuery Skill            │
│  • RetrieveRelevantKBContent (AI Search RAG) │
│  • ResolutionFinderInternalData Skill        │
│  • Assess if solution exists                 │
│       ↓ Resolution plan found                │
│       → Write to incident → Phase 3          │
│                                              │
│  Path B — Fallback:                          │
│  • Elastic log search via MCP                │
│  • GenerateWebSearchQns Skill                │
│  • Privacy-safe internet search              │
│       ↓ No resolution found                  │
│       → Escalate to L2 (Phase 3 skipped)     │
└──────────────────────┬───────────────────────┘
                       │ (Path A only)
                       ▼
Phase 3 — External Integration
┌──────────────────────────────────────────────┐
│  Observability & Action Agent (External/A2A) │
│  • Remediation plan dispatched to            │
│    Azure AI Foundry via A2A protocol         │
│  • OAuth 2.0 Client Credentials auth         │
│                                              │
│  Path 3A — Success:                          │
│  Incident auto-resolved, result written,     │
│  ticket closed                               │
│                                              │
│  Path 3B — Fail/Partial:                     │
│  Full execution log written to work notes,   │
│  incident escalated to L2 with full context  │
└──────────────────────────────────────────────┘
```

***

## Lab Documentation

Each document follows a consistent structure: what the capability is, where it fits in the flow, step-by-step build instructions, and key configuration details.

| #   | Document                                                                                                        | Phase | Capability                                              |
| --- | --------------------------------------------------------------------------------------------------------------- | ----- | ------------------------------------------------------- |
| 00  | [Prerequisites](docs/00-prerequisites.md)                                                                       | Setup | Instance requirements, plugins, roles                   |
| 01  | [Now Assist for Virtual Agent (NAVA)](docs/01-now-assist-virtual-agent.md)                                      | 1     | Chat entry point, VA configuration                      |
| 02  | [L1 First Responder Operations Analyst Agent](docs/02-L1-agent-first-responder-analyst-agent.md)                | 1     | L1 agent, KG lookup, image upload, incident creation    |
| 03  | [Now Assist Document Intelligence](docs/03-now-assist-document-intelligence.md)                                 | 1     | Error code extraction from uploaded screenshots         |
| 04a | [NASK — CreateOptimalSearchQuery](docs/04a-now-assist-skill-kit-createoptimalsearchqueryskill.md)               | 2     | Now Assist Skill Kit — AI search query generation       |
| 04b | [NASK — ResolutionFinderInternalData](docs/04b-now-assist-skill-kit-part2-resolutionfinderinternaldataskill.md) | 2     | NASK — KB retrieval + similar incidents skill           |
| 04c | [NASK — GenerateWebSearchQnsForResolutionPlan](docs/04c-now-assist-skill-kit-web-search.md)                     | 2     | NASK — skill chaining, privacy-safe web search          |
| 05  | [Elastic MCP Integration](docs/05-mcp-elastic-integration.md)                                                   | 2     | MCP client, Elastic log search configuration            |
| 06  | [Fulfiller AI Agent (Resolution Pathfinder)](docs/06-fulfiller-ai-agent.md)                                     | 2     | Resolution Finder Agent — tools, prompt, search cascade |
| 07  | [External Agent Integration — A2A](docs/07-external-agent-integration.md)                                       | 3     | OAuth 2.0 setup, External Agent registration, ObsAgent  |
| 08  | [Veritas Resolution Agentic Workflow](docs/08-wrapping-in-agentic-workflow.md)                                  | 2+3   | Agentic Workflow — trigger, agents, security, channels  |

***

## Key Technical Requirements

| Requirement        | Detail                                                                                                                |
| ------------------ | --------------------------------------------------------------------------------------------------------------------- |
| ServiceNow release | Zurich (Patch 4+ for A2A)                                                                                             |
| Licence            | Now Assist Pro+ or ITSM Pro+                                                                                          |
| Core plugins       | Now Assist AI Agents (`sn_aia`), Now Assist Skill Kit, Now Assist Document Intelligence, MCP Client (`sn_mcp_client`) |
| External systems   | Elastic instance with MCP-compatible endpoint                                                                         |
| External systems   | Azure AI Foundry ObsAgent with A2A-compliant Agent Card at `/.well-known/agent.json`                                  |
| Roles              | `sn_aia.admin`, `sn_skill_kit.admin`, `admin`                                                                         |

***

_ServiceNow · Global Intelligent Automation SC Team · India · April 2026_
