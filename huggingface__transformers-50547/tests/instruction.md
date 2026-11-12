# Issue

**Title:** Fix Step3.7 vision RoPE spatial W/H axis order

## Description

<!-- ci-dashboard-badge:start -->
[![🤗 Transformers CI](https://img.shields.io/badge/%F0%9F%A4%97%20Transformers%20CI-333333?style=flat)](https://transformers-ci.huggingface.cool/d/pytest-observability-by-pr/pytest-observability-branch?var-pr=50547)
<!-- ci-dashboard-badge:end -->

# What does this PR do?

 

**Problem**:
Step3.7's original vision encoder assigns rotary frequencies to W then H. The shared vision position IDs use H/W order, while the checkpoint's Q/K conversion only changes adjacent real/imaginary pairs into half-split channels. Flattening the H/W frequencies therefore rotates the wrong spatial channel bands.

**Patch**:
Swap the two spatial bands in `Step3p7VisionRotaryEmbedding.recomposition_frequencies()`, updating the modular source and regenerating the modeling file.

**Validation**:
- `python -m pytest -q tests/models/step3p7/test_modeling_step3p7.py`: CPU: 152 passed, 157 skipped; CUDA: 159 passed, 150 skipped.
- `make fix-repo` and Linux `make check-code-quality` passed.

## Who can review?
@zucchini-nlp

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `fc6cdba12ac3`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.