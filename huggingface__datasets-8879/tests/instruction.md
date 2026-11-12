# Issue

**Title:** Fix pickle RCE in pandas dataset builder (CVE-2026-57845)

## Description

Addresses **CVE-2026-57845**, arbitrary code execution through pickle deserialization in the pandas dataset builder.

Replaces `Pandas._generate_tables()` with an unconditional migration error before file opening or unpickling. Builder construction and compatible prepared Arrow cache reuse remain supported. File resolution and downloads may occur before rejection; streaming loads fail when iterated.

Updates regression tests and remediation documentation.

Validation: all 25 focused pandas/Parquet tests pass, along with lint and formatting checks.

Technical write-up and PoC: https://github.com/SaiTeja-Erukude/CVE-2026-57845-hf-datasets-rce

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `609f27cbf594`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.