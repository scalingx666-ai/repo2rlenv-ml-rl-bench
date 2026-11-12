# Issue

**Title:** BUG: to_numeric(errors='coerce') on ArrowDtype strings stored NaN ins…

## Description

- [x] 
- [x] [Tests added and passed](https://pandas.pydata.org/pandas-docs/stable/development/contributing_codebase.html#writing-tests) if fixing a bug or adding a new feature
- [x] All [code checks passed](https://pandas.pydata.org/pandas-docs/stable/development/contributing_codebase.html#running-the-performance-and-typing-test-suites).
- [x] Added [type annotations](https://pandas.pydata.org/pandas-docs/stable/development/contributing_codebase.html#type-annotations) to new arguments/methods/functions.
- [x] Added an entry in the latest `doc/source/whatsnew/v3.2.0.rst` file if fixing a bug or adding a new feature.
- [x] I have reviewed and followed all the steps in the [AI-assisted contributions guide](https://pandas.pydata.org/pandas-docs/stable/development/contributing.html#automated-contributions-policy). 

## Summary

`pd.to_numeric` with `errors="coerce"` on string data stored as `ArrowDtype` (e.g. the result of `convert_dtypes(dtype_backend="pyarrow")`) turned unparseable values into a stored float `NaN` instead of a missing value, so `isna()` did not detect them:

> >>> ser = pd.Series(["1", "x"]).convert_dtypes(dtype_backend="pyarrow")
> >>> pd.to_numeric(ser, errors="coerce")
> 0    1.0
> 1    NaN      # stored float NaN, isna() -> False
> dtype: double[pyarrow]

Worse, when the input already contained missing values, a successfully parsed value was duplicated into the position of a failed parse:

> >>> ser = pd.Series(["1", None, "x"]).convert_dtypes(dtype_backend="pyarrow")
> >>> pd.to_numeric(ser, errors="coerce", dtype_backend="pyarrow")
> 0       1
> 1    <NA>
> 2       1      # fabricated value
> dtype: int64[pyarrow]

## Root cause

In `to_numeric` (`pandas/core/tools/numeric.py`):

1. The `convert_to_masked_nullable` flag passed to `lib.maybe_convert_numeric` covered `StringDtype` with `NA` semantics but not `ArrowDtype`, so the converter returned float `NaN` and no mask, losing the positions of the failed parses.
2. The reconstruction step kept the original-null mask (full input length) and silently discarded the parse-failure mask (post-`dropna` length) when their shapes differed; NumPy broadcasting then spread the shortened values array over the non-null positions, fabricating the duplicated value above.

## Fix

- Include `ArrowDtype` in the `convert_to_masked_nullable` condition (and in the zero-mask fallback branch).
- When both masks exist with different lengths, merge the parse-failure mask back into the original positions (`mask[~mask] = new_mask`) before reconstructing the masked array.

After the fix:

> >>> pd.to_numeric(pd.Series(["1", None, "x"]).convert_dtypes(dtype_backend="pyarrow"), errors="coerce")
> 0       1
> 1    <NA>
> 2    <NA>
> dtype: int64[pyarrow]

## Tests

Four tests added to `pandas/tests/tools/test_to_numeric.py` covering: unparseable values becoming `NA`, the `"NaN"` string becoming a detectable missing value, merging with pre-existing missing values (with and without `dtype_backend="pyarrow"`), and the float case. `pandas/tests/tools/` passes in full (1764 passed).

## AI disclosure

This contribution was prepared with AI assistance, as required by the pandas automated contributions policy. The work was done by the contributor together with Muse Spark (Meta AI), an agent running in the Muse harness: the agent assisted with tracing the code path, drafting the fix and tests, and running the test suites. The contributor reproduced the bug, directed the investigation, reviewed the final diff, and takes responsibility for the change. Any text above quoted from the agent's output appears inside quotation blocks or code fences.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `0e93a6884748`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.