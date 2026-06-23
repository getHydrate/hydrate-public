# weather-bench — execution-tier cost ladder

A single fixed task (`prompt.txt`) — "build a weather dashboard app" — built many
ways, scored against one fixed 12-criterion functional rubric (`rubric.md`), to
measure **cost at constant quality** across execution tiers. It is the evidence
base for the lean / assisted / orchestrated execution-tier triage: pick the
cheapest tier that does the job, and step up only when the task earns it.

Every arm in `results/REPORT.md` scored **12/12** (verified from source + a live
Playwright load), so the spread is pure cost/output-size at equal quality:

- **lean (YAGNI single-shot):** ~$0.34–0.44, ~217–297 LOC
- **assisted (full-env single-shot, no YAGNI):** ~$0.72–1.53, ~736–844 LOC
- **orchestrated (design→develop fleet):** ~$7–11

A **~20× cost span at the same rubric score** — the result the triage exists to
exploit by matching the tier to the task.

## Headline findings

- **YAGNI is the dominant single-shot cost lever** — held-Hydrate-ON, the YAGNI
  directive alone cut a build **−78%** (v5h $1.53 → v4h $0.34) at 12/12 parity.
- **`/hydrate-yagni` (the shipped skill) reproduces the lean tier** — `v6`:
  $0.43 / 217 LOC / 12/12, the lowest LOC in the bench.
- **Hydrate is ~cost-neutral on a cold one-shot** — the expensive single-shots
  were driven by Playwright (browser-verification loops) and frontend-design
  (gold-plating), not Hydrate; its MCP tools were called 0/24 on a greenfield
  build. See REPORT §"MCP isolation was confounded".

## What's committed here

Results only — not the per-arm build trees or the throwaway Claude config dirs
used to isolate harness state:

- `prompt.txt`, `rubric.md` — the fixed task + scoring checklist.
- `results/REPORT.md` — the full cost ladder, isolations, and findings.
- `results/metrics.json` — structured per-arm metrics (cost, tokens, LOC, rubric).
- `results/*.png` — representative built-app screenshots.

Caveat carried from the report: cost bands are **n=1** except the YAGNI −78% and
the MCP-confound (n=3). Re-run the tier representatives at n=3 before any cost
figure ships publicly.
