---
name: atomize
description: Turn content into atomic Zettelkasten notes in ~/vault/atomic/ and keep ~/vault/index.md current. THREE modes by who drafts the note — FILE-IT (you wrote the insight, I only file it), DRAFT-IT (I draft one note from your scattered raw material, you edit a line and approve), IMPORT (I bulk-atomize a source you already authored — transcript, export, daily-logs — you elaborate at least one note before it commits). Connections are PROPOSE-then-DECIDE — I suggest [[wikilinks]] with reasons and flag contradictions, you choose which to keep; I never auto-write a link. Use on /atomize, "atomize this", "break this into notes", or to import an export/transcript/article.
allowed-tools:
  - AskUserQuestion
  - Write
  - Read
  - Edit
  - Bash
  - Glob
  - Grep
license: MIT
---

# atomize

The ingestion frontend for your Knowledge OS. Raw input → atomic notes in `~/vault/atomic/` → connections you approve → `~/vault/index.md` kept current.

**The principle it encodes** (post 1711): *«Писать заметки — это и есть думать.»* The LLM does the toil — finding, filing, dedup, index upkeep. **You** stay in the thinking: you write or shape the insight, and **you decide what connects to what.** Deciding the connection is thinking, not bookkeeping — so connections are always proposed, never auto-written.

**Language:** mirror the user's language in conversation; write note bodies in whatever language the user thinks/writes in. The structure (frontmatter keys, section headers) stays as below.

---

## The three modes — a spectrum of *who drafts the note*

Ask ONE question up front (`AskUserQuestion`) unless the user already said which mode:

| Mode | Input | Who writes the body | The human's act |
|---|---|---|---|
| **FILE-IT** | a note the user already wrote (one idea, their words) | the user — you do NOT rewrite it | they wrote it |
| **DRAFT-IT** | scattered raw material about ONE idea (3 threads, a voice memo, a meeting snippet) | you draft ONE candidate note in their voice | they **edit ≥1 sentence** before it saves |
| **IMPORT** | a long source they already authored (transcript, export, daily-logs) → many notes | you draft each note in their voice | they **elaborate ≥1 note** before anything commits |

The constant across all three: **the human always engages** (writes, edits, or elaborates), and **the human always decides the connections.** That is non-negotiable — it is the whole point.

DRAFT-IT is the middle path for the common real case: *"I half-said this across three places and want the crisp version."* You draft; they engage by editing. (This is the honest compromise with the LLM-Wiki view: let the model draft, but the human must touch it.)

---

## Procedure

Keep it to **one input → propose everything at once → one review round → commit.** Do NOT hold a per-note conversation.

