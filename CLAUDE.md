# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code plugin (`snakeo-visualize`) providing a single `/visualize` skill that turns a branch, ticket, or free-form codebase question into a rich, interactive HTML artifact. No compiled code, no build steps, no tests — purely a Markdown skill definition and JSON config.

## Architecture

```
.claude-plugin/marketplace.json   ← marketplace registration (name, version, plugin refs)
plugins/visualize/
  plugin.json                     ← plugin definition (auto-discovers skills via glob)
  skills/
    visualize/SKILL.md            ← the /visualize command
```

**Skill discovery:** `plugin.json` uses `"skills": ["./skills/*"]` — any subdirectory with a `SKILL.md` is auto-registered.

**SKILL.md format:** YAML frontmatter (`name`, `description`) followed by Markdown instructions that Claude Code follows when the skill is invoked.

## Design Principles of the Skill

The SKILL.md follows a three-phase flow: research the subject (ticket/branch/query), optionally capture real screenshots via browser MCP tools, then build and publish a self-contained HTML artifact. Its core content rules: plain English leads every section, data examples carry every idea, one protagonist entity threads the whole story, and honest shipped/planned/broken status. When editing, preserve these principles — they are the point of the command, not decoration.

The skill must stay generic: no company-specific repo names, hostnames, ticket prefixes (beyond example placeholders like `PROJ-123`), or personal MCP server names.

## Versioning

**Always bump the version when making changes.** Version must be updated in three places and kept in sync:

1. `.claude-plugin/marketplace.json` → `metadata.version`
2. `.claude-plugin/marketplace.json` → `plugins[0].version`
3. `plugins/visualize/plugin.json` → `version`
