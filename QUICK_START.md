# ⚡ Быстрый старт Wagtail Docker (Ubuntu)

## Для docker-compose версии 1.29.x (ваша текущая версия)

## ⚠️ PRODUCTION SETUP (для боевого сервера)

### 🚀 Первый запуск на продакшене

```bash
# 1. Создать файл окружения
cp .env.example .env

# 2. Сгенерировать сильный SECRET_KEY
python3 -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
# Скопировать результат

# 3. Узнать IP вашего сервера (если нет домена)
curl ifconfig.me

# 4. Редактировать .env для production
nano .env

# ВАЖНО! Обязательно измените:
# - DEBUG=False (не True!)
# - SECRET_KEY=<вставьте сгенерированный ключ>
# - POSTGRES_PASSWORD=<придумайте сильный пароль>
# - ALLOWED_HOSTS=<ваш_домен_или_IP>

# Пример для IP:
# DEBUG=False
# SECRET_KEY=django-insecure-abc123xyz...
# POSTGRES_PASSWORD=MySecureP@ssw0rd123!
# ALLOWED_HOSTS=116.203.123.45

# Пример для домена:
# DEBUG=False
# SECRET_KEY=django-insecure-abc123xyz...
# POSTGRES_PASSWORD=MySecureP@ssw0rd123!
# ALLOWED_HOSTS=yourdomain.com,www.yourdomain.com

# 4. Собрать контейнеры
docker-compose build

# 5. Запустить в фоне
docker-compose up -d

# 6. Проверить статус
docker-compose ps

# 7. Посмотреть логи (проверить на ошибки)
docker-compose logs -f
```

### ✅ Чеклист для production:

- [ ] `DEBUG=False` ⚠️ (ОБЯЗАТЕЛЬНО!)
- [ ] Сгенерирован и установлен уникальный `SECRET_KEY`
- [ ] Установлен сильный `POSTGRES_PASSWORD`
- [ ] `ALLOWED_HOSTS` содержит ваш домен или IP
- [ ] `SECURE_SSL_REDIRECT=True` (если используете HTTPS)
- [ ] Firewall настроен (открыты только нужные порты)
- [ ] Настроен backup базы данных

---

**Важно:** По умолчанию `docker-compose.yml` запускает тесты Wagtail.
Для production используйте `docker-compose.dev.yml` или создайте свой проект.

### 🎯 Запуск для разработки (Django сервер)

Если хотите запустить Django сервер вместо тестов:

```bash
# Вариант 1: Изменить команду в существующем контейнере
docker-compose stop wagtail_dev
docker-compose run -d --name wagtail_web -p 8000:8000 wagtail bash -c "python manage.py runserver 0.0.0.0:8000"

# Вариант 2: Использовать docker-compose.dev.yml (рекомендуется)
docker-compose -f docker-compose.dev.yml up -d

# Вариант 3: Запустить сервер напрямую
docker-compose exec wagtail_dev bash
# Внутри контейнера:
cd /path/to/your/project
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver 0.0.0.0:8000
```

---

## 🌐 Настройка ALLOWED_HOSTS

`ALLOWED_HOSTS` - это список доменов и IP-адресов, с которых разрешен доступ к Django.

### Для локальной разработки (на своем компьютере):
```bash
ALLOWED_HOSTS=localhost,127.0.0.1,0.0.0.0
```

### Для сервера (например, Hetzner):
```bash
# Узнать IP вашего сервера
curl ifconfig.me
# или
hostname -I

# Добавить в .env
ALLOWED_HOSTS=localhost,127.0.0.1,YOUR_SERVER_IP

# Например:
ALLOWED_HOSTS=localhost,127.0.0.1,116.203.123.45
```

### Для production с доменом:
```bash
# Если у вас есть домен example.com
ALLOWED_HOSTS=example.com,www.example.com

# С IP и доменом:
ALLOWED_HOSTS=example.com,www.example.com,116.203.123.45
```

### Для wildcard поддомены (не рекомендуется для production):
```bash
ALLOWED_HOSTS=.example.com  # Разрешит все поддомены
```

**Важно:** Без правильной настройки `ALLOWED_HOSTS` вы получите ошибку:
```
DisallowedHost at /
Invalid HTTP_HOST header: 'YOUR_IP:8000'. 
You may need to add 'YOUR_IP' to ALLOWED_HOSTS.
```

