---
name: mailgun-setup
description: Configure Mailgun MCP credentials and tool scoping
---

# Mailgun Setup

Guide the user through configuring Mailgun credentials for this platform.

Ask for:

- Whether they use Mailgun's US or EU region.
- Whether they want all MCP tools or a narrower tag set.

Explain:

- Claude prompts for the Mailgun API key and region when the plugin is enabled. The key is stored in the system credential store.
- To change either value, run `claude plugin configure mailgun-claude@mailgun`, or use `/config` for the region.
- The packaged MCP config omits `MAILGUN_MCP_TAGS`, so the MCP server exposes all tags.
- For narrowed tools, the user should add `MAILGUN_MCP_TAGS` to the installed plugin's `.mcp.json`.

Do not ask the user to paste the secret into chat.
