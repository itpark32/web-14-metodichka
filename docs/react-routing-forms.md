# React: роутинг, формы и авторизация

<p class="reading-time">Чтение: 12 минут</p>

Маршрутизация связывает URL с экраном, формы собирают данные, а авторизация ограничивает доступ к действиям и ресурсам.

!!! abstract "Фокус"
    - **Нужно знать:** маршруты, параметры, controlled fields, validation и redirect.
    - **Часто в работе:** вложенные и защищенные маршруты, ошибки формы, отправка в API.
    - **Достаточно узнавать:** Formik и Yup как возможные библиотеки, а не обязательный выбор.

## Клиентская маршрутизация

React Router связывает URL с деревом компонентов без полной перезагрузки страницы.

```jsx
<BrowserRouter>
  <Routes>
  <Route path="/" element={<Catalog />} />
  <Route path="/products/:id" element={<Product />} />
  <Route path="/login" element={<LoginForm />} />
  <Route element={<RequireAuth />}>
    <Route path="/profile" element={<Profile />} />
  </Route>
  <Route path="*" element={<NotFound />} />
  </Routes>
</BrowserRouter>
```

`Link` меняет маршрут, `useParams` читает параметры, `useSearchParams` работает со строкой запроса, `useNavigate` выполняет программный переход.

Для declarative-примера установите `npm install react-router`. Импортируйте `BrowserRouter`, `Routes`, `Route`, `Link`, `useParams`, `useSearchParams`, `useNavigate`, `Navigate`, `Outlet` из `react-router`. В существующих проектах часто встречается `react-router-dom`; ориентируйтесь на установленную версию. `Catalog`, `Product`, `Profile`, `NotFound` — компоненты экранов. BrowserRouter подключают один раз у корня.

```jsx
function Product() {
  const { id } = useParams();
  const navigate = useNavigate();
  return <main>
    <h1>Товар {id}</h1>
    <Link to="/">Каталог</Link>
    <button onClick={() => navigate("/profile")}>Мой профиль</button>
  </main>;
}
```

URL `/products/42` дает строковый `id === "42"`. Для загрузки товара передайте его API-функции. `Link` подходит видимому переходу: сохраняет привычное поведение ссылки. `navigate` нужен для программного перехода, например после успешного сохранения.

```jsx
function Catalog() {
  const [params, setParams] = useSearchParams();
  const query = params.get("q") ?? "";
  return <>
    <label>Поиск
      <input value={query} onChange={event => {
        const next = new URLSearchParams(params);
        next.set("q", event.target.value);
        setParams(next, { replace: true });
      }} />
    </label>
    <Link to="/products/42">Открыть товар 42</Link>
  </>;
}
```

Значение поиска хранится в URL, его можно скопировать вместе со ссылкой. Копия параметров сохраняет остальные фильтры; `replace` не добавляет запись истории на каждый символ. Для реального поиска добавьте список, debounce и состояния API из [главы о данных](react-data.md).

### Защищенный маршрут и 404

```jsx
function RequireAuth() {
  const { user, status } = useAuth();
  if (status === "loading") return <p>Проверяем сессию...</p>;
  if (!user) return <Navigate to="/login" replace />;
  return <Outlet />;
}
```

`useAuth` — хук вашего приложения, читающий проверенную сервером сессию из Provider. `Outlet` показывает вложенный защищенный экран. Маршрут `/login` в примере показывает форму; без него перенаправление попало бы в `*`. Маршрут `*` показывает понятную страницу 404, но в статической SPA это не обязательно HTTP-статус 404. Сервер проверяет права на каждый API-запрос независимо от React Router.

## Контролируемая форма

```jsx
function LoginForm() {
  const [email, setEmail] = useState("");

  function submit(event) {
    event.preventDefault();
    console.log({ email }); // здесь демонстрируется чтение значения, вход еще не выполняется
  }

  return (
    <form onSubmit={submit}>
      <label htmlFor="email">Почта</label>
      <input
        id="email"
        type="email"
        value={email}
        onChange={(event) => setEmail(event.target.value)}
      />
      <button type="submit">Войти</button>
    </form>
  );
}
```

Контролируемое поле получает значение из state. Неконтролируемое хранит значение в DOM и читается через `ref` или `FormData`.

## Валидация

