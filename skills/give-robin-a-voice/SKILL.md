---
name: give-robin-a-voice
description: >-
  Set up two-way Telegram for the user's AI assistant ("Robin"), end to end:
  create the bot, let Robin push messages to the user, let the user text Robin
  and get a reply that acts with their context and skills, and deploy it to the
  cloud so it runs 24/7 with the laptop closed. Auto-detects whether it's running
  in Claude Code or Codex and works on both. Use when the user says "give Robin a
  voice", "set up Telegram", "I want to text my assistant", "two-way bot",
  "/give-robin-a-voice", or wants their agent to message them on Telegram. Walks
  them through it conversationally — they never edit code by hand.
license: MIT
---

# Give Robin a Voice

You (the assistant) are running a **setup wizard**. Your job: get the user from
"my assistant can't message me" to "I can text Robin and he texts me back" —
**conversationally**, doing the technical work yourself. The user should only
ever (a) click around in Telegram/their desktop app when you tell them to, and
(b) paste one token. You do everything else.

Keep it warm and concrete. One step at a time. Confirm each step worked before
moving on. If something fails, fix it for them — don't hand them an error.

> **The mental model to give them up front (say this):**
> *"Robin's going to get ears. Right now he can leave you a note — after this you
> can text him back and he reads it and acts. The trick is that someone has to be
> listening: a tiny always-on program. We'll start it on your machine so you feel
> it work, then put it in the cloud so it keeps listening with your laptop closed."*

---

## Step 0 — Detect your setup (automatic — don't ask)

You're running *inside* Claude Code or Codex, so you already know which one — use
that column throughout (CC: `claude -p`, `.claude/`; Codex: `codex exec`,
`~/.codex/`). Only ask if you genuinely can't tell. There's no tier to choose:
everyone gets the always-on **listener**, tested locally then deployed to the
cloud. Just tell them what you're setting up and move on.

---

## Step 1 — Make the bot (the only thing they do by hand)

Tell them, in Telegram:
1. Open a chat with **@BotFather** → send `/newbot`.
2. Pick a name and a username (must end in `bot`).
3. **Copy the token** it gives back (looks like `8123456:AAH...`) and paste it to you here.

When they paste the token, **do not echo it back**. Store it (Step 3).

> If BotFather won't open or sending fails on their network: a VPN fixes it for
> this one step (creating the bot and, later, *sending*). *Receiving* works
> regardless. If they can't get a VPN working, they can still do the whole rest
> of the workshop on the shared workshop bot — tell them that and continue.

---

## Step 2 — Get their chat_id (do this for them)

Robin needs to know *where* to send. Two ways — prefer the automatic one:

- **Automatic:** ask them to open their new bot in Telegram and press **Start**
  (or send `/start`) — this is required; messages sent *before* Start don't reach
  `getUpdates`. Then you call `getUpdates` and read `result[].message.chat.id`.
  (See `reference.md` → "Get chat_id". Updates can lag a second — poll a few times.)
- **Manual fallback:** have them message **@userinfobot**, which replies with
  their numeric id.

Confirm you've got a number. Store it with the token.

---

## Step 3 — Store the secrets safely (you do this)

Write the token + chat_id to a config file the scripts read — **never hard-code
them, never commit them.** Create `~/.robin/telegram.conf` (mode 600) and make
sure `.robin/` is git-ignored. Exact commands: `reference.md` → "Store secrets".

---

## Step 4 — Give Robin a mouth (outbound) and test it

Install the **`telegram-send`** capability — a tiny script that calls the
Telegram Bot API to send a message. Same on Claude Code and Codex.

1. Drop the `telegram-send` skill/script in place (`reference.md` → "telegram-send").
2. **Test it now:** send the user a real message ("👋 Robin can talk now.").
3. Confirm their phone buzzed before moving on. If it 403s, it's the network /
   (for a scheduled cloud run) the allowlist — see `reference.md` → "Troubleshooting".

> This is the **push / "talk TO you"** half. They've now felt it. Next: the ears.

---

## Step 5 — Give Robin ears (inbound) — the listener

Set up a small **listener** that watches Telegram and, for each message the user
sends, runs *their* agent (their context, files, skills) and replies. Same idea
on Claude Code and Codex.

1. Scaffold the listener (`reference.md` → "bridge.py"): it long-polls
   `getUpdates` (offset cursor) → **allowlists their chat_id** → runs
   `claude -p "<msg>" --resume <session>` (or `codex exec`) in the project dir →
   replies with `telegram-send`. Optional: stream with `sendMessageDraft` for a
   live "typing" feel.
2. **Test it locally first** — run it on their machine (`python3 bridge.py`),
   have them text the bot something simple ("what's on my calendar today?"), and
   watch the reply land. This proves the whole loop before any cloud setup.
   Locally, Robin runs on the Claude/Codex subscription they're **already signed
   into** — no API key, no extra cost.
