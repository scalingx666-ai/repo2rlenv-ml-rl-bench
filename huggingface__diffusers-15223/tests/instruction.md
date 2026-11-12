# Issue

**Title:** Bump doc-builder to render math in docstring descriptions

## Description

Math in docstring parameters shows as raw `{@html "<span class=…"}` text, e.g. `gate_value` on [`AnyFlowTransformer3DModel`](https://huggingface.co/docs/diffusers/main/en/api/models/anyflow_transformer3d) and `sample` on [`FlowMapEulerDiscreteScheduler`](https://huggingface.co/docs/diffusers/main/en/api/schedulers/flow_map_euler_discrete). huggingface/doc-builder#905 fixes it; this moves the pin to that commit. No workflow inputs changed.

🤖 Generated with [Claude Code](https://claude.com/claude-code)

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `8c54b38fea3d`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.