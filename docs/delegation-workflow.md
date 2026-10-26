# The Delegation Workflow — Kaggle as a batch-compute service

The pattern that makes Kaggle an agent-controllable compute farm: a loop of
**author → push → run headless → poll → harvest → handoff**, with datasets as the
message queue between stages.

## The loop

```
                ┌────────────────────────────────────────────┐
                │                                            │
  scene/config  │   push notebook (SaveAndRunAll)            │  poll status
  dataset  ───► │   + CPU/GPU settings + internet toggle     ├──────────────┐
                │                                            │              │
                └───────────────┬────────────────────────────┘              ▼
                            output files                     completed? ──► download output
                            (in /kaggle/working)                            (or output-zip)
                                   │
                                   ▼
                        turn deliverables into a dataset (via API upload)
                                   │
                                   ▼
                     next notebook attaches that dataset as input…
```

Tools that drive each step (all available as Kaggle MCP tools, see agent-toolkit.md):

| Step | Tool(s) |
|---|---|
| Author | local file (+ optional `save_notebook` with full source text) |
| Push & run | `save_notebook` — `kernelExecutionType: "SaveAndRunAll"` |
| Poll | `get_notebook_session_status` |
| Harvest | `download_notebook_output`, `download_notebook_output_zip`, `list_notebook_session_output` |
| Handoff | `upload_dataset_file` + `update_dataset_metadata`, `get_dataset_status` |
| Inputs | dataset/competition/model sources configured in the notebook metadata |
| Abort | `cancel_notebook_session` |

## Why `SaveAndRunAll` is the backbone

- Runs **untended**, browser closed, machine asleep. Idle timeouts don't apply.
- Up to the full 12-hour wall-clock — an entire working day of compute per push.
- Produces a deterministic, reproducible output bundle every push.

Interactive sessions are for debugging only (they die after ~20 idle minutes and burn GPU
quota while idle).

## Cost accounting with the loop in mind

- Each headless GPU run bills **wall-clock session time against the 30 h weekly GPU quota**,
  from queue start to session end — setup time counts. A session that's 90% weight-loading
  and 10% real work is an expensive session; persist weights to a **private dataset** (or a
  Kaggle Model entry) and attach them as input, so session N+1 starts loading from disk
  instead of the internet.
- Non-GPU stages (dataset pushes, stitching, TTS, packaging) should run as **CPU-only
  notebooks — zero GPU quota burned.
- Interactive debugging before batch runs should be short and cheap, or skipped by running
  a *smoke test* batch (1–2 minutes of work) as the first headless push.

## Template: chaining a long job (defeats the 12 h wall)

**Producer scripts must be checkpoint-friendly.** The general contract for every session:

```python
# pseudo-structure of any stage notebook
INPUT = "/kaggle/input/<previous-stage-dataset>"   # attached dataset
OUT   = "/kaggle/working"                          # becomes the output bundle

todo = load_state(INPUT, fallback=start_of_pipeline())   # resume marker
artifact = do_work(todo)                                 # idempotent chunk
write(artifact, OUT)                                     # plain files, small enough for upload
write_manifest(next_stage_todo, OUT)                     # what stage N+1 still needs to do
```

Then the agent (or a wrapper notebook, CPU-only) promotes the output files into a versioned
dataset, and the next stage attaches it. Recovery is always "re-push the same notebook;
state lives in the dataset".

## What not to use Kaggle for

- **Latency-sensitive or always-on workloads** — sessions are rented in 12-h chunks, not
  served 24/7. Serving a web app is the wrong shape.
- **Lives truly bigger than 12 h per chunk** — possible to chain, but a job needing
  >3 chained sessions with per-session setup overhead is likely cheaper on rented GPUs.
- Multiple Kaggle accounts to multiply quota — instant ToS ban, never worth it.
