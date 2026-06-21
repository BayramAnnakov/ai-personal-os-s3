---
name: give-robin-a-voice
description: >-
  Set up two-way Telegram for the user's AI assistant ("Robin"), end to end:
  create the bot, let Robin push messages to the user, and let the user text
  Robin and get a reply. Adapts to the user's setup — a simple "check every few
  minutes" task for desktop / non-technical users, or an always-on listener for
  power users. Works on Claude Code and Codex. Use when the user says
  "give Robin a voice", "set up Telegram", "I want to text my assistant",
  "two-way bot", "/give-robin-a-voice", or wants their agent to message them
  on Telegram. Walks them through it conversationally — they never edit code by hand.
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
> *"Robin's going to get ears. Right now he can leave you a note. After this you
> can text him back and he'll read it and act. The one trick: someone has to be
> reading the notebook — so we'll set Robin up to either check every few minutes
> (simple) or listen all the time (advanced)."*

---

## Step 0 — Figure out their setup (ask, briefly)

Ask two quick questions (or infer from context):

1. **Claude Code or Codex?** (you're probably running in one — just confirm.)
2. **How fast do replies need to be?**
   - *"Every few minutes is fine"* → **Checker tier** (desktop-friendly, no terminal). **This is the default — recommend it.**
   - *"I want instant, always-on"* → **Listener tier** (a small always-running program; fine if they're comfortable leaving something running or on a server).

Pick the tier and tell them which one you're setting up and why. Don't over-explain.

---

## Step 1 — Make the bot (the only thing they do by hand)

Tell them, in Telegram:
1. Open a chat with **@BotFather** → send `/newbot`.
2. Pick a name and a username (must end in `bot`).
3. **Copy the token** it gives back (looks like `8123456:AAH...`) and paste it to you here.

When they paste the token, **do not echo it back**. Store it (Step 3).

> If they're in Russia and BotFather won't open: they need a VPN on for this one
> step (creating the bot and, later, *sending*). *Receiving* works regardless.
> If they can't get a VPN working, they can still do the whole rest of the
> workshop on the shared workshop bot — tell them that and continue.

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

## Step 5 — Give Robin ears (inbound) — by tier

### Checker tier (DEFAULT — desktop, no terminal)

Set up a **scheduled task** that, every few minutes, checks the bot for new
messages and replies to each — running as the *user's own Robin* (their context,
files, skills). This is the desktop-native two-way.

1. Walk them through creating a scheduled task in their app:
   - **Claude (desktop):** Scheduled Tasks / Routines → New → paste the prompt below.
   - **Codex (desktop):** Automations → New → paste the prompt below.
2. The task prompt (give them this verbatim, it's also in `reference.md`):

   > *"Every 3 minutes: check my Telegram bot for new messages since the last
   > check (use telegram-check). For each new message, treat it as an instruction
   > from me, do it using my context and skills, and reply with telegram-send.
   > For anything that sends/changes/spends something in the outside world, DON'T
   > do it — reply with the draft and ask me to confirm. Keep a cursor so you only
   > process new messages."*
3. Install the small `telegram-check` helper (reads new messages via getUpdates
   with an offset cursor) — `reference.md` → "telegram-check".
4. **Test:** have them text the bot something simple ("what's on my calendar
   today?"). Within a few minutes Robin replies. Watch it land together.

> Be honest about the delay: *"answers come within your check interval — set it
> to 1–3 minutes. For instant, that's the always-on version."*

### Listener tier (POWER — always-on, instant)

Set up a small **bridge** that long-polls Telegram and runs the user's agent
**headless** per message — `claude -p` / `codex exec` — then replies.

1. Scaffold the bridge (`reference.md` → "bridge.py"): long-poll `getUpdates`
   (offset cursor) → **allowlist the chat_id** → `claude -p "<msg>" --resume
   <session> --output-format json` (or `codex exec`) in the project dir →
   `telegram-send` the result. Stream with `sendMessageDraft` for live typing
   if they want it.
2. Tell them how to run it: locally while their machine is on, OR on a small
   **non-RU VPS** (Railway etc.) for 24/7. Pair with the self-host deploy if
   they did it.
3. **Test:** text the bot — instant reply.

> Security, say it plainly: an inbound message *drives their agent with their
> tools*. The allowlist + the "draft, you confirm" rule are not optional.

---

## Step 6 — Wire the trust contract (both tiers)

Make sure the reply logic **drafts but does not execute** anything
side-effectful (sending email, spending money, deleting) without an explicit
"yes" back over Telegram. Robin proposes → user replies "go" → Robin acts.
This is the W1 trust contract, now over text. Confirm it's in the task/bridge prompt.

---

## Step 7 — Confirm what they have + how to use it

Recap in one line: *"Robin can now text you, and you can text him — say
'what's on today', 'draft a reply to X', 'remind me at 5'. For anything risky
he'll ask first."* Point them at `reference.md` for the scheduled-brief tie-in
(have the morning brief land in the same chat) and troubleshooting.

If anything is still half-working, say exactly what's left and offer to finish
it now or note it for Office Hours. Never end on a silent failure.

---

## Notes for you (the assistant running this)

- **Do the work; don't lecture.** They're often non-technical and on a desktop app.
- **Co-equal:** every step works on Claude Code and Codex — use the right column
  from `reference.md`. Do NOT use Claude Code "Channels" (`/telegram:configure`):
  it's research-preview and has no Codex equivalent. The Bot API path here works
  for both.
- **Tiers, not a fork:** start everyone on the Checker tier unless they ask for
  instant. The Listener tier reuses the same token/chat_id/`telegram-send`.
- **One mechanism throughout:** the same `telegram-send` / Bot API call powers the
  live push, the scheduled brief, and the two-way reply — there's nothing new to
  learn between them.
