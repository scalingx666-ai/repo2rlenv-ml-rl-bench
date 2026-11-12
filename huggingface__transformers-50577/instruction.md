# Issue

**Title:** Fix RecurrentGemma padded generation by declaring its `layer_types`

## Description

<!-- ci-dashboard-badge:start -->
[![🤗 Transformers CI](https://img.shields.io/badge/%F0%9F%A4%97%20Transformers%20CI-333333?style=flat)](https://transformers-ci.huggingface.cool/d/pytest-observability-by-pr/pytest-observability-branch?var-pr=50577)
<!-- ci-dashboard-badge:end -->

Alternative to #50554, following @vasqu's : fix the layer types instead of patching the call sites.

## Problem

`RecurrentGemmaConfig` has no `layer_types`, so every layer is inferred as `sliding_attention` (`sliding_window` is set). The cache then gives the recurrent blocks KV slots too, and the mask / `get_seq_length()` read layer 0's slot, which belongs to a recurrent block and is always empty. With padding, the decode mask gets the wrong size: on GPU, CUDA SDPA raises `(*bias): last dimension must be contiguous`. On CPU, generation runs but the outputs are silently wrong. Regression from #48961.

## Fix

- Config: `layers_block_type` now returns standard layer types (recurrent → `linear_attention`, attention → `sliding_attention`) and is aliased as `layer_types` via `attribute_map`. Nothing changes in `config.json`, and the attention computation is unchanged (it reads `config.sliding_window` directly). Only the cache slots of the recurrent blocks change type, and those slots are never used: the recurrent block keeps its state on the module.
- Modeling: accept the mask dict that `generate` pre-builds once `layer_types` exists (needed by the static-cache / compile path).
- Tests: skip `test_recurrent_layers_mask_padding_on_continued_forward`. It now runs because the config declares a recurrent layer type, but the recurrent block has never supported a cached multi-token continuation (same skip as Mamba / Jamba).

Same generated tokens and scores as #50554 (tiny model, padded batch).

## Results (A10G, torch 2.15)

| Test | `main` | This PR |
|---|---|---|
| `test_generate_with_and_without_position_ids` | 200/200 failed | 200/200 passed |
| `test_generate_from_random_inputs_embeds` | 29/1000 failed | 1000/1000 passed |
| `tests/models/recurrent_gemma` | 1 failed, 129 passed | 130 passed |

The slow integration tests' expected values still need updating (same as #50554).

🤖 Generated with [Claude Code](https://claude.com/claude-code)

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `04935b4f86ec`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.