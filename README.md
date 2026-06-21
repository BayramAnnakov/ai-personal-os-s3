# AI Personal OS — Season 3

Course materials for **AI Personal OS — твой Chief of Staff за 5 недель** · Season 3, Cohort 3 · Instructor: Bayram Annakov.

5 workshops between May 24 and Jul 19, 2026. Online, in Russian. Materials in English. Tool-agnostic: Claude Code and Codex are co-equal first-class throughout.

---

## What's in here

This repo is the **`personal-os` plugin** — a growing skill bundle that ships across all 5 workshops. New skills land here as each workshop completes.

```
ai-personal-os-s3/
├── .claude-plugin/plugin.json   # Claude Code plugin manifest
├── .codex-plugin/plugin.json    # Codex plugin manifest
├── skills/
│   ├── init-robin/              # W1 · evidence-based identity (CLAUDE.md, SOUL.md, user-profile.md)
│   ├── finish-robin/            # W1 homework · declared identity (course-goals, goals-2026, achievements)
│   ├── atomize/                 # W2 · knowledge OS · atomic Zettelkasten notes into ~/vault/
│   └── give-robin-a-voice/      # W3 · two-way Telegram for Robin (push + reply + scheduled brief; Bot API, no MCP)
└── gists/
    └── codex-parity-card.md     # CC ↔ Codex command mapping (pinned in chat W1)
```

## Install the plugin

### Claude Code (recommended)

```bash
git clone https://github.com/BayramAnnakov/ai-personal-os-s3 ~/aipos-s3
claude --plugin-dir ~/aipos-s3
```

Then run `/init-robin` in any session (CC may show it as `/personal-os:init-robin` — same skill).

To pick up future updates: `cd ~/aipos-s3 && git pull` and restart Claude Code.

> **CC GUI users:** if you prefer the UI, open the `/plugin` command and use **Customize → Personal Plugins** to add a local plugin folder (point it at the unzipped repo).

### Codex (recommended)

```bash
codex plugin marketplace add BayramAnnakov/ai-personal-os-s3
# restart Codex
```

Then run `/init-robin`.

> **Codex Desktop:** the Plugins panel only browses the marketplace — there is no "Upload local plugin" button. If you're on Desktop without git, use the **standalone cp** fallback below (drops the skills directly into `~/.codex/skills/`). Loses the auto-update story but works identically.

### Desktop apps (Claude Desktop / Codex Desktop) — no terminal needed

1. Download **`personal-os-0.1.0.zip`** from the [latest release](https://github.com/BayramAnnakov/ai-personal-os-s3/releases/latest)
2. Open the Plugins UI in your desktop app → **Upload Plugin** → select the downloaded `.zip`

The same `.zip` works for both Claude Desktop and Codex Desktop — the archive carries both manifests at the root.

### No git? Download the ZIP

If you don't have `git` (or GitHub auth) configured, get the repo as a ZIP:

1. Open https://github.com/BayramAnnakov/ai-personal-os-s3 in a browser
2. Click the green **Code** button → **Download ZIP**
3. Unzip it. The folder name will be `ai-personal-os-s3-main`.

Then either point the CLI at the unzipped folder directly:

```bash
# Claude Code
claude --plugin-dir ~/Downloads/ai-personal-os-s3-main
```

Or drop the skills in by hand:

```bash
# Claude Code
mkdir -p ~/.claude/skills
cp -r ~/Downloads/ai-personal-os-s3-main/skills/* ~/.claude/skills/

# Codex
mkdir -p ~/.codex/skills
cp -r ~/Downloads/ai-personal-os-s3-main/skills/* ~/.codex/skills/
# restart Codex
```

Then run `/init-robin`.

### Fallback for plugin-unaware CLIs (with git)

If your CC/Codex version doesn't recognize the plugin install, drop the skills in directly:

```bash
git clone https://github.com/BayramAnnakov/ai-personal-os-s3 ~/aipos-s3

# Claude Code
mkdir -p ~/.claude/skills && cp -r ~/aipos-s3/skills/* ~/.claude/skills/

# Codex
mkdir -p ~/.codex/skills && cp -r ~/aipos-s3/skills/* ~/.codex/skills/
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
