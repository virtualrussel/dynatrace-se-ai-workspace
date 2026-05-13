# Changelog

All notable changes to this workspace are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [1.1.2] — 2026-05-13

### Fixed
- ARCHITECTURE.md MCP section: replaced generic "Copilot" with "AI assistants" / "the AI"
- ARCHITECTURE.md prompt locations: corrected `.claude/commands/` description from
  "Claude Code CLI" to "Claude Code (VS Code extension and CLI)"
- README.md Step 4: replaced generic "Copilot" with "the AI client"
- README.md file tree: moved `.claude/skills/` inside the `.claude/` directory block
  (was incorrectly listed as a sibling directory)

## [1.1.1] — 2026-05-13

### Fixed
- README line 3: "Claude AI" corrected to "Claude Code" (wrong brand name)
- README tools table: VS Code row qualified as "(VS Code paths only)" — CLI users don't need it
- README Step 1: Added CLI skip callout (consistent with ELI5.md)
- README Skills section and Key Concepts: replaced "Copilot" with "AI assistants" / "the AI" in
  generic sections that apply to all three client paths
- ARCHITECTURE.md skills description: replaced "teach Copilot" / "Copilot only loads" with
  client-agnostic language
- ARCHITECTURE.md dtctl section: replaced "created by Copilot" with "created by AI workflows"

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
  added `.mcp.json` as the Claude Code CLI path
- README MCP configuration table updated to correctly reflect Claude Code CLI as the `.mcp.json` client

## [1.0.0] — 2026-05-05

Initial release: VS Code workspace for GitHub Copilot and Claude Code (VS Code extension)
with 16 Dynatrace domain skills, 7 investigation prompt workflows, dual MCP server config
(production + sprint), and dtctl CLI integration.
