<!--
Sync Impact Report
==================

Version change: (uninitialized placeholder template) -> 1.0.0

Bump rationale: No prior filled constitution existed at
.specify/memory/constitution.md - the file was the untouched core scaffold.
This change is therefore the initial ratification, not an amendment to an
existing governance baseline, so the version starts at 1.0.0. The value 1.0.0
matches the version already declared by the project constitution in
docs/constitution.md, which is the source document for this ratification.

Source of project-specific values: docs/constitution.md (constitution.md),
amended with the target repository structure and architectural boundary rules
supplied in the current conversation.

Modified principles (old title -> new title)
--------------------------------------------
The template's five placeholder principles ([PRINCIPLE_1_NAME] ..
[PRINCIPLE_5_NAME]) were unpopulated. They are replaced by fifteen named
principles derived from docs/constitution.md:

  [PRINCIPLE_1_NAME] -> I. Enterprise RAG First
  [PRINCIPLE_2_NAME] -> II. Retrieval Is a First-Class System
  [PRINCIPLE_3_NAME] -> III. Security Before Generation (NON-NEGOTIABLE)
  [PRINCIPLE_4_NAME] -> IV. Grounded and Cited Answers
  [PRINCIPLE_5_NAME] -> V. Telecom Domain Fidelity
                      -> VI. Layered Architecture and Dependency Direction
                      -> VII. Provider and Framework Independence
                      -> VIII. Incremental Complexity
                      -> IX. Ingestion Provenance and Chunking Discipline
                      -> X. Configuration and Secret Management
                      -> XI. Evaluation-Driven Development
                      -> XII. Observability
                      -> XIII. Testing and Quality Gates
                      -> XIV. Synthetic Data First
                      -> XV. Human Oversight and Agentic Boundaries

Placeholder principle slots: 5 -> 15. The template explicitly permits a
different principle count; docs/constitution.md carries more than fifteen
non-negotiable constraints, and collapsing them further would drop enforceable
rules.

Added sections
--------------
  - Architecture Constraints and Technology Stack
      (replaces [SECTION_2_NAME] / [SECTION_2_CONTENT])
      Includes the canonical repository layout, the dependency-direction rule,
      versioned prompt files, Alembic migrations, and config/data/scripts
      placement. New governance content sourced from the conversation.
  - Development Workflow and Quality Gates
      (replaces [SECTION_3_NAME] / [SECTION_3_CONTENT])
  - Governance
      Amendment procedure, semantic versioning policy, compliance review.

Removed sections
----------------
  None. No governance content from docs/constitution.md was dropped. Sections
  that were not carried over verbatim were consolidated under the principles
  above or under Architecture Constraints.

Deferred items / TODOs
----------------------
  None. All placeholders are resolved. RATIFICATION_DATE and
  LAST_AMENDED_DATE are both 2026-10-05, derived from the git commit that
  introduced docs/constitution.md (8b95781) and confirmed against the system
  date; no TODO(...) marker was required.

Out-of-scope intents deferred (not written by this command)
-----------------------------------------------------------
  - Recording the target repository structure as a standalone architecture
    document under docs/architecture/.
  - Creating the config/, data/, prompts/, scripts/, migrations/, src/, tests/
    and frontend/ directory skeleton.
-->

# telco-rag Constitution

## Core Principles

### I. Enterprise RAG First

The product is an enterprise RAG platform, not an LLM chatbot. Enterprise
knowledge, retrieval, authorization, grounding, evaluation, and observability
MUST be treated as first-class system capabilities.

Rules:

- The LLM is a generation component and MUST NOT be treated as the source of
  truth.
- Every generated answer SHOULD be traceable to retrieved enterprise evidence.
- The system MUST be able to respond that sufficient evidence was not found
  rather than fabricating an answer.

Rationale: The system's value is trustworthy retrieval of authorized
enterprise knowledge. Optimizing for conversational fluency at the expense of
evidence traceability produces a system that cannot be governed, evaluated, or
trusted by telecom operations staff.

### II. Retrieval Is a First-Class System

Retrieval quality determines answer quality. Retrieval MUST be an independently
testable subsystem that can evolve without changing generation.

