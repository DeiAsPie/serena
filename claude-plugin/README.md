# Serena — Claude Code Plugin

Semantic code intelligence for Claude Code. Installs the [Serena MCP server](https://github.com/oraios/serena) and wires up the recommended hooks so the agent uses symbolic tools instead of grep/read\_file.

**What the plugin does automatically:**

- Starts `serena start-mcp-server` when Claude Code launches (MCP auto-config)
- Activates your project at session start (`serena-hooks activate`)
- Reminds the agent to use Serena's symbolic tools when it drifts (`serena-hooks remind`)
- Auto-approves Serena tool calls in permissive modes (`serena-hooks auto-approve`)
- Cleans up session state on exit (`serena-hooks cleanup`)

## Prerequisites

Install and initialize Serena before enabling this plugin:

```sh
uv tool install -p 3.13 serena-agent
serena init
```

Verify the CLI is available:

```sh
serena --version
serena-hooks --help
```

> **PATH note:** `serena` and `serena-hooks` are installed to `~/.local/bin/` by `uv tool install`. If Claude Code's hook subprocesses can't find them, add `~/.local/bin` to your system PATH (not just your shell profile). On macOS, edit `/etc/paths` or use `/etc/paths.d/`.

## Installation

### From the Serena marketplace (recommended)

```sh
# 1. Add the Serena repo as a Claude plugin marketplace
claude plugin marketplace add oraios/serena

# 2. Install the plugin
claude plugin install serena@serena
```

Or inside Claude Code:

```
/plugin marketplace add oraios/serena
/plugin install serena@serena
```

Restart Claude Code after installing to activate the MCP server and hooks.

### Local (testing / development)

```sh
git clone https://github.com/oraios/serena
claude --plugin-dir ./serena/claude-plugin
```

## Verification

1. Run `/mcp` — `serena` should appear as connected
2. Run `/hooks` — four `serena-hooks` entries should appear
3. Run `/serena:serena-setup` — prints a full status report and any missing steps

## Recommended: system prompt override

Recent Claude Code + Opus model updates introduced a strong bias toward built-in
tools (grep, read\_file, Edit) over MCP tools. To counteract this, start Claude Code with:

```sh
claude --system-prompt="$(serena prompts print-cc-system-prompt-override)"
```

This is especially important when using Opus models. You can also add the output of
`serena prompts print-cc-system-prompt-override` to your project's `CLAUDE.md`,
though the `--system-prompt` flag is more effective.

## How it works

| Hook | Event | What it does |
|------|-------|--------------|
| `activate` | `SessionStart` | Prompts Claude to call `activate_project` and read Serena instructions |
| `remind` | `PreToolUse` (all tools) | Denies call after 3+ consecutive grep/read without a Serena tool; rate-limited to 1 nudge/2 min |
| `auto-approve` | `PreToolUse` (`mcp__serena__*`) | Auto-approves Serena tools in `acceptEdits`/`auto` permission mode |
| `cleanup` | `SessionEnd` | Removes session state from `~/.serena/hook_data/{session_id}/` |

## MCP context

The plugin starts Serena with `--context=claude-code`, which:

- Disables tools Claude Code already has (`read_file`, `find_file`, `list_dir`, `execute_shell_command`, `search_for_pattern`, `create_text_file`)
- Sets single-project mode (`activate_project` is called once per session)
- Instructs the agent to prefer symbolic tools over line-based operations

## Troubleshooting

**Serena not starting:** Run `serena --version` in a terminal. If not found, install it. If found but Claude Code can't start it, check that `~/.local/bin` is in your system PATH.

**Hooks not firing:** Run `/hooks` in Claude Code. If Serena hooks are missing, try `/reload-plugins`. If still missing, verify the plugin is enabled with `claude plugin list`.

**MCP timeout:** If Serena takes too long to start (language server init), set in your shell profile:
```sh
export MCP_TIMEOUT=60000
```

**Agent ignoring Serena tools:** Use the system prompt override (see above). For long sessions, the `remind` hook helps, but the override is the most effective fix.

## Links

- [Serena documentation](https://oraios.github.io/serena/)
- [Claude Code client setup](https://oraios.github.io/serena/02-usage/030_clients.html#claude-code)
- [GitHub repository](https://github.com/oraios/serena)
