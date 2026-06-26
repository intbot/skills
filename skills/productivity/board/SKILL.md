---
name: board
description: Render or initialize a project's implementation board (one Markdown table — ID · Goal · Track · Item · Priority · Owner · Manual step · Status). `board` renders with a composable query (space=AND, `,`=OR, `!`=NOT over Goal/Track/Priority/State/Owner), plus `board next`, `board explain [query]` (detail cards with an inferred context line per item), `board by <column>`, `board <column>` (list a column's values), and `board help`. `board init [docs|root|internal|<path>]` scaffolds a board. Rendering is read-only; only `init` writes.
disable-model-invocation: true
---

Render — or, with `init`, create — this project's implementation board.

## Locating the board

First existing of (use Read, not shell-glob): `internal/board.md`, `.claude/board.md`, `board.md`,
`docs/board.md`. None exists → say so and tell the user to run `board init` (don't scaffold here).

## `board [query]` — render, READ-ONLY

Render the **Legend** + table, all columns (**ID | Goal | Track | Item | Priority | Owner | Manual
step | Status**), file order (done-first, then priority). **Wrap long cells; never truncate Item or
Status.** Item is `**Bold title** — description` (em-dash, not `<br>`, which terminal tables ignore).

**Sub-tasks:** a follow-up nests under its parent as `↳ Parent.N` (e.g. `↳ T-D.1`); `·` in Goal/Track
= inherits parent; it keeps its own Priority/Owner/Status. Keep each sub-task immediately under its
parent; render `↳`/`·` as-is. `⏸️ parked` = a follow-up deferred by choice.

### Reading the argument — resolve in order

1. **`help`** → print only this line: `board [query] · space=AND ,=OR !=NOT over goal·track·p0-p3·state·mine/yours · board explain [query] · board <column> · board by <column> · board next · board init`
2. **`next`** → live priorities (rows not done/parked/deferred/skip), ordered 🔴→🟠→P2→P3 then file order, capped ~8.
3. **`explain [query]`** → verbose detail view (see below).
4. **`by <column>`** (goal/track/owner/priority/status) → regroup every non-excluded row into one `###` table per distinct value, each group in priority order.
5. **a single column name** (goal/track/owner/priority/status) → don't render the table; list that column's distinct values with row counts, busiest first (e.g. `Presence (33) · Parity (3) · Moat (3)`).
6. **anything else → a query** (grammar below).
7. **no argument** → the full board.

### Query grammar

Space-separated tokens **AND**; a `,` inside a token = **OR**; a leading `!` = **NOT**.
Case-insensitive. A token matches a row's:

- **Goal** / **Track** (e.g. `presence`, `traffic`), or **Priority** (`p0`–`p3`).
- **State** — a literal status word (`live`, `parked`, …) or an alias: `done`=✅|🟢 · `todo`=⚪ ·
  `active`/`wip`=🟡 · `parked`=⏸️ · `blocked`/`deferred`=🔒 · `skip`=⛔.
- **Owner** — `mine`/`me`=🤖|👥 · `yours`/`you`=🧑|👥.
- **Parent** — a bare ID (`T-D`) matches that row **and** its `↳ T-D.N` sub-tasks.
- else **substring** over ID / Goal / Track / Item.

`!live` drops only ✅ live; `!done` drops everything finished (✅ and 🟢).
Examples: `presence !live` · `traffic,earned !done p1` · `mine !parked` · `T-D`.

### `explain [query]` — verbose detail, READ-ONLY

Select the same rows a query would (`explain` alone = whole board; `explain moat` / `explain T-D` =
that subset). Instead of a table, render **one card per item**:

```
<ID> · <Goal>/<Track> · <Priority> · <Owner> · <Status>[ · ⚠️ <Manual step>]
<full Item title — description>
  ↳ context: 1–2 plain-language lines on what this item actually is — inferred from the
    codebase, docs, and conversation. End with "(inferred)". Don't invent specifics you
    can't ground. Never write this back to the file.
```

Keep sub-tasks under their parent card. This costs more tokens by design — only on request.

### Output rules

- **Sub-task cohesion:** if a parent matches, include its `↳` sub-tasks (and a matched sub-task pulls
  in its parent); keep them adjacent.
- Filtered / `by` / `next` / `explain` renders: name the active filter on one line above.
- **Always end with a tally:** `_N done · M to-do · K parked · J deferred/skip_`
  (done=✅|🟢 · to-do=⚪|🟡 · parked=⏸️ · deferred/skip=🔒|⛔).

## `board init [target]` — create the board, the ONLY write path

Use when there's no board yet (or the user explicitly asks to set one up).

1. **Target** from the token after `init`: `docs`→`docs/board.md` · `root`→`board.md` ·
   `internal`→`internal/board.md` · `claude`/nothing→`.claude/board.md` · else a literal path.
   If under `.claude/`, note it's usually gitignored (suggest `docs/board.md` or `board.md` to commit).
2. **Never clobber.** If a board exists at any discovery path, show it and stop — overwrite only on
   explicit confirmation.
3. **Gather rows** from: this conversation (work in progress; nest a mid-task bug/improvement as a `↳`
   sub-task), and high-signal project markdown — GitHub task lists (`- [ ]`→⚪, `- [x]`→✅),
   `TODO*`/`ROADMAP*`/`PLAN*`/`BACKLOG*` files, and README/docs *Roadmap/TODO/Planned/Coming soon/Next*
   sections (Glob/Grep; skip node_modules/dist/build/vendor).
4. **Draft** best-effort Goal/Track/Priority/Owner/Status (guesses to refine); pick simple labels that
   fit this project — don't import another project's taxonomy.
5. **Show the draft** with a provenance caption (_"Drafted from: TODO.md (N), README roadmap (M), this
   session (K). Nothing written yet."_) and **ask to confirm or edit before writing.**
6. **On confirm, Write the file** (the only write step). No candidates → write the scaffold below.

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
- **Only `init` writes.** Every render (`board`, queries, `next`/`help`/`by`/`explain`/`<column>`)
  reads the file and never modifies it; don't invent rows.
- Project-agnostic: always operate on the current project's board.
- A companion `status` skill renders a condensed view of the same file.
