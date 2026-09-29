# pod-marimo

The `marimo` candy of the OpenCharly candy library, as a standalone repo
(kind-prefixed naming). It ships the marimo reactive notebook server — which also
runs as an MCP server on the same port — with the GPU OSM analytics stack, the
Apache Airflow Python deps, and the `marimo-team/skills` set for AI agents.

## What it provides

Installs marimo (a reactive notebook server that also runs as an MCP server via
`--mcp` on port `2718`) plus its data stack into a pixi env at
`${HOME}/.pixi/envs/default`: GPU-accelerated OSM analytics deps
(Polars-GPU/cuDF, geopandas, quackosm), Apache Airflow Python deps (the airflow
layer ships no pixi env of its own), and the `marimo-team/learn` curriculum. The
`marimo-team/skills` set is unpacked read-only at `/opt/marimo-skills`
(`MARIMO_SKILLS_DIR`) for AI agents driving marimo over MCP, and a baked
`marimo.toml` turns `auto_instantiate` on so cells run the moment the notebook URL
opens.

| Property | Value |
|---|---|
| Service | `marimo` (`marimo edit --host 0.0.0.0 --port 2718 --no-token --headless --mcp --mcp-allow-remote /workspace`, priority 30) |
| Port | `2718` (notebook UI + MCP at `/mcp/server`) |
| Requires | `layer-cuda`, `layer-supervisord`, `plugin-mcp` (the out-of-process `mcp:` check verb) |
| Volume | `workspace` at `/workspace` |
| Env | `NVIDIA_PYTHON_PROJECT=~/.pixi`, `LD_LIBRARY_PATH=/usr/lib64`, `MARIMO_SKILLS_DIR=/opt/marimo-skills` |
| mcp_provide | `marimo` at `http://{{.ContainerName}}:2718/mcp/server` (http transport) |

`--mcp-allow-remote` is mandatory because the server binds `0.0.0.0` (marimo
refuses non-localhost MCP requests without it); `--no-token` keeps the editor
token-free for the dev posture. Every claim is observable: the pixi-env binaries,
the skills directory, the config line, and — at deploy scope — the live notebook
UI plus the MCP endpoint on `/mcp/server`.

## How to use it

```bash
charly box build marimo
charly config marimo
charly start marimo
# notebook UI:  http://localhost:2718
# MCP endpoint: http://localhost:2718/mcp/server
```

The candy's own `check:` steps assert the `marimo` and `airflow` pixi binaries,
the unpacked skills directory, the baked `auto_instantiate` config, the OSM/MCP
curriculum imports, the reachable port and HTTP 200, and the `mcp:` `ping` /
`list-tools` surface.

## Layout

- `charly.yml` — the `marimo:` candy entity (description, `require`, `env`,
  `env_accept`, `mcp_provide`, `volume`, `service`, `plan`).
- `pixi.toml` / `pixi.lock` — the Python environment (marimo, the OSM/GPU stack,
  Airflow deps).
- `build.sh` — the pixi-builder post-install (the three PyG native extensions).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-versa:marimo-layer` — the marimo layer, its pixi
  environment, the supervisord service spec, and the rendering patterns.
- `/charly-versa:marimo-mcp` — marimo's built-in MCP server and its read-only
  tool catalog.
- `/charly-versa:notebook-osm` — the self-authoring OSM notebook this env drives.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