### 1 · INPUT
Get the mode (ask if unknown), the content (pasted text, a file path, or a short description), and the `source` (`self` for the user's own insight; otherwise the transcript / export / book / URL it came from).

### 2 · DRAFT (varies by mode)
- **FILE-IT** — use the user's note as the body, untouched. One idea per note; if they handed you two ideas, split into two and say so.
- **DRAFT-IT** — synthesize ONE candidate atomic note from the raw material, in the user's voice: ONE concept, 2-5 sentences, their words (not copy-pasted).
- **IMPORT** — extract the atomic ideas: ONE concept per note, 2-5 sentences each, rewritten in the user's voice (NOT copy-pasted). A paragraph → 2-5 notes; a transcript → 5-12. Quality over count; don't atomize filler.

### 3 · PROPOSE CONNECTIONS (all modes — never write links yet)
For each draft note, `ls ~/vault/atomic/*.md` and `grep -ri "<concept keyword>" ~/vault/atomic/` to find related notes by *concept*, not just keyword. Then assemble a **proposal**:
- 0-5 candidate `[[wikilinks]]` per note, each with a one-line **why it connects**.
- Any candidate **contradiction**, flagged: `[[note-id]] — CONTRADICTS: you argued the opposite here.`
- Do **not** write any link into a file yet. These are candidates for the human to rule on.

### 4 · REVIEW GATE (one round — this is where the human thinks)
Print all draft notes + all proposed connections in one view, numbered. **Then STOP and wait for the user's reply — do not write ANY file or proceed to step 5 until they respond.** Use `AskUserQuestion` (or an unmistakable numbered prompt) so the decision can't be skipped. Then:
- **Connections — PROPOSE-then-DECIDE.** Ask the user which candidate links to keep, drop, or add, e.g. *"keep 1,3; drop 2; add a link from note B to [[X]]."* You write **only** approved links. For kept links, use the user's "why" if they give one; otherwise keep your one-liner.
- **DRAFT-IT gate.** Show the drafted note and require the user to **edit at least one sentence** before you save it. If they say "looks good, save it," push back once: *"Change one line first — even a word. The edit is the thinking."* Then save.
- **IMPORT gate (forced elaborate).** Do **NOT** write any files until the user has **elaborated at least one note** — edited a line, sharpened a claim, or added their own sentence. Say so plainly: *"Pick one note and make it sharper or truer in your words — I won't file the batch until you've touched one. That touch is how an import becomes yours."* (For big batches, require one elaboration per ~20 notes.)
- **FILE-IT.** No body gate — they wrote it. Just settle the connections.

### 5 · WRITE
Get today's date (`date +%Y-%m-%d`). Number the FIRST new note `NNN` = (count of existing `~/vault/atomic/YYYY-MM-DD_*.md`) + 1, zero-padded (001, 002…), and **increment NNN for each subsequent note in the batch** so you never overwrite `_001`. Write to `~/vault/atomic/YYYY-MM-DD_NNN.md` using the template below. Write **only** the connections the user approved.

```
---
id: YYYY-MM-DD_NNN
title: "[Short label, not a sentence]"
created: YYYY-MM-DD
tags: [tag1, tag2]
source: "[transcript / export / URL — or 'self']"
---

# [Title]

[2-5 sentences, ONE idea, in the user's voice.]

## Connections

- [[related-note-id]] — [why this connects, approved by the user]

## Source

[1-2 lines: where it came from. Skip if source is 'self'.]
```

### 6 · UPDATE INDEX
Open `~/vault/index.md`. Add each new note under the right topic heading (create the heading if the topic is new). Keep it a clean map of topics → anchor notes. This is the navigation layer the user's identity file `@import`s.

### 7 · SUMMARY (print only this — then stop)
```
Atomized! X notes written · Y connections you approved · index updated.
- YYYY-MM-DD_001: "Title" [tags] → [[linked-note]]
- YYYY-MM-DD_002: "Title" [tags]
```
If the user rejected all link candidates, that's fine — say "0 connections kept" and move on.

---

## Guardrails

- **One decision round, not a conversation — but never zero.** Draft everything, propose everything, take one pass of the user's keep/drop/edit, commit. Speed comes from *batching* everything into that single checkpoint — NOT from skipping it. The "under 3 min / no chatter" guardrail means no per-note back-and-forth; it does NOT license writing files before the user has approved. If you find yourself asking about each note in turn, you're doing it wrong; if you find yourself writing without asking at all, you're doing it more wrong.
- **Never auto-write a connection.** Every `[[wikilink]]` that lands in a file was approved by the user in step 4. This is the rule that keeps the *connecting* — the deepest part of the thinking — human.
- **IMPORT never commits un-touched.** The forced-elaborate gate is the difference between a note pile and understanding. An imported note the user never re-read is the LLM's understanding wearing the user's voice. Make them touch one.
- **Don't force links.** Zero connections is correct for the first handful of notes. Connections compound.

> ⚠️ **An AI-phrased card can fool you into thinking you thought it.** *You are the easiest person to fool* (Feynman). That is why DRAFT-IT and IMPORT both make you touch the note before it saves.

---

## Where it lives (CC ↔ Codex)

| | Claude Code | Codex |
|---|---|---|
| **Path** | `.claude/skills/atomize/SKILL.md` (or `~/.claude/skills/` global) | Codex skills dir — same `SKILL.md` standard (agentskills.io) |
| **Invoke** | `/atomize` | `/atomize` |

`allowed-tools: AskUserQuestion, Write, Read, Edit, Bash, Glob, Grep`

The same `SKILL.md` works in both tools — the format is the cross-tool standard.

---

## Power-user options

- **Contradiction-lint pass** — after writing, grep the vault for notes that now contradict each other or near-duplicate the new ones; print a report (don't auto-edit), ask what to resolve. (Karpathy's "lint" operation.)
- **Semantic connection-finding** — if `qmd` is installed, use `qmd query "<note's idea, 2-3 sentences>" -n 10` (the count flag is `-n`, not `--limit`) instead of grep in step 3 to surface conceptual neighbors a keyword grep would miss; still PROPOSE, the user still decides.
- **Provenance schema** — add `source_url` to frontmatter to link back to the original post (the production-vault schema).
