# 38. TypeScript: DOM, API и React

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
  return "id" in value && "name" in value;
}
```

`response.json()` приходит с внешней границы. Сначала считайте значение `unknown`, затем проверьте вручную или библиотекой схем. Assertion `as User` проверки не выполняет.

## Props, state и события React

```tsx
type ProductCardProps = {
  product: Product;
  onAdd: (id: Product["id"]) => void;
  children?: React.ReactNode;
};

function ProductCard({ product, onAdd }: ProductCardProps) {
  const handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    console.log(event.currentTarget.value);
  };

  return <button onClick={() => onAdd(product.id)}>В корзину</button>;
}
```

Обычная функция часто яснее `React.FC`. Для состояния запроса используйте discriminated union, чтобы невозможные комбинации loading/error/data не компилировались.

## Подключение к существующему проекту

Переводите JavaScript постепенно:

1. включите `allowJs` и проверки;
2. типизируйте границы и часто используемые модели;
3. переименовывайте небольшие модули в `.ts` и `.tsx`;
4. уменьшайте `any`, не заменяя его массовыми assertion;
5. держите строгий режим целью и включайте проверки поэтапно.

Типы внешней библиотеки могут поставляться самим пакетом или через `@types/*`. ESLint проверяет соглашения и потенциальные ошибки, TypeScript отвечает за типовые контракты, Jest выполняет код тестов. Эти инструменты дополняют друг друга.

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
