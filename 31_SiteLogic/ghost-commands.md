# Команды скрипта ghost_publish.py

- **Папка контента:** `/home/vitaly/31_NewSite` — в конфиге (`.env`) и в скриптах указывать `CONTENT_PATH=/home/vitaly/31_NewSite`. Файлы для публикации берутся отсюда; загруженные с Ghost страницы сохраняются сюда же.
- **Запуск:** из папки со скриптами (например `/home/vitaly/Cursor/31_SiteLogic/`) или с указанием полного пути к скрипту. Путь к .md — полный (`/home/vitaly/31_NewSite/файл.md`) или относительно CONTENT_PATH, если скрипт это поддерживает.

| Что делает | Команда |
|-----------|---------|
| Создать **пост в блоге** (черновик, не виден на сайте) | `python3 scripts/ghost_publish.py /home/vitaly/31_NewSite/файл.md` |
| Создать **пост в блоге** и сразу опубликовать (виден всем) | `python3 scripts/ghost_publish.py /home/vitaly/31_NewSite/файл.md --publish` |
| Создать **статическую страницу** сайта (черновик) | `python3 scripts/ghost_publish.py /home/vitaly/31_NewSite/файл.md --type page` |
| Создать **статическую страницу** и сразу опубликовать | `python3 scripts/ghost_publish.py /home/vitaly/31_NewSite/файл.md --type page --publish` |
| Обновить существующий **пост в блоге** (по slug) | `python3 scripts/ghost_publish.py /home/vitaly/31_NewSite/файл.md --update` |
| Обновить существующую **статическую страницу** (по slug) | `python3 scripts/ghost_publish.py /home/vitaly/31_NewSite/файл.md --update --type page` |
| Показать список всех **постов блога** на сайте | `python3 scripts/ghost_publish.py --list` |
| Показать список всех **статических страниц** на сайте | `python3 scripts/ghost_publish.py --list --type page` |

---

*Дата актуализации: 10 февраля 2026, 15:00*
