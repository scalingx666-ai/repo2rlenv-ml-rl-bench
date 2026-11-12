# Issue

**Title:** Allow clip_grad_value_ under FSDP2

## Description

# What does this PR do?

`Accelerator.clip_grad_value_` raises for every FSDP run, FSDP2 included, although `torch.nn.utils.clip_grad_value_` already works on FSDP2's DTensor gradients. This PR lifts the exception for FSDP2 and keeps it for DeepSpeed and FSDP1.

The check dates from #405 (2022), before FSDP2 existed. Value clipping is elementwise, so clamping each rank's local shard gives exactly the clamp of the full gradient, with no collective needed (unlike `clip_grad_norm_`, whose FSDP2 path came in #4219).

## Usage

2 GPUs, FSDP2 (`fsdp_version=2`), `mixed_precision=no`, torch 2.14:

```python
import torch
from accelerate import Accelerator

accelerator = Accelerator()
model = torch.nn.Sequential(torch.nn.Linear(4, 4))
model, optimizer = accelerator.prepare(model, torch.optim.SGD(model.parameters(), lr=0.1))
accelerator.backward(model(torch.ones(2, 4, device=accelerator.device)).sum())  # every grad entry is 2
accelerator.clip_grad_value_(model.parameters(), 1.0)
print(max(p.grad.to_local().abs().max().item() for p in model.parameters()))
```

`main` raises `DeepSpeed and FSDP  do not support clip_grad_value_`; this branch prints `1.0` on both ranks.

## Tests

`FSDP2PluginIntegration.test_clip_grad_value`: one GPU, fp16 with a loss scale of 8. It fails on `main`, and fails if the gradients are not unscaled before clipping (0.5 would become 1).

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `3008af926ffe`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.