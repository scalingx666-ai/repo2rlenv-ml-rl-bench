# Issue

**Title:** fix: `LightningCLI` emitting `jsonargparse` deprecation warnings

## Description

## What does this PR do?

.

`LightningCLI` calls jsonargparse APIs that are deprecated and slated for removal in jsonargparse v5.0, so recent versions emit deprecation warnings.

**Migrated**, gated on `jsonargparse>=4.50`:

| deprecated | replacement |
| --- | --- |
| `ArgumentParser.instantiate_classes` | `ArgumentParser.instantiate` |
| `skip_none` argument of `dump`/`save` | `skip_unset` |

A single 4.50 gate covers both: `instantiate` already exists in 4.49, whereas `skip_unset` does not — and since `dump`/`save` accept `**kwargs`, passing it to 4.49 would be silently ignored and drop `None` entries from the saved config.

**Not migrated:** `ArgumentParser.add_instantiator`. Its replacement, the global `jsonargparse.add_instantiator`, registers process-wide instead of on a single parser, and the registry is keyed by class type — so a second `LightningCLI` using the same model class would overwrite the first one's instantiator. With no non-deprecated way to keep the per-parser scoping, the warning is silenced instead, with a `TODO` for jsonargparse v5.0.

The jsonargparse upper pin is raised from `<4.48.0` to `<4.51.0` so CI actually exercises the new code path.

Verified against jsonargparse 4.47, 4.49 and 4.50.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `a8a60bbde02f`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.