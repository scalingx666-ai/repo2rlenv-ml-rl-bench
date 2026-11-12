# Issue

**Title:** Fix IterableDataset.rename_columns with arrow, pandas and polars formats

## Description

`IterableDataset.rename_column` / `rename_columns` crash during iteration when the dataset has a table format (`arrow`, `pandas`, `polars`).

```python
from datasets import IterableDataset

ds = IterableDataset.from_dict({"a": [1, 2, 3], "b": ["x", "y", "z"]})
list(ds.with_format("arrow").rename_column("a", "c"))
# TypeError: unhashable type: 'pyarrow.lib.ChunkedArray'
list(ds.with_format("pandas").rename_column("a", "c"))
# TypeError: Provided `function` which is applied to pandas dataframe returns a variable of type <class 'dict'>.
# Make sure provided `function` returns a pandas dataframe to update the dataset.
```

Expected: tables/DataFrames with columns `["c", "b"]`, the same way `select_columns`, `remove_columns` and `cast_column` already work with these formats.

Cause: `rename_columns` is implemented as `self.map(_rename_columns_fn, remove_columns=...)`, and `_rename_columns_fn` assumes a dict input and returns a dict of only the renamed columns, to be merged with the input. With a table format, `map` passes the function a `pa.Table`/DataFrame and expects a table back that replaces the input (no merge). So the dict-based function can't work there.

Fix: when the format is a table format, `rename_columns` now runs the map in arrow format with `batched=True` and restores the original format afterwards, which is the approach `IterableDataset.batch()` already uses. `_rename_columns_fn` handles a `pa.Table` input by renaming the columns in place with `pa.Table.rename_columns`, keeping the same missing/existing column checks. The non-table path is unchanged.

Tests: added `test_iterable_dataset_rename_columns_with_table_format` (arrow and pandas) in `tests/test_iterable_dataset.py`. It fails on `main` with the two errors above and passes with the fix. I also checked polars and resuming from `state_dict()` by hand. `tests/test_iterable_dataset.py` passes except 4 `test_interleave_dataset_with_sharding` cases that need torch, which isn't installed in my env (Windows 11, Python 3.11, pyarrow 26.0.1, pandas 3.1.1). `make quality` passes.

`IterableDataset.add_column` fails the same way with table formats. It needs its own arrow implementation, so I left it out of this PR.

🤖 Generated with [Claude Code](https://claude.com/claude-code)

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `a1269ca9dbb6`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.