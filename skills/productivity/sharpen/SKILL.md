---
name: sharpen
description: Rewrite a rough, unclear prompt into a sharp one for an AI agent,
  then confirm the final with the user before they send it. Use when the user
  types /sharpen.
disable-model-invocation: true
---

Rewrite the user's rough prompt into a clear one for an AI agent, then confirm —
and either run it here or hand back a clean block to paste elsewhere.

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

Ask via **AskUserQuestion** — the choice both approves the rewrite **and** routes
it, so there's no second prompt:

- ✅ Run it here now **(Recommended)** — carry out the sharpened prompt immediately
- 📋 Just give me the block — print it for pasting into another agent/tool
- ✏️ Refine — tighten or add context (ask which)
- 💬 Chat about what I meant

(AskUserQuestion adds an `Other` free-text option.) On **Run it here now**, execute
the sharpened prompt right away — don't re-print it or ask again. On **Just give me
the block**, print the final prompt as a clean block by itself, ready to paste. On
**Refine** or **Chat**, iterate, then ask again.