Проверяйте данные на клиенте для удобства и повторно на сервере для доверия. Показывайте ошибку рядом с полем, связывайте ее через `aria-describedby`, переводите фокус к первой ошибке после отправки.

Formik управляет формой, Yup описывает схему, но небольшая форма может обойтись нативными API.

## Авторизация

Авторизация отвечает на вопрос «что пользователь может делать», а аутентификация «кто это». Защищенный маршрут в UI улучшает навигацию, но реальную проверку прав всегда выполняет сервер.

!!! warning "Частая ошибка"
    Не храните право доступа только в React state. Пользователь может изменить клиентский код или вызвать API напрямую.

## Неконтролируемая форма и FormData

```jsx
function ProfileForm() {
  function submit(event) {
    event.preventDefault();
    const data = new FormData(event.currentTarget);
    console.log(data.get("name"));
  }
  return <form onSubmit={submit}>
    <label>Имя <input name="name" defaultValue="Anna" required /></label>
    <button type="submit">Сохранить</button>
  </form>;
}
```

DOM хранит текущее значение, `defaultValue` задает начальное. `name` определяет ключ в FormData. Этот вариант удобен, если значение не требуется React на каждом вводе. Для живого фильтра controlled-поле уместнее. Не переключайте поле между `undefined` и строковым `value`.

## React Hook Form + Zod

Установите `npm install react-hook-form zod @hookform/resolvers`. Сначала опишем схему и подключим ее к форме:

```jsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

const schema = z.object({
  email: z.email("Введите корректную почту"),
  password: z.string().min(8, "Минимум 8 символов"),
});

function RegistrationForm({ onRegister }) {
  const { register, handleSubmit, formState: { errors, isSubmitting } } =
    useForm({ resolver: zodResolver(schema) });
  return (
    <form onSubmit={handleSubmit(onRegister)} noValidate>
      <RegistrationFields register={register} errors={errors} />
      <button disabled={isSubmitting} type="submit">Создать аккаунт</button>
    </form>
  );
}
```

На вход формы приходит асинхронный `onRegister(data)`. Resolver проверяет фактические значения по схеме до вызова callback. `register` связывает поля с библиотекой. Разметку полей вынесем в небольшой компонент:

```jsx
function RegistrationFields({ register, errors }) {
  return <>
    <label htmlFor="reg-email">Почта</label>
    <input id="reg-email" type="email" autoComplete="email"
      {...register("email")} aria-invalid={Boolean(errors.email)}
      aria-describedby={errors.email ? "reg-email-error" : undefined} />
    <p id="reg-email-error" role="alert">{errors.email?.message}</p>
    <label htmlFor="reg-password">Пароль</label>
    <input id="reg-password" type="password" autoComplete="new-password"
      {...register("password")} aria-invalid={Boolean(errors.password)}
      aria-describedby={errors.password ? "reg-password-error" : undefined} />
    <p id="reg-password-error" role="alert">{errors.password?.message}</p>
  </>;
}
```

Ошибки названы рядом с полями и доступны вспомогательным технологиям. RHF по умолчанию умеет фокусировать первое зарегистрированное неверное поле. `noValidate` в примере передает показ ошибок схеме; в небольшой нативной форме можно оставить браузерную валидацию.

Библиотека уменьшает служебный код, но не отменяет HTML и основы React-форм. `onRegister` должен обработать сетевой сбой и серверные `422`: показать ошибку поля через `setError` или общее сообщение. Нельзя считать успешную клиентскую проверку доказательством создания аккаунта. Минимальная длина здесь относится к регистрации, а не к проверке уже существующего пароля при входе.

## Как ответить на интервью

> React Router сопоставляет URL с деревом компонентов и поддерживает параметры и вложенные маршруты. В контролируемой форме React хранит значение поля в состоянии. Клиентская проверка помогает пользователю, но сервер повторяет валидацию и проверку прав. После входа полезно вернуть пользователя к исходному URL.

## Мини-задача

Сделайте маршруты каталога, карточки товара и профиля. Сохраните целевой URL и верните пользователя туда после входа.

## Проверьте себя

1. Чем path-параметр отличается от query-параметра?
2. Когда нужен Link, а когда useNavigate?
3. Где находится значение controlled и uncontrolled поля?
4. Что делает resolver и кто проверяет данные на сервере?
5. Почему защищенный маршрут не защищает API?

[Управление состоянием](state-management.md){ .md-button .md-button--primary }
