# Quickstart: Repository Foundation (Stage 0)

**Feature**: [spec.md](spec.md) | **Plan**: [plan.md](plan.md) | **Date**: 2026-10-05

This is a validation/run guide, not an implementation document. It proves the
feature works end-to-end. Implementation details belong in `tasks.md`.

## 1. Purpose

Verify that the repository foundation delivers its contract:
a contributor reaches a passing, meaningful verification result from a fresh
checkout using a single documented command, reproducibly, with secrets
excluded.

- [Spec requirements](spec.md)
- [Verification command contract](contracts/verification.md)
- [Manifest contract](contracts/verification.md#contract-project-manifest-pyprojecttoml-uvlock)
- [Configuration artifacts](data-model.md)

## 2. Prerequisites

| Tool | Required version | Notes |
| --- | --- | --- |
| `git` | any modern | repository checkout |
| `uv` | 0.11+ (current 2026 release) | the only required installer; it manages the Python interpreter itself |
| `make` | standard | the documented command entry point |

Python itself need **not** be pre-installed: `uv` downloads the pinned
interpreter (`.python-version`) automatically. Docker is optional and not used
by any validation below.

## 3. Validation Scenarios

No scenario below requires anything beyond the repository, `git`, `uv`, and
`make`. None start containers or call external providers.

### 3.1 Cold Checkout → Pass

**Validates**: SC-001 (clean checkout → pass < 10 min), FR-001, FR-004, FR-008.

```bash
git clone <repo-url> telco-rag && cd telco-rag
make env        # uv sync --group dev: installs locked env incl. interpreter
make verify
```

**Expected outcome**: `make verify` exits 0 and reports, in order:

```text
tests passed (N passed)          # N ≥ 1, from tests/test_smoke.py
ruff check: All checks passed!
mypy: Success
```

**Bound**: the end-to-end wall time from `git clone` to the final `make verify`
exit 0 must be under 10 minutes on a warm network (SC-001), with the checks
phase itself under 60 seconds (SC-002).

### 3.2 Reproducible Setup (Two Machines)

**Validates**: SC-004 (identical tooling across machines), FR-006.

1. Run scenario 3.1 on machine A. Record `uv.lock`'s resolved versions.
2. On machine B, run only the documented commands (no extra installs).
3. Compare `uv run pytest --version` / `uv run ruff --version` /
   `uv run mypy --version` outputs.

**Expected outcome**: identical tool versions on both machines; no personal
setup steps were required on either.

### 3.3 Single-Command Failure Semantics

**Validates**: FR-002, FR-003, and the empty-run guard.

1. On a clean tree, run `make verify` → exit 0 (see 3.4).
2. Run `uv run pytest --collect-only -q` and confirm ≥1 test exists.
3. Temporarily remove `tests/test_smoke.py`, run `make verify`.

**Expected outcome**: after step 3 the command exits non-zero and does not
report success — an empty suite must never signal "healthy" (FR-003 / D6
guard).

### 3.4 Clean-Tree Success

**Validates**: SC-003 (zero failing checks, ≥1 substantive check), FR-004.

```bash
make verify
```

**Expected outcome**: three composed steps all pass; exit 0. The test step
ran at least the three foundation assertions in `tests/test_smoke.py`
(manifest metadata, package import + version, non-empty suite).

### 3.5 Seeded Defect Detection

**Validates**: SC-005, SC-006 (100% detection with a locator), FR-009, FR-011.

1. Introduce a seeded defect of each class:
   - static inspection: an unused import or unused variable;
   - type analysis: `x: int = "oops"` in a temporary module under `src/telco_rag/`.
2. Run `make verify` and observe the run halts **at the first failing step**,
   naming `file:line`.

**Expected outcome**: in every trial the offending step exits non-zero and
reports the exact location. After removing the defect, `make verify` returns to
exit 0.

> Note: a type-analysis seed needs an analyzable module; until Stage 1 adds
> real modules, this part of 3.5 is exercised against a temporary file that is
> deleted before commit.

### 3.6 Runtime Mismatch and Recovery

**Validates**: FR-007, SC-007 (clear message, 0 opaque errors).

1. In a copy of the project, force an older runtime, e.g. set
   `requires-python` to `>=3.999` (temporary edit) or pin `.python-version`
   to an unsupported minor.
2. Run `make env`.

**Expected outcome**: resolution fails with an actionable message naming the
runtime vs. requirement mismatch — never a traceback or a silent partial
install. Revert the edit and re-run to confirm recovery.

### 3.7 Secret Hygiene

**Validates**: FR-015, Principle X.

1. `ls .env.example` → present, committed, contains only blank variable names
   and comments.
2. `git status --porcelain` after creating a local `.env` → `.env` does not
   appear (it is ignored).
3. `git grep -I -i 'openrouter_api_key[=:]' -- . ':!docs'` on the committed tree
   → no matches for a populated credential value.

## 4. Expected Outcomes (Success-Criteria Map)

| SC | Verifiable via |
| --- | --- |
| SC-001 cold pass < 10 min | §3.1 |
| SC-002 checks < 60 s | §3.1 / §3.4 (time the check phase) |
| SC-003 zero failures, ≥1 check | §3.4 |
| SC-004 identical tooling, 100% | §3.2 |
| SC-005 code review 100% (10 trials) | §3.5, repeated ×10 |
| SC-006 locator on every failure | §3.5 |
| SC-007 actionable runtime message | §3.6 |
| SC-008 contributor agreement ×3 | §3.2 run by the first 3 contributors |
| SC-009 one authoritative structure doc | repository audit (`docs/architecture/overview.md`), no divergent `implementation-plan.md` tree restating root layout |
| SC-010 zero unreached-stage dirs | `find . -type d \( -name prompts -o -name migrations -o -name frontend \)` → empty |

**Exit criteria for the feature**: all ten success criteria pass in a demo
session, including at least one full cold-checkout (§3.1) and one seeded-defect
run (§3.5).