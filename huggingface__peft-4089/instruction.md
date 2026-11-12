# Issue

**Title:** FEAT healthcheck: warn when a randomly initialized task head is not saved

## Description

As discussed in the issue, `healthcheck` now reports a new `TASK_HEAD_NOT_SAVED` finding (severity `warning`) when a randomly initialized task head is not saved with the adapter. This usually happens when `task_type` is not set, so the head is not added to `modules_to_save`.

### What changed

- `src/peft/utils/healthcheck.py`: new helper `_find_unsaved_new_task_heads` and the new finding. The check is:
  - The base model class ends with `ForSequenceClassification`, `ForTokenClassification` or `ForQuestionAnswering`.
  - `config.architectures` is set and does not contain the class name, so the head was not in the checkpoint.
  - A head (`CLASSIFIER_MODULE_NAMES` or `qa_outputs`) is not wrapped by `ModulesToSaveWrapper`. A head that is only targeted by LoRA is still reported, because only the LoRA weights would be saved, not the random base weight.
- `docs/source/developer_guides/troubleshooting.md`: short note in "Randomly initialized layers".
- `tests/test_tuners_utils.py`: new tests in `TestHealthcheck`.

`is_ready` and the format of the report are not changed. If `architectures` is missing (e.g. a model created from a config, or `microsoft/deberta-v3-base`) or the model is not from Transformers, the check is skipped. `get_peft_model` is not touched.

### Tests

New tests use a tiny GPT2 saved to `tmp_path`, so they run offline:

- flagged: no `task_type`, and head in `target_modules`
- not flagged: `task_type="SEQ_CLS"`, head in `modules_to_save`, an already fine-tuned classifier checkpoint, and a model without `architectures`

I ran `tests/test_tuners_utils.py` locally (273 passed, 16 skipped).

### Known limits

Same as in the issue: cases with `ignore_mismatched_sizes=True` are not detected, and a user who saves the head separately on purpose will still see the warning.

This PR was authored by a human.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `65e7758329ac`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.