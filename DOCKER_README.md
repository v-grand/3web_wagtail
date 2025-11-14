# Wagtail Docker - Инструкции по запуску

## Быстрый старт

### 1. Сборка и запуск контейнеров

```powershell
docker-compose up --build
```

### 2. Запуск в фоновом режиме

```powershell
docker-compose up -d
```

### 3. Остановка контейнеров

```powershell
docker-compose down
```

---

## Доступные сервисы

- **Wagtail (тесты)**: `http://localhost:8000`
- **PostgreSQL**: `localhost:5432`
- **Frontend (development)**: `http://localhost:3000`

---

## Команды для работы с Docker

### Запустить тесты

```powershell
docker-compose run --rm wagtail python runtests.py
```

### Запустить интерактивную оболочку Python

```powershell
docker-compose run --rm wagtail python manage.py shell
```

### Запустить bash внутри контейнера

```powershell
docker-compose exec wagtail bash
```

### Пересобрать контейнеры

```powershell
docker-compose build --no-cache
```

### Просмотр логов

```powershell
# Все сервисы
docker-compose logs -f

# Конкретный сервис
docker-compose logs -f wagtail
```

### Запустить линтинг

```powershell
docker-compose run --rm wagtail npm run lint
```

### Запустить форматирование кода

```powershell
docker-compose run --rm wagtail npm run format
```

---

## Создание нового Wagtail-сайта в Docker

### 1. Создать новый проект

```powershell
docker-compose run --rm wagtail wagtail start mysite
```

### 2. Запустить проект

```powershell
cd mysite
docker build -t mysite .
docker run -p 8000:8000 mysite
```

Или создать отдельный `docker-compose.yml` для вашего проекта.

---

## Работа с базой данных PostgreSQL

### Подключение к PostgreSQL

```powershell
docker-compose exec db psql -U wagtail -d wagtail
```

### Создать миграции

```powershell
docker-compose run --rm wagtail python manage.py makemigrations
```

### Применить миграции

```powershell
docker-compose run --rm wagtail python manage.py migrate
```

---

## Разработка

### Frontend в режиме разработки (hot reload)

```powershell
docker-compose up frontend
```

### Storybook

```powershell
docker-compose run --rm -p 6006:6006 wagtail npm run storybook
```

---

## Очистка

### Удалить все контейнеры и volumes

```powershell
docker-compose down -v
```

### Удалить образы

```powershell
docker-compose down --rmi all
```

---

## Переменные окружения

Настройки по умолчанию в `docker-compose.yml`:

- `POSTGRES_DB=wagtail`
- `POSTGRES_USER=wagtail`
- `POSTGRES_PASSWORD=wagtail`
- `DJANGO_SETTINGS_MODULE=wagtail.test.settings`

Для изменения создайте файл `.env` в корне проекта.

---

## Troubleshooting

### Проблемы с правами доступа

Если возникают проблемы с правами доступа к файлам:

```powershell
docker-compose run --rm wagtail chown -R $(id -u):$(id -g) /app
```

### Очистка кеша npm/pip

```powershell
docker-compose build --no-cache
```

### Проверка статуса контейнеров

```powershell
docker-compose ps
```
