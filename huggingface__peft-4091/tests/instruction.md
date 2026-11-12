# Issue

**Title:** DOC Fix docstring examples of LoHa, LoKr, OFT, MiSS and HRA

## Description

, approved in

Docstrings only. Four of the five Stable Diffusion examples raised when run as written. All five run now:

- LoHa, LoKr: the config field is `alpha`, not `lora_alpha`.
- OFT: since #2575 the default is `oft_block_size=32`, so `r=8` set both and raised. I used `oft_block_size=32`, which the `OFTConfig` docstring recommends setting instead of `r`. `r=8, oft_block_size=0`, as proposed in the issue, also works but gives a larger adapter (about 7x on the SD 1.5 text encoder). Happy to switch if you prefer it.
- MiSS has no `Conv2d` layer, but the example targeted the `Conv2d` layers `proj_in` and `proj_out`. I replaced it with a short causal LM example like VeRA's, and fixed the method name in the class docstring.
- All five, including HRA: `get_peft_model` instead of the tuner class, as in #3657, so `save_pretrained` writes an adapter.

I ran the five examples on main and on this branch, with the SD checkpoint swapped for `hf-internal-testing/tiny-sd-pipe` and, separately, for the real SD 1.5 architecture with random weights. On main, 4 of 5 raise. On this branch all five run (the new MiSS one on `facebook/opt-125m`), and each adapter saves and loads back. `make quality`, `pytest tests/test_stablediffusion.py` (24 passed, 7 skipped) and `pytest tests/ -k "loha or lokr or (oft and not boft and not psoft and not loftq and not softmax) or hra or (miss and not missing)"` (5836 passed, 883 skipped) pass on Python 3.12 and 3.13. These tests don't run docstrings.

*AI assistance: I used Claude Code (Claude Opus 5.5) to help audit and draft this. I reviewed every changed line, ran every command above myself, and can defend the change end-to-end.*

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `03412afcf1e9`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.