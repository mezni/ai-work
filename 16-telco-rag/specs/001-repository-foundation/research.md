# Research: Repository Foundation (Stage 0)

**Feature**: [spec.md](spec.md) | **Date**: 2026-10-05 | **Plan**: [plan.md](plan.md)

This document resolves every design unknown raised by the specification (the
spec deliberately contains no `[NEEDS CLARIFICATION]` markers, so the unknowns
below are the technical choices that flow from its 16 functional requirements).
Each entry records a decision, its rationale, and the alternatives rejected.

Sources consulted: `.specify/memory/constitution.md` v1.0.0,
`docs/implementation-plan.md` §4, `docs/architecture/overview.md` §5/§8,
`docs/configuration.md` §9/§26, `docs/testing.md` §4, and current `uv`
documentation and community guidance (astral-sh, pydevtools.com, 2026).

---

## D1 — The single verification command is `make verify`

**Decision**: The documented one-command entry point (`FR-001`) is
`make verify`, defined in `Makefile`. It runs, sequentially and short-circuiting:

```text
uv run pytest
uv run ruff check .
uv run mypy src
```

`make` is a mandated Stage 0 artifact (`docs/implementation-plan.md` §4) and is
available on every target development platform. `uv run` guarantees each tool
runs inside the locked project environment, so rules and versions come from the
committed manifest and lockfile.

**Rationale**: `FR-001` requires *all* automated verification under one command;
`FR-009` requires the static-inspection and type-analysis gates to be runnable
from that entry point; `FR-002` requires a failing result on any check failure.
`uv run pytest` alone (the literal Stage 0 deliverable text) satisfies none of
the gates. `make verify` composes the three with `&&` semantics: any failure
stops the run and yields a non-zero exit. `pytest` runs first because it is the
fastest signal.

**Alternatives considered**:
- *`uv run pytest` as the documented command* — rejected: fails FR-009 (gates
  not run from entry point) and FR-002 (passes even when lint/type gates fail).
- *A custom `scripts/verify.sh`* — rejected: `scripts/` is a later-stage
  directory (Stage 4) per overview §8; introducing it now violates FR-012.
- *`pre-commit` hooks* — rejected: hooks catch defects at commit time, not as a
  single verification gate; `FR-011` (gates must fail on defect) needs a runnable
  command, and CI integration is explicitly out of scope (assumption 7).

**Evidence**: SC-001 (cold clone → pass < 10 min), SC-002 (run < 60 s).

---

## D2 — Tool versions are locked, runtime pin is `3.12`

**Decision**: `requires-python = ">=3.12"` in `pyproject.toml`; `.python-version`
records `3.12`; all dev tool versions resolved into a committed `uv.lock`.

**Rationale**: `FR-005` (explicit manifest) and `FR-006` (reproducible) are
satisfied by a committed lockfile; `uv sync` from the lockfile is deterministic
(SC-004, 100% identical across machines). `.python-version` lets `uv` auto-select
the interpreter so a contributor without Python installed still converges on the
correct runtime (FR-008). `FR-007` is satisfied by `requires-python`: `uv`
refuses to resolve against an older interpreter and surfaces an actionable
message.

**Alternatives considered**:
- *Use whatever Python is present* — rejected: violates FR-007, produces
  SC-007 failures.
- *Declare `>=3.12,<3.13`* — rejected: unnecessary tightening; 3.13/3.14 are
  future candidates and the lockfile already pins behavior. Keep the minimum
  guard, not an artificial ceiling.

---

## D3 — Dev-only tooling now; runtime deps deferred (recorded deviation)

**Decision**: At Stage 0 the only declared requirements are the dev tools:
`pytest`, `ruff`, `mypy`, declared under `[dependency-groups].dev` (PEP 735,
`uv` native). `Pydantic`, `SQLAlchemy`, and `Alembic` are **not** declared here;
each is added by the stage that first exercises it (Stages 1/2/3 respectively).

