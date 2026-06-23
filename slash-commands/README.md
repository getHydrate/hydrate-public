# Hydrate slash commands (MIT)

Two Claude Code slash commands that apply Hydrate's concision discipline to a
single session, without running an orchestration fleet. Unlike the rest of this
repository, the files in this directory are MIT-licensed (see
[`LICENSE`](LICENSE)), so you are free to use, copy, modify and redistribute them.

- **`hydrate-yagni.md`** disciplines the *build*: carry out a task under the same
  "build only what is needed" directive Hydrate injects into its develop-mode
  implementer prompts. A single concision directive cut a measured single-shot
  build by about 78% in tokens and cost at equal quality (see
  [Benchmarks](#benchmarks)).
- **`hydrate-yagni-spec.md`** disciplines the *spec*: author a spec, plan, design
  doc or acceptance criteria under the concision rubric Hydrate's design critic
  enforces, so you write a tight spec the first time. It cuts prose but commits
  the expensive-to-change boundary decisions, which is the opposite emphasis to
  the build directive.

Both commands call `hydrate yagni-block` for the canonical directive text when the
Hydrate binary is present, and fall back to a verbatim copy of that directive when
it is not, so they work standalone.

## Where these directives come from

These are not prompts written for a blog post. They are the exact directives
**Hydrate's orchestration engine injects into its own worker agents**, lifted out
verbatim so a single session can use them without running a fleet.

- The **build** directive is the one Hydrate gives its **develop-mode
  implementer** — the agent that writes code.
- The **spec** directive is the one Hydrate's **design-mode critic** scores a spec
  against.

They are subtly, deliberately different. Running the orchestrator across many real
tasks showed that building and specifying need different emphases — in one place
the opposite emphasis — so the two directives diverged by experience rather than
by theory. The section below explains exactly how.

## Why we call it "YAGNI" (and how our definition differs)

"YAGNI" — *You Aren't Gonna Need It* — is a familiar shorthand for "don't build
what the task doesn't need", so we use the name. But these directives are **not**
the textbook principle. We define our own operational contract, because naive
YAGNI fails in two ways that matter, and the contract is written to get the same
logical result (lean, correct output) without either failure.

### The core difference from code YAGNI

The implementer directive optimises one thing: write less code. A spec directive
has to balance two axes that pull against each other:

1. **Prose verbosity** — cut words that don't change implementation, verification
   or operator behaviour. This pulls toward shorter.
2. **Scope / over-specification** — don't mandate seams, abstractions, config or
   extensibility the requirement doesn't evidence. This also pulls toward less.

…but with a guardrail that points the **opposite way** from code YAGNI, and this
is the part you can't get wrong:

- **Implementer YAGNI:** "build only what's needed — but DO build any seam the
  brief mandates."
- **Spec YAGNI:** "specify only what's needed — but DO commit the
  expensive-to-change boundary decisions now (contracts, data model, protocol,
  state ownership, lifecycle, naming, compatibility), because under-specifying a
  costly-to-retrofit boundary is the expensive mistake."

A naive "make the spec shorter" directive would violate that by encouraging
under-specification of contracts. So a spec YAGNI is "cut prose and speculative
scope, but pin the load-bearing boundary decisions" — not "specify less".

The build directive carries its own version of the same guard: concision never
overrides correctness. It keeps every necessary error check and edge-case guard
even when that adds lines, and it never drops a guard to shorten the code. That is
the second way naive YAGNI fails — trimming real correctness in the name of
brevity — and the directive is written to refuse it.

## Install

Drop the files into your Claude Code commands directory:

```sh
mkdir -p ~/.claude/commands
curl -fsSL -o ~/.claude/commands/hydrate-yagni.md \
  https://raw.githubusercontent.com/getHydrate/hydrate-public/main/slash-commands/hydrate-yagni.md
curl -fsSL -o ~/.claude/commands/hydrate-yagni-spec.md \
  https://raw.githubusercontent.com/getHydrate/hydrate-public/main/slash-commands/hydrate-yagni-spec.md
```

Then use them in any Claude Code session:

```
/hydrate-yagni build the X endpoint
/hydrate-yagni-spec write a spec for the X endpoint
```

Called with no task, each command prints its directive and tells you to add a task
after the command.

## When to use which

These commands are the cheap, single-session end of a spectrum. Pick the lowest
tier that clears the bar, and default to the cheapest.

| Situation | Reach for |
|-----------|-----------|
| A well-specified, known-pattern task you can verify cheaply and fix cheaply if it is wrong (most work) | **`/hydrate-yagni`** |
| The task is ambiguous, or the design could reasonably take several shapes | **`/hydrate-yagni-spec`** first to tighten the spec, then build it (optionally under `/hydrate-yagni`) |
| Correctness is subtle (concurrency, security, a protocol, a data migration); the design space is wide enough that converging a spec de-risks the build; the scope is large or spans many files; or a latent bug would cost far more than a multi-agent run | The full **Hydrate orchestration** pattern (design and develop fleets), which is the broader Hydrate product, not these MIT commands |

A quick way to triage: score the task on five questions and step up only when one
genuinely clears the bar.

1. **Specification.** Is the brief unambiguous and the pattern known? If not, spec first.
2. **Cost of being wrong.** Throwaway, or production / security / data-loss?
3. **Verification.** Can a cheap check confirm it, or does it need real integration testing?
4. **Design space.** One obvious approach, or several competing ones worth converging?
5. **Scope.** A single file, or many files and packages?

Low on all five is a `/hydrate-yagni` job. A wide design space or a large
multi-file scope is where orchestration earns its cost. Most tasks are the
former, which is the whole point: do not pay fleet prices for a one-file build.

## Benchmarks

The concision claim is measured, not asserted. The benchmark in
[`benchmarks/weather-bench`](benchmarks/weather-bench) builds one fixed task (a
weather dashboard app) many different ways and scores each against the same
12-point functional checklist. Every build scored 12/12, so the cost differences
below are cost at equal quality, not quality differences.

| How the task was run | Cost | Lines of code |
|----------------------|-----:|--------------:|
| YAGNI single shot (the `/hydrate-yagni` tier) | **$0.34 – $0.44** | ~217 – 297 |
| Plain single shot, no concision directive | $0.72 – $1.53 | ~736 – 844 |
| Full multi-agent orchestration | $7 – $11 | varies |

- **The YAGNI directive alone cut a build by about 78%** in cost (from $1.53 to
  $0.34), with the model, prompt and environment all held constant — only the
  directive changed.
- **The shipped `/hydrate-yagni` command reproduces it**: about $0.43 at 217 lines
  and 12/12, the smallest output in the whole benchmark.
- The full span from a lean shot to an orchestration is roughly **20x at the same
  rubric score**, which is why matching the tier to the task matters.

Caveat: most figures are single runs; the 78% headline is the mean of three. The
benchmark isolates *concision* on a greenfield build, not memory or
long-running-project behaviour. Full method, the verbatim prompt, the per-arm
table and the headless-vs-interactive analysis are in the
[benchmark README](benchmarks/weather-bench).

## Use the directive without the command

If you would rather not install anything, paste the directive straight into your
prompt before the task. This is exactly what the commands inject.

**Build (the `/hydrate-yagni` directive):**

> IMPLEMENTATION STYLE — YAGNI: bias to the most concise correct implementation. Build only what the task needs — no speculative abstractions, config knobs, interfaces, or flexibility for hypothetical futures, and no error handling for states that cannot occur given the call sites. Prefer the smallest direct expression and existing helpers over hand-rolled code. "Concise" never overrides correctness or this project's fail-safe invariants: keep every necessary guard and error check even when it adds a line — do NOT swallow errors or drop a guard to shorten the code. Exception: DO build any interface, seam, or boundary the task brief or plan explicitly mandates — that is design-time accommodation chosen to avoid foreseeable retrofit, not speculative flexibility.

**Spec (the `/hydrate-yagni-spec` directive):**

> SPECIFICATION STYLE — YAGNI: write the most concise spec that still fully determines the build. Cut prose that doesn't change implementation, verification, or operator behaviour (repeated context, hedging, obvious explanation, motivational filler, non-operative examples), and don't mandate abstractions, config knobs, interfaces, or extensibility the requirement doesn't evidence. Prefer the smallest statement that makes each decision unambiguous and verifiable. NET: cut prose and speculative scope, but DO commit the expensive-to-change boundary decisions now — contracts, data model, protocol, state ownership, lifecycle, naming, compatibility. Under-specifying a costly-to-retrofit boundary is a YAGNI violation, not concision. "Concise" never overrides completeness: keep every real decision, acceptance criterion, and load-bearing edge case — cut prose, never decisions.

Prepend the relevant block, then write your task on the next line.
