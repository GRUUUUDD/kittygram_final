# Kittygram (финальный проект)

**Kittygram** — учебное веб-приложение (Django REST + React): коты, достижения, загрузка изображений. В этом репозитории проект упакован в **Docker** (backend, frontend, PostgreSQL, Nginx-gateway) и настроен **CI/CD** в GitHub Actions: линтер, тесты, сборка образов и публикация на Docker Hub, уведомление в Telegram.

## Структура репозитория

| Путь | Назначение |
|------|------------|
| `backend/` | Django-проект `kittygram_backend`, приложение `cats` |
| `frontend/` | React (сборка статики в общий volume) |
| `nginx/` | Образ **kittygram_gateway**: раздача статики и медиа, прокси на API и админку |
| `docker-compose.yml` | Локальная разработка: `build` сервисов |
| `docker-compose.production.yml` | Прод: образы с Docker Hub, без `build` |
| `.github/workflows/main.yml` | CI/CD (копия логики дублируется в `kittygram_workflow.yml` для автопроверок) |
| `tests.yml` | Домены деплоя и логины для pytest-проверок с сетью |
| `tests/` | Автотесты (инфраструктура + проверки доступности сайтов) |

Имена сервисов и контейнеров: `db`, `backend`, `frontend`, `gateway`. Тома: `static`, `media`, `pg_data`.

## Локальный запуск в Docker

1. Склонируй репозиторий и перейди в корень проекта.

2. Создай файл `.env` (можно скопировать с примера):

   ```bash
   cp .env.example .env
   ```

   При необходимости поправь `SECRET_KEY`, `ALLOWED_HOSTS`, пароли БД.

3. Подними стек:

   ```bash
   docker compose up --build -d
   ```

4. Открой в браузере: **http://127.0.0.1:9000/** (порт проброшен на `gateway`).

Остановка:

```bash
docker compose down
```

## Локальные тесты и линтер

Из корня репозитория:

```bash
python3 -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r backend/requirements.txt ruff flake8
```

- Линтер **ruff** (как в CI):

  ```bash
  ruff check backend/
  ```

- Опционально **flake8** (если установлен, настройки в `.flake8`):

  ```bash
  flake8 backend/
  ```

- Тесты **только структуры репозитория** (быстро, без интернета):

  ```bash
  pytest tests/test_files.py
  ```

- **Полный** pytest (нужен корректный `tests.yml` с реальными `https://` доменами и образами на Docker Hub):

  ```bash
  pytest
  ```

## CI/CD (GitHub Actions)

При пуше в ветку **`main`**:

1. Устанавливаются зависимости backend, запускаются **ruff** и **pytest** (инфраструктурные тесты + тесты фронта в режиме CI).
2. После успеха собираются и пушатся образы:
   - `<DOCKER_USERNAME>/kittygram_backend:latest`
   - `<DOCKER_USERNAME>/kittygram_frontend:latest`
   - `<DOCKER_USERNAME>/kittygram_gateway:latest`
3. В **Telegram** уходит уведомление об успешном завершении.

### Секреты в GitHub

**Settings → Secrets and variables → Actions:**

| Секрет | Описание |
|--------|----------|
| `DOCKER_USERNAME` | Логин Docker Hub |
| `DOCKER_PASSWORD` | Пароль или Access Token Docker Hub |
| `TELEGRAM_TOKEN` | Токен бота от [@BotFather](https://t.me/BotFather) |
| `TELEGRAM_TO` | Твой числовой **chat_id** (не бот; узнать: [@userinfobot](https://t.me/userinfobot)). Сначала нажми **Start** у своего бота в Telegram. |

Ошибка `bots can't send messages to bots` означает, что в `TELEGRAM_TO` попал ID бота или неверное значение — нужен **твой** пользовательский chat id.

---

## Как работать с репозиторием финального задания (Яндекс Практикум)

### Что нужно сделать

Настроить запуск проекта Kittygram в контейнерах и CI/CD с помощью GitHub Actions.

### Как проверить работу с помощью автотестов

В корне репозитория создайте файл `tests.yml` со следующим содержимым:

```yaml
repo_owner: ваш_логин_на_гитхабе
kittygram_domain: полная ссылка (https://доменное_имя) на ваш проект Kittygram
taski_domain: полная ссылка (https://доменное_имя) на ваш проект Taski
dockerhub_username: ваш_логин_на_докерхабе
```

Скопируйте содержимое файла `.github/workflows/main.yml` в файл `kittygram_workflow.yml` в корневой директории проекта.

Для локального запуска тестов создайте виртуальное окружение, установите в него зависимости из `backend/requirements.txt` и запустите в корневой директории проекта `pytest`.

### Чек-лист для проверки перед отправкой задания

- Проект Taski доступен по доменному имени, указанному в `tests.yml`.
- Проект Kittygram доступен по доменному имени, указанному в `tests.yml`.
- Пуш в ветку `main` запускает тестирование и деплой Kittygram, а после успешного деплоя вам приходит сообщение в телеграм.
- В корне проекта есть файл `kittygram_workflow.yml`.

На сервере общий **Nginx** должен направлять запросы на Taski и Kittygram по соответствующим доменам; образы с Docker Hub подключаются через `docker-compose.production.yml` (переменная `DOCKER_USERNAME` на машине должна совпадать с логином в тегах образов).
