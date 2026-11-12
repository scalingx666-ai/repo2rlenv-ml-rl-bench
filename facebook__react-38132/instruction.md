# Issue

**Title:** Fix: Infinite loop caused by uDV + suspended hydration or retry

## Description

When a render that carries no updates (hydration, or a Suspense retry) mounts useDeferredValue with an initial value and then suspends, React re-attempts the suspended work in a loop at scheduler speed until its data arrives. Next.js App Router pages hit this whenever hydration suspends, because the router passes an initial value to useDeferredValue.

1. The render requests a deferred lane for the final value, then suspends. Because the render didn't commit, the only way the deferred task can redo the work is through entanglement: it's entangled with the suspended render's update lanes, so rendering it reprocesses those updates. Hydration and retry lanes aren't update lanes, so nothing gets entangled.
2. The deferred task renders, finds no work, and commits an empty tree.
3. Every commit clears the root's suspended lanes, so React attempts the suspended hydration or retry again. It suspends again, spawns another deferred lane, and so on.

For retries, this only happens when retries are throttled. Otherwise the retry commits the fallback, and the deferred lane spawned by that commit has real work to do. This is a separate loop from the one fixed in #37739.

The fix: when the suspended lanes include no update lanes, don't spawn the deferred lane. It had nothing to do anyway. The deferred value is requested again when the suspended work is retried, and the commit that finishes it spawns the deferred task as usual.

This doesn't affect useDeferredValue's ability to skip an initial value that suspends and switch to the final one. That relies either on entanglement with update lanes or on a deferred task spawned at commit, and neither changes.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `5b6109643a00`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.