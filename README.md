# nick-claude-skills — прокачка Claude

Набор навыков (skills) для Claude под работу Ника: недвижимость Пхукета, таблицы, Telegram-боты, n8n и сервер.
Это **маркетплейс плагинов**: один раз подключаешь репозиторий в Claude, дальше ставишь и обновляешь ветки навыков как в игре.

👉 Прогресс и дерево навыков — в [LEVELS.md](LEVELS.md).

## Зачем

- Claude сразу знает твои таблицы, проекты и порядок работы — меньше разведки, меньше потраченных лимитов.
- Навык загружается в работу только когда нужен: в обычном диалоге он стоит ~100 токенов (одна строка описания).
- Новые навыки Claude находит и добавляет сам по команде «Прокачай ветку …».

## Что внутри

| Плагин | Ветка | Навыки |
|---|---|---|
| `lvl1-core` | Основа | ekonom · nick-workspace · skill-scout |
| `lvl2-realestate` | Недвижимость | phuket-developer-db · property-analysis · listing-writer · thai-property-rules |
| `lvl2-bots` | Боты и автоматизация | telegram-bot · vps-deploy · n8n-quickstart · n8n-self-hosting |
| `n8n-skills` | 🔒 n8n PRO (уровень 3) | официальные навыки n8n — ставить, когда n8n работает на сервере |

## Как залить на GitHub (один раз)

**Вариант А — через Claude Code на Mac (проще всего).**
1. Создай аккаунт на github.com.
2. Распакуй архив в папку, например `~/nick-claude-skills`.
3. Открой Claude Code в этой папке и напиши: «Залей эту папку в новый публичный репозиторий nick-claude-skills на моём GitHub». Он установит `gh`, попросит войти в GitHub через браузер и всё отправит.

**Вариант Б — через сайт GitHub.**
1. github.com → New repository → имя `nick-claude-skills` → Public → Create.
2. «uploading an existing file» → перетащи **всё содержимое** папки.
   ⚠️ Папки `.claude-plugin` на Mac скрыты: в Finder нажми `Cmd + Shift + .`, чтобы их увидеть, и перетащи вместе с остальными.
3. Commit changes.

Секретов в репозитории нет, поэтому его можно держать публичным — так проще подключать.

## Как подключить к Claude

**Claude Code (терминал):**
```
/plugin marketplace add derevankonikita18-afk/nick-claude-skills
/plugin install lvl1-core@nick-skills
/plugin install lvl2-realestate@nick-skills
/plugin install lvl2-bots@nick-skills
```

**Claude (приложение / Cowork):** в разделе плагинов добавь маркетплейс по адресу `derevankonikita18-afk/nick-claude-skills` и установи нужные плагины.

Обновление после прокачки: `/plugin marketplace update nick-skills` (или «обновить» у маркетплейса в приложении).

## Как прокачивать

Просто скажи Claude:
- «Прокачай ветку YouTube» — найдёт, проверит и добавит навыки, обновит LEVELS.md;
- «Найди навык для монтажа видео»;
- «Навык phuket-developer-db устарел — в таблице новая колонка, обнови».

Без доступа к GitHub Claude отдаст изменённые файлы архивом — их нужно загрузить в репозиторий так же, как в первый раз.

## Источники

См. [CREDITS.md](CREDITS.md).
