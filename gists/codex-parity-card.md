# Codex Parity Card — AI Personal OS S3

Every Claude Code concept in this course has a Codex equivalent. Pin this in chat at workshop start.

## Identity files

| | Claude Code | Codex |
|---|---|---|
| Root identity | `~/.claude/CLAUDE.md` | `~/.codex/AGENTS.md` |
| Imports | `@import file.md` (native) | manual concat / `cat` chain |
| Per-project | `./CLAUDE.md` | `./AGENTS.md` |
| Reload | automatic on session start | automatic on session start |

## Skills (agentskills.io open standard — same file format)

| | Claude Code | Codex |
|---|---|---|
| File format | `SKILL.md` | `SKILL.md` (identical) |
| Install path | `~/.claude/skills/<name>/` | `~/.codex/skills/<name>/` |
| Invoke | `/skill-name` | `/skill-name` |
| Auto-trigger | matches on `description` | matches on `description` |
| Marketplace install | "Install the skill from <URL>" | none — `git clone` + `cp -r` + restart |

## MCP

| | Claude Code | Codex |
|---|---|---|
| Protocol | MCP | MCP (via Apps SDK) |
| Same server, same install | yes | yes |
| Config | `~/.claude/mcp.json` | `~/.codex/mcp.json` |

## Push to user (W3 material)

| | Claude Code | Codex |
|---|---|---|
| Native channel | `/telegram:configure` (first-party) | none |
| Workaround | — | `notify` hook + Telegram Bot API (~8 lines bash) |

## Scheduled background (W3 material)

| | Claude Code | Codex |
|---|---|---|
| Stable | **Routines** | **Automations** (verify week-of) |
| If unstable | — | `/goal` + `notify` hook + Bot API |

## Long-horizon goals (W4 material)

| | Claude Code | Codex |
|---|---|---|
| In-session | `/goal` | Goal Mode (persisted; may be buggy) |

## Permissions / safety

| | Claude Code | Codex |
|---|---|---|
| Auto-approve | Auto Mode + hard-deny rules | Approval modes |
| Recommended | Auto Mode for trusted skills only | Same posture |

## Multi-agent (W4 material)

| | Claude Code | Codex |
|---|---|---|
| Coordination | Agent Teams / sub-agents | Subagents (manager-worker, GA) |

## Memory store (W2 / W5 material)

| | Claude Code | Codex |
|---|---|---|
| Managed | Managed Memory + Dreaming | Chronicle (screen-based) |
| File-based fallback | knowledge vault in repo | same |

## Package & share (W5 material)

| | Claude Code | Codex |
|---|---|---|
| Bundle | `.zip` plugin / URL plugin | Plugin workflows |

---

## In practice

The deck shows commands in both columns whenever the difference matters. Most of the time, the command is identical — just install paths differ.

If you hit a Codex-specific quirk: paste the error in chat, we'll work through it.

## Conversation history paths (for `/init-robin`)

| Tool | Where conversations live |
|---|---|
| Claude Code | `~/.claude/projects/**/*.jsonl` (one file per session) |
| Codex CLI | `~/.codex/sessions/` (similar structure) |
| Claude.ai | Settings → Account → Export Data → `.zip` download |
| ChatGPT | Settings → Data Controls → Export Data → emailed `.zip` |
