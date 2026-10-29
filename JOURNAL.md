# Journal

## 2026-10-29: survey and plan
- Host: Ultra 9 285K, 125 GB RAM, RTX 5090 32 GB (shared, about 6.4 GB in use by another user), Ubuntu 24.04. Installed: uv, gh. Missing: docker, node.
- Local candidate: Qwen3.8-27B (Apache-2.0). GGUF Q4_K_XL is 17.6 GB, Q5_K_XL is 20.9 GB.
- Cluster candidates: GLM-5.3-Flash 321B (MIT), Qwen3.8-Flash-Next-FP8 186 GB (Qwen license), DeepSeek-V4.1-Flash 510 GB (MIT).
- Versions: OpenCode v1.19.5, vLLM 0.32.0, llama.cpp b11982 (ships `ubuntu-cuda-12.8-x64` binaries), Pi v1.4.0.
- UCloud: 9.1k GPU-hours left. SSH apps: terminal-almalinux, ubuntu-xfce, coder. No vLLM app exists, so vLLM gets installed with uv inside the job.
- Wrote [docs/PLAN.md](docs/PLAN.md).
- Next: Phase 1.
