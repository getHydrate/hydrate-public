# weather-bench: what the YAGNI directive actually saves

This is the benchmark behind the [`/hydrate-yagni`](../../README.md) slash
command. It answers one question: for a normal, well-specified build, how much
does a concision directive save, and at what cost to quality?

The method is deliberately simple. One fixed task is built many different ways.
Every build is scored against the same 12-point functional checklist. Because
every build scored 12/12, the differences in cost and output size are not
quality differences. They are the price of how the task was run.

## Credit and prior art

The task and the core idea come from Better Stack's video
[*This Claude Code Plugin Writes 94% Less Code (ponytail)*](https://www.youtube.com/watch?v=2xuFcmUAQUc).
It demonstrates ponytail, a Claude Code plugin that enforces a concision
discipline so the model writes far less code for the same result. We took the
weather-app prompt from that video, and the idea of pulling the same discipline
out of Hydrate's orchestrator into a single installable directive.

Ponytail appears in the results below as arm B, run head-to-head against our
directive. (The "94% less code" in the video title is Better Stack's figure
against their own baseline. The number we measured for ponytail here, against the
vanilla arm in this benchmark, is 57% less cost and 69% fewer lines of code.)

## The task

Every arm received this exact prompt, verbatim. It is a feature description only,
with no format or architecture constraints (it does not say "single file" or
"no frameworks"), so each run chose its own structure. That is the point: an
unconstrained prompt is where a model's tendency to over-build shows up.

> Build me a weather dashboard app. It should automatically detect the user's
> location and show current weather conditions including temperature, wind speed,
> humidity, and a general weather description. Display an hourly forecast for the
> rest of the day and a 7-day daily forecast. The current conditions should have
> an appropriate weather icon or visual indicator based on the conditions.
> Include a search box so the user can look up weather for any city by name.
> Temperatures should be toggleable between Celsius and Fahrenheit. The design
> should feel like a real weather app - clean, visual, and easy to scan at a
> glance. Use the Open-Meteo API for weather data and the Open-Meteo geocoding
> API for city search. Both are free and require no API key.

## The rubric

Twelve criteria, one point each, scored from loading the built page and reading
the source. Full text in [`rubric.md`](rubric.md). In brief: geolocation on load,
current temperature, wind, humidity and description, an hourly forecast, a 7-day
forecast, a condition icon, city search, a Celsius/Fahrenheit toggle, Open-Meteo
with no API key, and a clean responsive layout.

A quality-parity guardrail applies: a cost or size delta only counts as "leaner at
the same quality" if both arms actually scored the same. They did. Every arm here
is 12/12.

## Results

Every run in this benchmark used Claude Opus 4.8, single shots and orchestration
alike. Holding the model fixed is the point. The only things that varied were the
prompt (with or without the YAGNI directive), the environment (which plugins were
loaded), and how the task was run (a single shot versus a multi-agent
orchestration). Lower is better for cost and lines of code (LOC).

Note that arm B is our own Opus run of Ponytail in this harness, not Ponytail's
published figures. Ponytail's own benchmarks primarily use Claude Haiku 4.5, so its
headline numbers are not directly comparable to ours.

| Arm | How it was run | YAGNI? | Cost | LOC | Files | Score |
|-----|----------------|:------:|-----:|----:|:-----:|:-----:|
| **B** ponytail (Better Stack plugin) | single shot, headless, concision plugin | yes | **$0.34** | 253 | 1 | 12/12 |
| **v4h** | single shot, headless, YAGNI directive | yes | **$0.34** | 285 | 1 | 12/12 |
| **v6** `/hydrate-yagni` | single shot, **interactive**, the shipped command | yes | **$0.43** | **217** | 1 | 12/12 |
| **v4** | single shot, interactive, pasted YAGNI directive | yes | **$0.44** | 297 | 1 | 12/12 |
| **A-TUI** | single shot, **interactive**, no directive | no | $0.72 | 844 | 1 | 12/12 |
| **A** vanilla | single shot, headless, no directive | no | $0.78 | 807 | 3 | 12/12 |
| **v5** | single shot, interactive, no directive | no | $1.24 | 736 | 1 | 12/12 |
| **v5h** | single shot, headless, no directive | no | $1.53 | 740 | 1 | 12/12 |

Per-arm token, byte and timing detail is in [`results/metrics.json`](results/metrics.json).
Representative screenshots are in [`results/`](results/).

Two groups, and what each shows:

- **YAGNI single shots ($0.34 to $0.44, ~217 to 297 LOC).** A directive that says
  "build only what the task needs" produces a complete 12/12 app in about a third of
  the code. Ponytail (arm B, the Better Stack plugin) landed in this band too on this
  task. We are not claiming a directive equals the plugin in general. This is n=1 on
  one greenfield app, and Ponytail is an always-on skill that enforces its
  decision-ladder every turn, where a pasted directive is per-prompt and can drift.
  See [Relation to Ponytail](#relation-to-ponytail) for the honest version.
- **Plain single shots ($0.72 to $1.53, ~736 to 844 LOC).** With no concision
  directive the model gold-plates. The vanilla run added precipitation charts,
  temperature-range bars and its own "Skycast" branding that nobody asked for,
  and split one page into three files. None of it scored a single extra point.

Above both sits a third, much heavier tier these slash commands are not: Hydrate's
full design-and-develop orchestration, which built the same app for roughly $7 to
$11. It is not in the table because this benchmark is about the single-shot
concision lever, and because orchestration costs more tokens by definition. It runs
several agents (design author, critic, implementer, reviewer, verifier) across
multiple rounds, and carries the coordination overhead of managing them (re-reading
shared context each round, status polling, the driver loop). That is structural, not
waste. Orchestration buys adversarial cross-family review that catches defects a
single shot is never checked for, such as concurrency races or a self-contradictory
data contract, so it earns its cost on hard, expensive-to-get-wrong work, not on a
weather app. See *When to use which* in the
[slash-commands README](../../README.md).

## Relation to Ponytail

[Ponytail](https://github.com/DietrichGebert/ponytail) is a purpose-built
concision skill for AI coding agents, an always-on decision-ladder that fires
before each generation. Its creator has benchmarked it thoroughly (multiple tasks,
10 to 30 runs, several models, safety-scored, reproducible). This weather-app
benchmark is a single greenfield task (n=1). To compare properly, Hydrate has since
run ponytail's own agentic benchmark end to end, on its harness and its scorers, and
the full result lives in [BENCHMARKS.md](../../BENCHMARKS.md). The short version:

- **Ponytail's safety finding is reproduced exactly.** Running its suite, a naive
  "YAGNI plus write one-liners" prompt drops to 94.4% safe, and the failures land
  on the exact same task and model ponytail published. Hydrate's guarded directive
  stays at 100%. It forbids dropping any guard or error check to shorten code, and
  mandates building seams the brief requires. Ours is a concision-with-correctness
  contract, not "write less".
- **Backend code is a tie; ponytail wins frontend line count.** On backend tasks
  ponytail and Hydrate's directive land at the same median, about 6% under baseline.
  On frontend tasks ponytail reduces line count much more aggressively. That is a
  deliberate trade: Hydrate holds higher-fidelity UI (design is its own step) rather
  than golf the components away, and a test that chased the frontend number made
  the build worse, so it was reverted. Details in BENCHMARKS.md.
- **Hydrate wins a different axis, and the two combine.** Ponytail makes a build
  lean. Hydrate makes the *next* session reuse what the last one built instead of
  rebuilding it. On a cross-session reuse benchmark, Hydrate (and Hydrate + ponytail)
  reuse existing components every time, where a no-memory baseline rebuilds them.
  Ponytail is a dedicated concision tool; Hydrate's product is cross-session memory
  and orchestration. They are complementary, and running both is the strongest setup.

## The headline numbers

- **The YAGNI directive alone cut a build by about 78%** in cost, from $1.53
  (v5h) to $0.34 (v4h), with everything else held constant: same model, same
  prompt, same environment. Agent turns fell from 22 to 3 and output from 740 to
  285 lines. This is the single cleanest comparison in the set, because only the
  directive changed.
- **The shipped `/hydrate-yagni` command reproduces it** (v6): about $0.43, 217
  lines, 12/12, the smallest output in the whole benchmark. The concision also
  showed up behaviourally, not just on cost. The model deleted a dead API call it
  had written once it realised the endpoint did not support that lookup.
- **The cost span at one quality level is large.** Within single shots it is over
  4x ($0.34 to $1.53). Counting the orchestration tier above (around $7 to $11) it
  is roughly 20x. That span is the argument for matching the tier to the task instead
  of always reaching for the most powerful one.

## Headless vs interactive (TUI), and why it does not matter

Two ways of running Claude Code appear in the table, and it is worth being clear
about them because they are easy to conflate with the real cost drivers.

- **Headless** means the scripted, non-interactive mode (`claude -p "<prompt>"`).
  It is what a CI job or a batch script uses. Arms A, B, v4h and v5h are headless.
- **Interactive (TUI)** means a real terminal session, the way a person actually
  works. Arms A-TUI, v4, v5 and v6 are interactive. The `/hydrate-yagni` command
  only exists here, so v6 had to be an interactive run.

The natural worry is that the interactive session is the expensive one. It is not.
With everything else held equal (same plain prompt, same model, no directive) the
headless vanilla run cost $0.78 and the interactive one cost $0.72. That difference
is noise. The harness is cost-neutral.

What this lets you read off the table cleanly: the gap between a YAGNI arm and a
plain arm is the directive, not the harness. v6 (interactive, $0.43) sits a few
cents above v4h (headless, $0.34) only because the interactive session had its full
plugin environment loaded, where the headless arm was stripped to nothing. The
command delivers the concision prompt; it does not strip your environment. So in
everyday interactive use, expect the YAGNI cluster ($0.34 to $0.44), not the exact
headless floor.

## Honest caveats

- **Sample size.** Almost every figure is a single run (n=1). The 78% headline is
  the mean of three. Treat the bands as directional, not precise.
- **One task, one language.** This is a single front-end app. The directive is
  general, but these specific numbers are not a guarantee for every workload.
- **Greenfield by design.** "Build me a weather app" has no prior code to remember,
  so this benchmark isolates *concision*. It is not a test of memory or of
  long-running projects. That axis is measured separately by the cross-session
  reuse benchmark in [BENCHMARKS.md](../../BENCHMARKS.md).

## Files here

- [`prompt.txt`](prompt.txt): the verbatim task (embedded above).
- [`rubric.md`](rubric.md): the 12-point scoring checklist.
- [`results/metrics.json`](results/metrics.json): every arm's cost, tokens, LOC and score.
- [`results/*.png`](results/): representative built-app screenshots.
