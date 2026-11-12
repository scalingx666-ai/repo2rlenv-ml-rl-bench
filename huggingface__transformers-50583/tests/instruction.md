# Issue

**Title:** Support Enum in get_json_schema, raise on custom classes

## Description

<!-- ci-dashboard-badge:start -->
[![🤗 Transformers CI](https://img.shields.io/badge/%F0%9F%A4%97%20Transformers%20CI-333333?style=flat)](https://transformers-ci.huggingface.cool/d/pytest-observability-by-pr/pytest-observability-branch?var-pr=50583)
<!-- ci-dashboard-badge:end -->

`get_json_schema()` was supposed to raise on type hints it couldn't parse, like custom classes, but `_get_json_schema_type()` fell back to `{"type": "object"}` for anything it didn't recognize, so the error was unreachable. This meant hints like `Enum` classes, dataclasses or `datetime` all silently told the model to pass a dict.

This PR adds support for `Enum` classes, which become an `enum` of their member values, and makes other unknown classes raise as originally intended. This **is** a behaviour change, since tools with custom class hints (including return hints) used to get through, but I think a loud failure is better than a schema that's quietly wrong!

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `6488cabb56d4`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.