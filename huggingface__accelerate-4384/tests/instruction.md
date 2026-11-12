# Issue

**Title:** Skip `_no_split_modules` classes the model doesn't contain when auto-wrapping

## Description

`TRANSFORMER_BASED_WRAP` without explicit `transformer_cls_names_to_wrap` raises as soon as the model's `_no_split_modules` names a class the model doesn't instantiate. That's common in transformers: Gemma 4 lists `Gemma4AudioLayer`, and checkpoints without an audio tower (e.g. `google/gemma-4-31B-it`) don't have one. Same for `EsmModel`, which lists the ESMFold blocks (#2947, closed as stale). This is the policy in TRL's FSDP configs and the Trainer default, so these models can't be sharded without listing the layers by hand.

`torchrun --nproc_per_node 2 repro.py`:

```python
import torch
from accelerate import Accelerator
from accelerate.utils import FullyShardedDataParallelPlugin
from transformers import AutoModelForImageTextToText

model = AutoModelForImageTextToText.from_pretrained("trl-internal-testing/tiny-Gemma4ForConditionalGeneration")
optimizer = torch.optim.AdamW(model.parameters())
fsdp_plugin = FullyShardedDataParallelPlugin(fsdp_version=2, auto_wrap_policy="transformer_based_wrap")
accelerator = Accelerator(fsdp_plugin=fsdp_plugin)
model, optimizer = accelerator.prepare(model, optimizer)
accelerator.print("OK")
```

```
ValueError: Could not find the transformer layer class Gemma4AudioLayer in the model.
```

This keeps only the `_no_split_modules` classes found in the model, in both the FSDP1 and FSDP2 paths. A class the user names explicitly still raises when missing.

New test (ESM, FSDP1 and FSDP2): fails on main, passes here.

@SunMarc

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `419d0e0c871f`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.