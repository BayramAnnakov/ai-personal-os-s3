# give-robin-a-voice — Slack reference (scripts & exact steps)

The **Slack** path to the same outcome as `reference.md`: Robin pushes you a
message, you DM him back, he runs your agent with your context and replies, and
it all runs 24/7 in the cloud with your laptop closed. Works the same on Claude
Code and Codex — only the agent CLI differs, never the Slack wiring.

**The Slack model maps 1:1 onto the Telegram one** — read this against
`reference.md`:

| Concept | Telegram | Slack |
|---|---|---|
| "The bot" | BotFather bot + token | Slack **App** (from a manifest) |
| Outbound token | `TELEGRAM_TOKEN` | `SLACK_BOT_TOKEN` (`xoxb-…`) |
| Inbound transport | long-poll `getUpdates` | **Socket Mode** (WebSocket) + `SLACK_APP_TOKEN` (`xapp-…`) |
| "Where to send" | `chat_id` | your Slack **user id** (`U…`) — DMs you |
| Send a message | `sendMessage` | `chat.postMessage` |
| Allowlist on | `message.chat.id` | `event["user"]` |

Two reasons the Slack path is actually **easier** for this cohort:
- **No VPN dance.** `slack.com` isn't subject to the Telegram network block, so
  *sending* works locally with no VPN (the Telegram caveat disappears).
- **No `409` fights.** Socket Mode allows up to 10 connections and delivers each
  event to only **one** of them — a brief redeploy overlap won't double-reply.

The one cost vs Telegram: the listener needs the `slack_bolt` pip package (the
Telegram one is pure stdlib). One `pip install` line covers it, locally and in
the Dockerfile.

---

## Step 1 — Make the Slack App (the only thing they do by hand)

Send them to <https://api.slack.com/apps> → **Create New App** → **From an app
manifest** → pick their workspace → paste this YAML → **Create**. The manifest
sets every scope, the DM event, and Socket Mode in one shot — no clicking through
checkboxes.

```yaml
display_information:
  name: Robin
features:
  bot_user:
    display_name: Robin
    always_online: true
oauth_config:
  scopes:
    bot:
      - chat:write      # send messages
      - im:history      # read the DMs you send the bot
      - im:write        # open a DM with you (to push)
      - users:read      # resolve your name (optional)
settings:
  event_subscriptions:
    bot_events:
      - message.im      # fire when you DM the bot
  socket_mode_enabled: true
```

Then, still in the app's settings:
1. **Install App** (or **OAuth & Permissions** → *Install to Workspace*) →
   **Allow**. Copy the **Bot User OAuth Token** — starts `xoxb-…`. Paste it to you.
