# Agent Toolkit — driving Kaggle programmatically

Everything in this repo is executable, not aspirational: an agent session (OpenCode or
anything that can speak MCP or HTTPS) can do all of it with a Kaggle API token.

## Two transport options

### 1. Kaggle MCP server (76 tools)

`https://www.kaggle.com/mcp` — remote, **stateless**, authenticated **per tool call**:

- `Authorization: Bearer KGAT_<token>` header on every request — API tokens work directly,
  no OAuth dance needed (the server accepts them even though `/mcps`-style OAuth exists).
- Works through any `tools.kaggle.*`-capable client.

OpenCode config (global `~/.config/opencode/opencode.json`):

```jsonc
{
  "mcp": {
    "servers": {
      "kaggle": {
        "type": "remote",
        "url": "https://www.kaggle.com/mcp",
        "headers": { "Authorization": "Bearer {env:KAGGLE_API_TOKEN}" }
      }
    }
  }
}
```

with `export KAGGLE_API_TOKEN=...` in the shell profile. Never hardcode the token in the
config into a git-tracked file; `{env:}` substitution exists precisely for this.

### 2. Direct REST/JSON-RPC (fallback, zero-dependency)

Same protocol by hand:

```javascript
await fetch("https://www.kaggle.com/mcp", {
  method: "POST",
  headers: { "Content-Type": "application/json", "Accept": "application/json, text/event-stream",
             "Authorization": `Bearer ${token}` },
  body: JSON.stringify({ jsonrpc: "2.0", id: 1, method: "tools/call",
                         params: { name: "get_accelerator_quota", arguments: { request: {} } } }),
});
// SSE response: extract the `data: {...}` line, JSON-RPC result → content[0].text (JSON)
```

Useful when the registered MCP connection has no credentials yet (e.g. mid-session config
changes) — the tools are identical over either transport.

## Tool map (the ones that matter)

| Capability | Tools |
|---|---|
| **Notebooks** | `create_notebook_session`, `save_notebook` (push source → `SaveAndRunAll`), `get_notebook_session_status` (poll), `cancel_notebook_session`, `list_notebook_session_output`, `download_notebook_output(_zip)` |
| **Datasets** | `upload_dataset_file`, `update_dataset_metadata`, `get_dataset_status`, `list_dataset_(tree_)files`, `download_dataset`, `search_datasets` |
| **Models** | `list_models`, `get_model(_variation)`, `list_model_variation_versions`, `download_model_variation_version` |
| **Discovery** | `search_content`, `search_notebooks`, `search_competitions`, `get_notebook_info` (full source!) |
| **Competitions** | `get_competition`, `list_competition_data_files`, `get_competition_leaderboard`, submissions + uploads |
| **Account** | `get_accelerator_quota` (live-per-week budget), `get_user_profile` |

`get_notebook_info` on any public notebook returns its **full cell source** — the
community's working code is itself a reusable library.

## Scouting layer: FreeFlow MCP

An additional discovery server (`freeflow_mcp`) catalogs free AI tools/models with their
published limits and sources. Use it *before* spending Kaggle GPU time to check whether a
step is better served by a lightweight external (e.g., a hosted TTS/design API) — with the
caveat to verify exact current allowances there before relying on them.

## Security hygiene

- The token is account-scoped; anything with it **is** you. Treat leaks as incidents —
  rotate at kaggle.com → Settings → API.
- Keep tokens out of git repos (env-substitution in configs, `~/.bashrc` for shells).
- `save_notebook` publishes/versions code under your account — private notebooks stay
  private; never push a notebook whose source embeds the token.
