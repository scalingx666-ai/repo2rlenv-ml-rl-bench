# Issue

**Title:** FSDP2: don't upcast placeholder weights on non-main ranks with cpu_ram_efficient_loading

## Description

# What does this PR do?

With `cpu_ram_efficient_loading` and mixed precision, `fsdp2_prepare_model` upcasts all trainable parameters to fp32 on every rank before sharding. On non-main ranks these weights are later replaced by the main process' weights in `fsdp2_load_full_state_dict`, but the upcast still writes a full fp32 copy into host memory on each rank, ~4 bytes per trainable parameter per process.

This PR captures the non-persistent buffers and moves non-main ranks to `meta` before the upcast, so it is free there. The main rank is unchanged.

Host memory after `prepare` with Qwen2.5-7B (full fine-tuning, bf16 mixed precision) and 4 ranks:

|  | main rank | each non-main rank | losses |
|---|---|---|---|
| main | 28.4 GiB | 28.4 GiB | 12.5000, 11.9375 |
| this PR | 28.0 GiB | 1.1 GiB | 12.5000, 11.9375 |

`tests/fsdp` passes locally (single GPU); the multi-GPU tests need CI.

## Before submitting
- [ ] This PR fixes a typo or improves the docs.
- [x] Did you read the contributor guideline, Pull Request section?
- [ ] Was this discussed/approved via a Github issue or the forum?
- [ ] Did you make sure to update the documentation with your changes?
- [ ] Did you write any new necessary tests?

## Who can review?

@SunMarc @roycho96

🤖 Generated with [Claude Code](https://claude.com/claude-code)

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `08198562915f`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.