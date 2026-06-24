# Benchmarks: Hydrate and Ponytail

Hydrate and [Ponytail](https://github.com/DietrichGebert/ponytail) solve different
problems, and they work well together. Ponytail makes a single build lean. Hydrate
remembers what was built so the next session reuses it instead of rebuilding it. This
page reproduces Ponytail's published results, reports Hydrate's honestly (wins, ties,
and losses), and explains why running both is the strongest setup.

All numbers come from Ponytail's own agentic benchmark harness (`benchmarks/agentic`),
its scorers, and Anthropic subscription auth, plus a cross-session benchmark Hydrate
built to measure the axis Ponytail does not target.

![Benchmark scoreboard: safety reproduced (100%), backend a tie, frontend held by design, cross-session reuse 1.00, and Hydrate + Ponytail as the strongest combination](benchmarks/figures/figure-01-scoreboard.png)

## Results at a glance

| Benchmark | Baseline | Ponytail | Hydrate | Hydrate + Ponytail | Notes |
|---|---|---|---|---|---|
| **Safety** (% safe) | 100% | 100% | **100%** | n/a | 90 runs/arm. A naive "prefer one-liners" prompt drops to **94.4%** |
| **Backend** (median LOC) | 34 | 32 | **32** | n/a | A tie, both about 6% under baseline. 60 runs/arm |
| **Frontend** (median LOC) | 256 | **93** | ~256 | n/a | Ponytail 64% leaner. Hydrate holds higher-fidelity UI by design. 30 runs/arm |
| **Cross-session reuse** (reuse rate) | 0.67 | n/a | **1.00** | **1.00** | Ponytail has no cross-session memory. Early pilot, 18 runs |
| **New code on a reuse task** (avg LOC) | 196 | n/a | 247 | **207** | Combined arm keeps reuse and trims new code |

In one line: Ponytail and Hydrate tie on safety and backend concision, Ponytail wins
frontend line count, Hydrate wins cross-session reuse, and Hydrate + Ponytail is the
strongest combination. The sections below explain each row, including why the
differences exist.

## 1. Reproducing Ponytail's results

Hydrate ran Ponytail's suite end to end before claiming anything about it, and the
headline results reproduced cleanly.

Safety here means keeping the guards and error checks a correct implementation needs,
rather than stripping them to shorten the code. Ponytail, the baseline, and Hydrate's
concision directive all score 100%. The naive "follow YAGNI, prefer one-liners" prompt
drops to 94.4%, and Hydrate reproduced not just that rate but the exact failure
signature Ponytail published: every unsafe run was the same task and model (a CSV-sum
task on Sonnet). This is the core of Ponytail's argument, that a careless concision
prompt drops safety, and it holds.

On backend code, Ponytail and Hydrate's concision directive are a statistical tie. Both
land at a median of 32 lines against a 34-line baseline, at full correctness.

Ponytail's safety claim and its backend concision both replicate. Hydrate endorses them.

![Safety bars: baseline, Ponytail and Hydrate all 100% safe; the naive one-liner prompt drops to 94.4%](benchmarks/figures/figure-02-safety.png)

## 2. Where Ponytail wins, and why

On frontend component tasks Ponytail wins clearly: a median of about 93 lines against a
256-line baseline, roughly a 64% reduction. Hydrate's concision directive stays close to
baseline here. This is a deliberate difference, not a defect.

Ponytail reaches aggressively for the leanest control that exists: a native
`<input type="date">` over a hand-rolled calendar, an already-installed component over a
new one. That is excellent for raw line count. It can also mean a plainer interface than
a design-led product wants. Hydrate treats interface design as its own step, handled by a
design pass, and applies concision afterwards to reuse and tighten those components
rather than golf them away. The result is a higher-fidelity frontend at the cost of more
lines.

Hydrate tested the alternative honestly. A reuse-first instruction was added to Hydrate's
concision directive to chase the frontend line count directly. The result was a wash:
no net reduction across the frontend suite, and a worse build, because on components with
no native equivalent the instruction pushed the agent to over-build. The change was
reverted. The lesson is recorded rather than hidden. A one-paragraph directive does not
out-golf a purpose-built concision tool on frontend work, and chasing the number degrades
the product.

![Median lines of code: backend tied at 32 across Ponytail and Hydrate; on frontend Ponytail is leaner while Hydrate holds higher-fidelity UI by design](benchmarks/figures/figure-03-loc.png)

## 3. The two tools measure different things

The comparison is friendly rather than competitive because the tools work on different
axes.

- **Ponytail governs concision within a build:** the leanest correct implementation, in
  this file, right now.
- **Hydrate governs memory across builds:** what already exists, what was decided, what
  was built last session, carried into the next one.

Ponytail has no cross-session memory, so it can only reuse what it finds in the current
context. A line-count benchmark on a fresh, empty task cannot show memory, because there
is nothing prior to remember. Measuring memory needs a benchmark that forces a gap
between sessions.

## 4. The cross-session reuse benchmark

Hydrate built a two-phase benchmark for exactly this. A repository is seeded with two
components built by Ponytail, so phase one is Ponytail's own lean work. The context is
then cleared, and a fresh agent is asked for a feature that needs those components, for
example a booking form needing a date field and a file upload. The question scored is
simple: did the new session reuse the existing components, or rebuild them?

Without memory, a fresh agent rebuilds. Two of six cold runs hand-rolled a second copy of
a component the repository already had, roughly 170 redundant lines of duplicated work.
With Hydrate's memory injected, every run reused the existing component instead, for a
reuse rate of 1.00 against the cold baseline's 0.67.

These figures are an early 18-run pilot. The direction is clear, and the absolute numbers
will firm up with a larger run.

![Cross-session reuse: after the context gap, a no-memory agent rebuilds a duplicate (reuse rate 0.67) while Hydrate and Hydrate + Ponytail reuse every time (1.00)](benchmarks/figures/figure-04-reuse.png)

## 5. Why Hydrate + Ponytail is the strongest setup

The combined arm is not a compromise between the two. It takes the best of each. Hydrate's
memory delivers the reuse guarantee: every run reused the existing component, zero
duplicates. Ponytail's concision keeps the new code lean, with average new lines falling
from 247 with Hydrate alone to 207, and cost and time dropping back to roughly the cold
baseline.

In one line: Hydrate + Ponytail gives the cost and tightness of a no-memory run, with the
reuse guarantee of a memory layer. Memory stops the agent rebuilding what exists, and
Ponytail keeps whatever it does write minimal. The two layers compound, and the
combination is something Ponytail alone cannot produce, because it has no memory of prior
sessions.

Ponytail does everything it claims, and Hydrate has verified it. Hydrate wins on a
different axis, memory across sessions, and the two together are stronger than either
alone. Run both.

![Complementary layers: Ponytail governs concision within a build, Hydrate governs memory across builds, and together they give a reuse guarantee plus lean new code](benchmarks/figures/figure-05-layers.png)

---

*For the single-shot concision benchmark behind the `/hydrate-yagni` command, a weather
dashboard built many ways and scored against a fixed rubric, see
[`benchmarks/weather-bench`](benchmarks/weather-bench). Methodology, sample sizes, and the
full internal scorecard (including retrieval accuracy, compression, and latency
benchmarks) are maintained alongside the Hydrate source. The cross-session reuse figures
are an early pilot and will be updated as the benchmark is scaled.*
