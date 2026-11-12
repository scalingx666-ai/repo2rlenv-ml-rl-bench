# Issue

**Title:** Fix indices buffer loading in Dataset.from_buffer

## Description

`Dataset.from_buffer` reads the data buffer twice when `indices_buffer` is provided. this raises a type error for string columns and can silently return the wrong rows for uint64 data. Read the supplied indices buffer instead, with 8 regression cases covering empty, reordered and repeated indices, plus the no index case.

Validation: all 8 regression cases pass (6 fail before the fix). The broader Arrow dataset/table run has 664 passes and 50 skips. the same 14 multiprocessing tests fail on unchanged main because this environment blocks Unix sockets. `make quality` passes.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `18378f7322c7`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.