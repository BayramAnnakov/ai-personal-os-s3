# give-robin-a-voice — reference (scripts & exact steps)

Everything the wizard installs. All of it is plain Bot API (HTTPS) — works the
same whether the user is on Claude Code or Codex. Replace `<TOKEN>` / `<CHAT_ID>`
from the user's bot; the scripts read them from `~/.robin/telegram.conf`.

---

## Store secrets

```bash
mkdir -p ~/.robin && chmod 700 ~/.robin
cat > ~/.robin/telegram.conf <<EOF
TELEGRAM_TOKEN=<TOKEN>
TELEGRAM_CHAT_ID=<CHAT_ID>
EOF
chmod 600 ~/.robin/telegram.conf
# if inside a git repo, keep it out of version control:
grep -qxF '.robin/' .gitignore 2>/dev/null || echo '.robin/' >> .gitignore
```

## Get chat_id (after the user messages their bot once)

```bash
source ~/.robin/telegram.conf
curl -s "https://api.telegram.org/bot${TELEGRAM_TOKEN}/getUpdates" \
  | python3 -c 'import sys,json; u=json.load(sys.stdin)["result"]; print(u[-1]["message"]["chat"]["id"]) if u else print("no messages yet — ask them to text the bot")'
```

---

## telegram-send (outbound — "Robin's mouth")

Install as a skill (`.claude/skills/telegram-send/` for CC, `~/.codex/skills/telegram-send/`
for Codex) wrapping this script, or just drop the script at `~/.robin/telegram-send.sh`:

```bash
#!/usr/bin/env bash
# telegram-send.sh "message text"
source ~/.robin/telegram.conf
curl -s "https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage" \
  -d chat_id="${TELEGRAM_CHAT_ID}" \
  --data-urlencode text="$1" \
  -d parse_mode=Markdown | grep -q '"ok":true' && echo "sent" || echo "FAILED"
```
Invoke: *"Robin, use telegram-send to text me …"* → he runs `telegram-send.sh "…"`.

## bridge.py (the listener — always-on, instant)

Long-polls Telegram, runs the user's agent headless per message, replies.
Allowlist-gated. Reads secrets from `~/.robin/telegram.conf` **locally** or from
**environment variables** in the cloud — so the same file works in both homes.

```python
#!/usr/bin/env python3
# Long-poll Telegram → run the user's agent headless → reply. Allowlist-gated.
import json, subprocess, urllib.parse, urllib.request, os, pathlib

def load_conf():
    p = os.path.expanduser("~/.robin/telegram.conf")          # local
    if os.path.exists(p):
        return dict(l.strip().split("=",1) for l in open(p) if "=" in l)
    return {"TELEGRAM_TOKEN": os.environ["TELEGRAM_TOKEN"],    # cloud (Railway vars)
            "TELEGRAM_CHAT_ID": os.environ["TELEGRAM_CHAT_ID"]}

conf = load_conf()
TOKEN, CHAT = conf["TELEGRAM_TOKEN"], int(conf["TELEGRAM_CHAT_ID"])
API = f"https://api.telegram.org/bot{TOKEN}"
SESS = pathlib.Path(os.path.expanduser("~/.robin/session"))

def api(method, **p):
    return json.load(urllib.request.urlopen(f"{API}/{method}?"+urllib.parse.urlencode(p)))

def agent(text):
    # Claude Code (full context — NOT --bare). Codex: ["codex","exec",text]
    cmd = ["claude","-p",text,"--output-format","json",
           "--permission-mode","dontAsk","--allowedTools","Read,Bash(telegram-send.sh *)"]
    if SESS.exists(): cmd += ["--resume", SESS.read_text().strip()]
    out = json.loads(subprocess.run(cmd, capture_output=True, text=True).stdout)
    SESS.write_text(out.get("session_id",""))
    return out.get("result","(no reply)")

offset = 0
while True:
    for u in api("getUpdates", offset=offset, timeout=30).get("result", []):
        offset = u["update_id"] + 1
        m = u.get("message", {})
        if m.get("chat",{}).get("id") != CHAT: continue          # allowlist
        if "text" not in m: continue
        api("sendChatAction", chat_id=CHAT, action="typing")
        api("sendMessage", chat_id=CHAT, text=agent(m["text"]))
```
Run locally first — `python3 bridge.py` while the machine is on — to prove the
loop. Codex: swap the `cmd` line for `["codex","exec",text]`.

> *Verified live 2026-06-21: push, cursor-based polling, and the full
> inbound→reply loop all work end to end.*

---

## Deploy to Railway (24/7, no laptop)

