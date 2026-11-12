# Issue

**Title:** Fix modules_to_save selection when merging an inactive adapter

## Description

. Approved by @BenjaminBossan. merge_and_unload(adapter_names=...) merged the requested backbone adapter but kept the active adapter's modules_to_save copy instead. This PR selects the correct saved copy using the existing checks for each wrapper. If none of the requested adapters has a saved copy for a wrapper, it restores the original module. Default merging and unload() continue to work as before.

The change rejects selections that match more than one saved copy before merging or unloading any layers. Multiple backbone adapters are still supported as long as each wrapper matches no more than one saved copy.

The regression tests reuse the shared merge helper with different saved classifier heads and one existing configuration for each of the 25 methods. Three additional tests cover a missing saved copy, an empty list, and a mixed list. Another test checks that an invalid selection is rejected without changing the model.

Validation on Slurm CPU (Python 3.11.17, PyTorch 2.14.1+cpu, Transformers 5.19.0):

- All 29 regression tests fail without the fix and pass with it.
- 126 passed, 4 skipped (regressions plus existing unload, switching and nonoverlapping-wrapper activation):
  `python -m pytest tests/test_other.py tests/test_custom_models.py -k 'merge_modules_to_save or merge_layers_modules_to_save_selection or set_adapter_non_overlapping_modules_to_save or unload or test_multiple_active_adapters_with_different_modules_to_save_works' -q -o addopts=''`
- 235 passed, 8 skipped (existing merge tests):
  `python -m pytest tests/test_custom_models.py::TestPeftCustomModel::test_merge_layers -k MLP -q -o addopts=''`
- 24 passed, 8 creation-only cases skipped (checkpoint compatibility):
  `REGRESSION_CREATION_MODE=0 REGRESSION_FORCE_MODE=0 python -m pytest tests/regression/test_state_dict.py --regression -k 'lora_modules_to_save or loha or ia3 or hira or (vera and not pvera)' -q -o addopts=''`
- The original issue example now passes: the correct classifier is kept, predictions match, and the maximum logit difference is 2.98e-8.
- Ruff 0.16.4 check and format --check on all changed files, and `git diff --check`: passed.

The full suite and a dependency-version matrix were not run. I used codex assistance for the implementation and validation.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `03412afcf1e9`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.