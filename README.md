# Mailgun Plugins

Mailgun Plugins helps developers use Mailgun from agent tools like Cursor, Claude Code, Gemini CLI, and Codex. Each package connects the agent to [`@mailgun/mcp-server`](https://github.com/mailgun/mailgun-mcp-server) and adds practical workflows for common Mailgun tasks, including domain setup, delivery diagnostics, suppressions, template QA, and platform operations.

## Supported Platforms

| Platform | Package | Status | Notes |
| --- | --- | --- | --- |
| Cursor | `plugins/mailgun-cursor` | Full | Marketplace manifest, plugin manifest, MCP config, skills, rules, and commands. |
| Claude Code / Claude compatible | `plugins/mailgun-claude` | Supported | Claude marketplace/plugin manifest, MCP config, skills, and commands. Rules are included as guidance files where supported by the client. |
| Gemini CLI | `plugins/mailgun-gemini` | Supported | Gemini extension manifest, MCP server config, settings metadata, skills, and TOML commands. |
| Codex | `plugins/mailgun-codex` | Supported | Codex plugin manifest, MCP config, and skills. Commands and the safety rule ship as skills. |

See [docs/platform-support.md](docs/platform-support.md) for the detailed matrix.

## How It Works

- `shared/` is the canonical source for Mailgun skills, reusable rules, setup docs, example prompts, and examples.
- `plugins/mailgun-cursor`, `plugins/mailgun-claude`, `plugins/mailgun-gemini`, and `plugins/mailgun-codex` contain platform manifests and generated copies of shared content.
- `scripts/sync-shared.mjs` copies shared content into each platform package.
- `scripts/validate.mjs` checks manifests, referenced files, skill frontmatter, MCP version pinning, and required docs.
- CI runs sync drift checks and validation on every push and pull request.

## CI and Publication

GitHub Actions is enabled for this repository:

- `Validate` runs on pushes and pull requests to `main`, and builds every release archive so packaging breaks show up in review.
- `Release` runs when a `v*` tag is pushed. It validates the tag against the package version, runs the Claude and Gemini validators on the built archives, and creates a [GitHub Release](https://github.com/mailgun/mailgun-plugins/releases) with one archive per platform and a `SHA256SUMS` file.
- `Publish Gemini Extension Branch` builds `plugins/mailgun-gemini` and publishes it to the `gemini-extension` branch when Gemini package files change on `main`, and on every release.

## MCP Server Compatibility

This repo currently pins plugin MCP startup commands to:

```bash
npx -y @mailgun/mcp-server@2.3.1
```

Required environment:

- `MAILGUN_API_KEY`: Mailgun private API key.
- `MAILGUN_API_REGION`: optional MCP server setting, `us` or `eu`; the packaged configs omit it and the server defaults to `us`.
- `MAILGUN_MCP_TAGS`: optional MCP server setting for a comma separated list of products (`send`, `validate`, `optimize`, `inspect`), Send toolsets such as `send.templates`, `core`, or `dynamic`; the packaged configs omit it and expose all tools. See [Tags and Toolsets](shared/docs/mailgun-capabilities.md#tags-and-toolsets) for every value.

For EU accounts or scoped tool sets, edit the installed platform MCP config to add `MAILGUN_API_REGION` or `MAILGUN_MCP_TAGS`. The Codex package forwards both from the environment, so Codex users can export them instead.

## Install

Each [GitHub Release](https://github.com/mailgun/mailgun-plugins/releases) attaches an archive per platform: `mailgun-claude-<version>.zip`, `mailgun-cursor-<version>.zip`, `mailgun-codex-<version>.zip`, and `mailgun-gemini-<version>.zip`, plus `SHA256SUMS`. Marketplace installs are the easiest way to stay current; the archives are for hosts that take an upload and for pinned or offline installs.

### Cursor

Install from the Cursor plugin marketplace, or add this repository as a marketplace source in Cursor and install `mailgun-cursor`.

To install a release archive instead, unzip `mailgun-cursor-<version>.zip` into `~/.cursor/plugins/local/mailgun-cursor` and run **Developer: Reload Window**. Cursor doesn't install zip files directly.

For a step-by-step walkthrough, see [Get Started with Mailgun MCP and Skills in Cursor Marketplace](https://help.mailgun.com/hc/en-us/articles/56851546909979-Get-Started-with-Mailgun-MCP-and-Skills-in-Cursor-Marketplace).

The Cursor package includes:

- Mailgun MCP config at `plugins/mailgun-cursor/mcp.json`.
- Fifteen skills.
- Safety rules in `plugins/mailgun-cursor/rules`.
- Commands for domain health, bounces, DNS, suppressions, templates, and reporting across domains.

### Claude Code

Add this repository as a Claude plugin marketplace source, then install `mailgun-claude`:

```bash
claude plugin marketplace add mailgun/mailgun-plugins
claude plugin install mailgun-claude@mailgun
```

For the Claude desktop app, download `mailgun-claude-<version>.zip` from the latest release and choose it on the **Upload a plugin** screen.

The Claude package includes a `.mcp.json` file, the shared Mailgun skills, and Claude compatible commands.

### Gemini CLI

Install the latest release:

```bash
gemini extensions install https://github.com/mailgun/mailgun-plugins
```

Add `--ref v<version>` to pin a release. Gemini checks the latest GitHub Release for updates.

Or install the generated Gemini extension branch, which tracks `main`. Gemini asks whether to fall back to `git clone` because no release is named `gemini-extension`; answer yes:

```bash
gemini extensions install https://github.com/mailgun/mailgun-plugins --ref gemini-extension
```

The source package is under `plugins/mailgun-gemini` and includes `gemini-extension.json`, skills, and TOML commands. Gemini requires `gemini-extension.json` at the root of the repository or release archive, so GitHub Actions publishes the `gemini-extension` branch and release archives from that package.

### Codex

Add this repository as a Codex plugin marketplace, then install `mailgun-codex`:

```bash
codex plugin marketplace add mailgun/mailgun-plugins
codex plugin add mailgun-codex@mailgun
```

Codex doesn't install zip files, but the release archive bundles its own marketplace. Unzip it, then add the folder:

```bash
unzip mailgun-codex-<version>.zip -d mailgun-codex
codex plugin marketplace add ./mailgun-codex
codex plugin add mailgun-codex@mailgun
```

The Codex marketplace lives at `.agents/plugins/marketplace.json`. The package includes `.codex-plugin/plugin.json`, a `.mcp.json` that forwards Mailgun variables with `env_vars`, and the shared skills plus Codex versions of the commands and safety rule.

Public listing in the OpenAI plugin directory requires a remote HTTPS MCP server, so this package is installed from GitHub for now.

## Local Development

```bash
npm run sync
npm test
```

`npm test` runs `check:sync`, `validate`, and `build`. `npm run build` writes every platform archive and `SHA256SUMS` to `dist/release/`. With the Claude Code CLI installed, `npm run validate:claude` runs Claude's plugin validator.

To verify the MCP server tag list from npm:

```bash
npm run mcp:tags
```

To test against a local server checkout, build `../mailgun-mcp-server`, then use the local override pattern in [shared/docs/setup.md](shared/docs/setup.md).

## Release

1. Run `npm run bump <version>` and move the `Unreleased` entries in `CHANGELOG.md` under that version.
2. Run `npm test`, open a pull request, and merge it.
3. Tag the merge commit `v<version>` and push the tag. The `Release` workflow builds the archives, creates the GitHub Release, and updates the `gemini-extension` branch.
4. Refresh platform marketplace listings.

See [docs/release-process.md](docs/release-process.md) for details.

## Troubleshooting

- Missing API key: set `MAILGUN_API_KEY` in the platform environment and restart the agent app.
- Wrong region: add `MAILGUN_API_REGION=eu` to the installed MCP config for EU accounts.
- Too many tools: add `MAILGUN_MCP_TAGS` to the installed MCP config with a narrower set, such as `core,send.templates`, or `dynamic` if the client refreshes its tool list when the server reports a change.
- Local server testing: build the MCP server first, then point the platform MCP config at `dist/mailgun-mcp.js`.
