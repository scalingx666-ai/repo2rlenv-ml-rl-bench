# Issue

**Title:** Preserve active iterators when reconfiguring threaded decoding

## Description

Creating a new decoding view from a threaded `IterableDataset` currently shuts down the original dataset's decoding pool. If an existing iterator has prefetched examples queued behind active decoders, calling `ds.decode(False)` or changing its thread count cancels those examples. Continuing the original iterator then raises `asyncio.CancelledError`, even if the new view has never been iterated.

Keep the original pool alive when unwrapping the decoding layer to construct the new dataset. Add an event-synchronized regression with two occupied workers and queued decoding work; it checks that the original iterator completes in order and that the new view has the requested decoding behavior, for both disabling decoding and changing the thread count.

Validation on Windows / Python 3.12.14:

- Both new regression cases fail with `CancelledError` on unchanged main `18378f7` and pass with this patch.
- `python -m pytest tests/test_iterable_dataset.py -k 'test_decode or mapped_examples_iterable' -q`: **82 passed**.
- `ruff check` and `ruff format --check` pass for both changed files.
- A separate local control with six real PNG examples verified preserved decoded pixels/raw bytes and collection of the discarded original pool, followed by both finite decoder workers exiting.

The wider selected test run also printed a pending-task destruction diagnostic; the two new regression cases run in isolation did not. Full-suite, remote-media and blocked-I/O cleanup behavior were not validated here.

AI-assisted implementation and review; the listed local checks were executed.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `18378f7322c7`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.