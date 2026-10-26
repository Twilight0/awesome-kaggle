# Image Editing on Kaggle — img2img, inpaint, and the finishing tools

Companion to the [ComfyUI quickstart](comfyui-quickstart.md): same 3-cell setup, one more
premade graph. Everything here runs on the free CPU tier slow, or fast on a T4 — and the
whole editing suite snaps into the batch/delegation pattern without changes.

## The one graph that does both: img2img AND masked inpaint

Import [`docs/assets/comfyui-sd15-imgedit.json`](assets/comfyui-sd15-imgedit.json) the same
way as the txt2img one (drag onto the canvas). It is a typical img2img pipeline with a
**Set Latent Noise Mask** node in the middle — which makes the mask usage decision for you:

| What you do | What it becomes |
|---|---|
| Nothing (no mask painted) | **img2img** — the whole image is re-rendered at `denoise` strength |
| Paint a mask on the image (right-click / **MaskEditor** on the Load Image node) | **inpaint** — only the masked area regenerates |

Two behaviors out of one graph, because an empty mask is a no-op in ComfyUI.

| Node to touch | Tuning note |
|---|---|
| **Load Image** button | upload your edit target; MaskEditor paints the mask (white = edit here) |
| **Denoise** (in Sampler) | the strength dial of img2img: 0.3 = gentle restyle, 0.6 = faithful remix, 0.9+ = near text2img. Classic inpaint range: 0.7–0.95 |
| Empty Latent | **must be switched out** — img2img ignores it; the KSampler's `latent_image` comes from VAEEncode instead. Leave as wired. |

Why it works at potato scale, unchanged from txt2img: same checkpoint file, 512px, and
img2img commonly needs **fewer steps** (16 by default in the graph) than txt2img since it
starts from a real image's latent instead of pure noise.

## Batch editing at scale (headless mode)

Same `/prompt` API trick as the quickstart, with one addition per image set: `LoadImage`
in API format takes **base64-encoded** image data (`image` field), so the batch loop sends
`{filename → base64 pixels}` rather than relying on the disk `input/` folder. For
folder-driven processing, the lighter-weight tools below skip the graph entirely.

## The finishing suite (CPU-tier, zero GPU quota)

These run as plain Python in a CPU session — no reason to ever spend GPU quota on them:

| Job | Tool | One-liner |
|---|---|---|
| Upscale 4× | `realesrgan` (Real-ESRGAN, PyTorch) | runs fine on CPU (slower); the NCNN/Vulkan builds are the fast path when a GPU is around |
| Face restore | `gfpgan` / `codeformer` | after any SD batch that produced faces at 512px |
| Background removal | `rembg` | `session.new_session('birefnet-general')` is the quality tier |
| Old-photo / colorize | `basicsr` recipes | loop over folder, output to dataset |
| Upscale inside ComfyUI | `UpscaleModelLoader` + `ImageUpscaleWithModel` nodes — keep in-graph when you batch anyway | avoids a second pipeline |

According to the quota doc: treat these as "CPU session" stages. A 4×-upscale over
200 images fits in one 12 h CPU session with room to spare; if it doesn't, it takes
*two* CPU sessions — the dataset handoff makes that a non-event.

## Quality ladder, honestly stated

- **512px SD 1.5 base + finishing tools** is the potato baseline — free, fast, mid-range.
- **SDXL (requires a T4)** is where editing quality jumps: cleaner edges at 1024px,
  dramatically better hands/text. The same graph works if you swap the checkpoint node's
  filename; adjust Empty Latent size to 1024×1024 (img2img sizes inherit from the input
  image instead).
- **ControlNet** (Canny/Depth/Pose) is the pro tier of img2img: it *steers* the edit with
  an edge map. One extra loader + one `ControlNetApply` node to add to the graph; checkpoint
  file ~1.4 GB each. Worth it once you have repeatable structures to preserve.

The image-editing stack on Kaggle ends up mirroring our video plan: one GPU session where a
diffusion model runs, CPU sessions where everything else happens.
