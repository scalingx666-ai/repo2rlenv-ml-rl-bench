# Issue

**Title:** Fix `search_issues` trace count failing on SQL Server (error 8120)

## Description

> [!NOTE]
> Created by [automated triage](https://github.com/mlflow/mlflow/actions/runs/43022149633/attempts/1). Please review carefully.

### Related Issues/PRs

### What changes are proposed in this pull request?

`SqlAlchemyStore.search_issues(include_trace_count=True)` fails on SQL Server with error 8120, which breaks the experiment Issues tab. The trace count used `LEFT OUTER JOIN assessments ... GROUP BY issues.issue_id` while selecting every `issues` column. SQLite, PostgreSQL and MySQL accept that because `issue_id` is the primary key, but SQL Server doesn't support functional dependency in `GROUP BY`.

This PR computes the count with a correlated scalar subquery instead, so the query needs no join and no `GROUP BY`. The default ordering by trace count, `order_by` keyset pagination and `query.count()` keep working unchanged.

![The trace count query uses a GROUP BY that SQL Server rejects; a correlated subquery avoids it](https://github.com/user-attachments/assets/835783ec-bfc5-4990-825b-0cceb29b9ac2)

### How is this PR tested?

- [x] Existing unit/integration tests
- [x] New unit/integration tests
- [x] Manual tests

- Added `test_search_issues_with_trace_count` to `tests/db/test_tracking_operations.py`. It covers the Issues tab call (`include_trace_count`, `include_total_count`, `statuses`, `name`) and `order_by=["name ASC"]`. CI's `tests/db` jobs run it against SQL Server, PostgreSQL and MySQL. Locally it passes on SQLite; it couldn't run against SQL Server in the sandbox.
- Compiled the query with the `mssql` dialect before and after the change: `GROUP BY issues.issue_id` is gone after the fix.
- `uv run pytest tests/store/tracking/sqlalchemy_store/ -k issue`: 257 passed.

### Does this PR require documentation update?

- [x] No.

### Does this PR require updating the [MLflow Skills](https://github.com/mlflow/skills) repository?

- [x] No.

### Release Notes

#### Is this a user-facing change?

- [x] Yes. Give a description of this change to be included in the release notes for MLflow users.

Fixed `search_issues` with trace counts (used by the Issues tab) failing on SQL Server with error 8120.

#### What component(s), interfaces, languages, and integrations does this PR affect?

Components

- [x] `area/tracking`: Tracking Service, tracking client APIs, autologging

<a name="release-note-category"></a>

#### How should the PR be classified in the release notes? Choose one:

- [x] `rn/bug-fix` - A user-facing bug fix worth mentioning in the release notes

#### Is this PR a critical bugfix or security fix that should go into the next patch release?

<details>
<summary>What is a minor/patch release?</summary>

- Minor release: a release that increments the second part of the version number (e.g., 1.2.0 -> 1.3.0).
  Minor releases are expected to contain larger changes, such as new features and improvements. Non-critical bug fixes and doc updates can be included as well. By default, your PR should target the next minor release.
- Patch release: a release that increments the third part of the version number (e.g., 1.2.0 -> 1.2.1).
  Patch releases are typically only performed when there has been a major regression or bug in the latest release. For the sake of stability, your PR should not be included in a patch release unless it is a critical fix, or if the risk level of your PR is exceedingly low.

</details>

- [ ] This PR is critical and needs to be in the next patch release
- [ ] This PR can wait for the next minor release

🤖 Generated with [Claude Code](https://claude.com/claude-code)

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `781f028e82d1`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.