**Deviation**: `docs/implementation-plan.md` §4 Stage 0 lists Pydantic,
SQLAlchemy, and Alembic among items to "configure". This plan deliberately
defers them. Recorded here so the divergence is explicit rather than silent.

**Rationale**: The constitution requires dependencies to be "explicitly
declared" (§Technology) — declaring a dependency that nothing imports is not
explicitness, it is speculative payload (and contradicts Principle VIII —
Incremental Complexity). Stage 0's verification exercises precisely three tools;
a lockfile containing only them is small, stable, and genuinely reproducible
(SC-004). Alembic's `migrations/` is a Stage 3 directory per overview §8, so
configuring Alembic at Stage 0 would also create a `migrations/` tree that
FR-012 forbids.

**Alternatives considered**:
- *Follow impl-plan §4 literally* — rejected for the reasons above; the plan
  text is a stage summary, and overview §8 (the authoritative target structure
  per FR-013) places `migrations/` at Stage 3.
- *Declare all tooling as project dependencies* — rejected: `[dependency-groups]`
  keeps tools out of the runtime dependency set, matching how downstream installs
  would treat the package (confirmed by current `uv` guidance).

---

## D4 — Environment template and secret exclusion

**Decision**: `.env.example` is committed and contains only documented variable
names with empty values — the four named in `docs/configuration.md` §9
(`DATABASE_URL`, `OPENROUTER_API_KEY`, `ENVIRONMENT`, `LOG_LEVEL`) plus comments
explaining each. `.gitignore` excludes `.env` (any environment),
`.venv/`, `__pycache__/`, `*.py[cod]`, `.pytest_cache/`, `.mypy_cache/`,
`.ruff_cache/`, and build artifacts.

**Rationale**: FR-015 requires secrets excluded by default with a committed
template that documents required variables; Principle X forbids secrets in
version control. The four variable names come from configuration.md §9, keeping
`.env.example` and the configuration design aligned rather than introducing a
parallel vocabulary.

**Alternatives considered**:
- *Commit a real `.env` with dummy values* — rejected: invites rotation of a
  credential-shaped value and contradicts the constitution's secret handling.
- *A separate `.env.example` per environment* — rejected: over-engineering; one
  template plus profiles (Stage 2) covers it.

---

## D5 — No speculative directories; empty Stage 0 dirs use `.gitkeep`

**Decision**: Git cannot track empty directories, so the three Stage 0
directories with no content yet (`config/`, `data/`, `scripts/`) each carry a
single `.gitkeep`. Every directory whose stage is unreached
(`prompts/`, `migrations/`, `frontend/`, `agents/`, and all `src/telco_rag/*`
modules) is **absent**, per FR-012. The full tiered `tests/unit/...` skeleton
from `docs/testing.md` §4 is **not** pre-created; the tier directories are added
when each tier's first test lands.

**Rationale**: FR-012 is explicit — unreached-stage directories must be absent,
not empty placeholders. Pre-creating 13 empty test directories or module
packages would look "complete" while signaling nothing real, misleading exactly
the contributor US4 targets. `config/`, `data/`, `scripts/` are Stage 0
directories by impl-plan §4, so they exist — with `.gitkeep` as the git-visible
sentry.

**Alternatives considered**:
- *Pre-create all target structure directories* — rejected: directly violates
  FR-012 and the overview §8 mandate.
- *Document the empty dirs via comments in `.gitignore` without `.gitkeep`* —
  rejected: `.gitkeep` preserves emptiness server-side and is self-describing.
- *Create tests/unit + tests/integration now* — rejected: no Stage 0 tests belong
  in either tier; smoke validation sits at `tests/` root.

---

## D6 — The first test asserts the foundation, not a stub

**Decision**: One test module, `tests/test_smoke.py`, containing meaningful
assertions:

1. `test_manifest_declares_project()` — `pyproject.toml` parses and declares the
   package `telco-rag` with `requires-python >= 3.12`.
2. `test_package_imports_and_version()` — `import telco_rag` succeeds and
   `telco_rag.__version__` equals `0.1.0` (importlib.metadata-sourced), and the
   package root resolves inside `src/`.
