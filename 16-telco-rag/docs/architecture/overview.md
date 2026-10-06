# Architecture Overview — Target Project Structure

## 1. Document Information

| Field | Value |
| --- | --- |
| Project | `telco-rag` |
| Document | Architecture Overview |
| Scope | Repository structure and architectural boundaries |
| Status | Target architecture |

This document defines the target project structure for `telco-rag` and the
dependency boundaries that the structure must preserve.

It complements, and does not replace, the following documents:

| Document | Responsibility |
| --- | --- |
| `docs/constitution.md` | Non-negotiable principles |
| `docs/architecture.md` | Layered architecture and system-wide design |
| `docs/implementation-plan.md` | Stage-by-stage construction order |
| `docs/domain.md`, `docs/data-model.md` | Domain model and persistence model |
| `docs/configuration.md` | Configuration architecture |
| Per-package structure sections | Module layout inside each package |

---

## 2. Purpose

The structure below is the target architecture. It matches the architecture and
the implementation plan, while keeping the system modular enough to evolve into
agentic RAG later.

The most important part of this structure is not the number of files. It is the
dependency direction described in section 6.

---

## 3. Target Project Structure

```text
telco-rag/
│
├── README.md
├── pyproject.toml
├── uv.lock
├── .env.example
├── .gitignore
├── docker-compose.yml
├── Makefile
├── alembic.ini
│
├── config/
│   ├── settings.yaml
│   ├── ingestion.yaml
│   ├── retrieval.yaml
│   ├── evaluation.yaml
│   ├── security.yaml
│   ├── observability.yaml
│   │
│   └── profiles/
│       ├── dev.yaml
│       ├── test.yaml
│       └── prod.yaml
│
├── docs/
│   ├── constitution.md
│   ├── product.md
│   ├── prd.md
│   ├── architecture.md
│   ├── domain.md
│   ├── data-model.md
│   ├── configuration.md
│   ├── providers.md
│   ├── application.md
│   ├── ingestion.md
│   ├── retrieval.md
│   ├── query.md
│   ├── generation.md
│   ├── grounding.md
│   ├── security.md
│   ├── security-llm.md
│   ├── evaluation.md
│   ├── observability.md
│   ├── api.md
│   ├── testing.md
│   ├── implementation-plan.md
│   │
│   ├── architecture/
│   │   ├── overview.md
│   │   ├── ingestion.md
│   │   ├── retrieval.md
│   │   ├── security.md
│   │   └── agentic-rag.md
│   │
│   ├── domain/
│   │   ├── telco-domain.md
│   │   ├── personas.md
│   │   └── use-cases.md
│   │
│   ├── experiments/
│   │   ├── 01_baseline-rag.md
│   │   ├── 02_chunking.md
│   │   ├── 03_hybrid-retrieval.md
│   │   ├── 04_reranking.md
│   │   └── 05_query-routing.md
│   │
│   └── decisions/
│       ├── ADR-001-database.md
│       ├── ADR-002-vector-search.md
│       └── ADR-003-llm-provider.md
│
├── data/
│   ├── raw/
│   │   ├── network/
│   │   ├── products/
│   │   ├── support/
│   │   ├── incidents/
│   │   ├── runbooks/
│   │   └── sla/
│   │
│   ├── processed/
│   │
│   └── eval/
│       ├── questions.jsonl
│       ├── retrieval_cases.jsonl
│       └── expected_answers.jsonl
│
├── scripts/
│   ├── generate_data.py
│   ├── ingest_documents.py
│   ├── rebuild_index.py
│   └── run_evaluation.py
│
├── migrations/
│   ├── env.py
│   ├── script.py.mako
│   └── versions/
│
├── src/
│   └── telco_rag/
│       ├── __init__.py
│       ├── main.py
│       │
│       ├── api/
│       │   ├── __init__.py
│       │   ├── app.py
│       │   ├── dependencies.py
│       │   │
│       │   └── routes/
│       │       ├── __init__.py
│       │       ├── health.py
│       │       ├── query.py
│       │       ├── documents.py
│       │       ├── ingestion.py
│       │       └── incidents.py
│       │
│       ├── application/
│       │   ├── __init__.py
│       │   ├── dependencies.py
│       │   ├── errors.py
│       │   ├── query.py
│       │   ├── documents.py
│       │   ├── ingestion.py
│       │   ├── incidents.py
│       │   └── evaluation.py
│       │
│       ├── domain/
│       │   ├── __init__.py
│       │   ├── documents.py
│       │   ├── chunks.py
│       │   ├── users.py
│       │   ├── access.py
│       │   ├── incidents.py
│       │   ├── tickets.py
│       │   └── products.py
│       │
│       ├── ingestion/
│       │   ├── __init__.py
│       │   ├── pipeline.py
│       │   ├── cleaning.py
│       │   ├── chunking.py
│       │   ├── metadata.py
│       │   ├── embedding.py
│       │   │
│       │   └── loaders/
│       │       ├── __init__.py
│       │       ├── pdf.py
│       │       ├── docx.py
│       │       ├── markdown.py
│       │       └── text.py
│       │
│       ├── retrieval/
│       │   ├── __init__.py
│       │   ├── vector.py
│       │   ├── keyword.py
│       │   ├── hybrid.py
│       │   ├── reranker.py
│       │   ├── filters.py
│       │   └── retriever.py
│       │
│       ├── query/
│       │   ├── __init__.py
│       │   ├── classifier.py
│       │   ├── rewriter.py
│       │   ├── router.py
│       │   └── models.py
│       │
│       ├── generation/
│       │   ├── __init__.py
│       │   ├── generator.py
│       │   ├── prompts.py
│       │   ├── citations.py
│       │   └── grounding.py
│       │
│       ├── security/
│       │   ├── __init__.py
│       │   ├── authentication.py
│       │   ├── authorization.py
│       │   ├── acl.py
│       │   └── filters.py
│       │
│       ├── evaluation/
│       │   ├── __init__.py
│       │   ├── datasets.py
│       │   ├── retrieval.py
│       │   ├── generation.py
│       │   └── runner.py
│       │
│       ├── observability/
│       │   ├── __init__.py
│       │   ├── logging.py
│       │   ├── tracing.py
│       │   └── metrics.py
│       │
│       ├── infrastructure/
│       │   ├── __init__.py
│       │   ├── database.py
│       │   │
│       │   ├── repositories/
│       │   │   ├── __init__.py
│       │   │   ├── documents.py
│       │   │   ├── chunks.py
│       │   │   ├── incidents.py
│       │   │   ├── tickets.py
│       │   │   └── users.py
│       │   │
│       │   ├── llm/
│       │   │   ├── client.py
│       │   │   └── openrouter.py
│       │   │
│       │   ├── embeddings/
│       │   │   ├── client.py
│       │   │   └── openrouter.py
│       │   │
│       │   └── reranking/
│       │       ├── client.py
│       │       └── openrouter.py
│       │
│       └── config/
│           ├── __init__.py
│           ├── models.py
│           ├── loader.py
│           ├── settings.py
│           └── validation.py
│
├── prompts/
│   ├── query/
│   │   ├── classify.jinja
│   │   └── rewrite.jinja
│   │
│   ├── generation/
│   │   ├── answer_v1.jinja
│   │   └── answer_v2.jinja
│   │
│   └── agents/
│       └── investigation.jinja
│
├── tests/
│   ├── unit/
│   │   ├── domain/
│   │   ├── application/
│   │   ├── ingestion/
│   │   ├── retrieval/
│   │   ├── query/
│   │   ├── generation/
│   │   ├── security/
│   │   └── config/
│   │
│   ├── integration/
│   │   ├── test_database.py
│   │   ├── test_ingestion.py
│   │   ├── test_retrieval.py
│   │   ├── test_rag.py
│   │   └── test_security.py
│   │
│   ├── contract/
│   │   ├── test_llm_provider.py
│   │   ├── test_embedding_provider.py
│   │   └── test_reranker_provider.py
│   │
│   ├── evaluation/
│   │   ├── test_retrieval_quality.py
│   │   └── test_generation_quality.py
│   │
│   └── e2e/
│       └── test_api_flow.py
│
└── frontend/
    ├── package.json
    ├── README.md
    └── src/
        ├── components/
        ├── pages/
        ├── api/
        └── types/
```

