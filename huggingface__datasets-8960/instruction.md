# Issue

**Title:** Fix batch(by_column=) default accumulation causing full downloads

## Description

It was using the default of `batch_size` which is `None`, which means full downloads

By default it now uses max batch size (1000)

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `62e0b421bb99`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.