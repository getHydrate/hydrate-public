---
type: Reference
title: Cross-runtime integration
description: How Hydrate reaches each coding-agent runtime over three channels (inject hook, capture hook, MCP) and which ones actually deliver context to the model.
timestamp: 2026-06-21T00:00:00Z
hydrate:
  tier: canon
  confidence: 1.0
---

# Cross-runtime integration

Hydrate reaches a runtime over three independent channels. An inject hook
pushes remembered context into the model at session start. A capture hook
ingests the transcript when a session ends. MCP lets the model pull memory on
demand. They are independent, and a hook that fires is not the same as one the
host delivers to the model.

Auto-recall injection delivers on Claude Code and Codex (and the Mistral Vibe
fork). On Cursor, Antigravity and Copilot the host does not surface the inject
hook to the model, so the read path runs over MCP while capture still works.
IBM Bob is in development. The honest position is that the capture (write) path
works broadly, while auto-inject only delivers on Claude Code and Codex; the
project marks its own gaps rather than claim every runtime is fully supported.
