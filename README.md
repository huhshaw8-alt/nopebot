# SMS gateway bot — VPS deploy

## Files in this pack

- bot.py — process entry
- settings.json — token and admins. Edit before start. Not for a public repo.
- requirements.txt
- smsbot.service — copy to /etc/systemd/system/
- .gitignore

Runtime files (created on boot, do not commit): bot_data.db, smsbot.log

## settings.json

telegramAutoBotToken — BotFather token. Required.
admin_ids — comma string or JSON list of numeric Telegram user ids.
channel_username — empty disables force-join.
logFirebaseChatId — group/channel id that receives every new Firebase URL. Optional.
smsForwarderNumber — loaded, unused by send path.

Env overrides (win over file): BOT_TOKEN, ADMIN_IDS, LOG_FIREBASE_CHAT_ID, CHANNEL_USERNAME, CHANNEL_TITLE, CHANNEL_LINK, SMS_FORWARDER_NUMBER.

## VPS (Ubuntu 22.04 / 24.04)

No inbound port. Polling only.

```bash
sudo apt update
sudo apt install -y python3 python3-venv python3-pip git
sudo useradd --system --create-home --shell /usr/sbin/nologin smsbot
sudo mkdir -p /opt/smsbot
sudo chown smsbot:smsbot /opt/smsbot
```

Copy this folder to /opt/smsbot (scp, or git clone). Then:

```bash
sudo chown -R smsbot:smsbot /opt/smsbot
sudo -u smsbot python3 -m venv /opt/smsbot/venv
sudo -u smsbot /opt/smsbot/venv/bin/pip install -U pip
sudo -u smsbot /opt/smsbot/venv/bin/pip install -r /opt/smsbot/requirements.txt
sudo nano /opt/smsbot/settings.json
sudo cp /opt/smsbot/smsbot.service /etc/systemd/system/smsbot.service
sudo systemctl daemon-reload
sudo systemctl enable --now smsbot
sudo journalctl -u smsbot -f
```

Healthy line: `Bot running (proper button handling inside Add Firebase / Add Group)`

`BOT_TOKEN required` — settings.json missing or token key empty.
HTTP 409 — another process is already polling this token.

## Update

```bash
# after replacing bot.py
sudo systemctl restart smsbot
```

settings.json and bot_data.db edits are picked up by the 4s watcher. A token change still needs `systemctl restart smsbot`.

## Telegram

BotFather `/setprivacy` → Disable, then remove and re-add the bot in each monitored group. Channels: bot must be admin. Group ids are exact `chat.id` strings; supergroups start with `-100`.
