# code-agent

A coding agent stack built only from open-source software and open-weight models:
[OpenCode](https://opencode.ai) for the agent, [llama.cpp](https://github.com/ggml-org/llama.cpp) for local inference,
and [vLLM](https://github.com/vllm-project/vllm) on [UCloud](https://cloud.sdu.dk) B200s through
[ucloud-api](https://github.com/GuillaumeMougeot/ucloud-api).

Status: planning. See [docs/PLAN.md](docs/PLAN.md) and [JOURNAL.md](JOURNAL.md).

```bash
# target
curl -LsSf https://raw.githubusercontent.com/GuillaumeMougeot/code-agent/main/install.sh | sh
oca setup && oca pull && oca
```
