# ComfyUI for Agents — comfy-cli + MCP

The ComfyUI quickstart covers the UI path. This page covers the **agent path**: the
official CLI sibling that replaces manual notebook cells, and the MCP bridges that let
*this* session — or any agent — generate, iterate and re-graph images directly as tool
calls. No clicking at the far end.

## Part 1 — `comfy-cli`, the official CLI sibling

Comfy-Org's [`comfy-cli`](https://github.com/Comfy-Org/comfy-cli) is the scriptable single
command for the whole install/launch/run cycle (pip/Homebrew in its docs; same commands in
a Kaggle cell). It replaces our Cell 1 from the quickstart:

```bash
pip install comfy-cli      # then, one-time:
comfy setup                # configures versions + install path

comfy install              # clones ComfyUI + installs deps (≈ the manual clone cell)
comfy node install comfyui-impact-pack   # custom nodes, one command
comfy validate --workflow my_workflow.json   # catches missing nodes BEFORE running
comfy run --workflow my_workflow.json --wait # headless execution, blocks till done
```

Useful extras: `comfy node install --fast-deps` trades dependency-resolution strictness for
speed (`comfy install`, `comfy node install`, `comfy node reinstall`,
`comfy node restore-snapshot` all accept it), and `comfy node install-deps --workflow=<file>`
installs everything a given workflow JSON needs — the "Portability" step of headless batches.

### On Kaggle (replaces the quickstart's Cells 1 & 3)

```python
!pip -q install comfy-cli
!comfy install --fast-deps          # into the configured dir; keep it under /kaggle/tmp
!comfy node install <pack> --fast-deps
# download checkpoints exactly as in the quickstart (Cell 2), then:
!comfy run --workflow /kaggle/input/<your-dataset>/workflow.json --wait
```

One push = batch N images → `/kaggle/working` → dataset. The CLI makes the whole notebook
a one-liner per stage, which is exactly the delegation-workflow shape. Note the checkpoint
path stays the manual wget step — `comfy-cli` doesn't fetch model weights by default;
models-on-kaggle.md's attach-forever pattern remains the persistence layer.

## Part 2 — MCP: agents driving ComfyUI natively

Two community servers do the bridge, verified as active in October 2026:

| Server | Profile | Transport | When to pick it |
|---|---|---|---|
| [`artokun/comfyui-mcp`](https://github.com/artokun/comfyui-mcp) | **178 tools**: graph authoring/editing in natural language, image/video/audio, model & node management, defaults | MCP + its own sidebar agent UI | when the agent should *build* workflows for you |
| [`joenorton/comfyui-mcp-server`](https://github.com/joenorton/comfyui-mcp-server) | minimal (`generate_image`, `regenerate`), Python | streamable-HTTP on `:9000/mcp` | potato-simple: 2–3 min setup, agent calls one endpoint |

### Local potato setup (3 minutes)

1. ComfyUI running locally at `http://127.0.0.1:8188` (quickstart's potato section).
2. Run the light server (it sits in front of ComfyUI's HTTP API):

```bash
pip install -r <clone of joenorton/comfyui-mcp-server>/requirements.txt
python server.py            # serves http://127.0.0.1:9000/mcp
```

3. Register it with the agent — in `opencode.json`, alongside the kaggle entry:

```jsonc
"mcp": {
  "servers": {
    "kaggle":   { "type": "remote", "url": "https://www.kaggle.com/mcp",
                  "headers": { "Authorization": "Bearer {env:KAGGLE_API_TOKEN}" } },
    "comfyui":  { "type": "remote", "url": "http://127.0.0.1:9000/mcp" }
  }
}
```

After that, "generate a hero image for the thumbnail, then make the mistier variant" is
two tool calls from this session — the agent inspects the result and iterates itself.

### On Kaggle (remote GPU session)

The MCP bridge must run *next to* ComfyUI, then be tunneled like the UI itself:

- ComfyUI + bridge server on the notebook (bridge on `:9000`),
- two `cloudflared` quick tunnels (one for ComfyUI `:8188`, one for the bridge `:9000`),
- point your laptop-side MCP client at the bridge tunnel URL.

Honest trade-offs, unchanged from the connection guide: tunnel URLs are ephemeral, treat
them as password, and sessions die — long agent sessions favor the *local* potato path,
with Kaggle reserved for batch production in headless mode (Part 1).

## Why this matters for the agent stack (the bigger pattern)

The pieces now compose end-to-end:

| Layer | Tool | Agent status |
|---|---|---|
| Kaggle batch compute | kaggle MCP (76 tools) | already wired in this session |
| Image/video engine | ComfyUI | driven headless (CLI) or natively (MCP) |
| Graph authoring | artokun MCP or `Save (API format)` JSONs | natural-language editing |
| 3D/renders | Blender MCP — same pattern, `execute_blender_code` runs Python in live Blender | the same control style as everything above |

The one rule that keeps it sane: **agents improvise; servers persist.** Anything
proven (a workflow, a node set, a dataset) gets committed — to git, a Kaggle dataset, or
a workflow file — so the next session resumes from artifacts, not from vibes.
