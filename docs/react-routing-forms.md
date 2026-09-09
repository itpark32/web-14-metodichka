# React: роутинг, формы и авторизация

<p class="reading-time">Чтение: 12 минут</p>

## Клиентская маршрутизация

React Router связывает URL с деревом компонентов без полной перезагрузки страницы.

```jsx
<Routes>
  <Route path="/" element={<Catalog />} />
  <Route path="/products/:id" element={<Product />} />
  <Route element={<RequireAuth />}>
    <Route path="/profile" element={<Profile />} />
  </Route>
  <Route path="*" element={<NotFound />} />
</Routes>
```

`Link` меняет маршрут, `useParams` читает параметры, `useSearchParams` работает со строкой запроса, `useNavigate` выполняет программный переход.

## Контролируемая форма

```jsx
function LoginForm() {
  const [email, setEmail] = useState("");

  function submit(event) {
    event.preventDefault();
    // validate and send
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

## Мини-задача

Сделайте маршруты каталога, карточки товара и профиля. Сохраните целевой URL и верните пользователя туда после входа.

[Управление состоянием](state-management.md){ .md-button .md-button--primary }
