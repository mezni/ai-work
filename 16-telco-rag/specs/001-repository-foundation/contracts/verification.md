# Contract: Verification Command (`make verify`)

**Feature**: [../spec.md](../spec.md) | **Date**: 2026-10-05 | **Source**: spec FR-001–FR-004, FR-009, research D1/D6

## Purpose

`make verify` is the single documented command that composes all of Stage 0's
automated verification. It is the definition of "the project is healthy".

## Invocation

```text
make verify
```

Run from the repository root. All tools execute through `uv run` inside the
project environment locked by `uv.lock` (no activation step, no global tools).

## Composed checks

Order is fixed and short-circuiting (`&&`):

| # | Step | Exit semantics |
| --- | --- | --- |
| 1 | `uv run pytest` | 0 if all tests pass and ≥1 test ran |
| 2 | `uv run ruff check .` | 0 if no lint findings |
| 3 | `uv run mypy src` | 0 if no type errors |

## Result contract

| Condition | Exit code | Output expectations |
| --- | --- | --- |
| All three check steps pass | **0** | summary of each step's pass |
| Any single step fails | **non-zero** | the failing step's report is visible; later steps do not run |
| Suite empty (no tests collected) | **non-zero** | must *not* report success (FR-003); guard test enforces this |
| Interpreter below minimum runtime | **non-zero** | actionable message naming the mismatch (FR-007, SC-007) |

## Validation scenarios (quickstart cross-reference)

| Scenario | Quickstart § |
| --- | --- |
| Clean-tree success | [quickstart §3.4](../quickstart.md#34-clean-tree-success) |
| Cold checkout → pass | [quickstart §3.1](../quickstart.md#31-cold-checkout-pass) |
| Seeded-defect detection | [quickstart §3.5](../quickstart.md#35-seeded-defect-detection) |
| Runtime mismatch | [quickstart §3.6](../quickstart.md#36-runtime-mismatch-and-recovery) |

## Out of bounds

- `make verify` is not a formatter (`ruff format` is intentionally omitted at
  Stage 0 — research D10).
- `make verify` does not start containers or touch `docker-compose.yml`
  (spec assumption 6).
- `make verify` does not run CI, upload anything, or publish.

---

# Contract: Project Manifest (`pyproject.toml` + `uv.lock`)

**Source**: spec FR-005–FR-007, FR-015, research D2/D3

## Purpose

`pyproject.toml` is the manifest; `uv.lock` is its committed, machine-readable
reproducibility contract. Together they answer "what does this project need to
run, and in exactly what versions".

## Structure contract

| Key | Value / rule |
| --- | --- |
| `[project] name` | `telco-rag` |
| `[project] version` | `0.1.0` |
| `[project] requires-python` | `>=3.12` |
| `[project] dependencies` | empty at Stage 0 (research D3) |
| `[dependency-groups] dev` | exactly `pytest`, `ruff`, `mypy` |
| `[tool.ruff]` | `target-version = "py312"`, line-length 88, lint select E,F,I,UP,B,SIM |
| `[tool.mypy]` | `python_version = 3.12`, `strict = true`, scope `src/` |
| `uv.lock` | present and committed; in sync (`uv lock --check` passes) |

## Behavior contract

- `uv sync` on a clean machine reproduces the locked environment — identical
  tool versions to the author's (SC-004).
- Resolving against an interpreter older than 3.12 fails with an actionable
  message (SC-007).
- No credential-shaped value lives in the manifest. Secrets use `.env`, which
  is git-excluded (see [Secret policy](#secret-policy-fr-015) below).

## Version single-sourcing

- `telco_rag.__version__` is derived from distribution metadata
  (`importlib.metadata.version("telco-rag")`), so the version exists in exactly
  one place: `pyproject.toml`.

## Secret policy (FR-015)

- `.env` and all local environment files are git-excluded by default.
- `.env.example` is committed and contains only the variable names recorded in
  `docs/configuration.md` §9 (`DATABASE_URL`, `OPENROUTER_API_KEY`,
  `ENVIRONMENT`, `LOG_LEVEL`) with empty values and explanatory comments.
- Nothing in the manifest, gate configuration, or locked tool chain references
  a real secret.