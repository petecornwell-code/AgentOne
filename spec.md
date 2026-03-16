# AgentOne — Specification

## Overview

AgentOne is an Agentic AI solution designed to operate the same tools that a contact center human agent would use. Rather than requiring enterprise-scale integration with backend systems, AgentOne drives applications at the UI and API level — the same way a human operator does. This approach dramatically reduces integration complexity and time-to-value.

The long-term vision is for AgentOne to autonomously operate everything from the CCaaS interface to CRMs, Outlook, and enterprise-specific tools as required by the business process.

---

## Phase 1 — Email & CRM Query Handling

The first phase validates the core agent architecture by handling inbound customer queries using two channels:

- **Email** — reading, classifying, and responding to customer emails
- **CRM (Deskpro)** — looking up customer records, ticket history, and knowledge base articles to inform responses

### Phase 1 Goals

1. Receive an inbound customer query (email or ticket).
2. Retrieve relevant context from the CRM (customer record, prior tickets, KB articles).
3. Retrieve relevant context from embedded business data (policies, procedures, product info).
4. Compose and send an accurate, policy-compliant response.
5. Log all actions back to the CRM for audit and human review.

---

## Proposed Stack

| Layer | Technology | Role |
|---|---|---|
| **UI / Tool Control** | mcp-deskpro | MCP server for direct UI control of Deskpro. Allows the agent to interact with the CRM the same way a human would — navigating, reading, and updating records. |
| **API Gateway** | FastAPI | Endpoint management for inbound webhooks (email, ticket events), health checks, admin controls, and orchestration triggers. |
| **Agent Orchestration** | CrewAI | Defines agents, their tools, tasks, and the backing LLM. Manages multi-step reasoning, tool selection, and task delegation across the workflow. |
| **Data Access** | SQLAlchemy | ORM layer for business data and embedding metadata. Provides a clean abstraction over the database for both structured queries and vector similarity search. |
| **Data Store** | PostgreSQL + pgvector | Primary database. Stores structured business data (customers, products, policies) alongside vector embeddings for semantic retrieval (RAG). |

---

## Agent Design (CrewAI)

### Agents

| Agent | Responsibility |
|---|---|
| **Triage Agent** | Classifies the inbound query by intent, urgency, and customer. Pulls customer context from the CRM. Routes to the appropriate resolver. |
| **Resolver Agent** | Retrieves relevant knowledge (RAG + CRM history), drafts a response, validates it against policy, and sends the reply. Logs the resolution back to the CRM. |

### Tools

| Tool | Description |
|---|---|
| `deskpro_read` | Read customer records, tickets, and KB articles via mcp-deskpro. |
| `deskpro_write` | Create/update tickets, add notes, and change statuses via mcp-deskpro. |
| `email_send` | Send an email response to the customer. |
| `rag_search` | Semantic search over embedded business data (policies, procedures, product info) using pgvector. |
| `db_query` | Structured query against business data via SQLAlchemy. |

---

## Data Model (Phase 1)

### PostgreSQL Tables

- **customers** — customer master data (synced or referenced from CRM)
- **tickets** — local log of ticket interactions for audit
- **knowledge_articles** — business policies, procedures, product documentation
- **embeddings** — vector embeddings for knowledge articles and historical resolutions (pgvector)
- **agent_actions** — audit log of every action taken by AgentOne (what, when, why, outcome)

---

## Key Design Principles

1. **Operate like a human agent** — Drive tools at the UI/API level rather than requiring deep backend integration. This makes AgentOne deployable against any tool a human can use.
2. **Audit everything** — Every agent action is logged with reasoning. Human reviewers can inspect the full chain of thought and tool interactions.
3. **Policy compliance** — Responses are validated against embedded business rules before being sent. The Resolver Agent checks its draft against retrieved policies.
4. **Retrieval-Augmented Generation** — All responses are grounded in retrieved context (CRM data + embedded knowledge), not purely generated.
5. **Human-in-the-loop (configurable)** — Phase 1 supports a review queue where drafted responses await human approval before sending. This can be relaxed as confidence grows.

---

## Phase 1 Milestones

| # | Milestone | Description |
|---|---|---|
| 1 | **Environment & Stack** | Project scaffolding, Docker Compose for PostgreSQL + pgvector, FastAPI skeleton, CrewAI bootstrap. |
| 2 | **mcp-deskpro Integration** | Connect to Deskpro via MCP server. Validate read/write of tickets and customer records. |
| 3 | **RAG Pipeline** | Ingest knowledge articles, generate embeddings, store in pgvector. Validate semantic search. |
| 4 | **Agent Workflow** | Wire up Triage and Resolver agents in CrewAI with all tools. End-to-end test with a sample query. |
| 5 | **Email Channel** | Inbound email webhook, outbound email send. Full loop: email in → agent processes → email out. |
| 6 | **Audit & Review** | Agent action logging, human review queue, basic admin dashboard. |

---

## Future Phases (Out of Scope for Phase 1)

- **CCaaS integration** — voice and chat channel handling
- **Outlook / calendar control** — scheduling, meeting coordination
- **Enterprise tool expansion** — additional MCP servers for other line-of-business applications
- **Multi-agent collaboration** — specialist agents for billing, technical support, escalations
- **Continuous learning** — feedback loops from human reviewers to improve agent performance
