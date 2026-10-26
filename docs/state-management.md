# Context и Redux Toolkit

<p class="reading-time">Чтение: 13 минут</p>

Состояние размещают как можно ближе к месту использования и поднимают только при появлении реальной общей ответственности.

!!! abstract "Фокус"
    - **Нужно знать:** local state, lifting state, Context, store, slice и selector.
    - **Часто в работе:** Redux Toolkit, normalized state, RTK Query и разделение client/server state.
    - **Достаточно узнавать:** ручной Redux без Redux Toolkit.

## Как выбрать место состояния

1. Локальное состояние компонента.
2. Подъем к ближайшему общему родителю.
3. Context для редко меняющихся глобальных зависимостей.
4. Redux Toolkit для сложного клиентского состояния и предсказуемых переходов.
5. React Query или RTK Query для серверных данных и кеша.

## Context

Context передает значение глубоко без цепочки props. Он подходит для темы, локали, текущего пользователя или сервисов. Частое изменение большого объекта может перерисовать всех потребителей.

```jsx
import { createContext, useContext } from "react";

const ThemeContext = createContext("light");
function Toolbar() {
  const theme = useContext(ThemeContext);
  return <p>Тема: {theme}</p>;
}
function App() {
  return <ThemeContext.Provider value="dark"><Toolbar /></ThemeContext.Provider>;
}
```

Ближайший Provider передает `dark`; без него использовалось бы `light`. Context доставляет значение, но сам не описывает переходы состояния: для изменения добавьте state у владельца. Если передать заново созданный большой объект, потребители получат обновление; разделяйте независимые контексты.

## Redux Toolkit

Установите `npm install @reduxjs/toolkit react-redux`. Файл `cartSlice.js`:

```js
import { createSlice } from "@reduxjs/toolkit";

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

export const { added, removed } = cartSlice.actions;
export default cartSlice.reducer;
```

Внутри reducer разрешен «мутирующий» синтаксис, потому что Immer создает новое неизменяемое состояние.

Поток данных однонаправленный: UI вызывает `dispatch(action)`, reducer вычисляет state, подписанные компоненты получают новое значение.

### Store и Provider

```js
// store.js
import { configureStore } from "@reduxjs/toolkit";
import cartReducer from "./cartSlice";

export const store = configureStore({
  reducer: { cart: cartReducer },
});
```

`configureStore` подключает reducer под ключом `cart`. Создайте store один раз за пределами компонентов. React-поддерево должно получить его через Provider:

```jsx
// В точке входа приложения
import { Provider } from "react-redux";
import { store } from "./store";

// root — результат createRoot контейнера приложения
root.render(<Provider store={store}><App /></Provider>);
```

### Dispatch и selector в UI

```jsx
import { useDispatch, useSelector } from "react-redux";
import { added, removed } from "./cartSlice";

function CartActions({ product }) {
  const items = useSelector(state => state.cart);
  const dispatch = useDispatch();
  return <>
    <p>Позиций: {items.length}</p>
    <button onClick={() => dispatch(added(product))}>Добавить</button>
    <button onClick={() => dispatch(removed(product.id))}>Удалить</button>
  </>;
}
```

На вход приходит товар с `id`. Клик создает action `{ type, payload }`, dispatch передает его в store, reducer возвращает новый state. `useSelector` подписывает компонент на корзину и запускает обновление при изменении выбранного значения. Здесь повторные добавления дают повторные позиции, а удаление убирает все совпадающие id; учет количества можно добавить отдельно.

```text
configureStore + slice reducer → Provider → UI dispatch(action)
                                               ↓
                              reducer → new state → useSelector → UI
```

Синтаксис `state.push` допустим внутри reducer `createSlice` благодаря Immer. Это не разрешение мутировать полученный из selector массив в компоненте. Не отправляйте запросы из reducer и не вызывайте dispatch прямо в render.

## Серверное состояние

Список товаров с API имеет загрузку, ошибку, кеш, повторную проверку и устаревание. Не стоит вручную собирать все это в Redux slice, если подходит RTK Query или TanStack Query.

Полная настройка запросов, Provider, mutations и invalidation находится в [главе о server state](react-data.md).

## Селекторы

Компонент должен выбирать минимальный фрагмент state. Производные данные вычисляйте селектором. Мемоизированный селектор полезен для дорогих вычислений и стабильных ссылок.

!!! warning "Частая ошибка"
    Не переносите каждое поле формы в глобальное хранилище. Глобальность увеличивает связанность и стоимость изменений.

## Как ответить на интервью

> Локальные данные остаются в компоненте, Context подходит редким глобальным зависимостям, Redux Toolkit сложным переходам клиентского состояния, а React Query или RTK Query серверному кешу. Я выбираю инструмент по владельцу данных, частоте изменений и нужным операциям, а не по размеру проекта.

## Мини-задача

Спроектируйте состояние каталога с корзиной: отделите локальное состояние модального окна, клиентскую корзину и кеш товаров с сервера.

## Проверьте себя

1. Почему поле формы обычно остается в local state?
2. Что делает Context и чего он не делает?
3. Как action доходит до reducer и компонента?
4. Где допустим «мутирующий» синтаксис Immer?
5. Почему серверные данные требуют отдельного кеша?

[API в React](react-data.md){ .md-button }
