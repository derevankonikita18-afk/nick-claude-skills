---
name: vps-deploy
description: Настройка дешёвого VPS (Ubuntu, $5–7/мес) под ботов и автоматизацию Ника — первый вход, безопасность (SSH-ключи, firewall), запуск Python-бота как systemd-сервиса, Docker, домен и HTTPS, логи, бэкапы, обновления. Использовать, когда Ник арендует сервер, переносит бота на сервер, бот «упал» или нужно что-то поставить на VPS.
---

# VPS для ботов и n8n

Команды выполняет Claude Code на Mac Ника по SSH (или Ник копирует их сам). Давать команды готовыми к копированию, по одному шагу, объясняя одной строкой зачем.

## Выбор сервера

1 vCPU / 1–2 GB RAM, Ubuntu 24.04 LTS — хватит для 1–3 ботов. Для n8n — от 2 GB RAM. Регион — Сингапур (ближе к Таиланду). Провайдеры: Hetzner, DigitalOcean, Vultr, Timeweb и т.п. — сравнить цены на момент покупки.

## Первичная настройка (один раз)

```bash
# на Mac: ключ, если его ещё нет
ssh-keygen -t ed25519 -C "nick-mac"
ssh-copy-id root@IP

# на сервере
apt update && apt -y upgrade
adduser nick && usermod -aG sudo nick
rsync --archive --chown=nick:nick ~/.ssh /home/nick
# запрет входа по паролю и под root:
sed -i 's/^#\?PasswordAuthentication.*/PasswordAuthentication no/; s/^#\?PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
systemctl restart ssh   # перед этим проверить вход как nick в новом окне!
ufw allow OpenSSH && ufw allow 80,443/tcp && ufw enable
apt -y install unattended-upgrades fail2ban python3-venv git
```

## Python-бот как сервис

```bash
cd ~ && git clone <репо бота> bot && cd bot
python3 -m venv venv && venv/bin/pip install -r requirements.txt
nano .env   # вписать токены (не коммитить)
```
`/etc/systemd/system/bot.service`:
```ini
[Unit]
Description=Telegram bot
After=network-online.target

[Service]
User=nick
WorkingDirectory=/home/nick/bot
ExecStart=/home/nick/bot/venv/bin/python main.py
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl daemon-reload && sudo systemctl enable --now bot
journalctl -u bot -f          # логи
sudo systemctl restart bot    # после обновления: git pull → restart
```

## Docker и n8n

Docker: `curl -fsSL https://get.docker.com | sh && sudo usermod -aG docker nick`.
n8n с доменом и HTTPS — навык `n8n-self-hosting` (Docker Compose + Caddy). Нужен домен/поддомен с A-записью на IP сервера.

## Обслуживание

- Бэкап: `.env`, базы (SQLite/Postgres-дамп), данные n8n — раз в день cron'ом в архив + копия вне сервера.
- Бот упал → `systemctl status bot` и `journalctl -u bot -n 50`, затем чинить причину, а не просто перезапускать.
- Диск: `df -h`; логи Docker чистить `docker system prune` (осторожно, спросить Ника).
- Пароли и ключи сервера — в менеджере паролей Ника, не в чате.
