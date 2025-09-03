## Проект Kittygram - социальная сеть для кошек

Kittygram — это веб‑приложение, где пользователи могут делиться карточками своих котиков: создавать записи с фото, цветом, достижениями и описанием, просматривать карточки других, редактировать и удалять свои, а также авторизоваться для доступа к расширенным функциям.

### Основные возможности
- **Регистрация и авторизация** (Token Auth через Djoser)
- **Создание/редактирование/удаление** карточек котиков
- **Загрузка изображений** и хранение медиа
- **Пагинация** и удобная навигация по карточкам
- **REST API** для фронтенда и внешних интеграций

## Стек
- **Backend**: Django 3.2, Django REST Framework, Djoser, PostgreSQL, Gunicorn
- **Frontend**: React (react-scripts), React Router
- **Infra/CI/CD**: Docker, Docker Compose, Nginx, GitHub Actions

## Продакшен (Docker Compose)
Под продакшен используется `docker-compose.production.yml` с тремя сервисами: `backend`, `frontend`, `gateway` (Nginx) и `db` (PostgreSQL).

1. Подготовьте `.env` в корне репозитория (см. ниже).
2. Запустите:
   ```bash
   docker compose -f docker-compose.production.yml pull
   docker compose -f docker-compose.production.yml up -d
   docker compose -f docker-compose.production.yml exec backend python manage.py migrate
   docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic --noinput
   docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /static/
   ```

Состав образов (ожидаются публичными в Docker Hub):
- `kittygram_backend`
- `kittygram_frontend`
- `kittygram_gateway`

## Переменные окружения (.env)
Файл `.env` читается сервисами из `docker-compose.production.yml` и `backend/kittygram_backend/settings.py`.

Минимальный пример `.env` для продакшена:
```dotenv
# Django
SECRET_KEY=change_me
DJANGO_DEBUG=False
ALLOWED_HOSTS=example.com,www.example.com,127.0.0.1,localhost

# БД PostgreSQL
POSTGRES_DB=django_db
POSTGRES_USER=django_user
POSTGRES_PASSWORD=django_password
DB_HOST=db
DB_PORT=5432
```

Секреты для CI/CD (GitHub Actions → Settings → Secrets and variables → Actions → New repository secret):
- `DOCKER_USERNAME`, `DOCKER_PASSWORD`
- `HOST` (адрес сервера), `USER` (пользователь на сервере)
- `SSH_KEY`, `SSH_PASSPHRASE`
- `TELEGRAM_TO`, `TELEGRAM_TOKEN`

## CI/CD
Workflow расположен в файле `kittygram_workflow.yml`. Он:
- запускает тесты бекенда и фронтенда;
- собирает и публикует образы в Docker Hub;
- разворачивает контейнеры на сервере с помощью `docker-compose.production.yml`.