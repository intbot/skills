---
name: status
description: Condensed status view of the current project's implementation board. Auto-discovers the board file (internal/board.md, .claude/board.md, board.md, or docs/board.md), groups by Goal/layer, done-first then by priority. Use the "board" skill for the full view with filters. Read-only.
disable-model-invocation: true
---

Render a **condensed** status view of this project's board.

Locate the board file by checking these paths **in order** and using the first that exists (use the
Read tool): `internal/board.md`, `.claude/board.md`, `board.md`, `docs/board.md`. If none exists, say
so and point the user at the `board` skill (which can scaffold one).

Render:
- First line: `**Legend:** ✅ live · 🟢 done, pending push · 🟡 partial · ⚪ to-do · 🔒 deferred · ⛔ skip`
  (adapt to the statuses the board actually uses).
- Group rows by the board's **Goal** column (or its first grouping column) into `###` sections.
- Condensed columns only: **ID | Item | Priority | Status** — drop Goal/Track/Owner/Manual (those are
  the full-board view). For Item, show **only the bold title** (the text before the ` — ` separator),
  not the description.
- Order done-first, then by priority. End with any release/version line the board carries.

If an argument is passed naming a group/goal, show only that group.

Rules: pull state straight from the board file; don't invent rows or statuses; **read-only** — do not
modify the board.
