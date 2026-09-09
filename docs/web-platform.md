# Хранение данных, HTTP и REST

<p class="reading-time">Чтение: 12 минут</p>

> **Подтверждено проектами:** Posts App использует авторизацию, `Desserts` работает с JSON Server, а `GitHub User Finder` обращается к внешнему API.

## Хранение в браузере

| Механизм | Срок | Отправляется серверу | Для чего |
|---|---|---|---|
| `localStorage` | до удаления | нет | тема, простые настройки |
| `sessionStorage` | вкладка | нет | временный шаг формы |
| Cookie | по настройке | да | серверная сессия |
| IndexedDB | до удаления | нет | большой объем структурированных данных |

Web Storage хранит строки:

```js
localStorage.setItem("theme", "dark");
const theme = localStorage.getItem("theme") ?? "light";

localStorage.setItem("cart", JSON.stringify(cart));
const savedCart = JSON.parse(localStorage.getItem("cart") ?? "[]");
```

Не храните секреты в `localStorage`: JavaScript на странице может их прочитать.

## HTTP за минуту

Запрос содержит метод, URL, заголовки и иногда тело. Ответ содержит статус, заголовки и тело.

- `GET` получает ресурс;
- `POST` создает;
- `PUT` заменяет целиком;
- `PATCH` изменяет часть;
- `DELETE` удаляет.

Группы статусов: `2xx` успех, `3xx` перенаправление, `4xx` ошибка клиента, `5xx` ошибка сервера.

```js
const response = await fetch("/api/products");

if (!response.ok) {
  throw new Error(`HTTP ${response.status}`);
}

const products = await response.json();
```

`fetch` отклоняет Promise при сетевой ошибке, но не при статусе `404` или `500`. Поэтому проверка `response.ok` обязательна.

## REST

REST описывает ресурсы через URL и стандартные HTTP-операции:

```text
GET    /products
GET    /products/42
POST   /products
PATCH  /products/42
DELETE /products/42
```

Хороший API использует существительные в URL, корректные статусы, предсказуемый JSON и не хранит состояние клиента между запросами без явного механизма сессии.

## Состояния интерфейса

Любой сетевой экран проектируйте минимум в четырех состояниях:

1. загрузка;
2. успех с данными;
3. успех без данных;
4. ошибка с возможностью повтора.

## Отмена запроса

```js
const controller = new AbortController();

fetch(url, { signal: controller.signal });
controller.abort();
```

Это важно для живого поиска и размонтирования компонента.

## Мини-задача

Реализуйте поиск пользователя GitHub с задержкой, отменой предыдущего запроса, обработкой `404` и лимита API.

[Перейти к React](react-core.md){ .md-button .md-button--primary }
