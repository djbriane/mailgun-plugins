# Mailgun Claude Plugin

This package installs Mailgun workflows into Claude Code or Claude compatible plugin hosts.

## Includes

- MCP server config: `.mcp.json`
- Skills: `skills/`
- Commands: `commands/`
- Manifest: `.claude-plugin/plugin.json`

## Install

Claude Code:

```bash
claude plugin marketplace add mailgun/mailgun-plugins
claude plugin install mailgun-claude@mailgun
```

Claude desktop: download `mailgun-claude-<version>.zip` from the [latest release](https://github.com/mailgun/mailgun-plugins/releases/latest) and choose it on the **Upload a plugin** screen.

## Configure

Claude asks for two values when the plugin is enabled:

- Mailgun API key: your private API key. Claude stores it in the system credential store, not in `settings.json`.
- Mailgun region: `us` (default) or `eu`.

To change them later, run `claude plugin configure mailgun-claude@mailgun`, or use `/config` for the region. The values come from Claude rather than the shell, so the Claude desktop app doesn't need `MAILGUN_API_KEY` exported. The region picker requires Claude Code 2.1.271 or later.

The packaged config exposes all Mailgun tool tags. For scoped tool sets, add `MAILGUN_MCP_TAGS` to `.mcp.json`.
