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

**Cell 3 — launch the server in the background** (so later cells keep running):

```python
!nohup python /kaggle/tmp/ComfyUI/main.py --listen --port 8188 > /kaggle/tmp/ComfyUI/server.log 2>&1 &
!sleep 25 && tail -5 /kaggle/tmp/ComfyUI/server.log   # expect: "... To see the GUI: http://127.0.0.1:8188"
```

## Connecting to the UI — the part Kaggle makes non-obvious

Kaggle notebooks have **no built-in port-forwarding** (unlike Colab), so a browser on your
laptop cannot reach `localhost:8188` directly. The standard fix is a tunnel — one extra
cell, no account needed:

**Cell 4 — Cloudflare quick tunnel (recommended):**

```python
!wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 \
  -O /kaggle/tmp/cloudflared && chmod +x /kaggle/tmp/cloudflared
!sleep 15 && /kaggle/tmp/cloudflared tunnel --url http://localhost:8188 --no-autoupdate
# output will contain a line like:  https://xxxx-yyyy-aaaa.trycloudflare.com  ← open this
```

Keep this cell running (it holds the tunnel open; that also keeps the session from idling
out while you work). The first URL appears after ~10–30 s.

What to know before you click that link:

| Fact | Detail |
|---|---|
| **The URL is ephemeral** | a new trycloudflare URL (and new session setup) every session; there is no fixed address on the free tier |
| **Anyone with the URL has your machine** | the ComfyUI UI accepts file uploads by default — treat the link as a password, don't share or paste it to chat |
| **Session dies are expected** | a throwaway server on a rented notebook disappears when the session ends; configs you care about live in datasets (see models-on-kaggle.md), not the UI |
| **Needs the internet toggle** | already enabled in Cell 1 of the guide setup |

Alternative tunnels if Cloudflare is unavailable in your network: `ngrok` (same pattern,
requires a free account + authtoken), or `bore` (single static binary, no account). The
surveyed community notebooks use exactly this pattern (Cloudflare + UI server); nothing
differs under the hood.

**Or: skip connecting entirely** — the Kaggle-native option. Keep the whole loop *inside*
the notebook session and never open a browser:

```python
# after Cell 3, replacement for the UI for batch work:
import json, urllib.request
workflow = json.load(open("/kaggle/input/<your-dataset>/workflow.json"))
# ('Save (API format)' export of your premade graph happens in the UI; commit it to a dataset once)
urllib.request.urlopen(urllib.request.Request(
    "http://127.0.0.1:8188/prompt",
    data=json.dumps({"prompt": workflow, "client_id": "batch"}).encode(),
    headers={"Content-Type": "application/json"}))
```

Then poll `/history`, parse the output filenames, and hand everything to a dataset
per the [delegation workflow](delegation-workflow.md). Same idea as `SaveAndRunAll`
headless pushes: the notebook is the server, the session is the queue.

**On a potato PC (local)**, none of the above applies — the UI lives at plain
`http://127.0.0.1:8188` in your own browser. Tunnel instructions are Kaggle-only.

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
