# 14. События и формы

<p class="reading-time">Чтение: 8 минут</p>

Событие сообщает о действии пользователя или браузера. Обработчик получает объект события и решает, что делать дальше.

## Обработчик

```js
const button = document.querySelector("button");

function handleClick(event) {
  console.log(event.currentTarget);
}

button.addEventListener("click", handleClick);
```

`target` показывает исходный элемент события, `currentTarget` элемент с текущим обработчиком. Чтобы удалить обработчик, передайте в `removeEventListener` ту же функцию.

## Всплытие и делегирование

Большинство событий всплывает от вложенного элемента к предкам. Это позволяет поставить один обработчик на контейнер:

```js
document.querySelector(".calculator").addEventListener("click", (event) => {
  const button = event.target.closest("button[data-action]");
  if (!button) return;

  calculate(button.dataset.action);
});
```

Делегирование полезно для множества однотипных элементов и элементов, добавленных позже.

## Форма

```js
const form = document.querySelector("form");

form.addEventListener("submit", (event) => {
  event.preventDefault();

  const data = new FormData(form);
  const age = Number(data.get("age"));

  if (!Number.isInteger(age) || age < 16) {
    showError("Введите целый возраст от 16 лет");
    return;
  }

  showSuccess("Форма заполнена корректно");
});
```

Обрабатывайте `submit`, а не только клик по кнопке: форму можно отправить клавишей Enter. `preventDefault()` отменяет стандартную отправку, но не останавливает всплытие.

## Где это было

Калькулятор урока 22 использует создание и удаление ошибки, валидацию, `switch` и делегирование. Игра урока 23 обрабатывает `DOMContentLoaded`, отправку формы, состояние победы и перезапуск.

## Мини-практика

Добавьте калькулятору вывод ошибки рядом с конкретным полем, `aria-invalid` и перевод фокуса на первое неверное поле.

## Проверьте себя

1. Чем `target` отличается от `currentTarget`?
2. Зачем нужно делегирование?
3. Почему форму лучше слушать по событию `submit`?

[Следующая тема: асинхронность и Fetch API →](async-api.md)

