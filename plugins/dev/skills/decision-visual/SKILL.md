---
name: decision-visual
description: "After any long reply (roughly 250+ words, several sections, or a report of finished work), or any reply that leaves the user something to decide, also render a simple scratch HTML visual inline (show_widget) showing what the user needs to know and the decisions they need to take, with answer buttons. Use every time, unasked; skip only for short answers and pure chat."
user-invocable: true
---

# /dev:decision-visual — Decision visual

The user skims long replies. After any long text reply, render one simple
inline visual that answers two questions at a glance: **what do I need to
know**, and **what do I need to decide**. It's a scratch visual: quick,
plain, not a polished artifact.

## When

- Use it when the reply is long (about 250 words or more, several sections
  or lists, or a report of finished work), or when it ends with a choice,
  an approval, or open follow-ups.
- Skip it for short answers, quick confirmations, and plain conversation.
- Make one visual per reply, rendered after the text.

## How

1. If you haven't called it yet in this conversation, call
   `mcp__visualize__read_me` with `modules: ["interactive"]` silently.
   Don't mention that call.
2. Call `mcp__visualize__show_widget` with an HTML fragment (no
   `<html>`/`<body>`) using the host CSS variables. That way it renders in
   the same style as the host and works in light and dark mode. Title it in
   snake_case after the topic, for example `board_migration_summary`.
3. Don't repeat the visual's content in text after it. One closing line
   pointing at it is enough.

If those tools aren't available (another client), end the reply with a
short markdown block instead: **Know** (up to 5 bullets) and **Decide**
(numbered questions, your recommendation first).

## Layout, top to bottom (leave out empty sections)

1. **Status pills.** Up to 4 short facts: what was done or verified
   (success color), and what's not done or not verified (warning color),
   for example "Pushed as fb46a28" or "Not deployed".
2. **What you need to know.** Two cards side by side: what is true or
   works now, and what's still open, manual, or a limit. Each card has up
   to 5 bullets, one line each, with a Tabler outline icon.
3. **Optional mini picture.** Only when a shape explains it faster than
   words: a few indented rows, a before/after pair, or 3 to 4 steps.
   Leave it out otherwise.
4. **Decisions for you.** One card per decision, numbered. Each has:
   - the question, phrased the way the user would answer it;
   - one or two lines of context, ending with "Recommended: ...";
   - 1 to 3 buttons that call `sendPrompt('<the full answer as the user
     would type it>')`. The recommended answer comes first, and each label
     ends with ↗.

   With nothing to decide, show one quiet line instead: "Nothing to decide
   right now."

## Rules

- Write in plain words: the user's language, not internal jargon. Each
  bullet should make sense on its own.
- Never invent facts, statuses, or decisions. The visual reflects the
  reply exactly.
- Keep it short: about 20 lines of visible content at most.
- Follow the visualize design rules: sentence case, weights 400 and 500
  only, 0.5px borders, 12px card radius, and no gradients, shadows, emoji,
  or `position: fixed`.
- Start with a visually hidden `<h2 class="sr-only">` that sums up the
  visual in one sentence.
- Give every `sendPrompt` answer enough detail that it works as the user's
  next message on its own.
