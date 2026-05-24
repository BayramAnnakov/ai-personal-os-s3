---
name: finish-robin
description: Complete Robin's birth certificate. Runs three short interviews to produce course-goals.md, goals-2026.md, and achievements.md — the three files /init-robin cannot infer from conversation history because they require explicit declaration, not behavioral evidence. Use after /init-robin (Workshop 1 homework, ~15-20 min total), or invoke a single section to refresh just one file. Re-run goals-2026 annually; re-run achievements quarterly.
license: MIT
---

# finish-robin

Three short interviews. Each produces ONE file in `~/.claude/` (Codex: `~/.codex/`).

Total: ~15-20 min for all three. ~5 min for a single section.

## Dispatch

Look at how the user invoked this skill:

- **No additional text** → run all three sections in order
- **"course-goals"** → run Section 1 only
- **"goals-2026"** or **"2026"** → run Section 2 only
- **"achievements"** → run Section 3 only

After each section completes, save the file and continue (or stop, if single-section).

---

## Section 1 · course-goals.md (~5 min)

Ask the user 3 questions, **ONE AT A TIME**. Wait for each answer before the next.

1. What ONE specific outcome would make this course a 10/10 for you?
2. What is your single biggest constraint — time, technical depth, or team buy-in?
3. What will you have built by July 19, 2026 that you don't have today?

After the third answer, write a one-line synthesis: *"You want X, your bottleneck is Y, your end state is Z."*

Save to `~/.claude/course-goals.md` (Codex: `~/.codex/course-goals.md`):

```markdown
# Course goals — AI Personal OS S3

## Outcome
<user's answer to Q1>

## Biggest constraint
<user's answer to Q2>

## End state (by Jul 19, 2026)
<user's answer to Q3>

## Synthesis
<one-line synthesis>
```

Keep the whole file under 40 lines.

---

## Section 2 · goals-2026.md (~5 min)

Ask the user 3 questions about 2026, **ONE AT A TIME**:

1. What is ONE professional outcome you want to be true on Dec 31, 2026?
2. What is ONE personal outcome you want to be true?
3. What is ONE thing you want to STOP doing this year?

For each answer, ask a one-line follow-up: *why does this matter, and what is the observable metric?*

Save to `~/.claude/goals-2026.md` (Codex: `~/.codex/goals-2026.md`):

```markdown
# Goals 2026

## Professional
**Outcome:** <Q1 answer>
**Why:** <follow-up answer>
**Observable metric:** <follow-up answer>

## Personal
**Outcome:** <Q2 answer>
**Why:** <follow-up answer>
**Observable metric:** <follow-up answer>

## Stop doing
**Outcome:** <Q3 answer>
**Why:** <follow-up answer>
**Observable metric:** <follow-up answer>
```

Keep the whole file under 40 lines.

---

## Section 3 · achievements.md (~5-10 min)

Ask the user to list 5-10 wins from the last 3 years — anything they are proud of, professional or personal. Don't filter; if they include something small, keep it.

For each win, ask a one-line follow-up: *what made it possible?*

Save to `~/.claude/achievements.md` (Codex: `~/.codex/achievements.md`):

```markdown
# Achievements (last 3 years)

- **<title>** · <date or year> · <what made it possible>
- **<title>** · <date or year> · <what made it possible>
- ...
```

---

## After all three sections complete

Tell the user:

> Robin's birth certificate is complete (6 files). Test it now in a fresh session:
>
>     What do you know about me now? List 5 things, plus one risk in my CLAUDE.md.
>
> Compare Robin's answer to before /init-robin. The delta is the point of W1.

## After a single section completes

Tell the user which file updated. Offer to refresh another section, or stop.

## Notes

- Re-run anytime. `/finish-robin goals-2026` at the start of each calendar year. `/finish-robin achievements` quarterly.
- These three files complement the three from `/init-robin`. Together: Robin's complete birth certificate.
- Sister skill: `/init-robin` (evidence-based identity from conversation history).
