# Skill Foundry

This repository contains Claude Code skills and related project resources.

## Project Structure
- `.claude/` - Claude Code session files and configuration
- `skills/` - Claude Code skill definitions

## Development Guidelines
- Skills follow the YAML frontmatter + XML-structured content format
- Each skill should include both SKILL.md and README.md files

## External Resources
- Shared assets: [placeholder]
- Design files: [placeholder]

## Skill Location
When there is a specific reference to a Claude Skill, or the context indicates that a Claude Skill should be invoked, note that all skills used in this project are personal skills and located at: /Users/john/.claude/skills

## MCP Server Workflow (Docker MCP Gateway)

### Architecture

Core gateway (always present, ~6 tools, ~4k tokens):
- mcp-find, mcp-add, mcp-remove, mcp-config-set, mcp-exec, code-mode

Server management (via Bash, persists to gateway):
- `docker mcp server enable [name]` - adds server and tools
- `docker mcp server disable [name]` - removes server and tools
- `docker mcp server reset` - nuclear option, back to 6 core tools
- `docker mcp tools count` - check current tool count

After enable/disable: user needs /clear for Claude to see changes.

### Philosophy

- Clean slate: start sessions with only 6 core tools
- Human-in-the-loop: ask before adding servers
- Session-aware: track what's added, clean up at end

### Workflow

At session start (`/session-start`):
- Check tool count (should be 6)
- Ask which servers needed
- Enable via: `docker mcp server enable [name]`
- User runs /clear to load new tools

During session:
- "I could add [server] for [task]. Should I?"
- If yes: enable via Bash, user /clear, log in session-context.md

At session end (`/session-end`):
- Disable each server via: `docker mcp server disable [name]`
- Verify tool count returns to 6
- If stuck: `docker mcp server reset`

### Phrasing

- "I could add Context7 for up-to-date docs. Should I?"
- "GitHub MCP would help with this PR. Add it?"
- "This needs database access - add postgres?"
