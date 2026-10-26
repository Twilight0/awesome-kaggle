# ComfyUI — the 5-minute guide

ComfyUI is a node-graph runner for diffusion models (Stable Diffusion, SDXL, Flux, WAN…).
It is the one tool that keeps showing up in every Kaggle image/video workflow we surveyed,
because it needs no web service, no API key and no special hardware. This page stays
minimal on purpose: zero config files, three commands, works on cheap laptops.

## On Kaggle (recommended) — 3 cells, ~5 minutes

Create a notebook, enable **internet**, GPU optional (T4 if you have it, CPU works).

**Cell 1 — install (~90 s):**

```bash
!git clone --depth 1 https://github.com/comfyanonymous/ComfyUI /kaggle/tmp/ComfyUI
%cd /kaggle/tmp/ComfyUI
!pip -q install -r requirements.txt
```

> Note the path: `/kaggle/tmp`, **not** `/kaggle/working`. Model weights are huge;
> anything in `working` gets uploaded into the notebook's output bundle at session end
> (capped ~20 GB). The tmp disk (~70 GB) is ephemeral but never uploaded.

**Cell 2 — one checkpoint (~1–2 min on Kaggle's pipe):**

```bash
!mkdir -p /kaggle/tmp/ComfyUI/models/checkpoints
# SD 1.5 — the classic potato checkpoint (~4 GB, 512px):
!wget -q --show-progress -O /kaggle/tmp/ComfyUI/models/checkpoints/sd15.safetensors \
  https://huggingface.co/benjamin-paine/stable-diffusion-v1-5/resolve/main/model.safetensors
```

(SDXL, Flux or WAN files plug into the same folder — NVIDIA GPU just makes them fast.)

**Cell 3 — launch + port-forward:**

```bash
%cd /kaggle/tmp/ComfyUI
!python main.py --dont-print-server --listen
# the notebook will live-host the UI on :8188; open it via the notebook's proxy/tunnel
```

For **headless batch** (the delegation-workflow pattern), skip the UI entirely — ComfyUI
is also an HTTP API: save a workflow as JSON in the editor, then `POST {"prompt":
<workflow-json>}` to `http://127.0.0.1:8188/prompt` in a loop. One notebook, N images,
artifacts to a dataset. No clicking.

## Premade workflow — import and go

Don't wire the graph from scratch. Grab [`docs/assets/comfyui-sd15-txt2img.json`](assets/comfyui-sd15-txt2img.json)
(a complete SD 1.5 txt2img graph: checkpoint → prompt → sampler → save, 512×512, 20 steps,
seed fixed at 42 for reproducibility) and drag the file **onto the ComfyUI canvas**
(or: menu ≡ → *Load*). Every knob worth touching sits in its own titled node:

| Node | Turn it into… |
|---|---|
| **Prompt (positive/negative)** | your actual prompt |
| **Sampler** nodes' widgets | steps (20 → 8–12 for speed), size in *Empty Latent*, `randomize` seed for variety |
| **Save** | output filename prefix |

The checkpoint node expects the filename from **Cell 2** above (`sd15.safetensors`) —
that's the only coupling between setup and workflow. After tweaking one graph you're happy
with, hit **Save** — and if you're about to batch through the API, use **Save (API Format)**;
that JSON drops straight into the `POST /prompt` loop from the headless section.

## On a potato PC (local, no GPU required)

```bash
git clone --depth 1 https://github.com/comfyanonymous/ComfyUI && cd ComfyUI
pip install -r requirements.txt
python main.py --cpu        # CPU mode; add --lowvram on small GPUs
```

Same three flags, that's all the "config". Expectations, honestly stated:

- **CPU mode is slow but real** — SD 1.5 at 512px ≈ a minute-plus per 20-step image on a
  mid laptop. Good for iteration, previews and occasional use.
- The free Kaggle T4 is ~20–50× faster than most CPUs for this — a better partner than a
  local install for serious batches.
- Keep checkpoints small on disk: SD 1.5 (4 GB) → SDXL (~7 GB) → Flux GGUF-quantized
  (~6–12 GB) is the whole ladder; nothing here requires 24 GB VRAM.
- Workflows are single `.json` files — save one from someone else, drop it onto the
  canvas, done. ComfyUI can also embed a workflow inside generated `.png` outputs.

## Gotchas (the short list)

- Weights path vs output path (above) — the #1 way people blow up their output bundle.
- Port 8188: bind `--listen` for any remote/proxy access; localhost-only otherwise.
- Requirements install into the session each time — that's fine; **weights do not have to
  re-download each session** if you park them in a private dataset and symlink into
  `models/` (optional optimization, ~10 min of setup, saves ~2 min/session — skip it until
  sessions feel slow).
- First-run torch download is ~3 GB — only happens once per environment refresh.
