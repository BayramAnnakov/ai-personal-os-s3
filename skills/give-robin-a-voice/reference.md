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

## telegram-check (inbound — "Robin's ears", Checker tier)

Reads only **new** messages using an offset cursor stored in `~/.robin/offset`:

```bash
#!/usr/bin/env bash
# telegram-check.sh  → prints new messages (one JSON per line), advances the cursor
source ~/.robin/telegram.conf
OFFSET=$(cat ~/.robin/offset 2>/dev/null || echo 0)
RESP=$(curl -s "https://api.telegram.org/bot${TELEGRAM_TOKEN}/getUpdates?offset=${OFFSET}&timeout=0")
# pass data via env (NOT stdin): the heredoc below IS stdin (the program), so
# json must read $RESP from the environment, not sys.stdin.
RESP="$RESP" CHAT="$TELEGRAM_CHAT_ID" python3 <<'PY'
import os, json
chat=int(os.environ["CHAT"]); data=json.loads(os.environ["RESP"]); last=None
for u in data.get("result", []):
    last=u["update_id"]
    m=u.get("message", {})
    if m.get("chat", {}).get("id")==chat and "text" in m:
        print(json.dumps({"text": m["text"], "ts": m["date"]}))
if last is not None:
    open(os.path.expanduser("~/.robin/offset"), "w").write(str(last+1))
PY
```
> *Verified live 2026-06-21: push, cursor-based check, and the full inbound→reply
> loop all work. The `RESP=… python3 <<'PY'` form is deliberate — piping `$RESP`
> into `python3 -` while also using a heredoc collides on stdin and Python reads
> nothing.*

## The scheduled "check & reply" task prompt (Checker tier)

Paste into Claude Desktop **Scheduled Tasks / Routines** or Codex **Automations**,
interval every 1–3 min:

> Run telegram-check to get new Telegram messages from me. For each one: treat it
> as an instruction, carry it out using my context, files, and skills, and reply
> with telegram-send. **Anything that sends, changes, spends, or deletes in the
> outside world: do NOT execute — reply with the draft and ask me to confirm.**
> Only process messages telegram-check returns (it tracks what's new). Keep replies
> short. Log each action to ~/.robin/actions.log.

---

## bridge.py (Listener tier — always-on, instant)

```python
#!/usr/bin/env python3
# Long-poll Telegram → run the user's agent headless → reply. Allowlist-gated.
import json, subprocess, urllib.parse, urllib.request, os, pathlib
conf = dict(l.strip().split("=",1) for l in open(os.path.expanduser("~/.robin/telegram.conf")))
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
Run locally (`python3 bridge.py`) while the machine is on, or on a **non-RU VPS**
(Railway etc.) for 24/7. Codex: swap the `cmd` line for `["codex","exec",text]`.

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
| `sendMessage` → **403 host_not_allowed** | A *cloud* run (Routine) restricts outbound hosts → add `api.telegram.org` to its network allowlist. |
| `sendMessage` → **401 / chat not found** | Wrong token or chat_id — re-run "Get chat_id". |
| Send fails in Russia | The *send* needs a VPN (api.telegram.org blocked). *Receiving* still works; the shared workshop bot is the fallback. |
| Bot replies to strangers | The allowlist check (`chat.id == CHAT_ID`) isn't applied — add it. |
| `my.telegram.org` asked for | Not needed here — that's only for telegram-mcp *reading* your chats (a power-lane bonus), not for a bot. |
