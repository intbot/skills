---
name: quiz-me
description: Quiz the user about a plan, design, or topic using selectable
  multiple-choice prompts (the AskUserQuestion chip UI) instead of freeform
  questions — a friendlier take on /grill-me. Use when the user types /quiz-me
  or asks to be quizzed on something.
---

You're running a quiz. Ask through the **AskUserQuestion** tool so every answer
is a selectable chip, not prose.

## Arguments — `/quiz-me <type> [--test] [topic]`

- **`<type>`** (first token): `mul` multiple-choice · `tf` true/false (yes/no) ·
  `scale` rate / prioritize (Low→Critical) · `mix` best fit per question.
  Missing or unrecognized → default to **`mul`** and treat that token as topic text.
- **`--test`**: graded mode (below). Without it you're in **review** mode.
- **remaining words**: the topic. No topic → quiz the plan/design in the current
  conversation.

## Every question (both modes)

- **One question at a time.** Wait for the answer before the next — batching is
  bewildering.
- 2–4 options in the chosen format, plus a **`💬 Chat about this`** option.
  (AskUserQuestion auto-adds an `Other` free-text option — don't add it yourself.)
- If a question is answerable by reading the codebase, **explore the codebase
  instead** of asking.
- If they pick **`💬 Chat about this`**, leave the quiz, discuss the point freely,
  then resume with the next question.
- Default to **3–5 questions** on the biggest decisions, then wrap up — unless the
  topic clearly needs more, or the user asks to keep going.

## Review mode (default)

Walk the design tree, resolving dependencies one at a time. Options are candidate
decisions; mark your recommended one **`(Recommended)`** and list it first. When
they pick, **log it as a settled decision and move on** — don't act on it yet. At
the end, summarize every decision in one list.

## Test mode (`--test`)

Options contain exactly one correct answer — don't reveal which. After each answer,
say right or wrong with a one-line why, and keep a running score. After the last
question, give the final score and the weakest areas.
