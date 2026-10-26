# React: API и серверное состояние

<p class="reading-time">Чтение: 10 минут</p>

Работа с API включает не только успешные данные, но также загрузку, пустой результат, ошибку, повтор, отмену и кеш.

!!! abstract "Фокус"
    - **Нужно знать:** эффект запроса, cleanup, состояния запроса и `response.ok`.
    - **Часто в работе:** React Query, query key, кеш, invalidation и mutation.
    - **Достаточно узнавать:** Axios, если Fetch закрывает требования проекта.

## Ручной запрос

```jsx
function Users() {
  const [state, setState] = useState({ status: "loading", data: [] });

  useEffect(() => {
    const controller = new AbortController();

    fetchUsers(controller.signal)
      .then((data) => setState({ status: "success", data }))
      .catch((error) => {
        if (error.name !== "AbortError") {
          setState({ status: "error", data: [] });
        }
      });

    return () => controller.abort();
  }, []);

  if (state.status === "loading") return <p>Загрузка...</p>;
  if (state.status === "error") return <p>Не удалось загрузить пользователей</p>;
  if (state.data.length === 0) return <p>Пользователей нет</p>;
  return <ul>{state.data.map(user => <li key={user.id}>{user.name}</li>)}</ul>;
}
```

Отдельные `isLoading`, `isError`, `isSuccess` могут образовать невозможные комбинации. Поле `status` моделирует взаимоисключающие состояния яснее.

В этом компоненте импортируйте `useState` и `useEffect` из React. `fetchUsers(signal)` — API-функция, которая проверяет HTTP-статус и возвращает массив `{ id, name }`. На входе нет props, запрос запускается после добавления компонента в дерево. Cleanup отменяет Fetch, чтобы старый результат не применился после ухода со страницы. Когда появляются фильтры и повторная загрузка, ручное управление кешем становится отдельной задачей.

## TanStack Query

Библиотека хранит кеш по `queryKey`, управляет временем устаревания, повторами, отменой и повторной загрузкой.

Установите `npm install @tanstack/react-query`. Создайте один клиент и подключите Provider у корня:

```jsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient();
// root — результат createRoot; App — ваше приложение
root.render(<QueryClientProvider client={queryClient}><App /></QueryClientProvider>);
```

Клиент хранит кеш между renders. Создание нового клиента в теле компонента сбрасывало бы эту общую память. Добавим API-модуль:

```js
// productsApi.js
export async function getProducts({ signal }) {
  const response = await fetch("/api/products", { signal });
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json();
}

export async function createProduct(product) {
  const response = await fetch("/api/products", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(product),
  });
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json();
}
```

Контракт примера: GET возвращает массив `{ id, title }`, POST — созданный товар и статус `201` с JSON. Функции проверяют HTTP-статус, потому что Fetch сам не отклоняет Promise на 4xx/5xx. Runtime-проверку полей добавим в [главе о TypeScript и API](typescript-react.md).

```jsx
import { useQuery } from "@tanstack/react-query";
import { getProducts } from "./productsApi";

function Products() {
  const query = useQuery({
    queryKey: ["products"],
    queryFn: ({ signal }) => getProducts({ signal }),
    staleTime: 60_000,
  });
  if (query.isPending) return <p>Загрузка...</p>;
  if (query.isError) return <button onClick={() => query.refetch()}>Повторить</button>;
  if (query.data.length === 0) return <p>Товаров пока нет</p>;
  return <ul>{query.data.map(product => (
    <li key={product.id}>{product.title}</li>
  ))}</ul>;
}
```

`queryKey` определяет запись кеша, `queryFn` получает данные. Через 60 секунд они становятся stale: это не удаление, а разрешение перепроверить их при подходящем событии. Refetch может идти в фоне с показом старых данных; `isFetching` позволяет сообщить об обновлении. Для поиска включайте строку в ключ, например `["products", query]`, и используйте то же значение в queryFn. Иначе разные выборки разделят одну запись.

Mutation изменяет данные. После успеха можно инвалидировать запрос, обновить кеш из ответа или применить optimistic update с откатом.

## Границы ответственности

- компонент отвечает за представление;
- custom hook связывает UI с библиотекой данных;
- API-модуль строит запрос и нормализует ошибки;
- сервер остается источником истины для прав и бизнес-правил.

## Axios и Fetch

Axios автоматически преобразует JSON, отклоняет Promise для статусов вне `2xx` и имеет interceptors. Fetch встроен в платформу, требует проверки `response.ok` и явного чтения тела. Выбор зависит от требований, а не от привычки.

