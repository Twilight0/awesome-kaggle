# Models on Kaggle — where they live and how to haul them

Half the art of Kaggle compute is **not paying for model downloads every session**.
A 12 h T4 session that spends 45 minutes re-downloading 40 GB has burned 6% of its life —
and 1.5 hours of your weekly 30. This page is how to avoid that.

## The two registries (and when to use which)

| Source | What it's good for | How it arrives |
|---|---|---|
| **Kaggle Models** (`kagglehub`, or model attachments) | first-party family packs: Gemma, DeepSeek, Llama — huge, versioned, and **fast** on Kaggle's internal pipe | Attach as notebook input → mounts read-only at `/kaggle/input/<model-family>`; zero download cost, zero internet needed |
| **Hugging Face** | everything else: community checkpoints, GGUF quant exports, abliterated variants, SDXL/Flux/WAN — the long tail we actually care about | `huggingface_hub`/`hf` CLI downloads via the notebook's internet toggle |

Rule of thumb: **registry for the big lab families, HF for everything specific.**

## The persistence ladder (cheap → expensive)

1. **Attachment (Kaggle Models, competition data, or your dataset)**: mounted, not
   downloaded. session runtime cost ≈ zero. The goal state for *every* weight that matters.
2. **Persist to your own dataset**: HF/download once in a session → publish `/kaggle/tmp/models/<name>` as a private
   dataset (via `kagglehub` or the API tools) → next session attaches it. One-time cost,
   then reads are internal.
3. **Re-download per session** (happens when neither of the above is possible — e.g. weights
   that need a fresh internet fetch because the dataset upload of 40 GB was clumsy): billed to
   session wall-time. Use `hf download` with `HF_HUB_ENABLE_HF_TRANSFER=1` (`pip install
   hf_transfer` first) — it saturates Kaggle's pipe; SD1.5 in ~20 s vs several minutes.
4. **Worst case — weights in notebook *code* files or node nodes that hand-copy weights into
   /kaggle/working**: this is the "weights in output bundle" trap from the ComfyUI guide. Never.

## Building a private models-dataset (pay once, attach forever)

```python
# once, in a throwaway session:
!pip -q install kagglehub huggingface_hub hf_transfer
import kagglehub, huggingface_hub as hf
import os
os.environ["HF_HUB_ENABLE_HF_TRANSFER"] = "1"   # must be set before huggingface_hub import/use
hf.snapshot_download(
    repo_id="huihui-ai/Llama-3.2-1B-Instruct-abliterated",   # example; see abliterated-models.md
    local_dir="/kaggle/tmp/models/llama-1b-abliterated",
    ignore_patterns=["*.pth", "*.bin"],   # GGUF/safetensors only
)
kagglehub.dataset_upload(          # then attach in every future notebook as input
    handle="<you>/llama-1b-abliterated",
    local_dir="/kaggle/tmp/models/llama-1b-abliterated",
)
```

Then, in every future session: attach `<you>/llama-1b-abliterated` as the notebook's
dataset source, and it mounts straight into `/kaggle/input/`. The same works for comfort
stacks — keep your whole SD checkpoint collection as one dataset; keep LLMs as per-model
datasets (easier versioning).

## Quantization for the T4 reality (16 GB × 2, or CPU sessions)

| Class | What fits | GGUF size (7–8 B class) | Runs on |
|---|---|---|---|
| FP16 | small models, single GPU | — | T4 (one 8 B, barely) or CPU |
| **Q8/Q6_K GGUF** | same quality as FP16 with ~30% size cut | ~7–9 GB | **CPU sessions — free, zero quota** (llama.cpp uses all Kaggle vCPUs) |
| Q4_K_M GGUF | nearly indistinguishable on chat tasks; unmeasurably worth the size | ~4.5–5.5 GB | CPU + T4 (both T4s if sharded — GGUF `split_mode=row`) |
| Full precision 13 B+ | — | too big for FP16 on T4 | must be quantized anyway |

**The under-used trick**: CPU-only sessions carry **zero GPU-quota cost** — the 30 h weekly
meter doesn't tick for them, session cap is 12 h, and `llama.cpp` will happily take longer
but free. For prompt-eval, batch text jobs, and model smoke-testing this is the default.
Reserve GPU sessions for genuinely parallel work.

## Finding a model listing's exact files

- Kaggle registry/`kagglehub` → `get_model`/`list_model_variation_versions` (MCP tools) for
  variation/versions/files.
- HF → model pages list **different variant-heavy repos**: original + GGUF under a sibling
  repo by the same author — GGUF variants often live in the same repo, or in a sibling
  repo; check both before hunting elsewhere.
- Abliterated flavor mapping is in [abliterated-models.md](abliterated-models.md).
