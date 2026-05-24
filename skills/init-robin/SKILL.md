---
name: init-robin
description: Initialize Robin's identity (CLAUDE.md, SOUL.md, user-profile.md) from evidence — your AI conversation history (Claude Code sessions, Codex sessions, or exported ChatGPT / Claude.ai). Extracts behavioral evidence (not self-description), interviews you for any gaps, drafts the three files, and asks you to review before writing anything to disk. Use at the start of the AI Personal OS course (Workshop 1) or to refresh Robin's identity from new evidence. Always reviews drafts with you before committing.
license: MIT
---

# init-robin

Six phases. Each completes before the next starts. Files only land in `~/.claude/` (or `~/.codex/`) AFTER user approval in Phase 5.

## Phase 1 — Detect evidence

Search in this priority order. Stop at the first source with usable data:

1. `~/.claude/projects/**/*.jsonl` — Claude Code sessions
2. `~/.codex/sessions/` (or platform equivalent) — Codex sessions
3. ChatGPT / Claude.ai export — ask the user where they downloaded it (`.zip` or extracted folder, usually contains `conversations.json`)
4. Current working directory — if the user `cd`'d somewhere intentional

Report to the user:
- Source(s) found
- Count of conversations / sessions
- Date range (oldest → newest)
- Whether the volume is sufficient (threshold: 10+ conversations in the last 90 days)

If multiple sources exist (e.g. CC sessions AND a ChatGPT export), ask the user which to use, or whether to combine.

If no source is found and the user explicitly says they have no history → skip to Phase 3 (interview only).

## Phase 2 — Extract evidence (skip if Phase 1 = insufficient)

Read the most recent 30 sessions (or last 30 days, whichever is smaller).

**DO NOT draft any files yet.** This phase is observation only.

Create `./robin-from-evidence/evidence.md` containing:

### Topics that appear 3+ times
For each topic: name — N session cites — 1 short quote example per cite

### Verbal tics / phrases the user repeats
For each: the phrase — N occurrences — 1 quoted example

### Tasks delegated to AI vs. tasks kept
Two short lists.

### Direct corrections to AI output (these reveal standards)
Quote the correction. Cite the session.

### Open questions that recur
What does the user keep returning to?

**Rules:**
- Every observation must cite a specific session (filename, ID, or date).
- If a category has fewer than 5 examples, write "insufficient evidence" for that category.
- Skip anything that looks personal / sensitive (medical, relationships, finance). Stay work-focused.

Show evidence.md to the user. Ask:
> "Anything wrong? Anything I should drop? Anything missing?"

Wait for confirmation before Phase 4.

## Phase 3 — Interview (for gaps, or if no data at all)

Run only for categories Phase 2 couldn't fill — or all categories if Phase 1 found nothing.

**Rules:**
- ONE question at a time. Wait for the answer before the next question.
- Maximum 8 questions total.
- Each answer informs the framing of the next question.
- For gap-mode: only ask about categories where Phase 2 evidence was thin.

**Question bank** (pick 5-8 based on what's missing):

1. What's the work you do that nobody else on your team can do?
2. What's a recurring task that drains you?
3. What's a sentence you find yourself writing over and over (in emails, docs, Slack)?
4. What are you currently trying to figure out?
5. What's a decision you're sitting with right now?
6. If a new hire joined you tomorrow, what's the first thing you'd tell them about how you work?
7. What would you NEVER let AI do for you?
8. What's one thing you wish you'd written down 6 months ago that you've since forgotten?

Append answers to `evidence.md` under a new section: `### Interview answers` — each Q and A verbatim.

## Phase 4 — Synthesize three files

Read `./robin-from-evidence/evidence.md`. Draft three files in the same `./robin-from-evidence/` directory:

### CLAUDE.md (≤120 lines)
Root identity. Sections:
- Identity (who, what they do)
- How they work (cadence, focus blocks, preferred tools)
- Standards (inferred from observed corrections)
- Current focus
- Tool preferences

### SOUL.md
Voice + personality. Sections:
- Voice description (formal/casual, brief/expansive, etc.)
- 3-5 quoted examples of the user's actual phrasing
- Tone shifts (when do they switch register?)
- Things to avoid (jargon they push back on, etc.)

### user-profile.md
Work patterns and current state. Sections:
- Role / context
- Recurring asks (the 3-5 things they keep needing)
- Open questions they're sitting with
- Tools in their stack

**Rules for every claim:**
- Tag each line/bullet with confidence: `[HIGH]` (3+ evidence cites), `[MED]` (2 cites), `[LOW]` (1 cite or interview-only).
- If a section has no [HIGH] or [MED] evidence → leave it empty with `// no evidence — fill in manually` rather than inventing.
- Never generalize beyond the evidence. Be specific and concrete.
- Skip sensitive personal content even if it appears in evidence.

## Phase 5 — Review with user

Show the user the three drafts. Ask three short questions:
- "Anything obviously wrong I should fix?"
- "Anything to remove?"
- "Anything important I missed?"

Apply the user's edits. Re-show. Iterate until the user approves.

## Phase 6 — Commit

When the user approves, run (or ask the user to run):

```bash
# Claude Code
mv ./robin-from-evidence/CLAUDE.md     ~/.claude/CLAUDE.md
mv ./robin-from-evidence/SOUL.md       ~/.claude/SOUL.md
mv ./robin-from-evidence/user-profile.md ~/.claude/user-profile.md

# Codex
mv ./robin-from-evidence/CLAUDE.md     ~/.codex/AGENTS.md
mv ./robin-from-evidence/SOUL.md       ~/.codex/SOUL.md
mv ./robin-from-evidence/user-profile.md ~/.codex/user-profile.md
```

Three more files complete Robin's birth certificate (course-goals, goals-2026, achievements). These can't be inferred from conversations — see the homework prompts in `gists/homework-prompts.md`.

## Notes

- Default scope: last 30 days of conversations. User can override: "extend to 90 days" or "look at just last week."
- Everything stays local. Conversation content is read from local files; the only network calls are the model API calls Claude Code / Codex would make anyway.
- Power-user variant: "compare patterns between projects" — point at different project directories, contrast the profiles.
- Re-run anytime: when the user has months of new evidence, `/init-robin` regenerates from the richer signal.
