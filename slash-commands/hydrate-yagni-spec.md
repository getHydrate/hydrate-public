<!-- SPDX-License-Identifier: MIT -->

> Cost: ~150 tokens, ~50ms, one Bash call to `hydrate yagni-block --spec`

Author the spec / plan / design the user asks for under Hydrate's YAGNI concision
discipline for **specifications** — the spec-side analog of `/hydrate-yagni`
(which disciplines the *build*). This is the same concision rubric Hydrate's
orchestration design-critic enforces, applied up front as authoring guidance so
you write a tight spec the first time instead of having it cut afterwards.

Use this for writing a spec, plan, design doc, RFC, or acceptance criteria —
anything that *determines* a build rather than *being* the build. For writing the
code itself, use `/hydrate-yagni`.

## Step 1 — load the canonical directive

Run, via Bash, exactly:

    hydrate yagni-block --spec

This prints Hydrate's canonical **spec-authoring** YAGNI directive. It is pinned
to the **single source of truth** (`internal/provider/yagni.go`): its body embeds
the design critic's own objection criteria (Verbosity + Rework-risk and the
cut-prose-never-decisions rules) verbatim, so the guidance you write to and the
rubric that would grade the spec can never drift. Do NOT paraphrase or substitute
your own concision rules; use what the command prints.

If the `hydrate` binary is missing or the call fails, fall back to this directive
verbatim and proceed:

> SPECIFICATION STYLE — YAGNI: write the most concise spec that still fully
> determines the build. Cut prose that doesn't change implementation,
> verification, or operator behaviour (repeated context, hedging, obvious
> explanation, motivational filler, non-operative examples), and don't mandate
> abstractions, config knobs, interfaces, or extensibility the requirement doesn't
> evidence. Prefer the smallest statement that makes each decision unambiguous and
> verifiable. NET: cut prose and speculative scope, but DO commit the
> expensive-to-change boundary decisions now — contracts, data model, protocol,
> state ownership, lifecycle, naming, compatibility. Under-specifying a
> costly-to-retrofit boundary is a YAGNI violation, not concision. "Concise" never
> overrides completeness: keep every real decision, acceptance criterion, and
> load-bearing edge case — cut prose, never decisions.

## Step 2 — author the spec under that directive

Treat the printed directive as a **binding authoring-style constraint** for the
rest of this task, then carry out the user's request:

{{args}}

Hold the line on the two guardrails — note they cut in **opposite directions**,
which is the whole point of a spec-specific directive:

- **Cut prose, never decisions.** Concision never justifies dropping a real
  decision, acceptance criterion, or load-bearing edge case. Trim explanation,
  hedging, and repetition — not the substance that prevents divergent builds.
- **Commit the expensive-to-change boundary now.** Unlike code YAGNI (build only
  what's needed), a spec MUST pin the costly-to-retrofit decisions — contracts,
  data model, protocol, state ownership, lifecycle, naming, compatibility —
  wherever an in-context requirement makes a later change materially more
  invasive. Under-specifying those is a YAGNI violation, not brevity. Do NOT,
  however, invent a roadmap you cannot see: mandate a boundary only on evidence
  present in the task, not on a speculative future.

Write the tightest spec that fully and unambiguously determines the build. Don't
pad with motivational framing, restated context, or non-operative examples; don't
over-specify speculative flexibility; but don't under-specify the load-bearing
boundaries. If you find yourself writing far more prose than the decisions need,
cut back to the decisions.

If no task was supplied after `/hydrate-yagni-spec`, print the directive from
Step 1 and tell the user to add their task after the command
(e.g. `/hydrate-yagni-spec write a spec for X`).
