# Hydrate

**Local-first persistent memory for Claude Code.**

Hydrate runs as a small local daemon that watches every Claude Code
session, distils what mattered, and quietly re-injects the relevant
parts into your next prompt. Memory across sessions, across projects,
across machines you sync — without sending your work to anyone else.

## Why

- **Memory across sessions.** Claude Code forgets every `/clear`.
  Hydrate doesn't.
- **Works offline.** The daemon, the index, and the MCP server all
  run on your machine. No third-party calls in the hot path.
- **One binary, auditable surface.** Five Go binaries under 10 MB
  total, no kernel hooks, no background uploaders.

## Quick install

### macOS / Linux — Homebrew

```sh
brew tap getHydrate/hydrate
brew install hydrate
hydrate setup
```

### One-line installer (macOS / Linux)

```sh
curl -fsSL gethydrate.dev/install | sh
```

### Manual tarball (any platform)

Grab the platform tarball from
[Releases](https://github.com/getHydrate/hydrate-public/releases/latest)
and drop the binaries into `~/.local/bin/`.

### Windows

Download `hydrate-v0.4.0-windows-amd64.zip` from
[Releases](https://github.com/getHydrate/hydrate-public/releases/latest)
and unzip into a directory on `PATH`. An MSI installer is on the
roadmap.

After install, run:

```sh
hydrate setup           # interactive first-run wizard
hydrate doctor          # 16-point health check
```

Or fill the form at **[gethydrate.dev/install/first-steps](https://gethydrate.dev/install/first-steps)** —
it generates a five-stage script (license → project init → optional
hydration-pack import → optional team-git bootstrap → doctor)
that you copy and paste into your terminal.

Open Claude Code, run any prompt, and look for the `[hydrate]`
context block prepended to your first turn. That's it — Hydrate is
now reading from and writing to your memory on every session.

The first session you start inside a project that already has a
`CLAUDE.md` triggers a one-shot background pass that extracts the
file's contents into Hydrate's fact store, so the LLM doesn't have
to re-read the whole file every turn. The file itself is left
untouched — see [`USAGE.md`](USAGE.md#first-run-in-a-project--auto-prime-from-claudemd)
for the opt-in `hydrate dehydrate` flow if you want to also
shrink the on-disk file.

Full install paths (Linux tarball, Windows zip, MCP snippets, manual
hooks): see [`INSTALL.md`](INSTALL.md).

## What you can do

Hydrate ships seven slash commands inside Claude Code:

- `/hydrate` — recall relevant context for the current topic
- `/hydrate-last` — resume the most recent session (the
  `/hydrate-distill → /clear → /hydrate-last` recovery loop is the
  headline product flow)
- `/hydrate-project` — pull the current project's canon + facts
- `/hydrate-week` — last seven days of dream reports
- `/hydrate-distill` — capture and compress the current session
  before `/clear`
- `/hydrate-pack-load` — load a project pack (shareable `.hpack`)
- `/hydrate-help` — reference table

Plus a CLI rich enough that the dashboard's Builders tab can scaffold
almost every common workflow:

- **`hydrate init`** — turn an existing repo into a Hydrate project, with the LLM drafting starter canon
- **`hydrate dehydrate`** — ingest `CLAUDE.md` (and other markdown docs) into Hydrate's fact store, with optional rewrite to a summary or stub (reversible). Runs automatically in `--mode=full` (no rewrite) the first time you open a project.
- **`hydrate wiki curate`** — autonomous project wiki. Materialises five canonical project pages plus a Purpose/Public-API/Callers/Tests/Invariants page per source file into `<project>/HYDRATE-wiki/`, regenerable from the code. Runs automatically every six hours on session start; visible to every runtime via plain markdown in the repo.
- **`hydrate canon add` / `hydrate fact save`** — pin invariants or record incidental knowledge
- **`hydrate pack create` / `hydrate pack import`** — share a project's memory as a single `.hpack` file
- **`hydrate backup` / `hydrate restore`** — encrypted archive of a project's state
- **`hydrate team push` / `pull` / `status`** — git-as-sync-layer for shared canon
- **`hydrate skill-mine` / `skill-draft` / `skill-install`** — detect recurring tool-call workflows in your sessions and turn them into Claude Code skills
- **`hydrate narrate`** — weave session JSONLs + git history + project docs into a `NARRATIVE.md` (use `--update` for incremental appends)
- **`hydrate orchestrator set` / `get` / `dispatch`** — multi-pane tmux orchestration with state pinned to canon

Detailed usage + patterns: [`USAGE.md`](USAGE.md). Or open the
dashboard at `/help` for interactive builders that emit the exact
commands for your settings.

## Dashboard

`hydrate-server` exposes a local web dashboard at
`http://localhost:<port>/` (the port is written to
`~/.hydrate/server.port` on startup; or run `hydrate dashboard` to
open it). Every pane streams updates live over SSE.

The dashboard has eight pages, with three of them carrying tabbed
sub-sections:

- **Home** — live cockpit (token usage, rollups, active projects, orchestrations).
- **Displacement** — sessions / context-window pressure / model mix over time.
- **Orchestration** — three tabs: **Live** (in-flight orchestrations), **Build orchestration** (form-driven tmux session generator with one window per agent, save/load to localStorage, optional `hydrate orchestrator set` lines so the layout is recallable from SQLite), **Customise env** (env-var script builder).
- **Fatigue** — per-project distill / refresh cadence.
- **Facts** — force-directed graph of every learned fact, filter by project / area / time / pinned.
- **Skills** — mined workflow patterns; Draft + Install buttons turn one into a Claude Code skill.
- **Settings** — two tabs: **Current settings** (what's configured) and **Customise** (script builder for `HYDRATE_*` env vars).
- **Help** — two tabs: **Reference** (every CLI command with examples) and **Builders** (eight task builders).

The dashboard is loopback-only by default; remote access requires
your API key (written to `~/.hydrate/server.key`).

## Status

Beta. Free tier available, Pro tier in closed beta, Enterprise on
request. See [gethydrate.dev/pricing](https://gethydrate.dev/pricing).

## Documentation

- [`INSTALL.md`](INSTALL.md) — full install guide (macOS / Linux /
  Windows / MCP snippets / troubleshooting)
- [`USAGE.md`](USAGE.md) — slash commands + the recovery patterns
- [`SECURITY.md`](SECURITY.md) — reporting vulnerabilities
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — how to file issues
- [`ROADMAP.md`](ROADMAP.md) — what's next

Homepage + benchmarks + comparisons: [gethydrate.dev](https://gethydrate.dev).

## Get the source

Hydrate's source is proprietary; this repo distributes signed
binaries only. For commercial / source-access enquiries email
`licences@gethydrate.dev`.

## Issues

File bugs at [/issues](https://github.com/getHydrate/hydrate-public/issues).
The fastest route is `hydrate doctor --report`, which pre-fills a
GitHub issue URL with your system info and the diagnostic output.

## Licence

See [`LICENSE`](LICENSE). Binaries are distributed under the Hydrate
end-user licence agreement shipped inside each release tarball.
