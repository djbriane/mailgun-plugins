# Platform Support

This repo supports Cursor, Claude Code / Claude compatible plugins, Gemini CLI extensions, and Codex plugins.

| Feature | Cursor | Claude Code / Claude compatible | Gemini CLI | Codex |
| --- | --- | --- | --- | --- |
| Root marketplace | `.cursor-plugin/marketplace.json` | `.claude-plugin/marketplace.json` | Not used by Gemini CLI | `.agents/plugins/marketplace.json` |
| Plugin manifest | `plugins/mailgun-cursor/.cursor-plugin/plugin.json` | `plugins/mailgun-claude/.claude-plugin/plugin.json` | `plugins/mailgun-gemini/gemini-extension.json` | `plugins/mailgun-codex/.codex-plugin/plugin.json` |
| MCP config | `mcp.json` | `.mcp.json` | `mcpServers` in `gemini-extension.json` | `.mcp.json` with `env_vars` |
| Skills | Native `skills/` directory | Native `skills/` directory | `skills/` directory included with extension | Native `skills/` directory |
| Rules | Native `rules/*.mdc` | Included as guidance files where supported | No direct equivalent | Shipped as the `mailgun-operational-safety` skill |
| Commands | Markdown commands listed in plugin manifest | Markdown commands listed in plugin manifest | TOML commands under `commands/mailgun/` | Shipped as skills from `shared/commands/codex` |
| Shared content | Generated from `shared/` | Generated from `shared/` | Generated from `shared/` | Generated from `shared/` |
| Release archive | `mailgun-cursor-<version>.zip`, unzip into a local plugin folder | `mailgun-claude-<version>.zip`, upload to Claude desktop | `darwin.`/`linux.`/`win32.mailgun-gemini.zip`, installed by Gemini CLI from the GitHub Release | `mailgun-codex-<version>.zip`, unzip and add as a local marketplace |
| Full support | Yes | Yes, with platform differences | Yes, with platform differences | Yes, with platform differences |

Every tagged release attaches these archives to its [GitHub Release](https://github.com/mailgun/mailgun-plugins/releases), with a `SHA256SUMS` file. Each archive has its manifest at the root. See [release-process.md](release-process.md#release-assets) for what each contains.

## Cursor

Cursor has the richest package in this repo. It includes marketplace metadata, a plugin manifest, MCP wiring, skills, rules, and commands.

Install from the Cursor Marketplace, or add `https://github.com/mailgun/mailgun-plugins` as a marketplace in Customize (or with `agent plugin marketplace add`) and install `mailgun-cursor`.

Cursor doesn't install zip files. To install a release without a marketplace, unzip `mailgun-cursor-<version>.zip` into `~/.cursor/plugins/local/mailgun-cursor`, then run **Developer: Reload Window**. Customize's **Import from Disk** and `agent --plugin-dir <folder>` also take the unzipped folder. A marketplace install of the same plugin name takes precedence over the local copy, and Enterprise admins can turn local plugin imports off.

## Claude Code

The Claude package includes a root marketplace, plugin manifest, `.mcp.json`, skills, and commands. The release archive leaves out `rules/`, which Claude doesn't load.

Claude Code CLI:

```bash
claude plugin marketplace add mailgun/mailgun-plugins
claude plugin install mailgun-claude@mailgun
```

Claude desktop: download `mailgun-claude-<version>.zip` from the GitHub Release and choose it on the **Upload a plugin** screen. The desktop app doesn't read shell profile exports, so `MAILGUN_API_KEY` has to be set for GUI apps (on macOS, `launchctl setenv MAILGUN_API_KEY "key-..."`, then restart the app).

## Gemini CLI

Gemini uses a single extension manifest with MCP server settings. The source package lives under `plugins/mailgun-gemini`, but Gemini requires `gemini-extension.json` at the extension root. `npm run build:gemini` stages `dist/gemini` and its release archives, and the `Publish Gemini Extension Branch` workflow publishes `dist/gemini` to the `gemini-extension` branch.

Install the latest GitHub Release, which checks for updates against the latest release:

```bash
gemini extensions install https://github.com/mailgun/mailgun-plugins
```

Pin a release with `--ref v<version>`. Gemini finds the archive through the `darwin.`, `linux.`, and `win32.` asset names; without them it would fall back to the source tarball, which has no manifest at its root.

Or track the `gemini-extension` branch, which moves on every Gemini-related change to `main` and on every release. Gemini first looks for a release tagged `gemini-extension`, then asks to fall back to `git clone`; answer yes, or pass `--consent`:

```bash
gemini extensions install https://github.com/mailgun/mailgun-plugins --ref gemini-extension
```

## Codex

Codex reads the repo marketplace at `.agents/plugins/marketplace.json` and the manifest at `.codex-plugin/plugin.json`. Codex plugins have no commands or rules, so `shared/commands/codex` holds those workflows as skills, and `npm run sync` merges them with the shared skills.

Codex passes only listed variables to stdio MCP servers. The `.mcp.json` uses `env_vars` to forward `PATH`, `HOME`, `MAILGUN_API_KEY`, `MAILGUN_API_REGION`, and `MAILGUN_MCP_TAGS` from the environment Codex was started in. The desktop apps launched from the Dock or Finder do not see shell profile exports, so macOS desktop users set the key with `launchctl setenv MAILGUN_API_KEY "key-..."` and restart the app.

Install from GitHub:

```bash
codex plugin marketplace add mailgun/mailgun-plugins
codex plugin add mailgun-codex@mailgun
```

Codex installs only from a marketplace, a local folder or a Git repo, never from a zip file or download URL. The release archive bundles a one-plugin marketplace, so it installs after unzipping:

```bash
unzip mailgun-codex-<version>.zip -d mailgun-codex
codex plugin marketplace add ./mailgun-codex
codex plugin add mailgun-codex@mailgun
```

Keep the unzipped folder: Codex records it as the marketplace source. The bundled marketplace is also named `mailgun`, so adding it replaces a `mailgun` marketplace added from GitHub, and the reverse; the plugin keeps the ID `mailgun-codex@mailgun` either way. Codex copies the plugin to `~/.codex/plugins/cache/mailgun/mailgun-codex/<version>/`.

## Known Gaps

- Marketplace publication steps are platform controlled and intentionally documented rather than automated here.
- The Gemini CLI marketplace and distribution flow can differ by organization; this repo publishes a dedicated extension branch for GitHub installs.
- The OpenAI plugin directory requires a remote HTTPS MCP server (OAuth 2.1 for user data) for public listing. Local stdio servers are limited to allowlisted partners. Until a hosted Mailgun MCP exists, the Codex package installs from GitHub only.
- No platform package includes Mailgun API business logic. Any tool addition must happen in `mailgun-mcp-server` first.
