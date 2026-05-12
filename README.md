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

```sh
curl -fsSL gethydrate.dev/install | sh
```

The script detects your OS + architecture, downloads the matching
tarball from this repo's [Releases](https://github.com/getHydrate/hydrate-public/releases),
installs the binaries to `~/.local/bin/`, wires the Claude Code
hooks into `~/.claude/settings.json`, and starts the local daemon.

After it finishes:

```sh
hydrate install-hooks   # idempotent; re-runs the hook + MCP wiring
hydrate doctor          # 17-point health check
```

Open Claude Code, run any prompt, and look for the `[hydrate]`
context block prepended to your first turn. That's it — Hydrate is
now reading from and writing to your memory on every session.

Full install paths (Linux tarball, Windows MSI, manual hooks): see
[`INSTALL.md`](INSTALL.md).

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

Detailed usage + patterns: [`USAGE.md`](USAGE.md).

## Dashboard

`hydrate-server` exposes a local web dashboard at
`http://localhost:<port>/` (the port is written to
`~/.hydrate/server.port` on startup). Every pane streams updates
live over SSE — sessions, facts, dreams, packs, the displacement
view, orchestration, fatigue, and a homepage with the five
"what's happening right now" cards.

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
