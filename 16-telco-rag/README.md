# telco-rag

An enterprise Retrieval-Augmented Generation platform for telecommunications
organizations.

`telco-rag` ingests telecom enterprise knowledge, retrieves it with hybrid
search under server-side authorization, and produces answers that are grounded
in — and cited to — that evidence. The LLM is a generation component; it is
never the source of truth.

> **Project status: specification and architecture phase.** The design is
> complete and ratified. Application code has not been written yet. See
> [Current Status](#current-status) before assuming any functionality exists.

---

## Why

Telecom engineers answer questions under time pressure during live incidents
and change windows. Generic chat assistants fail them in three specific ways:

- They answer from model memory instead of authoritative internal
  documentation, with no way to verify the source.
- They cannot be constrained to what a given engineer is authorized to see.
- They confidently confabulate configuration values, thresholds, and procedures
  that sound plausible and are wrong.

`telco-rag` addresses each of these at the architectural level rather than with
prompt instructions. Authorization is enforced before retrieved content reaches
the model. Answers cite document, version, page, and chunk. When evidence is
insufficient the system abstains instead of guessing.

---

## Core Principles

The full set is defined in [`.specify/memory/constitution.md`](.specify/memory/constitution.md).
The load-bearing ones:

| Principle | Rule |
| --- | --- |
| Enterprise RAG first | Retrieval, authorization, grounding, evaluation, and observability are first-class capabilities, not features of a chatbot |
| Retrieval is a first-class system | Independently testable and evolvable without touching generation |
| Security before generation | Authorization is enforced before any retrieved content enters the LLM context. Non-negotiable |
| Grounded and cited answers | Every material claim traces to evidence; abstention is preferred over unsupported output |
| Telecom domain fidelity | 5G/LTE, VoLTE, fiber, SLAs, incidents, runbooks, regions, and products are modelled explicitly |
| Layered architecture | Dependencies flow inward only: API → Application → Domain |
| Provider independence | Application code depends on `LLMProvider`, never on a vendor client |
| Evaluation-driven development | No change is called an improvement without a benchmark result |

---

## Architecture

Dependencies point inward. The Domain layer depends on nothing.

```text
                    ┌─────────────────┐
                    │      API        │
                    └────────┬────────┘
                             ▼
                    ┌─────────────────┐
                    │  Application    │
                    └────────┬────────┘
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

Agentic RAG is added later as a second branch under Application — State,
Planner, Tools, Graph — reusing the existing retrieval, security, and grounding
capabilities rather than reimplementing them. See
[`docs/architecture/overview.md`](docs/architecture/overview.md) §7.

### Pipeline

```text
Source → Load → Parse → Clean → Normalize → Metadata → Chunk → Validate → Embed → Index

Query → Normalize → Classify → Rewrite → [Authorize] → Retrieve
     → Filter → Rerank → Assemble Evidence → Generate → Cite → Validate
```

Security metadata must exist and validate **before** indexing. Unclassified
documents are quarantined, never silently admitted to the searchable corpus.

---

## Technology

Python 3.12+ · `uv` · Pydantic · FastAPI · PostgreSQL · pgvector · SQLAlchemy ·
Alembic · OpenRouter · pytest · Docker

LangChain, LangGraph, Ragas, OpenTelemetry, Prometheus, and a frontend are
permitted only when they solve a demonstrated problem, and must remain
implementation details behind application boundaries.

---

## Current Status

There is no application code in this repository. `src/` and `tests/` exist but
are empty, and `pyproject.toml` declares no dependencies.

| Area | State |
| --- | --- |
| Product specification | Complete — `product.md`, `prd.md` |
| Architecture | Complete — `architecture.md`, `architecture/overview.md` |
| Domain and data model | Complete — `domain.md`, `data-model.md` |
| Subsystem designs | Complete — ingestion, retrieval, query, generation, grounding, security, evaluation, observability, api, testing |
| Constitution | Ratified v1.0.0 |
| Target project structure | Defined — `docs/architecture/overview.md` |
| Implementation plan | Defined — `docs/implementation-plan.md`, Stages 0–N |
| Application code | Not started |
| Repository scaffold | Not created — no `uv.lock`, `.gitignore`, `.env.example`, `Makefile`, `docker-compose.yml`, `alembic.ini`, `config/` |

Because there is no code and no dependency set, `uv run pytest` will not run
yet. Stage 0 (Repository Foundation) of
[`docs/implementation-plan.md`](docs/implementation-plan.md) is the first
executable step and its deliverable is a clean `uv run pytest`.

A prior prototype (2026-09-14/15) explored domain entities, SQLAlchemy
repositories, Alembic migrations, and LlamaIndex-based parsers, chunking, and
embeddings. It was removed in the 2026-09-16 repository cleanup in favour of
the specification-first approach and the provider-independent architecture. See
[`CHANGELOG.md`](CHANGELOG.md).

---

## Documentation

### Foundations

| Document | Contents |
| --- | --- |
| [`docs/constitution.md`](docs/constitution.md) | Non-negotiable principles, technology constraints, governance |
| [`docs/product.md`](docs/product.md) | Product vision, users, knowledge domains, capabilities |
| [`docs/prd.md`](docs/prd.md) | User stories `US-001`–`US-011`, functional requirements `FR-001`–`FR-030` |
| [`docs/architecture.md`](docs/architecture.md) | Layered architecture and system-wide design |
| [`docs/architecture/overview.md`](docs/architecture/overview.md) | Target repository structure and dependency boundaries |
| [`docs/implementation-plan.md`](docs/implementation-plan.md) | Stage-by-stage construction order |

### Domain and cross-cutting

| Document | Contents |
| --- | --- |
| [`docs/domain.md`](docs/domain.md) | Entities, aggregates, value objects, invariants, lifecycles |
| [`docs/data-model.md`](docs/data-model.md) | Tables, columns, constraints, indexes, relationships |
| [`docs/configuration.md`](docs/configuration.md) | Typed configuration, precedence, profiles, secrets |
| [`docs/providers.md`](docs/providers.md) | LLM, embedding, and reranker provider abstraction |

### Subsystems

| Document | Contents |
| --- | --- |
| [`docs/ingestion.md`](docs/ingestion.md) | Loaders, parsing, cleaning, metadata, chunking |
| [`docs/retrieval.md`](docs/retrieval.md) | Vector, keyword, hybrid, fusion, reranking |
| [`docs/query.md`](docs/query.md) | Normalization, classification, rewriting, routing |
| [`docs/generation.md`](docs/generation.md) | Context assembly, prompts, structured output, citation |
| [`docs/grounding.md`](docs/grounding.md) | Evidence model, sufficiency, conflict, claim support |
| [`docs/security.md`](docs/security.md) | Classification, RBAC/ABAC, retrieval filtering, audit |
| [`docs/security-llm.md`](docs/security-llm.md) | LLM-boundary security: injection, leakage, exfiltration |
| [`docs/evaluation.md`](docs/evaluation.md) | Retrieval and generation metrics, datasets, thresholds |
| [`docs/observability.md`](docs/observability.md) | Traces, spans, metrics, logging, cost |
| [`docs/api.md`](docs/api.md) | Endpoints, contracts, error model, status codes |
| [`docs/testing.md`](docs/testing.md) | Test tiers, markers, CI, quality gates |
| [`docs/application.md`](docs/application.md) | Application services, orchestration, boundaries |

### Planned

`docs/architecture/{ingestion,retrieval,security,agentic-rag}.md`,
`docs/domain/{telco-domain,personas,use-cases}.md`, `docs/experiments/`, and
`docs/decisions/` (ADR-001 database, ADR-002 vector search, ADR-003 LLM
provider) are part of the target structure and will be written as their
subjects are implemented.

---

## Contributing

Changes are governed by the constitution. In short:

1. Read [`docs/constitution.md`](docs/constitution.md) and the relevant
   subsystem document.
2. Follow the workflow in `.specify/memory/constitution.md` — Specification →
   Design → Plan → Implementation → Unit Tests → Integration Tests →
   Evaluation → Documentation.
3. Never weaken a non-negotiable principle without an ADR under
   `docs/decisions/`.
4. Record architecture decisions as ADRs and experiment results under
   `docs/experiments/`.
5. Security-sensitive changes require automated regression tests.

---

## License

Not yet specified.