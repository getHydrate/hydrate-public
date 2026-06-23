<!-- SPDX-License-Identifier: MIT -->

> Cost: ~150 tokens, ~50ms, one Bash call to `hydrate yagni-block`

Carry out the user's task under Hydrate's YAGNI concision discipline — the same
"build only what's needed" directive Hydrate injects into its orchestration
develop-mode implementer prompts, now available for a single normal session
without spinning up a fleet.

This is the user-invokable form of the orchestrator's `lean` execution tier: the
proven cost lever (a single concision directive cut a measured single-shot build
~78% in tokens/cost at equal quality) applied to whatever you ask next.

## Step 1 — load the canonical directive

Run, via Bash, exactly:

    hydrate yagni-block

This prints Hydrate's canonical implementer-side YAGNI directive. It is the
**single source of truth** (`internal/provider/yagni.go`) — the identical text the
orchestration implementer prompt uses, so the manual and automatic paths never
drift. Do NOT paraphrase or substitute your own concision rules; use what the
command prints.

If the `hydrate` binary is missing or the call fails, fall back to this directive
verbatim and proceed:

> IMPLEMENTATION STYLE — YAGNI: bias to the most concise correct implementation.
> Build only what the task needs — no speculative abstractions, config knobs,
> interfaces, or flexibility for hypothetical futures, and no error handling for
> states that cannot occur given the call sites. Prefer the smallest direct
> expression and existing helpers over hand-rolled code. "Concise" never overrides
> correctness or this project's fail-safe invariants: keep every necessary guard
> and error check even when it adds a line — do NOT swallow errors or drop a guard
> to shorten the code. Exception: DO build any interface, seam, or boundary the
> task brief or plan explicitly mandates — that is design-time accommodation
> chosen to avoid foreseeable retrofit, not speculative flexibility.

## Step 2 — do the task under that directive

Treat the printed directive as a **binding implementation-style constraint** for
the rest of this task, then carry out the user's request:

{{args}}

Hold the line on what the directive says, especially the two guardrails:
- **Correctness and fail-safe invariants always win.** Concision never justifies
  dropping a necessary guard, swallowing an error, or skipping a real edge case.
- **Mandated seams are not YAGNI violations.** If the task brief or an existing
  plan explicitly calls for an interface/boundary, build it — that is foreseen
  accommodation, not speculative flexibility.

Build the smallest correct thing that fully satisfies the request. Don't gold-plate
(no unrequested features, branding, visualisations, or "flexibility"), don't add
abstractions for single-use code, and don't write error handling for states that
cannot occur. If you find yourself writing far more than the task needs, stop and
cut back.

If no task was supplied after `/hydrate-yagni`, print the directive from Step 1 and
tell the user to add their task after the command (e.g. `/hydrate-yagni build X`).
