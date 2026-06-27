---
name: board
description: Render or initialize a project's implementation board (one Markdown table — ID · Goal · Track · Item · Priority · Owner · Manual step · Status). `board` renders with a composable query (space=AND, `,`=OR, `!`=NOT over Goal/Track/Priority/State/Owner) plus a progress bar + per-Goal completion, and supports `board next`, `board explain [query]`, `board sync`, `board archive [query]`, `board by <column>`, `board <column>`, `board help`. A `@<name>` token targets a separate named board (`board.<name>.md`); `board boards` lists them. `board init [docs|root|internal|<path>|@<name>]` scaffolds a board. Rendering is read-only; only `init`, `sync`, and `archive` write.
disable-model-invocation: true
---

Render — or, with `init`, create — this project's implementation board.

## Locating the board

First existing of (use Read, not shell-glob): `internal/board.md`, `.claude/board.md`, `board.md`,
`docs/board.md`. None exists → say so and tell the user to run `board init` (don't scaffold here).

**Named boards.** A `@<name>` first token targets a separate board file **`board.<name>.md` in the
default board's directory** (e.g. `internal/board.eb1a.md`), with its own archive
`board.<name>.archive.md`. Everything after `@<name>` runs exactly as below, against that file
(`board @eb1a next`, `board @eb1a explain X`, `board @eb1a sync`); `board @<name>` alone = that
board's full render. No `@<name>` → the default board. Each named board is fully independent — its own
tally, progress, and archive, never co-mingled. Use named boards for genuinely separate domains; for
facets of one effort, use a Track + a query instead.

## `board [query]` — render, READ-ONLY

Render the **Legend** + table, all columns (**ID | Goal | Track | Item | Priority | Owner | Manual
step | Status**), file order (done-first, then priority). **Wrap long cells; never truncate Item or
Status.** Item is `**Bold title** — description` (em-dash, not `<br>`, which terminal tables ignore).

**Sub-tasks:** a follow-up nests under its parent as `↳ Parent.N` (e.g. `↳ T-D.1`); `·` in Goal/Track
= inherits parent; it keeps its own Priority/Owner/Status. Keep each sub-task immediately under its
parent; render `↳`/`·` as-is. `⏸️ parked` = a follow-up deferred by choice.

### Reading the argument — resolve in order

If the first token is **`@<name>`**, set the target board to `board.<name>.md` (see *Named boards*),
drop that token, and resolve what remains below (nothing remaining = full render of that board).

1. **`boards`** → list the default board + every `board.*.md` (excluding `*.archive.md`) in the board
   directory, each with its name, file, and `done/total (pct)`.
2. **`help`** → print only this line: `board [query] · space=AND ,=OR !=NOT over goal·track·p0-p3·state·mine/yours · board @<name> · board boards · board explain [query] · board sync · board archive · board <column> · board by <column> · board next · board init`
3. **`next`** → live priorities (rows not done/parked/deferred/skip), ordered 🔴→🟠→P2→P3 then file order, capped ~8.
4. **`explain [query]`** → verbose detail view (see below).
5. **`sync`** → reconcile this conversation into the board (writes — see below).
6. **`archive [query]`** → move done rows out to `board.archive.md` (writes — see below).
7. **`archived`** → render the archive file, read-only, newest section first.
8. **`by <column>`** (goal/track/owner/priority/status) → regroup every non-excluded row into one `###` table per distinct value, each group in priority order.
9. **a single column name** (goal/track/owner/priority/status) → don't render the table; list that column's distinct values with row counts, busiest first (e.g. `Presence (33) · Parity (3) · Moat (3)`).
10. **anything else → a query** (grammar below).
11. **no argument** → the full board.

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

### `board sync` — reconcile conversation → board, WRITES (with confirm)

Update the (targeted) board from what happened in this conversation:

1. Scan the conversation for board-relevant changes: tasks finished, new follow-ups/bugs discovered,
   status or scope changes.
2. Map each to the board — flip a row's Status, add a `↳ Parent.N` sub-task under the item it came
   from, or add a new row for genuinely new work. Ground every change in the conversation; don't invent.
3. Show the changes as a diff (e.g. `L3.1 ⚪→🟡 · +↳ T-D.2 "…" · +ROW …`) with a one-line provenance,
   and ask to confirm or edit.
4. On confirm, edit the board file in place — preserve Legend, columns, sub-task nesting, and order.

### `board archive [query]` — retire done rows, WRITES (with confirm)

Move completed rows out of the (targeted) board into its archive — `board.archive.md` for the default,
`board.<name>.archive.md` for a named board:

1. Select **✅/🟢 done** rows matching the query (`archive` alone = all done rows; `archive moat` =
   done Moat rows). Never select live-but-unfinished, ⚪ to-do, ⏸️ parked, or 🔒 deferred rows.
2. Carry each selected row's done sub-tasks with it; never orphan a `↳` child, or strand a child whose
   parent is leaving (move the parent too, or keep both).
3. Show the rows to be moved and the destination, and ask to confirm.
4. On confirm: **append** them to the archive under a `## Archived <today's date, YYYY-MM-DD>` heading
   (create the file with the board's Legend + table header if it doesn't exist), then **remove** them
   from the board file. Preserve columns and sub-task nesting on both sides.

`board archived` renders the targeted board's archive read-only, newest section first.

### Output rules

- **Sub-task cohesion:** if a parent matches, include its `↳` sub-tasks (and a matched sub-task pulls
  in its parent); keep them adjacent.
- Filtered / `by` / `next` / `explain` renders: name the active filter on one line above. For a named
  board, prefix the render with its `@<name>`.
- **Always end with a tally:** `_N done · M to-do · K parked · J deferred/skip_`
  (done=✅|🟢 · to-do=⚪|🟡 · parked=⏸️ · deferred/skip=🔒|⛔).
- **Progress (full / unfiltered board and `by goal` only):** below the tally, add
  - a 10-cell completion bar — `▓▓▓▓▓▓░░░░ 31/47 (66%) done` (filled = done ✅|🟢; denominator = all
    rows except ⛔ skip), and
  - per-Goal completion — `Presence 28/33 · Parity 3/3 · Moat 0/4` (done / non-skip total per Goal).

  On filtered / `next` / `explain` renders, show only the plain tally (progress reflects the whole board).

## `board init [target]` — create a board (a write path)

Use when there's no board yet (or the user explicitly asks to set one up).

1. **Target** from the token after `init`: `@<name>`→`board.<name>.md` in the default board's directory
   (a named board) · `docs`→`docs/board.md` · `root`→`board.md` · `internal`→`internal/board.md` ·
   `claude`/nothing→`.claude/board.md` · else a literal path. If under `.claude/`, note it's usually
   gitignored (suggest `docs/board.md` or `board.md` to commit).
2. **Never clobber.** If the target board already exists, show it and stop — overwrite only on explicit
   confirmation.
3. **Gather rows** from: this conversation (work in progress; nest a mid-task bug/improvement as a `↳`
   sub-task), and high-signal project markdown — GitHub task lists (`- [ ]`→⚪, `- [x]`→✅),
   `TODO*`/`ROADMAP*`/`PLAN*`/`BACKLOG*` files, and README/docs *Roadmap/TODO/Planned/Coming soon/Next*
   sections (Glob/Grep; skip node_modules/dist/build/vendor). For a `@<name>` board, gather only rows
   that fit that domain.
4. **Draft** best-effort Goal/Track/Priority/Owner/Status (guesses to refine); pick simple labels that
   fit this board — don't import another board's taxonomy.
5. **Show the draft** with a provenance caption (_"Drafted from: TODO.md (N), README roadmap (M), this
   session (K). Nothing written yet."_) and **ask to confirm or edit before writing.**
6. **On confirm, Write the file** (writes only on confirm). No candidates → write the scaffold below.

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
- **Only `init`, `sync`, and `archive` write.** Every render (`board`, queries,
  `next`/`help`/`by`/`explain`/`archived`/`boards`/`<column>`) reads the file and never modifies it;
  don't invent rows.
- Project-agnostic: always operate on the current project's board(s).
- A companion `status` skill renders a condensed view of the same file.
