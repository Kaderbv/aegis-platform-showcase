# Design Record

Aegis is built documentation-first: an RFC proposes a design, a review checklist and log capture the
discussion, and an ADR records the decision with its alternatives and consequences. The full documents are in
the private repository; this page indexes them.

## Architecture Decision Records

| ADR | Decision |
|---|---|
| 002 | FastAPI as the API framework |
| 003 | LangGraph for agent orchestration |
| 004 | A provider-independent LLM interface |
| 005 | Streaming and error boundaries: adapters own SDK streaming; the agent sees one interface |
| 006 | NVIDIA as the default provider, with OpenAI as a configurable alternative |
| 007 | Bounded retries: allowlisted transient errors, exponential backoff with jitter, total deadline, no retry after streamed output |
| 008 | Observability as a provider decorator, composed as provider → retrying → observed |
| 009 | RAG chunking strategies (recursive and markdown) |
| 010 | RAG service provider wiring |
| 011 | Real token streaming through LangGraph, with mapping from API errors to HTTP responses |
| 012 | The retrieval path contract and the order of the safety layers |
| 013 | Batch embedding contract with `passage` / `query` input types and token usage |
| 014 | Grounded answers in the agent: an opt-in RAG node, citations, caller identity |
| 015 | App factory, background model warm-up, and readiness gating |
| 016 | File upload ingestion (.txt, .md, .pdf, .docx) |
| 017 | Document integrity: tenant-scoped identity, delete, change detection, failure stages |
| 018 | PostgreSQL persistence: SQLAlchemy 2 async on asyncpg, Alembic, repository protocols |

## Requests for Comments

| RFC | Topic |
|---|---|
| 001 | Platform foundation |
| 002 | FastAPI foundation |
| 003 | Agent orchestration |
| 004 | LLM provider abstraction |
| 005 | Production-ready LLM streaming and provider reliability |
| 006 | Multi-provider support |
| 007 | Provider reliability and failure handling |
| 008 | LLM observability and operational monitoring |
| 009 | RAG architecture |
| 010 | PostgreSQL persistence: document registry, ingestion jobs, conversations, agent runs |

## Other artifacts

- **PRD** (product requirements) and **SRS** (software requirements specification)
- **C4 model**: system context, containers, components, and code tree
- **EDI**: an engineering decision index tracing each release to its decisions
- **API references** for the `/agent` and `/rag` endpoints, plus RAG safety configuration and usage guides
- **Experiment reports** for provider latency, streaming, retries, observability, embedding and vector store
  selection, grounded answers, and warm-up
