---

description: "Task list for Repository Foundation (Stage 0) feature implementation"
---

# Tasks: Repository Foundation

**Input**: Design documents from `/specs/001-repository-foundation/`

**Prerequisites**: plan.md (required), spec.md (required), research.md, data-model.md, contracts/, quickstart.md

**Tests**: Test tasks appear where the specification requires them (FR-003
mandates a non-empty meaningful suite; every user story is validated through
`quickstart.md` scenarios). TDD-style before-code tests are not applicable to
this feature, so none are generated.

**Organization**: Tasks are grouped by user story; each story is independently
testable from Phase 3 onward.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: can run in parallel (different files, no dependencies)
- **[Story]**: user story phase tasks are labelled US1–US4; Setup, Foundational,
  and Polish phases carry no label
- Exact file paths appear in every task

## Path Conventions

Single Python project: `src/`, `tests/` at repository root (`telco-rag/`).

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization — manifest, toolchain, hygiene files.
Constrained by research D1–D5, D8.

- [ ] T001 Complete the package manifest in `pyproject.toml` with full metadata: `name = "telco-rag"`, `version = "0.1.0"`, `description`, `readme = "README.md"`, `requires-python = ">=3.12"`, empty `dependencies = []`, and a `[build-system]` declaring `hatchling` (hatchling.build backend). Do not touch the run-time dependency list.
- [ ] T002 Create `.python-version` containing exactly `3.12` so `uv` auto-selects the interpreter (research D2).
- [ ] T003 Add the dev toolchain: run `uv add --dev pytest ruff mypy` (writes `[dependency-groups].dev` in `pyproject.toml` and resolves + writes `uv.lock`). Confirm `uv.lock` is created and committed as the reproducibility contract (FR-005/FR-006).
- [ ] T004 [P] Configure pytest in `pyproject.toml` under `[tool.pytest.ini_options]` with `testpaths = ["tests"]` and `addopts = "-q"`.
- [ ] T005 [P] Configure Ruff in `pyproject.toml` under `[tool.ruff]` with `target-version = "py312"`, `line-length = 88`, and `[tool.ruff.lint] select = ["E","F","I","UP","B","SIM"]` (research D7; FR-009).
- [ ] T006 [P] Configure mypy in `pyproject.toml` under `[tool.mypy]` with `python_version = "3.12"`, `strict = true` (analysis scope: `src/`) (research D7).
- [ ] T007 [P] Create `.gitignore` excluding `.env`, all local environment files, `.venv/`, `__pycache__/`, `*.py[cod]`, `.pytest_cache/`, `.mypy_cache/`, `.ruff_cache/`, and build artifacts (FR-015).
- [ ] T008 [P] Create `.env.example` committing only the four variable names from `docs/configuration.md` §9 — `DATABASE_URL`, `OPENROUTER_API_KEY`, `ENVIRONMENT`, `LOG_LEVEL` — with empty values and a comment per variable. No credentials anywhere (FR-015, Principle X).
- [ ] T009 Create `Makefile` with two targets: `env` (`uv sync --group dev`) and `verify` running `uv run pytest` then `uv run ruff check .` then `uv run mypy src` with short-circuit `&&` semantics, exiting non-zero on any failure (FR-001/FR-002; research D1).
- [ ] T010 [P] Create `docker-compose.yml` with a single non-starting `postgres` service declaration (image reference, healthcheck stub, **no** credentials or secrets) that passes `docker compose config`; it is not exercised by verification (spec assumption 6; research D8).
- [ ] T011 [P] Create the three Stage 0 hygiene directories with `.gitkeep` sentries: `config/.gitkeep`, `data/.gitkeep`, `scripts/.gitkeep` (FR-012; research D5).

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: The minimum artifacts every story's validation runs against —
`make verify` must have a real suite and an importable package to report on.

**⚠️ CRITICAL**: No user story checkpoint can pass before this phase is complete.

