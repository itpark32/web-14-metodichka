# 37. TypeScript: система типов

<p class="reading-time">Чтение: 12 минут</p>

TypeScript проверяет типы до запуска и компилируется в JavaScript. Типы помогают выразить контракт, но не проверяют данные, пришедшие во время выполнения.

!!! abstract "Фокус"
    - **Нужно знать:** аннотации, type, interface, union, narrowing и generics.
    - **Часто в работе:** utility types, типизация функций, DOM, API и React-компонентов.
    - **Достаточно узнавать:** namespaces, enum и сложные conditional types.

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

## Пересечения, кортежи и enum

```ts
type Entity = { id: string };
type Timestamped = { createdAt: string };
type StoredProduct = Entity & Timestamped & Product;

type Point = [x: number, y: number];
```

Intersection `A & B` требует выполнить оба контракта. Tuple фиксирует позиции и их типы. `enum` создает значение в JavaScript, поэтому для простого набора вариантов часто достаточно union строк: `"new" | "paid"`.

## Классы и модификаторы доступа

```ts
abstract class Repository<T> {
  abstract find(id: string): Promise<T | null>;
}

class ProductRepository extends Repository<Product> {
  constructor(private readonly apiUrl: string) {
    super();
  }

  async find(id: string): Promise<Product | null> {
    // запрос и проверка ответа
    return null;
  }
}
```

`public` доступен везде, `protected` внутри класса и наследников, `private` только внутри класса. Абстрактный класс нельзя создать напрямую, он задает общую основу и обязательные методы.

## Модули и conditional types

TypeScript использует стандартные `import` и `export`. Namespace встречается в старом или глобальном коде, но для современных приложений обычно выбирают ES Modules.

Conditional type выбирает тип по условию:

```ts
type ApiResult<T> = T extends Error
  ? { ok: false; error: T }
  : { ok: true; data: T };
```

Сложный вычисляемый тип полезен только тогда, когда упрощает использование API. Если его трудно объяснить, явный union часто лучше.

## Проверка данных на границе

Утверждение `response.json() as Product[]` не делает JSON безопасным. Данные API нужно валидировать вручную или схемой, например Zod.

!!! warning "Частая ошибка"
    Type assertion `as` говорит компилятору довериться разработчику. Оно не преобразует значение и может скрыть ошибку.

## Как ответить на интервью

> TypeScript статически проверяет контракты и затем компилируется в JavaScript. Union описывает набор допустимых вариантов, narrowing уточняет вариант проверкой, generic сохраняет связь между типами, а `unknown` требует проверки перед использованием. Внешние данные все равно валидируются во время выполнения.

## Мини-задача

Типизируйте состояния GitHub User Finder как union, добавьте тип ответа API и функцию проверки минимально нужных полей.

[Следующая тема: TypeScript в DOM, API и React](typescript-react.md){ .md-button }
