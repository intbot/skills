---
name: sharpen
description: Rewrite a rough, unclear prompt into a sharp one for an AI agent,
  then confirm the final with the user before they send it. Use when the user
  types /sharpen.
disable-model-invocation: true
---

Rewrite the user's rough prompt into a clear one for an AI agent, then confirm
before they send it.

## Input

The prompt to sharpen = the `/sharpen` argument, or the message they just wrote.
If it's genuinely unreadable (you can't tell the goal), ask **one** clarifying
question first — otherwise rewrite straight away.

## Rewrite

Produce a sharper version that:

- States the goal and the concrete ask up front.
- Splits multiple requests into ordered, separate tasks.
- Names implied constraints, context, and the target (repo, file, agent).
- Cuts filler; keeps it copy-paste ready.

Then add one line — **"What I changed:"** — naming the main fixes.

## Confirm (always)

Ask via **AskUserQuestion**, options:

- ✅ Use this **(Recommended)**
- ✏️ Tighten further
- ➕ Add missing context
- 💬 Chat about what I meant

(AskUserQuestion adds an `Other` free-text option.) Loop until they pick
**Use this**, then print the final prompt as a clean block by itself, ready to
paste — nothing after it.
