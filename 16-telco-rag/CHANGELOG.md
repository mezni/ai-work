# Changelog

All notable changes to `telco-rag` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec.php#pec-2.0.0).

## Version History

| Version | Date | Feature Domain | Key Objectives |
|---------|------|----------------|-----------------|
| 0.1.0 | 2026-10-05 | Specification | Complete specification set, ratified constitution, target project structure |
| 0.1-pre | 2026-08-16 | Repository | Cleanup |
| 0.0.1 | 2026-08-16 | Project | Scaffold |

> **Version reconciliation (2026-10-05):** `pyproject.toml` declared `0.1.0`
> while this file recorded only `0.0.1` and the non-semver label `0.1-pre`.
> `0.1.0` is adopted as the real version for the specification milestone.
> `0.1-pre` is retained above as historical record but should not be reused;
> pre-release identifiers belong on a `0.1.0-rc.N` form.
>
> **Correction to `0.0.1`:** that entry claims `.env.example` and `.gitignore`
> were created. Neither file exists in the repository. The claim is left in
> place rather than rewritten, per Keep a Changelog immutability, but it is
> inaccurate. Both files remain outstanding Stage 0 work.

## [0.1.0] - 2026-10-05

Specification-first release. The repository contains a complete, ratified
design and no application code.

### Added

- **Constitution (v1.0.0):** ratified `.specify/memory/constitution.md` from
  `docs/constitution.md`, replacing an unresolved scaffold of 14 placeholder
  tokens with 15 named principles, architecture constraints, development
  workflow and quality gates, and governance sections with amendment procedure,
  semantic versioning policy, and compliance review. Ratified 2026-10-05.
- **Product specification:** `product.md` (vision, users, knowledge domains,
  capabilities) and `prd.md` (user stories `US-001`–`US-011`, functional
  requirements `FR-001`–`FR-030`).
- **Architecture:** `architecture.md` (layered design) and
  `architecture/overview.md` (target repository structure, dependency
  boundaries, agentic extension, stage-to-directory mapping).
- **Domain and data model:** `domain.md` (entities, aggregates, value objects,
  invariants, lifecycles) and `data-model.md` (tables, columns, constraints,
  indexes, relationships).
- **Cross-cutting designs:** `configuration.md`, `providers.md`.
- **Subsystem designs:** `ingestion.md`, `retrieval.md`, `query.md`,
  `generation.md`, `grounding.md`, `security.md`, `security-llm.md`,
  `evaluation.md`, `observability.md`, `api.md`, `testing.md`.
- **Implementation plan:** `implementation-plan.md`, staged construction order
  from repository foundation through agentic RAG.
- **README:** rewritten from the `# Project Name` placeholder to describe the
  platform, architecture, principles, documentation index, and an explicit
  statement of current status.

Total: 21 specification documents plus `docs/architecture/overview.md`
(37,763 lines added in commit `8b95781`).

### Changed

- `docs/constitution.md` is now the source document for the Spec Kit
  constitution at `.specify/memory/constitution.md`.
- Documentation layout is now governed: `docs/` is specified to hold
  `architecture/`, `domain/`, `experiments/`, and `decisions/` subdirectories in
  addition to the flat specification files.

### Known Issues

- **No application code.** `src/` and `tests/` are empty and `pyproject.toml`
  declares `dependencies = []`. `uv run pytest` does not run.
- **Repository scaffold incomplete.** `uv.lock`, `.gitignore`, `.env.example`,
  `Makefile`, `docker-compose.yml`, `alembic.ini`, and `config/` do not exist,
  despite all being required by the target structure.
- **`docs/architecture.md` and `docs/architecture/` coexist.** The target
  structure requires the directory while the flat specification document
  retains the base name. Any `docs/architecture*` glob is ambiguous.
- **`prompts/evaluation/` missing from the target structure** but required by
  `evaluation.md` §48 for judge prompt versioning.
- **`docs/implementation-plan.md` §4 Stage 0 carries a partial repository
  tree** that omits `uv.lock`, `alembic.ini`, `prompts/`, `migrations/`, and
  `frontend/`, making it a second, weaker source of truth for structure.
- **`security.md` and `security-llm.md` overlap.** Both declare the title
  `# Security Architecture` and cover overlapping ground. A split and an
  explicit boundary between platform security and LLM-boundary security is
  still required.

### Removed

- **Prototype implementation (2026-09-14/15, removed 2026-09-16).** An
  exploratory codebase was developed and then deleted in commit `da5606c`
  ("Cleanup repo", 9,006 deletions across 86 files, including a 3,268-line
  `uv.lock`). It had explored domain entities (`Document`, `Chunk`, document
  lifecycle and chunk lineage), a stage-based ingestion pipeline, a retrieval
  pipeline with a Chroma backing store, an evaluation pipeline, SQLAlchemy
  repositories, an Alembic initial schema, and LlamaIndex-based parsers,
  chunking, and embeddings. It was removed in favour of the
  specification-first approach and a provider-independent architecture. None of
  it was ever recorded in this changelog.

## [0.1-pre] - 2026-08-16

### Added
- **Cleanup repo**

## [0.0.1] - 2026-08-16

### Added
- **Project scaffold:** `pyproject.toml`, `.env.example`, `.gitignore`,
  `README.md`, test directory structure

  > Inaccurate: `.env.example` and `.gitignore` are not present in the
  > repository. See the correction note above.

[Unreleased]: https://example.invalid/telco-rag/compare/v0.1.0...HEAD
[0.1.0]: https://example.invalid/telco-rag/compare/v0.1-pre...v0.1.0
[0.1-pre]: https://example.invalid/telco-rag/compare/v0.0.1...v0.1-pre
[0.0.1]: https://example.invalid/telco-rag/releases/tag/v0.0.1