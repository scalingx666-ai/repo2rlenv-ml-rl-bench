# Issue

**Title:** CI: Better slow test report

## Description

Our test CI (the one running for each PR) is quite slow, taking up to 30 min at the time of writing. This is hard to improve because we lack visibility into which tests are actually slow. We do have `--durations=10` in the pytest invocation, but that is only of limited usefulness because many of our tests are heavily parametrized, so even if each individual test is fast, in aggregate they can be quite slow.

This PR stores the pytest logs (junit) and then runs a small script on the results to report not only the individually slowest tests, but also the slowest tests aggregated over the parametrizations. The test report now happens in a different stage ("Report slowest tests"), which also declutters the pytest log results.

The PR was written with the help of Codex (GPT 6.1 Sol).

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `593449b125f0`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.