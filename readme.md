## ⚡ Быстрый старт (установка и расписание)
1. Клонировать репозиторий:
   ```bash
   git clone https://github.com/Evgen-rus/Scorozvon_leadgenbureau
   cd Scorozvon_leadgenbureau
   ```
2. Создать и активировать виртуальное окружение:
   ```bash
   python -m venv venv
   source venv/bin/activate
   ```
3. Установить зависимости:
   ```bash
   pip install -r requirements.txt
   ```
4. Создать папку credentional/ и положить в нее файл sheets-data-bot-b8f4cc6634fc.json

5. Создать файл `.env`:
   ```env
   SKOROZVON_LOGIN=ваш_email@domain.ru
   SKOROZVON_API_KEY=ваш_api_ключ
   SKOROZVON_APP_ID=ваш_app_id
   SKOROZVON_APP_KEY=ваш_app_secret
   GOOGLE_CREDENTIALS_FILE=credentials/sheets-data-bot-b8f4cc6634fc.json
   GOOGLE_SHEET_ID=
   ```
5. Настроить расписание cron (MSK), добавить в `crontab -e`:
   ```bash
   crontab -e
   ```
   ```
   # ==== SCOROZVON_LOAD_API SCHEDULE ====
   CRON_TZ=Europe/Moscow
   33 8-17 * * * cd /opt/Scorozvon_leadgenbureau && /opt/Scorozvon_leadgenbureau/venv/bin/python 1_save_gsheet_to_sqlite.py >/dev/null 2>> /opt/Scorozvon_leadgenbureau/logs/cron_save_gsheet.log
   35 8-17 * * * cd /opt/Scorozvon_leadgenbureau && /opt/Scorozvon_leadgenbureau/venv/bin/python 2_upload_missing_leads_from_db.py >/dev/null 2>> /opt/Scorozvon_leadgenbureau/logs/cron_upload_missing.log
   ```

## 📋 Описание проекта

Этот проект содержит скрипты для работы с API сервиса **Скорозвон** (автоматизированные звонки). Все скрипты написаны на Python и используют библиотеку `requests` для HTTP-запросов.

## ⭐ Основные рабочие скрипты

### `1_save_gsheet_to_sqlite.py`
**Назначение:** Берёт данные из Google Sheets и сохраняет их в локальную SQLite БД `lr164.db`.

**Что делает:**
- Читает вкладку `Данные` из таблицы
- Сохраняет строки в БД
- Учитывает только последние N дней (по МСК), по умолчанию 3
- Логи сохраняются в `logs/save_gsheet_to_sqlite.log` (хранится 7 дней)

### `2_upload_missing_leads_from_db.py`
**Назначение:** Создаёт лиды в Скорозвоне на основе данных из БД и синхронизирует столбец `ID проекта и ID созданного в скорозвоне лида` в Google Sheets.

**Что делает:**
- Берёт записи за последние N дней (по МСК), по умолчанию 3
- Если `skorozvon_info` уже заполнен в БД — лид **не создаёт**, а только записывает значение в таблицу
- Если `skorozvon_info` пустой — создаёт лид, пишет результат в БД и таблицу
- Есть тест‑режим на 1 лид (переменная `TEST_MODE_ONE_LEAD`)
- Логи сохраняются в `logs/upload_missing_leads_from_db.log` (хранится 7 дней)

- **Заполняет поля лидов:**
  - **Название лида:** "ФИО не указано" (или можно изменить в коде)
  - **Телефон:** из первого столбца файла
  - **Комментарий:** из второго столбца файла (застройщик/комментарий)
  - **Теги:** ["столица_перехват"] (можно изменить в коде)

## 📁 Файлы проекта

### `list_projects.py`
**Назначение:** Получает и выводит список всех доступных проектов (кампаний звонков) из аккаунта Скорозвона.

**Что делает:**
- Авторизуется в API с помощью OAuth токена
- Запрашивает проекты постранично (по 200 проектов на страницу)
- Выводит ID и названия проектов в формате: `ID — Название`

### `get_lead_by_id.py`
**Назначение:** Получает и выводит подробную информацию о лиде по его ID.

**Что делает:**
- Запрашивает ID лида у пользователя
- Получает полную информацию о лиде из API
- Выводит все доступные поля: имя, телефоны, комментарий, теги, даты создания и обновления


## 🔧 Требования для запуска

### Системные требования:
- Python 3.7+
- Установленные зависимости из `requirements.txt`:
  ```
  requests==2.32.5
  python-dotenv==1.0.1
  openpyxl==3.1.5
  ```

### Установка зависимостей:
```bash
pip install -r requirements.txt
```

## ⚙️ Настройка переменных окружения (.env)

Файл `.env` содержит чувствительные данные для доступа к API Скорозвона:

```env
SKOROZVON_LOGIN=ваш_email@domain.ru
SKOROZVON_API_KEY=ваш_api_ключ
SKOROZVON_APP_ID=ваш_app_id
SKOROZVON_APP_KEY=ваш_app_secret
```

### Где получить эти данные:

**Зарегистрируйтесь или войди** в системе Скорозвон (https://skorozvon.ru)
**Перейдите в настройки API:**
https://app.skorozvon.ru/#/settings/integration/api
**Скопируйте**
Логин:
API-ключ:
ID приложения:
Ключ приложения:

### Инструкция для клиента

1. Войдите в свой аккаунт в Скорозвоне под пользователем, у которого есть права загружать лиды:
   https://app.skorozvon.ru/

2. Перейдите в настройки API:
  https://app.skorozvon.ru/#/settings/integration/api

3. В открывшемся окне скопируйте и мне пришлите данные для интеграции:
   Логин
   API-ключ
   ID приложения
   Ключ приложения

4. Напишите мне название проекта, в который нужно загружать лиды.

5. Еще можете скинуть форму, через которую вы загружаете лиды, чтобы я прописал все поля корректно.


## 🚀 Запуск скриптов

Все скрипты запускаются из командной строки:

```bash
# Список проектов
python list_projects.py

```

### Изменение параметров:
- `project_id` на нужный ID проекта
- можно изменить `CALL_PROJECT_ID`, `DEFAULT_TAGS`, `LEAD_DEFAULT_NAME`


## ⚠️ Замечания по безопасности:
- Скрипты имеют **низкий риск** — только чтение проектов или создание контактов
- Все запросы логируются в консоль для отладки
- При ошибках авторизации скрипты завершаются с кодом 1

## 📖 Документация API:
Полная документация API доступна в файле `SkorozvonAPI.pdf` в корне проекта.