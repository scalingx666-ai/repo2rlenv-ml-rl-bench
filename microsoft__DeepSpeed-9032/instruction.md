# Issue

**Title:** Treat an all-deselected selection as success in cpu-torch-latest-v1-multirank

## Description

## Summary

`cpu-torch-latest-v1-multirank` fails red on any PR whose diff-selected `tests/unit/v1` targets contain no multi-rank tests. `DS_MULTIRANK_ONLY=1` deselects every single-rank test (see the `pytest_collection_modifyitems` hook in `tests/conftest.py`), pytest then reports `no tests ran` and exits with code 5, and the job fails — even though the run is a legitimate green no-op.

The workflow already guards the empty-selection case (`No v1 tests selected; nothing to run.` → `exit 0`) but not the all-deselected case. This PR treats pytest exit code 5 the same way: a green skip with a clear message. Every other exit code is passed through unchanged, so real test failures (1), usage errors (4), and internal errors (3) still fail the job.

## Root cause

Observed on #8939 (run 42374006158): the diff-based selector (`ci/tests_fetcher.py`) correctly selected exactly one target, `tests/unit/v1/test_hybrid_engine_rollout_v1.py`, but all of its tests are plain single-process tests, not `DistributedTest` subclasses. The multirank filter deselected all of them:

```
### 1 pytest target(s) -> ci/.test_selection/test_list.txt ###
  tests/unit/v1/test_hybrid_engine_rollout_v1.py
...
============================ no tests ran in 4.02s =============================
##[error]Process completed with exit code 5.
```

Per pytest's documented exit-code convention, 5 means "no tests were collected" — a neutral "nothing to run" signal, not a failure. Any PR touching only non-distributed tests under `tests/unit/v1` hits this.

## Implementation

- Capture the pytest exit code (`set +e` / `set -e` scoped to that one command).
- Exit 0 with `No multi-rank tests in the selection; nothing to run.` when it is 5, mirroring the existing empty-selection guard one branch above.
- `exit "$pytest_exit"` for everything else.

The nightly `cpu-torch-multi-latest` also sets `DS_MULTIRANK_ONLY=1` but runs the whole `unit/` tree (excluding `v1/`), which always contains multi-rank tests, so it cannot hit this edge and is not changed.

## Validation

- YAML parse check and `bash -n` on the extracted `Unit tests` script: pass.
- Branch simulation of the new logic with a stub pytest: exit 5 → script exit 0 with the skip message; exit 1 → exit 1; exit 0 → exit 0.
- `pre-commit run --files .github/workflows/cpu-torch-latest-v1-multirank.yml`: all hooks pass.
- No hardware device is involved; the change is workflow-shell only. The live-workflow behavior cannot be executed locally — it will be validated by this PR's own CI run (this workflow triggers on the changed path).

## Duplicate-work check

Searched open PRs and issues for `multirank`, `no tests ran`, and `exit code 5`; nothing addresses this.

## AI assistance

This change was prepared with AI assistance (analysis of the failing CI run and drafting of the patch); every line was reviewed, and all listed validation was run, by the human submitter.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `a7ebdf96f062`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.