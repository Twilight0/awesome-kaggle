# Quota & Hardware — the free tier, verified

Numbers below were verified against the live Kaggle API (`get_accelerator_quota`) and current
docs in **October 2026**. Kaggle publishes a *minimum guaranteed* quota; real allocation can
float higher when the fleet has idle capacity, but plan against the minimum.

## Accelerator quota

| Resource | Quota (verified) | Session cap |
|---|---|---|
| GPU (any of: 2× T4, 1× P100) | **30 h / week** (108,000 s) | 12 h per session |
| TPU (v3-8) | **20 h / week** (72,000 s) | 9 h per session |
| CPU sessions | generous, effectively unmetered for our purposes | 12 h per session |

- Quota refreshes on a **weekly schedule** (next refresh date is returned by the API).
- **Enabling a GPU consumes quota while the session exists, not only while it computes.**
  Idle interactive sessions burn GPU quota — another reason to prefer headless runs.
- GPU/TPU and internet require **one-time phone verification** on the account.

## What one GPU session actually gives you

Configuration for GPU sessions (T4×2):

- 2× Tesla T4 (16 GB each), usable as two independent devices or sharded model-parallel
- ~30 GB system RAM, 4+ vCPUs
- ~70 GB ephemeral scratch disk
- Full outbound internet when enabled (pip, git, Hugging Face downloads — community
  notebooks routinely download ~40 GB of model weights per session)
- ~20 GB writable `/kaggle/working` that becomes the notebook's **output bundle** when the
  session ends — capped at ~20 GB per version

## Storage model (the important design constraint)

| Layer | Lifetime | Limit | Use |
|---|---|---|---|
| Notebook *session* disk | dies with the session | ~70 GB | model weights, caches, scratch |
| Notebook *output* | survives as "Output" of the version | ~20 GB / version | small results, logs, the deliverables |
| **Datasets** | persistent, yours forever | effectively tens of GB each, private by default | the real file system |

Rules of thumb:

1. **Weights go to the session disk**, never to output (a 40 GB model dir will blow the
   output cap and fail the version upload).
2. **Deliverables and checkpoints go to datasets.** A dataset is created/updated via API,
   can be attached as an input to any notebook, and is the only way to carry state across
   sessions.
3. Long jobs are structured as **chains of headless notebooks** passing artifacts through
   datasets: `Session A → output dataset → Session B → …`. This defeats both the 12-hour
   wall and the output cap.

## Quota economics

- Weekly budget ≈ 2½ × one 12-hour session. One full-depth unattended run costs ~1/3 of
  the week and leaves slack for smaller jobs.
- CPU sessions are free in practical terms (the 30/20-hour meters count GPU/TPU only) —
  ffmpeg stitching, dataset uploads, data wrangling, and TTS should prefer **CPU-only
  sessions** to preserve GPU budget. Whisper transcription is the one counter-example
  (keep it on GPU or budget it).

## Session types

| Mode | Timeout | Use |
|---|---|---|
| Interactive | logs out after ~20 min idle; session cap applies | debugging a notebook |
| **Save & Run All** (headless/batch) | runs queued, executes top-to-bottom untended, up to 12 h | everything real |

Headless runs are the backbone: they are launched via API, polled for status, and their
outputs downloaded — the notebook runs with the browser closed, the machine asleep.
