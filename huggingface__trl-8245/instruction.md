# Issue

**Title:** Remove `adjusted_mfu`, which nothing calls

## Description

`adjusted_mfu` converted an MFU computed from `compute_flops_per_token` to the convention where causal attention counts half the attention-score FLOPs (Llama, DeepSpeed Ulysses). It came with the other MFU helpers in #5698, and nothing has called it since: no trainer, doc or example uses it, and it isn't exported. Its tests go with it. Came up in #8236.

- The `compute_flops_per_token` docstring still says which convention it uses.
- Conflicts with #8236 on that docstring paragraph: whichever lands second drops the `adjusted_mfu` sentence from it.

<!-- CURSOR_SUMMARY -->
---

> [!NOTE]
> **Low Risk**
> Dead-code removal only; **`compute_mfu`** and **`compute_flops_per_token`** behavior is unchanged.
>
> **Overview**
> Removes the unused **`adjusted_mfu`** helper and its **`TestAdjustedMfu`** tests from the MFU utilities introduced alongside **`compute_flops_per_token`** / **`compute_mfu`**. That function converted MFU from the non-causal FLOP estimate to the causal-attention convention; nothing in the library imported or called it.
>
> The **`compute_flops_per_token`** docstring is trimmed so it no longer points readers at **`adjusted_mfu`**, while still noting the non-causal attention FLOP convention.
>
> <sup>Reviewed by [Cursor Bugbot](https://cursor.com/bugbot) for commit ddb01a516367333775fd81d38352fdf91c9828eb. Bugbot is set up for automated code reviews on this repo. Configure [here](https://www.cursor.com/dashboard/bugbot).</sup>
<!-- /CURSOR_SUMMARY -->

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `3ba02a9de41d`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.