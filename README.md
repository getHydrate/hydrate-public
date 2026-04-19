# hydrate-public

Public distribution channel for [Hydrate](https://gethydrate.dev) — a
local-first persistent memory layer for Claude Code.

## What's in this repo

- **Release tarballs** under [Releases](https://github.com/getHydrate/hydrate-public/releases),
  one tarball per platform (`linux/arm64`, `linux/amd64`, `darwin/arm64`,
  `darwin/amd64`). Each tarball contains five statically-linked binaries:
  - `hydrate` — the management CLI
  - `hydrate-server` — the local daemon
  - `hydrate-mcp` — MCP server for Cursor / Cline / Zed / Gemini CLI
  - `claude-context` — UserPromptSubmit hook
  - `claude-capture` — Stop hook
- **Issue tracker** — file bug reports at
  [/issues](https://github.com/getHydrate/hydrate-public/issues).
  The fastest route is `hydrate doctor --report`, which pre-fills a
  GitHub issue URL with your system info and the diagnostic output.

## What's *not* in this repo

- **Source code.** Hydrate is proprietary. This repo only distributes
  compiled binaries. If you're looking for the source, you're in the
  wrong place.

## Install

```
curl -fsSL gethydrate.dev/install | sh
```

The install script detects your OS + architecture, downloads the right
tarball from this repo's Releases page, installs the five binaries to
`~/.local/bin/`, wires the Claude Code hooks into `~/.claude/settings.json`,
and starts the local daemon.

## Uninstall

If you installed from source:

```
cd /path/to/hydrate && bash install.sh --uninstall --purge
```

If you installed via `curl gethydrate.dev/install | sh`, an uninstall
command will ship in a subsequent release. For now, remove binaries by
hand:

```
rm ~/.local/bin/{hydrate,hydrate-server,hydrate-mcp,claude-context,claude-capture}
rm -rf ~/.hydrate/
# And edit ~/.claude/settings.json to remove the claude-context / claude-capture hook entries.
```

## Links

- Homepage: https://gethydrate.dev
- Docs: https://gethydrate.dev/docs
- Benchmarks: https://gethydrate.dev/benchmarks
- Pricing: https://gethydrate.dev/pricing
- Compare against other memory products: https://gethydrate.dev/compare

## Licence

Proprietary. Use of the released binaries is governed by the Hydrate
end-user licence, which is shipped inside each tarball. You may install
and use the binaries on your own machines under the terms of that
licence. You may not redistribute, reverse-engineer, or derive works
from them.
