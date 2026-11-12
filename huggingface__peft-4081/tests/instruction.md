# Issue

**Title:** ENH: Svd lowrank option for lora intruder dimension reduction

## Description

# Performance improvement through svd lowrank option

The idea is to
1. Pre-screen each layer with torch.svd_lowrank. 
2. Run the full SVD only on layers flagged as having intruders (so we don't introduce randomness in mitigation)

Therefore two arguments are added to `reduce_intruder_dimension`:
- `use_lowrank` with default:False
- `niter` (to tune lowrank)

### Performance exploration
**Model:** fine-tuned [`llava-hf/llava-1.5-7b-hf`](https://huggingface.co/llava-hf/llava-1.5-7b-hf) from [repo](https://github.com/causeri3/emo-llava)
| | `torch.compile` | `torch.svd_lowrank` |
|---|---|---|
| **Hardware** | Colab: T4 (15 GB GPU RAM), 2 CPU cores, 12.7 GB RAM | Mac: M3, 36 GB memory (took too long in Colab)|

*`torch.compile`*
would not be able to scrape much compute. Since 95% (CPU) to 98% (GPU) of compute is used for SVD operations, which are already optimised via cuSOLVER (LAPACK for CPU).

*`torch.svd_lowrank`* approach
In my tests this is up to 3x faster when few layers have intruders. It breaks even at 73% of layers and is up to 20% slower when every layer has intruders.
<img width="578" height="455" alt="Unbenannt" src="https://github.com/user-attachments/assets/5c10e9b5-89e8-4443-98e9-bb50d09953d6" />

Here the Colab [notebook](https://colab.research.google.com/drive/1cAxUA_ZoKLI_MEFuRQanHHUyintpsHGG?usp=sharing) for the full exploration with code which also imports this [repo](https://github.com/causeri3/emo-llava).

*Tests*
`pytest tests/test_lora_intruders.py -v `



This PR was authored by a human.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `21c63eea07ae`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.