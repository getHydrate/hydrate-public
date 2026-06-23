# Figure brief — benchmark illustrations

Build briefs for five illustrative figures that convey the Hydrate / Ponytail
benchmark story visually, for the slash-commands README and BENCHMARKS.md. All data
is pinned here so the figures need no external lookup. Numbers come from
[`../../BENCHMARKS.md`](../../BENCHMARKS.md).

## How to build (read first)

Build each figure as a **self-contained HTML/CSS file rendered to PNG** with headless
Chrome — not a generated/diffusion image. Reason: the figures are data, and a
screenshot keeps the numbers exact, the text crisp, and the output reproducible if a
number changes.

```sh
# one figure
chromium --headless --disable-gpu --force-device-scale-factor=2 \
  --window-size=1200,675 --screenshot=figure-01-scoreboard.png figure-01-scoreboard.html
# (or: npx playwright screenshot --viewport-size=1200,675 figure-01-scoreboard.html figure-01-scoreboard.png)
```

- Render at 2x device scale for retina-crisp PNGs.
- Default canvas 1200x675 (16:9, embeds well in GitHub and on slides); figure 4 is
  taller (1200x900) because it stacks a flow over a chart.
- Output PNGs next to the HTML in this directory. Reference them from the READMEs as
  `benchmarks/figures/figure-0N-*.png`.

## Shared style tokens

Use one CSS variable block in every figure so the set looks consistent.

```css
:root{
  --bg:#fbfcfd; --ink:#0f172a; --muted:#64748b; --line:#e2e8f0;
  --hydrate:#0d9488;        /* teal  — Hydrate */
  --ponytail:#475569;       /* slate — Ponytail */
  --baseline:#cbd5e1;       /* grey  — Baseline */
  --win:#16a34a;            /* green — Hydrate wins this axis */
  --tie:#d97706;            /* amber — a tie / reproduced */
  --held:#2563eb;           /* blue  — deliberate trade, "held by design" */
  --trade:#94a3b8;          /* muted — the lower side of a deliberate trade (never alarm-red) */
  font-family:"Inter",system-ui,-apple-system,"Segoe UI",sans-serif;
}
```

Rules: no emojis; British spelling; the Hydrate wordmark small in a corner; one-line
captions; the legend (Hydrate / Ponytail / Baseline) shown once where bars appear.
Status colours mean: green = Hydrate wins, amber = tie/reproduced, blue = held by
design, muted = the traded-away side. **Never colour the frontend "loss" red** — it is
a deliberate product choice, not a defect.

`figure-01-scoreboard.html` in this directory is the reference build — copy its
`<style>` scaffold for the other four.

---

## Figure 1 — Hero scoreboard ("who wins what")

- **File:** `figure-01-scoreboard.html` → `figure-01-scoreboard.png` (1200x675). Built; use as the template.
- **Purpose:** the one image at the top of the Benchmarks section; replaces the dense tables at a glance.
- **Layout:** title row, then four equal tiles in a row, then a full-width footer strip.
- **Tiles (heading / big value / sub-line / accent):**
  1. `SAFETY` / `100%` / "reproduced exactly — naive one-liner drops to 94.4%" / amber (--tie)
  2. `BACKEND LOC` / `−6%` / "a tie with Ponytail" / amber (--tie)
  3. `FRONTEND LOC` / `held` / "Ponytail −64%; Hydrate keeps higher-fidelity UI by design" / blue (--held)
  4. `CROSS-SESSION REUSE` / `1.00` / "Hydrate reuses every time; Ponytail can't play (no memory)" / teal (--win)
- **Footer strip (highlighted):** "Hydrate + Ponytail — the strongest combination."

## Figure 2 — Safety bars

- **File:** `figure-02-safety.html` → `.png` (1200x675).
- **Purpose:** make Ponytail's safety argument land instantly.
- **Chart:** vertical bars, y-axis 90–100% (zoomed so the dip reads), value labels on each bar.
  - Baseline **100%** (grey) · Ponytail **100%** (slate) · Hydrate `/hydrate-yagni` **100%** (teal) · Naive "prefer one-liners" prompt **94.4%** (muted, the one dipped bar, with a small callout "drops a guard to shorten code").
- **Sub-label:** "% safe, 90 runs per arm, Claude Haiku, on Ponytail's own harness."
- **Caption:** "Hydrate reproduced Ponytail's safety result exactly — same failing task and model."

## Figure 3 — Backend vs Frontend LOC (grouped bars)

- **File:** `figure-03-loc.html` → `.png` (1200x675).
- **Purpose:** show the backend tie and the deliberate frontend trade in one view.
- **Chart:** two groups, three bars each (Baseline grey / Ponytail slate / Hydrate teal), median LOC, value labels.
  - **Backend:** 34 / 32 / 32 — group annotation "tie (~6% under baseline)".
  - **Frontend:** 256 / 93 / 256 — annotate the Hydrate bar in blue (--held): "held by design — higher-fidelity UI". Do not use red.
- **Sub-label:** "median lines of code, Claude Haiku."
- **Caption:** "Backend: tied. Frontend: a deliberate trade, not a loss."

## Figure 4 — Cross-session reuse (concept flow + bars)

- **File:** `figure-04-reuse.html` → `.png` (1200x900, taller).
- **Purpose:** explain the axis Hydrate wins — a table can't convey the two-phase shape.
- **Top half — flow, left to right:**
  - Node A: "Session 1 — build a DatePicker component".
  - Divider: "context cleared — the gap" (a dashed vertical break).
  - Then Session 2 splits into two rows:
    - **Cold (no memory):** "rebuilds a duplicate" — muted/--trade box, tag "+~170 redundant LOC".
    - **Hydrate:** "imports the existing component" — teal arrow back to component A, tag "reused".
- **Bottom half — bars, reuse rate (0–1):** Cold **0.67** (grey) · Hydrate **1.00** (teal) · Hydrate + Ponytail **1.00** (teal, slightly distinct outline).
- **Sub-label:** "reuse rate, early pilot, 18 runs, Claude Haiku."
- **Caption:** "No memory → rebuilds. Hydrate → reuses every time."

## Figure 5 — Complementary layers (thesis diagram)

- **File:** `figure-05-layers.html` → `.png` (1200x675).
- **Purpose:** drive home "complementary, not competing".
- **Layout:** two stacked horizontal bands feeding into one highlighted block.
  - Band 1 (slate): "Ponytail — concision *within* a build" (icon: a build/file shrinking).
  - Band 2 (teal): "Hydrate — memory *across* builds" (icon: an arrow from one session to the next).
  - Joined block (teal, emphasised): "Hydrate + Ponytail" with two ticks: "reuse guarantee" and "lean new code".
- **Subtext:** "Ponytail makes a build lean. Hydrate makes the next session reuse it. Orthogonal layers that compound."

---

## Placement once built

- Figure 1 (scoreboard): top of the README `## Benchmarks` section and top of BENCHMARKS.md.
- Figures 2 + 3: in/near README benchmark #1, beside the safety and LOC grids.
- Figure 4: README benchmark #1 cross-session reuse block, and BENCHMARKS.md section 4.
- Figure 5: BENCHMARKS.md section 5 ("Why Hydrate + Ponytail"), and the README TL;DR area.
