# Hydrate

**The platform layer for your coding agents.**
*Memory, orchestration, quality uplift, token reduction, cross-agent. One daemon, no cloud.*

Hydrate began as cross-platform, token-reduced memory. It is now the shared
platform layer underneath every coding agent you use. One small local daemon
gives Claude Code, OpenAI Codex, Cursor, Mistral Vibe and Copilot the things
they lack on their own, and your work never leaves your machines unless you
turn it on.

## What you get

- **Memory across sessions and runtimes.** Your agent forgets every `/clear`
  or new session; Hydrate does not. The same local store feeds every runtime,
  so you start in one and recall in another. See [Works with](#works-with).
- **Orchestration that raises the quality bar.** An adversarial, cross-family
  multi-agent loop with fail-closed gates. It catches what a single pass
  misses, and it will not loop forever or claim to be done when it is not. See
  [Orchestration](#orchestration).
- **Token reduction.** A measured 11.1% fewer output tokens at equal quality,
  so memory shrinks the bill instead of eating your context. See
  [Benchmarks](#benchmarks).
- **Cross-agent coordination.** Peernet lets activated sessions find each
  other over your own network and ask each other questions, opt-in,
  authenticated and audited. See [Peernet](#peernet).
- **Local-first and auditable.** Five Go binaries under 10 MB total, with no
  kernel hooks, no background uploaders and no third-party calls in the hot
  path.

## Why Hydrate

*Without Hydrate:* every `/clear` is amnesia, and each runtime is its own
island. Your agent re-asks what stack you use for the twelfth time this week.

*With Hydrate:* your conventions, decisions and corrections are already there
on the next turn, in Claude today and Codex tomorrow, and you pay fewer
tokens to use them.

Two things set Hydrate apart from other memory tools:

- **We are honest about what actually reaches your model.** We delivery-test
  every runtime and mark our own gaps (see [Works with](#works-with)).
  "Supported" means we proved the context reached the model, not that we
  wrote a config file. Most tools claim a runtime the moment the config is
  written; we check the model received it.
- **Private by default, with no telemetry of any kind.** Local, pure Go, no
  model download, nothing phoned home and no usage analytics. Your work never
  leaves your machine.

## Works with

Hydrate reaches a runtime over three independent channels. An **inject hook**
pushes remembered context into the model at session start, a **capture hook**
ingests the transcript when a session ends, and **MCP** lets the model pull
memory on demand. They are independent, and we are honest about which ones
actually reach the model on each host, because a hook that fires is not the
same as one the host delivers.

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
withholds injected context from the model, as Cursor and Antigravity do,
Hydrate still captures every session and serves recall over MCP. We mark our
own gaps rather than tick every box. The full method and the per-runtime
evidence are in [docs/RUNTIME_INTEGRATION.md](docs/RUNTIME_INTEGRATION.md).

## Quick install

### macOS / Linux, Homebrew

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
[Releases](https://github.com/getHydrate/hydrate-public/releases/latest) and
drop the binaries into `~/.local/bin/`.

### Windows

Download `hydrate-v0.4.0-windows-amd64.zip` from
[Releases](https://github.com/getHydrate/hydrate-public/releases/latest) and
unzip into a directory on `PATH`. An MSI installer is on the roadmap.

After install, run:

```sh
hydrate setup           # interactive first-run wizard
hydrate doctor          # 16-point health check
```

Or fill the form at **[gethydrate.dev/install/first-steps](https://gethydrate.dev/install/first-steps)**.
It generates a five-stage script (licence, project init, optional
hydration-pack import, optional team-git bootstrap, doctor) that you copy and
paste into your terminal.

Open Claude Code, run any prompt, and look for the `[hydrate]` context block
prepended to your first turn. That is it: Hydrate now reads from and writes
to your memory on every session. On Codex the same auto-injection applies; on
Cursor, Antigravity and Copilot, Hydrate still captures every session and
recall comes through MCP (see [Works with](#works-with)).

The first session you start inside a project that already has a `CLAUDE.md`
triggers a one-shot background pass that extracts the file's contents into
Hydrate's fact store, so the model does not have to re-read the whole file
every turn. The file itself is left untouched. See
[`USAGE.md`](USAGE.md#first-run-in-a-project--auto-prime-from-claudemd) for
the opt-in `hydrate dehydrate` flow if you want to also shrink the on-disk
file.

Full install paths (Linux tarball, Windows zip, MCP snippets, manual hooks)
are in [`INSTALL.md`](INSTALL.md).

## Orchestration

**Quality through structured disagreement, not a single confident pass.**
Orchestration replaces "one model, one shot" with a persisted, adversarial
loop. A single model cannot give itself a sceptical second opinion;
orchestration supplies one from a different model family. A generator and an
independent, cross-family critic argue through bounded rounds over a tracked
ledger of objections, and the loop converges only when material defects clear
and the measurable acceptance criteria are met. Fail-closed gates escalate to
a human rather than ship a false pass. The local `hydrate-server` daemon
drives it as a pure, resumable state machine with all state in SQLite, and it
is exposed as MCP tools (`design_*`, `develop_*`), so any connected runtime
can drive it.

| Mode | Artefact | Pipeline | Terminal gate |
|---|---|---|---|
| **Design** | plan or spec | Author (Opus) → Critic (Codex); human arbitrates | sign-off: zero material objections, acceptance met |
| **Develop** | code patches | Implementer → Reviewer → Judge → Audit | human integrate gate, post-merge audit |
| **Image** | generated images | Generator → Judge (design-judgement) | accept |
| **Studio** | UX design, then build | Designer → approve or revise → implement | design approval, build accept |

Why it produces better work:

- **Adversarial, cross-family review.** The critic is prompted to refute
  rather than rubber-stamp, and it is a different model family, so one model's
  blind spots are not shared by its reviewer.
- **Objection ledger with severity and basis.** Every concern is tracked,
  deduped and recurrence-counted. Nothing is silently dropped, and you get an
  auditable record of why an artefact changed.
- **Convergence test with a round cap.** The loop stops at zero material
  objections and diminishing returns. It will not ship with open defects, it
  will not polish forever, and it escalates to `needs_human` when it is
  genuinely stuck.
- **Fail-closed gates.** A gate that cannot render a verdict goes to
  `needs_human`, never to a false green.
- **Pure state machines plus SQLite.** Transitions are deterministic and
  unit-tested, and every run resumes across daemon restarts.

The result is two-sided: better work, because a cross-family critic catches
what the author cannot see, and trustworthy work, because the gates will not
claim the job is done when it is not. You can leave a run going, and it leaves
an auditable trail of every decision. This README's own plan was
pressure-tested through Design mode.

Full walkthrough, with the real bugs it has caught and what they were worth:
[docs/ORCHESTRATION_GUIDE.md](docs/ORCHESTRATION_GUIDE.md).

## Peernet

**Give your agents a way to talk to each other.** Hydrate gives your coding
agents shared memory. Peernet gives them a way to talk to each other.

It is opt-in and local-first. Activated agent sessions find each other over
your own network (LAN, VPN or Tailscale) with no central server, pair with a
short code, and exchange authenticated, audited messages. Nothing leaves your
machines unless you turn it on.

On top of Peernet, the session-addressed relay lets you name a live session
and ask it a question directly. The daemon never answers; the target agent
does, on its next turn, from its own working context:

```sh
hydrate ask <target-project> "what did you mean by 'gate' in the handoff?"
```

Whether that answer is released straight away or held for a person to approve
is set per project by a release policy (auto or human), so disclosure is
governed in one place rather than buried in each runtime's permission prompts.

The one thing that varies by runtime is autonomy: can the named session answer
on its own, or must a human approve the tool call first?

| Runtime | Autonomous answering | How |
|---|:--:|---|
| **Codex** | ✅ | answers peer questions unprompted |
| **Claude** | ✅ | relay tools pre-granted at install |
| **Copilot** | ✅ | per-repo MCP approval written at install |
| **Mistral Vibe** | 🔜 | planned |

> **Status.** Free-text relay answers work between sessions on the same
> machine today. Cross-device answering rides Peernet's v2 peer-onboarding and
> is on the roadmap. The cross-machine peer round-trip itself has been
> demonstrated over Tailscale.

## What you can do

Inside Claude Code, Hydrate ships ten slash commands:

- `/hydrate` recalls relevant context for the current topic.
- `/hydrate-last` resumes the most recent session. The `/hydrate-distill`,
  `/clear`, `/hydrate-last` loop is the headline recovery flow.
- `/hydrate-project` pulls the current project's canon and facts.
- `/hydrate-distill` captures, compresses and writes a handover before
  `/clear`.
- `/hydrate-week` shows the last seven days of dream reports.
- `/hydrate-timeline` gives a chronological view of the project's sessions.
- `/hydrate-dashboard` opens the local dashboard.
- `/hydrate-peernet` activates Peernet for cross-agent messaging.
- `/hydrate-pack-load` loads a shareable project pack (a `.hpack` file).
- `/hydrate-help` prints the reference table.

Codex and Vibe get the same commands as runtime-native skills.

Underneath sits a CLI rich enough that the dashboard's Builders tab can
scaffold most common workflows:

- **`hydrate init`** turns an existing repo into a Hydrate project, with the
  model drafting starter canon.
- **`hydrate dehydrate`** ingests `CLAUDE.md` and other markdown docs into the
  fact store, with an optional rewrite to a summary or stub (reversible). It
  runs automatically in `--mode=full` (no rewrite) the first time you open a
  project.
- **`hydrate canon add`** and **`hydrate fact save`** pin invariants or record
  incidental knowledge.
- **`hydrate pack create`** and **`hydrate pack import`** share a project's
  memory as a single `.hpack` file.
- **`hydrate backup`** and **`hydrate restore`** make an encrypted archive of a
  project's state.
- **`hydrate team push`**, **`pull`** and **`status`** use git as a sync layer
  for shared canon.
- **`hydrate skill-mine`**, **`skill-draft`** and **`skill-install`** detect
  recurring tool-call workflows in your sessions and turn them into Claude
  Code skills.
- **`hydrate narrate`** weaves session JSONLs, git history and project docs
  into a `NARRATIVE.md` (use `--update` for incremental appends).
- **`hydrate orchestrator set`**, **`get`** and **`dispatch`** drive
  multi-pane tmux orchestration with state pinned to canon.

Detailed usage and patterns are in [`USAGE.md`](USAGE.md), or open the
dashboard at `/help` for interactive builders that emit the exact commands for
your settings.

## Benchmarks

We benchmark on the axes that matter for a local memory layer, and we are
specific about what each number measures.

| Benchmark | Result | What it measures |
|---|---|---|
| 🪙 **Token reduction** (headline) | **11.1% fewer output tokens**, 6.3% lower cost, at quality parity | lquery coding A/B, n=10: the same work for fewer tokens |
| 🎯 **Retrieval** | **R@10 = 0.86** (0.95 on multi-session) | LongMemEval, n=500: the right context lands in the top ten |
| 🗜️ **Compression** | **99.2%** (475.7M to 3.7M tokens) | distil pipeline across 22,483 sessions, before re-injection |
| ⚡ **Speed and footprint** | **sub-10 ms** retrieval, no reranker, no GPU, no model download | pure Go, small enough to audit in an afternoon |

Retrieval recall and end-to-end QA accuracy are different measurements, so we
do not place our retrieval number beside another tool's QA-accuracy number.
Full methodology and head-to-head comparisons are at
[gethydrate.dev](https://gethydrate.dev).

## Dashboard

`hydrate-server` exposes a local web dashboard at `http://localhost:<port>/`.
The port is written to `~/.hydrate/server.port` on startup, or run `hydrate
dashboard` to open it. Every pane streams updates live over SSE.

The dashboard has eight pages, three of them with tabbed sub-sections:

- **Home.** A live cockpit: token usage, rollups, active projects and
  orchestrations.
- **Displacement.** Sessions, context-window pressure and model mix over time.
- **Orchestration.** Three tabs: Live (in-flight orchestrations), Build
  orchestration (a form-driven tmux session generator with one window per
  agent, save and load to localStorage, and optional `hydrate orchestrator
  set` lines so the layout is recallable from SQLite), and Customise env (an
  env-var script builder).
- **Fatigue.** Per-project distil and refresh cadence.
- **Facts.** A force-directed graph of every learned fact, filtered by
  project, area, time or pinned state.
- **Skills.** Mined workflow patterns, with Draft and Install buttons that
  turn one into a Claude Code skill.
- **Settings.** Two tabs: Current settings (what is configured) and Customise
  (a script builder for `HYDRATE_*` env vars).
- **Help.** Two tabs: Reference (every CLI command with examples) and Builders
  (eight task builders).

The dashboard is loopback-only by default; remote access requires your API
key, written to `~/.hydrate/server.key`.

## Status

Beta. A free tier is available, the Pro tier is in closed beta, and Enterprise
is available on request. See
[gethydrate.dev/pricing](https://gethydrate.dev/pricing).

## Documentation

- [`INSTALL.md`](INSTALL.md): the full install guide (macOS, Linux, Windows,
  MCP snippets and troubleshooting).
- [`USAGE.md`](USAGE.md): slash commands and the recovery patterns.
- [`SECURITY.md`](SECURITY.md): reporting vulnerabilities.
- [`CONTRIBUTING.md`](CONTRIBUTING.md): how to file issues.
- [`ROADMAP.md`](ROADMAP.md): what is next.
- [`docs/ORCHESTRATION_GUIDE.md`](docs/ORCHESTRATION_GUIDE.md): driving the
  multi-agent orchestration engine, with worked examples.

Homepage, benchmarks and comparisons: [gethydrate.dev](https://gethydrate.dev).

## Get the source

Hydrate's source is proprietary; this repo distributes signed binaries only.
For commercial or source-access enquiries, email `licences@gethydrate.dev`.

## Issues

File bugs at [/issues](https://github.com/getHydrate/hydrate-public/issues).
The fastest route is `hydrate doctor --report`, which pre-fills a GitHub issue
URL with your system info and the diagnostic output.

## Licence

See [`LICENSE`](LICENSE). Binaries are distributed under the Hydrate end-user
licence agreement shipped inside each release tarball.