Rules:

- Retrieval strategies SHALL evolve in this order: vector search, keyword
  search, hybrid retrieval, metadata filtering, reranking, query routing.
- Retrieval components MUST expose measurable results.
- Retrieval evaluation MUST support at least Recall@K, Precision@K, MRR, and
  NDCG.
- Retrieval MUST be deterministic where possible.

Rationale: Retrieval is where authorization, relevance, and evidence quality
are decided. Treating it as a thin wrapper over a vector database makes quality
unmeasurable and regressions undetectable.

### III. Security Before Generation (NON-NEGOTIABLE)

Security is a mandatory architectural boundary, not a prompt-level
instruction.

Rules:

- Authorization MUST be enforced before any retrieved information is provided
  to the LLM.
- Unauthorized content MUST NOT enter the LLM context under any circumstance.
- The system MUST NOT rely on prompt instructions such as "do not reveal
  confidential information" as a security control.
- The required flow is: authentication, authorization, retrieval policy,
  authorized retrieval, context construction, LLM.
- Access MUST fail closed. If policy evaluation is unavailable, the request MUST
  be denied.
- At minimum these classifications MUST be supported: `PUBLIC`, `INTERNAL`,
  `CONFIDENTIAL`, `RESTRICTED`.
- Document access MAY depend on user, role, department, region, document
  classification, ownership, and explicit policy.
- Access control MUST be enforced server-side. Client-supplied authorization
  claims MUST NOT be trusted without validation.
- Security-sensitive behavior MUST have automated regression tests.

Rationale: Prompt-level controls are advisory and defeatable. In a telecom
enterprise context, leaking restricted operational knowledge is a
disclosure incident, so the enforcement point must sit below the model.

### IV. Grounded and Cited Answers

Generation MUST use retrieved enterprise evidence as its primary factual basis.

Rules:

- Answers SHOULD carry source citations whenever supporting evidence exists.
- A citation SHOULD identify, where available: document, document version,
  page, section, chunk.
- The system MUST distinguish `Supported answer`, `Unsupported answer`, and
  `Insufficient evidence` as distinct outcomes.
- When evidence is insufficient the system SHOULD abstain - for example,
  "I don't have sufficient information to answer this" - rather than assert
  unsupported facts.
- Every material claim SHOULD be supported by cited evidence, and unsupported
  claims MUST be detectable and reported.

Rationale: Telecom engineers act on these answers during live incidents.
Uncited or unsupported output creates operational risk that no amount of model
quality compensates for.

### V. Telecom Domain Fidelity

Telecom concepts MUST be modeled explicitly rather than flattened into generic
documents.

Rules:

- The domain model MUST represent, at minimum: network, 5G, 4G/LTE, VoLTE,
  fiber, broadband, IoT, products, services, SLAs, incidents, support tickets,
  runbooks, postmortems, regions, departments, users, roles, access policies.
- Telecom metadata MUST be preserved during ingestion whenever available,
  including technology, region, service, product, incident severity, document
  type, department, and classification.
- Domain concepts MUST be explicit relationships and explicit value objects,
  not free-text annotations.

Rationale: Retrieval quality depends on structured filters. Generic document
handling discards the metadata that makes telecom knowledge queryable by region,
technology, product, and severity.

### VI. Layered Architecture and Dependency Direction

The architecture MUST preserve strict dependency direction. The dependency
graph, not the file count, is the governing constraint.

Rules:

- Dependencies MUST flow inward only: API depends on Application, Application
  depends on Domain, and Domain depends on nothing.
- Infrastructure and AI/RAG modules (ingestion, retrieval, query, generation)
  MUST depend on Domain and Application abstractions, never the reverse.
- Domain code MUST NOT import PostgreSQL, FastAPI, LangChain, LangGraph,
  OpenRouter, or a specific vector database.
- Domain concepts MUST be separated from database models and API schemas
  wherever their responsibilities differ.
- Application services MUST NOT accumulate unrelated responsibilities or import
  infrastructure directly.
- Business logic MUST NOT create global configuration objects.
- Every directory in the canonical layout below MUST exist only once its stage
  is reached, per Principle VIII.

