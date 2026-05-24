# AI Personal OS — Season 3

Course materials for **AI Personal OS — твой Chief of Staff за 5 недель** · Season 3, Cohort 3 · Instructor: Bayram Annakov.

5 workshops between May 24 and Jul 19, 2026. Online, in Russian. Materials in English. Tool-agnostic: Claude Code and Codex are co-equal first-class throughout.

---

## What's in here

```
ai-personal-os-s3/
├── skills/
│   ├── init-robin/       # Workshop 1 · evidence-based identity (CLAUDE.md, SOUL.md, user-profile.md)
│   └── finish-robin/     # Workshop 1 homework · declared identity (course-goals, goals-2026, achievements)
└── gists/
    └── codex-parity-card.md     # CC ↔ Codex command mapping (pinned in chat W1)
```

## Install the skills

### Claude Code

Ask Claude Code:
> Install the init-robin skill from https://github.com/BayramAnnakov/ai-personal-os-s3

Or manually:

```bash
git clone https://github.com/BayramAnnakov/ai-personal-os-s3 ~/tmp/aipos-s3
cp -r ~/tmp/aipos-s3/skills/init-robin ~/.claude/skills/
```

Then run `/init-robin` in any folder.

### Codex

```bash
git clone https://github.com/BayramAnnakov/ai-personal-os-s3
cp -r ai-personal-os-s3/skills/init-robin ~/.codex/skills/
# restart Codex
```

Then run `/init-robin`.

## Capability ladder

The course is organized around what Robin can DO, not which features power him. 8 rungs across 5 weeks:

| # | Robin can… | Built in |
|---|---|---|
| 1 | Know who you are | W1 |
| 2 | See your world (MCP) | W1 seed · W3 |
| 3 | Talk to you (push) | W1 preview · W3 |
| 4 | Do what you do (skills) | W1 seed · W2/W3 |
| 5 | Remember what you know | W2 |
| 6 | Act without you watching | W3 |
| 7 | Think & verify | W4 |
| 8 | Be your Chief of Staff | W5 |

Rungs 9 (team OS) and 10 (self-improving) are horizon — different course.

## Previous cohorts

- Cohorts 1 & 2: [BayramAnnakov/ai-personal-os-skills](https://github.com/BayramAnnakov/ai-personal-os-skills)

## License

MIT — see [LICENSE](LICENSE).

---

Built around Engelbart's H-LAM/T framework (1962) and the agentskills.io open standard (Dec 2025).
