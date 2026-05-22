# Using Hydrate

Hydrate runs invisibly underneath Claude Code — every prompt you
type is enriched with relevant past context, and every session is
distilled when you stop. The slash commands give you explicit
control for the moments when you want to steer that.

## Slash commands

### `/hydrate`

**Base recall.** Pulls facts from Hydrate's memory matching the
current topic and prints a short synthesis of what was loaded.

Use it when you've been deep in one conversation and want to
double-check that the relevant prior context is in scope, or
when starting a new topic mid-session.

```
/hydrate
/hydrate ingestion pipeline
```

### `/hydrate-last`

**Resume the most recent session.** Designed to run immediately
after `/clear` so the next prompt has full orientation without
you re-explaining anything. Scoped to the current project.

This is the headline product loop:

```
[long session]
/hydrate-distill        ← capture before the context gets thin
/clear
/hydrate-last           ← back where you left off, ~14 tokens against the daemon
```

A live test of the loop on a real 1902-turn session recovered the
in-flight task, scores, log paths, and cross-pane awareness for
~14 tokens — vs ~200K to re-paste the conversation.

### `/hydrate-project`

**Project audit.** Pulls the current project's canon (pinned
authoritative facts) plus the top-ranked semantic facts. Use it
when joining or re-joining a project to see the agreed conventions.

```
/hydrate-project
```

### `/hydrate-week`

**Last seven days of dream reports.** Hydrate runs a periodic
"dream cycle" that summarises recent activity into themes. This
command surfaces the last week's worth.

```
/hydrate-week
```

### `/hydrate-distill`

**Capture and compress the current session.** Run it before
`/clear` if you want a clean handover to a fresh context window.
Writes a four-tier summary (16 / 32 / 64 / key-facts) so a future
`/hydrate-last` has something tight to inject.

```
/hydrate-distill
```

The hooks already auto-capture every Stop event, so this is
explicitly for the "I'm about to `/clear` and want a tighter
snapshot than auto-capture would produce" case.

### `/hydrate-pack-load`

**Load a project pack.** Hydrate can export a project's memory as
a single `.hpack` file (canon + semantic facts + recent dream
summaries) so it can be shared between machines or teammates.
This command loads one.

```
/hydrate-pack-load ~/Downloads/payments-platform.hpack
```

### `/hydrate-help`

Reference table of the commands above. No tool calls.

```
/hydrate-help
```

## Patterns

### Distill / clear / last — the recovery loop

The most token-efficient way to start a fresh context window when
the current one is thin:

```
/hydrate-distill        ← four-tier compression + key-fact extraction
/clear                  ← new context window
/hydrate-last           ← briefing from the distill
```

Why this works: the distill is structured (key facts as bullets,
not prose), so `/hydrate-last` injects a tight summary instead of
a re-paste. The daemon's token cost for the recall is in the
double-digits even for multi-hour sessions.

### Pack onboarding

When sharing a project with a teammate (or yourself on another
machine), export the canon + recent context as a `.hpack`:

```
hydrate pack create --project=payments --out=payments.hpack
```

Send the file. Their `hydrate` loads it with:

```
hydrate pack load payments.hpack
```

…or, inside Claude Code:

```
/hydrate-pack-load payments.hpack
```

Pinned canon facts always inject; semantic facts surface when
relevant. A pack is portable — it doesn't carry your local DB,
just the memory the recipient should also have.

### First run in a project — auto-prime from CLAUDE.md

The first time Claude Code starts a session inside a project that
already has a `CLAUDE.md`, Hydrate runs `hydrate dehydrate` once
in the background to extract structured facts from it into the
local store. From then on those facts surface via `/hydrate` and
`/hydrate-project` like any other memory — the LLM no longer has
to re-read the file every turn to remember the project's rules.

The auto-prime:

- Runs once per project (marker at
  `~/.hydrate/auto-primed/<slug>`; delete it to retry).
- Detached subprocess — never blocks session start.
- Logs to `~/.hydrate/logs/auto-prime.log`.
- **Does not modify `CLAUDE.md`** — only reads it. Rewriting is a
  separate, opt-in CLI step (`hydrate dehydrate --mode=summary`).
- Skipped if a `HYDRATE.md` ledger is already present in the
  project (a prior explicit `hydrate dehydrate` run has covered it).

### CLAUDE.md ingestion — `hydrate dehydrate`

The slash-command surface above handles in-session memory. For
the one-shot "compress my existing CLAUDE.md and move the
knowledge into Hydrate" pass, use the CLI:

```sh
hydrate dehydrate                          # dry-run preview
hydrate dehydrate --apply                  # mode=summary (default)
hydrate dehydrate --apply --mode=stub      # collapse to a 5-line pointer
hydrate dehydrate --apply --mode=full      # extract facts only, leave file untouched
hydrate dehydrate --revert                 # restore CLAUDE.md.pre-hydrate.bak
```

Modes:

- **`summary`** (default) — rewrites CLAUDE.md to a compressed
  prose summary plus any operational sections preserved verbatim
  (build commands, hook config, setup steps). Readable by
  non-Hydrate users.
