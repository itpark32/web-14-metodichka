# 29. Асинхронность и Fetch API

<p class="reading-time">Чтение: 9 минут</p>

Асинхронная операция завершается позже: таймер, сетевой запрос или чтение файла не должны блокировать интерфейс. JavaScript продолжает выполнять синхронный код, а продолжение задачи попадает в очередь.

!!! abstract "Фокус"
    - **Нужно знать:** очередь выполнения, Promise, `async`/`await`, `try`/`catch`, `response.ok`.
    - **Часто в работе:** параллельный `Promise.all`, состояния загрузки и ошибки.
    - **Достаточно узнавать:** различия всех комбинаторов Promise и детали очередей задач.

## Таймер и Event Loop

```js
console.log(1);

setTimeout(() => console.log(2), 0);

console.log(3);
// 1, 3, 2
```

Нулевая задержка не означает немедленный запуск. Callback выполнится после освобождения стека вызовов.

## Promise

Promise находится в состоянии pending, fulfilled или rejected. После завершения состояние уже не меняется.

```js
loadProducts()
  .then(products => renderProducts(products))
  .catch(error => showError(error.message))
  .finally(() => hideLoader());
```

Каждый `then` возвращает новый Promise. Возвращайте из него асинхронный результат, чтобы сохранить цепочку.

## `async` и `await`

```js
async function loadProducts() {
  try {
    const response = await fetch("https://example.com/api/products");

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }

    return await response.json();
  } catch (error) {
    console.error("Не удалось загрузить товары", error);
    throw error;
  }
}
```

`async` всегда возвращает Promise. `await` приостанавливает только текущую async-функцию. Fetch отклоняется при сетевой ошибке, но ответ HTTP 404 или 500 нужно проверять через `response.ok`.

## Состояния интерфейса

Сетевой экран обязан иметь четыре состояния:

1. загрузка;
2. успешные данные;
3. пустой результат;
4. ошибка с возможностью повторить запрос.

!!! warning "Ошибка из занятия"
    `return setTimeout(...)` не превращает результат callback в ожидаемое значение. Оберните таймер в `new Promise((resolve, reject) => ...)`, затем вызывайте `resolve` внутри callback.

## Где это было

Урок 27 показывает callback, Promise, `async`/`await` и Event Loop. Урок 28 выполняет Fetch-запрос и разбирает JSON.

## Как ответить на интервью

> Асинхронный код не блокирует выполнение всего скрипта. Promise представляет будущий результат, async-функция всегда возвращает Promise, а `await` приостанавливает только эту функцию. У Fetch отдельно проверяю `response.ok`, потому что HTTP 404 и 500 не являются сетевой ошибкой.

## Мини-практика

Загрузите список задач JSONPlaceholder. Покажите загрузку, первые десять задач и понятную ошибку. Испытайте ошибку намеренно неверным доменом.

## Проверьте себя

1. Почему `setTimeout(fn, 0)` выполняется после синхронного кода?
2. Что возвращает async-функция?
3. Почему Fetch требует отдельной проверки `response.ok`?

[Следующая тема: HTTP, REST API и Network](http-rest.md)