3. `test_verification_has_content()` — the collected suite is non-empty
   (guards the empty-run regression described in the spec's Edge Cases).

**Rationale**: FR-003 demands a passing result be evidence, not an empty run
(pytest alone with zero tests exits 5 and reports failure — not a clean pass).
These three assertions exercise the actual foundation: the manifest contract,
the package marker under `src/`, and version single-sourcing — each catches a
real, likely regression in Stage 0 tooling. `__version__` is derived from the
distribution metadata so version lives in exactly one place (`pyproject.toml`).

**Alternatives considered**:
- *`tests/unit/test_repository.py` placement* — rejected in favour of a
  `tests/` root smoke test; tier directories are deferred (D5), so an
  orphaned `unit/` with one file is the wrong shape.
- *A trivial `assert True`* — rejected: passes but proves nothing, fails the
  spirit of FR-003.

---

## D7 — Static-inspection and type-analysis gates are Ruff and mypy

**Decision**: Configuration for both gates lives in `pyproject.toml`, version
controlled (FR-010):

- `[tool.ruff]` — `target-version = "py312"`, line length 88;
  `[tool.ruff.lint] select = ["E","F","I","UP","B","SIM"]`.
- `[tool.mypy]` — `python_version = 3.12`, `strict = true`, scope `src/`.

Both run through `uv run` inside the locked environment (see D1), so `uv tool
run`-style configuration loss cannot occur.

**Rationale**: FR-009 names two defect classes — static inspection and type
analysis. Ruff covers the first (E/F/I/UP/B/SIM is the modern baseline set),
mypy the second, and both are named in the plan's assumptions as the
pre-mandated mapping. `strict = true` is justified at zero coupling cost: the
only analyzed module is the package marker.

**Alternatives considered**: `ty` (Astral's beta checker) instead of mypy —
rejected: constitution and impl-plan name mypy; `ty` is pre-1.0. Pre-commit
mechanism — excluded by D1 reasoning (runnable single command is required).

---

## D8 — `docker-compose.yml` is a validating placeholder

**Decision**: A minimal `docker-compose.yml` with a single non-starting service
declaration of the future PostgreSQL backing store (image + healthcheck, no
credentials, no secrets). It validates with `docker compose config` but is not
required for `make verify`.

**Rationale**: The spec assumption is that the file is provisioned at Stage 0
but not exercised until Stage 3. Including a well-formed but inert definition
lets the file be reviewed for hygiene (no secrets) now rather than retrofitted
later.

**Alternatives considered**: Omit `docker-compose.yml` until Stage 3 — rejected:
impl-plan §4 lists it as a Stage 0 artifact; omitting creates a docs-vs-tree
divergence this stage is explicitly charged with eliminating.

---

## D9 — README verification section updated during implementation

**Decision**: The existing `README.md` (written this session) states `uv run
pytest` as the Stage 0 deliverable and a "not started" scaffold status. During
implementation, the README's verification instructions will be updated to the
single `make verify` command and the scaffold-status table will be marked as
having Stage 0 complete. README is the "repository entry point" of FR-014 and
FR-016.

**Rationale**: The spec requires the entry point to describe purpose, status,
and how to run verification. Leaving a stale verification command would
immediately violate FR-014/SC-001 the moment a contributor follows the README.

---

## D10 — No CI/CD, no pre-commit, no formatting ceremony at Stage 0

**Decision**: No CI configuration (Stage 25), no `.pre-commit-config.yaml`, no
`ruff format` target (lint only), and no coverage tool in the Stage 0 `make
verify` composition.

**Rationale**: Each exclusion is anchored in scope: assumption 7 (CI/CD out of
scope), D1 (single verifiable command, not commit hooks), requirements
(no formatting requirement exists in the 16 FRs), and incremental-complexity
(cover value is nil with a one-module package). `ruff format --check` can be
added with the first real code at Stage 1.

## Open items

None. Every decision above has an owner-in-the-plan and an acceptance mapping
to `SC-001`–`SC-010`. No `[NEEDS CLARIFICATION]` remains.