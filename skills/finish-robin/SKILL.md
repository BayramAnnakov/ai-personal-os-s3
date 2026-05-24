---
name: finish-robin
description: Complete Robin's birth certificate. Runs three short interviews to produce course-goals.md, goals-2026.md, and achievements.md — the three files /init-robin cannot infer from conversation history because they require explicit declaration, not behavioral evidence. Use after /init-robin (Workshop 1 homework, ~15-20 min total), or invoke a single section to refresh just one file. Re-run goals-2026 annually; re-run achievements quarterly.
license: MIT
---

# finish-robin

Three short interviews. Each produces ONE file in the workspace (`./robin-from-evidence/`). Install to `~/.claude/` (Codex: `~/.codex/`) is opt-in at the end, with backup-diff-confirm if files already exist.

Total: ~15-20 min for all three. ~5 min for a single section.

## Start — introduce yourself and get approval

Before any interview, greet the user and explain what's about to happen. Adapt this script naturally — do not read it verbatim:

> Hi — I'll run 3 short interviews to finish Robin's birth certificate:
>
> 1. **course-goals** (~5 min) — what you want from this course
> 2. **goals-2026** (~5 min) — what you want true on Dec 31, 2026
> 3. **achievements** (~5-10 min) — wins from the last 3 years
>
> Total ~15-20 min. Each section saves a draft to `./robin-from-evidence/` for your review. Nothing installs without your approval.
>
> Run all 3 now, or pick one (`course-goals` / `goals-2026` / `achievements`)?

Wait for explicit confirmation before starting the first interview. If the user named a single section, run only that.

**Language.** If the user replies in Russian (or any non-English language), switch to that language for all interview questions, follow-ups, and summaries. Match the user's language. The files themselves stay in English (course material standard), but the conversation mirrors the user.

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

Save to `./robin-from-evidence/course-goals.md` (creates the directory if missing):

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

Save to `./robin-from-evidence/goals-2026.md`:

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

Save to `./robin-from-evidence/achievements.md`:

```markdown
# Achievements (last 3 years)

- **<title>** · <date or year> · <what made it possible>
- **<title>** · <date or year> · <what made it possible>
- ...
```

---

## Install (workspace-first, never destructive)

The drafts are saved in `./robin-from-evidence/`. Installing them means making Robin read them in future sessions. Mirror the pattern from `/init-robin` Phase 6:

### Default — workspace install

```bash
cp ./robin-from-evidence/course-goals.md  ./course-goals.md
cp ./robin-from-evidence/goals-2026.md    ./goals-2026.md
cp ./robin-from-evidence/achievements.md  ./achievements.md
```

(Only copy files for sections that ran.) If a target already exists, ask first and show a diff.

### Optional — global install

Ask: *"Install globally so Robin reads these in every folder?"* If yes, target `~/.claude/<file>.md` (Codex: `~/.codex/<file>.md`).

- **No existing file** → `cp` the draft into place. Confirm.
- **Existing file** → STOP. Save backup `cp ~/.claude/<file>.md ~/.claude/<file>.md.bak.$(date +%Y%m%d-%H%M%S)`, show diff, ask: *"Replace, merge manually, or skip?"*

Never `mv` from the staging directory.

## After all three sections complete

Tell the user:

> Robin's birth certificate is complete (6 files). Test it now in a fresh session:
>
>     What do you know about me now? List 5 things, plus one risk in my CLAUDE.md.
>     (Codex users: substitute "AGENTS.md" for "CLAUDE.md" above.)
>
> Compare Robin's answer to before /init-robin. The delta is the point of W1.

## After a single section completes

Tell the user which file updated. Offer to refresh another section, or stop.

## Notes

- Re-run anytime. `/finish-robin goals-2026` at the start of each calendar year. `/finish-robin achievements` quarterly.
- These three files complement the three from `/init-robin`. Together: Robin's complete birth certificate.
- Sister skill: `/init-robin` (evidence-based identity from conversation history).