---

## 4. Top-Level Directory Responsibilities

| Path | Responsibility | Authority |
| --- | --- | --- |
| `config/` | Non-secret configuration and environment profiles | `docs/configuration.md` |
| `data/` | Synthetic raw documents, processed output, evaluation datasets | `docs/ingestion.md` |
| `docs/` | Specifications, architecture, experiments, ADRs | `docs/constitution.md` §28 |
| `prompts/` | Versioned prompt templates, external to application code | `docs/generation.md` §10 |
| `migrations/` | Alembic schema migrations | `docs/constitution.md` §14 |
| `scripts/` | Developer and operations entry points | `docs/implementation-plan.md` |
| `src/telco_rag/` | Application code | `docs/architecture.md` |
| `tests/` | Unit, integration, contract, evaluation, and e2e tests | `docs/testing.md` |
| `frontend/` | Optional UI; not created until a UI need is demonstrated | — |

`src/telco_rag/main.py` is the composition root. It loads configuration once and
wires components. Business logic must not construct configuration objects.

---

## 5. Structure Rules

The structure is enforceable. The following rules MUST hold.

### 5.1 Source layout

- Application code must live under `src/telco_rag/`.
- Application code must not be placed directly in the repository root.
- Tests must remain outside `src`.
- `pyproject.toml` must declare dependencies explicitly.
- The project must use `uv` with a committed `uv.lock`.

