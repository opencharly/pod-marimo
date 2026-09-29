# AGENTS.md — pod-marimo

Standalone candy repo for the `marimo` candy — the marimo reactive notebook
server (which also runs as an MCP server on the same port) with the GPU OSM
analytics stack, the Apache Airflow Python deps, and the `marimo-team/skills` set
for AI agents. The candy lives in `charly.yml` at the repo root plus its pixi
environment.

Canonical files:

- `charly.yml` — the `marimo:` candy entity (description, `require`, `env`,
  `env_accept`, `mcp_provide`, `volume`, `service`, `plan`).
- `pixi.toml` / `pixi.lock` — the Python environment (marimo, the OSM/GPU stack,
  Airflow deps).
- `build.sh` — the pixi-builder post-install (the three PyG native extensions).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-versa:marimo-layer` — the owning skill: the marimo layer, its pixi
  environment, the supervisord service spec, and the cell-display / `mo.iframe`
  rendering patterns. Load before editing, building, deploying, or
  troubleshooting this candy.
- `/charly-versa:marimo-mcp` — marimo's built-in MCP server and its read-only
  tool catalog (the cells-don't-execute-via-MCP gap).
- `/charly-pod:pod` — the `kind: pod` / deploy schema reference (this candy is
  composed into a box; tree-position nesting, volumes, ports).
- `/charly-check:check` — the check/R10 framework: the `check:` step verbs, the
  `mcp:` verb, and `charly check run <bed>`.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs, `mcp_provide`, service declarations).
- `/charly-internals:git-workflow` — before any git/PR action.

The candy carries no `skill:` entity of its own; the family skill
`/charly-versa:marimo-layer` covers the surface. The gap is routed to the named
skill-authoring batch
[opencharly/opencharly#291](https://github.com/opencharly/opencharly/issues/291);
when that lands, add the owning skill here.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The live R10 witness is a composing box's `check` bed; the candy's own `check:`
  steps assert the `marimo` and `airflow` pixi binaries, the unpacked
  `/opt/marimo-skills/skills` directory, the baked `auto_instantiate` config, the
  OSM/MCP curriculum imports, the reachable port and HTTP 200, and the `mcp:`
  `ping` / `list-tools` surface against `mcp_name: marimo`.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.

## Modify this repo

- Edit the `marimo:` candy entity in `charly.yml`.
- Python dependency changes belong in `pixi.toml` / `pixi.lock`, not in the plan;
  the PyG native extensions that need `pip --find-links` live in `build.sh` and
  stay ABI-pinned to the `torch` version in `pixi.toml`.
- The `--mcp-allow-remote` flag and the `mcp_provide` URL path (`/mcp/server`, NOT
  `/mcp`) are the MCP contract; the service exec and the URL must stay in step.
- The `workspace` volume at `/workspace` is the persistent notebook store; the
  notebook content itself belongs to the `notebook-osm` data layer, not an inline
  `write:` step (the volume-shadow anti-pattern).

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
