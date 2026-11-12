# Issue

**Title:** Fixed #37469 -- Made psycopg adapter UTF8Dumper handle None

## Description

#### Trac ticket number

ticket-37469

#### Branch description

The adapter for psycopg2 serialized `None` to SQL `NULL` [1]. This behavior should be maintained for psycopg3, as `psql_escape` returns `None` when no value remain after escaping and stripping.

Psycopg3 `Dumper` knows how to `quote` `None`: https://github.com/psycopg/psycopg/blob/4daf632fae52ad7493c76fdea11d3a65856df114/psycopg/psycopg/adapt.py#L59-L60.

Thanks to Camille Daniel for the initial bug report.

https://github.com/psycopg/psycopg2/blob/7d0375492ec3707b82b89f632c9654b400466d86/psycopg/adapter_asis.c#L42-L45

#### AI Assistance Disclosure (REQUIRED)
- [x] **No AI tools were used** in preparing this PR.

#### Checklist
- [x] This PR follows the [contribution guidelines](https://docs.djangoproject.com/en/stable/internals/contributing/writing-code/submitting-patches/).
- [x] This PR **does not** disclose a security vulnerability (see [vulnerability reporting](https://docs.djangoproject.com/en/stable/internals/security/)).
- [x] This PR targets the `main` branch. <!-- Backports will be evaluated and done by mergers, when necessary. -->
- [x] The commit message is written in past tense, mentions the ticket number (if applicable), and ends with a period (see [guidelines](https://docs.djangoproject.com/en/dev/internals/contributing/committing-code/#committing-guidelines)).
- [x] I have not requested, and will not request, an automated AI review for this PR. <!-- You are welcome to do so in your own fork. -->

<!-- Leave the following items unchecked if not applicable. -->
- [x] I have checked the "Has patch" ticket flag in the Trac system.
- [x] I have added or updated relevant tests.
- [ ] I have added or updated relevant docs, including release notes if applicable.
- [ ] I have attached screenshots in both light and dark modes for any UI changes.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `2382f40aed50`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.