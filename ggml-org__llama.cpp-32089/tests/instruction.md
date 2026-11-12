# Issue

**Title:** llama : reuse graph result instances across scheduler reserves

## Description

## Overview

Reuse the `llm_graph_result` instances owned by `llama_context` instead of reallocating them every time the backend scheduler is reserved.

- Allocate `gf_res_reserve` and both `gf_res_prev` entries once in the `llama_context` constructor.
- Add `llm_graph_result::set_max_nodes()` and keep `reset()` as the path for reusing an existing instance.
- In `sched_reserve()`, update each instance's `max_nodes` and reset it instead of destroying and recreating the `unique_ptr`.
- `get_gf_res_prev()` now only selects the arena for the current batch, with no lazy allocation.
- Remove the now-unnecessary null checks around `gf_res_prev`.

## Additional information

The graph results are created unconditionally during context construction, so vocab-only contexts also have valid instances.

Tested on Mac Studio, M5 Ultra, 256GB:

- `test-llama-archs -s 1`: all 569 tests passed.
- `test-llama-context --models build/tests/test-models -j 4` with `GGML_SCHED_DEBUG_REALLOC=1`: 145 models passed, 0 failed.

## Requirements

- I have read and agree with the [contributing guidelines](https://github.com/ggml-org/llama.cpp/blob/master/CONTRIBUTING.md)
- AI usage disclosure: YES
  - Harness: `dsh:llama.cpp`
  - Hardware: Mac Studio, M5 Ultra, 256GB
  - Model: [ggml-org/DeepSeek-V4.1-Flash-1017-GGUF](https://huggingface.co/ggml-org/DeepSeek-V4.1-Flash-1017-GGUF)
  - Stats: ◷ 24.5m · ⚙ 2.3m · ↑ 153k · ↓ 70k · ⇈ 436.0 tok/s · ⇊ 62.0 tok/s · ≡ 222 · ⌨ 14

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `ccd360f060a0`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.