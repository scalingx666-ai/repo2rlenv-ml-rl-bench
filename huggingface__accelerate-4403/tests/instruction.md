# Issue

**Title:** Fsdp2 keep output dtype

## Description

# What does this PR do?

This PR updates the `output_dtype` of FSDPv2 to None (default value). So when using mixed precision, we return the loss in fp32 as we do usually when using mixed precision in accelerate. This makes training a bit more precise without any cost. 

## Testing
tiny Qwen2 model in fp32 and a batch of 4 × 1024 tokens

main: loss torch.bfloat16 | logits torch.bfloat16 | peak 8393 MiB | step 18.3 ms
fix:  loss torch.float32  | logits torch.bfloat16 | peak 8393 MiB | step 18.5 ms

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `5dd7661443e3`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.