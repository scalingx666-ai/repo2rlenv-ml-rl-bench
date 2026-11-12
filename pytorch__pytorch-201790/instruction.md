# Issue

**Title:** [release/2.15] [c10d][nccl2] Do not treat zero-sized output tensors as aliased in all_gather

## Description

Cherry-pick of  (e96d01e48ed) into release/2.15.

Root issue: nccl2 (the default "nccl" backend in 2.15) treats zero-sized all_gather outputs as aliased and raises a RuntimeError (). Hit in Megatron FSDP checkpoint loading.

Fix: skip the aliasing check for zero-sized outputs. Clean cherry-pick.

Release validation:

Test plan:
- `python -m pytest test/distributed/test_c10d_nccl2.py`: 92 passed, 4 skipped (with all 5 nccl 2.15 picks applied, 2x GB200).

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `ef10301d9fdf`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.