- **`stub`** — replaces the file with a ~5-line pointer plus
  preserved operational content. The smallest CLAUDE.md that still
  carries forward commands and hooks.
- **`full`** — leaves CLAUDE.md on disk untouched. Facts still
  land in Hydrate. This is what the first-run auto-prime uses.

Safety: `--apply` always writes `CLAUDE.md.pre-hydrate.bak`
before rewriting; the backup is never overwritten on subsequent
runs. `--revert` restores from it and deletes the facts the
ledger attributes to prior runs.

Idempotent: re-running on unchanged input is a no-op. The
`HYDRATE.md` ledger records source hashes so only changed files
go through the LLM.

### Project wiki — `hydrate wiki curate`

v0.6.0 ships the **autonomous project wiki**: a set of regenerable
markdown pages, derived from your codebase, that live in your repo
under `<project>/HYDRATE-wiki/`.

Default install — five canonical project pages plus one page per
`.go` source file (Purpose, Public API, Callers, Tests, Invariants):

```
HYDRATE-wiki/
├── 00-overview.md
├── 01-architecture.md
├── 02-commands.md
├── 03-mcp-and-integrations.md
├── 04-onboarding.md
└── files/.../<source>.go.md
```

Trigger:

- **Automatic** — `claude-session-start` fires the worker every
  6 hours via a detached subprocess. Cadence marker at
  `~/.hydrate/auto-wiki/<slug>.last`; delete it to force a
  re-curate.
- **Manual** — `hydrate wiki curate [DIR]` with optional
  `--max-pages-per-cycle=N`, `--dry-run`, `--wiki-dir=<path>`.
- **Dashboard** — `/wiki` route. Project picker, page list,
  rendered body, "Curate now" button.
- **MCP** — `curate_wiki` tool: callable from any MCP-capable
  client.

Each page carries YAML frontmatter recording the SHA of every
source file it cites. The next curate cycle queues a page for
re-author when any cited source changes, any source is deleted,
or the page is older than 60 days.

LLM use:

- **`claude` on PATH** → worker calls `claude -p` headlessly; no
  API key needed.
- **Env keys** (`ANTHROPIC_API_KEY` / `OPENAI_API_KEY`) or a
  configured local endpoint → used in that precedence order.
- **No LLM** → pages still render! The structural sections
  (Public API, Callers, Imports, Tests) come from the Go parser
  with zero LLM calls; the prose sections show a clear "no LLM
  available" hint where the summary would have lived.

**Multi-language out of the box.** v0.6.0 ships a pure-Go tree-sitter
runtime embedded in the binary, with hand-written queries for 11
languages: Go, Python, JavaScript, TypeScript / TSX, Rust, Java,
Ruby, Swift, C, C++. A Python or Rust repo gets the same per-file
pages a Go repo does. No installer, no download — the grammars live
inside the `hydrate` binary.

### Multi-tool memory

Same facts surface across every MCP-capable client. Capture them
once in Claude Code; recall them in Cursor, Cline, Zed, or Gemini
CLI via the `hydrate_recall`, `hydrate_save_fact`,
`hydrate_canon_add`, and related tools the MCP server exposes.

See [`INSTALL.md`](INSTALL.md#mcp-server-setup-non-claude-code-clients)
for the per-tool wiring snippets.

## Configuration

Sensible defaults out of the box; tune with:

```sh
hydrate config get
hydrate config set <key> <value>
```

Common knobs:

- `hydrate.mode` — `default` / `economy` / `turbo`. Economy
  extractively compresses any `docs/CONTEXT/*.md` reference
  material before injection; turbo skips that block entirely.
  (CLAUDE.md / AGENTS.md are handled by `hydrate dehydrate` —
  see below — not by per-prompt compression.)
- `dream.cycle` — `micro` / `standard` / `deep`. How aggressively
  the periodic dream summariser runs.

`hydrate doctor` flags any non-default config it sees.

## Status line + dashboard

When the daemon is running, the Claude Code status line shows:

```
↑ N saved · ctx X%
```

— where `N` is the cumulative characters Hydrate has injected for
this session, and `X` is the current context-window utilisation.

The dashboard at `http://localhost:<port>/` (port in
`~/.hydrate/server.port`) has a homepage with the five "what's
happening" cards, a project-activity ribbon, a memory pulse, a
recent timeline, and per-pane drill-throughs for sessions, facts,
dreams, packs, copilot, MCP-recent, retrievals, fatigue, and
orchestration. Every pane pushes updates live over SSE — no
manual refresh.

## Troubleshooting

If a slash command doesn't return anything useful:

1. `hydrate doctor` — confirm the daemon + hooks + MCP wiring
   are healthy.
2. `hydrate server start` — bring the daemon up if it's down.
3. Check the dashboard's `/mcp-recent` pane — every MCP call is
   logged with its tool name + response token count, so a quiet
   pane means the slash command isn't actually hitting the
   daemon.
4. File at [/issues](https://github.com/getHydrate/hydrate-public/issues)
   with the output of `hydrate doctor --report`.
