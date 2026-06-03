---
name: serena-setup
description: >
  Set up and verify Serena MCP server integration with Claude Code. Use when the
  user wants to configure Serena, check if it is properly installed, verify hooks
  are active, or troubleshoot MCP connection issues. Also use when asked to
  "verify serena setup", "check serena", or "setup serena".
---

# Serena Setup & Verification

Run a complete setup verification for the Serena Claude Code plugin and print a status report. Follow these steps in order:

## Step 1 — Check Serena CLI

Run `serena --version` in the terminal.

- **✓ Found:** Note the version and continue.
- **✗ Not found:** Print install instructions and stop:

  ```sh
  uv tool install -p 3.13 serena-agent
  serena init
  ```

  After installing, ask the user to restart Claude Code (the plugin's MCP server needs a fresh start) and run `/serena:serena-setup` again.

## Step 2 — Check Serena initialization

Check whether `~/.serena/serena_config.yml` exists.

- **✓ Found:** Serena has been initialized. Continue.
- **✗ Missing:** Run `serena init` and continue.

## Step 3 — Check MCP connection

Run `/mcp` or ask the user to check it. The plugin's `.mcp.json` should have registered `serena` automatically.

- **✓ `serena` appears as connected:** MCP is working. Continue.
- **✗ Not connected or missing:** Try `/reload-plugins` first. If still missing, run manually:

  ```sh
  claude mcp add --scope user serena -- serena start-mcp-server --context=claude-code --project-from-cwd
  ```

  Then restart Claude Code.

## Step 4 — Check hooks

Run `/hooks` or note the hooks visible in this session.

- **✓ Four `serena-hooks` entries visible** (activate, remind, auto-approve, cleanup): Hooks are active.
- **✗ Missing hooks:** Try `/reload-plugins`. If still missing, verify the plugin is enabled with `claude plugin list`.

## Step 5 — Explain the system prompt override

Print this information for the user regardless of other check results:

---

**Recommended: system prompt override**

Recent Claude Code + Opus model updates introduced a strong bias toward built-in tools (grep, Edit, Read) over MCP tools. The `remind` hook helps, but the most effective fix is to start Claude Code with a custom system prompt:

```sh
claude --system-prompt="$(serena prompts print-cc-system-prompt-override)"
```

Run that command once to preview the override content. To make it permanent, consider creating a shell alias:

```sh
alias claude-serena='claude --system-prompt="$(serena prompts print-cc-system-prompt-override)"'
```

You can also add the prompt content to your project's `CLAUDE.md`, but the `--system-prompt` flag takes effect more reliably.

---

## Step 6 — Print summary

Print a clear status table:

```
Serena Setup Status
───────────────────────────────────────────
✓/✗  serena CLI installed       (version x.y.z or MISSING)
✓/✗  serena initialized         (~/.serena/serena_config.yml exists)
✓/✗  MCP server connected       (serena in /mcp output)
✓/✗  Hooks active               (4 serena-hooks entries in /hooks)
───────────────────────────────────────────
```

If all checks pass, add:

> Serena is fully configured. For best results with Opus models, use the system prompt override shown above.

If any check failed, list the specific steps the user needs to take.