Rationale: Inward-only dependency direction is what keeps the domain portable
and the system incrementally buildable. It is the property that allows
providers, storage engines, and frameworks to be replaced without rewriting
business logic.

### VII. Provider and Framework Independence

The project MUST avoid unnecessary vendor lock-in.

Rules:

- LLM access MUST be isolated behind an application/infrastructure interface;
  application code MUST depend on `LLMProvider`, never on a vendor client type.
- The same rule applies to embedding models, rerankers, vector storage, and
  observability providers.
- Provider-specific code MUST reside in infrastructure or adapter layers.
- The initial LLM integration is OpenRouter.
- Framework adoption (LangChain, LangGraph, Ragas, OpenTelemetry, Prometheus,
  React, TypeScript) MUST solve a demonstrated problem and MUST remain an
  implementation detail behind application boundaries where practical. Adoption
  MUST NOT be justified by popularity alone.

Rationale: Framework churn and provider pricing changes are routine. Coupling
the domain to either forces rewrites rather than configuration changes.

### VIII. Incremental Complexity

The project MUST evolve incrementally, and each stage MUST preserve the
boundaries established by earlier stages.

Rules:

- Evolution SHALL follow approximately: basic RAG, hybrid retrieval,
  reranking, query understanding, security, evaluation, observability,
  production API, agentic RAG, production hardening.
- Agentic capabilities MUST NOT be introduced before the core RAG pipeline is
  measurable and reliable.
- The project MUST NOT begin with a complex multi-agent architecture.
- Premature optimization MUST NOT compromise architectural clarity;
  optimization MUST be based on observed bottlenecks.
- The canonical repository layout is the target architecture, not a mandate to
  create every directory on day one.

Rationale: The system is both production-oriented and a learning project. Wide
stages are cheaper to correct than wide foundations.

### IX. Ingestion Provenance and Chunking Discipline

Ingestion quality determines retrieval quality. The pipeline MUST be
deterministic, observable, and provenance-preserving.

Rules:

- The pipeline SHALL conceptually follow: source, load, parse, clean,
  normalize, metadata extraction, chunk, validate, embed, index.
- Document provenance MUST be preserved. Every chunk SHOULD be traceable to its
  source document, document version, page or section, and the ingestion
  operation that created it.
- Documents MUST have content hashes or equivalent identifiers to support
  idempotent ingestion.
- Cleaning MUST be conservative. Content MUST NOT be silently altered or lost.
- Security metadata MUST exist and be validated BEFORE indexing. Unsafe or
  unclassified documents MUST be quarantined rather than entering the
  searchable corpus.
- Chunking is an engineering decision, not a fixed utility call. Chunk size,
  overlap, semantic boundaries, and structure awareness MUST be evaluated with
  retrieval metrics.
- No chunking strategy may be considered correct solely because it works on a
  small sample.

Rationale: Indexing unclassified or unprovenanced content is unrecoverable at
scale - it silently degrades both authorization and auditability.

### X. Configuration and Secret Management

Configuration MUST be externalized, strongly typed, and deterministic.

Rules:

- All configuration MUST be represented by typed Pydantic models and validated
  at startup. Invalid configuration MUST prevent the application from starting.
- Precedence MUST be deterministic: defaults, base YAML, environment profile,
  environment variables, runtime overrides.
- Environment-specific settings MUST NOT be hard-coded in application logic,
  and application code MUST NOT read environment variables or config files
  directly.
- The `dev`, `test`, and `prod` profiles MUST be supported.
- Secrets MUST NEVER be committed, logged, or included in error messages, and
  production MUST NOT rely on development `.env` files.
- `.env.example` MAY document required variables without real credentials.
- Feature flags MUST NOT bypass security controls; security cannot be disabled
  through ordinary feature flags in production.
- Loaded configuration SHOULD be treated as immutable.

Rationale: Fail-fast, typed configuration is the cheapest defense against
environment-specific security failures, and separating secrets from
source-controlled YAML is a prerequisite for reviewable configuration change.

### XI. Evaluation-Driven Development

RAG quality MUST be measurable, and improvement claims MUST be evidence-backed.

Rules:

