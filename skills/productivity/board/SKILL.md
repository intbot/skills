---
name: board
description: Render the current project's implementation board as a single Markdown table. Auto-discovers the board file (internal/board.md, .claude/board.md, board.md, or docs/board.md). Optional filter, e.g. "board presence", "board traffic", "board todo", "board parked", "board mine", or a parent ID like "board T-D". Read-only.
disable-model-invocation: true
---

Render this project's implementation board.

Locate the board file by checking these paths in order and using the first that exists (use Read,
don't shell-glob): `internal/board.md`, `.claude/board.md`, `board.md`, `docs/board.md`.

**If found:** render the board's **Legend** and its table, all columns
(**ID | Goal | Track | Item | Priority | Owner | Manual step | Status**), in the file's row order
(done-first, then by priority). **Wrap long cells — never truncate Item or Status.** The Item column
uses `**Bold title** — description` (em-dash, not `<br>`, which terminal tables ignore); render the
full title and description.

**Sub-tasks:** a discovered follow-up nests under its parent as a `↳ Parent.N` row (e.g. `↳ T-D.1`);
`·` in Goal/Track means "inherits parent"; it carries its own Priority/Owner/Status. Keep each
sub-task **immediately under its parent** (preserve file order) and render the `↳`/`·` as-is.
`⏸️ parked` = a follow-up deferred by choice.

**If no board file exists:** say so and offer to scaffold `.claude/board.md` (create only if the user agrees):

```
# Implementation board

**Legend** — Goal: … · Track: … · Priority: 🔴 P0 · 🟠 P1 · P2 · P3 · Owner: 🤖 me · 🧑 you · 👥 both · ⚠️ = a P0/P1 item waiting on you · Status: ✅ live · 🟢 done, pending push · 🟡 in progress · ⚪ to-do · ⏸️ parked · 🔒 deferred · ⛔ skip · Sub-tasks: ↳ Parent.N rows nested under their parent (· inherits Goal/Track)

| ID | Goal | Track | Item | Priority | Owner | Manual step | Status |
|---|---|---|---|:--:|:--:|---|---|
```

The argument passed to this skill (if any) filters which rows to show (case-insensitive):
- **Goal:** `presence` / `parity` / `moat` → that Goal.
- **Track:** `traffic` / `earned` / `infra` / `docs` / `feature` / `hygiene` → that Track.
- **State:** `todo` → ⚪ or 🟡 · `done` → ✅ or 🟢 · `parked` → ⏸️ · `blocked`/`deferred` → 🔒.
- **Owner:** `mine`/`me` → 🤖 or 👥 · `yours`/`you` → 🧑 or 👥.
- **Parent:** a bare ID like `T-D` matches the parent **and** its sub-tasks (`T-D.1`, …) via substring.
- anything else → substring over ID / Goal / Track / Item.

When filtered, name the filter on one line above the table, then a one-line tally:
_"N done · M to-do · K parked · J deferred/skip"_.

Rules:
- **Read-only.** Never modify the board file. Pull state straight from it; don't invent rows.
- Project-agnostic: always operate on the board of the current working directory's project.
- A companion `status` skill renders a condensed view of the same file.
