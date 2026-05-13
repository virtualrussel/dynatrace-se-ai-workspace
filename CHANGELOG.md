# Changelog

All notable changes to this workspace are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [1.1.0] — 2026-05-13

### Added
- Claude Code CLI support: run `claude` from the repo root — no VS Code required
- `.claude/commands/` with 7 investigation workflows symlinked from `.github/prompts/`
  (single source of truth; prompts serve both Copilot Chat and Claude Code CLI)
- "Using without VS Code" section in README covering the CLI setup path
- Claude Code CLI callouts in Setup steps 5 and 6 and in docs/ELI5.md

### Fixed
- Prompt trigger documented as `@` in CLAUDE.md, README, ELI5.md, and OVERVIEW.md —
  corrected to `/` (the actual syntax in both Copilot Chat and Claude Code)
- ARCHITECTURE.md MCP server location listed `.vscode/mcp.json` only —
  added `.mcp.json` as the Claude Code CLI and GitHub Copilot CLI path
- README MCP configuration table listed `.mcp.json` as "GitHub Copilot CLI" only —
  updated to include Claude Code CLI

## [1.0.0] — 2026-05-05

Initial release: VS Code workspace for GitHub Copilot and Claude Code (VS Code extension)
with 16 Dynatrace domain skills, 7 investigation prompt workflows, dual MCP server config
(production + sprint), and dtctl CLI integration.
