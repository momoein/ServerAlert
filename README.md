# System Resource & Security Watcher (Root Login + Monit Alerts)

This setup sends Telegram notifications when:
- Root user logs in via SSH
- CPU, RAM, or Disk usage crosses defined thresholds

## Requirements
- Ubuntu / Debian
- Telegram bot & chat ID

## Telegram Setup
Create bot using @BotFather, then get CHAT_ID:
```bash
curl -s https://api.telegram.org/bot<BOT_TOKEN>/getUpdates | jq .result[0].message.chat.id
```
## Notification Script
File: `/usr/local/bin/notify-telegram.sh`
```bash
#!/bin/bash
BOT_TOKEN="<YOUR_BOT_TOKEN>"
CHAT_ID="<YOUR_CHAT_ID>"
MESSAGE="$1"
curl -s -X POST https://api.telegram.org/bot${BOT_TOKEN}/sendMessage \
  -d chat_id=${CHAT_ID} \
  -d text="$MESSAGE" >/dev/null
```
```bash
chmod +x /usr/local/bin/notify-telegram.sh
```
-------------------------------------------------

## Root Login Alert
File: `/usr/local/bin/root-login-alert.sh`
```bash
#!/bin/bash
/usr/local/bin/notify-telegram.sh "⚠️ ROOT LOGIN detected on $(hostname) at $(date)"
```
```bash
chmod +x /usr/local/bin/root-login-alert.sh
```

Add to: `/etc/pam.d/sshd`
```
session optional pam_exec.so /usr/local/bin/root-login-alert.sh
```
-------------------------------------------------

## Monit Install
```bash
sudo apt install monit
```

Edit `/etc/monit/monitrc` and add:
```
set alert dummy@example.com

check system $HOST
  if loadavg (1min) > 2 then exec "/usr/local/bin/notify-telegram.sh '⚠️ High CPU Load on $(hostname)'"
  if memory usage > 80% then exec "/usr/local/bin/notify-telegram.sh '⚠️ High Memory Usage on $(hostname)'"
  if filesystem / usage > 85% then exec "/usr/local/bin/notify-telegram.sh '⚠️ Low Disk Space on $(hostname)'"
```
-------------------------------------------------

Enable Monit:
```bash
sudo systemctl enable --now monit
```

## Done 🎉
Root login alerts + server resource monitoring → Telegram.

