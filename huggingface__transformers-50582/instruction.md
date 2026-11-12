# Issue

**Title:** fix(pipelines): respect problem_type in image and video classification pipelines

## Description

<!-- ci-dashboard-badge:start -->
[![🤗 Transformers CI](https://img.shields.io/badge/%F0%9F%A4%97%20Transformers%20CI-333333?style=flat)](https://transformers-ci.huggingface.cool/d/pytest-observability-by-pr/pytest-observability-branch?var-pr=50582)
<!-- ci-dashboard-badge:end -->

# What does this PR do?

`TextClassificationPipeline` (#34066) and `AudioClassificationPipeline` (#50329) select their default `function_to_apply` according to `model.config.problem_type`, whereas `ImageClassificationPipeline` and `VideoClassificationPipeline` still apply activation functions that corrupt regression or multi-label predictions:

1. [`ImageClassificationPipeline.postprocess`](https://github.com/huggingface/transformers/blob/a8d43b20622149fab30762129c59a2b1ba829364/src/transformers/pipelines/image_classification.py#L195-L203) checks `problem_type == "multi_label_classification" or num_labels == 1` (`SIGMOID`) and `problem_type == "single_label_classification" or num_labels > 1` (`SOFTMAX`) without checking `problem_type == "regression"` first. Regression checkpoints on the Hub (e.g. [`kktlau115/trayzero-vit-regression`](https://huggingface.co/kktlau115/trayzero-vit-regression), [`ljnlonoljpiljm/dinov2-base-sensitivity-regression`](https://huggingface.co/ljnlonoljpiljm/dinov2-base-sensitivity-regression)) have their raw outputs squashed through `sigmoid` (when `num_labels == 1`) or `softmax` (when `num_labels > 1`). Fixed using the same check order as [`TextClassificationPipeline.postprocess`](https://github.com/huggingface/transformers/blob/a8d43b20622149fab30762129c59a2b1ba829364/src/transformers/pipelines/text_classification.py#L183-L193) (#34066).
2. [`VideoClassificationPipeline._sanitize_parameters`](https://github.com/huggingface/transformers/blob/a8d43b20622149fab30762129c59a2b1ba829364/src/transformers/pipelines/video_classification.py#L88-L96) unconditionally sets `postprocess_params["function_to_apply"] = "softmax"` when `function_to_apply` is `None`. This both ignores `model.config.problem_type` (`"regression"` and `"multi_label_classification"`) and overwrites any `function_to_apply` set at pipeline initialization when the pipeline is later called without passing `function_to_apply` again. Fixed using the same approach as [`AudioClassificationPipeline`](https://github.com/huggingface/transformers/blob/a8d43b20622149fab30762129c59a2b1ba829364/src/transformers/pipelines/audio_classification.py#L157-L164) (#50329).

### Reproducer

```python
from PIL import Image
from huggingface_hub import hf_hub_download
from transformers import AutoModelForVideoClassification, VideoMAEImageProcessor, pipeline

img = Image.new("RGB", (64, 64), color=(128, 128, 128))
img_pipe = pipeline("image-classification", model="hf-internal-testing/tiny-random-vit")
img_pipe.model.config.problem_type = "regression"
assert img_pipe(img) == img_pipe(img, function_to_apply="none")

video_model = AutoModelForVideoClassification.from_pretrained(
    "hf-internal-testing/tiny-random-VideoMAEForVideoClassification"
)
video_model.config.problem_type = "regression"
video_proc = VideoMAEImageProcessor(size={"shortest_edge": 10}, crop_size={"height": 10, "width": 10})
vid_pipe = pipeline("video-classification", model=video_model, image_processor=video_proc, frame_sampling_rate=4)
video_path = hf_hub_download(repo_id="nateraw/video-demo", filename="archery.mp4", repo_type="dataset")
assert vid_pipe(video_path, top_k=2) == vid_pipe(video_path, top_k=2, function_to_apply="none")
```

On `main` (`a8d43b2`):
- `img_pipe(img)` returns `[{'label': 'LABEL_0', 'score': 0.5363410115242004}, {'label': 'LABEL_1', 'score': 0.46365901827812195}]` instead of `[{'label': 'LABEL_0', 'score': 0.1062517836689949}, {'label': 'LABEL_1', 'score': -0.039368998259305954}]`.
- `vid_pipe(video_path, top_k=2)` returns `[{'score': 0.519940972328186, 'label': 'LABEL_0'}, {'score': 0.48005902767181396, 'label': 'LABEL_1'}]` instead of `[{'score': 0.08426864445209503, 'label': 'LABEL_0'}, {'score': 0.00446242094039917, 'label': 'LABEL_1'}]`.

### Validation

- Added regression coverage in `tests/pipelines/test_pipelines_image_classification.py` (`test_small_model_pt`) and `tests/pipelines/test_pipelines_video_classification.py` (`test_problem_type_defaults_and_overrides`), both of which fail on `main` and pass with this fix.
- `ruff format --check`, `ruff check`, and `python utils/check_noisy_comments.py --fail-on-findings` pass on all touched files.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `c4f1520d81a4`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.