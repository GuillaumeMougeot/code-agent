# Plan

Goal: a Claude Code–like coding agent built only from open-source software and
open-weight models. Install should be one command plus a model download
(and, optionally, a UCloud login).

## Architecture

```
 terminal / phone (SSH or web UI)
        │
     OpenCode  ── agent TUI, tools, MCP, LSP           (MIT)
        │ OpenAI-compatible HTTP
   ┌────┴─────────────────────┐
 local :8080                cluster :8001 (SSH tunnel)
 llama.cpp llama-server     vLLM on UCloud B200 job
 Qwen3.8-27B GGUF Q4        GLM-5.3-Flash / Qwen3.8-Flash-Next
 RTX 5090 (shared)          started/stopped by `oca` via ucloud-api
```

`oca` is the glue CLI in this repo (Python, uv, typer). It installs, configures,
and runs everything else.

## Choices

| Layer | Pick | License | Why | Not chosen |
|---|---|---|---|---|
| Agent | OpenCode | MIT | Mature, single binary, multi-provider, plan/build agents, `opencode web` for phone | Pi (minimal, good later for custom harness), Hermes/OpenClaw (personal assistants, not coding-first) |
| Local engine | llama.cpp | MIT | Prebuilt CUDA binaries, GGUF, small memory footprint on a shared GPU | Ollama (wraps llama.cpp, adds a daemon, new models arrive later), vLLM local (heavier, grabs a fixed VRAM fraction) |
| Cluster engine | vLLM 0.32 | Apache-2.0 | Fastest multi-GPU serving, FP8/FP4 on B200, tool-call parsers | SGLang (fine alternative), Triton app (needs model conversion) |
| Local model | Qwen3.8-27B (unsloth UD-Q4_K_XL, 17.6 GB) | Apache-2.0 | Best agentic coder that fits in about 24 GB free | Devstral Small 2, gpt-oss-20b |
| Cluster model | GLM-5.3-Flash, 321B, 4×B200 | MIT | Strong coder, permissive license | Qwen3.8-Flash-Next (186 GB FP8, 2×B200, Qwen license), DeepSeek-V4.1-Flash (MIT, 510 GB, 4×B200), Kimi-K3 (2.8T, too big) |
| Router | none | – | OpenCode handles multiple providers itself, and the tunnel gives the cluster a fixed port | LiteLLM (add later only if fallback or usage tracking is needed) |
| Automation | none | – | Not needed for coding | n8n (fair-code, not OSI-licensed) |
| UCloud | ucloud-api | GPL-3.0 | Already built: jobs, SSH, queue, auto-extend | – |

Facts behind these choices (2026-10-29):
- RTX 5090 32 GB, but another user is using about 6.4 GB and 99% utilization.
- UCloud quota: 9.1k B200 GPU-hours and 1.3 TB of storage.
- SSH-capable apps: `terminal-almalinux`, `ubuntu-xfce`, `coder`.
- `pytorch-te` has no SSH, so it is unusable for a tunnel.

## Data layout

`OCA_DATA` defaults to `/data/$USER/oca` if that exists, otherwise `~/.local/share/oca`.

```
$OCA_DATA/models/   GGUF + HF cache (HF_HOME)
$OCA_DATA/bin/      llama.cpp release
$OCA_DATA/logs/
```

Cluster weights live once on a UCloud drive (`/<drive>/oca/models`) and are mounted read-only.

## Target UX

```bash
curl -LsSf https://raw.githubusercontent.com/GuillaumeMougeot/code-agent/main/install.sh | sh
oca setup            # opencode + llama.cpp binaries, opencode.json
oca pull             # local model -> $OCA_DATA/models
oca                  # start local server if needed, launch opencode
# optional
ucloud login         # one-time token
oca cluster up       # B200 job + vLLM + tunnel; auto-extend
oca cluster down
oca doctor
```

## Phases

1. **Local MVP.** Scaffold the package with uv, ruff, mypy, pytest, pre-commit and CI. Build `oca setup/pull/serve` plus OpenCode config. Done when OpenCode edits a repo with the local model.
2. **Cluster.** Validate vLLM in `terminal-almalinux` on a MIG slice, then on 4×B200. Download weights to a UCloud drive. Build `oca cluster up/down/status` with a tunnel and auto-extend through ucloud-api.
3. **Eval and defaults.** Run a small fixed task set (local vs. cluster models), recording pass rate, tok/s and cost in GPU-hours. Pick defaults.
4. **Phone.** `opencode web` bound to localhost, reached via SSH tunnel or WireGuard (Headscale optional), or tmux + SSH from a phone client.
5. **Polish.** `install.sh`, `oca doctor`, mkdocs site, a release tag. Optionally a systemd user unit for the local server.

Containers: not needed. Local binaries are self-contained, and UCloud apps are already containers.
