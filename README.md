# AirOps Content Plugin

This repo is your workspace's content plugin. It contains playbooks (skills), agents, and configuration for Claude Code / Cursor.

## Structure

- **CLAUDE.md** — Workspace identity and how to run playbooks (step folders)
- **skills/** — Playbooks as step folders (e.g. `create-blog/`, `create-comparison/`)
- **agents/** — Subagent definitions (content-writer, content-critic, ai-search-analyst, page-analyst)
- **.mcp.json** — MCP configuration (AirOps MCP when used externally)
- **.claude-plugin/marketplace.json** — Marketplace catalog entry for plugin distribution

## Running a playbook

Run the playbook skill in your host environment and pass inputs that match
the playbook's `SKILL.md` frontmatter and argument hint.

Brand context and knowledge base are loaded via available tools (local MCP when running on AirOps runner, or AirOps MCP when using the plugin in Claude Code).