- [ ] T012 Create `src/telco_rag/__init__.py` package marker exposing `__version__` derived from distribution metadata (`importlib.metadata.version("telco-rag")`) so the version exists only in `pyproject.toml` (research D6).
- [ ] T013 Create `tests/test_smoke.py` with three meaningful tests: (1) `pyproject.toml` parses and declares package `telco-rag` with `requires-python >= 3.12`; (2) `import telco_rag` succeeds and `telco_rag.__version__ == "0.1.0"` and the package resolves inside `src/`; (3) the collected suite is non-empty (the empty-run guard, FR-003; research D6).

**Checkpoint**: `make verify` now runs a real suite; from here only validation
and integration tasks remain.

---

## Phase 3: User Story 1 — Verify a working project from a clean checkout (Priority: P1) 🎯 MVP

**Goal**: A contributor runs one documented command from a fresh clone and gets
a passing, meaningful result.

**Independent Test**: spec acceptance scenarios 1–4 (clean clone → `make verify`
passes; healthy tree exits 0; pre-feature tree cannot run at all; runtime
mismatch yields an actionable message). Runnable steps in quickstart §3.1, §3.3, §3.4, §3.6.

### Implementation for User Story 1

- [ ] T014 [US1] Wire the canonical command end to end: confirm `make verify` on the clean tree runs all three steps, exits 0, and reports the smoke tests passing, Ruff clean, and mypy success in order (quickstart §3.4; FR-001/FR-002/FR-004).
- [ ] T015 [US1] Prove the cold-checkout path: `git clone` this repo into a temporary directory, run only `make env` then `make verify` from the clone with no other setup, and confirm exit 0 within the SC-001 bound of 10 minutes (quickstart §3.1).
- [ ] T016 [US1] Validate the empty-run guard: temporarily remove `tests/test_smoke.py`, run `make verify`, and confirm the command exits non-zero and does **not** report success; restore the file (quickstart §3.3; FR-003).
- [ ] T017 [US1] Update `README.md` so its verification instructions lead with the single `make verify` command (replacing the `uv run pytest` phrasing), and update the Current Status table to mark Stage 0 scaffolding as present (FR-014, FR-016; research D9).

**Checkpoint**: User Story 1 is independently demonstrable via quickstart §3.1/§3.4 — MVP complete.

---

## Phase 4: User Story 2 — Reproducible environment across contributors (Priority: P2)

**Goal**: Two contributors on different machines reach an identical working
environment using only repository instructions.

**Independent Test**: spec acceptance scenarios 1–3 (both prepare from repo
instructions only; every declared requirement satisfied, none implied; results
match across machines). Runnable steps in quickstart §3.2.

### Implementation for User Story 2

- [ ] T018 [US2] Validate two-machine reproducibility: on a second clean machine, run only the documented steps from `README.md` (no manual installs), then compare `uv run pytest --version`, `uv run ruff --version`, and `uv run mypy --version` against machine A — every version must match exactly (quickstart §3.2; FR-006/SC-004).
- [ ] T019 [US2] Add a "Development Setup" section to `README.md` documenting the two prerequisites (`git`, `uv`), the `make env` step, and that Python itself is auto-provisioned — sufficient for a contributor with no prior project knowledge (FR-008/SC-001).

**Checkpoint**: User Story 2 is independently demonstrable via quickstart §3.2.

---

## Phase 5: User Story 3 — Automated quality gates catch defects before review (Priority: P3)

**Goal**: A seeded detectable defect makes the verification run fail with the
offending location identified.

**Independent Test**: spec acceptance scenarios 1–3 (seeded defect fails and
locates; clean tree passes; bypassing the gates still fails verification).
Runnable steps in quickstart §3.5.

### Implementation for User Story 3

