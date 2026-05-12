# Install Hydrate

Hydrate runs as a local daemon plus a Claude Code hook pair plus an
MCP server. The install script wires all three. This guide covers
the supported platforms, the MCP snippets for non-Claude-Code
clients, and the troubleshooting paths if something is off.

The canonical install page is
[gethydrate.dev/install](https://gethydrate.dev/install) — the
snippets here match that page; if they ever drift, the website
wins.

## macOS

### Recommended: the install script

```sh
curl -fsSL gethydrate.dev/install | sh
```

The script:

1. Detects `darwin/arm64` vs `darwin/amd64`.
2. Downloads the latest tarball from
   [Releases](https://github.com/getHydrate/hydrate-public/releases).
3. Installs five binaries to `~/.local/bin/`:
   - `hydrate` — management CLI
   - `hydrate-server` — local daemon (HTTP API + SSE dashboard)
   - `hydrate-mcp` — MCP server for Claude Desktop, Cursor, Cline, Zed, Gemini CLI
   - `claude-context` — Claude Code `UserPromptSubmit` hook
   - `claude-capture` — Claude Code `Stop` hook
4. Adds the two hooks to `~/.claude/settings.json` (idempotent).
5. Starts `hydrate-server` in the background and writes its port +
   API key to `~/.hydrate/server.{port,key}`.

Confirm with:

```sh
hydrate doctor
```

`doctor` runs 17 checks covering PATH, hook registration, daemon
liveness, key file permissions, MCP wiring, and Claude Code
settings. Anything red has a one-line fix beside it.

### Homebrew (closed beta)

A Homebrew tap will be public at launch. Until then it requires a
beta-tester token; if you're in the closed beta you already have
the steps. The `curl … | sh` path above is the supported path for
everyone else.

## Linux

```sh
curl -fsSL gethydrate.dev/install | sh
```

Same script as macOS — detects `linux/arm64` vs `linux/amd64`,
puts binaries in `~/.local/bin/`. Make sure `~/.local/bin` is on
your `PATH`.

If your distro doesn't have `curl`, fetch the tarball manually
from the Releases page, extract it, and run:

```sh
./install.sh
```

inside the extracted directory. The script is the same one the
hosted install URL serves.

## Windows

A signed MSI installer is published per release on the
[Releases](https://github.com/getHydrate/hydrate-public/releases)
page. Download `hydrate-<version>-windows-amd64.msi`, double-click,
and accept the elevation prompt. The installer drops the binaries
under `%LOCALAPPDATA%\Hydrate\bin\`, adds them to `%PATH%`, and
writes the Claude Code hook entries.

After install, open PowerShell:

```powershell
hydrate doctor
```

…and follow any fix-up hints.

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

The DB is a single SQLite file at `~/.hydrate/hydrate.db` — back
it up the same way you'd back up any other file.
