# 28. BOM, localStorage и sessionStorage

<p class="reading-time">Чтение: 10 минут</p>

BOM объединяет браузерные объекты вокруг документа. Через `window` код получает адрес страницы, историю, данные окружения, размеры окна и веб-хранилища.

!!! abstract "Фокус"
    - **Нужно знать:** `window`, `location`, `history`, `localStorage`, JSON.
    - **Часто в работе:** `navigator`, `matchMedia`, событие `storage`.
    - **Достаточно узнавать:** размеры экранов, ограничения `window.close` и редкие поля navigator.

## Window и BOM

В браузерном скрипте `window` является глобальным объектом. DOM доступен через `window.document`, но в коде обычно пишут просто `document`.

```js
console.log(window.innerWidth);
console.log(location.href);
console.log(navigator.language);
```

Полезные части:

- `location` читает и меняет текущий URL;
- `history` перемещается по истории текущей вкладки;
- `navigator` сообщает возможности и настройки среды;
- `matchMedia` проверяет медиавыражение из JavaScript;
- `setTimeout` и `setInterval` управляют таймерами.

```js
const mobile = window.matchMedia("(max-width: 48rem)");
console.log(mobile.matches);
```

Не определяйте устройство по строке `userAgent`, если задача решается размером или проверкой возможности.

## localStorage

```js
localStorage.setItem("theme", "dark");
const theme = localStorage.getItem("theme");
localStorage.removeItem("theme");
```

Данные сохраняются между перезапусками вкладки и браузера для того же origin. Ключ и значение являются строками.

```js
const settings = { theme: "dark", compact: true };
localStorage.setItem("settings", JSON.stringify(settings));

const saved = localStorage.getItem("settings");
const restored = saved ? JSON.parse(saved) : null;
```

Разбор внешних или старых данных лучше защищать `try/catch`.

## sessionStorage

API совпадает с `localStorage`, но данные связаны с конкретной вкладкой и очищаются после завершения ее сессии.

| Хранилище | Срок | Область |
|---|---|---|
| `localStorage` | Пока не удалено | Origin |
| `sessionStorage` | Сессия вкладки | Origin и вкладка |

Оба хранилища синхронные и подходят небольшим настройкам, черновикам и состоянию интерфейса. Это не база данных для больших объемов.

## Чего не хранить

Не помещайте в Web Storage пароль, секретный ключ или другую чувствительную информацию. JavaScript страницы может прочитать эти значения, поэтому XSS становится особенно опасен.

!!! warning "Ловушка: объект превращается в object Object"
    `localStorage.setItem("user", user)` преобразует объект в строку `"[object Object]"`. Сериализуйте через `JSON.stringify`, а после чтения вызывайте `JSON.parse`.

## Событие storage

```js
window.addEventListener("storage", (event) => {
  if (event.key === "theme") {
    applyTheme(event.newValue);
  }
});
```

Событие помогает синхронизировать вкладки. Оно обычно приходит в другие документы того же origin, а не в ту вкладку, которая сделала запись.

## Как ответить на интервью

> BOM предоставляет браузерные объекты вокруг документа: `window`, `location`, `history`, `navigator`. `localStorage` хранит строки между сессиями, `sessionStorage` живет в сессии вкладки. Объекты сериализую в JSON и не храню там секреты.

## Мини-практика

Сохраните выбранную тему и черновик формы. Тема должна восстановиться после закрытия браузера, а временный шаг формы можно хранить только в текущей вкладке. Обработайте поврежденный JSON.

## Проверьте себя

1. Чем BOM отличается от DOM?
2. Что хранит `location`?
3. Какой тип возвращает `getItem`?
4. Чем localStorage отличается от sessionStorage?
5. Почему нельзя хранить секреты в Web Storage?

[Следующая тема: асинхронность и Fetch API](async-api.md)
