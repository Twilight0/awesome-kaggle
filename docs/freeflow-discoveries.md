# FreeFlow Scouting Pass — hosted free tools vs. the Kaggle pipeline

What the FreeFlow catalogue (MCP `freeflow_mcp`) returned when queried against each stage
of our planned pipelines, on **2026-10-26**. Ground rules that apply to every entry:

- Allowances below are **published findings** (each row's `checked` date is the catalogue's
  last source check), **not live account balances** and **not generation tests by us**.
- Editorial scores are FreeFlow *desk reviews* — not user ratings or benchmarks.
- Before relying on any of these, re-verify on the provider's own page (`check_tool_source`
  does a fresh public-source read; hosted free tiers change constantly).

## The structural verdict first

The catalogue is dominated by **interactive, browser-bound tools with small daily
allowances**. Almost nothing in it speaks HTTP-API batch the way Kaggle notebooks do —
even the free LLM plans explicitly exclude API usage. So the architectural conclusion:

| Stage role | Where it belongs | Why |
|---|---|---|
| Batch production (clips, transcripts, edits, datasets) | **Kaggle** | no daily caps; batch-shaped by design |
| One-off samples, model auditions, thumbnails, inspiration | **hosted free tools** | zero setup, zero GPU quota; caps are irrelevant at N≤10 |
| Batch text inference | **Kaggle CPU (GGUF)** — always | free LLM web plans grant no batch API access |
| Premium-voice narration (occasional) | **ElevenLabs free tier** | quality per credit is worth more than local TTS for flagship takes |

## Stage tables (candidates that passed the fit filter)

### Video generation (45 catalogued; curated to relevant)

| Tool | Allowance (checked) | Catch | Our use |
|---|---|---|---|
| **Giz.ai** | "2 free uses/day for **H3 Turbo and LTX-2.5 Distilled**" (2026-10-20) | model cards limit to 480p–736p, not a free pass to every model | **audition our exact target models** before spending GPU quota |
| Google Flow | 50 credits/day, no rollover (2026-10-15) | free video gen may be down 14:00–17:00 UTC | occasional means/look references |
| Genspark | 100 credits/day (2026-10-15) | per-model cost varies; listing ≠ free access to every version | general sampling |
| Leonardo AI | 150 tokens/day (2026-10-15) | tokens ≠ videos; free creations are **public** | thumbnails, style probes |
| Arena | 2 requests/24 h | 5–8 s outputs; models chosen by battle mode | curiosity only |

Honest math: even stacked, hosted tiers yield maybe **a minute of footage per day max** —
gratefully used for inspiration, useless for production volume. Production stays on
Kaggle (see video-generation.md).

### Narration / TTS

| Tool | Allowance (checked) | Catch |
|---|---|---|
| **ElevenLabs** | 10,000 credits/month free (≈10 min of speech) | commercial rights depend on the plan — check before publishing |
| TTSMaker | weekly character quota, downloadable audio (2026-10-13) | voice-dependent limits |
| ttsmp3 | daily text limits | no unlimited-use promise |

Local alternative of record: edge-tts/Kokoro-class on **Kaggle CPU sessions — zero quota**.
Use hosted voices for flagship takes, local for drafts and bulk.

### Music (the one gem for our pipeline)

| Tool | Allowance | Catch |
|---|---|---|
| **MusicGen (AudioCraft)** — *run: local* | "Local music models · non-commercial weights" (2026-10-13) | **code MIT, weights CC-BY-NC 4.0** — hobby videos yes, monetized videos no |

This is the highest-value discovery for the guides: MusicGen is a **Kaggle-native stage** —
it runs on the free T4 (or slowly on CPU sessions) and slots into the long-form pipeline
between clip generation and the ffmpeg stitch, at zero hosted-tool friction. Worth a stub
the moment a video pipeline exists; until then it's one line in the projects guide.

Hosted alternates (both non-commercial free): AIVA (3 downloads/mo, they own copyright —
you credit), Mubert (25 tracks/mo, attribution rules).

### Images

| Tool | Allowance (checked) | Our use |
|---|---|---|
| Microsoft Bing Image Creator | 10 Fast creations/day (2026-10-13) | thumbnail stills, thumbnail tests |
| Perchance | free, no signup advertised | throwaway quick images |

Everything batch stays in ComfyUI-on-Kaggle (image-editing.md).

### Transcription

No hosted free ASR worth shortlisting — the cache returned TTS tools. Whisper-on-Kaggle
GPU (projects guide) remains our batch route, CPU/GPU tiering per quota doc.

## Actions adopted into the guides from this pass

1. `video-generation.md`: audition hosted Giz.ai uses of LTX-2.5/H3 Turbo **before**
   spending GPU quota on the Phase 0 benchmark.
2. `projects-that-fit-the-window.md`: MusicGen row in the batch-inference table
   (non-commercial weights caveat noted).
3. Narration strategy tier: Kaggle-local TTS (free) → ElevenLabs 10k/mo (flagship takes) → paid, only when monetized.
4. Standing rule recorded here: hosted tools sample; Kaggle produces.
