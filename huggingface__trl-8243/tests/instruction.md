# Issue

**Title:** Publish the new token allowance at the end of each step

## Description

. Part of #7841.

## Why

Bugbot found this on #8170 just before it merged. At the end of each optimizer step, the trainer recomputes `lookahead_tokens` from the tokens the step actually pulled. But it only wrote the new limit to the shared `token_allowance` on the next sample pull. Until then, the worker kept the previous limit through weight sync, logging and checkpointing.

It matters most after the first step. Until then the lookahead is the first estimate, `max_staleness × gradient_accumulation_steps × num_processes × token_budget`, and it can be wrong either way. It's too high when rows aren't full, so generation could run past `max_staleness` and those extra samples were later dropped as stale. It's too low with shared prompts, because the budget counts unique tokens and the allowance counts raw ones: in `test_train` a step pulls 369 tokens against an estimate of 64, so the gate could close early and generation waited for the next pull.

## What changes

The step end now also sets `token_allowance` to `consumed_tokens + lookahead_tokens`, in both `AsyncGRPOTrainer` and `AsyncDistillationTrainer`. That's the same value the next pull would set.

## Tests

The AsyncGRPO and AsyncDistillation test files pass on Python 3.13 with transformers 5.22 and on Python 3.12 with transformers 5.17.


<!-- CURSOR_SUMMARY -->
---

> [!NOTE]
> **Medium Risk**
> Touches async training pacing and shared multiprocessing state; incorrect allowance updates could still affect staleness or throughput, but the change is a small, targeted sync of an existing formula.
> 
> **Overview**
> Fixes a gap where **async GRPO and async distillation** trainers recomputed `lookahead_tokens` at step end but did not refresh the rollout worker’s shared **`token_allowance`** until the next sample pull. Generation could therefore run with a stale (often inflated) limit through weight sync, logging, and checkpointing—especially after the first step—causing samples past **`max_staleness`** to be dropped as stale.
> 
> **Both trainers** now set `token_allowance` to `consumed_tokens + lookahead_tokens` on the main process at step end, matching what the next pull would have applied. An **AsyncGRPO** regression test records allowance via `TrainerCallback.on_step_end` and asserts it equals the dataset’s expected value for each of three steps.
> 
> <sup>Reviewed by [Cursor Bugbot](https://cursor.com/bugbot) for commit 52aa90a895b963090daa8b17a9579835ac16628f. Bugbot is set up for automated code reviews on this repo. Configure [here](https://www.cursor.com/dashboard/bugbot).</sup>
<!-- /CURSOR_SUMMARY -->

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `9507c44fc171`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.