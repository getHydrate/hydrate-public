# Main README figure brief

Five conceptual diagrams to illustrate the top-level `README.md`. The README already
has screenshots (logo, distill, resume, dashboard); these are the conceptual sections
that prose makes dense. All copy and data are pinned here so no lookup is needed.

## Build method (read first)

Build each figure as a **self-contained HTML/CSS file rendered to PNG** with headless
Chrome, not a generated/diffusion image, so text and numbers stay exact and the output
re-renders if anything changes.

```sh
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu \
  --hide-scrollbars --force-device-scale-factor=2 --window-size=1200,675 \
  --screenshot=fig-01-platform-layer.png fig-01-platform-layer.html
```

Render at 2x. Canvas 1200x675 unless noted. Output PNGs next to the HTML in this
directory (`assets/figures/`). Reference them from README.md as `assets/figures/<name>.png`.

## Shared style tokens (use in every figure)

```css
:root{
  --bg:#fbfcfd; --ink:#0f172a; --muted:#64748b; --line:#e2e8f0;
  --hydrate:#0d9488;        /* teal  — Hydrate */
  --hydrate-deep:#0f766e;
  --slate:#475569;          /* neutral — runtimes / secondary */
  --grey:#cbd5e1;
  --win:#16a34a; --warn:#d97706; --info:#2563eb;
  font-family:"Inter",system-ui,-apple-system,"Segoe UI",sans-serif;
}
```

Rules: no emojis, British spelling, no em dashes or en dashes in figure text (use
commas, colons, or "to" for ranges), straight quotes, a small `HYDRATE` wordmark in a
corner. Keep captions to one line. The reference build pattern (scaffold, header,
caption) is `../../slash-commands/benchmarks/figures/figure-01-scoreboard.html` in this
repo; copy its structure for consistency.

---

## fig-01-platform-layer  (hero, for "What you get")

- **Purpose:** the core positioning, "the platform layer for your coding agents".
- **Layout:** top row of five runtime chips, all connecting down to one wide Hydrate
  daemon block, with a capability row inside it, and a bottom strip.
- **Runtime chips (top):** Claude Code, OpenAI Codex, Cursor, Mistral Vibe, GitHub Copilot.
- **Daemon block (centre, teal):** label "Hydrate, one local daemon". Inside, four
  capability pills: Memory, Orchestration, Token reduction, Peernet.
- **Connectors:** thin lines from each chip down into the daemon block.
- **Bottom strip:** "One local daemon. No cloud. Five Go binaries under 10 MB. Nothing leaves your machine."
- **Title:** "The platform layer for your coding agents."

## fig-02-why-without-with  (for "Why Hydrate")

- **Purpose:** the before/after.
- **Layout:** two equal panels side by side.
- **Left panel (muted/grey), heading "Without Hydrate":**
  - every /clear is amnesia
  - each runtime is its own island
  - re-asks your stack for the twelfth time this week
- **Right panel (teal), heading "With Hydrate":**
  - conventions, decisions and corrections already there next turn
  - Claude today, Codex tomorrow, one shared store
  - fewer tokens to use them (11.1% fewer output tokens)
- **Title:** "Why Hydrate."

## fig-03-works-with-channels  (for "Works with")

- **Purpose:** the three independent channels and the honesty differentiator.
- **Layout:** Hydrate daemon on the left, a Runtime (the model) on the right, three
  labelled horizontal connectors between them.
- **Channels:**
  1. **Inject hook** (Hydrate to model): pushes remembered context in at session start.
  2. **Capture hook** (model to Hydrate): ingests the transcript when a session ends.
  3. **MCP** (two-way): the model pulls memory on demand.
- **Honesty badge / caption:** "Three independent channels, delivery-tested per runtime.
  Supported means the model received the context, not that a config file was written."
- **Title:** "Works with: three channels, honestly tested."

## fig-04-orchestration-loop  (for "Orchestration")

- **Purpose:** the adversarial, fail-closed loop.
- **Layout:** a cycle. Author (Opus) to Critic (Codex, cross-family) to Objection ledger
  to Convergence test, looping back to Author while objections remain, and exiting to a
  Gate.
- **Node labels:**
  - Author (Opus): drafts the artefact
  - Critic (Codex, different family): prompted to refute, not rubber-stamp
  - Objection ledger: severity, deduped, recurrence-counted
  - Convergence test: zero material objections + acceptance met, with a round cap
  - Gate (fail-closed): ships on a clean pass, escalates to needs_human when stuck, never a false green
- **Title:** "Orchestration: quality through structured disagreement."

## fig-05-benchmarks-scoreboard  (for "Benchmarks")

- **Purpose:** make the four headline numbers pop.
- **Layout:** four metric tiles in a row + a footer strip.
- **Tiles (label / big value / sub-line):**
  1. Token reduction (headline) / **11.1%** / fewer output tokens, 6.3% lower cost, at quality parity (lquery A/B, n=10)
  2. Retrieval / **R@10 0.86** / 0.95 on multi-session (LongMemEval, n=500)
  3. Compression / **99.2%** / 475.7M to 3.7M tokens (22,483 sessions)
  4. Speed and footprint / **sub-10 ms** / no reranker, no GPU, no model download
- **Footer strip:** "Local-first, pure Go, no telemetry. Audit the whole surface in an afternoon."
- **Title:** "Benchmarks: the axes that matter for a local memory layer."

---

## Placement in README.md

- fig-01: in "What you get", near the top.
- fig-02: in "Why Hydrate".
- fig-03: in "Works with", above or below the runtime matrix.
- fig-04: in "Orchestration", below the intro paragraph.
- fig-05: in "Benchmarks", above the metrics table.
