# 🚀 Wagtail Docker - Полный гайд по развертыванию (Ubuntu Linux)

## 📋 Предварительные требования

- **Docker** и **Docker Compose** для Ubuntu
- **Git** для клонирования репозитория

### Установка Docker на Ubuntu

```bash
# Обновить систему
sudo apt update && sudo apt upgrade -y

# Вариант 1: Быстрая установка (старая версия docker-compose)
sudo apt install -y docker.io docker-compose

# Вариант 2: Установка последней версии Docker (рекомендуется)
# Установить зависимости
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Добавить GPG ключ Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Добавить репозиторий Docker
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Установить Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Добавить пользователя в группу docker (чтобы не использовать sudo)
sudo usermod -aG docker $USER

# Перезайти в систему или выполнить
newgrp docker

# Проверить установку
docker --version
docker-compose --version
# или для новой версии:
docker compose version
```

**Важно:** В зависимости от версии используйте либо `docker-compose` (старая версия), либо `docker compose` (новая версия, plugin).

---

## 1️⃣ Клонирование и подготовка проекта

```bash
# Клонировать репозиторий (если еще не сделано)
git clone https://github.com/v-grand/3web_wagtail.git
cd 3web_wagtail

# Или перейти в существующую директорию
cd ~/3web_wagtail
```

---

## 2️⃣ Настройка файла окружения

### Создать файл .env

```bash
# Скопировать пример файла окружения
cp .env.example .env
```

### Отредактировать .env

Откройте файл `.env` в редакторе и настройте переменные:

```env
# Основные настройки Django
DEBUG=1
SECRET_KEY=ваш-секретный-ключ-минимум-50-символов-случайных
DJANGO_SETTINGS_MODULE=wagtail.test.settings
ALLOWED_HOSTS=localhost,127.0.0.1

# База данных PostgreSQL
DATABASE_URL=postgresql://wagtail:wagtail@db:5432/wagtail
POSTGRES_DB=wagtail
POSTGRES_USER=wagtail
POSTGRES_PASSWORD=wagtail

# Остальные настройки можно оставить по умолчанию
PYTHONUNBUFFERED=1
PYTHONDONTWRITEBYTECODE=1
```

### Генерация SECRET_KEY

```bash
# Вариант 1: Использовать Python
python3 -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"

# Вариант 2: Использовать OpenSSL
openssl rand -base64 50

# Вариант 3: Использовать /dev/urandom
cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 50 | head -n 1
```

### Редактировать .env

```bash
# Использовать nano
nano .env

# Или vim
vim .env

# Или VS Code
code .env
```

---

## 3️⃣ Первый запуск (сборка контейнеров)

```bash
# Для старой версии docker-compose (1.29.x)
docker-compose up --build

# Или в фоновом режиме
docker-compose up --build -d

# Для новой версии docker compose (plugin)
docker compose up --build -d
```

**Примечание:** Если получаете ошибку "unknown flag: --build", используйте:
```bash
docker-compose build
docker-compose up -d
```

**Что произойдет:**
- ✅ Установятся Python зависимости (~3-5 минут)
- ✅ Установятся Node.js зависимости (~2-3 минуты)
- ✅ Соберутся фронтенд ассеты
- ✅ Запустится PostgreSQL
- ✅ Запустятся контейнеры Wagtail и Frontend

---

## 4️⃣ Проверка статуса контейнеров

```bash
# Посмотреть запущенные контейнеры
docker-compose ps
# или
docker compose ps

# Должно быть 3 контейнера:
# - wagtail_dev (порт 8000)
# - wagtail_db (порт 5432)
# - wagtail_frontend (порт 3000)
```

---

## 5️⃣ Создание нового Wagtail сайта (опционально)

Если хотите создать свой проект на базе Wagtail:

```bash
# Создать новый проект
docker compose run --rm wagtail wagtail start mysite

# Перейти в папку проекта
cd mysite

# Применить миграции
docker compose run --rm wagtail python manage.py migrate

# Создать суперпользователя
docker compose run --rm wagtail python manage.py createsuperuser

# Собрать статику
docker compose run --rm wagtail python manage.py collectstatic --noinput

# Запустить сервер
docker compose run --rm -p 8000:8000 wagtail python manage.py runserver 0.0.0.0:8000
```

---

## 6️⃣ Основные команды для работы

### Управление контейнерами

```bash
# Запустить все сервисы
docker compose up

# Запустить в фоне
docker compose up -d

# Остановить все сервисы
docker compose down

# Остановить с удалением volumes (БД будет очищена!)
docker compose down -v

# Перезапустить конкретный сервис
docker compose restart wagtail

# Просмотреть статус
docker compose ps
```

### Просмотр логов

```bash
# Все логи
docker compose logs -f

# Логи конкретного сервиса
docker compose logs -f wagtail
docker compose logs -f db
docker compose logs -f frontend

# Последние 100 строк
docker compose logs --tail=100 wagtail
```

### Работа внутри контейнера

```bash
# Открыть bash в контейнере Wagtail
docker compose exec wagtail bash

# Открыть Python shell Django
docker compose exec wagtail python manage.py shell

# Подключиться к PostgreSQL
docker compose exec db psql -U wagtail -d wagtail
```

### Запуск тестов

```bash
# Запустить все тесты Python
docker compose run --rm wagtail python runtests.py

# Запустить конкретный тест
docker compose run --rm wagtail python runtests.py wagtail.tests.test_hooks

# Запустить JavaScript тесты
docker compose run --rm wagtail npm test

# С coverage
docker compose run --rm wagtail npm run test:unit:coverage
```

### Работа с базой данных

```bash
# Создать миграции
docker compose run --rm wagtail python manage.py makemigrations

# Применить миграции
docker compose run --rm wagtail python manage.py migrate

# Создать суперпользователя
docker compose run --rm wagtail python manage.py createsuperuser

# Сделать дамп БД
docker compose exec db pg_dump -U wagtail wagtail > backup.sql

# Восстановить из дампа
docker compose exec -T db psql -U wagtail wagtail < backup.sql
```

### Frontend разработка

```bash
# Запустить frontend в dev режиме (hot reload)
docker compose up frontend

# Запустить Storybook
docker compose run --rm -p 6006:6006 wagtail npm run storybook

# Линтинг
docker compose run --rm wagtail npm run lint

# Форматирование
docker compose run --rm wagtail npm run format

# Сборка production
docker compose run --rm wagtail npm run build
```

---

## 7️⃣ Проверка работоспособности

```bash
# Проверить здоровье контейнеров
docker compose ps

# Проверить логи на ошибки
docker compose logs wagtail | grep -i error

# Проверить подключение к БД
docker compose exec db pg_isready -U wagtail

# Проверить версию Python
docker compose exec wagtail python --version

# Проверить версию Node.js
docker compose exec wagtail node --version
```

---

## 8️⃣ Доступ к сервисам

После успешного запуска доступны:

- **Wagtail тесты**: http://localhost:8000
- **PostgreSQL**: localhost:5432
- **Frontend dev**: http://localhost:3000
- **Storybook**: http://localhost:6006 (если запущен)

---

## 9️⃣ Troubleshooting (решение проблем)

### Ошибка: "порт уже используется"

```bash
# Найти процесс на порту 8000
sudo lsof -i :8000

# Или
sudo netstat -tlnp | grep :8000

# Убить процесс (замените PID)
sudo kill -9 <PID>

# Или изменить порт в docker-compose.yml
# ports:
#   - "8001:8000"
```

### Ошибка: "контейнер не запускается"

```bash
# Посмотреть подробные логи
docker compose logs wagtail

# Пересобрать с нуля
docker compose down -v
docker compose build --no-cache
docker compose up
```

### Ошибка: "нет подключения к БД"

```bash
# Проверить статус БД
docker compose ps db

# Перезапустить БД
docker compose restart db

# Подождать 10 секунд и попробовать снова
sleep 10
docker compose up wagtail
```