---

## 📋 Основные команды

```bash
# Запуск
docker-compose up -d

# Остановка
docker-compose down

# Перезапуск
docker-compose restart

# Статус
docker-compose ps

# Логи
docker-compose logs -f

# Логи конкретного сервиса
docker-compose logs -f wagtail
docker-compose logs -f db

# Пересборка
docker-compose build --no-cache
docker-compose up -d
```

---

## 🔧 Работа с контейнерами

```bash
# Открыть bash в контейнере
docker-compose exec wagtail bash

# Запустить команду без входа в контейнер
docker-compose exec wagtail python --version

# Запустить тесты
docker-compose run --rm wagtail python runtests.py

# Django shell
docker-compose exec wagtail python manage.py shell
```

---

## 🗄️ База данных

```bash
# Подключиться к PostgreSQL
docker-compose exec db psql -U wagtail -d wagtail

# Проверить статус БД
docker-compose exec db pg_isready -U wagtail

# Сделать дамп
docker-compose exec db pg_dump -U wagtail wagtail > backup.sql

# Восстановить из дампа
docker-compose exec -T db psql -U wagtail wagtail < backup.sql

# Применить миграции
docker-compose exec wagtail python manage.py migrate

# Создать суперпользователя
docker-compose exec wagtail python manage.py createsuperuser
```

---

## 🧹 Очистка

```bash
# Остановить и удалить контейнеры
docker-compose down

# Остановить и удалить volumes (БД будет удалена!)
docker-compose down -v

# Полная очистка
docker-compose down -v --rmi all
docker system prune -a --volumes
```

---

## 🐛 Решение проблем

### Ошибка: "unknown flag: --build"

```bash
# Вместо docker-compose up --build используйте:
docker-compose build
docker-compose up -d
```

### Ошибка: "port is already allocated"

```bash
# Найти процесс на порту
sudo lsof -i :8000

# Убить процесс
sudo kill -9 <PID>

# Или остановить все контейнеры
docker-compose down
```

### Ошибка: "permission denied"

```bash
# Добавить себя в группу docker
sudo usermod -aG docker $USER
newgrp docker

# Или использовать sudo
sudo docker-compose up -d
```

### Контейнер не запускается

```bash
# Посмотреть логи
docker-compose logs wagtail

# Пересобрать с нуля
docker-compose down -v
docker-compose build --no-cache
docker-compose up -d
```

---

## 📊 Мониторинг

```bash
# Использование ресурсов
docker stats

# Статус контейнеров
docker-compose ps

# Размер контейнеров
docker-compose images

# Логи в реальном времени
docker-compose logs -f --tail=50
```

---

## 🔑 Доступ к сервисам

### Текущий docker-compose.yml (тесты):
- **Wagtail тесты**: Запускает `python runtests.py`
- **PostgreSQL**: localhost:5432
- **Frontend**: http://localhost:3000

### С docker-compose.dev.yml (Django сервер):
- **Django Admin**: http://localhost:8000/admin или http://YOUR_SERVER_IP:8000/admin
- **Wagtail Admin**: http://localhost:8000/admin
- **PostgreSQL**: localhost:5432
- **Frontend dev**: http://localhost:3000

### Создание суперпользователя:

```bash
# Если используете docker-compose.dev.yml
docker-compose -f docker-compose.dev.yml exec web python manage.py createsuperuser

# Если используете docker-compose.yml
docker-compose exec wagtail_dev bash
# Затем внутри контейнера создайте Django проект или используйте существующий
```

---

## ⚡ Алиасы для .bashrc

Добавьте в `~/.bashrc`:

```bash
alias dc='docker-compose'
alias dcup='docker-compose up -d'
alias dcdown='docker-compose down'
alias dcps='docker-compose ps'
alias dclogs='docker-compose logs -f'
alias dcbuild='docker-compose build --no-cache'
alias dcrestart='docker-compose restart'
alias dcexec='docker-compose exec wagtail bash'
```

Применить:
```bash
source ~/.bashrc
```

Теперь можно использовать короткие команды:
```bash
dc up -d        # Запуск
dcps            # Статус
dclogs          # Логи
dcexec          # Bash в контейнере
dcdown          # Остановка
```

---

**Готово!** 🎉
