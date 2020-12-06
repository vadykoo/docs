# Посібник з оновлення

-   [Оновлення до 8.0 з 7.x](#upgrade-8.0)

<a name="high-impact-changes"></a>

## Значні зміни впливу

<div class="content-list" markdown="1">
- [Model Factories](#model-factories)
- [Queue `retryAfter` Method](#queue-retry-after-method)
- [Queue `timeoutAt` Property](#queue-timeout-at-property)
- [Queue `allOnQueue` and `allOnConnection`](#queue-allOnQueue-allOnConnection)
- [Pagination Defaults](#pagination-defaults)
- [Seeder & Factory Namespaces](#seeder-factory-namespaces)
</div>

<a name="medium-impact-changes"></a>

## Зміни середнього впливу

<div class="content-list" markdown="1">
- [PHP 7.3.0 Required](#php-7.3.0-required)
- [Failed Jobs Table Batch Support](#failed-jobs-table-batch-support)
- [Maintenance Mode Updates](#maintenance-mode-updates)
- [The `php artisan down --message` Option](#artisan-down-message)
- [The `assertExactJson` Method](#assert-exact-json-method)
</div>

<a name="upgrade-8.0"></a>

## Оновлення до 8.0 з 7.x

<a name="estimated-upgrade-time-15-minutes"></a>

#### Розрахунковий час оновлення: 15 хвилин

> {note} Ми намагаємося задокументувати всі можливі порушення. Оскільки деякі з цих невід'ємних змін знаходяться в незрозумілих частинах фреймворку, лише частина цих змін може насправді вплинути на вашу програму.

<a name="php-7.3.0-required"></a>

### Потрібно PHP 7.3.0

**Ймовірність впливу: середній**

Нова мінімальна версія PHP тепер 7.3.0.

<a name="updating-dependencies"></a>

### Оновлення залежностей

Оновіть наведені нижче залежності у вашому`composer.json`файл:

<div class="content-list" markdown="1">
- `guzzlehttp/guzzle` to `^7.0.1`
- `facade/ignition` to `^2.3.6`
- `laravel/framework` to `^8.0`
- `laravel/ui` to `^3.0`
- `nunomaduro/collision` to `^5.0`
- `phpunit/phpunit` to `^9.0`
</div>

Наступні основні пакети мають нові основні випуски для підтримки Laravel 8. Якщо це можливо, перед оновленням слід прочитати їхні індивідуальні посібники з оновлення:

<div class="content-list" markdown="1">
- [Horizon v5.0](https://github.com/laravel/horizon/blob/master/UPGRADE.md)
- [Passport v10.0](https://github.com/laravel/passport/blob/master/UPGRADE.md)
- [Socialite v5.0](https://github.com/laravel/socialite/blob/master/UPGRADE.md)
- [Telescope v4.0](https://github.com/laravel/telescope/blob/master/UPGRADE.md)
</div>

Крім того, інсталятор Laravel оновлено для підтримки`composer create-project`та Laravel Jetstream. Будь-який інсталятор старше 4,0 перестане працювати після жовтня 2020 року. Вам слід оновити глобальний інсталятор до`^4.0`якнайшвидше.

Нарешті, вивчіть будь-які інші сторонні пакети, споживані вашим додатком, і переконайтесь, що ви використовуєте відповідну версію для підтримки Laravel 8.

<a name="collections"></a>

### Колекції

<a name="the-isset-method"></a>

#### `isset`Метод

**Ймовірність впливу: низька**

Щоб відповідати типовій поведінці PHP, файл`offsetExists`метод`Illuminate\Support\Collection`було оновлено до використання`isset`замість`array_key_exists`. Це може змінити поведінку при роботі з предметами колекції, які мають значення`null`:

    $collection = collect([null]);

    // Laravel 7.x - true
    isset($collection[0]);

    // Laravel 8.x - false
    isset($collection[0]);

<a name="database"></a>

### База даних

<a name="seeder-factory-namespaces"></a>

#### Простір імен сівалок та фабрик

**Ймовірність впливу: висока**

Зараз сівалки та заводи розміщені на просторах імен. Щоб врахувати ці зміни, додайте`Database\Seeders`простору імен для ваших класів сівалок. Крім того, попередній`database/seeds`каталог повинен бути перейменований на`database/seeders`:

    <?php

    namespace Database\Seeders;

    use App\Models\User;
    use Illuminate\Database\Seeder;

    class DatabaseSeeder extends Seeder
    {
        /**
         * Seed the application's database.
         *
         * @return void
         */
        public function run()
        {
            ...
        }
    }

Якщо ви вирішили використовувати`laravel/legacy-factories`пакунку, ніяких змін у ваших заводських класах не потрібно. Однак, якщо ви модернізуєте свої заводи, вам слід додати`Database\Factories`простір імен для цих класів.

Далі, у вашому`composer.json`файл, видаліть`classmap`блок від`autoload`і додайте нові зіставлення каталогів класів із простором імен:

    "autoload": {
        "psr-4": {
            "App\\": "app/",
            "Database\\Factories\\": "database/factories/",
            "Database\\Seeders\\": "database/seeders/"
        }
    },

<a name="eloquent"></a>

### Красномовний

<a name="model-factories"></a>

#### Модельні фабрики

**Ймовірність впливу: висока**

Ларавеля[модельні заводи](/docs/{{version}}/database-testing#creating-factories)Функція була повністю переписана для підтримки класів і не сумісна з фабриками стилю Laravel 7.x. Однак, щоб полегшити процес оновлення, новий`laravel/legacy-factories`пакет створений для продовження використання існуючих фабрик з Laravel 8.x. Ви можете встановити цей пакет через Composer:

    composer require laravel/legacy-factories

<a name="the-castable-interface"></a>

#### `Castable`Інтерфейс

**Ймовірність впливу: низька**

`castUsing`метод`Castable`інтерфейс оновлений, щоб прийняти масив аргументів. Якщо ви впроваджуєте цей інтерфейс, вам слід відповідно оновити реалізацію:

    public static function castUsing(array $arguments);

<a name="increment-decrement-events"></a>

#### Події збільшення / зменшення

**Ймовірність впливу: низька**

Належне "оновлення" та "збереження" пов'язаних з моделлю подій тепер буде відправлено під час запуску`increment`або`decrement`методи на екземплярах красномовної моделі.

<a name="events"></a>

### Події

<a name="the-dispatcher-contract"></a>

#### `Dispatcher`Договір

**Ймовірність впливу: низька**

`listen`метод`Illuminate\Contracts\Events\Dispatcher`контракт був оновлений, щоб зробити`$listener`властивість необов’язкова. Ця зміна була зроблена для підтримки автоматичного виявлення оброблених типів подій за допомогою відображення. Якщо ви реалізуєте цей інтерфейс вручну, вам слід відповідно оновити реалізацію:

    public function listen($events, $listener = null);

<a name="framework"></a>

### Рамки

<a name="maintenance-mode-updates"></a>

#### Оновлення режиму обслуговування

**Імовірність впливу: Необов’язково**

[режим обслуговування](/docs/{{version}}/configuration#maintenance-mode)особливість Laravel була покращена в Laravel 8.x. Попередня візуалізація шаблону режиму обслуговування тепер підтримується і виключає ймовірність зіткнення кінцевих користувачів під час режиму обслуговування. Однак, щоб підтвердити це, наступні рядки потрібно додати до вашого`public/index.php`файл. Ці рядки слід розміщувати безпосередньо під існуючими`LARAVEL_START`постійне визначення:

    define('LARAVEL_START', microtime(true));

    if (file_exists(__DIR__.'/../storage/framework/maintenance.php')) {
        require __DIR__.'/../storage/framework/maintenance.php';
    }

<a name="artisan-down-message"></a>

#### `php artisan down --message`Варіант

**Ймовірність впливу: середній**

`--message`варіант`php artisan down`команда була видалена. В якості альтернативи розглянемо[попередній рендерінг подань режиму обслуговування](/docs/{{version}}/configuration#maintenance-mode)з повідомленням на ваш вибір.

<a name="php-artisan-serve-no-reload-option"></a>

#### `php artisan serve --no-reload`Варіант

**Ймовірність впливу: низька**

A`--no-reload`опція була додана до`php artisan serve`команди. Це дасть вказівку вбудованому серверу не перезавантажувати сервер, коли будуть виявлені зміни файлів середовища. Цей параметр в першу чергу корисний під час запуску тестів Laravel Dusk в середовищі CI.

<a name="manager-app-property"></a>

#### Менеджер`$app`Власність

**Ймовірність впливу: низька**

Раніше застарілий`$app`власність`Illuminate\Support\Manager`клас видалено. Якщо ви покладалися на цю властивість, вам слід використовувати`$container`власності замість цього.

<a name="the-elixir-helper"></a>

#### `elixir`Помічник

**Ймовірність впливу: низька**

Раніше застарілий`elixir`помічник видалено. Програми, які все ще використовують цей метод, рекомендується оновити до[Laravel Mix](https://github.com/JeffreyWay/laravel-mix).

<a name="mail"></a>

### Пошта

<a name="the-sendnow-method"></a>

#### `sendNow`Метод

**Ймовірність впливу: низька**

Раніше застарілий`sendNow`метод видалено. Натомість використовуйте`send`метод.

<a name="pagination"></a>

### Пагінація

<a name="pagination-defaults"></a>

#### За замовчуванням розміщення сторінок

**Ймовірність впливу: висока**

Пагінатор тепер використовує файл[Фреймворк CSS Tailwind](https://tailwindcss.com)для стилю за замовчуванням. Щоб продовжувати використовувати Bootstrap, слід додати наступний виклик методу до`boot`метод вашої програми`AppServiceProvider`:

    use Illuminate\Pagination\Paginator;

    Paginator::useBootstrap();

<a name="queue"></a>

### Черга

<a name="queue-retry-after-method"></a>

#### `retryAfter`Метод

**Ймовірність впливу: висока**

Для узгодження з іншими особливостями Laravel,`retryAfter`метод і`retryAfter`властивість завдань, що перебувають у черзі, розсильників, сповіщень та слухачів було перейменовано на`backoff`. Вам слід оновити назву цього методу / властивості у відповідних класах у вашій програмі.

<a name="queue-timeout-at-property"></a>

#### `timeoutAt`Власність

**Ймовірність впливу: висока**

`timeoutAt`властивість завдань у черзі, сповіщень та слухачів було перейменовано на`retryUntil`. Вам слід оновити назву цього властивості у відповідних класах у вашій програмі.

<a name="queue-allOnQueue-allOnConnection"></a>

#### `allOnQueue()`/`allOnConnection()`Методи

**Ймовірність впливу: висока**

Для узгодження з іншими методами диспетчеризації,`allOnQueue()`і`allOnConnection()`методи, що використовуються з ланцюжком робіт, були вилучені. Ви можете використовувати`onQueue()`і`onConnection()`методи замість цього. Ці методи слід викликати перед викликом`dispatch`метод:

    ProcessPodcast::withChain([
        new OptimizePodcast,
        new ReleasePodcast
    ])->onConnection('redis')->onQueue('podcasts')->dispatch();

Зверніть увагу, що ця зміна стосується лише коду, що використовує`withChain`метод.`allOnQueue()`і`allOnConnection()`все ще доступні при використанні глобальної`dispatch()`помічник.

<a name="failed-jobs-table-batch-support"></a>

#### Підтримка пакетної таблиці невдалих завдань

**Імовірність впливу: Необов’язково**

Якщо ви плануєте використовувати[пакет роботи](/docs/{{version}}/queues#job-batching)особливості Laravel 8.x, ваш`failed_jobs`таблицю бази даних потрібно буде оновити. По-перше, новий`uuid`стовпець слід додати до вашої таблиці:

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::table('failed_jobs', function (Blueprint $table) {
        $table->string('uuid')->after('id')->nullable()->unique();
    });

Далі,`failed.driver`варіант конфігурації у вашому`queue`файл конфігурації слід оновити до`database-uuids`.

Крім того, ви можете створити UUID для існуючих невдалих завдань:

    DB::table('failed_jobs')->whereNull('uuid')->cursor()->each(function ($job) {
        DB::table('failed_jobs')
            ->where('id', $job->id)
            ->update(['uuid' => (string) Illuminate\Support\Str::uuid()]);
    });

<a name="routing"></a>

### Маршрутизація

<a name="automatic-controller-namespace-prefixing"></a>

#### Префікс простору імен автоматичного контролера

**Імовірність впливу: Необов’язково**

У попередніх випусках Laravel,`RouteServiceProvider`клас містив a`$namespace`властивість зі значенням`App\Http\Controllers`. Це значення цієї властивості було використано для автоматичного префікса оголошень маршруту контролера та генерації URL-адреси маршруту контролера, наприклад, під час виклику`action`помічник.

У Laravel 8 для цієї властивості встановлено значення`null`за замовчуванням. Це дозволяє оголошенням маршрутів контролера використовувати стандартний синтаксис, що викликається PHP, що забезпечує кращу підтримку переходу до класу контролера у багатьох середовищах розробки:

    use App\Http\Controllers\UserController;

    // Using PHP callable syntax...
    Route::get('/users', [UserController::class, 'index']);

    // Using string syntax...
    Route::get('/users', 'App\Http\Controllers\UserController@index');

У більшості випадків це не вплине на програми, які оновлюються, оскільки ваш`RouteServiceProvider`все ще міститиме`$namespace`майно з попередньою вартістю. Однак, якщо ви оновите свою програму, створивши абсолютно новий проект Laravel, ви можете зіткнутися з цим як зломлива зміна.

Якщо ви хочете продовжувати використовувати оригінальну маршрутизацію контролера з попередньо встановленим префіксом, ви можете просто встановити значення`$namespace`майно в межах вашого`RouteServiceProvider`та оновити реєстрацію маршруту в межах`boot`метод використання`$namespace`властивість:

    class RouteServiceProvider extends ServiceProvider
    {
        /**
         * The path to the "home" route for your application.
         *
         * This is used by Laravel authentication to redirect users after login.
         *
         * @var string
         */
        public const HOME = '/home';

        /**
         * If specified, this namespace is automatically applied to your controller routes.
         *
         * In addition, it is set as the URL generator's root namespace.
         *
         * @var string
         */
        protected $namespace = 'App\Http\Controllers';

        /**
         * Define your route model bindings, pattern filters, etc.
         *
         * @return void
         */
        public function boot()
        {
            $this->configureRateLimiting();

            $this->routes(function () {
                Route::middleware('web')
                    ->namespace($this->namespace)
                    ->group(base_path('routes/web.php'));

                Route::prefix('api')
                    ->middleware('api')
                    ->namespace($this->namespace)
                    ->group(base_path('routes/api.php'));
            });
        }

        /**
         * Configure the rate limiters for the application.
         *
         * @return void
         */
        protected function configureRateLimiting()
        {
            RateLimiter::for('api', function (Request $request) {
                return Limit::perMinute(60);
            });
        }
    }

<a name="scheduling"></a>

### Планування

<a name="the-cron-expression-library"></a>

#### `cron-expression`Бібліотека

**Ймовірність впливу: низька**

Залежність Ларавеля від`dragonmantank/cron-expression`було оновлено з`2.x`до`3.x`. Це не повинно спричинити жодних змін у вашій програмі, якщо ви не взаємодієте з`cron-expression`бібліотека безпосередньо. Якщо ви взаємодієте з цією бібліотекою безпосередньо, перегляньте її[журнал змін](https://github.com/dragonmantank/cron-expression/blob/master/CHANGELOG.md).

<a name="session"></a>

### Сесія

<a name="the-session-contract"></a>

#### `Session`Договір

**Ймовірність впливу: низька**

`Illuminate\Contracts\Session\Session`контракт отримав новий`pull`метод. Якщо ви реалізуєте цей контракт вручну, вам слід відповідно оновити його реалізацію:

    /**
     * Get the value of a given key and then forget it.
     *
     * @param  string  $key
     * @param  mixed  $default
     * @return mixed
     */
    public function pull($key, $default = null);

<a name="testing"></a>

### Тестування

<a name="decode-response-json-method"></a>

#### `decodeResponseJson`Метод

**Ймовірність впливу: низька**

`decodeResponseJson`метод, що належить до`Illuminate\Testing\TestResponse`клас більше не приймає жодних аргументів. Будь ласка, розгляньте можливість використання`json`замість цього.

<a name="assert-exact-json-method"></a>

#### `assertExactJson`Метод

**Ймовірність впливу: середній**

`assertExactJson`Метод тепер вимагає, щоб числові клавіші порівняних масивів збігалися та були в тому самому порядку. Якщо ви хочете порівняти JSON з масивом, не вимагаючи, щоб масиви з цифровою клавіатурою мали однаковий порядок, ви можете використовувати`assertSimilarJson`замість цього.

<a name="validation"></a>

### Перевірка

<a name="database-rule-connections"></a>

### Зв'язки правил бази даних

**Ймовірність впливу: низька**

`unique`і`exists`правила тепер будуть поважати вказане ім'я з'єднання (доступ до якого здійснюється через`getConnectionName`метод) Eloquent моделей при виконанні запитів.

<a name="miscellaneous"></a>

### Різне

Ми також закликаємо Вас переглянути зміни в`laravel/laravel`[Репозиторій GitHub](https://github.com/laravel/laravel). Хоча багато з цих змін не потрібні, можливо, ви захочете синхронізувати ці файли із вашим додатком. Деякі з цих змін будуть висвітлені в цьому посібнику з оновлення, а інші, наприклад, зміни у файлах конфігурації чи коментарі, не будуть. Ви можете легко переглянути зміни за допомогою[Інструмент порівняння GitHub](https://github.com/laravel/laravel/compare/7.x...master)і виберіть, які оновлення є важливими для вас.
