# Issue

**Title:** Fix batch(by_column=) merging groups that are separated by null values

## Description

## Summary

`Dataset.batch(by_column=...)` and `IterableDataset.batch(by_column=...)` treated a null key as equal to its neighbours, so different groups were merged into one batch. Keys `[1, None, 2]` gave a single batch. A key column with only nulls raised `ArrowNotImplementedError`. Now a null is its own value: equal to other nulls, different from any non-null value.

## Root cause

`_batch_accumulate_arrow_table_by_columns` in `src/datasets/table.py` finds group boundaries with `pc.not_equal`. That returns null when either side is null, and `pc.indices_nonzero` reads null as "no boundary". The same happens in the check that decides whether to keep accumulating across chunks. `pc.not_equal` also has no kernel for the Arrow null type, which is what an all-null column gets.

## Changes

- Add `_not_equal_nulls_equal` in `table.py`: `not_equal` with nulls filled as False, OR'd with `xor(is_null(left), is_null(right))`. For null-typed inputs it returns only the null check.
- Use it for the boundary computation and the accumulation check. Both `Dataset.batch` and `IterableDataset.batch` go through this function, so both are fixed.
- Add `test_dataset_batch_by_column_with_nulls`, covering `Dataset` and `to_iterable_dataset()`, with `batch_size=None` and `batch_size=2` (accumulation across chunks).

## Reproduction

```python
from datasets import Dataset

ds = Dataset.from_dict({"id": [0, 1, 2], "key": [1, None, 2]})
[b["id"] for b in ds.batch(by_column="key")]
# before: [[0, 1, 2]]
# after:  [[0], [1], [2]]

ds = Dataset.from_dict({"id": [0, 1], "key": [None, None]})
[b["id"] for b in ds.batch(by_column="key")]
# before: ArrowNotImplementedError: Function 'not_equal' has no kernel matching input types (null, null)
# after:  [[0, 1]]
```

## Testing

- `python -m pytest tests/test_arrow_dataset.py -m unit -k batch_by_column_with_nulls -v`: 4 failed before the fix, 4 passed after.
- `python -m pytest tests/test_arrow_dataset.py -m unit -v`: 464 passed.
- `python -m pytest -m unit tests/test_table.py tests/test_arrow_dataset.py tests/test_iterable_dataset.py -n 2 --dist loadfile`: 1441 passed, 1 skipped.
- `python -m pytest -m unit -n 2 --dist loadfile ./tests/`: 3781 passed, 16 skipped. 11 failed and 1 error, the same set as on main without this change. All of them need network access (Hub, tiktoken encodings, a geoparquet URL) that my environment blocks.
- `make quality`: passes.

## Compatibility notes

No signature or default changes. Keys without nulls give the same batches as before. Only inputs with null keys change: they were merged into neighbouring groups, or raised for an all-null column.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `a1269ca9dbb6`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.