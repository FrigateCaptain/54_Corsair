# Ghost CMS — настройка и интеграция с Cursor

## Общая информация

- **Домен сайта**: `clyuch.zhandarov.com`
- **CMS**: Ghost (self-hosted, бесплатная MIT-лицензия)
- **Хостинг**: VPS на masterhost (Ubuntu 24.04)
- **Интеграция**: Cursor -> Ghost Admin API -> сайт

## Структура проекта

- **Скрипты и конфиги** — в папке `31_SiteLogic/` (или аналог в вашем репозитории).
- **Папка контента** — `/home/vitaly/31_NewSite`. В конфиге (`.env`) и в скриптах **обязательно** указывать путь к контенту: `CONTENT_PATH=/home/vitaly/31_NewSite`. Скрипты читают оттуда .md для отправки на Ghost и туда же сохраняют загруженные страницы (pull).

```
31_SiteLogic/
├── ghost-settings.md      # Эта документация
├── .env                   # API-ключи + CONTENT_PATH (не коммитить!)
├── .env.example           # Шаблон .env
├── .gitignore
├── requirements.txt       # Python-зависимости
└── scripts/
    └── ghost_publish.py   # Скрипт публикации (читает/пишет в CONTENT_PATH)

/home/vitaly/31_NewSite/   # Папка контента (вне репозитория или в отдельном)
├── *.md                   # Markdown-файлы для публикации
├── posts/                 # опционально
└── pages/                 # опционально
```

---

## Этап 1: Подготовка VPS (Ubuntu 24.04)

### 1.1 Подключение к серверу

Найти данные SSH-доступа в панели управления masterhost.

```bash
ssh user@IP_СЕРВЕРА
```

Проверить версию ОС:
```bash
lsb_release -a
```

### 1.2 Создать пользователя (если доступ только root)

Ghost-CLI отказывается работать из-под root. Нужен обычный пользователь с sudo:

```bash
adduser ghostuser
usermod -aG sudo ghostuser
su - ghostuser
```

### 1.3 Обновить систему

```bash
sudo apt update && sudo apt upgrade -y
```

### 1.4 Установить Nginx

```bash
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

Проверить: `sudo systemctl status nginx`

### 1.5 Установить MySQL 8

```bash
sudo apt install mysql-server -y
sudo systemctl enable mysql
sudo systemctl start mysql
```

Создать БД и пользователя для Ghost:

```bash
sudo mysql -u root
```

```sql
CREATE DATABASE ghost_production CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
CREATE USER 'ghostuser'@'localhost' IDENTIFIED BY 'НАДЁЖНЫЙ_ПАРОЛЬ';
GRANT ALL PRIVILEGES ON ghost_production.* TO 'ghostuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

**Запомните пароль** — он понадобится при установке Ghost.

### 1.6 Установить Node.js 20 LTS

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install nodejs -y
```

Проверить:
```bash
node -v    # должно быть v20.x.x
npm -v
```

### 1.7 Установить Ghost-CLI

```bash
sudo npm install -g ghost-cli
```

---

## Этап 2: Установка Ghost

### 2.1 Создать папку и запустить установку

```bash
sudo mkdir -p /var/www/ghost
sudo chown ghostuser:ghostuser /var/www/ghost
cd /var/www/ghost
ghost install
```

### 2.2 Ответы на вопросы мастера установки

| Вопрос | Ответ |
|--------|-------|
| Blog URL | `https://clyuch.zhandarov.com` |
| MySQL hostname | `localhost` |
| MySQL username | `ghostuser` |
| MySQL password | (пароль из шага 1.5) |
| MySQL database name | `ghost_production` |
| Set up Nginx? | Yes |
| Set up SSL? | Yes (если DNS уже настроен) |
| Set up systemd? | Yes |
| Start Ghost? | Yes |

### 2.3 Проверка

```bash
ghost status
```

Ghost Admin будет доступен по адресу: `https://clyuch.zhandarov.com/ghost/`

---

## Этап 3: Привязка домена

### 3.1 Настроить DNS

В панели управления masterhost (или регистратора домена) создать A-запись:

```
clyuch.zhandarov.com → IP_VPS_СЕРВЕРА
```

Обновление DNS занимает от 5 минут до 24 часов.

Проверить:
```bash
dig clyuch.zhandarov.com +short
# Должен показать IP вашего VPS
```

