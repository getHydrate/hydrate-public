# Roadmap

Hydrate ships weekly. This is a tease, not a commitment; the
ordering is reactive to what beta users surface.

## Now

Hydrate has become a platform layer: memory, orchestration
(adversarial multi-agent, MCP-driven), token reduction and
cross-agent coordination via Peernet.

- **Cross-runtime support.** The memory layer runs across
  Claude Code, Codex, Cursor, Antigravity, the Mistral Vibe fork
  and GitHub Copilot. IBM Bob is in development.
- **Orchestration.** Adversarial multi-agent workflows, driven
  through MCP, coordinate work across agents.
- **Peernet.** Opt-in, local-first agent-to-agent relay. Same-
  machine relay works today; cross-device relay is on the
  roadmap.
- **Dashboard SSE-everywhere.** Every dashboard pane (sessions,
  facts, working memory, dreams, packs, copilot, MCP-recent,
  retrievals, overview, project detail, orchestration, fatigue,
  displacement) pushes updates live; no manual refresh.
- **Re-imagined homepage.** The dashboard's `/` view answers
  "what's happening right now and where do I look next": five
  KPI cards, project activity ribbon, memory pulse, recent
  timeline, status footer.
- **Fatigue signals.** The status line shows a context-utilisation
  banner with a `distill now!` nudge when the current session is
  approaching the auto-compact threshold.
- **Goal nodes.** Long-lived statements of intent that surface
  ahead of tactical state on `/hydrate-last`.
- **Pre-hook self-check.** Hot-path hooks fail-safe with a one-
  line diagnostic log when the daemon is unreachable, so a hung
  daemon never blocks a Claude Code session.

## Up next

- **Multi-vendor MCP** — same memory surface across more clients;
  already works in Cursor / Cline / Zed / Windsurf;
  more landing as their MCP support matures.
- **Cross-machine sync** — git-backed personal sync between your
  laptop and your desktop, already in Pro; team sync expanding.
- **Cross-device Peernet** — extending the local-first relay
  beyond a single machine to agents on your other devices.
- **PreToolUse re-inject** — narrower context refresh right
  before a tool call, complementary to the per-prompt injection.
- **Compress survival** — better behaviour when Claude Code
  auto-compacts mid-session; the `claude-precompact` hook is
  shipped, ongoing tuning.

## Further out

- Plug-in MCP tools beyond memory. Doctor and pack tooling
  are already callable; the surface expands as MCP clients
  mature.
- An offline-only mode for environments where even
  `gethydrate.dev`'s update check is undesirable.

## What we're NOT doing

- No cloud-hosted Hydrate service. The architecture is local-
  first by design.
- No telemetry that leaves your machine without you opting in.
- No vendor lock-in. Hydrate works with whichever LLM provider
  you bring; your facts are in plain SQLite on your disk.

Feedback on priorities welcome at
[/discussions](https://github.com/getHydrate/hydrate-public/discussions)
or `hello@gethydrate.dev`.
