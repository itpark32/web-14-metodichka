# TypeScript: DOM, API и React

<p class="reading-time">Чтение: 10 минут</p>

Прикладная типизация связывает систему типов с реальными границами: DOM-событием, JSON-ответом и props React-компонента.

!!! abstract "Фокус"
    - **Нужно знать:** DOM-элементы, события, props, state и ответы API.
    - **Часто в работе:** type guards, generics компонентов, gradual migration и типы библиотек.
    - **Достаточно узнавать:** ручное написание сложных declaration files.

## DOM и события

```ts
const form = document.querySelector<HTMLFormElement>("#search-form");
const input = document.querySelector<HTMLInputElement>("#query");

form?.addEventListener("submit", (event: SubmitEvent) => {
  event.preventDefault();
  console.log(input?.value ?? "");
});
```

Поиск может вернуть `null`, поэтому это отражается в коде. Generic у `querySelector` сообщает ожидаемый элемент, но не доказывает, что селектор действительно указывает на него.

## API и проверка данных

```ts
type User = { id: number; name: string };

function isUser(value: unknown): value is User {
  if (typeof value !== "object" || value === null) return false;
  return "id" in value && typeof value.id === "number"
    && "name" in value && typeof value.name === "string";
}
```

`response.json()` приходит с внешней границы. Сначала считайте значение `unknown`, затем проверьте вручную или библиотекой схем. Assertion `as User` проверки не выполняет.

Этот guard проверяет и наличие, и типы полей. Проверка только через `in` ошибочно приняла бы `{ id: "wrong", name: 42 }` за User. Пользовательский предикат тоже может быть написан неверно — TypeScript не доказывает его реализацию.

### Runtime validation через Zod

Установите `npm install zod`. Схема выражает проверяемый контракт:

```ts
import { z } from "zod";

const userSchema = z.object({ id: z.number(), name: z.string() });
type UserDto = z.infer<typeof userSchema>;

async function loadUser(id: number): Promise<UserDto> {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  const data: unknown = await response.json();
  return userSchema.parse(data);
}
```

Вход — id пользователя; выход — Promise проверенного объекта. Неправильный JSON или несовпадение схемы отклонит операцию. `.parse` выбрасывает ZodError; обработайте ошибку в UI, не показывая внутренние детали ответа. Если исключение неудобно, `safeParse(data)` возвращает union с `success` и `data` либо `error`.

`response.json() as User` не валидирует реальные данные: assertion исчезает при компиляции. TypeScript проверяет код во время разработки, Zod или ручная проверка — фактические внешние данные во время выполнения. Проверка нужна на границе API, storage и ввода, а не при каждом чтении уже проверенного поля.

[Zod: parsing и safeParse](https://zod.dev/basics)

## Props, state и события React

```tsx
import { useState, type ChangeEvent } from "react";

type ProductCardProps = {
  title: string;
  price: number;
  onAdd: () => void;
};
function ProductCard({ title, price, onAdd }: ProductCardProps) {
  return <article><h2>{title}</h2><p>{price} ₽</p>
    <button onClick={onAdd}>Добавить</button>
  </article>;
}
function Search() {
  const [query, setQuery] = useState<string>("");
  function handleChange(event: ChangeEvent<HTMLInputElement>) {
    setQuery(event.currentTarget.value);
  }
  return <label>Поиск <input value={query} onChange={handleChange} /></label>;
}
```

Props требуют строку, число и callback; передача числа в title даст ошибку компиляции. В Search событие связано с HTMLInputElement, поэтому value известен как string. Для nullable-состояния задайте union явно: `useState<UserDto | null>(null)`.

Обычная функция часто яснее `React.FC`. Для состояния запроса используйте discriminated union, чтобы невозможные комбинации loading/error/data не компилировались.

## Подключение к существующему проекту

Переводите JavaScript постепенно:

1. включите `allowJs` и проверки;
2. типизируйте границы и часто используемые модели;
3. переименовывайте небольшие модули в `.ts` и `.tsx`;
4. уменьшайте `any`, не заменяя его массовыми assertion;
5. держите строгий режим целью и включайте проверки поэтапно.

Типы внешней библиотеки могут поставляться самим пакетом или через `@types/*`. ESLint проверяет соглашения и потенциальные ошибки, TypeScript отвечает за типовые контракты, Vitest/Jest выполняют код тестов. Эти инструменты дополняют друг друга.

!!! warning "Ловушка: тип вместо проверки"
    Тип существует только до запуска. Сервер, Local Storage и пользовательский ввод остаются недоверенными значениями во время выполнения.

## Как ответить на интервью

> В браузере TypeScript помогает типизировать DOM-элементы и события, в React props, state, ref и callbacks. Ответ API сначала является внешним неизвестным значением, поэтому тип не назначается слепо через `as`, а подтверждается runtime-проверкой. Миграцию JavaScript-проекта выполняю по границам и модулям.

## Мини-практика

Переведите поиск пользователя на TypeScript: типизируйте форму, состояние запроса, минимальный ответ API и React-компонент результата. Добавьте проверку поврежденного ответа.

## Проверьте себя

1. Почему `querySelector` возвращает nullable-значение?
2. Чем тип события DOM отличается от React SyntheticEvent?
3. Проверяет ли `as User` JSON?
4. Как типизировать callback prop?
5. С чего начать постепенную миграцию?

[Следующая тема: тестирование](testing.md)
