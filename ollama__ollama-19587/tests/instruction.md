# Issue

**Title:** launch: fix Claude Code routing and retired model errors

## Description

`ollama launch claude` can select an Ollama model while inherited provider flags or Claude settings route requests elsewhere. Keep routing scoped to the child process with Claude's host-managed provider control and clear competing provider selectors, preserving saved settings and explicit thinking/output preferences.

Also reject HTTP 410 in shared cloud-model readiness and return the server's retirement message, including any replacement advice, before launching a client.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `c053d1e98537`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.