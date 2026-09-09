# TypeScript: система типов

<p class="reading-time">Чтение: 15 минут</p>

TypeScript проверяет типы до запуска и компилируется в JavaScript. Типы помогают выразить контракт, но не проверяют данные, пришедшие во время выполнения.

## Базовые типы

```ts
type ProductId = string;

interface Product {
  id: ProductId;
  title: string;
  price: number;
  tags?: string[];
}

function total(products: Product[]): number {
  return products.reduce((sum, product) => sum + product.price, 0);
}
```

`type` удобен для объединений и вычисляемых типов. `interface` хорошо описывает расширяемую форму объекта. В большинстве прикладных случаев выбор определяется соглашением команды.

## Union и сужение

```ts
type RequestState<T> =
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; message: string };

function render(state: RequestState<Product[]>) {
  if (state.status === "success") {
    return state.data.length;
  }
}
```

Дискриминированное объединение не позволяет прочитать `data` в состоянии ошибки.

## `unknown`, `any`, `never`

- `any` отключает проверку и распространяет небезопасность;
- `unknown` требует сначала проверить значение;
- `never` означает невозможное значение или функцию без нормального завершения;
- `void` означает, что возвращаемое значение не используется.

## Generics

```ts
function first<T>(items: T[]): T | undefined {
  return items[0];
}

async function getJson<T>(url: string): Promise<T> {
  const response = await fetch(url);
  if (!response.ok) throw new Error(String(response.status));
  return response.json();
}
```

Generic связывает типы между входом и выходом. Он не заменяет конкретный тип, если связи нет.

## Utility types

- `Partial<T>` делает поля необязательными;
- `Required<T>` делает обязательными;
- `Pick<T, K>` выбирает поля;
- `Omit<T, K>` исключает поля;
- `Record<K, V>` описывает словарь;
- `ReturnType<F>` получает тип результата функции.

## React

```tsx
type ButtonProps = {
  children: React.ReactNode;
  onClick: () => void;
  disabled?: boolean;
};
```

Типизируйте props, события (`React.ChangeEvent<HTMLInputElement>`), ref и ответы API. Не используйте `React.FC` автоматически: обычная функция часто выражает контракт яснее.

## Проверка данных на границе

Утверждение `response.json() as Product[]` не делает JSON безопасным. Данные API нужно валидировать вручную или схемой, например Zod.

!!! warning "Частая ошибка"
    Type assertion `as` говорит компилятору довериться разработчику. Оно не преобразует значение и может скрыть ошибку.

## Мини-задача

Типизируйте состояния GitHub User Finder как union, добавьте тип ответа API и функцию проверки минимально нужных полей.

[Вопросы по TypeScript](interview/typescript-state.md){ .md-button }