- [ ] T020 [US3] Validate the static-inspection gate: seed an unused-import defect in a temporary file under `src/telco_rag/`, run `make verify`, and confirm the run halts at the Ruff step with a failing result naming `file:line`; remove the seed (quickstart §3.5; FR-009/FR-011/SC-005/SC-006).
- [ ] T021 [US3] Validate the type-analysis gate: seed `x: int = "oops"` in a temporary module under `src/telco_rag/`, run `make verify`, and confirm mypy fails naming the location; remove the seed (quickstart §3.5; SC-005/SC-006).
- [ ] T022 [US3] Confirm gate determinism: verify the gates behave identically with no contributor-local Ruff/mypy configuration present (no `pyproject.toml` override files outside the manifest), then re-run `make verify` to green (FR-010/FR-011).

**Checkpoint**: User Story 3 is independently demonstrable via quickstart §3.5.

---

## Phase 6: User Story 4 — Repository structure matches the target architecture (Priority: P4)

**Goal**: The working tree matches the single authoritative layout; a new
contributor knows exactly where future code goes.

**Independent Test**: spec acceptance scenarios 1–3 (docs discoverable; exactly
one authoritative structure document; unreached-stage directories absent).
Runnable audits in quickstart §4 (SC-009/SC-010 rows).

### Implementation for User Story 4

- [ ] T023 [P] [US4] Audit and confirm zero unreached-stage directories: `find . -type d \( -name prompts -o -name migrations -o -name frontend -o -name agents \)` returns nothing, and no `src/telco_rag/{api,application,domain,ingestion,retrieval,query,generation,security,evaluation,observability,infrastructure,config}` module exists (FR-012/SC-010; quickstart §4).
- [ ] T024 [P] [US4] Confirm the single structure authority: `docs/architecture/overview.md` is the only document defining the target layout — verify no remaining divergent root-tree restatement exists (FR-013/SC-009).
- [ ] T025 [P] [US4] Confirm documentation retention: all 22 `docs/` documents are present and reachable from the README documentation index (FR-016).
- [ ] T026 [US4] Update `docs/implementation-plan.md` §4 Stage 0 tree so it defers to `docs/architecture/overview.md` instead of restating a partial root layout, removing the divergent 12-entry listing (FR-013; research D/constitution).

**Checkpoint**: User Story 4 is independently demonstrable via the §4 success-criteria audit table.

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: End-to-end validation, coherence, and governance record.

- [ ] T027 Run the complete `specs/001-repository-foundation/quickstart.md` validation (scenarios §3.1–§3.7) and record results against the SC-001–SC-010 map in a short result report under `docs/experiments/01_repository-foundation.md`.
- [ ] T028 Verify declared-vs-resolved coherence: run `uv lock --check` (must pass) and `uv tree` to confirm only the three declared dev tools plus their transitive dependencies are present — no Pydantic/SQLAlchemy/Alembic (research D3).
- [ ] T029 Verify secret hygiene per quickstart §3.7: create a throwaway local `.env`, confirm `git status --porcelain` does not list it, and `git grep -I -i 'OPENROUTER_API_KEY=.'` returns no populated value (FR-015).
- [ ] T030 Validate runtime-mismatch messaging: temporarily edit `requires-python` to `>=3.999`, run `make env`, and confirm an actionable mismatch message (never an opaque traceback); revert (SC-007; quickstart §3.6).
- [ ] T031 Check version single-sourcing: `python -c "import telco_rag; print(telco_rag.__version__)"`, `pyproject.toml` `version`, and `CHANGELOG.md` 0.1.0 entry all agree (data-model `Derived value`; D6).
- [ ] T032 Update `CHANGELOG.md` with a stage-0 implementation entry under the existing `0.1.0` heading (Scaffold present, verification command, deliverables) and record the D3 deferral in its Known Issues if still open at completion (constitution definition-of-done: documentation updated).

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: no dependencies; T003 (dev deps + lockfile) depends on T001 (manifest) and T002 (`.python-version`)
- **Foundational (Phase 2)**: depends on T003/T009 (tools and verify command available) — BLOCKS all user stories
- **User Stories (Phase 3+)**: all depend on Phases 1–2
  - US1 (P1) → US2 (P2): US2 reuses the US1 command contract but is independently testable
  - US3 (P3): depends on US1 (gates compose into `make verify`) but tests independently
  - US4 (P4): independent of US2/US3, only needs Foundational
