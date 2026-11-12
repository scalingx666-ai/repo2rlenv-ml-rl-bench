# Issue

**Title:** Refuse image datasets in AsyncGRPO, which trains on text only

## Description

Part of #7841.

AsyncGRPO trains on text only, vision-language models included, so a dataset with an `image` or `images` column now raises at init with a message saying so. Covered by `test_vision_dataset_with_text_model_raises` in #8146.

<!-- CURSOR_SUMMARY -->
---

> [!NOTE]
> **Low Risk**
> Adds init-time dataset validation only; no changes to training, auth, or weight sync logic.
>
> **Overview**
> **AsyncGRPOTrainer** now **fails fast at construction** when `train_dataset` includes an **`image` or `images` column**, with a message that training is **text-only** (including for vision-language checkpoints where only the language side is trained).
>
> Users must **drop image columns** from the dataset instead of starting a run that cannot consume them.
>
> <sup>Reviewed by [Cursor Bugbot](https://cursor.com/bugbot) for commit 1a9ccff73e5638fc22a9d1e0c0d5511433e92139. Bugbot is set up for automated code reviews on this repo. Configure [here](https://www.cursor.com/dashboard/bugbot).</sup>
<!-- /CURSOR_SUMMARY -->

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `f2ea8a9ad41f`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.