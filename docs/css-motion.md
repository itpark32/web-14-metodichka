# 10. Переходы, трансформации и CSS-анимации

<p class="reading-time">Чтение: 8 минут</p>

Переход показывает изменение состояния, трансформация меняет визуальное положение, а keyframes описывают последовательность состояний.

!!! abstract "Фокус"
    - **Нужно знать:** `transition`, `transform`, `@keyframes`, доступность движения.
    - **Часто в работе:** `opacity`, `translate`, `scale`, `animation-fill-mode`.
    - **Достаточно узнавать:** 3D-трансформации и сложные временные функции.

## Плавный переход

```css
.button {
  background-color: #174c43;
  transform: translateY(0);
  transition: background-color 160ms ease, transform 160ms ease;
}

.button:hover {
  background-color: #103c35;
}

.button:active {
  transform: translateY(1px);
}
```

Переход запускается между старым и новым значением. Перечисляйте конкретные свойства вместо `transition: all`, чтобы случайное изменение раскладки не стало анимацией.

## Трансформации

```css
.card:hover {
  transform: translateY(-0.25rem) scale(1.01);
}
```

Трансформация влияет на отрисовку и обычно не сдвигает соседей в потоке. Порядок функций важен: поворот и перемещение используют текущую систему координат.

## Keyframes

```css
@keyframes appear {
  from {
    opacity: 0;
    transform: translateY(0.5rem);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.dialog[open] {
  animation: appear 200ms ease-out both;
}
```

`animation-duration` задает длительность, `iteration-count` число повторов, `delay` задержку, `fill-mode` состояние до или после выполнения.

## Производительность

Чаще всего плавно и дешево анимируются `transform` и `opacity`. Изменение `width`, `height`, `top` или `left` может потребовать нового расчета раскладки. Это не абсолютный запрет, но для движения обычно лучше трансформация.

## Уменьшение движения

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    scroll-behavior: auto !important;
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

Пользователь может попросить систему уменьшить движение. Функциональность интерфейса должна сохраниться без анимации.

!!! warning "Ловушка: бесконечная анимация"
    Движение, которое повторяется постоянно, отвлекает и расходует ресурсы. Используйте его только когда оно сообщает состояние, например загрузку, и останавливайте после завершения.

## Как ответить на интервью

> `transition` интерполирует изменение свойства между двумя состояниями. `transform` перемещает, масштабирует или поворачивает элемент. `@keyframes` описывает несколько этапов. Для движения предпочитаю `transform` и `opacity`, перечисляю свойства явно и учитываю `prefers-reduced-motion`.

## Мини-практика

Добавьте кнопке hover и active, а диалогу короткое появление. Затем включите уменьшение движения в системе или DevTools и убедитесь, что интерфейс остается понятным.

## Проверьте себя

1. Чем transition отличается от animation?
2. Почему `transition: all` может быть проблемой?
3. Сдвигает ли `transform` соседей в потоке?
4. Какие свойства обычно дешевле анимировать?
5. Для чего нужен `prefers-reduced-motion`?

[Следующая тема: Bootstrap](bootstrap.md)
