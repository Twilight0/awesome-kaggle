# awesome-kaggle

Operational knowledge for using **Kaggle's free compute tier** as a personal batch-compute
cloud — driven by an agent, not just by a browser.

Kaggle is best known as a data-science competition platform. What this repo documents is a
different framing: it is a **~30 GPU-hours/week free batch-compute service** (2× Tesla T4,
TPU access, ~70 GB scratch disk, datasets as persistent storage) that can run notebooks
headlessly and untended for up to 12 hours at a time — a suitable target for delegating
compute-heavy workflows.

## Documents

| File | What it covers |
|---|---|
| [docs/quota-and-hardware.md](docs/quota-and-hardware.md) | Verified free-tier specs: GPUs, TPUs, quotas, session caps, storage model |
| [docs/delegation-workflow.md](docs/delegation-workflow.md) | The headless notebook loop: push → run → poll → harvest → handoff via datasets |
| [docs/projects-that-fit-the-window.md](docs/projects-that-fit-the-window.md) | A catalog of project types that complete inside the allowed time windows |
| [docs/comfyui-quickstart.md](docs/comfyui-quickstart.md) | 5-minute ComfyUI setup (Kaggle + potato PCs), premade importable workflow |
| [docs/video-generation.md](docs/video-generation.md) | Flagship investigation: video+audio generation on 2 free T4s; the long-form pipeline |
| [docs/agent-toolkit.md](docs/agent-toolkit.md) | Driving Kaggle programmatically (MCP, REST, API tokens) from an agent |

## The numbers, in one table

| Resource | Free-tier value |
|---|---|
| GPU options | 2× Tesla T4 (16 GB each, can be used model-parallel) or 1× P100 |
| TPU | TPU v3-8 |
| Weekly accelerator budget | **30 h GPU · 20 h TPU** (verified via API; resets weekly) |
| Session wall-clock | 12 h (CPU/GPU) · 9 h (TPU) |
| RAM / scratch disk | ~30 GB RAM · ~70 GB ephemeral disk |
| Notebooks output | ~20 GB per version — push large artifacts to datasets instead |
| Internet | Toggleable per notebook (one-time phone verification for GPU/TPU/internet) |

The one thing Kaggle does not sell is a paid tier. Scaling beyond 30 h/week means stacking
other free tiers (Colab, Lightning, Modal) or burst-renting marketplace GPUs (Vast.ai spot
4090 at ~$0.08–0.50/h, RunPod at $0.34/h).

## Operating principles

1. **Batch over interactive.** `SaveAndRunAll` runs have no idle timeout; interactive
   notebooks die after ~20 minutes of inattention. Anything meaningful should be launched
   headlessly.
2. **Datasets are the file system.** Notebook output volumes are ephemeral and capped;
   long-running work checkpoints into datasets and resumes in the next session.
3. **Measure before committing quota.** Every throughput claim in these docs needs a
   timed benchmark run before it becomes an economic assumption.

## Status

- [x] Free-tier facts verified via Kaggle API (October 2026)
- [x] Kaggle video-gen ecosystem surveyed (LTX-2.x distilled, MiniMax H3, WAN 2.2)
- [ ] Phase 0: timed video-generation benchmark notebook (measures clips/hour on T4×2)
