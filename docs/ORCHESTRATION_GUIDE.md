# Orchestration: a practical guide

A working guide to Hydrate's orchestration engine: what it is, when to reach
for it, how to drive it, and the part you actually learn from, namely the real
design and development bugs it has caught and what that was worth.

The short version: orchestration is how Hydrate raises the **quality** of
agent work, not just your confidence in it. A single model doing a single pass
cannot give itself a sceptical second opinion. Orchestration supplies one, from
a different model family, and refuses to call the work done until the
disagreement is genuinely resolved.

> Audience: anyone driving an orchestration run from a Claude or Codex session.

## 1. What it is

Orchestration replaces "one model, one pass" with a persisted, adversarial
loop. A **generator** produces an artefact (a spec, a code patch). An
**independent reviewer**, ideally a different model family, attacks it. The
disagreement is tracked as a ledger of **objections**, and the loop stops only
when the material objections genuinely clear (**convergence**), or it escalates
honestly to a human (**`needs_human`**) when it cannot. Every transition is a
pure state machine, and all state is persisted to SQLite, so any run is
resumable and status-pollable. The whole thing is driven over MCP tools, so any
connected runtime can run it.

The thesis in one line: **quality comes from structured disagreement and
fail-closed gates, not from a single confident pass.** The benefit is
two-sided. You get better work, because a cross-family critic catches what the
author cannot see; and you get trustworthy work, because the gates will not
claim the job is done when it is not.

## 2. When to use it (and when not to)

Reach for it when the cost of being wrong is high and a second independent
opinion is worth the spend.

| Use orchestration when | Skip it when |
|---|---|
| Designing a spec or plan whose mistakes are expensive to unwind (schema, wire protocol, auth, public API) | The change is mechanical or obvious (rename, dependency bump, one-line fix) |
| The work is large enough to parallelise across independent units | A single focused edit with a clear test is faster done by hand |
| You want a cross-family second opinion before committing | You do not have a healthy second model family available (see section 7) |
| "Done" needs to be verified against criteria, not vibes | You would merge on a green test anyway and the blast radius is tiny |
| You will be away and want work that will not loop forever or claim to be done | You need a fast interactive turnaround and will review every line yourself |

A rule of thumb that has served us well: use Design mode for anything you would
otherwise write a design doc for, and Develop mode when the work genuinely fans
out into parallel units with a strong acceptance test.

## 3. The modes at a glance

| Mode | Artefact | Generator and reviewer | Stops on |
|---|---|---|---|
| **Design** | a plan or spec | Author → Critic (different family), you arbitrate | convergence (zero material objections, diminishing returns), then sign-off |
| **Develop** | code patches | Implementer → Reviewer → Judge (rubric) → post-merge Audit | every unit verified, then a human integrate gate |
| **Image / Studio** | images or UX | generator → design judge | accept |

Design and Develop are the mature core; the others reuse the same loop and
acceptance machinery. Each mode also runs an adversarial review pass over the
resulting diff, where every finding is verified against the real change before
it is reported, so a plausible-but-wrong objection does not block a good merge.

## 4. How to drive it: Design mode

Design mode is a set of MCP tools you call from your session:

```
design_start    { title, draft_path, hard_gate?, acceptance? }  -> session_id
design_round    { session_id }     # dispatch a critic round (slow: headless cross-family critic)
design_status   { session_id }     # poll: state, round, open objections
design_resolve  { ... }            # author addresses an objection (records a decision)
design_signoff  { session_id }     # only legal once material objections are zero and acceptance is met
design_abort    { session_id }
```

The loop you actually run:

1. Call **`design_start`** with a path to your draft spec. Optionally add an
   **acceptance block** (`acceptance:[...]`) so "done" is measurable, and
   `hard_gate:true` to escalate subjective-concision objections (see section 6).
2. **`design_round`**, then wait, then **`design_status`**. A round dispatches a
   headless cross-family critic and takes minutes. The MCP call may time out at
   the client while the round finishes server-side, so poll status rather than
   assume failure. The round has usually completed.
3. Read the objections. Address the material ones, then call `design_round`
   again.
4. Repeat until status shows convergence (zero open material objections,
   diminishing returns), then call **`design_signoff`**. If it cannot converge,
   it lands in `needs_human` at the round cap. That is the engine being honest,
   not broken.

One thing to watch: drive a single session from a single terminal. Two drivers
racing the same `session_id` will conflict.

## 5. How to drive it: Develop mode

```
develop_start             { title, goal, targets, hard_gate?, acceptance? } -> session_id
develop_plan              { session_id, units:[...] }   # one unit per parallelisable chunk
develop_status            { session_id }                # per-unit state and blocked reasons
develop_integrate         { session_id }                # human gate: merge verified units
develop_integrate_resolve { ... }                       # resolve a merge conflict
develop_audit / _resolve  { ... }                       # post-merge senior audit gate
develop_abort             { session_id }
```

