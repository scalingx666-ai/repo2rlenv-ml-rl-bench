# Issue

**Title:** fix(parallelism): don't build a pool when the lock is contended

## Description

A caller that found the pool lock held built a whole new rayon pool for one call. Now it waits out the holder (bounded) and falls back to serial.

N threads encoding a 16KB document (llama-3, M3 Pro), MB/s:

| callers | 1 | 4 | 12 | 32 |
|---|---|---|---|---|
| before | 899 | 3183 | 3388–3701 | 3924–4054 |
| after | 883 | 3493 | 5867–6206 | 9604–12536 |

🤖 Generated with [Claude Code](https://claude.com/claude-code)

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `851b23022df1`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.