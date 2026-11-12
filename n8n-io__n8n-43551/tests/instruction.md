# Issue

**Title:** ci: Run the affected unit, integration and DB tests in the merge queue (no-changelog)

## Description

## Summary

The merge queue tests each queued PR on top of the latest master. Since #43158, it runs build, format, typecheck and lint, but no tests. This PR makes the queue run the affected tests again: the unit tests, the backend and CLI integration suites, and the DB tests on SQLite and the primary Postgres version (18). The older Postgres versions (16 and 17) stay out of the queue.

In the queue, failed vitest tests retry 2 times (the `testRetries` input from #42525), and no coverage is collected. The DB test matrix gets a `queue` scope: the same legs as `pr`, without coverage. Otherwise the queue runs the tests like a PR run. PR runs and master runs do not change.

## Why

Without tests in the queue, two PRs that pass alone can break master together. On 2026-11-11, #43175 added a test that uses the old default OpenAI model, and #43407 changed that default. The queue run of #43407 passed on the same commit that then failed on master, because it ran no tests. Master stayed red until #43534.

#43158 removed the tests to save time and to stop flaky tests from removing PRs from the queue. The retries handle the flaky tests. The cost in time is real. These are critical-path times of queue runs, in minutes:

| Setup | Median | p90 |
| -- | -- | -- |
| Today, no tests (98 runs) | 4.8 | 8.1 |
| Before #43158, all tests (92 runs) | 10.8 | 13.2 |
| Before #43158, without Postgres 16 and 17 | 10.2 | 13.2 |

The CLI integration shards and the Postgres 18 jobs set the queue time: in the earlier runs they ended at about 9 min. Typecheck is faster now than before #43158, so the new queue time can be lower than the last row.

## How to test

- `node --test .github/scripts/db-test-matrix.test.mjs`
- `node .github/scripts/db-test-matrix.mjs --scope=queue` prints the SQLite and Postgres 18 jobs, all with `"collectCoverage":"false"`.
- Queue this PR. Its merge group run uses the new workflow: `Unit tests` runs without coverage, and `DB Tests` runs the SQLite and Postgres 18 jobs when DB files change.

## Related Linear tickets, Github issues, and Community forum posts

https://linear.app/n8n/issue/DEVP-1490

## Review / Merge checklist

- [x] I have seen this code, I have run this code, and I take responsibility for this code.
- [x] PR title and summary are descriptive. ([conventions](../blob/master/.github/pull_request_title_conventions.md))
- [ ] [Docs updated](https://github.com/n8n-io/n8n-docs) or follow-up ticket created.
- [x] Tests included.
- [ ] PR Labeled with `Backport to Beta`, `Backport to Stable`, or `Backport to v1` (if the PR is an urgent fix that needs to be backported)

🤖 Generated with [Claude Code](https://claude.com/claude-code)

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `5413bef3ad6d`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.