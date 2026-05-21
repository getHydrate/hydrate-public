# Install Hydrate

Hydrate runs as a local daemon plus a Claude Code hook pair plus an
MCP server. The install script wires all three. This guide covers
the supported platforms, the MCP snippets for non-Claude-Code
clients, and the troubleshooting paths if something is off.

The canonical install page is
[gethydrate.dev/install](https://gethydrate.dev/install) — the
snippets here match that page; if they ever drift, the website
wins.

The fastest path for any user is the **First-steps builder** at
[gethydrate.dev/install/first-steps](https://gethydrate.dev/install/first-steps).
Pick your settings in the form and it emits a single shell script
that activates your license, initialises your project, optionally
imports a hydration pack, optionally bootstraps a team-canon git
repo, and runs `hydrate doctor` at the end.

## macOS

### Homebrew (recommended)

```sh
brew tap getHydrate/hydrate
brew install hydrate
hydrate setup           # interactive first-run wizard
hydrate doctor          # 16-point health check
```

The tap pulls binaries from
[hydrate-public Releases](https://github.com/getHydrate/hydrate-public/releases/latest);
the formula handles arm64 vs amd64 automatically.

### Alternative: the install script

```sh
curl -fsSL gethydrate.dev/install | sh
```

The script:

1. Detects `darwin/arm64` vs `darwin/amd64`.
2. Downloads the latest tarball from
   [Releases](https://github.com/getHydrate/hydrate-public/releases).
3. Installs the binaries to `~/.local/bin/`:
   - `hydrate` — management CLI
   - `hydrate-server` — local daemon (HTTP API + SSE dashboard)
   - `hydrate-mcp` — MCP server for Claude Desktop, Cursor, Cline, Zed, Gemini CLI
   - `claude-context` — Claude Code `UserPromptSubmit` hook
   - `claude-capture` — Claude Code `Stop` hook
   - `claude-precompact`, `claude-session-start`, `claude-tool-{pre,post}` — additional hook shims
   - `codex-context`, `codex-capture`, `codex-session-start` — Codex equivalents
   - `vibe-*` — Mistral Vibe shims
4. Adds the hooks to `~/.claude/settings.json` (idempotent).
5. Starts `hydrate-server` in the background and writes its port +
   API key to `~/.hydrate/server.{port,key}`.

Confirm with:

```sh
hydrate doctor
```

`doctor` runs 16 checks covering PATH, hook registration, daemon
liveness, key file permissions, MCP wiring, and Claude Code
settings. Anything red has a one-line fix beside it.

### First Claude Code session in a project

The first time you open Claude Code inside a project that already
has a `CLAUDE.md`, Hydrate runs `hydrate dehydrate --mode=full`
once in the background to extract structured facts from the file
into the local store. The subprocess is detached — session start
is never blocked — and writes its log to
`~/.hydrate/logs/auto-prime.log`. A marker at
`~/.hydrate/auto-primed/<project-slug>` ensures it only runs once
per project; delete the marker to retry.

`CLAUDE.md` itself is **not modified** by the auto-prime. If you
also want to shrink the on-disk file, that's an opt-in CLI step:

```sh
hydrate dehydrate --apply --mode=summary   # readable summary + operational sections
hydrate dehydrate --apply --mode=stub      # 5-line pointer + operational sections
hydrate dehydrate --revert                 # restore CLAUDE.md.pre-hydrate.bak
```

See [`USAGE.md`](USAGE.md#claudemd-ingestion--hydrate-dehydrate)
for the full mode table and safety notes.


## Linux

### Homebrew on Linux

```sh
brew tap getHydrate/hydrate
brew install hydrate
hydrate setup
hydrate doctor
```

### Install script

```sh
curl -fsSL gethydrate.dev/install | sh
```

Detects `linux/arm64` vs `linux/amd64` and puts the binaries in
`~/.local/bin/`. Make sure `~/.local/bin` is on your `PATH`.

If your distro doesn't have `curl`, fetch the tarball manually
from [Releases](https://github.com/getHydrate/hydrate-public/releases/latest),
extract it, and copy `bin/*` into `~/.local/bin/`.

## Windows

Each release publishes a Windows ZIP at
[Releases](https://github.com/getHydrate/hydrate-public/releases/latest):

- `hydrate-v0.4.0-windows-amd64.zip` (Intel / AMD)
- `hydrate-v0.4.0-windows-arm64.zip` (ARM)

Unzip into a directory on `%PATH%` (for example
`%LOCALAPPDATA%\Hydrate\bin\`), then in PowerShell:

```powershell
hydrate setup     # interactive first-run wizard
hydrate doctor
```

A signed MSI installer is on the roadmap — track
[the MSI tracking issue](https://github.com/getHydrate/hydrate-public/issues)
for status. Until then the ZIP is the supported path.

WSL users can take either the Linux Homebrew or install-script
path instead — recommended if you want tmux-based orchestrations.

## MCP server setup (non-Claude-Code clients)

Hydrate ships a Model Context Protocol server (`hydrate-mcp`) so
the same facts surface in any MCP-capable client. Pick your tool:

### Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`
(macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "hydrate": {
      "command": "hydrate-mcp"
    }
  }
}
```

### Cursor

Settings → MCP → Add server:

- **Name:** `hydrate`
- **Command:** `hydrate-mcp`

### Cline

`~/.cline/config.json`:

```json
{
  "mcpServers": {
    "hydrate": {
      "command": "hydrate-mcp"
    }
  }
}
```

### Windsurf

Add to `~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "hydrate": {
      "command": "hydrate-mcp"
    }
  }
}
```

### Zed

`~/.config/zed/settings.json`:

```json
{
  "context_servers": {
    "hydrate": {
      "command": {
        "path": "hydrate-mcp"
      }
    }
  }
}
```

### Gemini CLI

`~/.gemini/mcp.json`:

```json
{
  "servers": {
    "hydrate": {
      "command": "hydrate-mcp"
    }
  }
}
```

`hydrate install-hooks` adds the Claude Desktop entry automatically
on macOS. For the other clients, paste the snippet above.

## Verification

```sh
hydrate doctor
```

Healthy output ends with:

```
17/17 checks passed. Hydrate is ready.
```

Anything else: each failing check prints a one-line `fix:` hint.
The most common ones:

| Symptom | Fix |
|---|---|
| `daemon not running` | `hydrate server start` |
| `hydrate not on PATH` | add `~/.local/bin` to your shell rc |
| `api.key permissions` | `chmod 600 ~/.hydrate/server.key` |
| `hooks not registered` | re-run `hydrate install-hooks` |
| `MCP not wired` | re-run `hydrate install-hooks --mcp` |

If `doctor` reports something that isn't on this list, run
`hydrate doctor --report` — it copies a pre-filled GitHub issue
URL to your clipboard with the full diagnostic output. Paste that
into a new issue and we'll triage.

## Uninstall

If you used the install script:

```sh
rm ~/.local/bin/{hydrate,hydrate-server,hydrate-mcp,claude-context,claude-capture}
rm -rf ~/.hydrate/
```

Then edit `~/.claude/settings.json` and remove the
`claude-context` / `claude-capture` hook entries.

A first-class `hydrate uninstall --purge` subcommand is in the
roadmap.

## Where things live

| Path | Purpose |
|---|---|
| `~/.local/bin/hydrate*` | binaries (macOS / Linux) |
| `%LOCALAPPDATA%\Hydrate\bin\` | binaries (Windows) |
| `~/.hydrate/` | DB, server port file, API key |
| `~/.claude/settings.json` | Claude Code hook registration |
| `~/.hydrate/hook-selfcheck.log` | hook-pre-flight failure log |
| `~/.hydrate/auto-primed/<slug>` | first-run dehydrate marker (one per project) |
| `~/.hydrate/logs/auto-prime.log` | first-run dehydrate subprocess log |
| `<project>/HYDRATE.md` | dehydrate ledger (created by `hydrate dehydrate --apply`) |
| `<project>/CLAUDE.md.pre-hydrate.bak` | safety copy written before any CLAUDE.md rewrite |

The DB is a single SQLite file at `~/.hydrate/hydrate.db` — back
it up the same way you'd back up any other file.
