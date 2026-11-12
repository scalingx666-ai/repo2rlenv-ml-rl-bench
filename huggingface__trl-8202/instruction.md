# Issue

**Title:** Fix the max_tool_calling_iterations docstring

## Description

The `max_tool_calling_iterations` docstring in `GRPOConfig` and `DistillationConfig` says generation stops when "the total response length reaches `max_model_length`". That's not quite what the tool loop does, and the field is called `vllm_max_model_length`.

## What changes

The docstring and the help text now list the two limits the tool loop checks (`grpo_trainer.py`, "Drop tool results whose addition would push the sequence past max_completion_length"):

- the completion, which is all turns and tool results, reaches `max_completion_length` tokens;
- the full sequence reaches the model's maximum length. In colocate mode, that's vLLM's `max_model_len`, which `vllm_max_model_length` sets. Otherwise, it's the model's `max_position_embeddings`.

When a tool result would go past either limit, the conversation ends there.

<!-- CURSOR_SUMMARY -->
---

> [!NOTE]
> **Low Risk**
> Documentation and config field help text only; no runtime or API behavior changes in this diff.
> 
> **Overview**
> Corrects **documentation only** for `max_tool_calling_iterations` on [`GRPOConfig`](trl/trainer/grpo_config.py) and [`DistillationConfig`](trl/trainer/distillation_config.py), plus the agent-training sections in `grpo_trainer.md` and `distillation_trainer.md`.
> 
> The old text said the tool loop ended when total response length hit `max_model_length`, which is not a config field. It now states generation stops when the model returns a turn without tool calls, when the full completion (model turns and tool results) hits `max_completion_length`, or when the prompt+completion sequence hits the context limit (`vllm_max_model_length` with `vllm_mode="colocate"`, otherwise the model's `max_position_embeddings`). Class docstrings and CLI `help` metadata were updated to match.
> 
> <sup>Reviewed by [Cursor Bugbot](https://cursor.com/bugbot) for commit 904fa1a8b52e30f7a7185e9948db178039294264. Bugbot is set up for automated code reviews on this repo. Configure [here](https://www.cursor.com/dashboard/bugbot).</sup>
<!-- /CURSOR_SUMMARY -->

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `5a423556bc32`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.