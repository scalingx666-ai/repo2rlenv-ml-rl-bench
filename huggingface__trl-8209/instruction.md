# Issue

**Title:** Tag the model in AsyncGRPO

## Description

Part of #7841.

The model now gets the `async-grpo` and `trl` tags, as in the stable trainers. Covered by `test_tag_added` and `test_tag_added_peft` in #8146.

<!-- CURSOR_SUMMARY -->
---

> [!NOTE]
> **Low Risk**
> Single metadata call at initialization with no changes to training, weight sync, or save logic.
>
> **Overview**
> **AsyncGRPO** now registers Hub metadata tags on the trained model during trainer init, matching stable TRL trainers.
>
> Right after `super().__init__`, it calls `add_model_tags` with the existing `_tag_names` list (`trl`, `async-grpo`), so checkpoints pushed from async GRPO runs are labeled consistently on the Hub.
>
> <sup>Reviewed by [Cursor Bugbot](https://cursor.com/bugbot) for commit 01b28836d0f920185b396e41ff2f833913a763c1. Bugbot is set up for automated code reviews on this repo. Configure [here](https://www.cursor.com/dashboard/bugbot).</sup>
<!-- /CURSOR_SUMMARY -->

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `f2ea8a9ad41f`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.