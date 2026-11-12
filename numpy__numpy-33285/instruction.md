# Issue

**Title:** BUG: ufunc.at: raise on a non-writeable first operand

## Description

### PR summary
<!-- Please take some time to make it easier for us to review
your pull request (PR). Describe the PR using the questions below as guidance.

- What problem does this PR solve? Link any relevant issues and PRs.
- Why are you interested in working on this PR?
- How does the proposed change help you?

Please keep in mind that even seemingly small changes can have unexpected consequences
for workflows unrelated to the ones you may be familiar with. For this reason, maintainers
carefully consider the risks of all changes, including bug fixes.

Make sure to review the guidelines for contributing code to NumPy (https://numpy.org/devdocs/dev/index.html#guidelines) and fill out the AI disclosure section below.

Please note that non-maintainers may only have 2 non-draft PRs open for review at a
time.

-->
- **What problem does this PR solve?**
  `ufunc.at` operations (such as `np.add.at` and `np.negative.at`) previously bypassed the array writeable flag check on their first operand. This allowed `ufunc.at` to silently write into read-only arrays (e.g., arrays with `writeable = False` or those backed by immutable objects via `np.frombuffer`), breaking NumPy's safety guarantees. This PR fixes the issue by introducing a call to `PyArray_FailUnlessWriteable` in `ufunc_at()`, ensuring that non-writeable arrays correctly raise a `ValueError` (consistent with `out=` and indexed assignment). Closes gh-33275.
- **Why are you interested in working on this PR?**
  A small, clear bug as a first contribution and to ensure consistency and correctness across all of NumPy.
- **How does the proposed change help you?**
  It prevents unexpected data corruption when working with read-only views or immutable buffer-backed arrays, protecting critical data integrity.

### First time contributor introduction
<!-- If you are new to the NumPy contributor community, please introduce yourself and
share how you use NumPy. It helps us get to know you, understand the context of your
contribution, and collaborate more effectively during the review process. 2-3
sentences is plenty, we are not looking for a work history or a measure of your
expertise.

-->
Hi! I am a software developer and regular user of NumPy for numerical and scientific computing with respect to Machine Learning. Looking forward towards contribute to improving NumPy's core robustness and safety guarantees.

#### AI Disclosure
<!-- If AI was used in the preparation of this PR, please disclose
the tool(s) used, how they were used, and specify what code or text is AI generated.
If no AI tools were used, please write "No AI tools used" in this section.

All interactions with reviewers must be performed by humans, including the submission of PRs.

Read the NumPy policy on AI generated code at https://numpy.org/devdocs/dev/ai_policy.html.

-->
AI assistant tools were used to help draft the bugfix implementation, write test cases for read-only validation. All code changes, verifications, and submissions are managed directly by me.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `31a42aed238c`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.