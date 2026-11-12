# Issue

**Title:** Replace `match_span_kind` macro with `SpanKind`

## Description

The macro was written to get around some suboptimal code generation. After the recent span compression overhaul it is no longer needed, and we can use a simpler type to achieve the same goal.

r? @petrochenkov

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `9de746938314`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.