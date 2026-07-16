# Hydrate — Claude Code plugin

Persistent local memory for Claude Code. Ships as a repo-marketplace plugin that
runs in every Claude Code surface: the CLI, the Claude Code panel inside the
desktop app, the IDE extensions, and web. Both memory legs — lifecycle **hooks**
and the **MCP** memory tools.

## What it does

| Leg | Mechanism |
|---|---|
| **hooks** | ambient inject before each prompt (`UserPromptSubmit`/`SessionStart`), capture on completion (`Stop`), and a compact-survival snapshot (`PreCompact`) — zero friction |
| **MCP** | explicit tools (`hydrate_recall`, `hydrate_save_fact`, `hydrate_session_resume`, …) + the `hydrate-memory` skill for deliberate use |

All storage is local (`~/.hydrate/data.db`); nothing leaves the machine.

## Layout (marketplace root)

The marketplace root is this directory: the catalog lives at
`.claude-plugin/marketplace.json` and the plugin is referenced relative to the
root as `./plugins/hydrate`.

```
packaging/claude-plugin/               <- marketplace root (publishes to hydrate-public)
├── .claude-plugin/marketplace.json    catalog → source "./plugins/hydrate"
└── plugins/hydrate/
    ├── .claude-plugin/plugin.json     manifest (name, version, component paths)
    ├── hooks/hooks.json               6 events → ${CLAUDE_PLUGIN_ROOT}/bin/hydrate-launch <shim>
    ├── .mcp.json                      ${CLAUDE_PLUGIN_ROOT}/bin/hydrate-launch --mcp
    ├── skills/hydrate-memory/         deliberate-use skill (namespaced /hydrate:hydrate-memory)
    └── bin/hydrate-launch             bootstrap: fetches per-platform binaries on first use
```

The six hook events map to the six `claude-*` shims:
`UserPromptSubmit`→claude-context, `Stop`→claude-capture,
`PreCompact`→claude-precompact, `SessionStart`→claude-session-start,
`PostToolUse`→claude-tool-post, `PreToolUse`→claude-tool-pre. `claude-tool-pre`
is observe-only (config-gated on `tool_guard.enabled`, default off) — it never
denies a tool call out of the box.

## Binary delivery

The ~40MB per-platform binaries are **not committed**. `bin/hydrate-launch`
fetches `hydrate-<ver>-<os>-<arch>.tar.gz` from `getHydrate/hydrate-public`
releases into `${CLAUDE_PLUGIN_DATA}` (falling back to `~/.hydrate/plugin/bin`) on
first use, then execs the real shim. The hook path is **fail-open** — a hook never
blocks a prompt; it installs in the background and succeeds on the next turn. The
MCP path blocks until the binary exists.

## Install

```
/plugin marketplace add getHydrate/hydrate-public
/plugin install hydrate@hydrate
```

Hooks arm at the next session-start boundary: run **`/reload-plugins`** (or start a
fresh session) after installing — a bare install does not arm them mid-session.
Plugin hooks are auto-trusted (no per-hook trust prompt).

Local dev against this working tree:

```
/plugin marketplace add ./packaging/claude-plugin
/plugin install hydrate@hydrate
# validate: claude plugin validate ./packaging/claude-plugin/plugins/hydrate
```

## What the plugin does NOT do (by design)

The plugin is the low-friction **onboarding** path, not a superset. It structurally
cannot:

- write `~/.claude/settings.json` `statusLine` (the Hydrate statusline),
- pre-approve the relay MCP tools in `permissions.allow`,
- un-namespace its skill — it is `/hydrate:hydrate-memory`, not a bare command,
- register the always-on managed daemon (it runs the daemon boot-on-use instead).

## Graduating to the full install

Because `bin/hydrate-launch` already downloads the full binary set, you can get the
full surface (statusline + the 14 `/hydrate-*` slash commands + the 3 agents +
relay pre-approval + the always-on managed daemon) **with no second download**:

```
hydrate install-hooks     # auto-graduates the plugin-owned runtime
# or, explicitly:
hydrate graduate
```

Graduation is transparent (it reports what it added) and reversible — uninstall the
full install and the plugin re-claims the runtime on its next hook fire. Pass
`hydrate install-hooks --keep-plugin claude-code` to keep the lean plugin owning
Claude Code instead.

## Repo topology

- **Dev repo:** `SeamusWaldron/hydrate` (private) — this working tree.
- **Public distribution repo:** `getHydrate/hydrate-public` (public) — hosts the
  release tarballs + the marketplace. On each non-prerelease tag, `release.yml`
  copies `packaging/claude-plugin/{.claude-plugin,plugins/hydrate}` to the public
  repo root, pins the version to the tag, and pushes.
