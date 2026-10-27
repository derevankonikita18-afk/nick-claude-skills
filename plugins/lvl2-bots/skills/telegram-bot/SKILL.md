---
name: telegram-bot
description: Создание и доработка Telegram-ботов на Python (aiogram 3) — команды, кнопки, ответы через Claude API, запись в Google Sheets, автоответчик от имени личного аккаунта через Telegram Business, бот в группах с менеджерами застройщиков, запуск на VPS. Использовать для @Raumer_bot и любых новых ботов Ника, а также вопросов «как сделать бота, который отвечает за меня».
---

# Telegram-боты (aiogram 3)

## Выбор подхода

| Задача | Решение |
|---|---|
| Простой бот: команды, кнопки, уведомления | aiogram 3 + polling, один файл |
| Бот пишет/читает Google Sheets | + `gspread` и сервисный аккаунт Google (JSON-ключ в `.env`/файле на сервере) |
| Умные ответы | + Anthropic API (`anthropic` SDK), модель — самая дешёвая, что справляется (Haiku) |
| Отвечать **за Ника** в личных чатах Telegram | Telegram Business: бот подключается в Настройки → Telegram для бизнеса → Чат-боты (нужен Telegram Premium). Бот получает `business_message` и отвечает с `business_connection_id` |
| Читать группы с менеджерами | Добавить бота в группу; в @BotFather отключить Group Privacy, иначе бот видит только команды |
| Без кода, быстро | n8n: Telegram Trigger → обработка → Google Sheets (навык `n8n-quickstart`) |

WhatsApp: официально — только WhatsApp Business Platform (Cloud API) через Meta или провайдера. Неофициальные библиотеки для личного WhatsApp — риск бана номера, предупреждать.

## Минимальный шаблон

```
bot/
  main.py
  .env            # BOT_TOKEN=..., ANTHROPIC_API_KEY=... (не коммитить!)
  .gitignore      # .env, *.json с ключами, venv/
  requirements.txt  # aiogram>=3, python-dotenv, (anthropic, gspread)
```

```python
import asyncio, os
from aiogram import Bot, Dispatcher, F
from aiogram.filters import CommandStart
from aiogram.types import Message
from dotenv import load_dotenv

load_dotenv()
bot = Bot(os.environ["BOT_TOKEN"])
dp = Dispatcher()

@dp.message(CommandStart())
async def start(m: Message):
    await m.answer("Привет! Я бот Ника.")

@dp.business_message(F.text)          # автоответ в личных чатах через Telegram Business
async def biz(m: Message):
    await bot.send_message(m.chat.id, "Ник скоро ответит 🙌",
                           business_connection_id=m.business_connection_id)

async def main():
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
```

Перед тем как писать код под новую версию aiogram — сверься с документацией (docs.aiogram.dev), API менялся между версиями.

## Автоответчик: правила

- Отвечать только по сценарию (часы работы, базовые вопросы, «Ник ответит позже»); сложное — переслать Нику уведомлением.
- Не отвечать повторно одному человеку чаще раза в N часов (хранить время последнего ответа: SQLite или Sheet).
- Не отвечать в чаты, где Ник уже написал сам недавно.
- Ответы через Claude — короткий системный промпт с фактами о Нике, лимит токенов на ответ, лог расходов.

## Безопасность

- Токен бота — только в `.env` на сервере. Если токен попал в чат/GitHub — сразу перевыпустить в @BotFather (/revoke).
- Проверять `from_user.id` для админ-команд (список ID в `.env`).

## Запуск

Локально: `python -m venv venv && source venv/bin/activate && pip install -r requirements.txt && python main.py`.
На сервере — навык `vps-deploy` (systemd-сервис, автоперезапуск, логи).
