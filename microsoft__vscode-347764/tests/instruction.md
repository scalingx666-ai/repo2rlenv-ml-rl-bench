# Issue

**Title:** agentHost: discover SSH hosts through quoted Include paths

## Description

<!-- Thank you for submitting a Pull Request. Please:
* Read our Pull Request guidelines:
  https://github.com/microsoft/vscode/wiki/How-to-Contribute#pull-requests
* Associate an issue with the Pull Request.
* Ensure that the code is up-to-date with the `main` branch.
* Include a description of the proposed changes and how to test them.
-->

Fixes SSH hosts missing from the Agents Window picker when their configuration is reached through a quoted `Include` path. Discovery retained quote characters and split paths containing spaces.

Parse Include arguments using OpenSSH's quoting, limited escapes, and comment boundaries. Keep known-hosts parsing, path resolution, recursion, and glob traversal unchanged. Malformed Include arguments are skipped with a warning.

Adds regression coverage for quoted paths, multiple paths, escapes, literal `#`, Windows path syntax, nested Includes, globs, and trailing comments.

The client build (`fnm exec --using=24.22.1 pnpm run build-fast --client-only`) passed. Both affected test files passed with 137 tests after merging current main: `fnm exec --using=24.22.1 ./scripts/test.sh --run src/vs/platform/agentHost/test/common/sshConfigParsing.test.ts --run src/vs/platform/agentHost/test/node/sshRemoteAgentHostService.test.ts --reporter dot`.

Also compared discovery with native `ssh -G -F` across 11 isolated Include fixtures; all agreed. No real SSH configuration or connections were used.

(Written by Copilot)

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `277e7a565da1`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.