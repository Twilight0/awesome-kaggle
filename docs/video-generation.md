# Video Generation on 2 Free T4s — investigation notes (Oct 2026)

Findings from surveying the active Kaggle video-generation ecosystem (`search_notebooks`,
`get_notebook_info` on the top kernels). Conclusion up front: **generating video with
synced audio on the free GPU tier works today**, is actively maintained by the community,
and the missing piece for long-form production is a *headless batch producer* — not a
model or a settings sheet.

## The ecosystem (observed October 2026)

| Notebook | Votes | Last run | What it does |
|---|---|---|---|
| `unlockbulk/ltx-2-3-22b-kaggle-t4-by-aiquest` | 2,580 | 2026-09 | LTX-2 22B on T4×2 |
| `unlockbulk/ltx-2-5-22b-distilled-video-generator` | 728 | **2026-10-23** | Gradio studio, full source available |
| `unlockbulk/minimax-h3-turbo-int8-video-audio` | 901 | **2026-10-23** | video + 32 kHz stereo audio |
| `unlockbulk/ltx-2-3-22b-distilled-1-1-q4-t2v-kaggle` | 986 | 2026-09 | simplest: T2V, Q4, minimal machinery |
| `techxion/wan-2-2-image-to-video-comfyui-setup-kaggle` | 431 | 2026-07 | WAN 2.2 I2V via ComfyUI |
| CogVideoX-2B/5B quantized (2024 era) | ~100 each | stale | historical baseline |

All top notebooks: **T4×2, internet on, Python notebook, weights downloaded per session
(~40 GB)**.

## How 22B fits in 2× 16 GB

Engineering tricks (all shipped inside these notebooks — study them before inventing):

- **Distillation**: 8-step (LTX) or 4-step (MiniMax H3 PDMD/DMAD LoRA) few-step samplers —
  each clip is a handful of diffusion steps, not 50.
- **INT8 / Q4 quantization** with Turing-friendly FP16 tensor-core kernels
  (T4 has no BF16; code paths are deliberately FP16).
- **Tensor parallelism across the two T4s**: DiT blocks split per-GPU, text encoder
  (Gemma-class, ~12 B) resident on the *second* GPU, KV-cached so re-prompts are cheap.
- **VRAM choreography**: tiled VAE decode, pinned-memory staging of weights (Kaggle's disk
  reads ~180 MB/s; the notes compute costs per tensor per token), aggressive memory hygiene.
- Session start economics: model load ≈ 3–4 min; first-run downloads ≈ 40 GB.

## Operating envelope (from the notebook configuration code)

- Resolutions: 384p / **480p (recommended)** / 540p / 704p / 720p (slow)
- Clip durations: 2 s (49 frames) · 3 s · 5 s · 10 s @ 24 fps (193-frame tiles)
- Conditioning: text · image · first+last frame · reference (voice cloning on MiniMax H3)
- Output: video **with synchronized audio** (speech + SFX + ambience) in one pass

## What's missing → the long-form gap

These are all **interactive studios** (Gradio UI + Cloudflare tunnel, human clicks
"Generate" per clip). No published notebook does:

1. scripted scene-list → batch clip loop (headless `SaveAndRunAll`),
2. clip manifests + dataset handoff across sessions,
3. ffmpeg stitch + caption + narration → finished long-form cut (CPU session, zero GPU quota).

That three-notebook chain is exactly the delegation-workflow.md pattern and is our
differentiator.

## Throughput: the one unmeasured number

No notebook publishes clips/hour. Everything downstream (weekly output, whether to rent
a 4090 bridge at ~$0.2–0.5/h) hangs on it. Phase 0 = port the T2V distilled variant into a
headless timed benchmark:

```text
Phase 0 — benchmark run (~1–2 GPU h):
  N=10 clips @ 480p × 5 s × 8 steps, weights from attached dataset (no download)
  log per-clip wall time → clips/h, s-of-footage/h, footage/weekly-budget
```

Decision rules after Phase 0:

- if ≥ 8 clips/h → produce a weekly voice-narrated video entirely free on Kaggle;
- if 2–8 clips/h → hybrid (Kaggle + occasional burst rental);
- if < 2 clips/h → switch flagship to rendered/animated (Manim path) — free and deterministic.

## References

- `unlockbulk/ltx-2-5-22b-distilled-video-generator` — best annotated source of how it works
- `unlockbulk/ltx-2-3-22b-distilled-1-1-q4-t2v-kaggle` — most portable source (T2V only)
- `unlockbulk/minimax-h3-turbo-int8-video-audio` — audio-native alternative, VRAM notes goldmine

**Before spending GPU quota on Phase 0**: Giz.ai's web app lists free daily access to the
exact same LTX-2.5 Distilled and H3 Turbo models (2 uses/day, 480p–736p as of 2026-10-20)
— free model auditions without touching the weekly budget. Details and the hosted-vs-Kaggle
tool split: [freeflow-discoveries.md](freeflow-discoveries.md).
