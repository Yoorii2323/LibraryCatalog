# Как выложить сайт на PythonAnywhere через GitHub

Краткая инструкция для папки `LibraryCatalog`: заливаем код на GitHub, подтягиваем на [PythonAnywhere](https://www.pythonanywhere.com), открываем в браузере.

**Ваш сайт:** [https://yoori2323.pythonanywhere.com/](https://yoori2323.pythonanywhere.com/)  
**Логин PythonAnywhere:** `yoori2323`

Код уже подготовлен для хостинга: API вызывается через `/api`, сессии — через `FLASK_SECRET_KEY`, агрегированный поиск — через URL текущего сайта. Готовый WSGI — в файле `pythonanywhere_wsgi.py`.

---

## Что понадобится

- аккаунт на [GitHub](https://github.com);
- аккаунт на [PythonAnywhere](https://www.pythonanywhere.com) (бесплатного хватит для диплома);
- проект в репозитории — либо вся папка `LibraryCatalog`, либо репозиторий = её содержимое.

**Важно:** на компьютере админка может крутиться отдельно (`start_admin.bat`, порт 5001). На хостинге всё в одном приложении: сайт и админка по адресу `https://вашлогин.pythonanywhere.com`, админка — по пути `/admin`.

---

## Шаг 1. Залить проект на GitHub

В папке `LibraryCatalog` на своём ПК:

```bash
git init
git add .
git commit -m "Первый коммит"
git branch -M main
git remote add origin https://github.com/ВАШ_ЛОГИН/ИМЯ_РЕПО.git
git push -u origin main
```

Файл `backend/library.db` в Git не попадёт — он в `.gitignore`. База появится на сервере при первом запуске.

Чтобы обновить сайт позже: правите код → `git push` → на PythonAnywhere в консоли `git pull` → кнопка **Reload** (см. ниже).

---

## Шаг 2. Код (уже в репозитории)

- В `static/app.js`, `templates/register.html`, `profile.html`, `admin.html` используется `const API_BASE = '/api'` — запросы идут на тот же домен, что и страница.
- В `backend/app.py` агрегированный поиск (`/api/search/aggregate`) строит URL от текущего сайта; при необходимости задайте переменную `APP_BASE_URL` (шаг 5).
- `app.secret_key` читается из `FLASK_SECRET_KEY` (шаг 5).

Локально по-прежнему работает `start.bat` → http://localhost:5000.

---

## Шаг 3. Настроить PythonAnywhere

1. Зарегистрируйтесь, откройте вкладку **Web** → **Add a new web app**.
2. Выберите **Manual configuration** (не Django).
3. Python **3.10** (или 3.11).

Адрес сайта: **https://yoori2323.pythonanywhere.com/**

### Скачать код с GitHub

**Consoles** → **Bash**:

```bash
cd ~
git clone https://github.com/ВАШ_ЛОГИН/ИМЯ_РЕПО.git
```

Дальше везде вместо `PROJECT` подставьте путь к проекту, например:

- `/home/yoori2323/LibraryCatalog` — если репозиторий на GitHub называется `LibraryCatalog` и вы клонировали его так же;
- `/home/yoori2323/ИМЯ_РЕПО` — если в корне репозитория сразу `backend`, `templates` и т.д.

### Установить зависимости

```bash
cd ~/PROJECT
python3.10 -m venv venv
source venv/bin/activate
pip install -r backend/requirements.txt
```

На вкладке **Web** в поле **Virtual environment** укажите:

`/home/yoori2323/PROJECT/venv`

(замените `PROJECT` на имя папки после `git clone`)

---

## Шаг 4. Файл WSGI

На **Web** откройте ссылку на WSGI-файл (в блоке Code). Скопируйте содержимое файла **`pythonanywhere_wsgi.py`** из репозитория.

Если после `git clone` папка на сервере не `/home/yoori2323/LibraryCatalog`, измените в WSGI только строку `path_project`.

Сохраните. PythonAnywhere ждёт переменную именно `application`.

---

## Шаг 5. Переменные окружения (по желанию)

**Web** → **Environment variables**:

| Переменная | Значение (пример) | Зачем |
|------------|-------------------|--------|
| `FLASK_SECRET_KEY` | длинная случайная строка | сессии не сбрасываются после Reload |
| `APP_BASE_URL` | `https://yoori2323.pythonanywhere.com` | поиск «Все источники», если агрегация не находит API |
| `SKIP_MAILRU_RCPT` | `1` | если регистрация Mail.ru падает на проверке ящика |
| `SMTP_USER`, `SMTP_PASSWORD`, … | — | письма с кодом подтверждения почты |

Пароли в GitHub не кладите — только в Environment variables на PythonAnywhere.

---

## Шаг 6. База и администратор

В Bash (с включённым `venv`):

```bash
cd ~/PROJECT
source venv/bin/activate
cd backend
python -c "from app import init_db; init_db()"
cd ..
python create_admin.py
```

Логин и пароль админа смотрите в `create_admin.py` — при необходимости поменяйте **до** запуска скрипта.

Админка на хостинге: **https://yoori2323.pythonanywhere.com/admin**

---

## Шаг 7. Запустить

На вкладке **Web** нажмите зелёную кнопку **Reload**.

Проверьте:

- главная открывается, стили на месте;
- поиск книг;
- регистрация / вход;
- профиль и избранное;
- `/admin`.

Если что-то падает — **Web** → **Log files** → `error.log`.

---

## Как обновлять сайт

```bash
cd ~/PROJECT
git pull
source venv/bin/activate
pip install -r backend/requirements.txt   # только если меняли requirements.txt
```

Потом снова **Reload** на вкладке Web.

---

## Если что-то не работает

**В консоли браузера ошибки про API** — не заменили `localhost` на `/api` (шаг 2).

**ImportError: app** — в WSGI неверный путь к папке `backend`.

**Нет модуля flask** — не указали venv на вкладке Web или ставили пакеты не в тот venv.

**После Reload выкидывает из аккаунта** — задайте `FLASK_SECRET_KEY`.

**Регистрация Mail.ru** — поставьте `SKIP_MAILRU_RCPT=1` или настройте SMTP.

**Пустой поиск «Все источники»** — поправьте `aggregate_search` (шаг 2).

**Пропали пользователи** — `library.db` живёт только на сервере, в Git её нет. Удалили папку — снова `init_db()` и `create_admin.py`.

---

## Про бесплатный тариф

Один сайт на аккаунт, адрес вида `логин.pythonanywhere.com`, без своего домена. Для показа диплома обычно хватает. Поиск книг через Open Library с сервера, как правило, работает; с почтой на бесплатном тарифе бывают ограничения.

---

## Чеклист перед сдачей

- [ ] код на GitHub  
- [ ] клон на PythonAnywhere (`/home/yoori2323/...`), venv, `pip install`  
- [ ] WSGI из `pythonanywhere_wsgi.py`, путь `path_project` верный  
- [ ] venv указан на вкладке Web  
- [ ] `FLASK_SECRET_KEY` (и при необходимости `APP_BASE_URL`, `SKIP_MAILRU_RCPT`)  
- [ ] `init_db` и `create_admin.py`  
- [ ] **Reload**  
- [ ] https://yoori2323.pythonanywhere.com/ — главная, вход, поиск, `/admin`  

---

Локально по-прежнему: `start.bat` — сайт, `start_admin.bat` — админка на порту 5001 только у себя на компьютере.
