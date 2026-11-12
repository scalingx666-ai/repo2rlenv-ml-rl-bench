# Issue

**Title:** Fix loading checkpoints with modules offloaded to disk

## Description

`load_checkpoint_and_dispatch` with some modules mapped to `"disk"` gives wrong outputs when those modules have buffers, for example BatchNorm running stats:

```python
import os, tempfile, torch, torch.nn as nn
from accelerate import init_empty_weights, load_checkpoint_and_dispatch

class Block(nn.Module):
    def __init__(self):
        super().__init__()
        self.lin, self.bn = nn.Linear(8, 8), nn.BatchNorm1d(8)
    def forward(self, x):
        return self.bn(self.lin(x))

class M(nn.Module):
    def __init__(self):
        super().__init__()
        self.a, self.b = Block(), Block()
    def forward(self, x):
        return self.b(self.a(x))

ref = M().eval()
with torch.no_grad():
    ref.b.bn.running_mean.uniform_(-1, 1)
    ref.b.bn.running_var.uniform_(0.5, 2)
tmp = tempfile.mkdtemp()
torch.save(ref.state_dict(), os.path.join(tmp, "model.pt"))

with init_empty_weights():
    model = M()
model = load_checkpoint_and_dispatch(
    model, os.path.join(tmp, "model.pt"), device_map={"a": "cpu", "b": "disk"}, offload_folder=tmp
).eval()
x = torch.randn(4, 8)
print((model(x) - ref(x)).abs().max())  # nonzero on main (0.8 in one run), 0 with this PR
```

In `load_checkpoint_in_model`, the `"disk"` branch only handles a tensor when `offload_buffers or param_name not in buffer_names`. With the default `offload_buffers=False`, a buffer of a disk-mapped module matches neither case, so its checkpoint value is dropped: it is not offloaded and not loaded, and the module keeps the value it was created with (`dispatch_model` then keeps buffers in memory, as intended). This dates back to #951; its test reloads a model's own state dict, so the values were already equal and only the devices were checked. The fix loads such buffers on the CPU, as the non-disk branch does for in-memory tensors.

While testing the same path with a `.safetensors` checkpoint I hit a second problem: when every module is mapped to disk (`device_map={"": "disk"}`), `load_state_dict` takes the single-device shortcut and passes `"disk"` to `safe_load_file`, which raises `SafetensorError: device disk is invalid`. The multi-device branch already reads disk weights on the CPU; the shortcut now does the same.

### Tests

* `test_load_checkpoint_in_model_disk_offload_loads_buffers`: loads a checkpoint whose BatchNorm stats differ from a fresh model's into a fresh model with the BatchNorm on disk, and checks the values.
* `test_load_checkpoint_in_model_all_disk_safetensors`: `device_map={"": "disk"}` with a safetensors checkpoint.

Both fail on main and pass here. `tests/test_modeling_utils.py` and `tests/test_big_modeling.py` otherwise give the same results with and without the change (4 `infer_auto_device_map` buffer-check tests fail on both in my CPU environment). I also compared outputs against the unmodified model for `cpu_offload`, `disk_offload` and `dispatch_model` (with and without `offload_buffers`) and for `load_checkpoint_and_dispatch` with four CPU/disk maps, including a non-persistent buffer and tied weights: all match with this change, and only the `load_checkpoint_and_dispatch` maps that put buffers on disk differed before it. `ruff` 0.13.1 is clean.

I used Claude Code to help find this and draft the description, and reviewed the change and the results above myself.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `421af8bee5bd`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.