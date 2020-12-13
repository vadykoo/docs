# Розвідник Ларавела

-   [Вступ](#introduction)
-   [Встановлення](#installation)
    -   [Черги](#queueing)
    -   [Передумови драйвера](#driver-prerequisites)
-   [Конфігурація](#configuration)
    -   [Налаштування індексів моделей](#configuring-model-indexes)
    -   [Configuring Searchable Data](#configuring-searchable-data)
    -   [Налаштування ідентифікатора моделі](#configuring-the-model-id)
    -   [Ідентифікація користувачів](#identifying-users)
-   [Індексація](#indexing)
    -   [Пакетний імпорт](#batch-import)
    -   [Додавання записів](#adding-records)
    -   [Оновлення записів](#updating-records)
    -   [Видалення записів](#removing-records)
    -   [Призупинення індексації](#pausing-indexing)
    -   [Умовно доступні екземпляри моделей](#conditionally-searchable-model-instances)
-   [Пошук](#searching)
    -   [Де речення](#where-clauses)
    -   [Пагінація](#pagination)
    -   [М'яке видалення](#soft-deleting)
    -   [Налаштування пошукових запитів двигуна](#customizing-engine-searches)
-   [Нестандартні двигуни](#custom-engines)
-   [Макроси будівельника](#builder-macros)

<a name="introduction"></a>

## Вступ

Laravel Scout пропонує просте рішення на основі драйверів для додавання повнотекстового пошуку до вашого[Красномовні моделі](/docs/{{version}}/eloquent). Використовуючи модельних спостерігачів, Scout автоматично синхронізує ваші індекси пошуку з вашими Eloquentи записами.

В даний час скаут постачає з[Альголія](https://www.algolia.com/)водій; однак писати власні драйвери просто, і ви можете вільно розширити Scout за допомогою власних реалізацій пошуку.

<a name="installation"></a>

## Встановлення

Спочатку встановіть Scout через менеджер пакетів Composer:

    composer require laravel/scout

Після встановлення Scout вам слід опублікувати конфігурацію Scout за допомогою`vendor:publish`Artisan командування. Ця команда опублікує файл`scout.php`файл конфігурації до вашого`config`каталог:

    php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"

Нарешті, додайте`Laravel\Scout\Searchable`риса моделі, яку ви хочете зробити пошуковою. Ця ознака реєструє спостерігача моделі, щоб підтримувати модель у синхронізації з вашим драйвером пошуку:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class Post extends Model
    {
        use Searchable;
    }

<a name="queueing"></a>

### Черги

Незважаючи на те, що суворо не потрібно використовувати Scout, вам слід настійно розглянути можливість налаштування a[драйвер черги](/docs/{{version}}/queues)перед використанням бібліотеки. Запуск робочого черги дозволить Scout поставити в чергу всі операції, які синхронізують інформацію про вашу модель з вашими пошуковими індексами, забезпечуючи набагато кращий час відгуку для веб-інтерфейсу вашої програми.

Після налаштування драйвера черги встановіть значення`queue`варіант у вашому`config/scout.php`файл конфігурації до`true`:

    'queue' => true,

<a name="driver-prerequisites"></a>

### Передумови драйвера

<a name="algolia"></a>

#### Альголія

Використовуючи драйвер Algolia, слід налаштувати Algolia`id`і`secret`облікові дані у вашому`config/scout.php`файл конфігурації. Після налаштування ваших облікових даних вам також потрібно буде встановити Algolia PHP SDK через менеджер пакетів Composer:

    composer require algolia/algoliasearch-client-php

<a name="configuration"></a>

## Конфігурація

<a name="configuring-model-indexes"></a>

### Налаштування індексів моделей

Кожна модель Eloquent синхронізується із заданим пошуковим "індексом", який містить усі записи, доступні для пошуку для цієї моделі. Іншими словами, ви можете уявити кожен індекс як таблицю MySQL. За замовчуванням кожна модель зберігатиметься в індексі, що відповідає типовому імені моделі "таблиці". Як правило, це форма множини назви моделі; однак ви можете налаштувати індекс моделі, замінивши`searchableAs`метод на моделі:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class Post extends Model
    {
        use Searchable;

        /**
         * Get the index name for the model.
         *
         * @return string
         */
        public function searchableAs()
        {
            return 'posts_index';
        }
    }

<a name="configuring-searchable-data"></a>

### Налаштування даних для пошуку

За замовчуванням весь`toArray`форма даної моделі буде збережена до її пошукового індексу. Якщо ви хочете налаштувати дані, які синхронізуються з пошуковим індексом, ви можете замінити`toSearchableArray`метод на моделі:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class Post extends Model
    {
        use Searchable;

        /**
         * Get the indexable data array for the model.
         *
         * @return array
         */
        public function toSearchableArray()
        {
            $array = $this->toArray();

            // Customize array...

            return $array;
        }
    }

<a name="configuring-the-model-id"></a>

### Налаштування ідентифікатора моделі

За замовчуванням Scout використовуватиме первинний ключ моделі як унікальний ідентифікатор, що зберігається в індексі пошуку. Якщо вам потрібно налаштувати цю поведінку, ви можете замінити`getScoutKey`та`getScoutKeyName`методи на моделі:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class User extends Model
    {
        use Searchable;

        /**
         * Get the value used to index the model.
         *
         * @return mixed
         */
        public function getScoutKey()
        {
            return $this->email;
        }

        /**
         * Get the key name used to index the model.
         *
         * @return mixed
         */
        public function getScoutKeyName()
        {
            return 'email';
        }
    }

<a name="identifying-users"></a>

### Ідентифікація користувачів

Scout також дозволяє автоматично ідентифікувати користувачів під час використання Algolia. Пов’язання автентифікованого користувача з пошуковими операціями може бути корисним при перегляді вашої аналітики пошуку на інформаційній панелі Algolia. Ви можете ввімкнути ідентифікацію користувача, встановивши`SCOUT_IDENTIFY`до`true`у вашому`.env`файл:

    SCOUT_IDENTIFY=true

Увімкнувши цю функцію, це також передасть IP-адресу запиту та основний ідентифікатор вашого аутентифікованого користувача в Алголію, тому ці дані асоціюються з будь-яким запитом на пошук, який робить користувач.

<a name="indexing"></a>

## Індексація

<a name="batch-import"></a>

### Пакетний імпорт

Якщо ви встановлюєте Scout у існуючий проект, можливо, ви вже маєте записи бази даних, які потрібно імпортувати у ваш драйвер пошуку. Розвідник надає`import`Команда Artisan, яку ви можете використовувати для імпорту всіх наявних записів у пошукові індекси:

    php artisan scout:import "App\Models\Post"

`flush` command may be used to remove all of a model's records from your search indexes:

    php artisan scout:flush "App\Models\Post"

<a name="modifying-the-import-query"></a>

#### Змінення запиту на імпорт

Якщо ви хочете змінити запит, який використовується для отримання всіх ваших моделей для пакетного імпорту, ви можете визначити`makeAllSearchableUsing`метод на вашій моделі. Це чудове місце, щоб додати будь-яке нетерпляче завантаження відносин, яке може знадобитися перед імпортом ваших моделей:

    /**
     * Modify the query used to retrieve models when making all of the models searchable.
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $query
     * @return \Illuminate\Database\Eloquent\Builder
     */
    protected function makeAllSearchableUsing($query)
    {
        return $query->with('author');
    }

<a name="adding-records"></a>

### Додавання записів

Після додавання`Laravel\Scout\Searchable`риси моделі, все, що вам потрібно зробити, це`save`примірник моделі, і він буде автоматично доданий до вашого пошукового індексу. Якщо ви налаштували Scout на[використовувати черги](#queueing)ця операція буде виконана у фоновому режимі вашим працівником черги:

    $order = new App\Models\Order;

    // ...

    $order->save();

<a name="adding-via-query"></a>

#### Додавання за допомогою запиту

Якщо ви хочете додати колекцію моделей до вашого індексу пошуку за допомогою красномовного запиту, ви можете встановити ланцюжок`searchable`метод на Eloquent запит.`searchable`метод буде[шматок результатів](/docs/{{version}}/eloquent#chunking-results)запиту та додайте записи до вашого пошукового індексу. Знову ж таки, якщо ви налаштували Scout на використання черг, усі фрагменти будуть додані у фоновому режимі вашими працівниками черги:

    // Adding via Eloquent query...
    App\Models\Order::where('price', '>', 100)->searchable();

    // You may also add records via relationships...
    $user->orders()->searchable();

    // You may also add records via collections...
    $orders->searchable();

`searchable`метод можна вважати операцією "підняття". Іншими словами, якщо запис моделі вже є у вашому індексі, він буде оновлений. Якщо він не існує в індексі пошуку, він буде доданий до індексу.

<a name="updating-records"></a>

### Оновлення записів

Щоб оновити модель, яку можна шукати, потрібно лише оновити властивості екземпляра моделі та`save`модель до вашої бази даних. Scout автоматично збереже зміни у вашому індексі пошуку:

    $order = App\Models\Order::find(1);

    // Update the order...

    $order->save();

Ви також можете використовувати`searchable`за запитом Eloquent для оновлення колекції моделей. Якщо моделі не існують у вашому індексі пошуку, вони будуть створені:

    // Updating via Eloquent query...
    App\Models\Order::where('price', '>', 100)->searchable();

    // You may also update via relationships...
    $user->orders()->searchable();

    // You may also update via collections...
    $orders->searchable();

<a name="removing-records"></a>

### Видалення записів

Щоб видалити запис із вашого індексу,`delete`модель з бази даних. Ця форма видалення навіть сумісна з[м'який видалений](/docs/{{version}}/eloquent#soft-deleting)моделі:

    $order = App\Models\Order::find(1);

    $order->delete();

Якщо ви не хочете отримувати модель перед видаленням запису, ви можете використовувати`unsearchable`метод на екземплярі або колекції запиту Eloquent:

    // Removing via Eloquent query...
    App\Models\Order::where('price', '>', 100)->unsearchable();

    // You may also remove via relationships...
    $user->orders()->unsearchable();

    // You may also remove via collections...
    $orders->unsearchable();

<a name="pausing-indexing"></a>

### Призупинення індексації

Іноді вам може знадобитися виконати пакет Eloquent операцій над моделлю без синхронізації даних моделі з вашим пошуковим індексом. Ви можете зробити це за допомогою`withoutSyncingToSearch`метод. Цей метод приймає один зворотний виклик, який буде негайно виконаний. Будь-які операції з моделлю, які відбуваються в рамках зворотного виклику, не синхронізуються з індексом моделі:

    App\Models\Order::withoutSyncingToSearch(function () {
        // Perform model actions...
    });

<a name="conditionally-searchable-model-instances"></a>

### Умовно доступні екземпляри моделей

Іноді вам може знадобитися зробити модель доступною для пошуку лише за певних умов. Наприклад, уявіть, що у вас є`App\Models\Post`модель, яка може бути в одному з двох станів: "проект" та "опубліковано". Ви можете дозволити лише пошук у "опублікованих" публікаціях. Для цього ви можете визначити a`shouldBeSearchable`метод на вашій моделі:

    public function shouldBeSearchable()
    {
        return $this->isPublished();
    }

`shouldBeSearchable`метод застосовується лише при маніпулюванні моделями через`save`метод, запити або взаємозв'язки. Безпосереднє створення моделей або колекцій для пошуку за допомогою`searchable`метод замінить результат`shouldBeSearchable`метод:

    // Will respect "shouldBeSearchable"...
    App\Models\Order::where('price', '>', 100)->searchable();

    $user->orders()->searchable();

    $order->save();

    // Will override "shouldBeSearchable"...
    $orders->searchable();

    $order->searchable();

<a name="searching"></a>

## Searching

Ви можете розпочати пошук моделі за допомогою`search`метод. Метод пошуку приймає єдиний рядок, який буде використовуватися для пошуку ваших моделей. Потім слід ланцюгом`get`на пошуковий запит для отримання Eloquent моделей, які відповідають даному пошуковому запиту:

    $orders = App\Models\Order::search('Star Trek')->get();

Оскільки скаутські пошуки повертають колекцію Eloquent моделей, ви можете навіть повернути результати безпосередньо з маршруту або контролера, і вони будуть автоматично перетворені в JSON:

    use Illuminate\Http\Request;

    Route::get('/search', function (Request $request) {
        return App\Models\Order::search($request->search)->get();
    });

Якщо ви хочете отримати вихідні результати, перш ніж вони будуть перетворені на красномовні моделі, вам слід скористатися`raw`метод:

    $orders = App\Models\Order::search('Star Trek')->raw();

Пошукові запити, як правило, виконуються за індексом, визначеним моделлю[`searchableAs`](#configuring-model-indexes)метод. Однак ви можете використовувати`within`спосіб вказати власний індекс, який слід замість цього шукати:

    $orders = App\Models\Order::search('Star Trek')
        ->within('tv_shows_popularity_desc')
        ->get();

<a name="where-clauses"></a>

### Де речення

Scout дозволяє додавати прості речення "де" до ваших пошукових запитів. Наразі ці пункти підтримують лише основні перевірки числової рівності і в першу чергу корисні для обсягу пошукових запитів за ідентифікатором орендаря. Оскільки пошуковий індекс не є реляційною базою даних, більш досконалі речення "де" наразі не підтримуються:

    $orders = App\Models\Order::search('Star Trek')->where('user_id', 1)->get();

<a name="pagination"></a>

### Пагінація

Окрім отримання колекції моделей, ви можете перенести результати пошуку за допомогою`paginate`метод. Цей метод поверне a`Paginator`екземпляр так само, як якщо б у вас був[переклав сторінку на традиційний Eloquent запит](/docs/{{version}}/pagination):

    $orders = App\Models\Order::search('Star Trek')->paginate();

Ви можете вказати, скільки моделей отримувати на кожній сторінці, передавши суму як перший аргумент в`paginate`метод:

    $orders = App\Models\Order::search('Star Trek')->paginate(15);

Отримавши результати, ви можете відобразити результати та відтворити посилання на сторінки за допомогою[Blade](/docs/{{version}}/blade)так само, як ніби ви розбили сторінку традиційного красномовного запиту:

    <div class="container">
        @foreach ($orders as $order)
            {{ $order->price }}
        @endforeach
    </div>

    {{ $orders->links() }}

<a name="soft-deleting"></a>

### М'яке видалення

Якщо ваші проіндексовані моделі є[м'яке видалення](/docs/{{version}}/eloquent#soft-deleting)і вам потрібно здійснити пошук ваших м'яких видалених моделей, встановити`soft_delete`варіант`config/scout.php`файл конфігурації до`true`:

    'soft_delete' => true,

Коли цей параметр конфігурації є`true`, Scout не видалятиме м’яко видалені моделі з індексу пошуку. Натомість він встановить прихований`__soft_deleted`атрибут на індексованому записі. Потім ви можете використовувати`withTrashed`або`onlyTrashed`методи отримання м'яко видалених записів під час пошуку:

    // Include trashed records when retrieving results...
    $orders = App\Models\Order::search('Star Trek')->withTrashed()->get();

    // Only include trashed records when retrieving results...
    $orders = App\Models\Order::search('Star Trek')->onlyTrashed()->get();

> {tip} Коли м’яко видалена модель назавжди видаляється за допомогою`forceDelete`, Scout автоматично видалить його з індексу пошуку.

<a name="customizing-engine-searches"></a>

### Налаштування пошукових запитів двигуна

Якщо вам потрібно налаштувати поведінку пошуку механізму, ви можете передати зворотний виклик як другий аргумент для`search`метод. Наприклад, ви можете використати цей зворотний дзвінок, щоб додати дані про геолокацію до параметрів пошуку перед тим, як пошуковий запит буде переданий до Алголії:

    use Algolia\AlgoliaSearch\SearchIndex;

    App\Models\Order::search('Star Trek', function (SearchIndex $algolia, string $query, array $options) {
        $options['body']['query']['bool']['filter']['geo_distance'] = [
            'distance' => '1000km',
            'location' => ['lat' => 36, 'lon' => 111],
        ];

        return $algolia->search($query, $options);
    })->get();

<a name="custom-engines"></a>

## Нестандартні двигуни

<a name="writing-the-engine"></a>

#### Написання двигуна

Якщо одна з вбудованих пошукових систем Scout не відповідає вашим потребам, ви можете написати власний власний механізм і зареєструвати його у Scout. Ваш двигун повинен розширити`Laravel\Scout\Engines\Engine`абстрактний клас. Цей абстрактний клас містить вісім методів, які повинен реалізувати ваш користувальницький движок:

    use Laravel\Scout\Builder;

    abstract public function update($models);
    abstract public function delete($models);
    abstract public function search(Builder $builder);
    abstract public function paginate(Builder $builder, $perPage, $page);
    abstract public function mapIds($results);
    abstract public function map(Builder $builder, $results, $model);
    abstract public function getTotalCount($results);
    abstract public function flush($model);

Можливо, вам буде корисно переглянути реалізацію цих методів на`Laravel\Scout\Engines\AlgoliaEngine`клас. Цей клас забезпечить вам хорошу початкову точку для вивчення того, як впровадити кожен із цих методів у власному движку.

<a name="registering-the-engine"></a>

#### Реєстрація двигуна

Після того, як ви написали свій власний движок, ви можете зареєструвати його у Scout за допомогою`extend`метод менеджера двигуна Scout. Вам слід зателефонувати до`extend`метод з`boot`метод вашого`AppServiceProvider`або будь-який інший постачальник послуг, що використовується вашою програмою. Наприклад, якщо ви написали`MySqlSearchEngine`, Ви можете зареєструвати це так:

    use Laravel\Scout\EngineManager;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        resolve(EngineManager::class)->extend('mysql', function () {
            return new MySqlSearchEngine;
        });
    }

Після реєстрації вашого двигуна ви можете вказати його як свого розвідника за замовчуванням`driver`у вашому`config/scout.php`файл конфігурації:

    'driver' => 'mysql',

<a name="builder-macros"></a>

## Макроси будівельника

Якщо ви хочете визначити власний метод конструктора, ви можете використовувати`macro`метод на`Laravel\Scout\Builder`клас. Як правило, "макроси" слід визначати в межах[постачальника послуг](/docs/{{version}}/providers)`boot`метод:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Response;
    use Illuminate\Support\ServiceProvider;
    use Laravel\Scout\Builder;

    class ScoutMacroServiceProvider extends ServiceProvider
    {
        /**
         * Register the application's scout macros.
         *
         * @return void
         */
        public function boot()
        {
            Builder::macro('count', function () {
                return $this->engine->getTotalCount(
                    $this->engine()->search($this)
                );
            });
        }
    }

`macro`Функція приймає ім'я як перший аргумент, а Закриття як другий аргумент. Закриття макросу буде виконано при виклику імені макроса з`Laravel\Scout\Builder`реалізація:

    App\Models\Order::search('Star Trek')->count();