- The project MUST maintain a versioned evaluation dataset containing at
  minimum: question, expected intent, expected sources, expected answer,
  difficulty, domain.
- Evaluation MUST cover retrieval and generation separately. Coupled evaluation
  makes it impossible to attribute a regression to a layer.
- Changes to chunking, embeddings, retrieval, reranking, prompts, or LLMs MUST
  be evaluated against a consistent benchmark before being accepted.
- A change MUST NOT be declared an improvement because a few manually tested
  examples look better.
- Evaluation thresholds MUST be version controlled.
- Experiments MUST document hypothesis, configuration, dataset, method,
  results, conclusion, and decision.

Rationale: RAG behavior is statistical. Without a fixed benchmark, every change
is a coin flip and regressions ship silently.

### XII. Observability

The system MUST be observable end to end.

Rules:

- Observability MUST cover at minimum: request, query, retrieval, reranking,
  context, LLM invocation, generation, response, errors, latency, and token
  usage.
- The project SHALL use structured logging with correlation identifiers
  propagated across request, query, retrieval, and LLM boundaries.
- LLM and embedding usage MUST be measurable: input tokens, output tokens,
  embedding calls, LLM calls, estimated cost.
- Performance MUST be measured rather than assumed, tracking ingestion,
  retrieval, reranking, LLM, and end-to-end latency plus throughput.
- Observability MUST NOT expose secrets or unnecessary enterprise content.
- The architecture SHOULD support cheaper models for appropriate tasks, such
  as classification and query rewriting.

Rationale: Retrieval and generation failures are silent by default. Without
traces and cost telemetry, debugging a bad answer means guessing which layer
degraded.

### XIII. Testing and Quality Gates

Testing is mandatory, and security-sensitive behavior MUST have automated
regression tests.

Rules:

- Unit tests MUST cover domain logic, chunking, metadata extraction, retrieval
  logic, security policies, prompt construction, and response parsing.
- Integration tests MUST cover PostgreSQL, pgvector, ingestion, retrieval, API,
  and authentication/authorization.
- Contract tests MUST cover every external provider adapter.
- Evaluation tests MUST cover retrieval quality, answer quality, citation
  quality, and grounding.
- End-to-end tests MUST cover the API flow.
- Tests MUST construct deterministic configuration and MUST NOT depend on
  developer machine configuration.
- External dependencies MUST be exercised through fakes in tests, never live
  calls.

Rationale: Security and grounding regressions are invisible in manual review.
Automating them is the only mechanism that keeps the non-negotiable principles
true over time.

### XIV. Synthetic Data First

Development MUST initially use synthetic telecom data.

Rules:

- Synthetic data SHOULD be realistic: network documentation, runbooks,
  incidents, support tickets, products, SLAs, postmortems.
- Synthetic data MUST contain realistic metadata and relationships.
- The project MUST NOT require proprietary telecom data to demonstrate its core
  capabilities.

Rationale: Reproducible development and testable evaluation require data that
is permissively licensed and regenerable, which proprietary data cannot be.

### XV. Human Oversight and Agentic Boundaries

The system assists telecom employees; it does not replace operational authority.

Rules:

- Agentic functionality SHALL be introduced only after the core platform has
  reliable retrieval, security enforcement, evaluation, observability, and
  grounded generation.
- Agents MUST use explicit tools with explicit action boundaries, and agentic
  workflows MUST clearly identify actions requiring human authorization.
- The system MUST NOT autonomously modify network configuration, disable
  services, change customer subscriptions, alter production infrastructure, or
  execute destructive operations.
- Memory MUST be treated as a separate architectural concern, distinguishing
  conversation state, long-term knowledge, operational state, and agent memory.
  The enterprise knowledge base MUST NOT be automatically treated as
  conversational memory.
- High-impact operational decisions SHOULD remain subject to human review.

Rationale: The blast radius of an unaudited autonomous action in a telecom
network is orders of magnitude larger than the cost of a human approval step.

## Architecture Constraints and Technology Stack

### Required stack

The initial technology stack SHALL be:

```text
Python 3.12+
uv
Pydantic
FastAPI
PostgreSQL
pgvector
SQLAlchemy
Alembic
OpenRouter
pytest
Docker
```