!!! warning "Частая ошибка"
    Не помещайте объект фильтров, создаваемый заново на каждом рендере, в зависимости эффекта без необходимости. Это может запустить бесконечную цепочку запросов.

## Mutation и invalidation

```jsx
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { createProduct } from "./productsApi";

function CreateProductButton() {
  const client = useQueryClient();
  const mutation = useMutation({
    mutationFn: createProduct,
    onSuccess: () => client.invalidateQueries({ queryKey: ["products"] }),
  });
  return <>
    <button disabled={mutation.isPending}
      onClick={() => mutation.mutate({ title: "Клавиатура" })}>Создать</button>
    {mutation.isError && <p role="alert">Не удалось создать товар</p>}
  </>;
}
```

Mutation отправляет изменение, invalidation помечает список устаревшим и запускает повторную загрузку активного запроса. Возвращенный Promise invalidation позволяет дождаться обновления списка. Ошибка создания остается рядом с кнопкой. Не инвалидируйте все данные приложения ради одного списка.

Optimistic update заранее меняет кеш до ответа сервера. Он ускоряет отклик, но требует отката при ошибке и учета параллельных изменений; для начала надежнее обновить список после успеха.

## RTK Query: тот же жизненный цикл в Redux

Если Redux Toolkit уже используется, RTK Query интегрирует кеш с его store. TanStack Query работает с собственным клиентом. Обычно выбирают один подход для одного набора данных, чтобы не поддерживать два независимых кеша.

```js
// api.js
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react";

export const api = createApi({
  reducerPath: "api",
  baseQuery: fetchBaseQuery({ baseUrl: "/api" }),
  tagTypes: ["Product"],
  endpoints: build => ({
    products: build.query({
      query: () => "/products",
      providesTags: ["Product"],
    }),
    createProduct: build.mutation({
      query: body => ({ url: "/products", method: "POST", body }),
      invalidatesTags: ["Product"],
    }),
  }),
});
export const { useProductsQuery, useCreateProductMutation } = api;
```

`build.query` читает данные, `build.mutation` изменяет их. Общий tag связывает изменение с кешем списка. Это минимальная схема; более точные tags могут отличать отдельные товары.

```js
// store.js: подключение API к Redux
import { configureStore } from "@reduxjs/toolkit";
import { api } from "./api";

export const store = configureStore({
  reducer: { [api.reducerPath]: api.reducer },
  middleware: getDefault => getDefault().concat(api.middleware),
});
```

Этот store передайте React Redux Provider, как в [предыдущей главе](state-management.md). В приложении с корзиной добавьте также `cart: cartReducer`, а не создавайте второй независимый store.

```jsx
import { useProductsQuery, useCreateProductMutation } from "./api";

function RtkProducts() {
  const { data = [], isLoading, isError, refetch } = useProductsQuery();
  const [create, mutation] = useCreateProductMutation();
  if (isLoading) return <p>Загрузка...</p>;
  if (isError) return <button onClick={refetch}>Повторить</button>;
  return <>
    {data.length ? data.map(p => <p key={p.id}>{p.title}</p>) : <p>Товаров нет</p>}
    <button disabled={mutation.isLoading}
      onClick={() => create({ title: "Клавиатура" })}>Создать</button>
    {mutation.isError && <p role="alert">Не удалось создать товар</p>}
  </>;
}
```

Хуки генерируются из endpoints. После успешной mutation tag инвалидируется и активный список обновляется. Без reducer и middleware код запросов не будет работать как настроенное хранилище кеша.

## Как ответить на интервью

> Серверные данные имеют собственный жизненный цикл и кеш. В простом случае запрос запускает эффект с отменой и явными состояниями loading, success, empty и error. React Query добавляет кеширование, повторные запросы, invalidation и mutations, но не заменяет правильный API-контракт.

## Мини-задача

Добавьте в приложение поиска пользователей кеш запросов, debounce поиска, отмену старого запроса и понятное сообщение для `403 rate limit`.

## Проверьте себя

1. Чем server state отличается от открытого модального окна?
2. Почему фильтр должен входить в queryKey?
3. Чем stale отличается от удаленного кеша?
4. Как mutation связана с invalidation?
5. Какие части RTK Query нужно подключить к store?

[Документация TanStack Query](https://tanstack.com/query/latest/docs/framework/react/quick-start) · [RTK Query](https://redux-toolkit.js.org/rtk-query/overview)

[Перейти к TypeScript](typescript.md){ .md-button .md-button--primary }
