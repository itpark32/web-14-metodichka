# Context, Redux Toolkit и RTK Query

<p class="reading-time">Чтение: 13 минут</p>

## Как выбрать место состояния

1. Локальное состояние компонента.
2. Подъем к ближайшему общему родителю.
3. Context для редко меняющихся глобальных зависимостей.
4. Redux Toolkit для сложного клиентского состояния и предсказуемых переходов.
5. React Query или RTK Query для серверных данных и кеша.

## Context

Context передает значение глубоко без цепочки props. Он подходит для темы, локали, текущего пользователя или сервисов. Частое изменение большого объекта может перерисовать всех потребителей.

## Redux Toolkit

```js
const cartSlice = createSlice({
  name: "cart",
  initialState: [],
  reducers: {
    added(state, action) {
      state.push(action.payload);
    },
    removed(state, action) {
      return state.filter((item) => item.id !== action.payload);
    },
  },
});

const store = configureStore({
  reducer: { cart: cartSlice.reducer },
});
```

Внутри reducer разрешен «мутирующий» синтаксис, потому что Immer создает новое неизменяемое состояние.

Поток данных однонаправленный: UI вызывает `dispatch(action)`, reducer вычисляет state, подписанные компоненты получают новое значение.

## Серверное состояние

Список товаров с API имеет загрузку, ошибку, кеш, повторную проверку и устаревание. Не стоит вручную собирать все это в Redux slice, если подходит RTK Query или TanStack Query.

```js
const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: "/api" }),
  endpoints: (build) => ({
    products: build.query({ query: () => "/products" }),
  }),
});
```

## Селекторы

Компонент должен выбирать минимальный фрагмент state. Производные данные вычисляйте селектором. Мемоизированный селектор полезен для дорогих вычислений и стабильных ссылок.

!!! warning "Частая ошибка"
    Не переносите каждое поле формы в глобальное хранилище. Глобальность увеличивает связанность и стоимость изменений.

## Мини-задача

Спроектируйте состояние `Desserts`: отделите локальное состояние модального окна, клиентскую корзину и кеш товаров с сервера.

[API в React](react-data.md){ .md-button }
