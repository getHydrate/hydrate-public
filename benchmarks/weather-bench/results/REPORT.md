# Weather-app YAGNI benchmark — results

**Date:** 2026-06-22 · **n=1 per arm** (anecdotal, like the ponytail video).
**Prompt:** the owner's verbatim natural request (`prompt.txt`) — a feature
description only, **no format/architecture constraints** (no "single file", no
"no frameworks"). So each arm chose its own structure.

## Headline

For a straightforward, well-specified app, **a single Opus shot with a good (YAGNI)
prompt builds a working 12/12 weather dashboard for ~$0.44 in under 2 minutes.**
The full Hydrate orchestration builds a comparable 12/12 app for **$10–11 over
~30 minutes — a ~16–26× cost premium at the same rubric score.** That premium buys
adversarial cross-family review (which caught real concurrency bugs + a
self-contradictory spec the single-shot arms were never checked for), a
dialectically-converged design doc, and verification. **It is worth it for hard,
expensive-to-get-wrong work — not for a weather app.** Use single-shot + a strong
prompt for straightforward tasks; reserve orchestration for genuinely hard ones.

## What v5 isolates: Hydrate is not a single-shot cost lever

v5 (prompt-only, **Hydrate's global hooks ON**, no YAGNI) closes the gap A/B left open
— it isolates *Hydrate's own* effect on a single shot, at full **12/12** quality. The
result is unambiguous: on the single-shot ladder (all Opus 4.8, all 12/12) v5 is the
**most expensive** arm at **$1.24 — 1.6× vanilla A, 3.6× ponytail B** — and it gold-plated
to 736 LOC just like vanilla. Its 615.6k cache-read tokens of injected context bought
**zero** extra rubric points.

Two things follow, and they are not a knock on Hydrate — they're the **memory-gap
principle**: a benchmark only measures a memory layer if the task *forces a gap*. A cold
greenfield "build me a weather app" has no prior decision, no cross-session state, nothing
to recall — so the layer whose value is memory can only add tokens here.

1. **The lever that cuts single-shot cost is concision (YAGNI), not memory.** Ponytail and
   v4 win ~57% on cost and ~65% on LOC from a YAGNI constraint alone. That's the
   repeatable signal.
2. **Don't pitch Hydrate as a one-shot cost-reducer — the data says the opposite.** Pitch
   concision for one-shots; pitch Hydrate for the cross-session / cross-runtime memory case
   where a one-shot can't compete because it has no memory at all (LongMemEval, c8b,
   99.2% distill compression). (All arms here are **n=1** — v5's number is directionally
   clear but needs 3× before it's load-bearing in anything public.)

## Recommendation: triage before orchestrating

The actionable takeaway: **orchestration should be opt-in by warrant, not a default.**
Add a sanity-check gate up front — *is this task hard enough to justify the ~16–26×
premium?* If not, skip multi-agent orchestration and instead run a **single-shot
prompt with the YAGNI/correctness principles injected** (the v4 path: $0.44, 2 min,
12/12).

- **Single-shot + injected principles** — well-specified, known-pattern, small,
  cheaply-verifiable, cheap-to-fix-if-wrong work. *This is the common case.*
- **Orchestration** — only when correctness is subtle (concurrency, security,
  protocol, migrations), the design space is wide and a converged spec de-risks the
  build, scope is large/multi-unit, or a latent bug costs far more than ~$10.

Shipped this session: a **Phase 0 triage gate** in the `/hydrate-orchestrate` skill
that makes this call before any fleet spins up, and hands the user the single-shot
path when orchestration isn't warranted.

## Full cost ladder (all measured)

| Run | Mechanism | Cost | LOC | Files | Rubric | Time (API) |
|-----|-----------|-----:|----:|------:|:------:|-----------:|
| **B** ponytail single-shot | YAGNI plugin (headless, Hydrate off) | **$0.34** | 253 | 1 | 12/12 | ~1m |
| **v4h** headless + YAGNI + **Hydrate ON** | YAGNI directive | **$0.34** | 285 | 1 | 12/12 | 1m22s |
| **v6** `/hydrate-yagni` **skill** + **Hydrate ON** | shipped skill (TUI) | **$0.43** | **217** | 1 | 12/12 | 1m08s |
| **v4** Opus + YAGNI prompt | single-shot (TUI) | **$0.44** | ~297 | 1 | 12/12 | 1m44s |
| **A** vanilla single-shot | headless, no YAGNI, Hydrate off | **$0.78** | 807 | 3 | 12/12 | ~1.5m |
| **A-TUI** vanilla in the TUI | **TUI**, no YAGNI, Hydrate off | **$0.72** | 844 | 1 | 12/12 | 2m54s |
| **v5** prompt-only + Hydrate | single-shot (TUI), no YAGNI | **$1.24** | 736 | 1 | 12/12 | 3m57s |
| **v5h** headless plain + **Hydrate ON** | no YAGNI | **$1.53** | 740 | 1 | 12/12 | 6m06s |
| **C-3** develop-only | orchestration | **$7.17** | 506 | 1 | 12/12 | 11m33s |
| **C-2** design+develop (owner) | orchestration | **$10.48** | 841 | 1 | 12/12 | 14m36s |
| **C-1** design+develop (agent) | orchestration | **$11.40** | 396 | 1 | 12/12 | 14m19s |

Orchestration totals = driver `/cost` + headless worker `$` (captured per-role/model
by `hydrate orch usage`). Decomposition: **develop-only $7.17** vs **full $10.48** ⇒
the **design phase adds ~$3.3 (≈ +46%)**, almost all on the driver side. The driver
cost itself is **dominated by poll-loop overhead** (9–14M cache-read tokens across
30+ status polls against the timeout-then-poll round UX) — a fixable UX artifact,
not intrinsic compute.

The Hydrate instrumentation that makes the orchestration arms measurable
(`orch_worker_usage` table, `hydrate orch usage` CLI, `HYDRATE_ORCH_YAGNI` toggle)
shipped this session (hydrate `01105748`, merged to local main).

**v6 — the shipped skill reproduces the lean tier.** Every prior YAGNI arm used a
hand-pasted directive (v4), a headless flag (v4h), or a plugin (ponytail). **v6 is
the first run of the user-invokable `/hydrate-yagni` skill** — the read-path delivery
of the `lean` tier. It lands at **$0.43 / 217 LOC / 12/12**, squarely in the YAGNI
cluster ($0.34–0.44) and the **lowest LOC of the entire bench**. The concision shows
up behaviourally, not just on cost: the model deleted its own dead reverse-geocode
call mid-run (Open-Meteo has no lat/lon lookup → it would have silently failed) and
itemised what it declined to build. The ~$0.09 over the $0.34 headless floor is the
interactive-TUI full-env premium (skill-load + MCP present), identical to v4's $0.44 —
the skill delivers the lean *prompt half*, not the stripped environment.

**Model:** every single-shot arm (A, B, v4, v5, v6) ran on **Opus 4.8 (`claude-opus-4-8[1m]`)**.
A and B are confirmed from their `result.json` `modelUsage` (A also used a little Haiku for
a sub-task; neither pinned `--model` — the clean config defaulted to Opus). v4 is from its
run note and v5 from the run's `/cost` (`modelUsage` showed `claude-opus-4-8`). So the
single-shot ladder is apples-to-apples on model; the only variables are the prompt/plugin
and whether Hydrate's hooks were active.

## Clean isolation: the headless on/off runs (v4h, v5h)

The original v5 (TUI + Hydrate) confounded *four* differences vs the headless A baseline:
harness (TUI vs headless), the `frontend-design` skill v5 loaded, a Playwright verification
loop, and Hydrate. To isolate Hydrate properly, **v5h** and **v4h** re-run the same prompts
**headless** (same harness as A/B), same Opus 4.8, with Hydrate's hooks + MCP **ON**. Both
score **12/12** (source + live Playwright; v5h has a London geolocation fallback, v4h passes
criterion 12 via fluid layout not `@media`).

| Arm | Harness | Hydrate | Prompt | Cost | LOC | Turns |
|-----|---------|:-------:|--------|-----:|----:|------:|
| **A**     | headless | OFF | plain          | $0.78 | 807 | 11 |
| **A-TUI** | **TUI**  | OFF | plain          | $0.72 | 844 | — |
| **v5h**   | headless | **ON** | plain        | **$1.53** | 740 | 22 |
| **v5**    | **TUI**  | **ON** | plain        | $1.24 | 736 | — |
| **v4h**   | headless | **ON** | **+ YAGNI**  | **$0.34** | 285 | 3 |
| **B**     | headless | OFF | ponytail (YAGNI) | $0.34 | 253 | 4 |

Three deltas fall out — one clean, one null, one confounded (read the mechanism note):

0. **Harness is cost-neutral (null result) — A → A-TUI: $0.78 → $0.72.** Same prompt, model,
   and Hydrate-off; only headless vs the interactive TUI differs. Cost is **statistically
   identical** (−8%, n=1 noise; A-TUI even gold-plated harder, 844 LOC + two self-correcting
   bug-fix passes). So the **TUI is not the expensive part** — this kills the confound I
   suspected in the original v5, and is corroborated by headless v5h ($1.53) *exceeding* TUI
   v5 ($1.24).

1. **YAGNI dominates (clean) — v5h → v4h: $1.53 → $0.34 (−78%), Hydrate held ON both runs.**
   The *only* change is the YAGNI directive in the prompt. It cut cost 78%, turns 22 → 3,
   LOC 740 → 285, landing exactly on ponytail's $0.34. This is the robust, repeatable lever
   (mirrors A→B −57%).
2. **Hydrate-on vs off (confounded by turn count) — A → v5h: $0.78 → $1.53 (+96%).** Both
   plain/no-YAGNI; Hydrate is the nominal variable. **But the mechanism is NOT canon
   injection:** the `prompt_submit` hook injected only **~1.3 KB** (v4h logged 1325 chars;
   v5h negligible) — Hydrate correctly stayed near-silent on a project with no memory to
   recall. The +96% is driven almost entirely by **v5h running 2× the turns (22 vs 11)**,
   which re-reads the growing context each turn (941k cache-read) and includes the `hydrate`
   MCP tool-definition surface loaded per turn. At **n=1**, the turn doubling is not cleanly
   attributable to Hydrate vs run-to-run variance vs the MCP toolset tempting tool calls.
   **Do not read this as "Hydrate doubles cost"** — read it as "Hydrate's *injection* is
   already gap-aware (tiny here); the open question is whether its MCP surface / behavioural
   nudges add turns on cold one-shots." Needs a 3× re-run to separate signal from variance.

**Read:** on greenfield one-shots, **YAGNI/concision is the proven cost lever**. Hydrate's
read-path injection is correctly minimal when there's nothing to recall (the memory-gap
principle working as intended); its only measured one-shot tax is the per-turn MCP
tool-definition surface plus an unexplained (n=1) turn-count swing. All arms **n=1**.

