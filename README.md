# GiftRadar — бесплатный Telegram Mini App

Схема:

**GitHub Pages** → красивый Mini App frontend

**Google Apps Script** → бесплатный HTTPS API-прокси

**Tonnel** → live read-only данные о выставленных подарках

Покупка всегда происходит на внешней площадке. GiftRadar не принимает деньги и не хранит Telegram-коды, seed-фразы или чужие marketplace-сессии.

## Файлы

- `index.html` — весь интерфейс Mini App.
- `Code.gs` — backend для Google Apps Script.
- `README.md` — эта инструкция.

## ШАГ 1. Создать GitHub repository

1. Откройте GitHub.
2. Нажмите **New repository**.
3. Название: `GiftRadar`.
4. При GitHub Free репозиторий для Pages должен быть public.
5. Создайте repository.

## ШАГ 2. Загрузить frontend

Репозиторий должен выглядеть так:

```text
GiftRadar/
├── index.html
└── README.md
```

`index.html` должен лежать именно в корне.

## ШАГ 3. Включить GitHub Pages

В репозитории:

`Settings` → `Pages`

В разделе Build and deployment:

`Source` → `Deploy from a branch`

`Branch` → `main`

`Folder` → `/ (root)`

Нажмите `Save`.

Через некоторое время GitHub выдаст адрес:

`https://ВАШ_USERNAME.github.io/GiftRadar/`

GitHub Pages для `github.io` поддерживает HTTPS автоматически.

## ШАГ 4. Создать Google Apps Script

1. Откройте `https://script.google.com/`.
2. Нажмите **New project**.
3. Название проекта: `GiftRadar API`.
4. Удалите пример `myFunction`.
5. Откройте файл `Code.gs`.
6. Скопируйте весь код из этого проекта `Code.gs` и вставьте его в редактор Apps Script.
7. Нажмите `Save`.

## ШАГ 5. Первый тест Apps Script

В редакторе выберите функцию `getHealth_` и запустите её один раз, чтобы Google показал возможные разрешения.

Для публичного web app функция `doGet(e)` должна быть входной точкой.

## ШАГ 6. Опубликовать Apps Script как Web app

В Apps Script:

`Deploy` → `New deployment`

Тип:

`Web app`

Настройки:

`Execute as` → **Me**

`Who has access` → **Anyone**

Нажмите `Deploy`.

Google выдаст Web app URL вида:

`https://script.google.com/macros/s/XXXXXXXXXXXXXXXX/exec`

Используйте именно URL с `/exec`, а не `/dev`.

## ШАГ 7. Проверить backend

Откройте URL в браузере:

`https://script.google.com/macros/s/XXXXXXXXXXXXXXXX/exec?action=health`

Должен вернуться JSON примерно:

```json
{
  "ok": true,
  "app": "GiftRadar",
  "live_source": "Tonnel read-only"
}
```

Для подарков:

`https://script.google.com/macros/s/XXXXXXXXXXXXXXXX/exec?action=gifts`

## ШАГ 8. Связать GitHub Pages с Apps Script

Откройте `index.html` в GitHub.

Найдите строку:

```js
const API_URL = 'PASTE_GOOGLE_APPS_SCRIPT_EXEC_URL_HERE';
```

Замените содержимое кавычек на ваш `/exec` URL:

```js
const API_URL = 'https://script.google.com/macros/s/XXXXXXXXXXXXXXXX/exec';
```

Нажмите `Commit changes`.

После публикации GitHub Pages Mini App начнёт запрашивать подарки через Apps Script.

### Почему используется JSONP

GitHub Pages и Apps Script находятся на разных доменах. Поэтому frontend обращается к Apps Script как к JSONP-источнику. Функция `doGet` возвращает JavaScript-вызов только для безопасного имени callback.

## ШАГ 9. Проверить GiftRadar

Откройте:

`https://ВАШ_USERNAME.github.io/GiftRadar/`

Проверьте:

- появляются карточки;
- меняется `LIVE`;
- работают режимы Сделки / Номера / Дёшево / Новые;
- открывается подробная карточка;
- кнопка источника переводит на внешний рынок;
- избранное сохраняется.

Если карточки не появились, сначала отдельно откройте Apps Script URL:

`.../exec?action=gifts`

Если там `items: []` и есть `error`, проблема между Apps Script и внешним источником Tonnel, а не в Telegram.

## ШАГ 10. Подключить Telegram

Откройте `@BotFather`.

Создайте бота через `/newbot`, если его ещё нет.

В настройках бота назначьте Main Mini App и укажите URL GitHub Pages:

`https://ВАШ_USERNAME.github.io/GiftRadar/`

Также можно настроить Menu Button и указать тот же URL.

В Telegram пользователю ничего дополнительно устанавливать не нужно: Mini App открывается прямо в Telegram.

## Обновление кода

### Frontend

Меняете `index.html` на GitHub → Commit → GitHub Pages сам обновляет сайт.

### Backend

Меняете `Code.gs` в Apps Script → Save → Deploy → Manage deployments → Edit → создать/обновить версию. Публичное versioned deployment нужно использовать для пользователей.

## Кэш и лимиты

Backend кэширует готовую live-выборку на 90 секунд. Это уменьшает количество обращений к Tonnel при нескольких пользователях.

Google Apps Script имеет дневные квоты и лимиты. Для consumer-аккаунтов Google сейчас указывает, в частности, 20 000 URL Fetch вызовов в сутки и 6 минут максимального runtime одного запуска; квоты могут меняться.

## Безопасность

В GitHub не храните:

- Telegram Bot Token;
- Telegram authData пользователей;
- seed-фразы;
- cookies маркетплейсов;
- пароли и коды Telegram;
- marketplace access tokens.

Эта бесплатная версия не использует секретные токены вообще.

## Важно про реальные данные

Приложение не генерирует фиктивные цены. В карточки попадает только то, что удалось получить из live read-only выборки Tonnel. Если внешний источник недоступен или меняет API, GiftRadar показывает ошибку вместо выдуманных данных.

Ссылки на изображения при отсутствии готового `photo_url` строятся по NFT media URL Fragment для подарка и его номера. Это best-effort fallback; фактическая доступность изображения зависит от конкретного подарка.

## Что добавить дальше

Следующий этап можно сделать без изменения интерфейса:

- MRKT adapter;
- Portals adapter;
- история цен;
- floor по коллекции/модели;
- сигнал «дешевле рынка»;
- слежение за конкретным номером;
- Telegram-уведомления о новых листингах;
- персональные фильтры;
- платные Pro-функции.
