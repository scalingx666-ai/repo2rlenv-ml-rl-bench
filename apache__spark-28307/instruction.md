# Issue

**Title:** [SPARK-18886][CORE][FOLLOWUP] fix legacyResetOnTaskLaunch version and method docs

## Description

# What changes were proposed in this pull request?
fix version for config spark.locality.wait.legacyResetOnTaskLaunch
fix method return type doc

### Why are the changes needed?
Incorrect docs/versioning for config


### Does this PR introduce any user-facing change?
No


### How was this patch tested?
Tests in PR

@viirya thanks, for catching these. good 👀 
@cloud-fan 
@tgravescs

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `f543d6a1ee76`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.