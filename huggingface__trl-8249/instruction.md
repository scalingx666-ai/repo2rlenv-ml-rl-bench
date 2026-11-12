# Issue

**Title:** Raise at AsyncGRPO init when there is no reward source

## Description

Part of #7841.

`GRPOTrainer` raises "No reward source provided" at init when nothing provides a reward:

https://github.com/huggingface/trl/blob/9507c44fc1714ac2b2d9ba9cc65efd2b5c9a351a/trl/trainer/grpo_trainer.py#L781-L787

`AsyncGRPOTrainer` only raised it once `train()` had started the rollout worker. It now raises at init too when there is no `reward_funcs`, `environment_factory` or custom `loop_cls`, with the same message as the worker.

- The worker keeps its own check for environments, which are only created there: an `environment_factory` whose environment has no `get_reward` still fails at train start.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `9507c44fc171`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.