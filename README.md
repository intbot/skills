# skills

My collection of Agent Skills for Claude Code (and any [skills.sh](https://skills.sh)-compatible agent).

## Quickstart

```bash
npx skills@latest add intbot/skills
```

Then invoke a skill by name in your agent (e.g. `/board`).

## Skills

| Skill | What it does |
|---|---|
| **board** | Renders a project's implementation board as a single Markdown table (ID · Goal · Track · Item · Priority · Owner · Manual step · Status), done-first then by priority. Takes an optional filter — `board traffic`, `board todo`, `board mine`, `board blocked`, or any substring. `board init` scaffolds a new board and offers to auto-populate it. |
| **status** | A condensed view of the same board — grouped by Goal, columns trimmed to ID · Item · Priority · Status. Good for a quick "where do things stand" glance. |

Rendering is read-only and project-agnostic: both skills operate on the board file of whatever project you run them in. Only `board init` writes.

## Getting started

No board file yet? That's the normal starting point.

1. Install: `npx skills@latest add intbot/skills`
2. Run `/board init` in your project. It scaffolds a board and offers to draft your first rows.
3. **Auto-populate (optional).** Before writing, `init` looks for existing work to seed the board:
   - GitHub task lists (`- [ ]` → to-do, `- [x]` → done) anywhere in your markdown,
   - files named `TODO*` / `ROADMAP*` / `PLAN*` / `BACKLOG*`,
   - README/docs sections titled *Roadmap / TODO / Planned / Coming soon*,
   - and the work from your current conversation.
   It drafts rows from those, shows them with a provenance line, and writes **only after you confirm**. Empty project? You get a one-row example starter instead.
4. From then on: `/board` (full), `/status` (condensed), `/board traffic` (filter). No init step again.

Pick where the board lives at init time — the skill uses the first path that exists:

| `init` target | File | Use when |
|---|---|---|
| `init internal` | `internal/board.md` | private/gitignored working area |
| `init docs` / `init root` | `docs/board.md` / `board.md` | you want it committed |
| `init` (default) | `.claude/board.md` | local-only (usually gitignored) |

## The board file

The board is one Markdown table. The **Item** column uses the pattern `**Bold title** — description` so
each row explains itself (a `<br>` line break is avoided because terminal Markdown renderers ignore it).
A discovered follow-up nests under its parent as a `↳ Parent.N` sub-task row. The skills look for the
board at the first path that exists: `internal/board.md` → `.claude/board.md` → `board.md` → `docs/board.md`.

## Updating

Already installed? Pull the latest:

```bash
npx skills@latest update board status -g     # update these two, globally
# or update everything you installed from here:
npx skills@latest update -g
```

(Drop `-g` to update a project-local install.) `npx skills@latest list` shows what you have.

## Badge

[![skills.sh](https://skills.sh/b/intbot/skills)](https://skills.sh/intbot/skills)

## License

MIT.
