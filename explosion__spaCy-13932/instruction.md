# Issue

**Title:** Fix memory_zone cleanup on exception

## Description

## Summary

When an exception propagates out of `nlp.memory_zone()`, the cleanup code in `StringStore.memory_zone()` and `Vocab.memory_zone()` never runs. This leaves the vocab in a broken state (transient strings/lexemes not cleared, `self.mem` still pointing to a temporary pool), causing an `AssertionError` in `Vocab.get()` on the next call to `nlp()`.

## Root cause

Both `StringStore.memory_zone()` and `Vocab.memory_zone()` use `@contextmanager` with cleanup code placed after `yield` but without a `try/finally` wrapper. In a `@contextmanager`, code after `yield` only runs on normal exit. If an exception is thrown at the `yield` point, it propagates without executing the cleanup.

## Changes

- **`spacy/strings.pyx`**: Wrap `yield` and cleanup (clearing transient keys, restoring `self.mem`) in `try/finally`
- **`spacy/vocab.pyx`**: Same pattern for `_clear_transient_orths()` and `self.mem` restore
- **`spacy/tests/vocab_vectors/test_memory_zone.py`**: Add regression test that raises inside a memory zone and verifies the vocab is usable afterward

## Testing

- All existing `test_memory_zone` tests pass (no change to happy-path behavior)
- New `test_memory_zone_exception_cleanup` verifies:
  - `vocab.in_memory_zone` is `False` after exception
  - Pre-existing words still accessible
  - Transient words from the failed zone are cleaned up
  - Vocab accepts new words without errors

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `453732d32d55`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.