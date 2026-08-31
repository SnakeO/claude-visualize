---
name: visualize
description: Turn a branch, a ticket, or a free-form codebase question into a rich, interactive HTML artifact a teammate can actually understand. Use when the user wants to visualize or explain a change, a PR, a ticket's scope, or how part of a codebase works - plain English first, with concrete data examples carrying every idea.
---

# Visualize

Turn a branch, a ticket, or a free-form codebase question into a rich, interactive HTML artifact that a teammate can actually understand - plain English first, with concrete data examples, structures, and contracts carrying every idea.

## Input

`$ARGUMENTS` - one of:

- **A ticket key or URL** from your issue tracker (e.g. `PROJ-123`, a Jira/Linear URL, a GitHub issue link)
- **A branch name or PR URL** (e.g. `feature/PROJ-123-service-versions`, a GitHub PR link)
- **A free-form codebase query** (e.g. "how does the search indexing pipeline work")

If no argument is given, ask which of the three the user wants to visualize.

## Phase 1 - Resolve the subject

Classify the input and gather the raw material. Use parallel Explore subagents for anything that means sweeping multiple files or repos.

- **Ticket**: fetch the issue (summary, description, acceptance criteria, parent epic, linked tickets) via whatever issue-tracker MCP tool or CLI is available. Check for planning/design docs referencing it in the repo(s) (`grep -rl "<KEY>"` across docs/plans directories). Find associated PRs (`gh pr list --search "<KEY>" --state all` across the relevant repos). The visualization covers: the problem, the design, what shipped vs what remains.
- **Branch / PR**: identify the repo, diff against the base branch (`git diff <base>...<branch> --stat` first, then the meaningful hunks). Read the PR body and review threads if a PR exists. The visualization covers: what changed, why, and how the data flows differently before vs after.
- **Query**: explore the codebase(s) to answer it properly - entry points, data flow, storage, contracts between services. The visualization covers: how the thing actually works today, with the real code paths cited.

While researching, collect the raw material the artifact will be built FROM:

1. **Real contracts**: request/response shapes from API resources, serializers, request validators, controllers, TypeScript types - reconstruct realistic sample JSON from them (plausible invented values are fine; never real customer data - use reference numbers, fake names like your seed/fixture data, round dollar amounts).
2. **Real branch logic**: every conditional/fallback worth explaining, with what each branch produces.
3. **One protagonist**: a single realistic entity (an order, a user, a document - whatever the domain's central object is) that can be traced through the whole story end to end.
4. **The codenames**: every internal term a reader will hit (table names, enum values, node names) - these feed a glossary.

## Phase 1.5 - Screenshots (when the subject has a UI surface)

If the subject touches anything a user sees (a new screen, a changed component, a before/after behavior), capture REAL screenshots with a browser MCP tool and embed them - a screenshot of the actual feature beats any mockup or description.

**Tool selection** (use what is available and fits the target):

- **A browser MCP tool that carries an authenticated session** (if one is configured) - best for pages behind SSO/login: the session already carries the auth. Treat it as a shared surface: navigate and read only, never click through destructive actions, and return to a neutral page when done.
- `mcp__playwright__*` / `mcp__chrome-devtools__*` - isolated browsers. Best for local dev servers (`npm run dev`), public pages, and anything needing viewport emulation (`browser_resize` / `emulate`).

**What to capture:**

- **Feature shots**: the surface in its meaningful states (empty, populated, error, the dialog open) - drive the state with seed/fixture data, never real customer data; mask or crop anything doubtful.
- **Before/after pairs**: for branch/PR subjects, capture "before" from the base branch or the deployed environment and "after" from the branch running locally. Capture both at the SAME viewport and page position so the comparison reads instantly. If a faithful "before" is not practically reachable, say so in the artifact rather than faking one.
- **Desktop AND mobile where the surface is responsive**: standard pair = 1440x900 and 390x844. Skip mobile for admin-only desktop tools that have no mobile audience - "where applicable" is a judgment call, make it.

**Embedding (CSP forbids remote images):** inline every capture as a `data:` URI (`<img src="data:image/png;base64,...">`). Mind the 16MB artifact cap: scale captures down (`sips -Z 1200 shot.png`) and prefer JPEG (`sips -s format jpeg -s formatOptions 70`) for photographic content; budget roughly 200-400KB per image and drop redundant shots before dropping quality. Base64 via `base64 -i shot.jpg`.

## Phase 2 - Build the artifact

**Load the `artifact-design` skill BEFORE writing any HTML** (required by the Artifact tool). Write the file to the session scratchpad directory with a short, distinctive basename derived from the subject (e.g. `proj-123-atlas.html`); re-using the same path on a later run redeploys to the same URL. If the user wants an artifact from an EARLIER session updated, find its URL via `Artifact action: "list"` and pass it as `url`.

Hard constraints (CSP): fully self-contained - no CDN scripts, external fonts, or remote images; inline all CSS/JS. Mermaid diagrams render natively via `<pre class="mermaid">` blocks. Both light and dark themes via tokenized custom properties (`prefers-color-scheme` + `:root[data-theme=...]` overrides). Wide content scrolls inside its own `overflow-x: auto` container. Pick a favicon emoji that fits the subject and keep it stable across redeploys.

### Content principles (the point of this command)

1. **Plain English leads.** Every section opens with 1-3 sentences a non-engineer could follow - what this thing is FOR and why anyone should care - before any technical detail. Write like explaining to a smart teammate from another team, not like documentation.
2. **Data carries the ideas.** Never explain a concept with prose alone when an example can show it:
   - A contract -> a realistic sample request AND response, syntax-highlighted, with the interesting fields annotated.
   - Conditional/fallback logic -> a table with one row per branch, each row holding EXAMPLE VALUES showing what that branch produces.
   - A pipeline or lifecycle -> the protagonist entity traced stage by stage, showing its actual data mutating (before/after rows, growing JSON).
   - A schema -> a filled-in example row, not a column list.
   - An error path -> the literal error body the caller sees.
3. **One worked example end to end.** The protagonist from Phase 1 threads through the whole artifact - the same IDs and names recur so the reader accumulates familiarity instead of juggling abstractions.
4. **Glossary.** Every codename gets a plain-English definition, ideally with an example value.
5. **Honest state.** Mark what is shipped, in review, planned, or known-broken. Show file references as `path/to/file.ext:123` so an engineer can jump in.

### Interactivity menu (use what serves the content, skip what does not)

- Sticky section nav or tab bar for multi-act structure
- Collapsible detail blocks (`<details>` or JS toggles): plain-English summary visible, deep detail expandable
- Before/after or draft/published toggles that swap the SAME example between states
- Hover/click on diagram nodes revealing the sample payload at that step
- Copy buttons on slugs, IDs, and sample payloads
- Status chips (SHIPPED / IN REVIEW / PLANNED / BUG) with a consistent color language
- Mermaid for flows and sequence diagrams; hand-rolled HTML/CSS for data-example layouts (tables beat diagrams for field-level detail)
- **Desktop/Mobile switcher** wherever both viewports were captured: one toggle swapping the SAME shot between viewports (mobile rendered at phone width inside a device-style frame); keep the pair visually adjacent to the concept it illustrates
- **Before/After comparison** for change shots: a labeled toggle or side-by-side with a divider - identical viewport and crop on both sides, with a one-line plain-English caption stating what changed

## Phase 3 - Publish and report

Publish with the Artifact tool (concise `<title>` in the HTML, one-sentence `description`, stable `favicon`). Then report back:

- The artifact URL
- A 3-5 bullet summary of what the artifact contains
- Anything the research surfaced that the user should know even without opening it (a bug found, a stale doc, a surprising contract)

## Quality bar

Before publishing, self-check: could someone who has never seen this code follow the story using only the plain-English paragraphs and the examples? Does every concept have at least one concrete data example? Does the worked example use the same protagonist throughout? Are both themes legible? If the subject has a UI, are there real screenshots (with desktop/mobile and before/after switchers where applicable), and is the total page under the 16MB cap? Do screenshots show only seed/fixture data? If any answer is no, fix it first.