The following MAY be introduced when justified by the corresponding
implementation phase: LangChain, LangGraph, Ragas, OpenTelemetry, Prometheus,
React, TypeScript.

### Required boundaries

The primary package boundaries MUST be:

```text
domain/
ingestion/
retrieval/
query/
generation/
security/
evaluation/
observability/
agents/
infrastructure/
api/
```

Plus `config/` for externalized configuration and `main.py` as the composition
root. The composition root loads configuration once and wires components;
business logic MUST NOT construct configuration objects.

### Canonical repository layout

This is the target architecture. Directories MUST be created incrementally as
their stage is reached, not all at once.

```text
telco-rag/
├── README.md
├── pyproject.toml
├── uv.lock
├── .env.example
├── .gitignore
├── docker-compose.yml
├── Makefile
├── alembic.ini
├── config/
│   ├── settings.yaml
│   ├── ingestion.yaml
│   ├── retrieval.yaml
│   ├── evaluation.yaml
│   ├── security.yaml
│   ├── observability.yaml
│   └── profiles/
│       ├── dev.yaml
│       ├── test.yaml
│       └── prod.yaml
├── data/
│   ├── raw/{network,products,support,incidents,runbooks,sla}/
│   ├── processed/
│   └── eval/
│       ├── questions.jsonl
│       ├── retrieval_cases.jsonl
│       └── expected_answers.jsonl
├── docs/
│   ├── architecture/{overview,ingestion,retrieval,security,agentic-rag}.md
│   ├── domain/{telco-domain,personas,use-cases}.md
│   ├── experiments/
│   │   ├── 01_baseline-rag.md
│   │   ├── 02_chunking.md
│   │   ├── 03_hybrid-retrieval.md
│   │   ├── 04_reranking.md
│   │   └── 05_query-routing.md
│   └── decisions/
│       ├── ADR-001-database.md
│       ├── ADR-002-vector-search.md
│       └── ADR-003-llm-provider.md
├── prompts/
│   ├── query/{classify,rewrite}.jinja
│   ├── generation/{answer_v1,answer_v2}.jinja
│   └── agents/investigation.jinja
├── scripts/
│   ├── generate_data.py
│   ├── ingest_documents.py
│   ├── rebuild_index.py
│   └── run_evaluation.py
├── migrations/
│   ├── env.py
│   ├── script.py.mako
│   └── versions/
├── src/
│   └── telco_rag/
│       ├── main.py
│       ├── api/{app,dependencies}.py + routes/{health,query,documents,ingestion,incidents}.py
│       ├── application/{query,documents,ingestion,incidents,evaluation,errors,dependencies}.py
│       ├── domain/{documents,chunks,users,access,incidents,tickets,products}.py
│       ├── ingestion/{pipeline,cleaning,chunking,metadata,embedding}.py + loaders/
│       ├── retrieval/{vector,keyword,hybrid,reranker,filters,retriever}.py
│       ├── query/{classifier,rewriter,router,models}.py
│       ├── generation/{generator,prompts,citations,grounding}.py
│       ├── security/{authentication,authorization,acl,filters}.py
│       ├── evaluation/{datasets,retrieval,generation,runner}.py
│       ├── observability/{logging,tracing,metrics}.py
│       ├── infrastructure/
│       │   ├── database.py + repositories/
│       │   ├── llm/{client,openrouter}.py
│       │   ├── embeddings/{client,openrouter}.py
│       │   └── reranking/{client,openrouter}.py
│       └── config/{models,loader,settings,validation}.py
├── tests/
│   ├── unit/{domain,application,ingestion,retrieval,query,generation,security,config}/
│   ├── integration/{test_database,test_ingestion,test_retrieval,test_rag,test_security}.py
│   ├── contract/{test_llm_provider,test_embedding_provider,test_reranker_provider}.py
│   ├── evaluation/{test_retrieval_quality,test_generation_quality}.py
│   └── e2e/test_api_flow.py
└── frontend/
    └── src/{components,pages,api,types}/
```

Layout rules:

