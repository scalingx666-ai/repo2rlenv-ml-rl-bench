# Issue

**Title:** [jax-buildcop] Eliminate redundant XLA compilations in `hyp1f1`, `hyp2f1`, and `betaln` gradient tests

## Description

[jax-buildcop] Eliminate redundant XLA compilations in `hyp1f1`, `hyp2f1`, and `betaln` gradient tests

`hyp1f1` and `hyp2f1` previously applied `promote_args_inexact` inside `@custom_jvp` and `@jit`, wrapped `_hyp1f1_x_derivative` and `_hyp2f1_x_derivative` in an outer `@jit`, and passed the differentiated parameter array `p` directly to `_hyp1f1_param_derivative` / `_hyp2f1_param_derivative`. On mixed-shape or mixed-dtype inputs (such as in `testScipySpecialFun`), this forced `_hyp1f1_x_derivative` and `_hyp2f1_x_derivative` to re-inline and recompile the full `hyp1f1` / `hyp2f1` loop graphs instead of reusing the cached `@jit` executable, and caused `_hyp1f1_param_derivative` / `_hyp2f1_param_derivative` to miss the `@jit` cache across parameters of different broadcast shapes.

- Move `promote_args_inexact` into public `hyp1f1` and `hyp2f1` wrappers ahead of `@custom_jvp`, drop the outer `@jit` on `_hyp1f1_x_derivative` and `_hyp2f1_x_derivative` so `_hyp1f1(a + 1, b + 1, x)` and `_hyp2f1(a + 1, b + 1, c + 1, x)` hit the primal `@jit` cache, and select `p` via a scalar index inside `_hyp1f1_param_derivative` and `_hyp2f1_param_derivative` so all parameter derivatives share one compiled executable per input shape tuple.
- In `testBetalnGrads`, pad secondary extreme-input arrays to `a.shape` to reuse the compiled VJP and split `order=2` forward-mode checks into `testBetalnSecondOrderGrads` so first- and second-order AD compilations distribute across shards without reducing test coverage.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `941f3e82a5a1`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.