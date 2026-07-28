# Install Sarvam MCP in any harness

Server runs locally over **stdio** — the client spawns it. Nothing to host.

## Before you start

1. API key from [dashboard.sarvam.ai/key-management](https://dashboard.sarvam.ai/key-management) (`sk_...`).
2. Run method:

| Method | Config | Prerequisite |
|--------|--------|--------------|
| **uvx** (recommended) | `"command": "uvx", "args": ["sarvam-mcp"]` | [uv](https://docs.astral.sh/uv/) |
| **pip** | `"command": "sarvam-mcp"` | Python 3.11+, `pip install sarvam-mcp` |

3. Auth — either config `env.SARVAM_API_KEY` **or** once in `~/.sarvam/credentials`:

```ini
api_key = sk_...
```

Env var wins if both are set. With the credentials file you can omit `env` from JSON.

Canonical package docs: [github.com/sarvamai/sarvam-mcp](https://github.com/sarvamai/sarvam-mcp) → `docs/INSTALLATION.md`.

## Shared JSON shape

Most clients use:

```json
{
  "mcpServers": {
    "sarvam": {
      "command": "uvx",
      "args": ["sarvam-mcp"],
      "env": { "SARVAM_API_KEY": "sk_..." }
    }
  }
}
```

## Cursor

- One-click: [Add to Cursor](https://cursor.com/install-mcp?name=sarvam&config=eyJjb21tYW5kIjoidXZ4IiwiYXJncyI6WyJzYXJ2YW0tbWNwIl19)
- Manual: `~/.cursor/mcp.json` or project `.cursor/mcp.json` (shape above)
- Verify: Settings → MCP → **sarvam** green + tool list

## Claude Code

```bash
claude mcp add --scope user sarvam --env SARVAM_API_KEY=sk_... -- uvx sarvam-mcp
```

Verify: `/mcp` shows **sarvam** connected.

## Claude Desktop

- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

Same `mcpServers` block. Fully quit and reopen the app.

## VS Code (Copilot)

```bash
code --add-mcp '{"name":"sarvam","command":"uvx","args":["sarvam-mcp"],"env":{"SARVAM_API_KEY":"sk_..."}}'
```

Or `.vscode/mcp.json` with top-level key **`servers`** (not `mcpServers`):

```json
{
  "servers": {
    "sarvam": {
      "command": "uvx",
      "args": ["sarvam-mcp"],
      "env": { "SARVAM_API_KEY": "sk_..." }
    }
  }
}
```

## Windsurf

`~/.codeium/windsurf/mcp_config.json` — same `mcpServers` block → Refresh in Cascade MCP panel.

## Zed

`settings.json`:

```json
{
  "context_servers": {
    "sarvam": {
      "command": {
        "path": "uvx",
        "args": ["sarvam-mcp"],
        "env": { "SARVAM_API_KEY": "sk_..." }
      }
    }
  }
}
```

## Codex CLI

```bash
codex mcp add sarvam --env SARVAM_API_KEY=sk_... -- uvx sarvam-mcp
```

Or `~/.codex/config.toml`:

```toml
[mcp_servers.sarvam]
command = "uvx"
args = ["sarvam-mcp"]
env = { SARVAM_API_KEY = "sk_..." }
```

## Gemini CLI

```bash
gemini mcp add sarvam -e SARVAM_API_KEY=sk_... uvx sarvam-mcp
```

## Cline / Roo Code

Extension MCP settings → same `mcpServers` JSON as Cursor.

## Continue

`~/.continue/config.yaml`:

```yaml
mcpServers:
  - name: sarvam
    command: uvx
    args:
      - sarvam-mcp
    env:
      SARVAM_API_KEY: sk_...
```

## LM Studio

Program → Install → Edit `mcp.json` → same `mcpServers` block → enable server.

## Verify

Ask: *Translate "good morning" to Hindi using Sarvam.*

Or outside a client:

```bash
npx @modelcontextprotocol/inspector uvx sarvam-mcp
```

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `uvx` / `sarvam-mcp` not found | Install uv, or use absolute path from `pip show -f sarvam-mcp`; restart client for PATH |
| Auth errors | Check `SARVAM_API_KEY` or `~/.sarvam/credentials`; call `sarvam_tools_set_api_key` |
| Server "hangs" in a terminal | Expected for stdio — let the client spawn it |
| Slow first `uvx` call | Cold download; pre-warm: `uv tool install sarvam-mcp` |
| Stale package | `uvx sarvam-mcp@latest` or `pip install -U sarvam-mcp` |
