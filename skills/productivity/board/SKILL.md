---
name: board
description: Render or initialize a project's implementation board (one Markdown table — ID · Goal · Track · Item · Priority · Owner · Manual step · Status). `board` renders it with a composable query (space = AND, `,` = OR, `!` = NOT over Goal/Track/Priority/State/Owner), plus `board next`, `board help`, `board by <column>`, and `board <column>` to list a column's distinct values. `board init [docs|root|internal|<path>]` scaffolds a board and offers to auto-populate it. Rendering is read-only; only `init` writes.
disable-model-invocation: true
---

Render — or, with `init`, create — this project's implementation board.

## Locating the board

Check these paths in order, first that exists wins (use Read, don't shell-glob):
`internal/board.md`, `.claude/board.md`, `board.md`, `docs/board.md`.

## `board [query]` — render, READ-ONLY

**If a board file exists:** render its **Legend** and table, all columns
(**ID | Goal | Track | Item | Priority | Owner | Manual step | Status**), in file order
(done-first, then by priority). **Wrap long cells — never truncate Item or Status.** The Item column
uses `**Bold title** — description` (em-dash, not `<br>`, which terminal tables ignore); render the
full title and description.

**Sub-tasks:** a follow-up nests under its parent as a `↳ Parent.N` row (e.g. `↳ T-D.1`); `·` in
Goal/Track means "inherits parent"; it carries its own Priority/Owner/Status. Keep each sub-task
**immediately under its parent** (preserve file order) and render `↳`/`·` as-is. `⏸️ parked` = a
follow-up deferred by choice.

**If no board file exists:** say so and tell the user to run `board init` (don't scaffold here —
that's init's job).

### Reading the argument — resolve in this order

1. **`help`** → print *only* this one-line cheatsheet, nothing else:
   `board [query] · space=AND ,=OR !=NOT over goal·track·p0-p3·state·mine/yours · board <column> lists its values · board by <column> · board next · board init`
2. **`next`** → the live priorities: rows that are **not** done/parked/deferred/skip, ordered
   🔴 → 🟠 → P2 → P3 (then file order), capped at ~8. A "what do I do now" view.
3. **`by <column>`** (`by goal` / `by track` / `by owner` / `by priority` / `by status`) → render every
   non-excluded row **re-grouped** into one `###` table per distinct value of that column (instead of
   file order), each group in priority order.
4. **a single column name** (`goal` / `track` / `owner` / `priority` / `status`) → don't render the
   table; print that column's **distinct values with row counts**, busiest first
   (e.g. `Presence (33) · Parity (3) · Moat (3)`). Lets a newcomer discover this project's labels.
5. **anything else → a query** (see grammar below) that filters which rows to show.
6. **no argument** → the full board.

### Query grammar

Tokens are **space-separated and AND together**; a `,` inside a token is **OR**; a leading `!`
**negates**. Case-insensitive. A token matches a row when it matches any of:

- **Goal** — the Goal cell (e.g. `presence`).
- **Track** — the Track cell (e.g. `traffic`, `earned`).
- **Priority** — `p0` `p1` `p2` `p3`.
- **State** — a literal status word (`live`, `published`, `review`, `parked`, `deferred`, `skip`, …)
  **or** a group alias: `done` = ✅|🟢 · `todo` = ⚪ · `active`/`wip` = 🟡 · `parked` = ⏸️ ·
  `blocked`/`deferred` = 🔒 · `skip` = ⛔.
- **Owner** — `mine`/`me` = 🤖|👥 · `yours`/`you` = 🧑|👥.
- **Parent** — a bare ID like `T-D` matches that row **and** its `↳ T-D.N` sub-tasks.
- else **substring** over ID / Goal / Track / Item.

Literal vs group: `!live` drops only ✅ **live**; `!done` drops everything finished (✅ and 🟢).

Examples: `presence !live` · `traffic,earned !done p1` · `mine !parked` · `presence p0 !live` · `T-D`.

### Output rules

- **Sub-task cohesion:** if a parent row matches, include its `↳` sub-tasks even when they don't match;
  if a sub-task matches, include its parent for context. Keep them adjacent.
- **Filtered / `by` / `next` renders:** name the active filter on one line above the table.
- **Always end with a tally** of the rows shown: `_N done · M to-do · K parked · J deferred/skip_`
  (done = ✅|🟢 · to-do = ⚪|🟡 · parked = ⏸️ · deferred/skip = 🔒|⛔).

## `board init [target]` — create the board, the ONLY write path

Use this when there's no board yet (or the user explicitly asks to set one up).

1. **Resolve the target path** from the token after `init`:
   `docs` → `docs/board.md` · `root` → `board.md` · `internal` → `internal/board.md` ·
   `claude` or nothing → `.claude/board.md` · otherwise treat the token as a literal path.
   If the target is under `.claude/`, note it's usually gitignored (suggest `docs/board.md` or
   `board.md` if they want it committed).
2. **Never clobber.** If a board already exists at *any* discovery path, show that one and stop —
   overwrite only if the user explicitly confirms.
3. **Gather candidate rows** from two sources:
   - **This conversation** — work in progress, plus any bug/improvement discovered mid-task (nest the
     latter as a `↳` sub-task under the item it came from).
   - **Project markdown**, high-signal only — GitHub task lists (`- [ ]` → ⚪ to-do, `- [x]` → ✅ done),
     files named `TODO*` / `ROADMAP*` / `PLAN*` / `BACKLOG*` (case-insensitive), and README/docs
     sections titled *Roadmap / TODO / Planned / Coming soon / Next*. Use Glob/Grep; skip
     `node_modules`, `dist`, `build`, and vendor dirs. Don't parse every `.md` in the tree.
4. **Draft rows** with best-effort Goal/Track/Priority/Owner/Status — these are guesses the user will
   refine. Goal/Track are free-form labels: pick simple ones that fit *this* project; don't import
   another project's taxonomy.
5. **Show the drafted board** with a one-line provenance caption
   (_"Drafted from: TODO.md (N), README roadmap (M), this session (K). Review — nothing written yet."_)
   and **ask the user to confirm or edit before writing**.
6. **On confirm, Write the file** (the only step that writes). If no candidates were found, write the
   starter scaffold below instead.

### Starter scaffold (empty project)
```
# Implementation board

> One Markdown table is the whole board. Add or edit rows by hand, or tell your agent
> "add a board item for X" / "mark 1 done" — `board` and `status` only *render* this file.

**Legend** — **Priority:** 🔴 P0 · 🟠 P1 · P2 · P3 · **Owner:** 🤖 agent · 🧑 you · 👥 both · **Status:** ✅ done · 🟡 in progress · ⚪ to-do · ⏸️ parked · 🔒 blocked · ⛔ skip · **Goal / Track** are free-form labels you pick (Goal = a milestone, Track = a workstream) · ⚠️ = a P0/P1 waiting on you · **Sub-task:** a `↳ Parent.N` row nested under its parent.

| ID | Goal | Track | Item | Priority | Owner | Manual step | Status |
|---|---|---|---|:--:|:--:|---|---|
| 1 | v1 | Docs | **Write the README** — cover install and one basic example. | 🟠 P1 | 🤖 | — | ⚪ to-do |
```

## Rules
- **Only `init` writes.** Rendering (`board`, `board <query>`, `board next/help/by/<column>`) never
  modifies the file; pull state straight from it and don't invent rows.
- Project-agnostic: always operate on the board of the current working directory's project.
- A companion `status` skill renders a condensed view of the same file.
