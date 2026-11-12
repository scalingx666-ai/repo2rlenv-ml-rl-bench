# Issue

**Title:** Speed up `plot_pareto_front` by replacing `O(N * |P|)` list lookup in `_get_non_pareto_front_trials`

## Description

## Motivation

In `optuna/visualization/_pareto_front.py`, `_get_non_pareto_front_trials(trials, pareto_trials)` filters out Pareto-optimal trials using `if trial not in pareto_trials:` inside a loop over all `trials`. Because `pareto_trials` is a `list[FrozenTrial]`, each membership check performs a linear scan comparing full `FrozenTrial` objects (`params`, `distributions`, `user_attrs`, `system_attrs`, `intermediate_values`, etc.), resulting in $O(N \cdot |P|)$ complexity where $N = \text{len(trials)}$ and $|P| = \text{len(pareto\_trials)}$.

## Description of the Changes

Precompute `pareto_trial_numbers = {trial.number for trial in pareto_trials}` once and filter by `trial.number not in pareto_trial_numbers`, reducing `_get_non_pareto_front_trials` from $O(N \cdot |P|)$ `FrozenTrial.__eq__` comparisons to $O(N + |P|)$ integer set lookups while preserving the exact output order and semantics.

### Micro-benchmark (`N = 3,000` trials, `|P| = 1,500` Pareto-front trials)

- **Before (`list` membership on `FrozenTrial`):** `431.70 ms`
- **After (`set` membership on `trial.number`):** `0.21 ms` (**~2,036x speedup**)

### Acknowledgments

- Credits to @semiexp (#3128, #3389), @nabenabe0928 (#5302), and @c-bata for designing and maintaining `optuna/visualization/_pareto_front.py`.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `c353e29912ea`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.