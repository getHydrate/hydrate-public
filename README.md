# Hydrate

**The platform layer for your coding agents.**
*Memory · orchestration · token reduction · cross-agent — one daemon, no cloud.*

Hydrate began as cross-platform, token-reduced memory. It's now the shared
platform layer underneath every coding agent you use. One small local
daemon gives Claude Code, OpenAI Codex, Cursor, Mistral Vibe and Copilot
the things they don't have on their own — and your work never leaves your
machines unless you turn it on.

## What you get

- **Memory across sessions and runtimes.** Your agent forgets every
  `/clear` or new session; Hydrate doesn't. The same local store feeds
  every runtime — start in one, recall in another.
  → [Works with](#works-with)
- **Orchestration you can trust.** An adversarial, cross-family multi-agent
  loop with fail-closed gates — work you can leave running because it won't
  loop forever or lie about being done.
  → [Orchestration](#orchestration--multi-agent-work-you-can-leave-running)
- **Token reduction that pays for itself.** A measured −11.1% output tokens
  at quality parity — memory that shrinks the bill instead of eating your
  context. → [Benchmarks](#benchmarks)
- **Cross-agent coordination.** Peernet lets activated sessions find each
  other over your own network and ask each other questions — opt-in,
  authenticated, audited.
  → [Peernet](#peernet--give-your-agents-a-way-to-talk-to-each-other)
- **Local-first & auditable.** Five Go binaries under 10 MB — no torch, no
  GPU, no background uploaders, no third-party calls in the hot path.

## Why Hydrate

*Without Hydrate:* every `/clear` is amnesia, and each runtime is its own
island — your agent re-asks what stack you use for the 12th time this week.

*With Hydrate:* your conventions, decisions and corrections are already
there on the next turn — in Claude today, in Codex tomorrow — and you pay
fewer tokens to use them.

Two things set Hydrate apart from other memory tools:

- **We're honest about what actually reaches your model.** We
  *delivery-test* every runtime and mark our own gaps (see
  [Works with](#works-with)) — "supported" means we proved the context
  reached the model, not that we wrote a config file. Most tools claim a
  runtime once the config is written; we verify the model received it.
- **Private by default — no telemetry of any kind.** Local, pure-Go, no
  model download, nothing phoned home, no usage analytics. Your work never
  leaves your machine.

## Works with

Hydrate reaches a runtime over three independent channels: an **inject
hook** (pushes remembered context into the model at session start), a
**capture hook** (ingests the transcript when a session ends), and **MCP**
(the model pulls memory on demand). They're independent — and we're honest
about which actually reach the model on each host, because a hook that
*fires* isn't the same as one the host *delivers*.

| Runtime | Auto-recall (inject) | Capture | MCP recall | Status |
|---|:--:|:--:|:--:|---|
| **Claude Code** | ✅ | ✅ | ✅ | full auto-memory |
| **OpenAI Codex** | ✅ | ✅ | ✅ | full auto-memory |
| **Mistral Vibe** | ✅ (fork) | ✅ | ✅ | full (fork) |
| **Cursor** | ⚠️ host-gated | ✅ | ⚠️ registered | capture + MCP |
| **Antigravity** | ❌ host-gated | ✅ | ✅ | capture + MCP |
| **GitHub Copilot** | ⚠️ no session-start inject | ✅ | ✅ | capture + MCP |
| **IBM Bob** | ❌ (host hooks inert) | ➖ | ✅ | in development |

On Claude Code and Codex, recall is automatic every session. Where a host
withholds injected context from the model (Cursor, Antigravity), Hydrate
still **captures** every session and serves recall over **MCP**. We mark
our own gaps rather than tick every box — full method and per-runtime
evidence in [**docs/RUNTIME_INTEGRATION.md**](docs/RUNTIME_INTEGRATION.md).

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
now reading from and writing to your memory on every session. (On
Codex the same auto-injection applies; on Cursor, Antigravity and
Copilot, Hydrate still captures every session and recall comes through
MCP — see [Works with](#works-with).)

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

## Orchestration — multi-agent work you can leave running

Orchestration replaces "one model, one shot" with a persisted,
**adversarial** loop: a generator and an *independent, cross-family*
critic argue through bounded rounds over a tracked ledger of objections,
converging only when material defects clear and measurable acceptance
criteria are met — with **fail-closed gates** that escalate to a human
rather than ever ship a false pass. The local `hydrate-server` daemon
drives it as a pure, resumable state machine (all state in SQLite), and
it's exposed as MCP tools (`design_*`, `develop_*`, …) so **any connected
runtime can drive it** — not locked to one vendor's agent.

| Mode | Artifact | Generator → Reviewer | Terminal gate |
|---|---|---|---|
| **Design** | plan / spec | Author (Opus) → Critic (Codex), human arbitrates | Sign-off: zero material objections + acceptance met |
| **Develop** | code patches | Implementer → Reviewer → Judge → Audit | Human integrate gate + post-merge audit |
| **Image** | generated images | Generator → Judge (design-judgement) | Accept |
| **Studio** | UX design → build | Designer → approve/revise → implement | Design approval + build accept |

Why it produces trustworthy work:

- **Adversarial, cross-family review** — the critic is prompted to refute,
  not rubber-stamp, and is a *different model family* so one model's blind
  spots aren't shared by its reviewer.
- **Objection ledger (severity + basis)** — every concern is tracked,
  deduped, recurrence-counted; nothing is silently dropped, and you get an
  auditable record of *why* an artifact changed.
- **Convergence test + round cap** — stops only at zero-material +
  diminishing returns; won't ship with open defects, won't polish forever,
  and escalates honestly to `needs_human` when stuck.
- **Fail-closed gates** — a gate that can't render a verdict goes to
  `needs_human`, never false-green.
- **Pure state machines + SQLite** — deterministic, unit-tested,
  resumable across daemon restarts.

The payoff is **trustworthy autonomy** — work you can leave running
because it won't loop forever, won't lie about being done, and leaves an
auditable trail of every decision. *(This README's plan was itself
pressure-tested through Design mode.)*

## Peernet — give your agents a way to talk to each other

Hydrate gives your coding agents shared memory. **Peernet** gives them a
way to talk to each other.

It is **opt-in and local-first**. Activated agent sessions find each other
over your own network (LAN, VPN, or Tailscale) with no central server,
pair with a short code, and exchange authenticated, audited messages.
Nothing leaves your machines unless you turn it on.

On top of Peernet, the **session-addressed relay** lets you name a live
session and ask it a question directly. The daemon never answers; the
target agent does, on its next turn, from its own working context:

```sh
hydrate ask <target-project> "what did you mean by 'gate' in the handoff?"
```

Whether that answer is released straight away or held for a person to
approve is set per project by a **release policy** (auto or human), so
disclosure is governed in one place rather than buried in each runtime's
permission prompts.

The one thing that varies by runtime is **autonomy**: can the named
session answer on its own, or must a human approve the tool call first?

| Runtime | Autonomous answering | How |
|---|:--:|---|
| **Codex** | ✅ | Answers peer questions unprompted |
| **Claude** | ✅ | Relay tools pre-granted at install |
| **Copilot** | ✅ | Per-repo MCP approval written at install |
| **Mistral Vibe** | 🔜 | Planned |

> **Status.** Free-text relay answers work between sessions on the same
> machine today. Cross-device answering rides Peernet's v2 peer-onboarding
> and is on the roadmap; the cross-machine peer round-trip itself has been
> demonstrated over Tailscale.

## Benchmarks

- **Token reduction (the headline).** On a real coding-task A/B
  (`lquery`, n=10), Hydrate cut **output tokens −11.1%** and **cost −6.3%**
  at **quality parity** — memory that pays for itself rather than costing
  context budget.
- **Retrieval.** LongMemEval **recall@10 = 0.86** (0.95 on multi-session
  questions) — the right context is in the top results almost every time.
- **Compression.** The distil pipeline compresses captured sessions
  **99.2%** (475.7M → 3.7M tokens across 22k+ sessions) before they re-enter
  a prompt.
- **Speed & footprint.** **Sub-10 ms** local retrieval, **no reranker, no
  GPU, no model download** — a pure-Go binary you can audit in an afternoon.

Full methodology + head-to-head comparisons:
[gethydrate.dev](https://gethydrate.dev).

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
