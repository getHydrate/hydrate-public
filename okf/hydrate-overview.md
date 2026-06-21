---
type: Reference
title: Hydrate
description: A local-first platform layer that gives coding agents shared memory, orchestration, token reduction and cross-agent coordination.
timestamp: 2026-06-21T00:00:00Z
hydrate:
  tier: canon
  confidence: 1.0
---

# Hydrate

Hydrate is the platform layer for coding agents. One small local daemon gives
Claude Code, OpenAI Codex, Cursor, Mistral Vibe and Copilot four things they do
not have on their own: persistent memory across sessions, adversarial
multi-agent orchestration, measurable token reduction, and a way for sessions to
talk to each other over Peernet. Nothing leaves the machine unless the user
turns it on.

The durable memory is plain and portable. Canon (pinned authoritative facts),
semantic facts, session digests, the autonomous wiki and the handover archive
are all stored as text and can be exported as an OKF v0.1 bundle, so the memory
stays readable and forkable rather than locked in a proprietary store.