### 3.2 SSL-сертификат

Если DNS был настроен до установки Ghost — SSL настроен автоматически.

Если нет — добавить позже:
```bash
cd /var/www/ghost
ghost setup ssl
```

---

## Этап 4: Интеграция Cursor с Ghost

### 4.1 Создать Custom Integration в Ghost

1. Открыть `https://clyuch.zhandarov.com/ghost/`
2. Перейти: **Settings → Integrations → Add custom integration**
3. Назвать: **Cursor Publisher**
4. Скопировать **Admin API Key** (формат `id:secret`)

### 4.2 Настроить .env

В файле `.env` в папке со скриптами (например `31_SiteLogic/.env`) заполнить:

```
GHOST_URL=https://clyuch.zhandarov.com
GHOST_ADMIN_API_KEY=скопированный_ключ
CONTENT_PATH=/home/vitaly/31_NewSite
```

### 4.3 Установить Python-зависимости

```bash
cd /home/vitaly/Cursor/31_SiteLogic
pip install -r requirements.txt
```

---

## Использование скрипта ghost_publish.py

Путь к файлу — относительно `CONTENT_PATH` (`/home/vitaly/31_NewSite`). Запуск из папки со скриптами (`31_SiteLogic`) или с указанием полного пути к скрипту.

### Создать черновик поста

```bash
cd /home/vitaly/Cursor/31_SiteLogic
python3 scripts/ghost_publish.py /home/vitaly/31_NewSite/example-post.md
# или если скрипт берёт путь относительно CONTENT_PATH:
python3 scripts/ghost_publish.py example-post.md
```

### Создать и сразу опубликовать пост

```bash
python3 scripts/ghost_publish.py /home/vitaly/31_NewSite/example-post.md --publish
```

### Создать страницу (page)

```bash
python3 scripts/ghost_publish.py /home/vitaly/31_NewSite/about.md --type page
```

### Обновить существующий пост (по slug)

```bash
python3 scripts/ghost_publish.py /home/vitaly/31_NewSite/example-post.md --update
```

### Посмотреть список постов на сайте

```bash
python3 scripts/ghost_publish.py --list
python3 scripts/ghost_publish.py --list --type page
```

### Формат markdown-файлов

Каждый файл в папке контента (`/home/vitaly/31_NewSite/`) поддерживает YAML front matter:

```markdown
---
title: Заголовок поста
slug: custom-slug
tags: [Тег1, Тег2]
featured: true
feature_image: https://example.com/image.jpg
custom_excerpt: Краткое описание
meta_title: SEO-заголовок
meta_description: SEO-описание
---

Содержимое поста в формате Markdown...
```

- `title` — заголовок (если не указан, берётся из имени файла)
- `slug` — URL-путь (если не указан, берётся из имени файла)
- `tags` — список тегов (создаются автоматически, если не существуют)
- Остальные поля — опциональные

---

## Рабочий процесс

```
1. Написать/отредактировать markdown в /home/vitaly/31_NewSite/
2. Запустить скрипт из папки со скриптами (31_SiteLogic)
3. Скрипт конвертирует markdown → HTML → Ghost Admin API
4. Контент появляется на сайте
```

---

## Полезные команды для VPS

| Команда | Описание |
|---------|----------|
| `ghost status` | Статус Ghost |
| `ghost start` | Запустить Ghost |
| `ghost stop` | Остановить Ghost |
| `ghost restart` | Перезапустить Ghost |
| `ghost update` | Обновить Ghost до последней версии |
| `ghost log` | Посмотреть логи |
| `ghost doctor` | Диагностика проблем |
| `ghost setup ssl` | Настроить SSL |
| `ghost config` | Посмотреть конфигурацию |

Все команды выполняются из `/var/www/ghost` на VPS.

---

## Обновление Ghost

```bash
cd /var/www/ghost
ghost update
```

Перед обновлением:
- Проверить [release notes](https://ghost.org/changelog/)
- Сделать бэкап: Ghost Admin → Settings → Labs → Export content

---

## Источники

- [Ghost Admin API Docs](https://ghost.org/docs/admin-api/)
- [Ghost Install Guide](https://ghost.org/docs/install/)
- [Ghost CLI Reference](https://ghost.org/docs/ghost-cli/)

---

*Дата актуализации: 8 февраля 2026, 19:23*
