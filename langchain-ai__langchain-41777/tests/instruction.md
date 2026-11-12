# Issue

**Title:** docs(langchain): explain Baseten minimum version

## Description

Explain that the Baseten extra requires the 0.2.0 `ChatBaseten` rewrite based on `BaseChatOpenAI`, rather than its legacy tool-calling and streaming implementation.

## References
- [Slack thread](https://langchain.slack.com/archives/C0B4H54MB35/p1794283565322519)

Made by [Robin](https://github.com/langchain-ai/open-swe) · [view thread](https://robin.langchain.dev/agents/d0003d7a-b3df-54aa-aa20-ec132fe765c4) · openai:gpt-6.1-sol (medium)

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `1a41b51b0295`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.