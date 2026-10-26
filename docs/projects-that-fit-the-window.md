# Projects That Fit The Window — what actually fits in 12 h / 30 h

A catalog of workloads that complete inside the allowed time windows, with realistic
budget estimates. Two windows define feasibility:

- **Session** = one unattended headless push (up to 12 h CPU/GPU, 9 h TPU)
- **Week** = the 30 h GPU + 20 h TPU meter

"✅ 1 session" = comfortably fits with margin. "⚠️ tight" = fits at reduced
size/quality or with resume. "⛔" = wrong shape for Kaggle.

## Model training

| Project | Window | Notes |
|---|---|---|
| LoRA fine-tune a 4–8 B LLM ( Unsloth, FP16, small corpus) | ✅ 1 session (2–6 GPU h) | Unsloth tunes 7 B‑class models on a single 16 GB card; 2× T4 doubles it |
| Train a small transformer from scratch (nanoGPT-class, ~10–100 M params) on a custom corpus | ✅ 1 session | textbook fit; checkpoint via output→dataset |
| Fine-tune a small speech model (Whisper small/medium, or XTTS adapt) | ✅ 1 session | datasets exist with voice corpora |
| Train YOLO / segmentation / classifier CNNs (YOLOv8 n–m, ViT-B) | ✅ 1 session | classic Kaggle workload |
| Diffusion LoRA (SD 1.5/2, SDXL) on a curated image set | ✅ 1 session | SDXL LoRA tight on 16 GB — use gradient checkpointing |
| Fine-tune a 13 B+ LLM | ⚠️ tight | needs QLoRA + sharding; possible, painful |
| Post-train any frontier model | ⛔ | not free-tier scale |

## Batch inference ( GPU quota's best use: embarrassingly parallel work)

| Project | Window | Notes |
|---|---|---|
| Whisper ASR over hours of audio (large-v3 on T4) | ✅ 1 session processes several audio-hours | produce transcripts + subtitles as dataset artifacts |
| Embeddings over a large corpus (sentence-transformers on T4) | ✅ 1 session ≈ millions of chunks | store as a parquet dataset; zero-setup retrieval |
| Local LLM batch inference, 8–27 B quantized (llama.cpp GGUF multi-GPU, or transformers Q4/Q8) | ✅ 1–2 h per ~10⁴–10⁵ prompts | this is a private "batch LLM as a service" pipeline |
| Stable Diffusion / SDXL / Flux-quantized batch image production | ✅ 100–1000 images per session | deliverable: image dataset |
| **Video + audio generation (LTX-2.x distilled, MiniMax H3)** | ⚠️ see video-generation.md | scheduled: ≈10–20 min of 480p footage per 12 h session (unmeasured — Phase 0 pending) |
| Video upscaling / interpolation (RIFE, Real-ESRGAN) over existing footage | ✅ 1 session | CPU stages can pair with GPU stage notebooks |
| Whisper-generated subtitles → styled caption renders (ffmpeg, CPU) | ✅ 0 GPU quota | run on CPU sessions to save budget |

## Data ops ( CPU-heavy, no accelerator needed)

| Project | Window | Notes |
|---|---|---|
| Scrape → clean → enrich → publish as a Kaggle dataset (a canonical use of the platform) | ✅ CPU session | internet toggle on; be polite, respect ToS/robots |
| OCR/document extraction over scanned corpora (Tesseract, PaddleOCR, Surya) | ✅ CPU or GPU session | schedule: GPU only if the corpus is image-heavy |
| Audio processing at scale (transcode, denoise with Demucs, loudness-normalize) | ✅ CPU session | output straight into a dataset for other stages |
| Big-data wrangling (pandas/Polars over competition or scraped datasets, feature builds) | ✅ CPU session | datasets as scratch |
| Large-scale deduplication / embedding clustering / FAISS index builds | ✅ CPU + GPU pairing | index artifacts are small dataset uploads |

## Rendering & publishing ( CPU = free)

| Project | Window | Notes |
|---|---|---|
| Manim / motion-graphics explainer renders (scripted scenes) | ✅ CPU session | deterministic, quota-free, needs zero simplification |
| ffmpeg pipelines: concat, mux, loudness, thumbnails, color, chapters | ✅ CPU session | consumed after any GPU generation stage |
| SVG → 4K posters / slide decks / cover art | ✅ CPU session | headless browsers work fine when needed |

## Competitions ( the original purpose — still worth listing)

| Project | Window | Notes |
|---|---|---|
| Train a tabular/lightweight competition entry (XGBoost/LightGBM/CatBoost or small NN) | ✅ CPU/GPU session | every "Getting Started" competition runs in hours |
| Enter a medal-track competition with an existing notebook baseline | ⚠️ | quota math matters; GPU comps eat 30 h/week fast |

## The meta pattern

Every long-form project = a **pipeline of stages**, each stage one headless session,
each handoff one dataset. Anything expressible as "process artifacts → produce artifacts
→ next stage" fits Kaggle — regardless of GPU needs — as long as no single stage needs
more than ~12 uninterrupted hours.

The scarce currency is GPU time, so the operating rule: **spend GPU only where
progress depends on parallel tensor math; spend CPU for everything else.**
