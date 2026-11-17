# Changelog

Each release is tagged `v<version>` and published as a GitHub Release with an installable archive per platform. See [docs/release-process.md](docs/release-process.md).

## Unreleased

- Add the Codex plugin package.
- Build release archives for Claude, Cursor, Codex, and Gemini with `npm run build`, and attach them to a GitHub Release when a `v*` tag is pushed.
- Add `npm run bump <version>` and fail validation when package, manifest, and marketplace versions disagree.

## 0.1.2

- Pin plugin MCP startup configs to `@mailgun/mcp-server@2.3.1`.
