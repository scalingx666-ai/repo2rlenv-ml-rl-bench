---
pretty_name: Repo2RLEnv ML/RL PR-Diff RL Environments
tags:
- reinforcement-learning
- rl-environment
- harbor
- repo2rlenv
- code-repair
- huggingface
size_categories:
- n<1K
---

# Repo2RLEnv ML/RL PR-Diff Benchmark

54 个可验证的强化学习（RL）环境任务，由 HuggingFace 官方框架 Repo2RLEnv v0.9.6 的 pr_diff pipeline，从 6 个 HuggingFace 机器学习 / 强化学习仓库的真实 Pull Request 生成，输出为 Harbor 任务格式。每个任务要求 agent 阅读真实 issue、定位并修改仓库代码，由容器内私有 verifier 给出 [0, 1] reward。

English: 54 verifiable RL environments generated with HuggingFace Repo2RLEnv (pr_diff pipeline) from real PRs across 6 HF ML/RL repos, in Harbor format. Each task asks an agent to fix a real GitHub issue; a private in-container verifier emits a scalar reward.

生成框架：[huggingface/Repo2RLEnv](https://github.com/huggingface/Repo2RLEnv)

## 来源与任务数 / Sources

| Source repo | Tasks |
| --- | --- |
| huggingface/trl | 12 |
| huggingface/accelerate | 10 |
| huggingface/peft | 9 |
| huggingface/datasets | 14 |
| huggingface/diffusers | 3 |
| huggingface/transformers | 6 |
| **Total** | **54** |

每个任务的 task.toml 里 metadata.repo2env.reference 指向源 PR，ref 指向修复前 commit。完整清单见 manifest.json。

## 单个任务结构 / Task layout

- task.toml — Harbor 元数据 + repo2env 溯源
- instruction.md — 真实 issue 正文 / 复现步骤
- environment/Dockerfile — python:3.12-slim 基础镜像，scrub git 历史防作弊
- solution/patch.diff + solution/solve.sh — oracle 标准答案与应用脚本
- tests/test.sh + tests/verifier.py + tests/oracle.patch — 验证入口、私有判分器、判分基准

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

本数据集 validate --deep --oracle 结果为 all 54 tasks valid；框架自带判分单测 pytest tests/test_pr_diff_verifier.py 为 102 passed。

## 边界与诚实说明 / Caveats

- 生成时 task.toml 的 evaluation.status 为 unverified：只做了生成 + 静态校验 + 判分逻辑单测，未在 Docker/Harbor 中真实 rollout，也未产出任何真实 reward。
- 生成环境无 Docker daemon、无 LLM API key，因此无法实跑 harbor run；任务使用公开基础镜像 python:3.12-slim，不依赖私有镜像。
- 这是 code-repair（改代码修 bug）类 RL 环境，判分为 diff 相似度 + LLM judge；不是训练神经网络达标某指标、训练 RL 策略看回报或联网检索答题类任务，也未基于具体论文挖掘。
- 旧数据集 cell-response-bench（确定性算术 bench）与本数据集无关，保持原样、未改动。

## 溯源 / Provenance

由 Repo2RLEnv 0.9.6 生成；每个任务的源码与 diff 来自对应上游 PR，上游代码版权归各自仓库所有，请遵循各上游仓库许可证。本数据集仅用于研究与评测目的。
