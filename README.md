# awesome-kaggle

Operational knowledge for using **Kaggle's free compute tier** as a personal batch-compute
cloud — driven by an agent, not just by a browser.

Kaggle is best known as a data-science competition platform. What this repo documents is a
different framing: it is a **~30 GPU-hours/week free batch-compute service** (2× Tesla T4,
TPU access, ~70 GB scratch disk, datasets as persistent storage) that runs notebooks
headlessly and untended for up to 12 h at a time — a suitable target for delegating
compute-heavy workflows: training, batch inference, image/video production.

## The numbers, in one table

| Resource | Free-tier value |
|---|---|
| GPU options | 2× Tesla T4 (16 GB each) or 1× P100 |
| TPU | v3-8 |
| Weekly accelerator budget | **30 h GPU · 20 h TPU** (verified via API; resets weekly) |
| Session wall-clock | 12 h (CPU/GPU) · 9 h (TPU) — CPU sessions are **free** of the GPU meter |
| RAM / scratch disk | ~30 GB RAM · ~70 GB ephemeral disk |
| Notebook output | ~20 GB per version — push large artifacts to datasets instead |
| Internet | toggleable per notebook (one-time phone verification) |
| Kaggle pricing | there is none — free only; scaling up = other free tiers or burst-rental GPUs |

## Docs

### Foundations — read these two first

| File | What it covers |
|---|---|
| [docs/quota-and-hardware.md](docs/quota-and-hardware.md) | Free-tier specs, storage model, quota economics |
| [docs/delegation-workflow.md](docs/delegation-workflow.md) | The headless notebook loop: push → run → poll → harvest → dataset handoff |

### Playbooks — what to run

| File | What it covers |
|---|---|
| [docs/projects-that-fit-the-window.md](docs/projects-that-fit-the-window.md) | 20+ workload types, each budgeted against the time windows |
| [docs/video-generation.md](docs/video-generation.md) | Video + synced audio on 2 free T4s (LTX-2.x, MiniMax H3); long-form pipeline; Phase-0 plan |
| [docs/image-editing.md](docs/image-editing.md) | img2img/masked inpaint on one graph; upscale/restore/rembg finishing suite |

### Tooling — the hands

| File | What it covers |
|---|---|
| [docs/agent-toolkit.md](docs/agent-toolkit.md) | Driving Kaggle programmatically: MCP + REST, token hygiene, FreeFlow scouting |
| [docs/freeflow-discoveries.md](docs/freeflow-discoveries.md) | Hosted free-tool scouting per pipeline stage: what to sample hosted vs produce on Kaggle |
| [docs/comfyui-quickstart.md](docs/comfyui-quickstart.md) | 5-minute ComfyUI (Kaggle + potato PCs) + [`premade txt2img workflow`](docs/assets/comfyui-sd15-txt2img.json) |
| [docs/comfyui-cli-and-mcp.md](docs/comfyui-cli-and-mcp.md) | The agent path: official `comfy-cli` headless workflows + MCP bridges that let an agent drive ComfyUI as tools |
| — | + [`premade img2img/inpaint workflow`](docs/assets/comfyui-sd15-imgedit.json) (one graph, both modes) |

### Models — the weights

| File | What it covers |
|---|---|
| [docs/models-on-kaggle.md](docs/models-on-kaggle.md) | Kaggle Models vs HF, quantization ladder, pay-once-attach-forever dataset pattern |
| [docs/abliterated-models.md](docs/abliterated-models.md) | Abliterated/uncensored models: technique, publishers, GGUF on free CPU quota; licenses & responsibility |

## Operating principles

1. **Batch over interactive.** `SaveAndRunAll` runs untended up to 12 h; interactive
   notebooks die after ~20 idle minutes and burn GPU quota while idle.
2. **Datasets are the file system.** Outputs are ephemeral and capped; checkpoints and
   deliverables live in datasets, which also carry state between sessions.
3. **CPU is free — spend GPU only where tensors need it.** Stitching, TTS, dataset ops,
   upscales, GGUF inference: CPU sessions. Diffusion and training: GPU.
4. **Measure before committing quota.** Throughput claims are hypotheses until a timed
   benchmark confirms them.

## Status

- [x] Free-tier facts verified via Kaggle API (October 2026)
- [x] Kaggle video-gen ecosystem surveyed (LTX-2.x distilled, MiniMax H3, WAN 2.2)
- [x] ComfyUI quickstart + two premade importable workflows (txt2img, img2img/inpaint)
- [x] Model-management guide (persistence ladder, quantization ladder)
- [x] Abliterated/uncensored models guide (technique, publishers, free-CPU recipe)
- [x] FreeFlow scouting pass documented (hosted free tools vs Kaggle roles; MusicGen gem)
- [x] comfy-cli + MCP agent path documented (native tool-call generation, Kaggle + potato variants)
- [ ] Phase 0: timed video-generation benchmark (measures clips/hour on T4×2)
