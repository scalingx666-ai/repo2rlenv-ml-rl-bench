# Issue

**Title:** FIX reduce_intruder_dimension for fan_in_fan_out and embedding LoRA layers

## Description

`reduce_intruder_dimension` rebuilt `lora_A` and `lora_B` from the SVD of `get_delta_weight` without transposing it for `fan_in_fan_out` (GPT-2 `Conv1D`) and `nn.Embedding` layers, whose delta is in the base weight layout. The new factors had the wrong shape, so a square `Conv1D` gave wrong output silently and other shapes failed in `forward`.

The fix transposes `W` and `dW` before the SVD for those layers.

Test: `pytest tests/test_lora_intruders.py` on CPU. The three new cases (`conv1d`, `conv1d_rect`, `embedding`, with `mitigation_lambda=1.0` so output must not change) fail before the fix and all 9 pass after.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `d672982142a7`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.