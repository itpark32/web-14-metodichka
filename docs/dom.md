# 13. DOM

<p class="reading-time">Чтение: 7 минут</p>

DOM представляет HTML-документ как дерево объектов. JavaScript находит узлы, читает и меняет их свойства, создает новые элементы и удаляет старые.

## Поиск

```js
const title = document.querySelector("h1");
const cards = document.querySelectorAll(".card");
const form = document.getElementById("order-form");
```

`querySelector` возвращает первый элемент или `null`. `querySelectorAll` возвращает статический `NodeList`, который можно перебрать через `forEach`.

## Текст, классы и атрибуты

```js
title.textContent = "Новый заголовок";
title.classList.add("title--active");
title.classList.toggle("is-hidden");

const input = document.querySelector("input");
input.setAttribute("aria-invalid", "true");
console.log(input.value);
```

Для обычного текста выбирайте `textContent`. `innerHTML` разбирает строку как разметку и становится источником XSS, если строка содержит непроверенные пользовательские данные.

Атрибут хранится в HTML, свойство отражает текущее состояние объекта. Например, `input.value` меняется во время ввода, а атрибут `value` может остаться исходным.

## Создание элемента

```js
const message = document.createElement("p");
message.className = "form__message";
message.textContent = "Данные сохранены";
form.append(message);
```

Используйте `append`, `prepend`, `before`, `after`, `replaceWith` и `remove`. Сначала создайте и настройте узел, затем вставьте его в документ.

## Где это было

В уроках 21-24 JavaScript меняет классы, стили, текст, атрибуты и размеры элементов, добавляет сообщения об ошибках и управляет слайдами.

## Мини-практика

По нажатию кнопки создайте элемент списка из безопасно прочитанного текста поля. После добавления очистите поле. Пустое значение не добавляйте.

## Проверьте себя

1. Что вернет `querySelector`, если элемент не найден?
2. Чем `textContent` безопаснее `innerHTML`?
3. Чем HTML-атрибут может отличаться от DOM-свойства?

[Следующая тема: события и формы →](events-forms.md)

