# Issue

**Title:** Fix batched map on a dataset with an index when the function returns a table

## Description

`cast()` and `cast_column()` fail on any dataset that has a FAISS or Elasticsearch index attached:

```python
import numpy as np
from datasets import Dataset, Value

ds = Dataset.from_dict({"id": list(range(5)), "v": np.eye(5, dtype=np.float32).tolist()})
ds.add_faiss_index("v")
ds.cast_column("id", Value("int32"))
# AttributeError: 'pyarrow.lib.Table' object has no attribute 'keys'
```

When indexes are attached, a batched `map()` checks that the function doesn't add or remove examples. The check reads the length of the first value with `processed_inputs.keys()`, so it assumes the function returned a dict. `cast()` maps `table_cast`, which returns a `pa.Table`, so it crashes there. The same happens for any batched map that returns a `pa.Table` or a polars DataFrame (both are accepted outputs), and an empty dict raises `RuntimeError: generator raised StopIteration`.

The fix takes `len()` of tables and DataFrames, keeps the first-column length for dicts, and skips the check for an empty dict since it doesn't update any column. A function that changes the number of examples still raises `DatasetTransformationNotAllowedError`.

Tests: added `test_batched_map_with_index` to `tests/test_search.py` (fails on main, passes with the fix). Ran `tests/test_search.py` (the two Elasticsearch tests fail the same way on main in my env because it has elasticsearch 9 installed) and `tests/test_arrow_dataset.py -k map` (56 passed). `make quality` passes.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `ef61228a5756`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.