- **Polish (Phase 7)**: depends on Phases 1–6

### User Story Dependencies

- **US1 (P1)**: after Foundational — no other story required ✅ **MVP**
- **US2 (P2)**: after Foundational — may integrate with US1 (README) but independently testable
- **US3 (P3)**: after Foundational (the `make verify` composition from T009/T014)
- **US4 (P4)**: after Foundational — fully parallel-safe with US2 and US3

### Within Each User Story

- Setup tasks precede gate configuration (T004–T006 need the tools from T003)
- Package marker (T012) before smoke tests (T013) before US1 validation
- Validations run after each story's implementation tasks

### Parallel Opportunities

- **Setup**: T004–T008 and T010–T011 are all `[P]` — run together after T003
- **Foundational**: T012 and T013 are sequential (marker before test that imports it)
- **Stories**: once Foundational completes, US4 runs fully parallel with US2/US3; US1 is the serial-critical path
- **US3**: T020 and T021 are `[P]` (both temp seeds, different defect classes)
- **US4 activities**: T023, T024, T025 are `[P]`; T026 must land after T024 confirms the authority
- **Polish**: T028/T029/T031 are `[P]` after T027

---

## Parallel Example: User Story 3

```bash
# Launch both seeded-defect validations together (independent temp files):
Task: "T020 [US3] seed unused-import; run make verify; confirm ruff fails with file:line"
Task: "T021 [US3] seed type error; run make verify; confirm mypy fails with location"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (T001–T011)
2. Complete Phase 2: Foundational (T012–T013) — **CRITICAL, blocks everything**
3. Complete Phase 3: User Story 1 (T014–T017)
4. **STOP and VALIDATE** quickstart §3.1/§3.4/§3.3 — this is the Stage 0
   deliverable: "a project that can execute verification with a clean test
   result," now expressed as `make verify`
5. Deploy/demo the foundation before any further story

### Incremental Delivery

1. Setup + Foundational → runnable skeleton
2. + US1 → **MVP**: clean-checkout verification proven
3. + US2 → reproducibility proven across machines
4. + US3 → gates proven to catch defects
5. + US4 → structure governance finalized
6. Polish (T027–T032) → SC map green, changelog/governance record complete

### Parallel Team Strategy

With multiple developers:

1. One developer completes T001–T011 (Setup) and T012–T013 (Foundational)
2. Once Foundational lands:
   - Developer A: US1 (T014–T017) — serial-critical path
   - Developer B: US4 audits (T023–T026)
   - Developer C: US2/US3 validations (T018–T022) once US1's command contract exists
3. One owner runs Polish (T027–T032) and records `docs/experiments/01_repository-foundation.md`

---

## Notes

- `[P]` = different files, no dependencies
- `[Story]` labels map tasks to spec user stories for traceability
- Each story is independently completable and testable (its own quickstart
  scenario)
- This feature writes code that must pass its own gates: every task closing
  must leave `make verify` green (or, for gate-validating tasks, green after
  the deliberate seed is removed)
- Commit after each task or logical group; the session's uncommitted work
  (constitution, overview.md, README, CHANGELOG) should be committed with the
  T014 milestone so the tree is coherent at each checkpoint
- Avoid: vague tasks, same-file conflicts, cross-story dependencies that break
  US2/US3/US4 independence
- **Deviation checkpoint**: research D3 deliberately defers Pydantic/SQLAlchemy/
  Alembic past Stage 0; do NOT reintroduce them in these tasks (T028 asserts
  their absence)