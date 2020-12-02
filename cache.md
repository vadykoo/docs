# Кеш

-   [Конфігурація](#configuration)
    -   [Передумови драйвера](#driver-prerequisites)
-   [Використання кешу](#cache-usage)
    -   [Отримання екземпляра кешу](#obtaining-a-cache-instance)
    -   [Отримання елементів із кешу](#retrieving-items-from-the-cache)
    -   [Зберігання предметів у кеші](#storing-items-in-the-cache)
    -   [Видалення елементів із кешу](#removing-items-from-the-cache)
    -   [Помічник кешу](#the-cache-helper)
-   [Теги кешу](#cache-tags)
    -   [Зберігання позначених елементів кешу](#storing-tagged-cache-items)
    -   [Доступ до позначених елементів кешу](#accessing-tagged-cache-items)
    -   [Видалення позначених елементів кешу](#removing-tagged-cache-items)
-   [Атомні замки](#atomic-locks)
    -   [Передумови драйвера](#lock-driver-prerequisites)
    -   [Управління замками](#managing-locks)
    -   [Управління замками в різних процесах](#managing-locks-across-processes)
-   [Додавання власних драйверів кешу](#adding-custom-cache-drivers)
    -   [Написання драйвера](#writing-the-driver)
    -   [Реєстрація драйвера](#registering-the-driver)
-   [Події](#events)

<a name="configuration"></a>

## Конфігурація

Laravel забезпечує виразний уніфікований API для різних кешуючих кеш-файлів. Конфігурація кешу знаходиться за адресою`config/cache.php`. У цьому файлі ви можете вказати, який драйвер кешу ви хочете використовувати за замовчуванням у всій програмі. Laravel підтримує такі популярні кешування, як[Memcached](https://memcached.org),[Редіс](https://redis.io), і[ДинамоДБ](https://aws.amazon.com/dynamodb)з коробки. Laravel також підтримує APC, Array, Database, File та`null`кеш драйвера.

Файл конфігурації кешу також містить різні інші параметри, які задокументовані у файлі, тому обов’язково прочитайте ці параметри. За замовчуванням Laravel налаштовано на використання`file`кеш-драйвер, який зберігає серіалізовані кешовані об’єкти у файловій системі. Для великих програм рекомендується використовувати більш надійний драйвер, такий як Memcached або Redis. Ви навіть можете налаштувати кілька конфігурацій кешу для одного драйвера.

<a name="driver-prerequisites"></a>

### Передумови драйвера

<a name="prerequisites-database"></a>

#### База даних

При використанні`database`кеш-драйвер, вам потрібно буде налаштувати таблицю, щоб містити елементи кешу. Ви знайдете приклад`Schema`декларація для таблиці нижче:

    Schema::create('cache', function ($table) {
        $table->string('key')->unique();
        $table->text('value');
        $table->integer('expiration');
    });

> {tip} Ви також можете використовувати`php artisan cache:table`Команда Artisan для генерації міграції з відповідною схемою.

<a name="memcached"></a>

#### Memcached

Using the Memcached driver requires the [Пакет Memcached PECL](https://pecl.php.net/package/memcached)для встановлення. Ви можете перерахувати всі свої сервери Memcached у`config/cache.php`файл конфігурації:

    'memcached' => [
        [
            'host' => '127.0.0.1',
            'port' => 11211,
            'weight' => 100
        ],
    ],

Ви також можете встановити`host`опція до шляху сокета UNIX. Якщо ви зробите це,`port`для параметра слід встановити значення`0`:

    'memcached' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],

<a name="redis"></a>

#### Редіс

Перш ніж використовувати кеш Redis з Laravel, вам доведеться або встановити розширення PHpRedis PHP через PECL, або встановити`predis/predis`пакет (~ 1.0) через Composer.

Щоб отримати додаткову інформацію про налаштування Redis, перегляньте його[Сторінка документації Laravel](/docs/{{version}}/redis#configuration).

<a name="cache-usage"></a>

## Використання кешу

<a name="obtaining-a-cache-instance"></a>

### Отримання екземпляра кешу

`Illuminate\Contracts\Cache\Factory`і`Illuminate\Contracts\Cache\Repository`[контракти](/docs/{{version}}/contracts)надати доступ до кеш-послуг Laravel.`Factory`Договір надає доступ до всіх драйверів кешу, визначених для вашої програми.`Repository`Контракт, як правило, є реалізацією драйвера кешування за замовчуванням для Вашої програми, як вказано Вашим`cache`файл конфігурації.

Однак ви також можете використовувати`Cache`фасад, що і будемо використовувати в цій документації.`Cache`фасад забезпечує зручний, стислий доступ до основних реалізацій контрактів кешування Laravel:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Show a list of all users of the application.
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

<a name="accessing-multiple-cache-stores"></a>

#### Доступ до декількох магазинів кеш-пам’яті

Використання`Cache`Фасад, ви можете отримати доступ до різних сховищ кешу через`store`метод. Ключ переданий до`store`метод повинен відповідати одному з магазинів, перелічених у`stores`конфігураційний масив у вашому`cache`файл конфігурації:

    $value = Cache::store('file')->get('foo');

    Cache::store('redis')->put('bar', 'baz', 600); // 10 Minutes

<a name="retrieving-items-from-the-cache"></a>

### Отримання елементів із кешу

`get` method on the `Cache`фасад використовується для отримання елементів з кешу. Якщо елемент не існує в кеші,`null`буде повернено. Якщо ви хочете, ви можете передати другий аргумент`get`метод, що визначає значення за замовчуванням, яке потрібно повернути, якщо елемент не існує:

    $value = Cache::get('key');

    $value = Cache::get('key', 'default');

Ви можете навіть скласти`Closure`як значення за замовчуванням. Результат`Closure`буде повернено, якщо вказаний елемент не існує в кеші. Передача закриття дозволяє відкласти отримання значень за замовчуванням з бази даних або іншої зовнішньої служби:

    $value = Cache::get('key', function () {
        return DB::table(...)->get();
    });

<a name="checking-for-item-existence"></a>

#### Перевірка наявності товару

`has`метод може бути використаний, щоб визначити, чи існує елемент у кеші. Цей метод повернеться`false`якщо значення дорівнює`null`:

    if (Cache::has('key')) {
        //
    }

<a name="incrementing-decrementing-values"></a>

#### Збільшення / зменшення значень

`increment`і`decrement` methods may be used to adjust the value of integer items in the cache. Both of these methods accept an optional second argument indicating the amount by which to increment or decrement the item's value:

    Cache::increment('key');
    Cache::increment('key', $amount);
    Cache::decrement('key');
    Cache::decrement('key', $amount);

<a name="retrieve-store"></a>

#### Отримати та зберегти

Іноді вам може знадобитися отримати елемент із кешу, але також зберегти значення за замовчуванням, якщо запитуваний елемент не існує. Наприклад, можливо, ви захочете отримати всіх користувачів із кешу або, якщо вони не існують, отримати їх із бази даних та додати до кешу. Ви можете зробити це за допомогою`Cache::remember`метод:

    $value = Cache::remember('users', $seconds, function () {
        return DB::table('users')->get();
    });

Якщо елемент не існує в кеші, файл`Closure`перейшов до`remember`буде виконано, а його результат буде поміщено в кеш.

Ви можете використовувати`rememberForever`спосіб отримати елемент з кешу або зберегти його назавжди:

    $value = Cache::rememberForever('users', function () {
        return DB::table('users')->get();
    });

<a name="retrieve-delete"></a>

#### Отримати та видалити

Якщо вам потрібно отримати елемент з кешу, а потім видалити елемент, ви можете використовувати`pull`метод. Подобається`get`метод,`null`буде повернено, якщо елемент не існує в кеші:

    $value = Cache::pull('key');

<a name="storing-items-in-the-cache"></a>

### Зберігання предметів у кеші

Ви можете використовувати`put`метод на`Cache`фасад для зберігання предметів у кеші:

    Cache::put('key', 'value', $seconds);

Якщо час зберігання не передано в`put`метод, предмет буде зберігатися необмежений час:

    Cache::put('key', 'value');

Замість того, щоб передавати кількість секунд як ціле число, ви також можете передати a`DateTime`екземпляр, що представляє час закінчення кешованого елемента:

    Cache::put('key', 'value', now()->addMinutes(10));

<a name="store-if-not-present"></a>

#### Зберігати, якщо немає

`add`метод додасть елемент до кешу, лише якщо він ще не існує у сховищі кешу. Метод повернеться`true`якщо елемент фактично додано до кешу. В іншому випадку метод повернеться`false`:

    Cache::add('key', 'value', $seconds);

<a name="storing-items-forever"></a>

#### Зберігання предметів назавжди

`forever`метод може використовуватися для постійного зберігання елемента в кеші. Оскільки ці елементи не закінчуються, їх потрібно вручну видалити з кешу за допомогою`forget`метод:

    Cache::forever('key', 'value');

> {tip} Якщо ви використовуєте драйвер Memcached, елементи, які зберігаються "назавжди", можуть бути видалені, коли кеш досягне граничного розміру.

<a name="removing-items-from-the-cache"></a>

### Видалення елементів із кешу

Ви можете видалити елементи з кешу за допомогою`forget`метод:

    Cache::forget('key');

Ви також можете видалити елементи, надавши нульовий або від’ємний TTL:

    Cache::put('key', 'value', 0);

    Cache::put('key', 'value', -5);

Ви можете очистити весь кеш за допомогою`flush`метод:

    Cache::flush();

> {note} Очищення кеш-пам’яті не поважає префікс кеш-пам’яті та видалить усі записи з кешу. Розгляньте це уважно, очищаючи кеш-пам’ять, яким користуються інші програми.

<a name="the-cache-helper"></a>

### Помічник кешу

На додаток до використання`Cache`фасад або[договір кешування](/docs/{{version}}/contracts), Ви також можете використовувати глобальний`cache`функція для отримання та збереження даних через кеш. Коли`cache`Функція викликається одним аргументом рядка, вона поверне значення даного ключа:

    $value = cache('key');

Якщо ви надаєте масив пар ключ / значення та час закінчення для функції, вона буде зберігати значення в кеші протягом зазначеної тривалості:

    cache(['key' => 'value'], $seconds);

    cache(['key' => 'value'], now()->addMinutes(10));

Коли`cache`Функція викликається без будь-яких аргументів, вона повертає екземпляр`Illuminate\Contracts\Cache\Factory`реалізація, що дозволяє викликати інші методи кешування:

    cache()->remember('users', $seconds, function () {
        return DB::table('users')->get();
    });

> {tip} Під час тестування дзвінок у глобальний`cache`Ви можете використовувати`Cache::shouldReceive`метод так само, як ніби ти[випробування фасаду](/docs/{{version}}/mocking#mocking-facades).

<a name="cache-tags"></a>

## Теги кешу

> {note} Теги кешу не підтримуються під час використання`file`,`dynamodb`, або`database`драйвери кешу. Крім того, при використанні декількох тегів з кешами, які зберігаються "назавжди", продуктивність буде найкращою з таким драйвером, як`memcached`, який автоматично видаляє застарілі записи.

<a name="storing-tagged-cache-items"></a>

### Зберігання позначених елементів кешу

Теги кеш-пам’яті дозволяють позначати пов’язані елементи в кеші, а потім очищати всі кешовані значення, яким було призначено даний тег. Ви можете отримати доступ до позначеного кешу, передавши впорядкований масив імен тегів. Наприклад, давайте отримаємо доступ до позначеного кешу та`put`значення в кеші:

    Cache::tags(['people', 'artists'])->put('John', $john, $seconds);

    Cache::tags(['people', 'authors'])->put('Anne', $anne, $seconds);

<a name="accessing-tagged-cache-items"></a>

### Доступ до позначених елементів кешу

Щоб отримати позначений елемент кешу, передайте той самий упорядкований список тегів до`tags`а потім викличте`get`метод із ключем, який ви хочете отримати:

    $john = Cache::tags(['people', 'artists'])->get('John');

    $anne = Cache::tags(['people', 'authors'])->get('Anne');

<a name="removing-tagged-cache-items"></a>

### Видалення позначених елементів кешу

Ви можете очистити всі елементи, яким призначено тег або список тегів. Наприклад, це твердження видалить усі кеші, позначені будь-яким`people`,`authors`, або обидва. Отже, обидва`Anne`і`John`буде видалено з кешу:

    Cache::tags(['people', 'authors'])->flush();

На відміну від цього, це твердження видалить лише кеші, позначені тегом`authors`, тому`Anne`буде видалено, але ні`John`:

    Cache::tags('authors')->flush();

<a name="atomic-locks"></a>

## Атомні замки

> {note} Щоб використовувати цю функцію, ваша програма повинна використовувати`memcached`,`redis`,`dynamodb`,`database`,`file`, або`array`кеш-драйвер як драйвер кешу вашого додатка за замовчуванням. Крім того, усі сервери повинні взаємодіяти з одним і тим же сервером центрального кешу.

<a name="lock-driver-prerequisites"></a>

### Передумови драйвера

<a name="atomic-locks-prerequisites-database"></a>

#### База даних

При використанні`database`кеш-драйвер, вам потрібно буде налаштувати таблицю, щоб містити блокування кешу. Ви знайдете приклад`Schema`декларація для таблиці нижче:

    Schema::create('cache_locks', function ($table) {
        $table->string('key')->primary();
        $table->string('owner');
        $table->integer('expiration');
    });

<a name="managing-locks"></a>

### Управління замками

Атомні замки дозволяють маніпулювати розподіленими замками, не турбуючись про умови перегонів. Наприклад,[Кузня Laravel](https://forge.laravel.com)використовує атомні блокування, щоб гарантувати, що на сервері одночасно виконується лише одне віддалене завдання. Ви можете створювати та керувати замками за допомогою`Cache::lock`метод:

    use Illuminate\Support\Facades\Cache;

    $lock = Cache::lock('foo', 10);

    if ($lock->get()) {
        // Lock acquired for 10 seconds...

        $lock->release();
    }

`get`метод також приймає Закриття. Після виконання Закриття Laravel автоматично відпустить замок:

    Cache::lock('foo')->get(function () {
        // Lock acquired indefinitely and automatically released...
    });

Якщо замок недоступний на момент запиту, ви можете доручити Laravel почекати вказану кількість секунд. Якщо замок не вдається отримати протягом зазначеного терміну, a`Illuminate\Contracts\Cache\LockTimeoutException`буде кинуто:

    use Illuminate\Contracts\Cache\LockTimeoutException;

    $lock = Cache::lock('foo', 10);

    try {
        $lock->block(5);

        // Lock acquired after waiting maximum of 5 seconds...
    } catch (LockTimeoutException $e) {
        // Unable to acquire lock...
    } finally {
        optional($lock)->release();
    }

    Cache::lock('foo', 10)->block(5, function () {
        // Lock acquired after waiting maximum of 5 seconds...
    });

<a name="managing-locks-across-processes"></a>

### Управління замками в різних процесах

Іноді, можливо, ви захочете отримати замок в одному процесі та звільнити його в іншому процесі. Наприклад, ви можете отримати блокування під час веб-запиту і захочете звільнити блокування в кінці завдання, яке знаходиться в черзі, яке ініціюється цим запитом. У цьому сценарії ви повинні передати охоплюваний замок "токен власника" до завдання в черзі, щоб завдання могло повторно створити блокування за допомогою даного маркера:

    // Within Controller...
    $podcast = Podcast::find($id);

    $lock = Cache::lock('foo', 120);

    if ($result = $lock->get()) {
        ProcessPodcast::dispatch($podcast, $lock->owner());
    }

    // Within ProcessPodcast Job...
    Cache::restoreLock('foo', $this->owner)->release();

Якщо ви хочете звільнити замок, не поважаючи його поточного власника, ви можете використовувати`forceRelease`метод:

    Cache::lock('foo')->forceRelease();

<a name="adding-custom-cache-drivers"></a>

## Додавання власних драйверів кешу

<a name="writing-the-driver"></a>

### Написання драйвера

Щоб створити наш власний драйвер кешу, нам спочатку потрібно реалізувати`Illuminate\Contracts\Cache\Store`[контракт](/docs/{{version}}/contracts). Отже, реалізація кешу MongoDB буде виглядати приблизно так:

    <?php

    namespace App\Extensions;

    use Illuminate\Contracts\Cache\Store;

    class MongoStore implements Store
    {
        public function get($key) {}
        public function many(array $keys) {}
        public function put($key, $value, $seconds) {}
        public function putMany(array $values, $seconds) {}
        public function increment($key, $value = 1) {}
        public function decrement($key, $value = 1) {}
        public function forever($key, $value) {}
        public function forget($key) {}
        public function flush() {}
        public function getPrefix() {}
    }

Нам просто потрібно реалізувати кожен із цих методів за допомогою з’єднання MongoDB. Для прикладу того, як реалізувати кожен із цих методів, погляньте на`Illuminate\Cache\MemcachedStore`у вихідному коді фреймворку. Після завершення нашого впровадження ми зможемо закінчити власну реєстрацію драйверів.

    Cache::extend('mongo', function ($app) {
        return Cache::repository(new MongoStore);
    });

> {tip} Якщо вам цікаво, куди покласти власний код драйвера кеш-пам'яті, ви можете створити файл`Extensions`простір імен у вашому`app`каталог. Однак майте на увазі, що Laravel не має жорсткої структури додатків, і ви можете організувати свою заявку відповідно до своїх уподобань.

<a name="registering-the-driver"></a>

### Реєстрація драйвера

Щоб зареєструвати власний драйвер кешу в Laravel, ми використаємо`extend`метод на`Cache`фасад. Заклик до`Cache::extend`може бути зроблено в`boot`метод за замовчуванням`App\Providers\AppServiceProvider`що постачається зі свіжими програмами Laravel, або ви можете створити власного постачальника послуг для розміщення розширення - просто не забудьте зареєструвати постачальника в`config/app.php`масив постачальника:

    <?php

    namespace App\Providers;

    use App\Extensions\MongoStore;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\Support\ServiceProvider;

    class CacheServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Cache::extend('mongo', function ($app) {
                return Cache::repository(new MongoStore);
            });
        }
    }

Перший аргумент, переданий в`extend`метод - це ім'я драйвера. Це відповідатиме вашому`driver`варіант у`config/cache.php`файл конфігурації. Другий аргумент - це Закриття, яке повинно повернути файл`Illuminate\Cache\Repository`інстанції. Закриття буде прийнято`$app`екземпляр, який є екземпляром[службовий контейнер](/docs/{{version}}/container).

Після реєстрації вашого розширення оновіть`config/cache.php`файли конфігурації`driver`параметр до імені вашого розширення.

<a name="events"></a>

## Події

Щоб виконати код на кожній операції кешування, ви можете прослухати[події](/docs/{{version}}/events)спрацьовує кеш. Як правило, ви повинні розміщувати ці слухачі подій у вашому`EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Cache\Events\CacheHit' => [
            'App\Listeners\LogCacheHit',
        ],

        'Illuminate\Cache\Events\CacheMissed' => [
            'App\Listeners\LogCacheMissed',
        ],

        'Illuminate\Cache\Events\KeyForgotten' => [
            'App\Listeners\LogKeyForgotten',
        ],

        'Illuminate\Cache\Events\KeyWritten' => [
            'App\Listeners\LogKeyWritten',
        ],
    ];
