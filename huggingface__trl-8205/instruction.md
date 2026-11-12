# Issue

**Title:** Check the AsyncGRPO reward count and warn when every reward is None

## Description

Part of #7841.

A reward function that returns the wrong number of rewards now raises GRPO's `ValueError` in the rollout worker, instead of silently broadcasting a single value or failing later on the array shape. A completion that every reward function scored `None` now logs GRPO's warning. Covered by `test_reward_func_wrong_number_of_rewards` and `test_warning_raised_all_rewards_none` in #8146.

<!-- CURSOR_SUMMARY -->
---

> [!NOTE]
> **Low Risk**
> Adds validation and logging in the async rollout reward path only; no auth, security, or training-algorithm changes beyond failing fast on malformed reward outputs.
>
> **Overview**
> Brings Async GRPO rollout scoring in line with `GRPOTrainer` by tightening how rewards are assembled in `_calculate_rewards`.
>
> After each configured reward function runs, the worker now **raises `ValueError`** when a function does not return exactly one score per completion in the group, instead of failing later on a shape/broadcast mismatch. When every reward source (funcs, env columns, rollout rewards) is `None` for a completion, it **logs a warning** with that row’s prompt, completion, and reward kwargs so misconfigured graders are easier to spot.
>
> Reward `None` values are still converted to NaN in a single `np.array` build; behavior for advantage computation is unchanged.
>
> <sup>Reviewed by [Cursor Bugbot](https://cursor.com/bugbot) for commit 5bfb57196808bfe80a81453904353d473355fbd5. Bugbot is set up for automated code reviews on this repo. Configure [here](https://www.cursor.com/dashboard/bugbot).</sup>
<!-- /CURSOR_SUMMARY -->

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `9507c44fc171`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.