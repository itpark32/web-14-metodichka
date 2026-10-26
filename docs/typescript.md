# TypeScript: система типов

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
  | { status: "idle" }
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

async function getJson(url: string): Promise<unknown> {
  const response = await fetch(url);
  if (!response.ok) throw new Error(String(response.status));
  return response.json();
}
```

`first` связывает входной и выходной тип: для `first([1, 2])` результат имеет тип `number | undefined`. `getJson` намеренно возвращает `unknown`: generic `<T>` не мог бы проверить реальный ответ сети. Связь типов должна быть обоснована, а не обещана вызывающему коду.

## `tsconfig.json` и строгая проверка

TypeScript-проект настраивается через `tsconfig.json`: он определяет, какие файлы проверять и какие правила компилятора использовать. Минимальный пример настройки строгой проверки:

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

`strict: true` включает группу строгих проверок, в том числе проверки `null`/`undefined` и неявного `any`. Для нового проекта это хороший базовый режим: компилятор раньше показывает неоднозначные и небезопасные места.

Например, этот код намеренно содержит ошибку типов:

```ts
function printLength(value: string | null) {
  console.log(value.length); // Ошибка: value может быть null
}
```

На вход может прийти строка или `null`. В строгом режиме сначала нужно исключить `null`:

```ts
function printLength(value: string | null) {
  if (value === null) {
    return;
  }

  console.log(value.length);
}
```

Теперь для строки выводится её длина, а при `null` функция завершается без обращения к свойству. Проверка сужает тип до `string`.

```text
tsconfig.json → правила TypeScript-проекта
strict → более строгая проверка типов
ошибка компилятора → исправляем причину, а не скрываем её через any или as
```

Не обязательно запоминать все флаги. В существующем проекте сначала изучите текущие настройки: усиление проверок может открыть старые проблемы, поэтому включайте их осознанно. [Подробнее о strict](https://www.typescriptlang.org/tsconfig/strict.html).

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

!!! warning "Частые ошибки"
    - Type assertion `as` говорит компилятору довериться разработчику, но не преобразует значение и может скрыть ошибку.
    - Отключать `strict`, чтобы убрать ошибки типов, вместо исправления причины.
    - Массово заменять ошибки компилятора на `any` или `as`, создавая ложное ощущение безопасности.

## Сужение unknown и практические типы

```ts
function printError(error: unknown): void {
  if (error instanceof Error) console.error(error.message);
  else console.error("Неизвестная ошибка");
}

type ProductDraft = Omit<Product, "id">;
type ProductPatch = Partial<Pick<Product, "title" | "price">>;
const patch: ProductPatch = { price: 500 };
```

Ошибка приходит как неизвестное значение; `instanceof` позволяет безопасно прочитать сообщение. Utility types повторно используют модель: черновик еще не имеет id, частичное изменение допускает только выбранные поля. Это помогает во время разработки, но сервер все равно валидирует PATCH.

Дискриминант `status` в RequestState исключает сочетание «ошибка и успешные data одновременно». Сужение по `status === "success"` открывает data без assertion. `never` полезен для проверки исчерпанности вариантов в `switch`: при добавлении нового состояния компилятор покажет необработанную ветку.

## Как ответить на интервью

> TypeScript статически проверяет контракты и затем компилируется в JavaScript. Union описывает набор допустимых вариантов, narrowing уточняет вариант проверкой, generic сохраняет связь между типами, а `unknown` требует проверки перед использованием. Внешние данные все равно валидируются во время выполнения.

## Мини-задача

Типизируйте состояния приложения поиска пользователей как union, добавьте тип ответа API и функцию проверки минимально нужных полей.

## Проверьте себя

1. Когда удобнее interface, а когда type?
2. Как narrowing открывает свойства union?
3. Почему unknown безопаснее any на внешней границе?
4. Как generic сохраняет связь входа и выхода?
5. Почему Promise<User> не доказывает форму JSON?
6. Что хранится в `tsconfig.json` и зачем обычно включают `strict`?

[Следующая тема: TypeScript в DOM, API и React](typescript-react.md){ .md-button }
