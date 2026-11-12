# Issue

**Title:** Reject duplicate WebDataset fields instead of silently overwriting data

## Description

A TAR archive can contain consecutive members with the same name, such as two `00000.txt` entries with different contents. The WebDataset pipeline currently overwrites the first value and returns only the second without reporting that data was lost.

This change raises `ValueError` before overwriting an existing field in the current sample. The message includes the field, sample key, and TAR archive path. The check resets at sample boundaries, so different samples can still use the same field name.

Regression tests build real TAR archives for duplicate text, compound JSON, and binary fields, and verify that the same field is accepted in different samples.

Validation:
- Before the fix: the three duplicate-field regression cases fail because no exception is raised; the separate-sample control passes.
- After the fix: `python -m pytest tests/packaged_modules/test_webdataset.py -q -rs` — 11 passed, 2 skipped (TorchCodec unavailable; tensor test requires NumPy < 2 on Windows).
- Ruff lint and format checks pass for both changed files; `git diff --check` passes.

On Windows, the test run used a shorter `--basetemp` path to avoid the platform's filename-length limit for cache lock files.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `18378f7322c7`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.