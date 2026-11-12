# Issue

**Title:** FIX AdaLoRA Conv2d with groups and padding_mode

## Description

, .

- `SVDConv2d.lora_A` is now sized `in_channels // groups * kH * kW`, so `groups > 1` no longer fails in forward and merge.
- The adapter branch now uses `base._conv_forward`, so it respects `padding_mode` (it was always zero-padded).

Tests: removed the AdaLoRA `xfail` from `test_unmerged_and_merged_forward_same_result`, and added AdaLoRA to the `Conv2dGroups`/`Conv2dGroups2` custom model cases, with the same wider merge tolerance as grouped LoHa/LoKr.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `03412afcf1e9`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.