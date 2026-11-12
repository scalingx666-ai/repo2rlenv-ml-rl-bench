# Issue

**Title:** Check the tool requirements at AsyncGRPO init

## Description

Part of #7841.

With `tools` or an `environment_factory`, init now runs GRPO's two checks: jmespath must be installed on transformers below 5.13, and the chat template must be able to render a tool call. Both used to fail later, in the rollout worker. Covered by `test_tools_without_jmespath` in #8146.

<!-- CURSOR_SUMMARY -->
---

> [!NOTE]
> **Low Risk**
> Init-time validation only; no training-loop or rollout behavior changes beyond earlier failure on misconfiguration.
>
> **Overview**
> **`AsyncGRPOTrainer.__init__` now fails fast** when `tools` or `environment_factory` is set, using the same checks as **`GRPOTrainer`**: on transformers **&lt; 5.13**, it requires **jmespath** for legacy response parsing; it always verifies the tokenizer’s chat template **`supports_tool_calling`**.
>
> These errors used to surface later in the rollout worker; moving them to construction gives an immediate, clear fix (install jmespath or upgrade transformers / fix the template).
>
> <sup>Reviewed by [Cursor Bugbot](https://cursor.com/bugbot) for commit 6c5cc5c45618e48c695c895200d8f80354a3b18c. Bugbot is set up for automated code reviews on this repo. Configure [here](https://www.cursor.com/dashboard/bugbot).</sup>
<!-- /CURSOR_SUMMARY -->

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `f2ea8a9ad41f`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.