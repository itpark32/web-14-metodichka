# React: JSX и компоненты

<p class="reading-time">Чтение: 11 минут</p>

React описывает интерфейс как функцию от данных: `UI = f(state)`. При изменении props или state React снова вызывает компонент и согласует результат с DOM.

!!! abstract "Фокус"
    - **Нужно знать:** JSX, функциональный компонент, props, state и `key`.
    - **Часто в работе:** композиция, однонаправленный поток данных и условный рендеринг.
    - **Достаточно узнавать:** классовые компоненты и их жизненный цикл.

## Компонент

```jsx
function ProductCard({ product, onAdd }) {
  return (
    <article>
      <h2>{product.name}</h2>
      <p>{product.price} ₽</p>
      <button onClick={() => onAdd(product.id)}>В корзину</button>
    </article>
  );
}
```

Правила:

- имя компонента начинается с заглавной буквы;
- компонент остается чистым во время рендера;
- props читаются, но не изменяются;
- данные идут вниз, события поднимаются вверх;
- композиция компонентов обычно гибче наследования.

## JSX

JSX похож на HTML, но является синтаксисом JavaScript. Выражения помещаются в `{}`. Используются `className`, `htmlFor`, camelCase-события.

```jsx
const message = isReady ? <Result /> : <Loader />;

const list = products.map((product) => (
  <ProductCard key={product.id} product={product} onAdd={addToCart} />
));
```

## Зачем нужен `key`

`key` помогает React сопоставить старые и новые элементы списка. Он должен быть уникальным среди соседей и стабильным между рендерами. Индекс массива плох, если список сортируется, фильтруется или изменяется.

## Классовые и функциональные компоненты

Программа включает оба вида, но новый код обычно пишут функциональными компонентами с хуками. Классовые компоненты важно уметь читать: `state`, `setState`, `componentDidMount`, `componentDidUpdate`, `componentWillUnmount`.

!!! warning "Частая ошибка"
    Не вызывайте обработчик во время рендера: `onClick={save()}`. Передайте функцию: `onClick={save}` или `onClick={() => save(id)}`.

## От события к новому render

Добавим родителя к `ProductCard` из примера выше:

```jsx
import { useState } from "react";

function Catalog({ products }) {
  const [cart, setCart] = useState([]);
  function addToCart(productId) {
    setCart(current => [...current, productId]);
  }
  return (
    <>
      <p>Товаров в корзине: {cart.length}</p>
      {products.map(product => (
        <ProductCard key={product.id} product={product} onAdd={addToCart} />
      ))}
    </>
  );
}
```

Вход — массив `{ id, name, price }` с уникальными id. Сначала корзина пустая. Карточка получает данные и callback, при клике передает id родителю. Setter сохраняет новый массив, React снова вызывает `Catalog`, счетчик увеличивается. В примере повторный клик добавляет еще один id; учет количества товара — отдельное бизнес-правило.

```text
компонент + props + state → render → событие → изменение state → новый render
```

Render — вычисление JSX, а commit — применение изменений к DOM. Новый render не означает, что React заново заменяет весь документ. Не отправляйте запрос или изменение заказа прямо из тела компонента: render должен оставаться чистым.

## Композиция и условный UI

```jsx
function Panel({ title, children }) {
  return <section><h2>{title}</h2>{children}</section>;
}

const summary = cart.length === 0
  ? <p>Корзина пуста</p>
  : <p>Выбрано: {cart.length}</p>;
// Внутри Catalog: <Panel title="Корзина">{summary}</Panel>
```

`children` позволяет переиспользовать рамку панели с разным содержимым. Условие выбирает JSX по данным; не нужно вручную искать элементы в DOM и скрывать их. Переменная `cart` в этом фрагменте — состояние родителя из предыдущего примера.

## Что находится вокруг React

React отвечает прежде всего за UI. Рядом обычно находятся Vite или framework для запуска и сборки, router для URL, библиотека server state для API, TypeScript, тесты и инструменты performance. SSR формирует HTML на сервере при запросе, SSG — заранее при сборке. Серверные возможности frameworks изучают после компонентов, Hooks и API; Next.js не обязателен для базового маршрута.

React Compiler при подключении к сборке может автоматически мемоизировать часть вычислений и компонентов. Это не отменяет понимание props, state, referential equality и React Profiler. Сначала понятный код, затем измерение и оптимизация. [Обзор Compiler](https://react.dev/learn/react-compiler/introduction).

## Как ответить на интервью

> React строит интерфейс из компонентов. Компонент получает неизменяемые props и возвращает JSX, а изменение state запускает новый рендер и согласование дерева. Данные передаются вниз, события поднимаются вверх, стабильный `key` помогает сопоставлять элементы списка.

## Мини-задача

Разделите каталог с корзиной на компоненты `ProductList`, `ProductCard`, `Cart` и `OrderConfirmation`. Определите минимальные props для каждого.

## Проверьте себя

1. Как callback карточки меняет состояние родителя?
2. Почему изменение state приводит к render, но не к полной замене DOM?
3. Что передает `children`?
4. Как стабильный key помогает сохранить состояние строки?

[Состояние и хуки](react-hooks.md){ .md-button .md-button--primary }
