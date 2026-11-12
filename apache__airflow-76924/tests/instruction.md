# Issue

**Title:** Copy the stack trace from the Dag import error modals

## Description

The copy button in the Dag import error modals copied the Dag file name, which is already shown next to it, instead of the error. It now copies the full import error message (the stack trace), which is what users want to paste into an issue or search. This covers both the dashboard's import errors list and the modal opened from a single Dag.

---

##### Was generative AI tooling used to co-author this PR?

- [x] Yes (please specify the tool below)

---

* Read the **[Pull Request Guidelines](https://github.com/apache/airflow/blob/main/contributing-docs/05_pull_requests.rst#pull-request-guidelines)** for more information. Note: commit author/co-author name and email in commits become permanently public when merged.
* For fundamental code changes, an Airflow Improvement Proposal ([AIP](https://cwiki.apache.org/confluence/display/AIRFLOW/Airflow+Improvement+Proposals)) is needed.
* When adding dependency, check compliance with the [ASF 3rd Party License Policy](https://www.apache.org/legal/resolved.html#category-x).
* For significant user-facing changes create newsfragment: `{pr_number}.significant.rst`, in [airflow-core/newsfragments](https://github.com/apache/airflow/tree/main/airflow-core/newsfragments). You can add this file in a follow-up commit after the PR is created so you know the PR number.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `e42b5a92aeb6`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.