# Feature Specification: Repository Foundation (Stage 0)

**Feature Branch**: `001-repository-foundation`

**Created**: 2026-10-05

**Status**: Draft

**Input**: User description: "from docs/implementation-plan.md stage 0"

**Source**: `docs/implementation-plan.md` §4 (Stage 0 — Repository Foundation)

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 — Verify a working project from a clean checkout (Priority: P1)

A developer clones the repository and runs a single documented command. The
project prepares itself and reports a passing result. The developer knows
within one command whether their environment and the project are healthy.

**Why this priority**: This is the stated deliverable of Stage 0 — "a project
that can execute verification with a clean test result." Until this works,
every later stage has no foundation to build on and no way to detect
regressions. It is the gate that all other work passes through.

**Independent Test**: From a fresh clone with no local setup beyond the
documented prerequisites, run the single documented verification command and
observe a passing result. Deliverable: a developer's confident one-command
health check.

**Acceptance Scenarios**:

1. **Given** a fresh clone of the repository, **When** the developer runs the
   documented verification command, **Then** it completes and reports a passing
   result.
2. **Given** a healthy checkout, **When** the verification command runs, **Then**
   it exits successfully and reports zero failing checks.
3. **Given** the project before this feature exists, **When** a developer
   attempts to run verification, **Then** no command can be run because the
   project has no dependency set and no tests — confirming the feature's value.
4. **Given** a checkout where the environment does not satisfy the project's
   stated minimum runtime, **When** the developer runs verification, **Then**
   they receive an unambiguous message naming the mismatch rather than a
   confusing failure.

---

### User Story 2 — Reproducible environment across contributors (Priority: P2)

Two developers on different machines install the project from the declared
requirements and reach an identical working environment. Neither maintains a
private, undocumented list of setup steps.

**Why this priority**: A foundation that only works on the machine that created
it forces every contributor to re-derive the setup independently, and makes
"it works on my machine" unfalsifiable. Reproducibility is what allows the
success of later stages to be trusted.

**Independent Test**: Two contributors on separate clean machines follow only
the repository's own instructions to prepare their environment, then both run
verification. Deliverable: identical, passing results from both.

**Acceptance Scenarios**:

1. **Given** two contributors starting from clean machines, **When** each
   prepares the environment using only repository instructions, **Then** both
   obtain a working environment without manual intervention.
2. **Given** a declared dependency set, **When** the environment is prepared on
   a clean machine, **Then** every declared requirement is satisfied and no
   undeclared requirement is introduced.
3. **Given** a contributor whose machine differs from the original author's,
   **When** they run verification, **Then** the result matches the result on the
   original machine.
4. **Given** a working environment containing local credentials, **When** the
   contributor commits their work, **Then** the credential is excluded from
   version control while a committed template still records which variables are
   required.

---

### User Story 3 — Automated quality gates catch defects before review (Priority: P3)

A developer introduces a defect that an automated check can detect. Running the
project's quality gates surfaces the problem immediately, so it is not left for
a human reviewer or for a later stage to discover.

**Why this priority**: Quality gates are cheap to establish at the foundation
and expensive to retrofit. However, the project can still function without them
at Stage 0 — it just carries more risk forward. Hence P3 rather than P1/P2.

**Independent Test**: Introduce a defect that the configured checks are designed
to detect, run the quality gates, and observe that the run fails and identifies
the offending location. Deliverable: a demonstrably effective automated gate.

**Acceptance Scenarios**:

1. **Given** a defect the checks are designed to catch, **When** the developer
   runs the quality gates, **Then** the run fails and identifies the location of
   the problem.
2. **Given** a clean codebase, **When** the developer runs the quality gates,
   **Then** the run passes with no findings.
3. **Given** a developer who bypasses the gates, **When** the defects reach the
   verification command, **Then** verification still fails — the single
   documented command remains the authoritative gate.

---

### User Story 4 — Repository structure matches the target architecture (Priority: P4)

A contributor exploring the repository can locate where new code will go and
where the specification lives, because the layout matches the structure the
project has committed to. Duplicated or conflicting definitions of that layout
do not exist.

**Why this priority**: The structure is a documented architectural commitment,
but the project delivers value at Stage 0 even if the layout is imperfect —
misplacement is recoverable, whereas an unverified foundation is not.

**Independent Test**: Given a contributor asked to add a new component, observe
that they place it correctly using only the repository's own guidance, and that
only one document defines the target layout. Deliverable: unambiguous placement
guidance.