2. **Basic Information** → **App-Level Tokens** → **Generate Token and Scopes** →
   add scope **`connections:write`** → **Generate**. Copy the token — starts
   `xapp-…`. Paste it to you. *(This is the Socket Mode token; the manifest can't
   create it for you — it's generated manually, once.)*

When they paste either token, **do not echo it back**. Store both (Step 3).

> If you later change scopes or events, you must **reinstall** the app for them to
> take effect — the manifest above already includes everything, so you shouldn't
> need to.

---

## Step 2 — Get their Slack user id (do this for them)

Robin DMs a **user id** (`U…`), not a channel. Two ways — prefer the manual one,
it's instant in Slack:

- **Manual (fast):** in Slack, click their **profile** → **⋮ (More)** →
  **Copy member ID**. It looks like `U07ABCD1234`. Have them paste it.
- **Automatic:** once the listener is running (Step 5), have them DM the bot once;
  the listener sees `event["user"]` — that's the id. Useful as a confirmation.

Store it with the tokens.

---

## Step 3 — Store the secrets safely (you do this)

```bash
mkdir -p ~/.robin && chmod 700 ~/.robin
cat > ~/.robin/slack.conf <<EOF
SLACK_BOT_TOKEN=<xoxb-...>
SLACK_APP_TOKEN=<xapp-...>
SLACK_USER_ID=<U...>
EOF
chmod 600 ~/.robin/slack.conf
# if inside a git repo, keep it out of version control:
grep -qxF '.robin/' .gitignore 2>/dev/null || echo '.robin/' >> .gitignore
```

Never hard-code the tokens, never commit them.

---

## slack-send (outbound — "Robin's mouth")

Install as a skill (`.claude/skills/slack-send/` for CC, `~/.codex/skills/slack-send/`
for Codex) wrapping this script, or drop it at `~/.robin/slack-send.sh`:

```bash
#!/usr/bin/env bash
# slack-send.sh "message text"   — DMs SLACK_USER_ID
source ~/.robin/slack.conf
curl -s -X POST https://slack.com/api/chat.postMessage \
  -H "Authorization: Bearer ${SLACK_BOT_TOKEN}" \
  -H "Content-type: application/json; charset=utf-8" \
  --data "$(python3 -c 'import json,sys; print(json.dumps({"channel":sys.argv[1],"text":sys.argv[2]}))' "$SLACK_USER_ID" "$1")" \
  | grep -q '"ok":true' && echo "sent" || echo "FAILED"
```

`python3` builds the JSON so any quotes/newlines in the message are safe (the
Slack equivalent of Telegram's `--data-urlencode`). Posting with `channel` set to
a `U…` user id makes Slack open/route the DM automatically (`im:write`).

Invoke: *"Robin, use slack-send to message me …"* → he runs `slack-send.sh "…"`.

Test it now — send a real DM ("👋 Robin can talk now.") and confirm it landed
before moving on. If it fails, check `"error"` in the JSON against Troubleshooting.

---

## bridge_slack.py (the listener — always-on, instant)

Connects via Socket Mode (no public URL), runs the user's agent headless per DM,
replies. Allowlist-gated. Reads secrets from `~/.robin/slack.conf` **locally** or
from **environment variables** in the cloud — the same file works in both homes.
The `agent()` body is identical to `reference.md`'s `bridge.py` — only the
transport (Slack Socket Mode instead of Telegram long-poll) changes.

```python
#!/usr/bin/env python3
# Slack Socket Mode → run the user's agent headless → reply. Allowlist-gated.
import os, json, subprocess, pathlib
from slack_bolt import App
from slack_bolt.adapter.socket_mode import SocketModeHandler

def load_conf():
    p = os.path.expanduser("~/.robin/slack.conf")               # local
    if os.path.exists(p):
        return dict(l.strip().split("=",1) for l in open(p) if "=" in l)
    return {k: os.environ[k] for k in                           # cloud (Railway vars)
            ("SLACK_BOT_TOKEN","SLACK_APP_TOKEN","SLACK_USER_ID")}

conf  = load_conf()
ALLOW = conf["SLACK_USER_ID"]
AGENT = os.environ.get("AGENT", "claude")        # "claude" | "codex"
SESS  = pathlib.Path(os.path.expanduser("~/.robin/session"))
app   = App(token=conf["SLACK_BOT_TOKEN"])

# Codex needs an explicit api-key login (the env var alone 401s). Run once at boot.
if AGENT == "codex":
    k = os.environ.get("OPENAI_API_KEY", "")
    if k:
        subprocess.run(["codex","login","--with-api-key"], input=k, capture_output=True, text=True)

def agent(text):                                  # identical to reference.md
    if AGENT == "codex":                          # uses ~/.codex/auth.json from the login above
        r = subprocess.run(["codex","exec","--skip-git-repo-check",
                            "--dangerously-bypass-approvals-and-sandbox", text],
                           capture_output=True, text=True)
        return (r.stdout.strip() or r.stderr.strip() or "(no reply)")[-3500:]
    cmd = ["claude","-p",text,"--output-format","json",
           "--permission-mode","dontAsk","--allowedTools","Read"]
    if SESS.exists(): cmd += ["--resume", SESS.read_text().strip()]
    out = json.loads(subprocess.run(cmd, capture_output=True, text=True).stdout)
    sid = out.get("session_id","")
    if sid:
        SESS.parent.mkdir(parents=True, exist_ok=True)   # ~/.robin may not exist in the cloud
        SESS.write_text(sid)
    return out.get("result","(no reply)")

@app.event("message")
def on_message(event, client):
    if event.get("channel_type") != "im": return     # DMs only
    if event.get("bot_id") or event.get("subtype"):  # ignore the bot's own echoes / edits
        return
    if event.get("user") != ALLOW: return            # allowlist
    if "text" not in event: return
    client.chat_postMessage(channel=event["channel"], text=agent(event["text"]))

if __name__ == "__main__":
    SocketModeHandler(app, conf["SLACK_APP_TOKEN"]).start()
```

Run locally first:
```bash
pip3 install slack_bolt          # one dependency (pulls slack_sdk + websocket-client)
python3 bridge_slack.py          # while the machine is on — proves the loop
```
DM the bot something simple ("what's on my calendar today?") and watch the reply
land. Set `AGENT=codex` to use Codex instead of Claude. Locally the agent runs on
the Claude/Codex subscription they're **already signed into** — no API key, $0.

> `slack_bolt` auto-acknowledges each event (the `envelope_id` handshake), so you
> don't manage acks. The `bot_id`/`subtype` guard stops Robin from replying to his
> own messages — without it a bot can loop on itself.

---

## Deploy to Railway (24/7, no laptop)

Identical in shape to the Telegram deploy in `reference.md` — a persistent
**worker** process (no exposed port; Socket Mode dials out). Only the env vars,
the one pip line, and the entry script differ.

**Repo layout** (a private GitHub repo Robin deploys from):
```
robin-listener/
├── bridge_slack.py             # the listener above
├── slack-send.sh               # Robin's mouth (same script)
├── Dockerfile                  # builds the agent CLI + python + slack_bolt
├── CLAUDE.md  (or AGENTS.md)    # a MINIMAL copy of Robin's context
└── skills/                     # only the skills the listener needs
```

**Dockerfile** — adds `python3-pip` and a `pip install slack_bolt` over the
Telegram one. `git` is required for Codex. Installing both CLIs is fine; trim to
one to slim the image.
```dockerfile
FROM node:22-slim
RUN apt-get update && apt-get install -y python3 python3-pip git && rm -rf /var/lib/apt/lists/*
RUN pip3 install --break-system-packages slack_bolt          # Debian needs the flag (PEP 668)
RUN npm install -g @anthropic-ai/claude-code @openai/codex   # trim to the one you use
WORKDIR /robin
COPY . .
CMD ["python3", "bridge_slack.py"]                           # a worker — no exposed port
```

**Get an API key** (the cloud agent needs one — same cost note as `reference.md`):
- Claude: <https://console.anthropic.com> → API Keys → `ANTHROPIC_API_KEY`.
- Codex: <https://platform.openai.com/api-keys> → `OPENAI_API_KEY`.

**Deploy steps** (you walk them through; they click + paste):
1. Push `robin-listener/` to a **private** GitHub repo (never commit the tokens —
   they arrive as Railway variables, not files).
2. Railway → **New Project → Deploy from GitHub repo** → pick the repo. It
   auto-detects the Dockerfile and runs it as a background worker.
3. Railway → **Variables**, add: `SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN`,
   `SLACK_USER_ID`, and the key — `ANTHROPIC_API_KEY` for Claude, or
   `OPENAI_API_KEY` + `AGENT=codex` for Codex. (Claude reads `ANTHROPIC_API_KEY`
   automatically; Codex's `codex login --with-api-key` runs at boot — see above.)
4. Deploy. DM the bot with the laptop **closed** — it answers.

> **Replicas / duplicates.** Unlike Telegram, two readers don't `409` — Slack
> sends each event to only one connection. But still keep Railway at **1 replica**
> and don't run local + cloud against the same app at once: during a redeploy
> overlap both could process new DMs and burn tokens twice. One home at a time.

**Cost & auth — same as Telegram.** Two bills: the **host** (~$5/mo — Railway
**Hobby**, $5 incl. $5 usage; a tiny worker fits inside it) and the **agent** via
the API key (per-use tokens). The cloud agent needs an **API key, not a
subscription** (a headless OAuth token 401s within ~15 min). So: **local** =
subscription, $0 (laptop on); **cloud** = API key + host. Cheaper host, same
tokens: **Fly.io**.

> **Context drift:** the cloud Robin only knows what's in the repo. Keep the
> CLAUDE.md / skills copy minimal and re-push when it changes. Session resume is
> best-effort — Railway's filesystem resets on redeploy, so `~/.robin/session`
> won't survive a deploy. Fine for a Q&A bot; don't rely on it for long threads.

---

## Tie in the morning brief (so it lands in the same DM)

The scheduled brief just calls `slack-send` like everything else — point its push
at the same `SLACK_USER_ID`, and now you can **reply to the brief in the DM**
("draft the prep doc") and Robin acts on it. One app, one mechanism: push, brief,
and two-way.

## Streaming (optional polish)

Slack has no bot "typing" indicator over the Web API, but you can fake live
output: post a placeholder ("🤔 …"), capture its `ts` from the response, then
`chat.update` it with the growing text fed by
`claude -p … --output-format stream-json --include-partial-messages`.

## Troubleshooting

| Symptom | Fix |
|---|---|
| `chat.postMessage` → **`not_authed` / `invalid_auth`** | Wrong/expired `SLACK_BOT_TOKEN` (`xoxb-…`). Re-copy from OAuth & Permissions. |
| **`missing_scope`** | A scope from the manifest wasn't granted — re-check `oauth_config.scopes.bot`, then **reinstall** the app. |
| Bot never receives DMs | Socket Mode off, `message.im` not subscribed, or `im:history` missing → fix manifest + reinstall. Confirm `bridge_slack.py` is running and `SLACK_APP_TOKEN` (`xapp-…`) is set. |
| **`channel_not_found`** on send | Use the user id `U…` (or a DM `D…` id) as `channel`, and keep `im:write` in scopes so the bot can open the DM. |
| App-level token errors / Socket Mode won't connect | `SLACK_APP_TOKEN` must be an `xapp-…` token with the **`connections:write`** scope. Regenerate under Basic Information → App-Level Tokens. |
| Bot replies to its own messages (loop) | The `bot_id`/`subtype` guard isn't applied — add it. |
| Bot replies to strangers | The allowlist check (`event["user"] == SLACK_USER_ID`) isn't applied — add it. |
| Railway build fails: `pip: not found` / `externally-managed-environment` | Add `python3-pip` to apt and use `pip3 install --break-system-packages slack_bolt`. |
| Cloud agent dies after ~15 min (401) | You used a subscription OAuth token in the cloud — use an `ANTHROPIC_API_KEY` (or `OPENAI_API_KEY` for Codex). |
| Cloud bot has no context / "I don't know your goals" | The repo's CLAUDE.md / skills copy is missing or stale — add it and re-push. |

## Docs (authoritative references)

- Socket Mode (no public URL; app-level token): <https://docs.slack.dev/apis/events-api/using-socket-mode/>
- App manifest reference: <https://docs.slack.dev/reference/app-manifest>
- `chat.postMessage`: <https://docs.slack.dev/reference/methods/chat.postMessage>
- `message.im` event: <https://docs.slack.dev/reference/events/message.im>
- Bolt for Python — Socket Mode: <https://tools.slack.dev/bolt-python/concepts/socket-mode/>
- Claude Code headless / `-p` mode: <https://code.claude.com/docs/en/headless>
- Codex non-interactive / `exec`: <https://developers.openai.com/codex/noninteractive>