### ⚠ The "Hydrate on/off" comparisons are confounded by the whole plugin suite

The `.clean-claude-config` "Hydrate off" baseline strips **every enabled plugin**, not just
Hydrate — the normal config carries `frontend-design`, `playwright`, etc.; the clean config
enables none. So **"Hydrate off vs on" actually means "stripped vs full plugin environment."**
This shows up sharply in the two TUI runs (same harness, same plain prompt, Opus):

| Arm | Harness | Config | frontend-design? | Playwright? | Cost |
|-----|---------|--------|:---:|:---:|-----:|
| **A-TUI**         | TUI | clean (Hydrate off) | no  | no  | $0.72 |
| **a-tui-hydrate** | TUI | normal (Hydrate on) | **yes** | **yes** | **$1.67** |

The $0.72 → $1.67 jump is **dominated by `frontend-design` (the atmospheric "living-sky /
Fraunces" gold-plating) + a Playwright verification loop**, plus — in `a-tui-hydrate` — an
*environmental* stale-`http.server` port collision the run had to debug (kill + re-serve).
Hydrate's own injection was **~1.3 KB**. So **`a-tui-hydrate` is neither representative nor a
Hydrate isolation**, and the expensive plain runs (v5 $1.24, v5h $1.53, a-tui-hydrate $1.67)
all share the same real driver: *the model pulled in the design skill and browser-verified*,
not Hydrate.

**What this invalidates:** treating any clean-vs-normal delta as "Hydrate's cost." **What
survives clean:** (a) **YAGNI −78%** (v4h vs v5h) and (b) **harness-neutral** (A vs A-TUI).
The clean Hydrate isolation itself is now done — see below.

## ⚠ The "MCP isolation" was confounded by Playwright — corrected via transcripts

An n=3 attempt ran the default config (Keychain auth; `CLAUDE_CONFIG_DIR` overrides all fail
auth headless), Opus, YAGNI prompt in both arms, hooks in both, and tried to isolate the
82-tool `hydrate` MCP by dropping it in the OFF arm with `--strict-mcp-config --mcp-config
{empty}`. The raw numbers looked dramatic:

| Arm (n=3) | Mean cost | Mean turns |
|-----------|----------:|-----------:|
| MCP ON  (default) | $1.47 | 24 |
| MCP OFF (stripped) | $0.67 | 5 |

**But reading the transcripts kills the interpretation.** `--strict-mcp-config` removes
**every** MCP server, not just hydrate — including **`playwright`**. Per-run tool tallies:
`hydrate_calls = 0` in all three ON runs; **`playwright_calls = 10–13`**. So the OFF arm
couldn't browser-verify (5 turns = build + done) while the ON arm ran a full Playwright
verification loop (24 turns). **The −54% / 5× is Playwright, not hydrate.**

**What this actually establishes:**
- **The hydrate MCP's 82 tools were never called** (0/24 calls across 3 runs) — so hydrate
  added **no behavioural turn-inflation** on this build.
- Hydrate's only single-shot cost is therefore the **passive tool-definition presence**:
  ~22 k tokens of schemas in context every turn (cache-read), defined-but-unused. Real, but
  modest, and dwarfed by the Playwright/`frontend-design` behaviour.
- The genuine cost drivers across this whole bench were **Playwright (verification loops)**
  and **`frontend-design` (gold-plating)** — both ordinary plugins, never hydrate.

**Defining an MCP server ≠ more turns.** It adds a fixed per-turn *token* cost (its schemas
in context); it only adds *turns* if the model **uses** the tools. Hydrate (defined, unused)
→ ~0 turns; Playwright (defined, used) → +19 turns.

**Improvements that survive the correction:** (1) trim / lazy-load the hydrate MCP surface to
cut the ~22 k-token passive tax (most sessions touch 0–3 of 82 tools); (2) to actually
measure hydrate's marginal cost, redo the isolation keeping Playwright in **both** arms and
removing **only** hydrate (`--mcp-config` with a playwright-only server set) — expectation,
given 0 calls here, is a small passive-token delta, not a behavioural one.

## Status

| Arm | What | Status |
|-----|------|--------|
| **A** vanilla single-shot | baseline (headless, no YAGNI) | ✅ measured — $0.78 |
| **A-TUI** vanilla in the TUI | harness control (TUI, Hydrate off) | ✅ measured — $0.72, 12/12 |
| **B** ponytail single-shot | YAGNI plugin | ✅ measured — $0.34 |
| **v4** Opus + YAGNI prompt | single-shot, manual YAGNI | ✅ measured — $0.44 |
| **v5** prompt-only + Hydrate hooks | single-shot (TUI), Hydrate ON, no YAGNI | ✅ measured — $1.24, 12/12 |
| **v5h** headless plain + Hydrate ON | clean isolation vs A (Hydrate on/off) | ✅ measured — $1.53, 12/12 |
| **v4h** headless + YAGNI + Hydrate ON | clean isolation vs v5h (YAGNI on/off) | ✅ measured — $0.34, 12/12 |
| **C-1/C-2** design+develop | full orchestration (×2 runs) | ✅ measured — $11.40 / $10.48 |
| **C-3** develop-only | orchestration, no design phase | ✅ measured — $7.17 |
| **D** orchestration, YAGNI **off** | design→develop | ⏳ optional — `RUNBOOK-CD.md` |

## Measured arms — single-shot (A vs B)

Both built a **functionally complete, visually polished** weather app and score
**12/12** on the rubric. Verified by Playwright load (no JS errors beyond a benign
favicon 404) + full-page screenshots (`results/arm-{a,b}-screenshot.png`).

| Metric | A — vanilla | B — ponytail | B vs A |
|--------|------------:|-------------:|-------:|
| Rubric score | 12 / 12 | 12 / 12 | parity |
| **Cost (USD, vendor-reported)** | **$0.782** | **$0.339** | **−56.6%** |
| Output tokens | 17,462 | 6,827 | −60.9% |
| Input tokens | 3,311 | 5,693 | +71.9% |
| Cache read | 339,325 | 92,034 | −72.9% |
| **Source LOC (non-blank, all files)** | **807** | **253** | **−68.6%** |
| Source bytes | 29,346 | 10,602 | −63.9% |
| **Files produced** | **3** (html+css+js) | **1** (html) | — |
| Agent turns | 11 | 4 | −64% |
| Wall-clock | 188 s | 74 s | −60.6% |

### Read

- **Same requested quality (12/12), far less of everything else.** Ponytail hit
  the spec in **69% less code**, **57% lower cost**, **61% fewer output tokens**,
  in a third of the turns.
- **Unconstrained, vanilla CC gold-plated.** With no format constraint it built a
  **3-file** app (`index.html` + `styles.css` + `app.js`, 807 LOC) and added
  features nobody asked for — precipitation chance, temperature-range bar
  visualizations, "Skycast" branding — over **11 turns**. Ponytail produced a
  single 253-LOC `index.html` covering exactly the requested feature set in 4
  turns. The extra vanilla work scored **zero** additional rubric points.
- **This is the cleaner, more representative result.** An earlier run used an
  over-specified prompt (it pinned "single self-contained index.html, no
  frameworks") — that flattered the baseline by suppressing exactly the sprawl
  YAGNI is meant to prevent. Removing it widened the gap (LOC −68.6% vs −73.0%
  before is similar, but the architecture divergence — 3 files vs 1, 11 turns vs
  4 — only shows up on the natural prompt, and is the real-world behaviour).
- Input tokens are higher for ponytail (its always-on YAGNI system prompt + hooks,
  ~900 tok) but dwarfed by the output savings. Net strongly positive.
- **Baseline is genuinely vanilla.** Both arms ran with Hydrate's global hooks
  disabled via an isolated `CLAUDE_CONFIG_DIR` (Keychain cred seeded, no
  `settings.json`); verified no `HYDRATE-wiki` / context injection. A = clean CC,
  B = clean CC + ponytail 4.7.0. Only variable = the plugin.

## Arm C — Hydrate orchestration (YAGNI on)

Full design → develop orchestration (codex critic; Opus implementer / reviewer /
judge / audit). Run **twice, independently** — once driven by the benchmark agent
(C-1) and once by the project owner with his own prompt and process (C-2) — to
separate the cost of *how it's driven* from the cost of *the orchestration itself*.
Both converged design in 3 rounds and develop in 3 rounds; both verified 12/12 on
the rubric live (mocked Open-Meteo, desktop + mobile); both produced a single
self-contained `index.html`.

### Cost — two independent runs

| Component (role · model · src) | C-1 (agent-driven) | C-2 (owner-driven) |
|---|--:|--:|
| **Driver session** (design author + arbiter + verification, opus) | **$9.14** | **$7.16** |
| Design critic · codex | $0.2493 (3) | $0.3019 (3) |
| Develop implement · opus | $1.4678 (3) | $2.3554 (3) |
| Develop review · codex | $0.4661 (3) | $0.5588 (3) |
| Develop verify · codex | $0.0728 (1) | $0.0994 (1) |
| Worker subtotal | $2.256 | $3.315 |
| **TOTAL** | **≈ $11.40** | **≈ $10.48** |
| Output | 396 LOC / 15.8 KB | **841 LOC / 33.0 KB** |
| Driver tokens (opus) | 21.4k in / 48.6k out / **13.5M cache-read** | 20.3k in / 52.3k out / **9.4M cache-read** |
| Driver API duration | 14m19s | 14m36s |

Worker token/$ captured by `hydrate orch usage` (codex $ computed from the
GPT-5-codex rate table; opus $ vendor-reported). Driver $ from each session's
`/cost`. Wall time is meaningless (includes idle); API duration is the real
compute, ~14.5m in both.

### Read

- **Full orchestration costs ≈ $10–11, near-invariant to who drives.** The two
  totals ($11.40, $10.48) land within ~9% of each other despite different drivers,
  prompts, and a 2.1× output-size difference. So the cost is **structural**, not an
  artifact of one driver's style.
- **vs single-shot: ~15–30× the cost** ($10–11 vs A $0.78 / B $0.34) for the **same
  rubric (12/12)**. Per the plan, this is a **structure-cost data point, not a win
  claim** — orchestration is multi-agent + multi-round and was never expected to be
  cheaper than one shot.
- **The 12/12 parity understates what orchestration bought.** Across the two runs
  the adversarial loop caught real defects the single-shot arms were never checked
  for: a self-contradictory unit-data contract (design critic, both runs), and in
  develop, concurrency races (Enter-select, concurrent-load, stale-data-on-toggle)
  and a same-named-city label collapse. The rubric can't score "race-free /
  internally consistent," so the quality delta is invisible to it. That
  verification *is* the product.
- **Output size tracks design thoroughness, not the YAGNI flag.** Both runs were
  YAGNI-on, yet C-2 (841 LOC) is 2.1× C-1 (396 LOC) — because C-2's design was
  richer (134-line spec, 5 acceptance criteria vs 93 lines). The lever bites the
  **design author's** decisions; a detailed YAGNI-on design still yields a large
  app. (Caveat for any C-vs-A LOC comparison: both C runs chose single-file, so
  LOC differences vs the multi-file vanilla arm mix architecture with verbosity.)
- **~$7–9 of each driver cost is poll-loop overhead, not reasoning.** Both drivers
  burned huge cache-read volumes (13.5M / 9.4M tokens) across 30+ status polls
  against the **timeout-then-poll** round UX — the MCP `design_round`/develop calls
  return an HTTP timeout while the worker runs daemon-side, forcing a sleep/poll
  loop. A clean async round (dispatch → single poll, or streamed progress) would
  cut the dominant cost term sharply. **This is the single biggest fixable lever in
  orchestration economics**, and it's a UX artifact, not intrinsic.

### Reproduced engine issues (both runs)
- **`audit_verifier_unavailable`** — the post-integration Opus audit verifier
  couldn't render a verdict and left **no resumable row**, so neither
  `develop_audit` nor `develop_integrate` can re-drive it. Both runs ended in
  `needs_human`; the verified integration branch was hand-merged to main and the
  audit done manually. Two-for-two → a reproducible engine bug, not a fluke.
- **`hydrate server restart` is unsafe under launchd** — it can spawn a second
  daemon beside the launch-agent one (two writers → one SQLite DB → orchestration
  wedges). Cycle via `launchctl kickstart -k` instead.
- **`develop_plan` phase schema** — strict, blind-retry field errors
  (`unit_id`/`id`/`goal`/`change`) cost several round-trips.

The clean YAGNI-lever comparison is still **C vs D** (both orchestration, YAGNI
off), pending. A **develop-only** variant (skip design) is also queued, to isolate
the design phase's cost contribution.

## Honest framing

- **n=1** on **one app** — directional, not statistically robust. (First/second
  ponytail runs were $0.232 / $0.339 — real run-to-run variance at n=1.)
- Arms C/D (orchestration) will cost **more total tokens/$** than A/B (multi-agent,
  multi-round, Opus-heavy). Orchestration's value is **verified correctness + a
  converged design + YAGNI-lean output**, NOT fewer total tokens. C/D will not be
  framed as "cheaper than single-shot".
- The two defensible headline comparisons: **(1) ponytail on/off** single-shot
  (this report ✅) and **(2) Hydrate YAGNI on/off** within orchestration (C/D,
  pending). "Orchestration vs single-shot" is a structure-cost data point.

## How the numbers were captured

- A/B: `claude -p "$(cat prompt.txt)" --output-format json` → `total_cost_usd` +
  `usage{…}` from the result JSON (`run-singleshot-arm.sh` / `run-arm-b.sh`).
  LOC/bytes summed over all produced web-source files (`extract-metrics.sh`).
- C/D (pending): per-role/model token+$ from `hydrate orch usage --session <id>`
  for both the design and develop sessions, plus the driver `/cost` for the
  design author/arbiter. See `RUNBOOK-CD.md`.

## Engine findings & fixes (surfaced by these runs)

The orchestration runs doubled as a live stress test. Real bugs found:

1. **`audit_verifier_unavailable` wedge — reproduced 3×.** The post-integration
   audit verifier leaves **no resumable row**, so neither `develop_audit` nor
   `develop_integrate` can re-drive it; the session sticks in `needs_human` and the
   verified branch must be hand-merged. Two distinct causes seen: capacity, and (v3)
   the codex audit **pinning `gpt-5-codex`, unsupported on a ChatGPT-auth account**
   (`400`) while the working review/judge used the default model — a fixable
   model-pin bug.
2. **`hydrate server restart` spawns a second daemon under launchd** → two writers
   on one SQLite DB (691 MB stuck WAL) → orchestration wedges. Cycle via
   `launchctl kickstart -k` instead. (This caused the first C-run "hang.")
3. **Driver cost dominated by the timeout-then-poll round UX** (9–14M cache-read
   tokens/run). The round call blocks server-side and times out, forcing a
   sleep/poll loop on an Opus session.
4. **Codex critic fired ~7×/build** (design critic + dev review + verify, every
   round) — the §9 role-resolution also silently overrode the plist's `claude`
   reviewer to codex. Real quota burn.
5. **`develop_plan` phase schema** — strict, blind-retry field errors.

**Fixes shipped this session** (branch `feat/orch-async-wait`, not yet installed to
the live daemon):
- **Cross-family-at-the-gate critic cadence** (addresses #4) — `HYDRATE_DESIGN_CRITIC_CADENCE=gate`
  (default): iteration rounds run the cheap same-family Claude critic; codex fires
  only on an explicit `design_round gate=true`. Cuts codex from ~7 calls to ~1–2.
  Built + tested; `/hydrate-orchestrate` skill updated to drive it.
- **Brief status** (`?brief=1`) (partial fix for #3) — compact poll payload.
- Designed/scaffolded: a long-poll `*_wait` (kills the sleep/poll loop) and async
  design-round dispatch (#3) — not yet wired.

## How the numbers were captured

- Single-shot (A / B / v4): `claude -p` (A/B via clean `CLAUDE_CONFIG_DIR` so no
  Hydrate hooks fire; v4 from a normal session) → `total_cost_usd` + `usage{…}`.
  LOC/bytes from the produced source (`extract-metrics.sh`).
- Orchestration (C-1/2/3): per-role/model token+$ from `hydrate orch usage --session
  <id>` (codex $ computed from the GPT-5-codex rate table, opus/claude $
  vendor-reported) for the design + develop sessions, plus the driver `/cost`.

## Artifacts
- `arm-a-vanilla/`, `arm-b-ponytail/`, `v5-prompt-only-hydrate/`, `arm-c-hydrate-yagni-on{,_v2,_v3}/` — built apps.
- `results/arm-{a,b}-screenshot.png` — live renders; `results/metrics.json` — all metrics.
- `prompt.txt`, `rubric.md`, `RUNBOOK-CD.md`.
