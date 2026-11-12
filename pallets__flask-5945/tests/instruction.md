# Issue

**Title:** add zizmor to scan workflows

## Description

https://docs.zizmor.sh/

Among the findings:

- apply empty permissions with `permissions: {}`, although we have read only defaults set at the organization level
- use concurrency group for each workflow
- disable credentials for checkout
- remove some template variables in favor of env vars

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `a29f88ce6f2f`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.