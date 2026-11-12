# Issue

**Title:** Speed up `bad_words_ids` and `sequence_bias` by matching multi-token sequences in one pass

## Description

<!-- ci-dashboard-badge:start -->
[![🤗 Transformers CI](https://img.shields.io/badge/%F0%9F%A4%97%20Transformers%20CI-333333?style=flat)](https://transformers-ci.huggingface.cool/d/pytest-observability-by-pr/pytest-observability-branch?var-pr=50532)
<!-- ci-dashboard-badge:end -->

# What does this PR do?

`SequenceBiasLogitsProcessor` (and `NoBadWordsLogitsProcessor`, which subclasses it) loops in Python over every multi-token sequence on every generation step. Each iteration builds two small tensors on the device and runs its own comparison, so the cost grows linearly with the number of sequences. A blocklist built by tokenizing words, as the `bad_words_ids` docs suggest, is mostly 2 to 4 token entries. On CUDA each one costs about 110 us per step, so 100 banned words add about 17 ms to every generated token ( for a related slowness report).

This PR groups the multi-token sequences by prefix length once, in `_prepare_bias_variables` (which already runs on the first call), and matches each group against the tail of `input_ids` with a single broadcast comparison. The biases are added with `index_add_`, so several sequences ending in the same token still add up. Prefixes longer than the context are still skipped, and single-token sequences are handled as before.

| processor call, vocab 32k, context 512 | CUDA before | CUDA after | CPU before | CPU after |
|---|---|---|---|---|
| 1 word | 148 us | 84 us | 57 us | 43 us |
| 10 words | 940 us | 190 us | 276 us | 79 us |
| 200 words | 17.3 ms | 0.18 ms | 6.05 ms | 0.10 ms |
| 200 words, batch 8 | 19.3 ms | 0.19 ms | 8.07 ms | 1.20 ms |
| `generate()`, 100 bad words (tok/s) | 50 | 242 | 40 | 50 |

Medians of 7 alternating rounds in one process (5 for `generate()`, a 2-layer Llama with vocab 32k, 100 greedy tokens). Word lengths are drawn from 1 to 4 tokens. CUDA is an RTX 4070 SUPER with torch 2.11; CPU is a Ryzen 5 5600X on one thread with torch 2.15. `generate()` outputs are identical before and after, and runs without bad words take the same code path.

I compared the old and new `__call__` on about 16k random cases with matches planted at the end of `input_ids` (up to 150 sequences of length 1 to 12, `-inf`, `inf` and finite biases, batch 0 to 9, empty context, int64 and int32 ids, fp64/fp32/fp16/bf16 scores, several calls per processor). The positions of `-inf`, `inf` and NaN always match. Values only differ when two or more matching sequences end in the same token, because the additions happen in a different order, and then only by rounding. I added a test for overlapping last tokens, a prefix longer than the context, and reuse across calls.

## Code Agent Policy

- [ ] (First-time contributors only): I confirm that this PR description and code is not written by an LLM or code agent

## Before submitting
- [ ] This PR fixes a typo or improves the docs (you can dismiss the other checks if that's the case).
- [x] Did you read the [contributor guideline](https://huggingface.co/docs/transformers/contributing) and the
      [Pull Request](https://huggingface.co/docs/transformers/pr_checks) checks?
- [ ] Was this discussed/approved via a Github issue or the [forum](https://discuss.huggingface.co/)? Please add a link
      to it if that's the case.
- [ ] Did you make sure to update the documentation with your changes according to the [guidelines](https://github.com/huggingface/transformers/tree/main/docs)?
- [x] Did you write any new necessary [tests](https://huggingface.co/docs/transformers/testing)?

## Who can review?

generate: @Cyrilvallez

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `147e66b34836`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.