**Acceptance Scenarios**:

1. **Given** the repository, **When** a contributor looks for the specification
   and project documentation, **Then** both are discoverable from the repository
   entry point.
2. **Given** the target structure definition, **When** a contributor consults
   the repository for where a new component belongs, **Then** exactly one
   authoritative document describes the target layout.
3. **Given** a directory belonging to a stage that has not been reached, **When**
   a contributor inspects the repository, **Then** the directory is absent rather
   than present but empty and misleading.

---

### Edge Cases

- **No tests exist yet.** An empty test suite must not report a healthy
  project. If the verification command runs against zero checks it may signal
  nothing was verified rather than success — Stage 0 must include at least one
  meaningful check so the result is a real, non-empty pass.
- **Minimum runtime not satisfied.** The developer has an older runtime than the
  project supports. Verification must name the mismatch plainly instead of
  failing with an unrelated error.
- **Environment preparation fails partway.** A partially prepared environment
  must not silently appear healthy; the failure must be visible and the cause
  named.
- **No network access.** Environment preparation from a cold state requires
  fetching requirements. Verification against an already-prepared environment
  must still be possible.
- **Secrets present in the working tree.** Credentials that must never be
  committed must be excluded by default, with only a template committed that
  documents which variables are required.
- **Tooling defect introduced but bypassed.** A contributor runs the quality
  gates, finds them clean, but the defect is caught by the verification command
  anyway — verification remains the final authority.
- **Target layout conflict.** The canonical structure definition and a stage-level
  partial listing disagree on which root entries exist at Stage 0. The conflict
  must be resolved rather than left for contributors to interpret.

---

## Requirements *(mandatory)*

### Functional Requirements

**Verification and health**

- **FR-001**: The project MUST provide a single documented command that runs
  all automated verification.
- **FR-002**: That command MUST return a failing result when any check fails and
  a passing result when all checks pass.
- **FR-003**: The verification suite MUST contain at least one meaningful check,
  so that a passing result is evidence of real verification rather than an
  empty run.
- **FR-004**: The repository's default state at completion of this stage MUST
  produce a passing verification result.

**Reproducible environment**

- **FR-005**: All project requirements MUST be declared explicitly in a
  version-controlled manifest rather than implied by convention.
- **FR-006**: The declared requirement set MUST be reproducible: preparing the
  environment from the manifest on a clean machine MUST yield the same result
  as on the machine that produced the manifest.
- **FR-007**: The project MUST declare a minimum supported runtime version, and
  MUST report a clear, actionable message when the local runtime does not
  satisfy it.
- **FR-008**: The documented instructions MUST be sufficient for a contributor
  with no prior project knowledge to reach a passing verification result
  without supplementary help.

**Quality gates**

- **FR-009**: The project MUST include configured automated checks for at least
  two classes of defect — one detectable by static code inspection and one
  detectable by type analysis — and these MUST be runnable from the documented
  entry point.
- **FR-010**: Quality-gate configuration MUST be version-controlled so that all
  contributors and machines apply identical rules; local, per-developer
  configuration MUST NOT be required.
- **FR-011**: A defect detectable by the configured checks MUST cause a failing
  run when the checks are executed.

**Repository layout and safety**

- **FR-012**: The repository MUST be arranged so that later-stage directories
  are not created before their stage is reached; directories belonging to
  unreached stages MUST be absent, not empty placeholders.
- **FR-013**: Exactly one document MUST be authoritative for the target
  repository structure; stage-level listings MUST defer to it rather than
  restate a divergent version.
- **FR-014**: The repository entry point MUST describe the project's purpose,
  its current status, and how to run verification.
- **FR-015**: Files that must never be committed (credentials and
  environment-specific secrets) MUST be excluded from version control by
  default, while a committed template documents which variables are required.
- **FR-016**: Documentation and specification files already present MUST be
  retained and remain reachable from the repository entry point.

### Key Entities *(configuration artifacts, not domain data)*

- **Project Manifest**: The authoritative declaration of what the project
  requires to run — runtime version and declared requirements. Single source
  for environment preparation; must be machine-reproducible.
- **Verification Command**: The one documented entry point for automated
  health checking. Composes test execution and quality gates; its result is the
  project's definition of "healthy".
- **Quality Gate**: A named, version-controlled automated check with an
  explicit defect class it detects, a pass/fail result, and a location when it
  fails.
