# Repository Guidelines

## Project Overview

This is the Vio MCP Skill — a portable skill that teaches AI agents how to search hotels, compare prices, and explore detailed hotel information via the Vio MCP server. It targets OpenClaw, claude.ai, Claude Code, and any agent that supports the skill format.

## Project Structure

- `SKILL.md` — Core skill definition with YAML frontmatter and conversation flow guidance
- `references/tool-reference.md` — Complete tool schemas, parameters, and example payloads
- `README.md` — Setup and usage documentation
- `.mise.toml` — Build commands (`mise run zip`)
- `version.txt` — Current version (managed by release-please)
- `skills/` — Legacy nested structure for agents that discover skills from subdirectories

## Build & Development

- `mise run zip` — Build `vio.zip` for upload to claude.ai or distribution
- The zip structure must be: `vio/SKILL.md` + `vio/references/` (folder at root named `vio`)

## Editing the Skill

- `SKILL.md` is the core file. Keep it under 2,000 words — move detailed content to `references/`
- Write in **imperative form**, not second person ("Search for hotels" not "You should search")
- The `description` field in frontmatter must use **third person** with specific trigger phrases
- Tool schemas live in `references/tool-reference.md` — update this when the MCP server tools change
- Source of truth for tool schemas: the Vio MCP server (`https://mcp.vio.com/docs/`)

## Testing

- **claude.ai**: Upload `vio.zip` via Customize → Skills, add MCP server via Integrations
- **OpenClaw**: Copy skill to `~/.agents/skills/vio/`, add MCP server to `~/.openclaw/openclaw.json`
- **Claude Code**: Use `--plugin-dir` with a plugin wrapper

## Releases

- Uses [release-please](https://github.com/googleapis/release-please) for automated versioning
- Push conventional commits to `main` → release-please creates a release PR
- When merged, the release workflow builds `vio.zip` and attaches it to the GitHub release
- Use [Conventional Commits](https://www.conventionalcommits.org/) prefixes: `feat:`, `fix:`, `chore:`, `docs:`

## Coding Style

- Markdown throughout
- Keep SKILL.md lean — progressive disclosure via `references/`
- No emojis in skill content unless explicitly requested
