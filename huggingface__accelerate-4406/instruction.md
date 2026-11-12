# Issue

**Title:** Preserve zero gradient clipping in DeepSpeed configuration

## Description

Comparing loss after training exposed DeepSpeed clipping gradients while the single-GPU Accelerate reference did not. Setting `gradient_clipping=0.0` to disable clipping still left the runs behaving differently.

- **Zero was silently discarded.** Accelerate's configuration builder checks `if self.gradient_clipping`, treating zero as an omitted setting. DeepSpeed 0.19.7 then uses its default threshold of 1.0, despite the explicit request for no clipping.

- **Preserve explicit zero.** Use `is not None` so zero reaches DeepSpeed. Check the generated configuration directly, so a discarded zero fails without launching training.

The bug only affected Accelerate’s generated configuration. Supplying a DeepSpeed JSON file took a different path that already preserved zero.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `ffc870d6dc2e`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.