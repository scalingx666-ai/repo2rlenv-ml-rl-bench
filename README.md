---
pretty_name: Repo2RLEnv ML/RL PR-Diff RL Environments (100 tasks)
tags:
- reinforcement-learning
- rl-environment
- harbor
- repo2rlenv
- code-repair
- huggingface
- high-star-repos
size_categories:
- 100<n<1K
---

# Repo2RLEnv ML/RL PR-Diff Benchmark (100 tasks)

100 个可验证的强化学习（RL）环境任务，由 HuggingFace 官方框架 Repo2RLEnv v0.9.6 的 pr_diff pipeline，从真实 GitHub Pull Request 生成，输出为 Harbor 任务格式。每个任务要求 agent 阅读真实 issue、定位并修改仓库代码，由容器内私有 verifier 给出 [0, 1] reward。

English: 100 verifiable RL environments generated with HuggingFace Repo2RLEnv (pr_diff pipeline) from real PRs across 46 GitHub repositories, in Harbor format. Each task asks an agent to fix a real GitHub issue; a private in-container verifier emits a scalar reward.

生成框架：[huggingface/Repo2RLEnv](https://github.com/huggingface/Repo2RLEnv)

## 组成 / Composition

| 批次 | 任务数 | 源仓库 | 说明 |
| --- | --- | --- | --- |
| 原始 | 54 | 6 个 HuggingFace ML/RL 仓库 | trl / accelerate / peft / datasets / diffusers / transformers |
| 扩展 | 46 | 40 个高 star 高价值仓库 | 按真实 star 优先，每个新仓库最多 2 个 task（≤3 约束内更严格） |
| **Total** | **100** | **46** | |

原始 6 仓库任务数：trl 12、accelerate 10、peft 9、datasets 14、diffusers 3、transformers 6。

## 扩展批次：40 个高 star 仓库 / 46 tasks

star 数由 gh api repos/<owner>/<repo> 的 .stargazers_count 字段实测（2026-11-12），按 star 从高到低入选，每个仓库取 1–2 个 task：

| Source repo | Stars | Tasks |
| --- | --- | --- |
| facebook/react | 252531 | 2 |
| n8n-io/n8n | 210234 | 2 |
| microsoft/vscode | 195312 | 2 |
| ollama/ollama | 186723 | 2 |
| langchain-ai/langchain | 149386 | 2 |
| ggml-org/llama.cpp | 134183 | 2 |
| kubernetes/kubernetes | 129013 | 1 |
| nodejs/node | 123485 | 1 |
| rust-lang/rust | 121799 | 1 |
| openai/whisper | 112528 | 1 |
| denoland/deno | 108867 | 1 |
| pytorch/pytorch | 105446 | 1 |
| fastapi/fastapi | 104643 | 1 |
| vllm-project/vllm | 96781 | 1 |
| django/django | 91719 | 1 |
| opencv/opencv | 91536 | 1 |
| vitejs/vite | 83875 | 1 |
| redis/redis | 77463 | 1 |
| pallets/flask | 75103 | 1 |
| expressjs/express | 69673 | 1 |
| scikit-learn/scikit-learn | 68147 | 1 |
| keras-team/keras | 64422 | 1 |
| ultralytics/ultralytics | 63315 | 1 |
| vuejs/core | 55006 | 1 |
| pandas-dev/pandas | 50493 | 1 |
| apache/airflow | 47455 | 1 |
| apache/spark | 44476 | 1 |
| ray-project/ray | 44269 | 1 |
| microsoft/DeepSpeed | 43326 | 1 |
| jax-ml/jax | 36616 | 1 |
| explosion/spaCy | 34044 | 1 |
| numpy/numpy | 33342 | 1 |
| Lightning-AI/pytorch-lightning | 31449 | 1 |
| dmlc/xgboost | 28967 | 1 |
| mlflow/mlflow | 28772 | 1 |
| microsoft/onnxruntime | 22291 | 1 |
| microsoft/LightGBM | 18968 | 1 |
| optuna/optuna | 15039 | 1 |
| huggingface/tokenizers | 11250 | 1 |
| catboost/catboost | 9200 | 1 |
| **Subtotal** | | **46** |

入选仓库覆盖 DL 框架（pytorch / keras / jax / DeepSpeed / vllm / llama.cpp / ollama / whisper）、ML 库（scikit-learn / xgboost / LightGBM / numpy / pandas / spaCy / catboost / optuna / onnxruntime）、MLOps 与数据（mlflow / airflow / spark / ray / Lightning）以及顶级通用开源项目（react / vscode / n8n / kubernetes / rust / nodejs / deno / vite / redis / django / fastapi / flask / express / vue / opencv）。tensorflow/tensorflow 与 golang/go 因 pr_diff 列 PR 返回 candidates=0 被跳过。

完整的 100 任务溯源（源仓库、PR 链接、commit、pipeline、reward 类型）见 manifest.json。

## 单个任务结构 / Task layout

- task.toml — Harbor 元数据 + repo2env 溯源
- instruction.md — 真实 issue 正文 / 复现步骤
- environment/Dockerfile — python:3.12-slim 基础镜像，scrub git 历史防作弊
- solution/patch.diff + solution/solve.sh — oracle 标准答案与应用脚本
- tests/test.sh + tests/verifier.py + tests/oracle.patch + tests/capture.py + tests/instruction.md — 验证入口、私有判分器、判分基准

## 判分 / Reward

tests/verifier.py 为 6 分量加权求和，结果写入 /logs/verifier/reward.txt（Harbor 读取）：

| 分量 | 权重 | 含义 |
| --- | --- | --- |
| format_valid | 0.00 | 预测是否为合法 unified diff |
| size_sanity | 0.08 | 改动规模与 oracle 的比值 |
| file_targeting | 0.12 | 改动文件集合的 F1 |
| region_overlap | 0.20 | hunk 区间重叠度 |
| similarity | 0.10 | +/- 行 SequenceMatcher 相似度 |
| llm_judge | 0.50 | LLM 判定是否解决该 issue |

llm_judge 默认调用 Anthropic Haiku，也可用 R2E_JUDGE_ENDPOINT 指向任意 OpenAI-compatible server。缺 LLM key 时该分量为 null，任务记为 ungraded、不写 reward 文件。权重可用 R2E_W_* 环境变量覆盖。

## 使用 / Usage

```bash
repo2rlenv --no-ui validate <dataset> --deep --oracle
harbor run --dataset <dataset> --env docker --ve ANTHROPIC_API_KEY=... R2E_JUDGE_MODEL=...
```

本数据集 validate --deep --oracle 结果为 all 100 tasks valid；框架自带判分单测 pytest tests/test_pr_diff_verifier.py 为 102 passed。

## 边界与诚实说明 / Caveats

- 生成时 task.toml 的 evaluation.status 为 unverified：只做了生成 + 静态校验 + 判分逻辑单测，未在 Docker/Harbor 中真实 rollout，也未产出任何真实 reward。
- 生成环境无 Docker daemon、无 LLM API key，因此无法实跑 harbor run；任务使用公开基础镜像 python:3.12-slim，不依赖私有镜像。
- 这是 code-repair（改代码修 bug）类 RL 环境，判分为 diff 相似度 + LLM judge；不是训练神经网络达标某指标、训练 RL 策略看回报或联网检索答题类任务，也未基于具体论文挖掘。
- 高 star 高价值依据为实测 GitHub star 数与仓库知名度；pr_diff 只从已合并 PR 中筛选可生成任务，不代表对仓库全部代码质量的评测。
- 旧数据集 cell-response-bench（确定性算术 bench）与本数据集无关，保持原样、未改动。

## 溯源 / Provenance

由 Repo2RLEnv 0.9.6 生成；每个任务的源码与 diff 来自对应上游 PR，上游代码版权归各自仓库所有，请遵循各上游仓库许可证。本数据集仅用于研究与评测目的。