The shape: plan, then units run in parallel worktrees (implement, review,
judge, each unit bounded by a round cap), then units land in `verified`. You
then call **`develop_integrate`**, the human gate, and a post-merge audit gate
runs a whole-package senior pass before `done`. Anything that cannot converge
(round cap, blocked unit) projects the session to `needs_human` with the reason
surfaced in `develop_status`.

A hard-won caveat: only trust Develop mode for byte-identical or mechanical work
when a strong golden or equivalence test already pins the output. Without such a
pin, integrate by hand. We have watched a fleet confidently produce subtly wrong
scaffolding that only a real golden test would have caught; on one occasion we
found and fixed four wrong values by reading the real implementation instead of
trusting the fleet.

## 6. The hard gate (per-run concision)

Both modes take `hard_gate:true`. It escalates subjective concision and
over-engineering objections from advisory to material, and a stall valve detects
two consecutive rounds ping-ponging purely on taste and routes them to a human
to reclassify. The gate sharpens concision without letting subjective taste loop
forever or mask a real defect. Use it when you specifically want tightness
enforced, for a spec that keeps bloating or code that keeps over-abstracting.
Leave it off for normal runs.

## 7. What it actually caught, and why that is quality

These are real runs from building Hydrate. They are the reason the engine exists
in the shape it does, and they are the clearest evidence that the value is
higher-quality work, not just a tidier process.

**A security blocker stopped before it reached `main`.** The cross-family audit
on a Peernet identity and authority build flagged a revocation-bypass blocker
plus two high-severity siblings that the implementation pass had missed. A
genuine auth bypass never merged. A single-pass self-review would have shipped
it: the diff looked right, and the cross-family reviewer attacked the threat
model rather than the code's appearance.

**The engine caught a false green in its own critic.** While dogfooding Design
mode we found an empty critic artefact being read as "zero objections, so
converged", a silent rubber-stamp. We fixed it fail-closed, so a failed or empty
critic now goes to `needs_human`, never to false convergence. The worst failure
mode of an automated reviewer, passing because it did not actually run, is now
impossible.

**Graceful degradation instead of a silent stall.** The design critic preferred
one model; when that model was withdrawn, the prober checked only that the
binary existed, not that the model resolved, so it bound and then died at
runtime, producing empty rounds. We fixed the prober to treat withdrawn models
as unavailable and the resolver to fall back to another family. A roster gap now
degrades loudly and keeps working instead of wedging.

**Convergence discipline produced specs that survived pressure.** Several build
specs each converged over roughly ten or eleven critic rounds, with about
twenty-four objections and zero recurrences. Each one shipped as a build spec
that had already absorbed two dozen adversarial objections before a line of code
existed. Zero recurrence is the signal that the disagreement was actually
resolved rather than papered over.

**Over-scoping is never silent.** An audit run came back with a SHIP verdict, but
its mandatory scope line read `changedFiles: 0` against a stale base. That
instantly exposed a harness bug that had pointed it at the wrong tree, so it had
reviewed stray working-tree edits rather than the intended change. The design
choice that the scope of a review is always reported turned a false SHIP into an
obvious re-run instead of a bad merge.

**Adversarial verification beats a confident scout.** An exploration agent once
reported, with confidence, that a struct field did not exist. It did. Building on
that claim would have meant re-adding a field and dropping a working parse path.
This is exactly why every finding is verified against the real diff before it is
reported. Trust the verified verdict, not the first plausible claim.

The through-line: orchestration's value is not "the model writes the code". It is
the second, independent, sceptical pass that a single session structurally
cannot give itself, plus gates that refuse to lie about being done. Better work,
and trustworthy work.

## 8. Operational notes (learned the hard way)

- Fail-closed is a feature. An empty or failed critic, an unavailable verifier,
  or a round cap reached without convergence all go to `needs_human`, never a
  false pass. Do not "fix" a `needs_human` by forcing sign-off; read why it
  stopped.
- Cross-family needs a healthy second family. Rate limits and withdrawn models
  have both wedged runs. Configure a fallback family. If your cross-family
  reviewer is rate-limited, a same-family reviewer is a partial stand-in but
  loses the independence that catches the most.
- Do not double-drive one session. Two terminals on one `session_id` race the
  same compare-and-set. One driver per session.
- Poll, do not have faith. A `design_round` call can time out client-side while
  the round completes server-side. Poll the `_status` tool.
- Build off `origin/main` in a clean worktree. Concurrent runs can move your
  local `main` underneath you; check `git branch --show-current` and the
  resolved base right before you commit, and check the review's scope line
  matches what you intended.
- Distil interactive sessions before `/clear`, but never auto-distil fleet
  workers. Their artefacts are the record; distilling them just bloats canon.

## 9. The short version

Use Design mode for decisions that are expensive to get wrong, and Develop mode
for parallelisable work with a real acceptance test. Drive it through the
`design_*` and `develop_*` MCP tools, one driver per session, polling status.
Trust convergence and the verified verdict, and treat `needs_human` as the
engine being honest. The payoff is quality you can defend and autonomy you can
leave running: a sceptical, cross-family second pass, with gates that will not
loop forever and will not claim to be done when they are not.
