# Issue

**Title:** Make the tiny Gemma2 model Flash Attention compatible with head_dim=8

## Description

This PR raises `head_dim` from 2 to 8 in the tiny Gemma2 generator, so the model runs with Flash Attention 3.

Part of #6857.

### Motivation

The tiny model sets `head_dim=2`. Flash Attention 2 pads and accepts it, but Flash Attention 3 (`kernels-community/flash-attn3`) requires a multiple of 8 and raises `head_size should be a multiple of 8`.

### Solution

Gemma2 already decouples the head size from the width through `config.head_dim`, so only that value changes: `hidden_size` stays 8 and the heads become 8 wide. `query_pre_attn_scalar` follows it to 8, as the reference sets it equal to `head_dim` (256). Only the attention projections grow, by about 1,150 parameters on a model of 2.05M that is almost all embeddings; as measured in #6857, an explicit `head_dim` costs nothing measurable in peak VRAM (+0 MiB at p50 and p90).

- Built with transformers 4.56.2, the version the script pins: the smoke test and the dtype check pass, and against `google/gemma-2-2b-it` the config only differs in its size fields, with `head_dim` and `query_pre_attn_scalar` going from 256 to 8.
- Reloaded from the saved files with transformers 5.22.1: Flash Attention 3 runs the forward and the backward, with logits matching eager attention up to bf16 rounding (Gemma2 has no SDPA support), while the current Hub model raises the error above.

### Changes

- Set `head_dim=8` and `query_pre_attn_scalar=8` in the tiny Gemma2 config

<!-- CURSOR_SUMMARY -->
---

> [!NOTE]
> **Low Risk**
> Only changes tiny-model generation config in a dev script; no runtime library or production model behavior.
>
> **Overview**
> Updates the **tiny Gemma2** hub generator so attention uses **`head_dim=8`** and **`query_pre_attn_scalar=8`** instead of 2. That aligns the smoke-test model with **Flash Attention 3**, which rejects head sizes that are not multiples of 8, while keeping **`hidden_size=8`** and other tiny dimensions unchanged via Gemma2’s decoupled `head_dim`.
>
> <sup>Reviewed by [Cursor Bugbot](https://cursor.com/bugbot) for commit c6fd2d2d6d1ab01a9e4e7373fde2e19364fb5671. Bugbot is set up for automated code reviews on this repo. Configure [here](https://www.cursor.com/dashboard/bugbot).</sup>
<!-- /CURSOR_SUMMARY -->

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `d9caf95c1f6a`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.