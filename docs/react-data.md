# 36. React: API и серверное состояние

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

  // render loading, empty, error or data
}
```

Отдельные `isLoading`, `isError`, `isSuccess` могут образовать невозможные комбинации. Поле `status` моделирует взаимоисключающие состояния яснее.

## TanStack Query

Библиотека хранит кеш по `queryKey`, управляет временем устаревания, повторами, отменой и повторной загрузкой.

```jsx
const query = useQuery({
  queryKey: ["products", filters],
  queryFn: ({ signal }) => getProducts(filters, signal),
  staleTime: 60_000,
});
```

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

## Как ответить на интервью

> Серверные данные имеют собственный жизненный цикл и кеш. В простом случае запрос запускает эффект с отменой и явными состояниями loading, success, empty и error. React Query добавляет кеширование, повторные запросы, invalidation и mutations, но не заменяет правильный API-контракт.

## Мини-задача

Добавьте в GitHub User Finder кеш запросов, debounce поиска, отмену старого запроса и понятное сообщение для `403 rate limit`.

[Перейти к TypeScript](typescript.md){ .md-button .md-button--primary }
