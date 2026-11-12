# Issue

**Title:** Fix a passed PeftModel and refuse prompt-learning adapters in AsyncGRPO

## Description

Part of #7841.

Passing a `PeftModel` as `model` crashed at the first training step (`'CausalLMOutputWithPast' object has no attribute 'last_hidden_state'`): the fused LM head was patched on the wrapper, whose `base_model` is PEFT's tuner rather than the decoder. It is now patched on the base model.

Prompt-learning adapters (prefix tuning, prompt tuning, P-tuning) now raise at init: they add virtual tokens rather than weights, and the weight sync cannot carry them to the vLLM server.

Covered by `test_train_peft_model` and `test_peft_prompt_learning_raises` in #8146.

<!-- CURSOR_SUMMARY -->
---

> [!NOTE]
> **Medium Risk**
> Changes PEFT initialization in AsyncGRPO (fused head target and new hard failure for prompt-learning configs); incorrect handling would break training or silently desync policy vs server.
>
> **Overview**
> Fixes training when **`model` is already a `PeftModel`**: `add_fused_lm_head` now targets **`model.get_base_model()`** instead of the PEFT wrapper, so the fused head attaches to the real decoder and the first step no longer fails on missing `last_hidden_state`.
>
> Adds an **init-time guard** for **prompt-learning PEFT** (`PromptTuning`, `PrefixTuning`, `P-Tuning`) via `PromptLearningConfig`: these adapters only add virtual tokens, which **weight sync to vLLM cannot carry**, so the server would roll out without them. Users are directed to weight-based adapters such as **LoRA**.
>
> <sup>Reviewed by [Cursor Bugbot](https://cursor.com/bugbot) for commit 9e266e3aedd82b0e9c71088693e93d92617c1562. Bugbot is set up for automated code reviews on this repo. Configure [here](https://www.cursor.com/dashboard/bugbot).</sup>
<!-- /CURSOR_SUMMARY -->

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `9507c44fc171`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.