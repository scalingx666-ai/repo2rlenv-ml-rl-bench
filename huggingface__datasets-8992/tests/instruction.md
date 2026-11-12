# Issue

**Title:** Fix IterableDataset state_dict freezing when a sibling dataset is created

## Description

Datasets derived from the same dataset (e.g. `ds.skip(n)` and `ds.take(n)`) share its ex_iterables. Creating or iterating one of them runs `_prepare_ex_iterable_for_iteration()`, which calls `_init_state_dict()` on the shared nodes and replaces their state dicts. The other dataset's running iterator then keeps updating a dict its `state_dict()` no longer points at, so its checkpoint stops advancing and a resume replays examples.

This PR shallow-copies the ex_iterables tree at the start of `_prepare_ex_iterable_for_iteration()`, so each iteration owns the state dicts it updates. It walks the tree the same way `shift_ex_examples_rngs` already does for `epoch > 0`; the default `epoch == 0` path had no copy at all.

The new test `test_iterable_dataset_state_dict_not_frozen_by_sibling_dataset` covers a Python generator source and an Arrow source, with the sibling dataset either created or iterated mid-iteration. All four cases fail on main and pass with the fix.

Tests run locally (macOS arm64, Python 3.12, torch 2.15 CPU): `tests/test_iterable_dataset.py`, `tests/test_distributed.py` and the iterable/state/resume/dataloader tests in `tests/test_arrow_dataset.py` give 701 passed. The 4 `test_torch_distributed_run*` tests fail with the same asyncio error on main without this change, so they look environment-related. `ruff check` and `ruff format --check` pass.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `381faf51417e`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.