The listener becomes always-on by running on a small cloud box. **Railway** fits
because it runs a persistent process (Vercel can't — its ~10s function cap kills
an agent run). Bonus: if a local network blocks `api.telegram.org`, Railway's
egress isn't restricted, so the *send* works with no VPN.

**Repo layout** (a private GitHub repo Robin deploys from):
```
robin-listener/
├── bridge.py                 # the listener above
├── telegram-send.sh          # Robin's mouth (same script)
├── Dockerfile                # builds the agent CLI + python
├── CLAUDE.md  (or AGENTS.md)  # a MINIMAL copy of Robin's context
└── skills/                   # only the skills the listener needs
```

**Dockerfile** (Claude Code path; Codex swap noted):
```dockerfile
FROM node:22-slim
RUN apt-get update && apt-get install -y python3 && rm -rf /var/lib/apt/lists/*
RUN npm install -g @anthropic-ai/claude-code      # Codex: @openai/codex
WORKDIR /robin
COPY . .
CMD ["python3", "bridge.py"]                        # a worker — no exposed port
```

**Deploy steps** (you walk them through; they click + paste):
1. Push `robin-listener/` to a **private** GitHub repo (never commit the token —
   it arrives as a Railway variable, not a file).
2. Railway → **New Project → Deploy from GitHub repo** → pick the repo. Railway
   auto-detects the Dockerfile and runs it as a background worker.
3. Railway → **Variables**, add: `TELEGRAM_TOKEN`, `TELEGRAM_CHAT_ID`,
   `ANTHROPIC_API_KEY` (Codex: the Codex/OpenAI key). `bridge.py` reads these
   from the env; the `claude` CLI picks up `ANTHROPIC_API_KEY` automatically.
4. Deploy. Text the bot with the laptop **closed** — it answers.

**Cost & auth — say it plainly.** Two bills: the **host** (~$5/mo — Railway
**Hobby**, $5/mo incl. $5 usage; a tiny worker's ~$1 fits inside it) and the
**agent** via `ANTHROPIC_API_KEY` (per-use tokens). The cloud agent needs an API
key, **not** a subscription: a headless OAuth token (`claude setup-token`) 401s
within ~15 min (it doesn't refresh non-interactively) and the terms route
always-on automation to the API path. So: **local** listener = the subscription
they're already logged into, $0 (laptop on); **cloud** = API key + host. Cheaper
host, same tokens: **Fly.io**.

**Railway ToS:** a BotFather **Bot API** bot replying to its owner is allowed —
the Fair Use clause only bars "bots or scrapers that violate applicable terms of
service," and the "userbot" ban is for MTProto user-account automation (the
`my.telegram.org` path we don't use). If unsure, Railway invites you to ask first.

> **Context drift:** the cloud Robin only knows what's in the repo. Keep that
> CLAUDE.md / skills copy minimal and re-push when it changes. Session resume is
> best-effort — Railway's filesystem resets on redeploy, so `~/.robin/session`
> won't survive a deploy. That's fine for a Q&A bot; don't rely on it for long
> threaded state.

---

## Tie in the morning brief (so it lands in the same chat)

The scheduled brief just calls `telegram-send` like everything else — point its
push at the same bot, and now you can **reply to the brief** ("draft the prep
doc") and Robin acts on it. One bot, one mechanism: push, brief, and two-way.

## Streaming (optional polish)

For a "Robin is typing" feel, stream partial output into a Telegram **draft**
with `sendMessageDraft` (Bot API 9.4+, all bots) fed by
`claude -p … --output-format stream-json --include-partial-messages`.

## Troubleshooting

| Symptom | Fix |
|---|---|
| `sendMessage` → **401 / chat not found** | Wrong token or chat_id — re-run "Get chat_id". |
| Send fails on a restricted network (local listener) | The *send* needs a VPN if the network blocks api.telegram.org while running locally. Deploy to Railway and it disappears — its egress isn't restricted. *Receiving* always works. |
| Bot replies to strangers | The allowlist check (`chat.id == CHAT_ID`) isn't applied — add it. |
| Cloud bot has no context / "I don't know your goals" | The repo's CLAUDE.md / skills copy is missing or stale — add it and re-push. |
| Cloud agent dies after ~15 min (401) | You used a subscription OAuth token (`CLAUDE_CODE_OAUTH_TOKEN`) in the cloud — it doesn't refresh headless. Use an `ANTHROPIC_API_KEY` for the cloud listener. |
| Railway build fails on `claude: not found` | The `npm install -g` line didn't run — confirm the Dockerfile (Node base image) is detected, not Nixpacks. |
| `my.telegram.org` asked for | Not needed here — that's only for telegram-mcp *reading* your chats (a power-lane bonus), not for a bot. |
