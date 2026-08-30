# Workbench → Portfolio Contract

How to write entries that the portfolio site can read.

## File Location

```
workbench/
  projects/<slug>/README.md
  explorations/<slug>/README.md
```

One `README.md` per entry. The filename `README.md` is required — it's the only file the portfolio reads.

## Frontmatter

Every `README.md` must start with YAML frontmatter:

```yaml
---
title: "Entry Title"
summary: "One-line description shown on cards and page header."
type: project | exploration
status: draft | active | completed | archived
tags:
  - Tag1
  - Tag2
started: "YYYY-MM"
updated: "YYYY-MM"
context: "Optional longer context shown as a callout box."
source: "https://github.com/user/repo"
---
```

### Fields

| Field      | Required | Notes                                                  |
| ---------- | -------- | ------------------------------------------------------ |
| `title`    | yes      | Used to generate the URL slug                          |
| `summary`  | yes      | One line. Shown on cards + page header                 |
| `type`     | yes      | `project` = built thing, `exploration` = research      |
| `status`   | yes      | `draft`, `active`, `completed`, `archived`             |
| `tags`     | yes      | Array of strings. Shown as chips on cards + header     |
| `started`  | no       | `YYYY-MM` format. Used for sorting (newest first)     |
| `updated`  | no       | `YYYY-MM` format. Last content update. Overrides `started` for sitemap `lastmod` + RSS dates |
| `context`  | no       | Longer description. Shown as "Record Context" box      |
| `source`   | no       | URL to project repo. Shown as "Source ↗" chip          |

## Slug Generation

Slugs are generated from the `title` field, not the folder name.

```
"Radio And Embedded Systems" → /workbench/radio-and-embedded-systems
"NAT Traversal"              → /workbench/nat-traversal
"irosh"                      → /workbench/irosh
```

Rules: lowercase, spaces→hyphens, strip special characters, no leading/trailing hyphens.

**The folder name doesn't matter for the URL.** Only the title does.

## Markdown Body

After frontmatter, write whatever you want. The portfolio renders full Markdown:

- Headings (`##`, `###`)
- Code blocks (with language highlighting)
- Mermaid diagrams (rendered at build time, expandable fullscreen)
- Tables
- Images
- Blockquotes
- Lists

No special structure required. Write the entry to stand on its own.

## Example Entry

```markdown
---
title: NAT Traversal
summary: Investigating how devices communicate across private networks, firewalls, and peer-to-peer paths.
type: exploration
status: active
tags:
  - Networking
  - Protocols
  - Infrastructure
started: "2025-03"
updated: "2025-08"
context: A deep dive into STUN/TURN, ICE protocols, hole punching, and peer connectivity across constrained network boundaries.
source: https://github.com/shedrackgodstime/nat-traversal
---

## Technical Background

NAT traversal is one of the hardest problems in networking...

## Key Concepts

### NAT Types

| Type | Behavior | Traversable? |
|------|----------|-------------|
| Full Cone | Any external host can send | Easy |
| Symmetric | Different port per dest | Requires TURN |

## Code Example

\`\`\`rust
// Example code here
\`\`\`

## Lessons Learned

- ...
- ...
```

## Quick Checklist

- [ ] File is `README.md` inside `projects/<name>/` or `explorations/<topic>/`
- [ ] Frontmatter has all required fields (`title`, `summary`, `type`, `status`, `tags`)
- [ ] `type` is either `project` or `exploration`
- [ ] `status` is one of: `draft`, `active`, `completed`, `archived`
- [ ] `started` is `YYYY-MM` format (if included)
- [ ] `updated` is `YYYY-MM` format (if included)
- [ ] `source` is a valid URL (if included)
- [ ] Markdown body is the full public narrative
- [ ] Entry can stand on its own without external context
