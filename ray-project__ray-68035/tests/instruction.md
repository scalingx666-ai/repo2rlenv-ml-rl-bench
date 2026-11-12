# Issue

**Title:** [2.59.2][deps] Bump brotli to 1.2.1 in the large_image_embedding release lock

## Description

## Description

Follow-up to the #67789 pick (#68027) on `releases/2.59.2`. It bumps brotli from 1.1.0 to 1.2.1 in `release/ray_release/byod/large_image_embedding_py3.10.lock`, the last lock on this branch that still pins a brotli affected by CVE-2026-111497 ([GHSA-9v8h-qqgm-4m35](https://github.com/advisories/GHSA-9v8h-qqgm-4m35)).

**Middle of a 3-PR stack on `releases/2.59.2`:** #68027 (the #67789 pick), then this PR, then #68028 (the #67596 pick). Merge the stack with `gh stack merge 68028`, which lands all three in order.

This lock is the BYOD dependency set of the `large_image_embedding` release test in `release/release_multimodal_inference_benchmarks_tests.yaml`, whose `ray` variation runs nightly. The test image installs it with `uv pip install --system --no-deps -r`, so it would put brotli 1.1.0 back over the 1.2.1 that #68028 puts in the base images. #67789 bumps the other four py3.10 multimodal release locks but has no change for this one, because master removed this test in #65819 before #67789 merged.

## Related issues

, the #67789 pick. Advisory: GHSA-9v8h-qqgm-4m35 / CVE-2026-111497.

## Additional information

### How the lock was regenerated

I used the same approach #67789 used for the other four py3.10 multimodal locks. I added `--upgrade-package brotli` to the `large_image_embedding_py310` depset's `append_flags`, ran raydepsets, and then reverted the config, so this PR doesn't change it.

Brotli is the only package that changes. Its pin, hashes and `# via geventhttpclient` are identical to the `audio_transcription_py3.10.lock` entry that #68027 brings over from master.

After this PR, the only brotli pin below 1.2.1 on this branch is in `release/ray_release/byod/requirements_ml_byod_3.10.txt`. Nothing references that file, and master's #67789 leaves it at 1.1.0 too.

Not a duplicate: master and `releases/2.60.1` no longer have this lock, so there's no master commit to pick and no 2.60.1 counterpart. No PR against `releases/2.59.2` touches it.

### Testing

| Command | Result |
|---|---|
| `bazel run //ci/raydepsets:raydepsets -- build ci/raydepsets/configs/release_multimodal_inference_benchmarks_tests.depsets.yaml --name large_image_embedding_py310`, with `--upgrade-package brotli` added temporarily | Only `brotli` changes, 1.1.0 → 1.2.1 |
| `bazel run //ci/raydepsets:raydepsets -- build --all-configs --check`, with the config reverted | `Lock files are up to date.` |
| `pre-commit run --files release/ray_release/byod/large_image_embedding_py3.10.lock` | Passed |

I didn't run the `large_image_embedding` release test, which needs a GPU cluster.

AI assistance was used to prepare this PR.

🤖 Generated with [Claude Code](https://claude.com/claude-code)

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `09a98f4e2832`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.