# Laravel

<p class="reading-time">Чтение: 14 минут</p>

Laravel организует PHP-приложение вокруг маршрутов, middleware, контроллеров, сервисов, моделей, миграций и представлений.

!!! abstract "Фокус"
    - **Нужно знать:** route, request, controller, migration, model, Eloquent и response.
    - **Часто в работе:** Form Request, Policy, middleware, API Resource, Blade и auth.
    - **Достаточно узнавать:** очереди, кеш и устройство контейнера до практической задачи.

## Путь запроса

```text
Request -> Route -> Middleware -> Controller -> Service -> Model -> Response
```

Middleware проверяет сквозное условие, например аутентификацию. Контроллер преобразует HTTP в вызов сценария. Бизнес-правила не стоит прятать в route closure или Blade.

## Маленький endpoint создания товара

В настроенном Laravel-проекте с подключенным API routing создадим классы:

```bash
php artisan make:model Product -m
php artisan make:request StoreProductRequest
php artisan make:controller ProductController
php artisan make:resource ProductResource
php artisan make:policy ProductPolicy --model=Product
```

Команды генерируют заготовки. Нужны настроенная база в `.env` и API authentication, например Sanctum. В проектах без `routes/api.php` сначала подключите API-маршруты по документации своей версии Laravel. Дальнейшие фрагменты размещаются в сгенерированных классах, сохраняя их namespaces и базовые классы.

### 1. Route и middleware

```php
// routes/api.php
use App\Http\Controllers\ProductController;
use Illuminate\Support\Facades\Route;

Route::post('/products', [ProductController::class, 'store'])
    ->middleware('auth:sanctum');
```

API-префикс обычно дает URL `/api/products`. Middleware проверяет аутентификацию до выполнения операции. Клиент передает JSON и `Accept: application/json`, чтобы ошибки возвращались как API-ответы.

### 2. Form Request: права и данные

```php
// В StoreProductRequest; импортируйте App\Models\Product
public function authorize(): bool
{
    return $this->user()?->can('create', Product::class) ?? false;
}

public function rules(): array
{
    return [
        'title' => ['required', 'string', 'max:255'],
        'price' => ['required', 'integer', 'min:0'],
    ];
}
```

Вход — title и price в целых копейках. При неверных данных Laravel возвращает 422 с ошибками полей, при отказе в авторизации — 403. Проверка Form Request выполняется до тела метода контроллера. Не оставляйте authorize случайно возвращающим false, если ожидаете успешный запрос.

В ProductPolicy определите правило создания. Для этого учебного каталога любой аутентифицированный пользователь может создавать товар:

```php
// В ProductPolicy; импортируйте App\Models\User
public function create(User $user): bool
{
    return true;
}
```

Это явно выбранное правило примера. Для реального каталога ограничьте его ролью или разрешением команды. Laravel должен обнаружить Policy по стандартным именам либо получить явную регистрацию. Для изменения конкретного товара потребуется отдельное правило update с проверкой права на эту модель.

### 3. Model и миграция

```php
// В App\Models\Product
protected $fillable = ['title', 'price'];
```

`fillable` разрешает массовое присваивание перечисленных полей. Не передавайте в create весь request: используйте только validated. Миграция создает нужную таблицу:

```php
// В up() миграции products; Schema и Blueprint уже импортированы
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->unsignedInteger('price');
    $table->timestamps();
});
// В down(): Schema::dropIfExists('products');
```

Выполните `php artisan migrate`. Валидация входа и ограничения базы дополняют друг друга; изменение данных не заменяет миграцию схемы.

### 4. Controller и API Resource

```php
// В ProductController; импортируйте StoreProductRequest, ProductResource, Product
public function store(StoreProductRequest $request): ProductResource
{
    $product = Product::create($request->validated());
    return new ProductResource($product);
}
```

Метод получает уже проверенный Request, создает запись и возвращает Resource. Если появятся расчет скидок или несколько связанных записей, вынесите бизнес-сценарий в service и используйте транзакцию.

```php
// В ProductResource; Request — Illuminate\Http\Request
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'title' => $this->title,
        'price' => (int) $this->price,
    ];
}
```

Resource задает публичные поля вместо случайной сериализации всей модели. Для новой модели стандартный Resource-ответ получает 201 и оболочку data; сверяйте это с контрактом клиента. Проверьте успешное создание, неверную цену, отсутствие сессии и отказ Policy.

```text
Route → middleware → Controller (Form Request уже проверен)
                       ↓
                   Model / Eloquent → Resource → JSON
```

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

Загрузки принимаются через Request, валидируются по размеру и MIME-типу и сохраняются через Storage. Путь и публичный URL не должны составляться напрямую из пользовательского имени файла.

## Аутентификация

Сессионная аутентификация подходит серверному приложению. Sanctum помогает SPA и токенам. Проверяйте CSRF для cookie-сессии, права на каждом защищенном действии и не возвращайте лишние поля модели.

## Кеш и очереди

Кеш ускоряет дорогие чтения, но требует стратегии инвалидирования. Очередь переносит отправку почты, обработку изображений и другие долгие задачи из HTTP-запроса.

Админ-панель не является отдельной зоной доверия. Ее маршруты защищают authentication, middleware и policies, а каждое изменение проходит обычную валидацию и авторизацию.

!!! warning "Частая ошибка"
    Массовое присваивание без `$fillable` или `$guarded` позволяет изменить поля, которые пользователь не должен контролировать.

## Окружение и кеш

`.env` хранит локальные параметры, `.env.example` описывает необходимые имена без секретов. Читайте настройки приложения через config; после кеширования конфигурации Laravel работа с env напрямую вне config-файлов может дать неожиданный результат.

```php
use Illuminate\Support\Facades\Cache;
use App\Models\Product;

$products = Cache::remember('catalog.products', 60, function () {
    return Product::query()->select('id', 'title', 'price')->get();
});
// После изменения товара: Cache::forget('catalog.products');
```

Первый вызов читает базу, следующие до истечения TTL могут вернуть кеш. После записи данные устаревают: инвалидируйте ключ или выберите допустимое время устаревания. Кеш не должен позволять одному пользователю прочитать чужие защищенные данные.

## Как ответить на интервью

> Laravel проводит запрос через route и middleware к контроллеру. Form Request проверяет данные, Policy права, Eloquent работает с моделями и связями, API Resource формирует внешний ответ. Миграции версионируют схему, а бизнес-операцию с несколькими записями я оборачиваю в транзакцию.

## Мини-задача

Создайте REST API заказов: миграции, связи, Form Request, Policy, транзакцию создания и API Resource.

## Проверьте себя

1. Что проверяет middleware, Form Request и Policy?
2. Почему validated и fillable нужны вместе?
3. Чем migration отличается от Model?
4. Какие поля должен возвращать API Resource?
5. Когда нужно инвалидировать кеш?

[Laravel: validation](https://laravel.com/docs/13.x/validation) · [Authorization](https://laravel.com/docs/13.x/authorization)

[Безопасность](security.md){ .md-button .md-button--primary }
