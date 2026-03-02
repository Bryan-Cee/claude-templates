# claude-templates

A collection of project-level `CLAUDE.md` templates for use with [Claude Code](https://code.claude.com). Each template encodes project-specific conventions, stack decisions, and hard rules so Claude has accurate context from the first session — without repeating yourself every time.

---

## What This Repo Is

When working with Claude Code, context is everything. A well-written `CLAUDE.md` is the difference between Claude producing code that fits your project and Claude producing code you have to correct and rewrite.

This repo contains ready-to-use templates for common project types. Each one is:

- **Opinionated** — built around real stack decisions, not generic advice
- **Layered** — designed to extend a global `~/.claude/CLAUDE.md`, not replace it
- **Production-tested** — conventions come from shipping real projects, not documentation

---

## How It Connects to Your Global CLAUDE.md

Claude Code merges CLAUDE.md files in a three-layer hierarchy before every session:

```
Layer 1 — ~/.claude/CLAUDE.md         # Your personal preferences (applies everywhere)
Layer 2 — /your-project/CLAUDE.md     # This template (applies to this codebase)
Layer 3 — /your-project/package/CLAUDE.md  # Sub-package overrides (monorepos)
```

The templates in this repo are **Layer 2** files. They only define what's specific to that project type — communication style, commit conventions, and general engineering principles belong in your global Layer 1 file and don't need to be repeated here.

If you don't have a global `CLAUDE.md` yet, start there before using these templates. Without Layer 1, Layer 2 has to carry too much and becomes bloated.

---

## Available Templates

| Template | Stack | Use When |
|---|---|---|
| `react-nextjs.md` | Next.js App Router, Tailwind, React Query, Zustand | Building a React frontend with Next.js |

More templates will be added over time.

---

## How to Use

**1. Pick the right template for your project type** using the table above.

**2. Copy it into your project root:**

```bash
cp react-nextjs.md /your-project/CLAUDE.md
```

**3. Fill in the placeholders** at the top of the file:

```markdown
## Project Overview
- **Name:** [PLACEHOLDER]
- **Purpose:** [PLACEHOLDER]
- **API / Backend:** [PLACEHOLDER]
```

Every template has a checklist at the bottom listing exactly what needs to be filled in before your first session.

**4. Open Claude Code in your project** — it will pick up the CLAUDE.md automatically.

```bash
cd /your-project
claude
```

---

## Tips for Getting the Most Out of These Templates

**Don't skip the placeholders.** The project overview section is the first thing Claude reads. Vague or missing context there leads to vague output everywhere.

**Treat the template as a starting point, not a finished file.** After your first few sessions, you'll notice things Claude gets wrong or assumptions it makes that don't match your project. Add those corrections directly to the CLAUDE.md while they're fresh.

**Use subdirectory CLAUDE.md files for large projects.** In a monorepo or a project with distinct modules (e.g. `packages/api`, `packages/web`), create additional CLAUDE.md files at the subdirectory level for package-specific conventions. The root CLAUDE.md covers the whole repo; subdirectory files cover only that package.

**Version control your CLAUDE.md.** Commit it to the repo alongside your code. Teammates using Claude Code will benefit from the same context automatically, and you get a history of how your conventions evolved.

---

## Contributing

If you've built a template for a stack not covered here and want to contribute, open a PR. Templates should follow the same structure: stack overview, folder layout, conventions by concern, and a placeholder checklist at the bottom.
