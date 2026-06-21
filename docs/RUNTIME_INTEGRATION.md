# How Hydrate integrates with a coding-agent runtime

Hydrate is a **cross-runtime** memory layer. The same local store feeds Claude
Code, OpenAI Codex, Cursor, Mistral Vibe, GitHub Copilot, and Antigravity. But
"supported" is not one bit per runtime — it's **three independent channels**, and
honesty about which channels actually reach the model is the whole point of this
page.

## The three channels

Every integration is some combination of these. They are independent: a runtime
can have a working capture channel and a dead inject channel at the same time.

| Channel | Direction | What it does |
|---|---|---|
| **Inject hook** | read (into the model) | On session start / before a prompt, Hydrate's hook emits `{"additionalContext": "..."}`. **The host decides whether to surface that text to the model.** Some hosts run the hook but never deliver its output. |
| **Capture hook** | write (out of the session) | On stop / session-end, Hydrate ingests the transcript into the local store. This is the robust leg — it fires and works across runtimes. |
| **MCP** | read (model-driven) | The model calls Hydrate's MCP tools (`hydrate_recall`, `hydrate_facts_list`, …) on demand. Works wherever the host loads an MCP server, **independent of the inject-hook gate.** |

**Net read path = inject hook OR MCP.** Where the inject hook delivers (Claude,
Codex), recall is automatic every session. Where the host doesn't deliver it
(Cursor, Antigravity), **MCP carries the read path** — model-initiated, not
guaranteed every turn. The capture (write) path is separate and works regardless.

### The trap: "the hook fired" ≠ "the model saw it"

A hook can run, emit a full context block, and the host can still drop it before
the model. So we do not infer delivery from the hook running, from the shim's
output, or from the agent's chat reply. **We verify delivery server-side.**

## How we verify a channel (and why it matters)

We **delivery-test** the inject channel with a **canary**:

1. Pin a distinctive marker into project canon.
2. Disable MCP for the runtime (so MCP can't answer instead).
3. Send a first-turn prompt asking the model to echo the marker, else say a fixed
   fallback.
4. Confirm **via server-side hook telemetry** that the hook actually fired with
   content for that turn, and which runtime it came from.

Model echoes the marker → the inject channel **delivers**. Model gives the
fallback while telemetry shows the hook fired with a full payload → the host
**ran the hook but didn't deliver it to the model**. This method is what
corrected our own record when an earlier "inject works here" claim turned out to
be the MCP channel doing the work.

Contrast a config-file unit test, which only proves *the config was written* —
not that any context reached the model. Both matter; only the first tells you the
feature works.

## Coverage — Hydrate, per channel

Honest by construction: where our own inject channel doesn't reach the model, it
is marked ⚠️/❌, not hidden behind a single "supported" tick.

| Runtime | Inject (read) | Capture (write) | MCP (read) | Net |
|---|:--:|:--:|:--:|---|
| **Claude Code** | ✅ delivers | ✅ | ✅ | **full auto-memory** |
| **OpenAI Codex** | ✅ delivers | ✅ | ✅ | **full auto-memory** |
| **Mistral Vibe** | ✅ delivers (fork) | ✅ | ✅ | **full (fork)** |
| **Cursor** | ⚠️ host-gated | ✅ | ⚠️ registered, recall unverified | capture + MCP |
| **Antigravity** | ❌ host doesn't deliver | ✅ | ✅ recall observed | capture + MCP |
| **GitHub Copilot** | ⚠️ no session-start inject | ✅ | ✅ | capture + MCP |
| **IBM Bob** | ❌ engine inert (v1.0.4) | ➖ not built | ✅ | in development |
| **Gemini CLI** | ➖ deprecated | ➖ deprecated | ➖ | deprecated |

<!-- HYDRATE-MATRIX:v1 -->

Legend: ✅ verified delivering · ⚠️ host-gated / partial · ❌ host runs the hook but doesn't deliver · ➖ not built / deprecated.

Notes:
- **Cursor** runs the inject hook but its host gate (`enable_hook_additional_context`)
  withholds the output from the model on the accounts we tested; capture is
  verified; MCP is registered but recall-on-Cursor is not separately confirmed.
- **Antigravity** runs the inject hook (verified: a full payload was injected on
  the turn) yet the model did not receive it; capture lands correctly and the
  model uses MCP for recall.
- **GitHub Copilot** is partial: the headless `copilot -p` path receives no
  session-start injection (the inject channel doesn't reach the model there), so
  the read path is MCP; the **capture channel works** and the MCP server is
  registered. Interactive Copilot may differ — only the headless path is
  confirmed today.
- **Gemini CLI** is deprecated (the individual free tier was sunset); the shared
  Gemini-engine adapter still backs Bob.
- **IBM Bob** is in development: its hook engine does not fire on v1.0.4, so
  neither inject nor a capture hook would run; only MCP works today.

## What `hydrate doctor` checks

`hydrate doctor` verifies each runtime to the granularity its host actually
allows — and **reports honestly when a host can't be verified** rather than
showing a false green. For hosts with no load-state introspection it reports
"config written; host-load unverifiable" instead of claiming a delivery it cannot
observe. Where `doctor` confirms config-written but cannot confirm
inject-delivered for a host, that is a **named known gap**, tracked separately —
the tool is never made to overstate what it can see.

## The honest summary

- **Auto-inject (read) only truly delivers on Claude + Codex** (and Vibe via the
  fork). For Cursor / Antigravity / Bob, the read path leans on **MCP**, because
  the host doesn't surface the inject hook to the model.
- The **capture (write) path works broadly** — Hydrate remembers your sessions on
  every listed runtime that fires a stop hook.
- We **mark our own gaps**. A matrix that ticks every box for itself is marketing,
  not engineering — and it's exactly the failure mode (config-written claimed as
  working) this page exists to avoid.