### Ошибка: "permission denied"

```bash
# Исправить права на файлы
sudo chown -R $USER:$USER .

# Или добавить пользователя в группу docker
sudo usermod -aG docker $USER
newgrp docker
```

### Очистить всё и начать заново

```bash
# Остановить и удалить всё
docker compose down -v --rmi all

# Удалить неиспользуемые образы и volumes
docker system prune -a --volumes

# Собрать заново
docker compose up --build
```

---

## 🔟 Production развертывание

### 1. Подготовить production окружение

```bash
# Скопировать production настройки
cp .env.production .env

# Отредактировать .env и изменить:
nano .env
# - DEBUG=0
# - SECRET_KEY (сгенерировать новый!)
# - POSTGRES_PASSWORD (надежный пароль)
# - ALLOWED_HOSTS (ваш домен)
```

### 2. Создать production docker-compose

```bash
# Использовать docker-compose.prod.yml (нужно создать)
docker compose -f docker-compose.prod.yml up --build -d
```

### 3. Настроить SSL/HTTPS

Добавить Nginx или Traefik для SSL-сертификатов (Let's Encrypt).

---

## 📊 Мониторинг

```bash
# Использование ресурсов
docker stats

# Размер контейнеров
docker compose images

# Использование дискового пространства
docker system df

# Использование диска в системе
df -h

# Использование памяти
free -h
```

---

## 🔒 Безопасность

**Важно для production:**

1. ✅ Измените `SECRET_KEY` на случайную строку
2. ✅ Установите `DEBUG=0`
3. ✅ Используйте сильные пароли для `POSTGRES_PASSWORD`
4. ✅ Настройте `ALLOWED_HOSTS` с вашим доменом
5. ✅ Включите HTTPS (`SECURE_SSL_REDIRECT=1`)
6. ✅ Настройте backup базы данных
7. ✅ Используйте Docker secrets для чувствительных данных

---

## 📚 Дополнительные команды

```bash
# Обновить зависимости
docker compose run --rm wagtail pip install -r requirements.txt --upgrade
docker compose run --rm wagtail npm update

# Создать новое приложение Django
docker compose run --rm wagtail python manage.py startapp myapp

# Запустить shell с установленными пакетами
docker compose run --rm wagtail ipython

# Экспортировать requirements
docker compose run --rm wagtail pip freeze > requirements-frozen.txt

# Очистить Python кеш
find . -type d -name __pycache__ -exec rm -r {} +
find . -type f -name '*.pyc' -delete
```

---

## ✅ Чеклист развертывания

- [ ] Установлен Docker и Docker Compose на Ubuntu
- [ ] Пользователь добавлен в группу docker
- [ ] Создан файл `.env` из `.env.example`
- [ ] Сгенерирован и установлен `SECRET_KEY`
- [ ] Настроены переменные базы данных
- [ ] Выполнена команда `docker compose up --build`
- [ ] Контейнеры запущены (проверка `docker compose ps`)
- [ ] База данных доступна
- [ ] Тесты проходят успешно
- [ ] Логи не содержат критических ошибок

---

## 🔥 Полезные алиасы для .bashrc

Добавьте в `~/.bashrc` для удобства:

```bash
# Docker aliases
alias dc='docker compose'
alias dcup='docker compose up'
alias dcdown='docker compose down'
alias dcps='docker compose ps'
alias dclogs='docker compose logs -f'
alias dcbuild='docker compose build --no-cache'
alias dcrestart='docker compose restart'

# Wagtail specific
alias wagtail-shell='docker compose exec wagtail bash'
alias wagtail-test='docker compose run --rm wagtail python runtests.py'
alias wagtail-migrate='docker compose run --rm wagtail python manage.py migrate'
```

Затем выполните:
```bash
source ~/.bashrc
```

---

**Готово!** Wagtail развернут и работает в Docker на Ubuntu 🎉