### 5.2 Prompts are external files

- Prompts must not be inline string literals in Python modules.
- Prompt templates must be versioned so that behaviour changes are evaluable.
- Prompt changes are behavioural changes and must be evaluated, not merely
  merged.

### 5.3 Migrations

- Database schema changes must be managed through Alembic migrations.
- Manually modified schemas must not be relied upon.
- `migrations/versions/` is the only source of schema truth.

### 5.4 Configuration and secrets

- Secrets must never be committed.
- `.env.example` documents required variables without real credentials.
- Secrets must not be stored in `config/*.yaml`.

### 5.5 Frontend is deferred

`frontend/` is part of the target architecture but must not be created until a
concrete UI requirement exists. The initial system is an API.

---

## 6. Important Architectural Boundaries

The most important part of this structure is not the number of files; it is the
dependency direction.

```text
                    ┌─────────────────┐
                    │      API        │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  Application    │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │     Domain      │
                    └─────────────────┘
                             ▲
                             │
              ┌──────────────┴──────────────┐
              │                             │
      ┌───────┴────────┐           ┌────────┴────────┐
      │  Infrastructure │           │ AI/RAG Modules  │
      │                 │           │                │
      │ PostgreSQL      │           │ Ingestion      │
      │ OpenRouter      │           │ Retrieval      │
      │ Repositories    │           │ Query          │
      │ Embeddings      │           │ Generation     │
      └─────────────────┘           └────────────────┘
```

Rules that follow from this diagram:

- Dependencies flow inward only: API to Application, Application to Domain.
- The Domain layer depends on nothing.
- Infrastructure and AI/RAG modules depend on Domain and Application
  abstractions, never the reverse.
- Domain code must not import PostgreSQL, FastAPI, LangChain, LangGraph,
  OpenRouter, or a specific vector database.

---

## 7. Agentic RAG Extension

Later, when agentic RAG is introduced, the Application layer gains a second
branch. The Domain layer does not change.

```text
API
 │
 ▼
Application
 │
 ├── Traditional RAG
 │      ├── Query
 │      ├── Retrieval
 │      ├── Grounding
 │      └── Generation
 │
 └── Agentic RAG
        ├── State
        ├── Planner
        ├── Tools
        └── Graph
                │
                ▼
          Existing RAG capabilities
```

Constraints on the agentic branch:

- Agentic functionality must not be introduced before the core RAG pipeline is
  measurable and reliable.
- Agents must expose explicit tools with explicit action boundaries.
- The initial agent must not autonomously execute production network changes.
- Agents must reuse existing retrieval, security, and grounding capabilities
  rather than reimplementing them.

---

## 8. Incremental Creation

Do not implement all of these directories now. This is the target architecture.
The repository should be created incrementally according to
`docs/implementation-plan.md`.

| Stage | Directories introduced |
| --- | --- |
| Stage 0 — Repository Foundation | `src/`, `tests/`, `config/`, `docs/`, `data/`, `scripts/`, root files |
| Stage 1 — Domain Model | `src/telco_rag/domain/` |
| Stage 2 — Configuration | `src/telco_rag/config/`, `config/profiles/` |
| Stage 3 — PostgreSQL and pgvector | `migrations/`, `src/telco_rag/infrastructure/` |
| Stage 4 — Synthetic Telecom Data | `data/raw/`, `data/eval/`, `scripts/generate_data.py` |
| Stage 5+ — Loaders, Chunking, Embeddings | `src/telco_rag/ingestion/` |
| Stage 9+ — Retrieval | `src/telco_rag/retrieval/`, `src/telco_rag/query/` |
| Stage 12+ — Generation | `src/telco_rag/generation/`, `prompts/generation/` |
| Security stages | `src/telco_rag/security/` |
| Evaluation stages | `src/telco_rag/evaluation/`, `prompts/evaluation/` |
| Observability stages | `src/telco_rag/observability/` |
| Production API stage | `src/telco_rag/api/`, `src/telco_rag/application/` |
| Agentic RAG stage | `src/telco_rag/agents/`, `prompts/agents/` |
| On demonstrated UI need | `frontend/` |

Each stage must preserve the boundaries established by previous stages. A
directory that exists ahead of its stage must remain empty rather than being
filled with speculative code.

---

## 9. Deviations From This Structure

Any deviation must be recorded as an ADR under `docs/decisions/` with a written
rationale, the affected boundary, and the trade-off. Deviations must not be made
implicitly by adding files.