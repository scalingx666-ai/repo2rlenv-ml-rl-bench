# Issue

**Title:** [python] Dedupe training callbacks by identity so an unhashable callback works

## Description

The callback container builds `dict.fromkeys(callbacks)` to drop repeats, before it checks the types. A callback that is not hashable fails there. A `@dataclass` subclass is the common case, because `eq=True` sets `__hash__` to `None`:

```python
from dataclasses import dataclass
import numpy as np
import xgboost as xgb

@dataclass
class Counter(xgb.callback.TrainingCallback):
    calls: int = 0

d = xgb.DMatrix(np.random.rand(20, 2), np.random.rand(20))
xgb.train({}, d, 2, callbacks=[Counter()])
# TypeError: cannot use 'Counter' as a dict key (unhashable type: 'Counter')
xgb.train({}, d, 2, callbacks=[{}])
# the same error, instead of "callback must be an instance of `TrainingCallback`."
```

This checks the types first, then drops repeats by identity, so the same object listed twice still runs once and the order is kept.

The existing `test_callback_list` in `tests/python/test_callback.py` now uses two distinct equal dataclass callbacks and repeats one instance. It checks that both instances run once per round, that training preserves the caller’s list across repeated calls, and that an unhashable non-callback gets the callback type error. Against the released 3.4.1 package it fails with the error above; with this change it passes.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `ee6432316f56`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.