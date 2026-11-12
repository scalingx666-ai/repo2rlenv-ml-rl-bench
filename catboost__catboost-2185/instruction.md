# Issue

**Title:** add catboos.Pool creation benchmark

## Description

Before submitting a pull request, please do the following steps:

1. Read [instructions for contributors](https://catboost.ai/docs/concepts/development-and-contributions.html).
2. Run `ya make` in catboost folder to make sure the code builds.
3. Add tests that test your change.
4. Run tests using `ya make -t -A` command.
5. If you haven't already, complete the [CLA](https://yandex.ru/legal/cla/).

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `e7ae760c13fa`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.