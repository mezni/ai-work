# Implementation Plan: Repository Foundation (Stage 0)

**Branch**: `001-repository-foundation` | **Date**: 2026-10-05 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/001-repository-foundation/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command; its definition describes the execution workflow.

## Summary

Stage 0 establishes a clean, reproducible Python project foundation so that a
contributor can, from a fresh checkout and one documented command, obtain a
passing, meaningful verification result. The foundation composes:

- a package manifest that pins a minimum runtime and a reproducible dependency
  set (dev tooling only at this stage),
- a single verification command (`make verify`) running the automated tests,
  the static-inspection gate, and the type-analysis gate,
- the minimal repository surface defined by the target structure, with all
  later-stage directories deliberately absent,
- secret-exclusion and environment-template hygiene.

The primary delivery, per `docs/implementation-plan.md` §4, is "a project that
can execute verification with a clean test result" — now expressed as
`make verify` (see [research.md](research.md) D1).

## Technical Context

**Language/Version**: Python 3.12+; local pin `3.12` in `.python-version`
(constrained by `docs/constitution.md` §32 and enforced by FR-007).

**Primary Dependencies**: `pytest`, `Ruff`, `mypy` — declared as a dev-only
dependency group and pinned by a committed lockfile. Runtime dependencies
(`Pydantic`, `SQLAlchemy`, `Alembic`) are intentionally NOT declared at this
stage; see [research.md](research.md) D3 for the recorded deviation from
`docs/implementation-plan.md` §4.

**Storage**: N/A at this stage. No persistence. `docker-compose.yml` is
provisioned as a placeholder but not exercised (spec Assumption).

**Testing**: `pytest` under `uv run`, configured via `pyproject.toml`
(`testpaths`, non-empty suite per FR-003).

**Target Platform**: Linux and macOS development environments; the CI target
machines (linux/x86_64) that the committed lockfile must reproduce.

**Project Type**: Python application package (src layout), `src/telco_rag/`,
with package markers only at this stage.

**Performance Goals**: Verification completes in under 60 seconds once the
environment is prepared (SC-002); cold checkout-to-pass in under 10 minutes
(SC-001).

**Constraints**: Single documented verification command (FR-001); failing
result on any check failure (FR-002); exactly one authoritative structure
document (FR-013); no later-stage directories (FR-012); secrets excluded by
default with a committed template (FR-015); quality-gate configuration
version-controlled (FR-010).

**Scale/Scope**: Minimal — 1 package marker, 1 test file, 6 root metadata
artifacts, 3 hygiene directories, 3 tool configurations. No application code.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Gates derived from `.specify/memory/constitution.md` v1.0.0:

| Gate | Constitution basis | Stage 0 evidence | Verdict |
| --- | --- | --- | --- |
| G1 Dependency direction | Principle VI | Only `src/telco_rag/__init__.py` created; no API/Application/Domain modules exist yet, so no dependencies to invert | PASS |
| G2 Incremental complexity | Principle VIII | FR-012 keeps later-stage dirs absent; only Stage 0 surface created | PASS |
| G3 Configuration & secrets | Principle X | `.env.example` is an empty-value template; `.gitignore` excludes `.env`, caches; no secrets in `config/` (empty) | PASS |
| G4 Testing | Principle XIII | pytest foundation with ≥1 meaningful test (FR-003/FR-004); dev tooling pinned in lockfile | PASS |
| G5 Repository layout | §10 Python Architecture, Overview §5 | Application code under `src/telco_rag/`; tests outside `src`; `uv` with committed `uv.lock` | PASS |
| G6 Technology stack | §32 Technology Principles; §9 Provider independence | Stack complies; runtime deps deferred to their stage (documented deviation, D3) | PASS (with recorded deviation) |
| G7 Docs retention | §Documentation | `docs/` (22 documents) retained and reachable from README (FR-016) | PASS |

No gate violations. Complexity Tracking table omitted.

> **Post-Phase 1 re-check**: After generating `research.md`, `data-model.md`,
> `contracts/`, and `quickstart.md`, no new constitution obligations were
> introduced and no gate regressed. Status unchanged: **G1–G7 PASS**.

## Project Structure

### Documentation (this feature)

```text
specs/001-repository-foundation/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
├── checklists/          # Requirements quality checklist (/speckit.specify output)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
telco-rag/
├── pyproject.toml           # Manifest: name, requires-python, [dependency-groups].dev
├── uv.lock                  # Committed reproducibility contract (output of uv)
├── .python-version          # Local runtime pin: 3.12
├── README.md                # FR-014 entry point (update verification command)
├── .env.example             # FR-015 template, no real credentials
├── .gitignore               # .env, .venv, __pycache__, *.pyc, .pytest_cache,
│                            #   .mypy_cache, .ruff_cache
├── Makefile                 # `make verify` (FR-001); `make env` convenience
├── docker-compose.yml       # Placeholder service definitions; not exercised
├── config/.gitkeep          # Created empty; YAML arrives Stage 2
├── data/.gitkeep            # Created empty; raw/eval arrive Stage 4
├── scripts/.gitkeep         # Created empty; scripts arrive Stage 4
├── docs/                    # Existing (22 documents) — retained, indexed in README
├── .specify/                # Existing governance — retained
├── src/
│   └── telco_rag/
│       └── __init__.py      # Package marker + __version__ (single source)
└── tests/
    └── test_smoke.py        # ≥1 meaningful test (FR-003); tested by `make verify`
```

Deliberately absent at this stage (per FR-012 and `docs/architecture/overview.md`
§8): `prompts/`, `migrations/`, `frontend/`, `alembic.ini`, and every
`src/telco_rag/` module beyond the package marker (`api/`, `application/`,
`domain/`, `ingestion/`, `retrieval/`, `query/`, `generation/`, `security/`,
`evaluation/`, `observability/`, `agents/`, `infrastructure/`, `config/`).

**Structure Decision**: Single Python application package under `src/`
(`src/telco_rag/`) per Constitution §10 and the target structure; tests outside
`src`. Stage 0 materializes only the surface that verification exercises plus
the governance-mandated root artifacts. The full tiered `tests/unit/...`
skeleton from `docs/testing.md` §4 is deferred until each tier's first test
lands, avoiding speculative empty directories (research D5).

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

No constitution violations. Table omitted.