3. Say the security rule plainly: an inbound message *drives their agent with
   their tools*. The allowlist (only their chat_id) + the "draft, you confirm"
   rule (Step 7) are not optional.

> A local listener only answers while their machine is awake. The next step puts
> it in the cloud so it keeps listening with the laptop closed — that's the whole
> point of "Robin works while you sleep."

---

## Step 6 — Deploy it so it runs without your computer (Railway)

Move the listener to a small always-on cloud box so it keeps answering with the
laptop shut. **Railway** is the right host — a persistent process that can run
the agent. (Vercel can't: its functions cap at ~10s, which kills an agent run.)
If their local network blocks `api.telegram.org`, deploying also fixes *sending*
— Railway's egress isn't restricted. A BotFather **Bot API** bot replying to its
owner is fine on Railway; their "userbot" ban targets MTProto user-account
automation, which we don't use. Full recipe: `reference.md` → "Deploy to Railway".

### First — the cost gate. Inform them, then ASK. Don't deploy silently.

The cloud listener needs a **paid API key** (`ANTHROPIC_API_KEY`; Codex: an API
key). **Their subscription can't run it** — a headless OAuth token 401s within
~15 min and the terms put always-on automation on the API path. So spell out the
trade and wait for a yes:

> *"To run 24/7 with your laptop closed costs money: ~$5/mo for the host **plus**
> per-use API tokens, and it needs an API key — not your Claude/Codex
> subscription. Your free option is the local listener we just tested (works
> while your machine is on). Want the always-on cloud version, or keep it local?"*

If they say no, **stop here** — they already have a working two-way Robin. Only
continue if they opt in.

### Then walk them through it (you do the wiring; they click + paste):
1. A **GitHub repo** holds the listener + a minimal copy of their Robin context
   (CLAUDE.md / AGENTS.md + the skills the listener needs). `reference.md` shows
   the layout and the Dockerfile.
2. On Railway: **New Project → Deploy from GitHub repo**, pick that repo.
3. The Dockerfile installs the agent CLI on the box (`npm i -g …` — `reference.md`).
   Get the API key for them: Claude → <https://console.anthropic.com> (`ANTHROPIC_API_KEY`);
   Codex → <https://platform.openai.com/api-keys> (`OPENAI_API_KEY`).
4. Add variables in Railway: `TELEGRAM_TOKEN`, `TELEGRAM_CHAT_ID`, and the key —
   `ANTHROPIC_API_KEY` for Claude, or `OPENAI_API_KEY` + `AGENT=codex` for Codex.
   `bridge.py` reads these from the env. (Codex needs a one-time api-key login,
   which the bridge does at boot — see `reference.md`.)
5. Deploy, then text the bot — it answers with the laptop closed. Run only **one**
   listener per bot (local *or* cloud, not both) — two pollers fight (409).

> **Cost detail:** Railway has no real always-on free tier — a tiny listener is
> ~$1/mo of usage on **Hobby** ($5/mo, includes $5 usage) → budget **~$5/mo** for
> the host, plus the per-use API tokens above. Cheaper host, same API tokens:
> **Fly.io**. The only truly-free path stays the **local** listener (laptop on).

---

## Step 7 — Wire the trust contract

Make sure the reply logic **drafts but does not execute** anything
side-effectful (sending email, spending money, deleting) without an explicit
"yes" back over Telegram. Robin proposes → user replies "go" → Robin acts.
This is the W1 trust contract, now over text. Confirm it's in the listener prompt.

---

## Step 8 — Confirm what they have + how to use it

Recap in one line: *"Robin can now text you, and you can text him — say
'what's on today', 'draft a reply to X', 'remind me at 5'. For anything risky
he'll ask first."* Point them at `reference.md` for the scheduled-brief tie-in
(have the morning brief land in the same chat) and troubleshooting.

If anything is still half-working, say exactly what's left and offer to finish
it now or note it for Office Hours. Never end on a silent failure.

---

## Notes for you (the assistant running this)

- **Do the work; don't lecture.** They're often non-technical and on a desktop app.
- **Auto-detect, don't ask:** you already know if you're Claude Code or Codex —
  pick the right column from `reference.md` and go. No "which tool?" question.
- **Co-equal:** every step works on Claude Code and Codex — use the right column
  from `reference.md`. Do NOT use Claude Code "Channels" (`/telegram:configure`):
  it's research-preview and has no Codex equivalent. The Bot API path here works
  for both.
- **One listener, two homes:** the same `bridge.py` runs locally (test, $0, laptop
  must be on) and on Railway (24/7, ~$5/mo, laptop closed). Local first to feel it
  work, then deploy. Reading secrets from a conf file locally vs env vars in the
  cloud is the only difference.
- **One mechanism throughout:** the same `telegram-send` / Bot API call powers the
  live push, the scheduled brief, and the two-way reply — there's nothing new to
  learn between them.
