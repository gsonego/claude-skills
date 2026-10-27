---
name: clarify
description: Interview the user one question at a time, with selectable answer options, to resolve doubts, ambiguities and loose ends about whatever is being studied or planned. Use when the user asks to be asked questions ("clarify", "interview me", "grill me", "ask me what you need"), or when a study or planning session still has open unknowns before acting.
argument-hint: "[topic]"
allowed-tools: AskUserQuestion, Read, Grep, Glob
---

Run an interview that removes every material doubt about the topic, so planning or implementation can start on solid ground.

Topic: `$ARGUMENTS` (if empty, use the subject of the current session; if there is none, ask what to clarify).

## Rules

1. **Read before asking.** Check the code, docs and conversation first. Never ask something you can answer yourself. A question is only for a decision that belongs to the user.
2. **List the unknowns** before the first question: ambiguities, assumptions, missing decisions, dependencies, edge cases, risks. Rank them by impact, meaning how much rework a wrong guess would cause.
3. **One question per turn**, using `AskUserQuestion` with a single question:
   - 2–4 options, mutually exclusive, each with its trade-off in the description.
   - Your recommendation first, labelled "(Recommended)". Never present options as equal when you have a view.
   - Free text is always available through "Other"; do not add an option for it.
4. **Complex item → offer to explain.** When a question needs background to be answered well, give up to 3 real options plus a last option, "Explain this first". If chosen, explain briefly and plainly, then ask the same question again.
5. **Re-evaluate after every answer.** An answer can close items, open new ones or make pending questions moot. Update the list, drop what is resolved, and never re-ask or re-litigate a decision already made.
6. **Ask highest impact first.** Do not interrogate for the sake of it: when what is left is low impact, state the default you will assume instead of asking.
7. **Stop when no material unknown remains.** Do not pad the interview.
8. **Do not implement** anything during the interview. No edits, no commands that change state.

## Finish

Close with a short summary:

- **Decisions** made, one line each.
- **Assumed defaults**, which the user can still change.
- **Residual risks or open points**, if any.
- **Suggested next step.**

Write in the user's language. If `AskUserQuestion` is not available, ask the same single question as plain text with numbered options, one question per message.
