---
name: status
description: Condensed status view of the current project's implementation board. Auto-discovers the board file (internal/board.md, .claude/board.md, board.md, or docs/board.md), groups by Goal, done-first then by priority. Use the "board" skill for the full view with filters. Read-only.
disable-model-invocation: true
---

Render a **condensed** status view of this project's board.

Locate the board file (first of `internal/board.md`, `.claude/board.md`, `board.md`, `docs/board.md`).
If none, say so and point the user at the `board` skill (which can scaffold one).

Render:
- First line: `**Legend:** ✅🟢 done · 🟡 in progress · ⚪ to-do · ⏸️ parked · 🔒 deferred · ⛔ skip · 🔴 P0 · 🟠 P1`
- Group rows by the board's **Goal** column (Presence / Parity / Moat), one `###` table each.
- Columns: **ID | Item | Priority | Status** (drop Goal/Track/Owner/Manual). For Item, show **only the
  bold title** (text before the ` — ` separator).
- Keep **sub-task rows nested**: render their `↳ Parent.N` ID and `↳` title directly under the parent.
- Order done-first, then by priority. End with any release/version line.

If an argument names a goal (`presence`/`parity`/`moat`), show only that group.

Rules: pull state straight from the board file; don't invent rows or statuses; **read-only**.
