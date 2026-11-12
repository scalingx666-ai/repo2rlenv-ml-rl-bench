# Issue

**Title:** [Flight] Fix hooks in streamed Server Components

## Description

Flight renders stream items and async iterator return values directly from promise callbacks. These paths can render Server Components after `performWork` restores the hook dispatcher, so supported hooks such as `useId` throw.

PR #38108 and PR #38114 set up hook context around individual stream rendering paths. This change establishes context at the common rendering entry points, which also cover asynchronous error causes in development.

`ReactFlightServer` and `ReactFlightHooks` already track the current request separately. Previously, `performWork` restored the server request but reset the hooks request to null. The shared helper restores the previous value in each module, along with the hook dispatcher. This preserves the enclosing render's hook context when an iterator thenable resolves synchronously.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `2363bc2b85ed`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.