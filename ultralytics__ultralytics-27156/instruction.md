# Issue

**Title:** Let CPU CI runners save and restore zstd-compressed Actions caches

## Description

The `cpu-latest` runner image (`ultralytics/ultralytics:latest-runner-cpu`, Ubuntu 24.04) built by [`docker/Dockerfile-runner-cpu`](https://github.com/ultralytics/ultralytics/blob/main/docker/Dockerfile-runner-cpu) ships no `zstd` binary. Without it, `actions/cache` compresses with `gzip`, and `gzip` is hashed into the cache version (`sha256("weights|datasets|<compression>|1.0")`), so a `gpu-latest` job (or GitHub-hosted runner) with `zstd` can never restore a cache that a `cpu-latest` runner saved with `gzip`, and vice versa.

This PR adds `zstd` to [`docker/Dockerfile-runner-cpu`](https://github.com/ultralytics/ultralytics/blob/main/docker/Dockerfile-runner-cpu#L29-L36)'s existing `apt-get install` line, matching  for [`docker/Dockerfile-runner`](https://github.com/ultralytics/ultralytics/blob/main/docker/Dockerfile-runner#L29-L30). Following  (which wired `GPU` in [`.github/workflows/ci.yml`](https://github.com/ultralytics/ultralytics/blob/main/.github/workflows/ci.yml#L429-L453) to restore [`.github/actions/ci-assets`](https://github.com/ultralytics/ultralytics/blob/main/.github/actions/ci-assets/action.yml)), CI workflows that build `Assets` on `cpu-latest` and restore them in `GPU` on `gpu-latest` fail at `actions/cache/restore@v6` (`fail-on-cache-miss: true`).

## Evidence

- **Missing binary.** Inside a live `cpu-latest` container (`ultralytics/ultralytics:latest-runner-cpu`), `zstd` is absent, so `actions/cache/save@v6` archives with `/usr/bin/tar --posix -cf cache.tgz ... -z` (`gzip`).
- **Version mismatch.** `actions/cache` computes the cache version as `sha256("weights|datasets|<compression>|1.0")`:

    | runner / image | compression | version | matches `gpu-latest`? |
    | --- | --- | --- | --- |
    | `gpu-latest` ([`Dockerfile-runner`](https://github.com/ultralytics/ultralytics/blob/main/docker/Dockerfile-runner#L30)) | `zstd-without-long` | `31753f701cbe24e6…` | yes |
    | `cpu-latest` ([`Dockerfile-runner-cpu`](https://github.com/ultralytics/ultralytics/blob/main/docker/Dockerfile-runner-cpu#L29)) | `gzip` (no `zstd`) | `513ed381dd596496…` | no, always a miss |

## Rollout

[`.github/workflows/docker.yml`](https://github.com/ultralytics/ultralytics/blob/main/.github/workflows/docker.yml) rebuilds `latest-runner-cpu` on push to `main`, so single-use `cpu-latest` runners pick up `zstd` automatically once published.

## 🛠️ PR Summary

<sub>Made with ❤️ by [Ultralytics Actions](https://www.ultralytics.com/actions)</sub>

### 🌟 Summary
Added `zstd` to the CPU runner image so its Actions caches use the compression format used by GPU runners.

### 📊 Key Changes
- Added `zstd` to the package installation in `docker/Dockerfile-runner-cpu`.
- Documented that this avoids the CPU runner falling back to `gzip`, which produces a cache version that does not match zstd-based runners.

### 🎯 Purpose & Impact
- CPU runners can save caches that GPU runners can restore without a compression-based cache version mismatch.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `fffaa093319c`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.