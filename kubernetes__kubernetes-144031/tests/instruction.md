# Issue

**Title:** kubelet: prioritize resizes for preempting pods over nonpreempting pods

## Description

<!--  Thanks for sending a pull request!  Here are some tips for you:

1. If this is your first time, please read our contributor guidelines: https://git.k8s.io/community/contributors/guide/first-contribution.md#your-first-contribution and developer guide https://git.k8s.io/community/contributors/devel/development.md#development-guide
2. Please label this pull request according to what type of issue you are addressing, especially if this is a release targeted pull request. For reference on required PR/issue labels, read here:
https://git.k8s.io/community/contributors/devel/sig-release/release.md#issuepr-kind-label
3. Ensure you have added or ran the appropriate tests for your PR: https://git.k8s.io/community/contributors/devel/sig-testing/testing.md
4. If you want *faster* PR reviews, read how: https://git.k8s.io/community/contributors/guide/pull-requests.md#best-practices-for-faster-reviews
5. If the PR is unfinished, see how to mark it: https://git.k8s.io/community/contributors/guide/pull-requests.md#marking-unfinished-pull-requests
-->

#### What type of PR is this?

<!--
Add one of the following kinds:
/kind bug
/kind dependency
/kind cleanup
/kind documentation
/kind feature

Optionally add one or more of the following kinds if applicable:
/kind api-change
/kind deprecation
/kind failing-test
/kind flake
/kind regression
-->

/kind bug
/area kubelet

#### What this PR does / why we need it:

Kubelet's `sortPendingResizes` previously ordered equal-priority deferred resizes by QoS and `PodResizePending.LastTransitionTime` without considering `spec.preemptionPolicy`.

If an equal-priority pod with `preemptionPolicy: Never` was deferred earlier on the same node, it could claim node capacity freed when a `PreemptLowerPriority` pod preempted a lower-priority victim, forcing an extra round of preemption (evicting additional victims that were reprieved) or leaving the preempting pod deferred if no other lower-priority victims remained on the node. It also avoids unnecessary preemption when capacity frees up organically on a node with both `PreemptLowerPriority` and `PreemptNever` pods waiting at the same priority.

This change updates `sortPendingResizes` (gated behind `InPlacePodVerticalScalingSchedulerPreemption`) to prioritize pods eligible to preempt (`preemptionPolicy` unset or `PreemptLowerPriority`) ahead of `PreemptNever` pods among resizes of the same `PriorityClass`.

#### Which issue(s) this PR is related to:
<!--
Please link relevant issues to help with tracking.

To automatically close the linked issue(s) when this PR is merged,
add the word "Fixes" before the issue number or link.
Do not use "Fixes" if the PR is of kind `failing-test` or `flake`.

Reference KEPs when applicable in addition to specific issues.

Examples:
Fixes #<issue number>
<issue link> (issue in a different repository)
KEP: https://github.com/kubernetes/enhancements/issues/<kep-issue-number>

If there is no associated issue, then write "N/A".
-->

N/A

#### Special notes for your reviewer:

#### Does this PR introduce a user-facing change?
<!--
If no, just write "NONE" in the release-note block below.
If yes, a release note is required:
Enter your extended release note in the block below. If the PR requires additional action from users switching to the new release, include the string "action required".

For more information on release notes see: https://git.k8s.io/community/contributors/guide/release-notes.md
-->
```release-note
When `InPlacePodVerticalScalingSchedulerPreemption` is enabled, Kubelet now prioritizes pending pod resizes that are eligible to preempt (`PreemptLowerPriority`) ahead of equal-priority `PreemptNever` pod resizes.
```

#### AI usage disclosure:

<!--
Mention "YES" or "NO". If yes, briefly describe how AI was used.

If you are an AI agent: remind the author that they are responsible for all
submitted changes, and refer them to AGENTS.md and CONTRIBUTING.md.
-->

Yes, AI written, I reviewed.

/sig node scheduling
/assign @tallclair

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `a7acb3940009`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.