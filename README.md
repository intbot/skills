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
| **board** | Renders a project's implementation board as a single Markdown table (ID · Goal · Track · Item · Priority · Owner · Manual step · Status), done-first then by priority. Takes an optional filter — `board traffic`, `board todo`, `board mine`, `board blocked`, or any substring. Auto-discovers the board file; offers to scaffold one if none exists. |
| **status** | A condensed view of the same board — grouped by Goal, columns trimmed to ID · Item · Priority · Status. Good for a quick "where do things stand" glance. |

Both are read-only and project-agnostic: they operate on the board file of whatever project you run them in.

## The board file

Skills look for the board at the first path that exists: `internal/board.md` → `.claude/board.md` →
`board.md` → `docs/board.md`. The board is one Markdown table. The **Item** column uses the pattern
`**Bold title** — description` so each row explains itself (a `<br>` line break is avoided because
terminal Markdown renderers ignore it). Run `/board` in a project with no board file and it offers to
scaffold a starter `.claude/board.md`.

## Badge

[![skills.sh](https://skills.sh/b/intbot/skills)](https://skills.sh/intbot/skills)

## License

MIT.
