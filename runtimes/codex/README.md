# Codex + Hydrate: what your Codex sessions gain when they remember

> **Status: scaffold — results pending a run.** This page has the method, the
> task, the rubric and the protocol locked. The numbers in the tables are marked
> `pending` and must not be cited until the run lands. We publish measured
> figures only.

[OpenAI Codex](https://developers.openai.com/codex) is one of the runtimes
Hydrate plugs into. This page does not pit Hydrate against Codex — Hydrate runs
*inside* Codex, as session-start and capture hooks plus an MCP server. The
question it answers is the only honest one:

> **What does a Codex session gain when Hydrate gives it a memory?**

Two things, measured two ways.

## The two benchmarks

| | What it proves | How |
|---|----------------|-----|
| **A. The directive ports** | The [`/hydrate-yagni`](../../slash-commands/README.md) concision lever works on Codex's own model, not just Claude | Same weather task, single shot, **with vs without** the directive — cost, LOC, 12/12 quality |
| **B. Only Hydrate gives Codex a memory** | Codex picks up where another runtime left off, and reuses what the last session built | Cross-runtime hand-off + cross-session reuse + compact-survival |

Benchmark A is the portable lever. Benchmark B is the part no single-vendor tool
can do, and it is where Codex gains the most.

## Benchmark A — does the YAGNI directive port to Codex?

Same task as [weather-bench](../../slash-commands/benchmarks/weather-bench/README.md), scored on the same
[12-point rubric](rubric.md). One fixed prompt, built with and without the
directive, on Codex's current default model. Because both arms are held to 12/12,
any cost or line-count gap is concision, not quality. Lower is better.

| Arm | Directive? | Cost | LOC | Files | Score |
|-----|:----------:|-----:|----:|:-----:|:-----:|
| codex-vanilla | no | _pending_ | _pending_ | _pending_ | _pending_ |
| codex-yagni `/hydrate-yagni` | yes | _pending_ | _pending_ | _pending_ | _pending_ |

What this arm answers: the slash-command directives are runtime-portable. On
Claude Code the directive cut a build ~78% in cost at the same 12/12 quality
([weather-bench](../../slash-commands/benchmarks/weather-bench/README.md)). This arm checks whether the same
discipline holds on Codex.

## Benchmark B — cross-runtime memory (the headline)

This is the Codex story. Hydrate's store is shared across runtimes, so a session
in one agent is visible to the next agent — even a different vendor's.

### B1. Cross-runtime hand-off

> **Start the weather feature in Claude Code → `/clear` → resume it in Codex.**
> Codex's session-start hook injects what Claude already did, and Codex continues
> the build without being re-briefed.

No single-vendor memory can tell this story, because it requires the two runtimes
to share one memory layer. The proven Claude↔Codex result on Hydrate's
cross-runtime compact-survival bench is a 1.00 / 0.90 recall pair; this page
re-runs it on current infra and shows the weather build crossing the boundary.

| Hand-off | Recall | Continued without re-brief? |
|----------|:------:|:---------------------------:|
| Claude Code → Codex | _pending_ | _pending_ |

A capture gate runs first: `hydrate resume --project=<slug>` must show the Claude
session before the Codex reader starts, so a slug-split false-empty can never be
misread as a real recall.

### B2. Compact-survival

Codex has no PreCompact event of its own — so for a long Codex session, Hydrate
*is* the compact memory. Build the app, force a context compaction, confirm the
build state survives and the session keeps its thread.

| Event | Survives compaction? |
|-------|:--------------------:|
| Codex session state | _pending_ |

### B3. Cross-session reuse

Two Codex sessions on the same project: session 1 builds, session 2 extends. A
no-memory arm rebuilds the component; a Hydrate arm reuses it. Higher reuse and
lower new-LOC is better.

| Arm | Memory? | Reuse rate | Avg new LOC |
|-----|:-------:|:----------:|:-----------:|
| codex-cold | no | _pending_ | _pending_ |
| codex-hydrate | yes | _pending_ | _pending_ |

## How to run this (for whoever fills the tables)

1. **Re-baseline on the latest Codex CLI** and its current default model. Do not
   reuse the Claude Opus weather-bench numbers; label the exact Codex version and
   model in `results/metrics.json`.
2. **Headless flag check.** Codex headless needs
   `--dangerously-bypass-hook-trust` plus the sandbox flags on 0.139+. The older
   0.130 invocation silently degrades to baseline — hooks never fire and the run
   looks like "Hydrate adds nothing". Verify the hooks actually fired before
   trusting any arm.
3. **Verify capture before recall.** Export `HYDRATE_PROJECT`, then gate every
   hand-off and reuse cell on `hydrate resume --project=<slug>` showing the
   writer's session first.
4. Score every build on the [rubric](rubric.md); a cost/LOC delta only counts at
   12/12 parity.

## Honest caveats

- **Nothing here is measured yet.** Every figure is `pending`. This page is the
  method and the protocol, not a result.
- **Different model from weather-bench.** Codex runs its own model; cross-page
  comparisons are about the *uplift on each runtime*, not absolute cost between
  runtimes.
- **One task.** Like weather-bench, this is a single greenfield app for benchmark
  A. The memory benchmarks (B) are the ones that actually exercise Codex's gain
  from Hydrate; benchmark A is the portability check.

## Files here

- [`prompt.txt`](prompt.txt): the verbatim task (identical to weather-bench).
- [`rubric.md`](rubric.md): the 12-point checklist.
- [`results/metrics.json`](results/metrics.json): the results scaffold to fill in.
