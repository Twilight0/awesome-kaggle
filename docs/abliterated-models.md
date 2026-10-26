# Uncensored & Abliterated Models — what exists and how to run them

"abliterated" models are refusal-ablated weight edits, publicly published on Hugging
Face and GitHub with a peer-reviewed technique behind them. They remove a model's
trained-in refusal behavior via tensor surgery — the same weights, but willing to answer
sensitive-but-legal prompts (security research, fiction and adult creative writing,
medical triage language, dataset detox for controlled pipelines) without a lab-policed
disclaimer layer.

This page documents the technique and the mechanics, plus the license and responsibility
boundaries that apply regardless of which model you pick.

## The technique in two minutes

- **Method**: *"Refusal in LLMs is mediated by a single direction"* — Arditi et al., 2024.
  Refusal behavior turns out to be a single (roughly rank-one) direction in the residual
  stream. The abliteration workflow computes that direction from contrastive
  harmful/harmless prompt pairs, then **projects it out of the weight matrices**, so the
  refusal activation can no longer form.
- **Tooling** (all open-source):
  - `FailSpy/abliterator` — the original interactive notebook pipeline.
  - `m14t/abliteration` (Max Tomeh's streamlined fork) — the one-click variant everyone
    runs now.
  - `Sumandora/remove-refusals-with-transformers` — the minimal reference implementation.
- **Naming**: `abliterated` = weights ablated via that technique (surgery — capability
  untouched); `uncensored` = the umbrella, including fine-tunes on unfiltered datasets
  (teaching — the model's overall behavior profile changes a bit). Both shipping
  communities label their repos accordingly, so search HF for either tag.

## The model shelf (as of Oct 2026, all shipping GGUF)

| Publisher / family | Sizes | Why reach for it |
|---|---|---|
| `huihui-ai/*-ablit` (Llama-3.x, Qwen-2.5, Mistral, Gemma-2/3, Phi…) | 0.5 B–72 B | most prolific publisher; GGUF for every size |
| `Qwen-2.5 abliterated` line | 3 B–32 B | strongest multilingual tier; community reviews consistently rank it closest to its censored source |
| `Mistral`-based uncensored fine-tunes | 7 B–22 B | the `uncensored` (dataset) flavor dominates this family — fine for casual everything |
| re-quants and mirrors | various | same weights, repackage GGUFs — fine when the original is download-hostile |

Practical selection rule: search HF for `abliterated gguf`, sort by downloads, and read
the last N review comments — the community is loud about regressions on any given size.

## Driving them on Kaggle — the free-quota-friendly path

The play: **text inference on CPU sessions burns zero GPU quota.** Your 30 h weekly GPU
budget stays reserved for diffusion/training; the 12 h CPU session cap per push covers
agents/prompt loops all day. (See [models-on-kaggle.md](models-on-kaggle.md) for the
persistence hooks.)

```bash
# CPU session — GPU is off, so this costs you nothing but wall-time:
%cd /kaggle/tmp
!git clone --depth 1 https://github.com/ggml-org/llama.cpp
!cmake -B build -DGGML_NATIVE=ON && cmake --build build -j --target llama-cli
# weights come from an attached dataset (see models-on-kaggle.md), so no re-download:
!/kaggle/tmp/llama.cpp/build/bin/llama-cli \
  -m /kaggle/input/<your-model-dataset>/model-Q4_K_M.gguf \
  -ngl 0 --temp 0.7 -p "<your prompt>" --no-display-prompt
```

- `-ngl 0` keeps every layer on CPU — llama.cpp will happily eat all of Kaggle's cores.
- A **7B at Q4_K_M** (~4.5–5 GB) is the sweet spot: snappy speed-on-CPU turnarounds and
  still small enough that 12 h sessions give thousands of generation calls.
- If a specific job truly warrants GPU (huge batch, embedding-heavy pipeline), a ≤7 B
  Q4 model fits one T4; anything bigger needs quantization anyway.

## What to expect (honest version)

- **Abliterated ≈ same model, refusal-free.** Public reports consistently measure
  ≤1–2% drift on standard benchmarks vs the censored base; the direction-removal
  surgery doesn't touch general capability.
- **Not magic**: it removes the trained refusal, it does not add skill. A bliterated 7 B
  still reasons like a 7 B — censored or not.
- **Edge behavior**: an occasional warning-leak or refusal-shaped ramble can still slip
  through on the most ambiguous prompts — a known, rare artifact of the projection.

## Responsibility section (short, non-negotiable)

- **License inheritance**: ablation edits weights — every base-model license (Llama
  Community License, Gemma, Qwen, Mistral terms) still applies to you, unchanged.
- **Same law as everywhere**: no CSAM, no malware/helper abuse, no stalking-adjacent
  harm, nothing illegal. None of this repo is written or operated for those, and both
  Kaggle's ToS and Hugging Face's model terms bind every step described here.
- **Your output is your signature** — you are the editor and the publisher; use these
  models the way you'd want anyone else to.
