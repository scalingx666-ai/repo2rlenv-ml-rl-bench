# Issue

**Title:** fix(cli): list the lts channel in the deno upgrade usage

## Description

## Summary

`deno upgrade` accepts `lts` as a release channel, but the usage block that `deno
upgrade` prints when it rejects a version or hash does not list it, so the only
user-facing enumeration of channels is incomplete.

- `cli/lib/shared.rs:75` — `ReleaseChannel::deserialize` maps `"lts" => Self::Lts`
  (and `:58` serializes `Self::Lts => "lts"`).
- `cli/tools/upgrade.rs:1236` — `RequestedVersion::from_upgrade_flags` tries
  `ReleaseChannel::deserialize(&val.to_lowercase())` on the positional
  `version_or_hash_or_channel`, so `deno upgrade lts` resolves to
  `Latest(ReleaseChannel::Lts)`.
- `cli/tools/upgrade.rs:391-397` — the update notification for an LTS release tells the
  user "Run `deno upgrade lts` to install it", and `:455-459` handles
  `ReleaseChannel::Lts` when comparing versions, so the channel is not just parseable, it
  is what deno itself recommends typing.
- `cli/args/flags.rs:47-53` — the `Channel` block of `UPGRADE_USAGE` listed only
  `stable`, `alpha`, `beta`, `rc`, `canary`. That string is printed by the two
  `bail!` sites at `cli/tools/upgrade.rs:1263` and `:1273`.

The fix adds `lts` after `stable`, in the existing most-stable-to-most-unstable order,
and updates the three spec snapshots that embed the printed block
(`tests/specs/upgrade/invalid_version/{shorthand,canary,version}.out`).

Users already reach for it: #28194 and #29443 are both about `deno upgrade lts`.

## Test plan

Not linked to an open issue — I found the gap by reading `UPGRADE_USAGE` against
`ReleaseChannel::deserialize`. The change is help text only, so no new behaviour needs a
test; the existing `tests/specs/upgrade/invalid_version` specs are the coverage and their
expected output is updated here.

- `rustfmt --edition 2024 --check cli/args/flags.rs` inside the checkout reports no diff
  for the edited file, so the new line matches the configured formatting (the file was
  also clean before the edit; rustfmt has no opinion about the contents of the string).
- I could not run `./x fmt` / `./x lint` / the spec suite locally: they need a full deno
  build (V8), which is not set up on this machine.
- Instead I verified the snapshots mechanically: I stripped the `cstr!` style tags from
  `UPGRADE_USAGE` and compared the result line by line with the `Usage:` block of each of
  the three `.out` files; all three match exactly after the edit. The renderer strips
  those same tags, so CI should agree.

AI assistance disclosure: this change was prepared with Claude Code. I read both sides of
the claim in the source myself, verified the formatting check above, and take ownership of
every line.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `9766cc5e3bc9`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.