- Application code MUST NOT be placed directly in the repository root.
- Tests MUST remain outside `src`.
- The Python package MUST live at `src/telco_rag/`.
- Secrets and non-secret configuration MUST NOT share a file.
- Prompts MUST live under `prompts/` as versioned template files, not as
  inline string literals in Python modules. This makes prompt versioning and
  regression evaluation auditable.
- Database schema changes MUST be managed through Alembic migrations under
  `migrations/`. Manually modified schemas MUST NOT be relied upon.
- The `frontend/` directory is optional and MUST NOT be created until there is
  a demonstrated UI requirement.

### API and reliability constraints

- The service API SHALL use FastAPI with versioned endpoints under `/api/v1/`.
- API contracts SHALL use Pydantic models.
- The API SHOULD provide consistent validation, error responses, structured
  logging, request identifiers, authentication, and authorization.
- API design MUST NOT expose internal infrastructure details unnecessarily.
- The system MUST handle LLM timeouts, embedding failures, database failures,
  transient network failures, malformed documents, malformed model responses,
  and rate limits.
- The system SHOULD implement timeouts, retries, exponential backoff, circuit
  breakers where appropriate, and graceful failure.
- Retries MUST NOT create uncontrolled duplicate operations; write operations
  MUST be idempotent.

## Development Workflow and Quality Gates

### Feature workflow

Every significant feature SHOULD follow:

```text
Specification
    -> Design
    -> Implementation Plan
    -> Implementation
    -> Unit Tests
    -> Integration Tests
    -> Evaluation
    -> Documentation
```

Features MUST NOT be implemented solely by modifying code without updating the
relevant specification when behavior changes.

### Documentation requirements

- Important architectural decisions SHALL be documented as ADRs under
  `docs/decisions/`.
- Experiment results MUST be recorded under `docs/experiments/` following the
  template in Principle XI, and are particularly important for chunking,
  embeddings, retrieval, reranking, prompts, and LLM selection.
- Deep-dive architecture documents MUST live under `docs/architecture/` and
  domain material under `docs/domain/`.

### Definition of done

A feature is complete only when all of the following hold:

- The implementation exists.
- Relevant tests exist.
- Error behavior is handled.
- Security implications are considered.
- Observability is added where appropriate.
- Documentation is updated.
- Evaluation is performed when the feature affects RAG quality.
- The code passes the project quality checks.

For RAG changes, "it works on one example" is NOT sufficient evidence of
completion.

### Compliance

- All feature specifications, implementation plans, and architectural decisions
  MUST be evaluated against this constitution.
- When a proposed implementation conflicts with this constitution, the conflict
  MUST be explicitly identified.
- Exceptions require all of: a documented rationale, identification of the
  affected principle, an explanation of the trade-off, an explicit
  architectural decision, and an update to this constitution when the principle
  is permanently changed.

## Governance

This constitution supersedes all other development practices in the repository.
Where a specification, plan, or implementation conflicts with it, this document
governs.

### Amendment procedure

1. Propose the change with a written rationale identifying every affected
   principle.
2. Record any permanent principle change as an ADR under `docs/decisions/`.
3. Update the Version and Last Amended fields below.
4. Note the version bump type in the change description.

### Versioning policy

`CONSTITUTION_VERSION` follows semantic versioning, evaluated against the
governance content rather than the code:

- MAJOR: backward-incompatible governance change - a principle is removed or
  redefined in a way that invalidates previously compliant work.
- MINOR: a principle or section is added, or existing guidance is materially
  expanded.
- PATCH: clarifications, wording, typo fixes, and non-semantic refinements.

If the bump type is ambiguous, the rationale MUST be stated before finalizing.

### Compliance review

- Every review MUST verify constitution compliance for the change under review.
- Added complexity MUST be justified against a principle, not against
  preference.
- Runtime development guidance, where present, MUST NOT contradict this
  constitution; where it does, this constitution governs.

### Precedence

1. This constitution.
2. ADRs under `docs/decisions/`.
3. Specifications and implementation plans under `docs/`.
4. Code, tests, and operational tooling.

**Version**: 1.0.0 | **Ratified**: 2026-10-05 | **Last Amended**: 2026-10-05