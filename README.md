# Claude Visualize Plugin

A `/visualize` command for Claude Code that turns a branch, a ticket, or a free-form codebase question into a rich, interactive HTML artifact a teammate can actually understand — plain English first, with concrete data examples, structures, and contracts carrying every idea.

## What It Does

Give it one of three inputs:

| Input | Example | The artifact covers |
|-------|---------|---------------------|
| Ticket key or URL | `/visualize PROJ-123` | The problem, the design, what shipped vs what remains |
| Branch or PR URL | `/visualize feature/PROJ-123-service-versions` | What changed, why, and how data flows differently before vs after |
| Free-form query | `/visualize how does the search indexing pipeline work` | How the thing actually works today, with real code paths cited |

Claude researches the subject (diffs, tickets, code paths), collects real contracts and sample data, optionally captures real screenshots of the UI, then builds and publishes a self-contained interactive HTML artifact with:

- **Plain English leading every section** — written for a smart teammate from another team, not documentation
- **One protagonist entity** traced end to end so the reader accumulates familiarity
- **Real data examples everywhere** — sample requests/responses, branch-logic tables with example values, filled-in schema rows, literal error bodies
- **Syntax-colored code samples** — an inlined tokenizer (CSP forbids CDN highlighters) colors JSON and HTTP blocks in both light and dark themes
- **Per-endpoint request/response cards** for API subjects — side-by-side payloads covering the happy path plus empty, validation (422), throttle (429), and auth (401) variants, with load-bearing fields annotated
- **Real screenshots** (desktop/mobile switchers, before/after comparisons) when the subject has a UI surface
- **A glossary** of every internal codename, plus honest SHIPPED / IN REVIEW / PLANNED / BUG status chips
- **Light and dark themes**, Mermaid diagrams, collapsible detail blocks, copy buttons

## Installation

### Option 1: Plugin Marketplace (Recommended)

```bash
# Add the marketplace
/plugin marketplace add SnakeO/claude-visualize

# Install the plugin
/plugin install visualize@snakeo-visualize
```

### Option 2: Git Clone

```bash
git clone https://github.com/SnakeO/claude-visualize.git
# Copy skill folders to ~/.claude/skills/
cp -r claude-visualize/plugins/visualize/skills/* ~/.claude/skills/
```

### Option 3: Manual Copy

Copy `plugins/visualize/skills/` contents to `~/.claude/skills/`.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) with the Artifact tool (used to publish the HTML page)

### Optional integrations (used when available)

- **A browser MCP server** ([Playwright](https://github.com/microsoft/playwright-mcp) or [Chrome DevTools](https://github.com/ChromeDevTools/chrome-devtools-mcp)) — for capturing real screenshots of UI surfaces
- **An issue-tracker MCP server** (Jira, Linear, etc.) — for ticket subjects
- **[GitHub CLI](https://cli.github.com/)** (`gh`) — for PR subjects and finding PRs linked to tickets

Without these, `/visualize` still works — it just skips screenshots or asks you to paste ticket details.

## Usage

```
/visualize PROJ-123
/visualize feature/PROJ-123-service-versions
/visualize https://github.com/your-org/your-repo/pull/456
/visualize how does neighborhood resolution work in the curation pipeline
```

Run it again on the same subject and the artifact redeploys to the same URL.

## License

MIT
