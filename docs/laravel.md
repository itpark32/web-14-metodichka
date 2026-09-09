# Laravel

<p class="reading-time">Чтение: 14 минут</p>

Laravel организует PHP-приложение вокруг маршрутов, middleware, контроллеров, сервисов, моделей, миграций и представлений.

## Путь запроса

```text
Request -> Route -> Middleware -> Controller -> Service -> Model -> Response
```

Middleware проверяет сквозное условие, например аутентификацию. Контроллер преобразует HTTP в вызов сценария. Бизнес-правила не стоит прятать в route closure или Blade.

## Маршрут и контроллер

```php
Route::get('/products/{product}', [ProductController::class, 'show']);

public function show(Product $product): JsonResponse
{
    return response()->json($product);
}
```

Route model binding находит модель по параметру. Resource controller дает стандартные CRUD-методы.

## Миграции и Eloquent

Миграция версионирует структуру базы. Модель описывает данные и связи.

```php
class Order extends Model
{
    public function items(): HasMany
    {
        return $this->hasMany(OrderItem::class);
    }
}
```

Отношения: `hasOne`, `hasMany`, `belongsTo`, `belongsToMany`. Для списка заранее загружайте связи через `with`, чтобы избежать N+1 запросов.

## Валидация и права

Form Request хранит правила и авторизацию запроса. Policy отвечает, может ли пользователь выполнить действие над моделью. Валидация формы не заменяет ограничение базы.

## Blade и API

Blade экранирует `{{ $value }}` по умолчанию. Для React-клиента Laravel может вернуть API Resource, который стабилизирует JSON-контракт.

## Аутентификация

Сессионная аутентификация подходит серверному приложению. Sanctum помогает SPA и токенам. Проверяйте CSRF для cookie-сессии, права на каждом защищенном действии и не возвращайте лишние поля модели.

## Кеш и очереди

Кеш ускоряет дорогие чтения, но требует стратегии инвалидирования. Очередь переносит отправку почты, обработку изображений и другие долгие задачи из HTTP-запроса.

!!! warning "Частая ошибка"
    Массовое присваивание без `$fillable` или `$guarded` позволяет изменить поля, которые пользователь не должен контролировать.

## Мини-задача

Создайте REST API заказов: миграции, связи, Form Request, Policy, транзакцию создания и API Resource.

[Безопасность](security.md){ .md-button .md-button--primary }