- **Repository Layout**: The committed definition of the target structure and
  which directories exist at which stage. Authoritative in exactly one place.

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A contributor with no prior knowledge of the project reaches a
  passing verification result from a clean clone in **under 10 minutes**,
  following only repository documentation.
- **SC-002**: Once the environment is prepared, a verification run completes in
  **under 60 seconds**, so it is run habitually rather than occasionally.
- **SC-003**: On the repository's default state, **zero** verification checks
  fail and **at least one** substantive check executes.
- **SC-004**: Preparing the environment from the manifest on two different
  clean machines produces **identical** tooling versions in **100%** of cases.
- **SC-005**: A seeded, detectable defect is caught by the configured checks in
  **100%** of trials across **10** seeded defects.
- **SC-006**: When a seeded defect causes a quality gate to fail, the run
  identifies the offending location in **100%** of trials — no failure is
  reported without a locator.
- **SC-007**: An intentionally stale or unsupported runtime produces a clear,
  actionable message in **100%** of attempts, and **0%** of attempts surface an
  unrelated or opaque error as the first indication.
- **SC-008**: **100%** of contributors report that verification results match
  across their machines, surveyed over the first **3** contributors to run it.
- **SC-009**: Exactly **one** document is authoritative for the target
  repository structure, verified by **0** divergent restatements of root-level
  layout elsewhere in the documentation.
- **SC-010**: Directories belonging to unreached stages number **0** in the
  committed repository.

---

## Assumptions

- **Toolchain is pre-decided, not chosen here.** `docs/constitution.md`
  §Technology Principles and `docs/implementation-plan.md` §4 already mandate
  the stack (Python 3.12+, `uv`, `pytest`, `Ruff`, `mypy`, `Pydantic`,
  `SQLAlchemy`, `Alembic`). This specification constrains *outcomes and
  reproducibility* and does not re-open that choice.
- **Scope is the Stage 0 root scaffolding only.** The entry points created are
  those listed in `docs/implementation-plan.md` §4: project manifest,
  repository metadata files, container/orchestration entry point, task runner,
  and `config/`, `docs/`, `data/`, `scripts/`, `src/`, `tests/`.
- **`uv.lock` is an output, not an input.** It is produced by environment
  preparation and is expected to exist and be committed, even though the Stage
  0 listing omits it. It is required by `docs/architecture/overview.md` §5.1.
- **Later-stage directories are out of scope.** Per
  `docs/architecture/overview.md` §8, `prompts/`, `migrations/`,
  `frontend/`, `alembic.ini`, and `agents/` belong to later stages and MUST NOT
  be created at Stage 0. They are absent by design, not missing.
- **`docs/` already exists and needs no creation.** It currently holds 22
  specification documents; the requirement is retention and reachability.
- **`docker-compose.yml` is provisioned but not exercised.** Stage 0 creates
  the file; no containerized service is required to reach a passing
  verification result. Containers become mandatory at Stage 3 (Database).
- **CI/CD is explicitly out of scope.** Automated checks are run locally by
  contributors; pipeline execution belongs to Stage 25.
- **"At least two defect classes" maps to the pre-decided tooling.** The
  static-inspection class maps to the mandated linter and the type-analysis
  class to the mandated type checker; both are named in the assumptions above
  rather than in the requirements, so the specification stays tool-agnostic.
- **No authentication, authorization, or domain behaviour exists at this
  stage.** Users of this feature are contributors, not platform end users.
  The personas in `docs/product.md` are unaffected by Stage 0.

## Dependencies

- `docs/implementation-plan.md` §4 — the source stage definition.
- `docs/architecture/overview.md` §5, §8 — authoritative target layout and
  stage-to-directory mapping.
- `docs/constitution.md` §10 (Python Architecture), §15 (Configuration
  Management), §19 (Testing), §30 (Definition of Done) — governing constraints.
- `docs/testing.md` §4 (Test Directory), §33 (Test Markers) — test layout
  expectations.
- `.specify/memory/constitution.md` v1.0.0 — governance under which this
  specification is authored.

## Out of Scope

- Domain model, configuration loading, database, ingestion, retrieval,
  generation, security, evaluation, observability, API — all Stages 1+.
- Any application source file under `src/` beyond package markers.
- CI/CD pipelines (Stage 25), frontend (Stage 24), agentic RAG (Stage 21).
- Resolving the `docs/architecture.md` vs `docs/architecture/` naming
  coexistence — tracked as a known issue, not resolved by this stage.
- Reconciling `prompts/evaluation/` with `docs/evaluation.md` §48.