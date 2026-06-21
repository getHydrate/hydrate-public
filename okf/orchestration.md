---
type: Reference
title: Orchestration
description: Hydrate's adversarial, cross-family multi-agent engine that raises the quality of agent work through structured disagreement and fail-closed gates.
timestamp: 2026-06-21T00:00:00Z
hydrate:
  tier: canon
  confidence: 1.0
---

# Orchestration

Orchestration replaces "one model, one pass" with a persisted, adversarial
loop. A generator produces an artefact, an independent cross-family critic
attacks it, the disagreement is tracked as a ledger of objections, and the loop
converges only when the material objections clear or escalates to a human when
it cannot. Every transition is a pure state machine, and all state is persisted
to SQLite, so a run is resumable and status-pollable. It is driven over MCP
tools, so any connected runtime can run it.

Design mode pressure-tests a plan or spec; Develop mode implements code across
parallel units with a post-merge audit; Image and Studio reuse the same loop
for visual and UX work. The value is two-sided: better work, because a
cross-family critic catches what the author cannot see, and trustworthy work,
because the gates will not claim the job is done when it is not.
