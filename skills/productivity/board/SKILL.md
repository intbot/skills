---
name: board
description: Render the current project's implementation board as a single Markdown table. Auto-discovers the board file (internal/board.md, .claude/board.md, board.md, or docs/board.md). Takes an optional filter argument, e.g. "board traffic", "board todo", "board mine". Read-only.
disable-model-invocation: true
---

Render this project's implementation board.

Locate the board file by checking these paths **in order** and using the first that exists (use the
Read tool — don't shell-glob): `internal/board.md`, `.claude/board.md`, `board.md`, `docs/board.md`.

**If no board file exists:** tell the user none was found in this project and offer to scaffold one at
`.claude/board.md` with this skeleton (create it only if they say yes):

```
# Implementation board

**Legend** — Goal: … · Track: … · Pri: P0–P3 · Owner: 🤖 me · 🧑 you · 👥 both ·
Status: ✅ live · 🟢 done, pending push · 🟡 partial · ⏳ to-do · 🔒 deferred · ⛔ skip ·
Item: **bold title** — then a description that wraps (`**Title** — …`)

| ID | Goal | Track | Item | Pri | Owner | Manual step | Status |
|---|---|---|---|:--:|:--:|---|---|
```

**If found:** render the board's **Legend** (if present) and its table, preserving the file's columns
and row order (boards are typically ordered done-first, then by priority). **Wrap long cells — never
truncate the Item or Status text.** The Item column uses the pattern `**Bold title** — description`
(a bold title, an em-dash, then a description that wraps within the cell — `<br>` is avoided because
terminal table renderers ignore it). Render the full title **and** description so each row explains
itself.

The argument passed to this skill (if any) filters which rows to show (case-insensitive). When the
board uses the standard grouping columns, map:
- **Goal (umbrella):** any value in the Goal column — e.g. `presence` / `parity` / `moat`.
- **Track (slice):** any value in the Track column — e.g. `traffic` / `earned` / `infra` / `docs` / `feature` / `hygiene`.
- **State:** `todo` → Status ∈ {⏳, 🟡, 🔒} · `done` → {✅, 🟢} · `blocked` → 🔒 or a manual step on the user still pending.
- **Owner:** `mine`/`me` → 🤖 or 👥 · `yours`/`you` → 🧑 or 👥.
- anything else → substring match over the row's text (ID / Goal / Track / Item).

If the board lacks Goal/Track columns, treat the argument purely as a substring filter. When filtered,
name the filter on one line above the table, then print a one-line tally for the shown rows
(e.g. _"N done · M to-do · K deferred/skip"_).

Rules:
- **Read-only.** Never modify the board file. Pull state straight from it; don't invent rows.
- Project-agnostic: always operate on the board of the current working directory's project.
- A companion `status` skill renders a condensed view of the same file.
