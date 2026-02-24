# CLAUDE.md - Worlds n8n Documentation (docs)

> For system architecture, inter-repo API contracts, and cross-repo change guidelines, see the top-level [CLAUDE.md](../CLAUDE.md).

This repo contains the Mintlify-based documentation site for the Worlds n8n platform, covering both internal and externally published content.

## Quick Reference

| Task | Command |
|------|---------|
| Install Mintlify CLI | `npm i -g mint` |
| Start dev server | `mint dev` (serves at http://localhost:3000) |
| Check for broken links | `mint broken-links` |
| Update Mintlify CLI | `npm mint update` |
| Deploy | Automatic on push to `main` (GitHub app integration) |

## Repo Structure

```
docs.json                    # Site configuration (navigation, theme, settings)
.mintignore                  # Files excluded from published site

index.mdx                   # Landing page
getting-started/             # Overview and prerequisites
concepts/                    # Detection pipeline, tracks/zones, signals, check chaining
guides/                      # End-to-end workflow tutorials
nodes/                       # Node reference pages (triggers, checks, events, actions)
state-machine/               # State machine overview and API reference
ai-tools/                    # Claude Code, Cursor, Windsurf integration guides

images/                      # Screenshots and diagrams
logo/                        # Brand logos (dark/light, PNG/SVG)
essentials/                  # Mintlify tutorial pages (reference only)
api-reference/               # Placeholder (sample OpenAPI spec, not Worlds-specific yet)
```

## Configuration

All site configuration lives in `docs.json`:

- **Theme**: Mint with brand colors (primary `#1170E3`, light `#4D9BF7`, dark `#0E58B9`)
- **Logos**: `/logo/light.png` (light mode), `/logo/dark.png` (dark mode)
- **Favicon**: `/favicon.svg`
- **Navigation**: Two tabs -- "Guides" and "Node Reference"
- **Footer**: GitHub link to https://github.com/worlds-io
- **Navbar**: Support mailto link

When adding new pages, they must be added to the appropriate navigation group in `docs.json` to appear on the site.

## Internal vs External Content

Everything referenced in `docs.json` navigation is **published externally**. The following are excluded via `.mintignore` and are internal only:

- `.git`, `.github`, `.claude`, `.agents`, `.idea`, `node_modules`
- `README.md`, `LICENSE.md`, `CHANGELOG.md`, `CONTRIBUTING.md`
- `drafts/`
- `AGENTS.md`

## Writing Standards

### Frontmatter

Every `.mdx` file requires YAML frontmatter with at minimum:

```yaml
---
title: "Clear, Descriptive Page Title"
description: "Concise summary for SEO and navigation."
---
```

Use `sidebarTitle` in frontmatter if the navigation title should differ from the page title.

### Voice and Style

- Second person ("you"), active voice, imperative mood for procedures
- Keep sentences concise -- one idea per sentence
- Use sentence case for headings
- h2 (`##`) for main sections, h3 (`###`) for subsections
- When referencing UI elements, use bold: Click **Settings**
- Use code formatting for file names, commands, paths, and code references
- Prerequisites listed at the start of procedural content
- Include introductory context before diving into steps or details
- Both basic and advanced usage examples where relevant
- Add "Next steps" or related links where helpful
- Match the style and formatting of existing pages before introducing new patterns

### Code Examples

- Use realistic values (not `foo`/`bar`)
- Always include language tags on fenced code blocks
- Add filename labels when referencing specific files

### Links and Images

- Always use **relative paths** for internal links (never absolute URLs)
- PNG for screenshots, SVG for logos
- Alt text required on all images
- Apply `border-radius: 0.5rem` styling for screenshots (via `<Frame>`)

### Tables

Use tables for reference information and configuration parameters.

## Mintlify Components

Prefer these components over raw HTML:

| Component | Purpose |
|-----------|---------|
| `<Tip>` | Actionable advice |
| `<Note>` | Important information the reader should not miss |
| `<Info>` | Prerequisites or contextual information |
| `<Accordion>` | Collapsible sections for optional detail |
| `<Columns cols={2}>` | Side-by-side card layouts |
| `<Card>` | Linked content blocks for navigation |
| `<Steps>` | Numbered procedural instructions |
| `<Tabs>` | Mode or option selection |
| `<Frame>` | Image wrappers with captions and styling |

Refer to existing pages for usage examples before introducing a component in a new way.

## Content Guidelines

- **Document just enough** -- prioritize accuracy and usability over completeness
- **Evergreen content** -- avoid version-specific details that will go stale
- **No duplication** -- search for existing coverage before adding new content
- **Check existing patterns** -- look at similar pages before creating new ones
- **Start small** -- make the smallest reasonable change, then expand
- **Prerequisites first** -- always state what the reader needs before starting a procedure
- **Don't remove pages** without checking for inbound links (`mint broken-links`)
- **Don't add pages to navigation** in `docs.json` that don't exist yet
- **Don't use raw HTML** when a Mintlify MDX component exists for the same purpose

## Working Principles

- Push back on ideas when warranted. Cite sources and explain your reasoning
- Always ask for clarification rather than making assumptions
- Never fabricate information. If something is unknown, say so
- Test all code examples before including them

## Git Workflow

- Never use `--no-verify` when committing
- Never skip or disable pre-commit hooks
- Ask how to handle uncommitted changes before starting work
- Create a new branch when no clear branch exists for changes
- Commit frequently throughout development
- **Never include Claude as an author or co-author in commits**
