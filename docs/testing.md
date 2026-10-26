# Тестирование веб-приложений

<p class="reading-time">Чтение и разбор примеров: 10–15 минут</p>

Тест фиксирует ожидаемое поведение. Хороший набор дает уверенность при изменениях, а не пытается проверить каждую строку. Для современного Vite-проекта начнем с Vitest, React Testing Library и MSW. Jest остается очень распространенным runner в существующих проектах; модель arrange → act → assert та же.

!!! abstract "Фокус"
    - **Нужно знать:** unit, component/integration, E2E и наблюдаемое поведение.
    - **Часто в работе:** Vitest/Jest, RTL, user events, async queries и MSW.
    - **Достаточно узнавать:** внутреннее устройство runner и редкие matcher-функции.

## Как выбрать уровень

| Проверка | Что подтверждает | Пример |
|---|---|---|
| Статическая | Корректность типов и соглашений без выполнения сценария | TypeScript, ESLint |
| Unit | Поведение небольшой единицы | Сумма корзины |
| Component | Видимое поведение компонента | Нажатие увеличивает счетчик |
| Integration | Взаимодействие нескольких частей | Компонент, API-клиент и подмененная сеть |
| E2E | Пользовательский путь в настоящем браузере | Вход и открытие профиля |

Component-тест может одновременно быть интеграционным. Чем больше системы охвачено, тем реалистичнее и дороже проверка. Начните с бизнес-правил, ошибок и критических сценариев.

## Подготовка Vitest в React/Vite

В существующем Vite-проекте установите:

```bash
npm install -D vitest jsdom @testing-library/react @testing-library/user-event @testing-library/jest-dom msw
```

В `vite.config.js` сохраните настройки приложения и добавьте test-конфигурацию. Для стандартного React-шаблона она выглядит так:

```js
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    setupFiles: ["./src/test/setup.js"],
  },
});
```

jsdom предоставляет DOM для компонентных тестов в Node.js. Это не полноценный браузер: реальную раскладку и навигацию проверяют отдельно. В `package.json` добавьте scripts `"test": "vitest"` и `"test:run": "vitest run"`. Первый удобен для разработки с повторным запуском, второй — для разовой проверки и CI.

## Unit: функция суммы

```js
// src/calculateTotal.js
export function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}
```

Контракт: цены в минимальных целых единицах, количества — неотрицательные целые. Функция принимает список и возвращает число, не изменяя вход. Проверим обычный и пустой список:

```js
// src/calculateTotal.test.js
import { describe, expect, it } from "vitest";
import { calculateTotal } from "./calculateTotal";

describe("calculateTotal", () => {
  it("считает сумму с количеством", () => {
    const items = [{ price: 100, quantity: 2 }, { price: 50, quantity: 1 }];
    const result = calculateTotal(items);
    expect(result).toBe(250);
  });
  it("возвращает ноль для пустого списка", () => {
    expect(calculateTotal([])).toBe(0);
  });
});
```

Подготовка items — arrange, вызов — act, expect — assert. Тест проверяет правило умножения на количество, а не повторяет алгоритм реализации. Для неподходящего ввода сначала определите контракт: отклонять его на границе или выбрасывать ошибку.

## Component: пользователь нажимает кнопку

```jsx
// src/Counter.jsx
import { useState } from "react";

export function Counter() {
  const [count, setCount] = useState(0);
  return <>
    <button onClick={() => setCount(current => current + 1)}>Увеличить</button>
    <output aria-label="Счетчик">{count}</output>
  </>;
}
```

Значение на экране — наблюдаемый результат. Тест не получает доступ к setter или внутреннему state:

```jsx
// src/Counter.test.jsx
import { expect, it } from "vitest";
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { Counter } from "./Counter";

it("увеличивает счетчик", async () => {
  const user = userEvent.setup();
  render(<Counter />);
  await user.click(screen.getByRole("button", { name: "Увеличить" }));
  expect(screen.getByLabelText("Счетчик")).toHaveTextContent("1");
});
```

Role и доступное имя описывают интерфейс с точки зрения пользователя. `await user.click` позволяет завершиться обновлениям. Для сетевого результата, появляющегося позже, используйте `findBy...` или `waitFor`, а не фиксированный таймер.

## API через mock network: MSW

Mock задает подмененное поведение, stub возвращает подготовленный результат, spy наблюдает вызовы. Мокайте внешнюю границу: сеть или время. HTTP удобно перехватывать MSW, сохраняя обычный Fetch внутри приложения.

```js
// src/productsApi.js
export async function loadProducts() {
  const response = await fetch("http://localhost/api/products");
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json();
}
```

Абсолютный локальный URL делает пример однозначным и в Node.js. В приложении адрес задает конфигурация среды. Опишем сетевой ответ:

```js
// src/test/server.js
import { http, HttpResponse } from "msw";
import { setupServer } from "msw/node";

export const server = setupServer(
  http.get("http://localhost/api/products", () =>
    HttpResponse.json([{ id: 1, title: "Keyboard" }])
  ),
);
```

Обработчик возвращает данные без внешнего сервера. Подключим общую подготовку и очистку:

```js
// src/test/setup.js
import "@testing-library/jest-dom/vitest";
import { afterAll, afterEach, beforeAll } from "vitest";
import { cleanup } from "@testing-library/react";
import { server } from "./server";

beforeAll(() => server.listen({ onUnhandledRequest: "error" }));
afterEach(() => {
  cleanup();
  server.resetHandlers();
});
afterAll(() => server.close());
```

`listen` включает перехват до тестов, reset убирает локальные изменения обработчиков, close восстанавливает сеть. Cleanup удаляет DOM предыдущего теста. Необработанный запрос становится ошибкой, чтобы тест случайно не обратился во внешнюю сеть.

```js
// src/productsApi.test.js
import { expect, it } from "vitest";
import { http, HttpResponse } from "msw";
import { server } from "./test/server";
import { loadProducts } from "./productsApi";

it("читает товары из JSON", async () => {
  await expect(loadProducts()).resolves.toEqual([{ id: 1, title: "Keyboard" }]);
});
it("не принимает HTTP 500 за успех", async () => {
  server.use(http.get("http://localhost/api/products", () =>
    new HttpResponse(null, { status: 500 })
  ));
  await expect(loadProducts()).rejects.toThrow("HTTP 500");
});
```

На входе API-клиент без аргументов; в первом тесте он возвращает массив, во втором отклоняет Promise. Запустите `npm run test:run`. Для интеграционной проверки отрендерите компонент, использующий тот же клиент, и проверьте loading, данные, пустой список и ошибку. Серверная реализация и ее правила доступа требуют собственных backend-тестов.

## E2E: короткий Playwright-сценарий

Выполните `npm init playwright@latest` и выберите JavaScript или TypeScript. В конфигурации укажите `use.baseURL`, например `http://localhost:5173`, и запустите приложение; для CI настройте `webServer`, чтобы runner запускал его автоматически.

```ts
import { test, expect } from "@playwright/test";

test("пользователь входит в аккаунт", async ({ page }) => {
  await page.goto("/login");
  await page.getByLabel("Почта").fill("user@example.com");
  await page.getByLabel("Пароль").fill("password123");
  await page.getByRole("button", { name: "Войти" }).click();
  await expect(page).toHaveURL(/\/profile$/);
  await expect(page.getByRole("heading", { name: "Профиль" })).toBeVisible();
});
```

Открыть страницу → найти элемент → выполнить действие → проверить результат. Сценарий предполагает реализованные `/login`, `/profile` и тестовую учетную запись с известным паролем. Эти данные относятся к изолированной тестовой среде. Запуск: `npx playwright test`.

Playwright автоматически ожидает готовность элементов и повторяет проверки. Не добавляйте `waitForTimeout` ради стабильности: ожидайте наблюдаемый результат. E2E стоит оставить для нескольких критических путей, а варианты чистой функции дешевле покрыть unit-тестами.

## Частые ошибки

- Проверять внутренний state: такой тест ломается при безопасном рефакторинге.
- Забывать await: тест заканчивается до действия или проверки Promise.
- Проверять только успех и забывать пустой ответ и ошибку.
- Не сбрасывать обработчики и store между тестами: результат зависит от порядка запуска.

## Как ответить на интервью

> Vitest или Jest запускает тесты, RTL помогает проверить UI как пользователь, MSW подменяет сетевую границу. Unit проверяет правило, integration — взаимодействие частей, E2E — критический путь в настоящем браузере. Проверяю наблюдаемый результат и важные ошибки.

## Мини-практика

Добавьте unit-тест суммы с нулевым количеством, component-тест формы с неверной почтой и MSW-ответ с пустым списком. Затем напишите один Playwright-сценарий открытия товара из каталога без произвольных задержек.

## Проверьте себя

1. Чем component-тест может отличаться от unit-теста функции?
2. Почему role/label лучше обращения к внутреннему state?
3. Зачем resetHandlers и cleanup после каждого теста?
4. Как проверить HTTP-ошибку, сохранив настоящий Fetch?
5. Какие сценарии стоит оставить E2E?

[Vitest](https://vitest.dev/guide/) · [MSW в Node.js](https://mswjs.io/docs/integrations/node/) · [Playwright](https://playwright.dev/docs/intro)

[Следующая тема: производительность](performance.md){ .md-button }
