# Issue

**Title:** [ci] fix 'docker' workflows on 'main' and PRs from forks

## Description

#6638 added CI jobs building the images in `docker/`.

Those Dockerfiles `git clone` LightGBM and I wanted to explicitly test that that's working (instead of, say, using `actions/checkout` and mounting in files). I made some mistakes in how I set up the `git` references there... long story short, right now that workflow only works in PR CI on PRs from branches in this repo.

This updates it to also support:

* pushes to `main`
* PRs from forks

## Notes for Reviewers

### How I tested this

CI on this PR seems to be setting the right values:

```text
  docker build \
    -t lightgbm:cli \
    --build-arg LIGHTGBM_GIT_REF="${LIGHTGBM_GIT_REF}" \
    --build-arg LIGHTGBM_GIT_URL="${LIGHTGBM_GIT_URL}" \
    - < ./docker/dockerfile-cli
  shell: /usr/bin/bash -e {0}
  env:
    LIGHTGBM_GIT_REF: 5a7b26c49272979cb22fad2531f7c1140689ca62
    LIGHTGBM_GIT_URL: https://github.com/lightgbm-org/LightGBM.git
```

([build link](https://github.com/lightgbm-org/LightGBM/actions/runs/40264036947/job/121382714030?pr=7505))

And on a PR from a fork (#7506)

```text
  docker build \
    -t lightgbm:cli \
    --build-arg LIGHTGBM_GIT_REF="${LIGHTGBM_GIT_REF}" \
    --build-arg LIGHTGBM_GIT_URL="${LIGHTGBM_GIT_URL}" \
    - < ./docker/dockerfile-cli
  shell: /usr/bin/bash -e {0}
  env:
    LIGHTGBM_GIT_REF: 92483740e056f630b4eed31a631191a9bc905c8b
    LIGHTGBM_GIT_URL: https://github.com/jameslamb/LightGBM.git
```

([build link](https://github.com/lightgbm-org/LightGBM/actions/runs/40264376310/job/121383417901?pr=7506))

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `d7d70d7b82fa`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.