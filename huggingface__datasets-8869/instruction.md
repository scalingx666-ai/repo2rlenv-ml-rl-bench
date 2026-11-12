# Issue

**Title:** Unescape doubled quotes in GenBank qualifier values

## Description

The GenBank loader keeps the `""` escape inside quoted qualifier values. A wrapped line that ends in an escaped `""` also closes the value early, and a stray `"` lands at the end. The INSDC feature table (section 3.3.3.1) escapes each quotation mark in free text by doubling it. Its own example is `/note="This is an example of ""escaped"" quotation marks"`.

Repro with that example plus a wrapped value, `spec.gb`:

```
LOCUS       T                         10 bp    DNA     linear   PLN 01-JAN-2024
FEATURES             Location/Qualifiers
     gene            1..10
                     /note="This is an example of ""escaped"" quotation marks"
                     /product="wrapped after ""
                     quoted"" rest"
                     /gene="abc"
ORIGIN
        1 atcgatcgat
//
```

```
$ python repro.py spec.gb        # prints each qualifier of the first feature; main at 60c225f
note: This is an example of ""escaped"" quotation marks
product: wrapped after " quoted"" rest"
gene: abc
$ python repro.py spec.gb        # this branch
note: This is an example of "escaped" quotation marks
product: wrapped after " quoted" rest
gene: abc
```

Biopython 1.88 returns the branch output for the same file. This builds on the quote tracking from #7951. A quoted value now closes only on an odd run of trailing `"`, and the commit step turns `""` back into `"`. Unquoted values keep their text. A malformed value such as `"abc"def"` still closes at its line end, as before.

The cost: a dataset that somebody already converted to Parquet keeps the doubled quotes until they regenerate it.

### Verification

```
$ python -m pytest tests/packaged_modules/test_genbank.py tests/features/test_bio.py -q -n 4   # Python 3.12.6, pyarrow 26.0.0, biopython 1.88
119 passed in 2.49s      # main at 60c225f
121 passed in 2.43s      # this branch, two new tests
$ python -m pytest tests/packaged_modules/test_genbank.py -q -n 4   # genbank.py back at 60c225f, tests kept
2 failed, 71 passed in 2.37s
$ ruff check tests src benchmarks utils setup.py && ruff format --check tests src benchmarks utils setup.py   # ruff 0.17.1
All checks passed!
257 files already formatted
# not run: the full test suite outside these two modules
```

No issue exists for this.

@lhoestq, a docs note on the escapes would be the smaller change. I'd still take the code fix. The `record` column decodes through Biopython and already returns the unescaped text, so today the `features` column and the `record` column disagree on one file. Does that trade work for you?

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `60c225fe67d2`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.