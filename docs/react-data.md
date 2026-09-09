# React: API и серверное состояние

<p class="reading-time">Чтение: 10 минут</p>

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

## Мини-задача

Добавьте в GitHub User Finder кеш запросов, debounce поиска, отмену старого запроса и понятное сообщение для `403 rate limit`.

[Перейти к TypeScript](typescript.md){ .md-button .md-button--primary }
