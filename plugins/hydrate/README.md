# Hydrate — Codex/ChatGPT plugin

Persistent local memory for the coding agent. Ships as a repo-marketplace plugin.

## What it does per surface

| Surface | Mechanism |
|---|---|
| Codex mode, Work mode, Codex CLI, IDE | **hooks** → ambient inject (UserPromptSubmit/SessionStart) + capture (Stop) |
| Chat mode | no hooks — **MCP tools + the `hydrate-memory` skill** (deliberate memory) |

## Layout (marketplace root)

The marketplace root is the repo root: the manifest lives at `.agents/plugins/marketplace.json`
and plugins are referenced relative to the root as `./plugins/<name>`. In the dev repo this
whole root is staged under `packaging/codex-plugin/`:

```
packaging/codex-plugin/            <- marketplace root (staging; publishes to hydrate-public root)
├── .agents/plugins/marketplace.json   catalog → source.path "./plugins/hydrate"
├── plugins/hydrate/
│   ├── .codex-plugin/plugin.json      display fields nested under "interface"
│   ├── hooks/hooks.json               ${PLUGIN_ROOT}/bin/hydrate-launch <shim>
│   ├── .mcp.json                      ${PLUGIN_ROOT}/bin/hydrate-launch --mcp
│   ├── skills/hydrate-memory/         deliberate-use skill (load-bearing in Chat/Work)
│   └── bin/hydrate-launch             bootstrap: fetches per-platform binaries on first use
└── probe/                             dev-only hook-firing probe — NOT published
```

## Repo topology (where this gets published)

- **Dev repo:** `SeamusWaldron/hydrate` (PRIVATE) — this working tree. End users cannot reach it.
- **Public distribution repo:** `getHydrate/hydrate-public` (PUBLIC) — hosts the release
  tarballs + Homebrew tap. On each non-prerelease tag, `release.yml` copies
  `packaging/codex-plugin/{.agents/plugins,plugins/hydrate}` to this repo's **root**, pins the
  version to the tag, and commits/pushes (see `docs/RELEASE_PROCESS.md` step 8).

Both the marketplace `add` target and the binary download source are the **public** repo.

## Binary delivery

The ~40MB per-platform binaries are **not committed**. `bin/hydrate-launch` fetches
`hydrate-<ver>-<os>-<arch>.tar.gz` from `getHydrate/hydrate-public` releases into
`${PLUGIN_DATA}/bin` on first use, then execs the real shim. Hook path is **fail-open**
(never blocks a prompt; installs in the background and succeeds on the next turn). MCP path
blocks until the binary exists.

## Install (private / team — no OpenAI review)

```bash
codex plugin marketplace add getHydrate/hydrate-public   # after a release publishes the marketplace
codex plugin add hydrate@hydrate                          # or: /plugins in any Codex client → install
```

Local dev against this working tree (proven working — marketplace add → discover → add → hooks fire):

```bash
codex plugin marketplace add ./packaging/codex-plugin     # root contains .agents/plugins/marketplace.json
codex plugin add hydrate@hydrate
#   uninstall: codex plugin remove hydrate ; codex plugin marketplace remove hydrate
```

## Publish to the official directory (public, reviewed)

Submit at the plugin portal (`platform.openai.com/plugins` per submit-plugins docs):
verified identity + MCP validation + domain verification for the app. Optional — the
repo-marketplace path above needs none of it.

## Verified (local, 2026-07-12)

- `codex plugin marketplace add ./packaging/codex-plugin` → discover → `codex plugin add
  hydrate@hydrate` (installed/enabled/0.10.1) → a `codex exec` turn fired the plugin's
  `${PLUGIN_ROOT}/bin/hydrate-launch` hook, which bootstrapped binaries from
  `getHydrate/hydrate-public` into `~/.codex/plugins/data/hydrate-hydrate/bin/`. Full chain works.

## Still to confirm

- `${PLUGIN_ROOT}` in `.mcp.json` `command` (only the hooks path was exercised) — verify the
  MCP server launches; fall back to an absolute/PATH command if the var doesn't expand there.
- Whether **installing** the plugin auto-trusts its hooks (the local test used
  `--dangerously-bypass-hook-trust`). Likely yes since install is an explicit opt-in.
- Whether hooks fire in **Work** mode specifically (doc-stated, not yet tested).
