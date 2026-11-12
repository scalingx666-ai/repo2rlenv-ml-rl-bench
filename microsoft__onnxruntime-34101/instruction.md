# Issue

**Title:** Read device memory type when adapting API tensors in EP adapter

## Description

### Description

`ep::adapter::CreateTensorFromApiValue` built the tensor's `OrtDevice` from `OrtMemoryInfo::GetMemoryType()`. That returns an `OrtMemType`, which can be `OrtMemTypeCPUInput` (-2) or `OrtMemTypeCPUOutput`/`OrtMemTypeCPU` (-1). `OrtDevice` accepts only device memory types (`DEFAULT` or `HOST_ACCESSIBLE`), so wrapping a host tensor tagged with one of those values threw `Invalid memory type: -1`.

This change uses `GetDeviceMemoryType()`, the same call `CreateMemoryInfoFromApi` in `allocator.h` already makes. The original `OrtMemType` still populates the `OrtMemoryInfo::mem_type` field.

### Motivation and Context

In plugin EP kernels the exception escapes through `noexcept` paths and the process aborts. microsoft/onnxruntime-genai Qwen-VL / Qwen3.5 multi-image prompts on the CUDA plugin EP reach this path because GenAI wraps per-image slices with `OrtMemTypeCPU`. GenAI is also fixing its side in microsoft/onnxruntime-genai#2941, but any API tensor tagged with `OrtMemTypeCPU*` can trigger this.

### Testing

New test: `OrtEpLibrary.TensorAdapterUsesDeviceMemoryTypeFromApi` wraps host buffers created with `OrtMemTypeCPUInput`, `OrtMemTypeCPUOutput` and `OrtMemTypeDefault`. It checks the device type, the device memory type, `mem_type` and the data pointer.

Linux x64 Release build, `onnxruntime_autoep_test`:

| Build | Result |
|---|---|
| Without fix | New test aborts: `Invalid memory type: -2` |
| With fix | All 142 tests pass |

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `6cc3c09506ec`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.