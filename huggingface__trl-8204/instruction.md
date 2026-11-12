# Issue

**Title:** Raise a TypeError for an unsupported AsyncGRPO train_dataset

## Description

Part of #7841.

`AsyncGRPOTrainer` now raises GRPO's `TypeError` at init when `train_dataset` is neither a `Dataset` nor an `IterableDataset`. Covered by `test_init_with_train_dataset[unsupported_dataset_dict]` in #8146.

<!-- CURSOR_SUMMARY -->
---

> [!NOTE]
> **Low Risk**
> Init-time input validation only; no change to training, rollout, or weight sync behavior.
>
> **Overview**
> **`AsyncGRPOTrainer`** now rejects invalid **`train_dataset`** values at construction time: if a dataset is provided and it is neither **`Dataset`** nor **`IterableDataset`** (e.g. a plain dict), initialization raises **`TypeError`** with the actual type name—matching GRPO’s validation instead of failing later in the training stack.
>
> Environment-only training (no dataset + **`environment_factory`**) is unchanged; the new check applies only on the non-**`None`** path after placeholder-dataset handling.
>
> <sup>Reviewed by [Cursor Bugbot](https://cursor.com/bugbot) for commit 2a7f2e8374efd54d35f61133616ddea3537770f1. Bugbot is set up for automated code reviews on this repo. Configure [here](https://www.cursor.com/dashboard/bugbot).</sup>
<!-- /CURSOR_SUMMARY -->

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `f2ea8a9ad41f`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.