# Issue

**Title:** windows: add CUDA 13.5 and SM 103 for ARM64

## Description

Install the separate ARM64 cross package in CI and retain SM 121 support. Other platforms are unchanged.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `51e7030f9442`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.