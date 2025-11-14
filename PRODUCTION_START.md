# 🚀 Запуск Wagtail Production на сервере

## Шаг 1: Настройка .env файла на сервере

```bash
# На сервере создайте/отредактируйте файл .env
cd ~/3web_wagtail
nano .env
```

### Обязательно измените:

1. **SECRET_KEY** - сгенерируйте новый:
```bash
python3 -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

2. **ALLOWED_HOSTS** - узнайте IP вашего сервера:
```bash
curl ifconfig.me
# Или
hostname -I
```
Затем в .env замените:
```env
ALLOWED_HOSTS=ВАШ_IP_АДРЕС
# Например: ALLOWED_HOSTS=116.203.123.45
```

3. **POSTGRES_PASSWORD** - придумайте сильный пароль:
```env
POSTGRES_PASSWORD=ВашСильныйПароль123!@#
```

## Шаг 2: Проверка конфигурации

Убедитесь что в .env:
- `DEBUG=False` ✅
- `SECRET_KEY` изменен ✅
- `ALLOWED_HOSTS` содержит IP/домен сервера ✅
- `POSTGRES_PASSWORD` надежный ✅

## Шаг 3: Очистка старых контейнеров

```bash
cd ~/3web_wagtail

# Остановить и удалить все контейнеры
docker-compose down -v

# Удалить старые образы (опционально)
docker system prune -af
```

## Шаг 4: Запустить Production

```bash
# Собрать образ
docker-compose -f docker-compose.prod.yml build --no-cache

# Запустить
docker-compose -f docker-compose.prod.yml up -d

# Проверить статус
docker-compose -f docker-compose.prod.yml ps

# Посмотреть логи
docker-compose -f docker-compose.prod.yml logs -f web
```

## Шаг 5: Проверка работы

После запуска откройте в браузере:
```
http://ВАШ_IP:8000
```

**Первый вход в админку:**
- URL: `http://ВАШ_IP:8000/admin/`
- Логин: `admin`
- Пароль: `changeme123`

⚠️ **ОБЯЗАТЕЛЬНО смените пароль после первого входа!**

## Команды управления

```bash
# Остановить
docker-compose -f docker-compose.prod.yml down

# Перезапустить
docker-compose -f docker-compose.prod.yml restart

# Посмотреть логи конкретного сервиса
docker-compose -f docker-compose.prod.yml logs -f web

# Зайти в контейнер
docker-compose -f docker-compose.prod.yml exec web bash

# Создать нового суперпользователя
docker-compose -f docker-compose.prod.yml exec web python manage.py createsuperuser
```

## 🔧 Устранение проблем

### Контейнер web не запускается

```bash
# Посмотреть логи
docker-compose -f docker-compose.prod.yml logs web

# Пересобрать образ
docker-compose -f docker-compose.prod.yml build --no-cache web
docker-compose -f docker-compose.prod.yml up -d
```

### База данных не подключается

Проверьте что пароли в .env совпадают:
- `POSTGRES_PASSWORD`
- В `DATABASE_URL` должен быть тот же пароль

### Сайт недоступен извне

Проверьте firewall:
```bash
# Ubuntu/Debian
sudo ufw allow 8000/tcp
sudo ufw status

# Или используйте iptables
sudo iptables -A INPUT -p tcp --dport 8000 -j ACCEPT
```

## 📝 Важные заметки

- По умолчанию Wagtail будет доступен на порту **8000**
- Для HTTPS нужен Nginx + Let's Encrypt (отдельная настройка)
- Регулярно делайте backup базы данных PostgreSQL
- Логи находятся в `docker-compose -f docker-compose.prod.yml logs`
