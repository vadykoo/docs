# Eloquent: Початок роботи

-   [Вступ](#introduction)
-   [Визначення моделей](#defining-models)
    -   [Eloquent модельні конвенції](#eloquent-model-conventions)
    -   [Значення атрибута за замовчуванням](#default-attribute-values)
-   [Отримання моделей](#retrieving-models)
    -   [Колекції](#collections)
    -   [Результати Chunking(Chunking)](#chunking-results)
    -   [Розширені підзапити](#advanced-subqueries)
-   [Отримання одиночних моделей / агрегатів](#retrieving-single-models)
    -   [Отримання агрегатів](#retrieving-aggregates)
-   [Вставка та оновлення моделей](#inserting-and-updating-models)
    -   [Вставки](#inserts)
    -   [Оновлення](#updates)
    -   [Масове призначення](#mass-assignment)
    -   [Інші методи створення](#other-creation-methods)
-   [Видалення моделей](#deleting-models)
    -   [М'яке(Soft) видалення](#soft-deleting)
    -   [Запит на Soft-видалені Моделі](#querying-soft-deleted-models)
-   [Реплікаційні моделі](#replicating-models)
-   [Scopes запитів](#query-scopes)
    -   [Глобальні Scopes](#global-scopes)
    -   [Місцеві Scopes застосування](#local-scopes)
-   [Порівняння моделей](#comparing-models)
-   [Події](#events)
    -   [Використання замикань](#events-using-closures)
    -   [Observers(Спостерігачі)](#observers)
    -   [Muting подій](#muting-events)

<a name="introduction"></a>

## Вступ

Eloquent ORM, що входить до складу Laravel, забезпечує чудову, просту реалізацію ActiveRecord для роботи з вашою базою даних. Кожна таблиця бази даних має відповідну "Модель", яка використовується для взаємодії з цією таблицею. Моделі дозволяють запитувати дані у ваших таблицях, а також вставляти нові записи до таблиці.

Перед початком роботи обов’язково налаштуйте підключення до бази даних у`config/database.php`. Щоб отримати додаткову інформацію про налаштування бази даних, ознайомтесь із цим[документація](/docs/{{version}}/database#configuration).

<a name="defining-models"></a>

## Визначення моделей

Для початку давайте створимо Eloquent модель. Моделі зазвичай живуть в`app\Models`каталог, але ви можете вільно розміщувати їх у будь-якому місці, яке можна автоматично завантажити відповідно до вашого`composer.json`файл. Всі моделі Eloquent розширюються`Illuminate\Database\Eloquent\Model`клас.

Найпростіший спосіб створити екземпляр моделі - використовувати`make:model`[artisan командування](/docs/{{version}}/artisan):

    php artisan make:model Flight

Якщо ви хочете створити файл[міграція баз даних](/docs/{{version}}/migrations)коли ви генеруєте модель, ви можете використовувати`--migration`або`-m`варіант:

    php artisan make:model Flight --migration

    php artisan make:model Flight -m

Ви можете генерувати різні інші типи класів під час створення моделі, такі як Factory, сівалки та контролери. Крім того, ці параметри можна комбінувати для створення декількох класів одночасно:

    php artisan make:model Flight --factory
    php artisan make:model Flight -f

    php artisan make:model Flight --seed
    php artisan make:model Flight -s

    php artisan make:model Flight --controller
    php artisan make:model Flight -c

    php artisan make:model Flight -mfsc

<a name="eloquent-model-conventions"></a>

### Eloquent модельні конвенції

Тепер давайте розглянемо приклад`Flight`модель, яку ми будемо використовувати для отримання та збереження інформації з нашого`flights`таблиця бази даних:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        //
    }

<a name="table-names"></a>

#### Назви таблиць

Зверніть увагу, що ми не говорили Eloquent, яку таблицю використовувати для нашої`Flight`модель. За домовленістю, "змієний випадок", множинне ім'я класу буде використовуватися як ім'я таблиці, якщо інше ім'я не вказано явно. Отже, у цьому випадку Eloquent припустить`Flight`модель зберігає записи в`flights`таблиці, тоді як an`AirTrafficController`модель буде зберігати записи в`air_traffic_controllers`таблиця.

Ви можете вручну вказати назву таблиці, визначивши a`table`властивість на вашій моделі:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The table associated with the model.
         *
         * @var string
         */
        protected $table = 'my_flights';
    }

<a name="primary-keys"></a>

#### Первинні ключі

Eloquent також припустить, що кожна таблиця має стовпець первинного ключа з іменем`id`. Ви можете визначити захищений`$primaryKey`властивість замінити цю конвенцію:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The primary key associated with the table.
         *
         * @var string
         */
        protected $primaryKey = 'flight_id';
    }

Крім того, Eloquent припускає, що первинний ключ - це ціле число, що збільшується, а це означає, що за замовчуванням первинний ключ буде автоматично переданий до`int`. Якщо ви хочете використовувати первинний ключ, що не збільшується або не числовий, ви повинні встановити загальнодоступний`$incrementing`властивість на вашій моделі до`false`:

    <?php

    class Flight extends Model
    {
        /**
         * Indicates if the IDs are auto-incrementing.
         *
         * @var bool
         */
        public $incrementing = false;
    }

Якщо ваш первинний ключ не є цілим числом, вам слід встановити захищений`$keyType`властивість на вашій моделі до`string`:

    <?php

    class Flight extends Model
    {
        /**
         * The "type" of the auto-incrementing ID.
         *
         * @var string
         */
        protected $keyType = 'string';
    }

<a name="timestamps"></a>

#### Мітки часу

За замовчуванням Eloquent очікує`created_at`і`updated_at`стовпців, які існують у ваших таблицях. Якщо ви не хочете, щоб ці стовпці автоматично керувалися Eloquent, встановіть`$timestamps`властивість на вашій моделі до`false`:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Indicates if the model should be timestamped.
         *
         * @var bool
         */
        public $timestamps = false;
    }

Якщо вам потрібно налаштувати формат міток часу, встановіть`$dateFormat`властивість на вашій моделі. Ця властивість визначає, як атрибути дати зберігаються в базі даних, а також їх формат, коли модель серіалізована до масиву або JSON:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The storage format of the model's date columns.
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

Якщо вам потрібно налаштувати імена стовпців, що використовуються для зберігання міток часу, ви можете встановити`CREATED_AT`і`UPDATED_AT`константи у вашій моделі:

    <?php

    class Flight extends Model
    {
        const CREATED_AT = 'creation_date';
        const UPDATED_AT = 'last_update';
    }

<a name="database-connection"></a>

#### Підключення до бази даних

За замовчуванням усі моделі Eloquent будуть використовувати підключення до бази даних за замовчуванням, налаштоване для вашої програми. Якщо ви хочете вказати інше з'єднання для моделі, використовуйте`$connection`властивість:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The connection name for the model.
         *
         * @var string
         */
        protected $connection = 'connection-name';
    }

<a name="default-attribute-values"></a>

### Значення атрибута за замовчуванням

Якщо ви хочете визначити значення за замовчуванням для деяких атрибутів вашої моделі, ви можете визначити`$attributes`властивість на вашій моделі:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The model's default values for attributes.
         *
         * @var array
         */
        protected $attributes = [
            'delayed' => false,
        ];
    }

<a name="retrieving-models"></a>

## Отримання моделей

Після того, як ви створили модель і[відповідну таблицю бази даних](/docs/{{version}}/migrations#writing-migrations), ви готові розпочати отримання даних із вашої бази даних. Подумайте про кожну Eloquent модель як про потужну[конструктор запитів](/docs/{{version}}/queries)що дозволяє вільно запитувати таблицю бази даних, пов’язану з моделлю. Наприклад:

    <?php

    $flights = App\Models\Flight::all();

    foreach ($flights as $flight) {
        echo $flight->name;
    }

<a name="adding-additional-constraints"></a>

#### Додавання додаткових обмежень

Eloquent`all`метод поверне всі результати в таблиці моделі. Оскільки кожна Eloquent модель виконує функції[конструктор запитів](/docs/{{version}}/queries), Ви також можете додати обмеження до запитів, а потім використовувати`get`спосіб отримання результатів:

    $flights = App\Models\Flight::where('active', 1)
                   ->orderBy('name', 'desc')
                   ->take(10)
                   ->get();

> {tip} Оскільки Eloquent моделі створюють запити, вам слід переглянути всі методи, доступні в[конструктор запитів](/docs/{{version}}/queries). Ви можете використовувати будь-який із цих методів у своїх Eloquent запитах.

<a name="refreshing-models"></a>

#### Освіжаючі моделі

Ви можете оновити моделі за допомогою`fresh`і`refresh`методи.`fresh`метод знову отримає модель із бази даних. На існуючий екземпляр моделі це не вплине:

    $flight = App\Models\Flight::where('number', 'FR 900')->first();

    $freshFlight = $flight->fresh();

`refresh`метод повторно гідратує існуючу модель, використовуючи свіжі дані з бази даних. Крім того, оновлять також усі його завантажені зв'язки:

    $flight = App\Models\Flight::where('number', 'FR 900')->first();

    $flight->number = 'FR 456';

    $flight->refresh();

    $flight->number; // "FR 900"

<a name="collections"></a>

### Колекції

Для Eloquent методів типу`all`і`get`які отримують кілька результатів, екземпляр`Illuminate\Database\Eloquent\Collection`буде повернено.`Collection`клас забезпечує[різноманітні корисні методи](/docs/{{version}}/eloquent-collections#available-methods)для роботи з вашими Eloquent результатами:

    $flights = $flights->reject(function ($flight) {
        return $flight->cancelled;
    });

Ви також можете прокрутити колекцію як масив:

    foreach ($flights as $flight) {
        echo $flight->name;
    }

<a name="chunking-results"></a>

### Результати Chunking

Якщо вам потрібно обробити тисячі Eloquent записів, використовуйте`chunk`команди.`chunk`метод отримає "шматок" Eloquent моделей, подаючи їх заданому`Closure`для переробки. Використання`chunk`метод збереже пам'ять при роботі з великими наборами результатів:

    Flight::chunk(200, function ($flights) {
        foreach ($flights as $flight) {
            //
        }
    });

Першим аргументом, що передається методу, є кількість записів, які ви хочете отримати за "шматок". Закриття, передане як другий аргумент, буде викликатися для кожного фрагмента, який отримується з бази даних. Буде виконано запит до бази даних, щоб отримати кожен шматок записів, переданих до Закриття.

Якщо ви фільтруєте результати`chunk`метод, заснований на стовпці, який ви також будете оновлювати, переглядаючи результати, вам слід використовувати`chunkById`метод. Використання`chunk`метод у цих сценаріях може призвести до несподіваних і суперечливих результатів:

    Flight::where('departed', true)->chunkById(200, function ($flights) {
        $flights->each->update(['departed' => false]);
    });

<a name="using-cursors"></a>

#### Використання курсорів

`cursor`Метод дозволяє переглядати записи бази даних за допомогою курсору, який виконуватиме лише один запит. При обробці великих обсягів даних файл`cursor`метод може бути використаний для значного зменшення використання пам'яті:

    foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
        //
    }

`cursor`повертає`Illuminate\Support\LazyCollection`екземпляр.[Lazy колекції](/docs/{{version}}/collections#lazy-collections)дозволяють використовувати багато методів збирання, доступних у типових колекціях Laravel, одночасно завантажуючи в пам’ять лише одну модель:

    $users = App\Models\User::cursor()->filter(function ($user) {
        return $user->id > 500;
    });

    foreach ($users as $user) {
        echo $user->id;
    }

<a name="advanced-subqueries"></a>

### Розширені підзапити

<a name="subquery-selects"></a>

#### Вибір підзапиту

Eloquent також пропонує розширену підтримку підзапитів, яка дозволяє витягувати інформацію з відповідних таблиць в одному запиті. Наприклад, уявімо, що у нас є таблиця польоту`destinations`і таблиця`flights`до пунктів призначення.`flights`таблиця містить`arrived_at`графа, яка вказує, коли рейс прибув у пункт призначення.

Використовуючи функціональність підзапиту, доступну для`select`і`addSelect`методи, ми можемо вибрати всі`destinations`і назва рейсу, який нещодавно прибув у цей пункт призначення за допомогою одного запиту:

    use App\Models\Destination;
    use App\Models\Flight;

    return Destination::addSelect(['last_flight' => Flight::select('name')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderBy('arrived_at', 'desc')
        ->limit(1)
    ])->get();

<a name="subquery-ordering"></a>

#### Впорядкування підзапитів

Крім того, запит будівельника`orderBy`функція підтримує підзапити. Ми можемо використовувати цю функцію для сортування всіх пунктів призначення на основі того, коли останній рейс прибув у цей пункт призначення. Знову ж таки, це можна зробити під час виконання одного запиту до бази даних:

    return Destination::orderByDesc(
        Flight::select('arrived_at')
            ->whereColumn('destination_id', 'destinations.id')
            ->orderBy('arrived_at', 'desc')
            ->limit(1)
    )->get();

<a name="retrieving-single-models"></a>

## Отримання одиночних моделей / агрегатів

На додаток до отримання всіх записів для даної таблиці, ви також можете отримати окремі записи за допомогою`find`,`first`, або`firstWhere`. Замість повернення колекції моделей ці методи повертають один екземпляр моделі:

    // Retrieve a model by its primary key...
    $flight = App\Models\Flight::find(1);

    // Retrieve the first model matching the query constraints...
    $flight = App\Models\Flight::where('active', 1)->first();

    // Shorthand for retrieving the first model matching the query constraints...
    $flight = App\Models\Flight::firstWhere('active', 1);

Ви також можете зателефонувати до`find`метод з масивом первинних ключів, який поверне колекцію відповідних записів:

    $flights = App\Models\Flight::find([1, 2, 3]);

Іноді вам може знадобитися отримати перший результат запиту або виконати якусь іншу дію, якщо результатів не знайдено.`firstOr`метод поверне перший знайдений результат або, якщо результатів не знайдено, виконає заданий зворотний виклик. Результат зворотного виклику буде вважатися результатом`firstOr`метод:

    $model = App\Models\Flight::where('legs', '>', 100)->firstOr(function () {
            // ...
    });

`firstOr`метод також приймає масив стовпців для отримання:

    $model = App\Models\Flight::where('legs', '>', 100)
                ->firstOr(['id', 'legs'], function () {
                    // ...
                });

<a name="not-found-exceptions"></a>

#### Не знайдено винятків

Іноді, можливо, ви захочете створити виняток, якщо модель не знайдено. Це особливо корисно в маршрутах або контролерах.`findOrFail`і`firstOrFail`методи отримують перший результат запиту; однак, якщо результату не знайдено, символ`Illuminate\Database\Eloquent\ModelNotFoundException`буде кинуто:

    $model = App\Models\Flight::findOrFail(1);

    $model = App\Models\Flight::where('legs', '>', 100)->firstOrFail();

Якщо виняток не вловлюється, a`404`Відповідь HTTP автоматично надсилається назад користувачеві. Для повернення не потрібно писати явні чеки`404`відповіді при використанні цих методів:

    Route::get('/api/flights/{id}', function ($id) {
        return App\Models\Flight::findOrFail($id);
    });

<a name="retrieving-aggregates"></a>

### Отримання агрегатів

Ви також можете використовувати`count`,`sum`,`max`, і інші[агреговані методи](/docs/{{version}}/queries#aggregates)надані[конструктор запитів](/docs/{{version}}/queries). Ці методи повертають відповідне скалярне значення замість повного екземпляра моделі:

    $count = App\Models\Flight::where('active', 1)->count();

    $max = App\Models\Flight::where('active', 1)->max('price');

<a name="inserting-and-updating-models"></a>

## Вставка та оновлення моделей

<a name="inserts"></a>

### Вставки

Щоб створити новий запис у базі даних, створіть новий екземпляр моделі, встановіть атрибути на моделі, а потім викликайте`save`метод:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\Flight;
    use Illuminate\Http\Request;

    class FlightController extends Controller
    {
        /**
         * Create a new flight instance.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Validate the request...

            $flight = new Flight;

            $flight->name = $request->name;

            $flight->save();
        }
    }

У цьому прикладі ми призначаємо`name`з вхідного HTTP-запиту на`name`атрибут`App\Models\Flight`екземпляр моделі. Коли ми телефонуємо до`save`метод, запис буде вставлений у базу даних.`created_at`і`updated_at`позначки часу будуть автоматично встановлені, коли`save`метод, тому немає необхідності встановлювати їх вручну.

<a name="updates"></a>

### Оновлення

`save`метод також може бути використаний для оновлення моделей, які вже існують у базі даних. Щоб оновити модель, вам слід отримати її, встановити будь-які атрибути, які ви хочете оновити, а потім зателефонувати до`save`метод. Знову ж таки`updated_at`позначка часу буде автоматично оновлена, тому немає необхідності встановлювати її значення вручну:

    $flight = App\Models\Flight::find(1);

    $flight->name = 'New Flight Name';

    $flight->save();

<a name="mass-updates"></a>

#### Масові оновлення

Оновлення також можна виконати для будь-якої кількості моделей, які відповідають даному запиту. У цьому прикладі всі рейси, які є`active`і мати`destination`з`San Diego`буде позначено як відкладене:

    App\Models\Flight::where('active', 1)
              ->where('destination', 'San Diego')
              ->update(['delayed' => 1]);

`update`метод очікує масив пар стовпців і значень, що представляють стовпці, які слід оновити.

> {note} Під час масового оновлення через Eloquent,`saving`,`saved`,`updating`, і`updated`події моделей не запускатимуться для оновлених моделей. Це тому, що моделі фактично ніколи не отримуються під час масового оновлення.

<a name="examining-attribute-changes"></a>

#### Вивчення змін атрибутів

Eloquent забезпечує`isDirty`,`isClean`, і`wasChanged`методи для вивчення внутрішнього стану вашої моделі та визначення того, як змінилися її атрибути в порівнянні з початковим завантаженням.

`isDirty`Метод визначає, чи були змінені будь-які атрибути після завантаження моделі. Ви можете передати конкретну назву атрибута, щоб визначити, чи певний атрибут забруднений.`isClean`метод протилежний`isDirty`а також приймає необов'язковий аргумент атрибута:

    $user = User::create([
        'first_name' => 'Taylor',
        'last_name' => 'Otwell',
        'title' => 'Developer',
    ]);

    $user->title = 'Painter';

    $user->isDirty(); // true
    $user->isDirty('title'); // true
    $user->isDirty('first_name'); // false

    $user->isClean(); // false
    $user->isClean('title'); // false
    $user->isClean('first_name'); // true

    $user->save();

    $user->isDirty(); // false
    $user->isClean(); // true

`wasChanged`Метод визначає, чи були змінені будь-які атрибути, коли модель востаннє зберігалась у межах поточного циклу запиту. Ви також можете передати ім'я атрибута, щоб побачити, чи було змінено певний атрибут:

    $user = User::create([
        'first_name' => 'Taylor',
        'last_name' => 'Otwell',
        'title' => 'Developer',
    ]);

    $user->title = 'Painter';
    $user->save();

    $user->wasChanged(); // true
    $user->wasChanged('title'); // true
    $user->wasChanged('first_name'); // false

`getOriginal`Метод повертає масив, що містить вихідні атрибути моделі, незалежно від будь-яких змін після завантаження моделі. Ви можете передати конкретну назву атрибута, щоб отримати вихідне значення певного атрибута:

    $user = User::find(1);

    $user->name; // John
    $user->email; // john@example.com

    $user->name = "Jack";
    $user->name; // Jack

    $user->getOriginal('name'); // John
    $user->getOriginal(); // Array of original attributes...

<a name="mass-assignment"></a>

### Масове призначення

Ви також можете використовувати`create`метод збереження нової моделі в один рядок. Вставлений примірник моделі буде повернено вам із методу. Однак перед цим вам потрібно буде вказати або`fillable`або`guarded`атрибут на моделі, оскільки всі моделі Eloquent за замовчуванням захищають від масового призначення.

Уразливість масового призначення виникає, коли користувач передає несподіваний параметр HTTP через запит, і цей параметр змінює стовпець у вашій базі даних, якого ви не очікували. Наприклад, зловмисний користувач може надіслати файл`is_admin`параметру через HTTP-запит, який потім передається у вашу модель`create`метод, що дозволяє користувачеві перейти до адміністратора.

Отже, для початку слід визначити, які атрибути моделі ви хочете зробити масовим присвоюваним. Ви можете зробити це за допомогою`$fillable`властивість на моделі. Наприклад, давайте зробимо`name`атрибут нашого`Flight`модель, яка може призначатися:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The attributes that are mass assignable.
         *
         * @var array
         */
        protected $fillable = ['name'];
    }

Після того, як ми зробили атрибути масовими, їх можна призначити, ми можемо використовувати`create`метод для вставки нового запису в базу даних.`create`метод повертає збережений примірник моделі:

    $flight = App\Models\Flight::create(['name' => 'Flight 10']);

Якщо у вас вже є екземпляр моделі, ви можете використовувати`fill`метод, щоб заповнити його масивом атрибутів:

    $flight->fill(['name' => 'Flight 22']);

<a name="mass-assignment-json-columns"></a>

#### Масове призначення та стовпці JSON

При призначенні стовпців JSON у коді вашої моделі повинен бути вказаний ключ, що призначається масі кожного стовпця`$fillable`масив. З міркувань безпеки Laravel не підтримує оновлення вкладених атрибутів JSON під час використання`guarded`властивість:

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    $fillable = [
        'options->enabled',
    ];

<a name="allowing-mass-assignment"></a>

#### Дозвіл масового призначення

Якщо ви хочете зробити всі атрибути масовими, можна визначити,`$guarded`властивість як порожній масив:

    /**
     * The attributes that aren't mass assignable.
     *
     * @var array
     */
    protected $guarded = [];

<a name="other-creation-methods"></a>

### Інші методи створення

<a name="firstorcreate-firstornew"></a>

#### `firstOrCreate`/`firstOrNew`

Існує ще два методи, які ви можете використовувати для створення моделей шляхом масового присвоєння атрибутів:`firstOrCreate`і`firstOrNew`.`firstOrCreate`метод спробує знайти запис бази даних за допомогою заданих пар стовпець / значення. Якщо модель не вдається знайти в базі даних, буде вставлено запис із атрибутами з першого параметра, а також із необов’язковим другим параметром.

`firstOrNew`метод, як`firstOrCreate`спробує знайти запис у базі даних, що відповідає заданим атрибутам. Однак, якщо модель не знайдено, буде повернено новий екземпляр моделі. Зверніть увагу, що модель повернута`firstOrNew`ще не збережено в базі даних. Вам потрібно буде зателефонувати`save`вручну, щоб зберегти його:

    // Retrieve flight by name, or create it if it doesn't exist...
    $flight = App\Models\Flight::firstOrCreate(['name' => 'Flight 10']);

    // Retrieve flight by name, or create it with the name, delayed, and arrival_time attributes...
    $flight = App\Models\Flight::firstOrCreate(
        ['name' => 'Flight 10'],
        ['delayed' => 1, 'arrival_time' => '11:30']
    );

    // Retrieve by name, or instantiate...
    $flight = App\Models\Flight::firstOrNew(['name' => 'Flight 10']);

    // Retrieve by name, or instantiate with the name, delayed, and arrival_time attributes...
    $flight = App\Models\Flight::firstOrNew(
        ['name' => 'Flight 10'],
        ['delayed' => 1, 'arrival_time' => '11:30']
    );

<a name="updateorcreate"></a>

#### `updateOrCreate`

Ви також можете зіткнутися з ситуаціями, коли ви хочете оновити існуючу модель або створити нову модель, якщо такої немає. Laravel пропонує`updateOrCreate`метод зробити це за один крок. Подобається`firstOrCreate`метод,`updateOrCreate`модель зберігається, тому дзвонити не потрібно`save()`:

    // If there's a flight from Oakland to San Diego, set the price to $99...
    // If no matching model exists, create one...
    $flight = App\Models\Flight::updateOrCreate(
        ['departure' => 'Oakland', 'destination' => 'San Diego'],
        ['price' => 99, 'discounted' => 1]
    );

Якщо ви хочете виконати кілька "виправлень" в одному запиті, тоді вам слід використовувати`upsert`замість цього. Перший аргумент методу складається зі значень, які потрібно вставити або оновити, тоді як другий аргумент містить перелік стовпців (колонок), які однозначно ідентифікують записи в асоційованій таблиці. Третім і остаточним аргументом методу є масив стовпців, який слід оновити, якщо відповідний запис уже існує в базі даних.`upsert`метод автоматично встановить`created_at`і`updated_at`мітки часу, якщо на моделі ввімкнено мітки часу:

    App\Models\Flight::upsert([
        ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
        ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
    ], ['departure', 'destination'], ['price']);

> {note} Для всіх баз даних, крім SQL Server, потрібні стовпці у другому аргументі`upsert`метод, щоб мати "первинний" або "унікальний" індекс.

<a name="deleting-models"></a>

## Видалення моделей

Щоб видалити модель, зателефонуйте на`delete`метод на екземплярі моделі:

    $flight = App\Models\Flight::find(1);

    $flight->delete();

<a name="deleting-an-existing-model-by-key"></a>

#### Видалення існуючої моделі за ключем

У наведеному вище прикладі ми отримуємо модель із бази даних перед викликом`delete`метод. Однак, якщо ви знаєте первинний ключ моделі, ви можете видалити модель без явного отримання, зателефонувавши до`destroy`метод. На додаток до одного первинного ключа як аргументу,`destroy`метод приймає кілька первинних ключів, масив первинних ключів або a[колекція](/docs/{{version}}/collections)первинних ключів:

    App\Models\Flight::destroy(1);

    App\Models\Flight::destroy(1, 2, 3);

    App\Models\Flight::destroy([1, 2, 3]);

    App\Models\Flight::destroy(collect([1, 2, 3]));

> {примітка}`destroy`метод завантажує кожну модель окремо і викликає`delete`метод на них, так що`deleting`і`deleted`події звільняються.

<a name="deleting-models-by-query"></a>

#### Видалення моделей за запитом

Ви також можете запустити оператор видалення на наборі моделей. У цьому прикладі ми видалимо всі рейси, позначені як неактивні. Як і масові оновлення, масові видалення не запускатимуть події моделі для моделей, які видаляються:

    $deletedRows = App\Models\Flight::where('active', 0)->delete();

> {note} При виконанні оператора масового видалення за допомогою Eloquent,`deleting`і`deleted`події моделі не запускатимуться для видалених моделей. Це тому, що моделі фактично ніколи не отримуються під час виконання оператора delete.

<a name="soft-deleting"></a>

### М'яке видалення

На додаток до того, що фактично видаляє записи з вашої бази даних, Eloquent також може «м'яко видаляти» моделі. Коли моделі видаляються м’яко, вони насправді не видаляються з вашої бази даних. Натомість a`deleted_at`атрибут встановлюється на моделі і вставляється в базу даних. Якщо модель має ненульове значення`deleted_at`значення, модель була видалена м'яко. Щоб увімкнути плавне видалення для моделі, використовуйте`Illuminate\Database\Eloquent\SoftDeletes`риса на моделі:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\SoftDeletes;

    class Flight extends Model
    {
        use SoftDeletes;
    }

> {tip} The`SoftDeletes`риса автоматично додасть`deleted_at`атрибут до`DateTime`/`Carbon`екземпляр для вас.

Слід також додати`deleted_at`стовпець до таблиці бази даних. Laravelь[конструктор схем](/docs/{{version}}/migrations)містить допоміжний метод для створення цього стовпця:

    public function up()
    {
        Schema::table('flights', function (Blueprint $table) {
            $table->softDeletes();
        });
    }

    public function down()
    {
        Schema::table('flights', function (Blueprint $table) {
            $table->dropSoftDeletes();
        });
    }

Тепер, коли ви телефонуєте на`delete`метод на моделі,`deleted_at`у стовпці буде встановлено поточну дату та час. При запиті до моделі, яка використовує м’яке видалення, м’яко видалені моделі автоматично виключаються з усіх результатів запиту.

Щоб визначити, чи було видалено даний екземпляр моделі м'яким способом, використовуйте`trashed`метод:

    if ($flight->trashed()) {
        //
    }

<a name="querying-soft-deleted-models"></a>

### Запит на видалені програми

<a name="including-soft-deleted-models"></a>

#### У тому числі програми з м'яким видаленням

Як зазначалося вище, м'яко видалені моделі будуть автоматично виключені з результатів запиту. Однак ви можете змусити м'яко видалені моделі з'являтися у наборі результатів за допомогою`withTrashed`метод за запитом:

    $flights = App\Models\Flight::withTrashed()
                    ->where('account_id', 1)
                    ->get();

`withTrashed`метод також може бути використаний на[зв'язки](/docs/{{version}}/eloquent-relationships)запит:

    $flight->history()->withTrashed()->get();

<a name="retrieving-only-soft-deleted-models"></a>

#### Отримання лише програм, які видалено з м’яким програмним забезпеченням

`onlyTrashed`метод отримає**лише**м'яко видалені моделі:

    $flights = App\Models\Flight::onlyTrashed()
                    ->where('airline_id', 1)
                    ->get();

<a name="restoring-soft-deleted-models"></a>

#### Відновлення видалених моделей

Іноді вам може знадобитися "видалити" м'яко видалену модель. Щоб відновити м'яку видалену модель у активний стан, використовуйте`restore`метод на екземплярі моделі:

    $flight->restore();

Ви також можете використовувати`restore`метод у запиті для швидкого відновлення декількох моделей. Знову ж таки, як і інші "масові" операції, це не буде запускати жодні події моделей для моделей, які відновляються:

    App\Models\Flight::withTrashed()
            ->where('airline_id', 1)
            ->restore();

Подобається`withTrashed`метод,`restore`метод також може бути використаний на[зв'язки](/docs/{{version}}/eloquent-relationships):

    $flight->history()->restore();

<a name="permanently-deleting-models"></a>

#### Постійне видалення моделей

Іноді вам може знадобитися справді видалити модель зі своєї бази даних. Щоб назавжди видалити з бази даних програмне забезпечення, що видаляється, використовуйте`forceDelete`метод:

    // Force deleting a single model instance...
    $flight->forceDelete();

    // Force deleting all related models...
    $flight->history()->forceDelete();

<a name="replicating-models"></a>

## Реплікаційні моделі

Ви можете створити незбережену копію екземпляра моделі за допомогою`replicate`метод. Це особливо корисно, коли у вас є екземпляри моделей, які мають багато однакових атрибутів:

    $shipping = App\Models\Address::create([
        'type' => 'shipping',
        'line_1' => '123 Example Street',
        'city' => 'Victorville',
        'state' => 'CA',
        'postcode' => '90001',
    ]);

    $billing = $shipping->replicate()->fill([
        'type' => 'billing'
    ]);

    $billing->save();

<a name="query-scopes"></a>

## Scopes запитів

<a name="global-scopes"></a>

### Глобальні Scopes

Глобальні Scopes дозволяють додавати обмеження до всіх запитів для даної моделі. Власного Laravel[м'яке видалення](#soft-deleting)функціональність використовує глобальні Scopes застосування, щоб витягувати з бази даних лише "не видалені" моделі. Написання власних глобальних масштабів може забезпечити зручний і простий спосіб переконатися, що кожен запит для даної моделі отримує певні обмеження.

<a name="writing-global-scopes"></a>

#### Написання глобальних масштабів

Написати глобальну сферу просто. Визначте клас, який реалізує`Illuminate\Database\Eloquent\Scope`інтерфейс. Цей інтерфейс вимагає реалізації одного методу:`apply`.`apply`метод може додати`where`обмеження на запит за необхідності:

    <?php

    namespace App\Scopes;

    use Illuminate\Database\Eloquent\Builder;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Scope;

    class AgeScope implements Scope
    {
        /**
         * Apply the scope to a given Eloquent query builder.
         *
         * @param  \Illuminate\Database\Eloquent\Builder  $builder
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @return void
         */
        public function apply(Builder $builder, Model $model)
        {
            $builder->where('age', '>', 200);
        }
    }

> {tip} Якщо ваша глобальна область додає стовпці до пункту select запиту, вам слід використовувати`addSelect`метод замість`select`. Це запобіжить ненавмисній заміні існуючого запиту вибору запиту.

<a name="applying-global-scopes"></a>

#### Застосування глобальних сфер

Щоб призначити модель глобальному обсягу, слід замінити дану модель`booted`метод і використовувати`addGlobalScope`метод:

    <?php

    namespace App\Models;

    use App\Scopes\AgeScope;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The "booted" method of the model.
         *
         * @return void
         */
        protected static function booted()
        {
            static::addGlobalScope(new AgeScope);
        }
    }

Після додавання Scopes, запит до`User::all()`створить наступний SQL:

    select * from `users` where `age` > 200

<a name="anonymous-global-scopes"></a>

#### Анонімні глобальні Scopes

Eloquent також дозволяє визначати глобальні Scopes застосування за допомогою Closures, що особливо корисно для простих областей застосування, які не вимагають окремого класу:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Builder;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The "booted" method of the model.
         *
         * @return void
         */
        protected static function booted()
        {
            static::addGlobalScope('age', function (Builder $builder) {
                $builder->where('age', '>', 200);
            });
        }
    }

<a name="removing-global-scopes"></a>

#### Видалення глобальних масштабів

Якщо ви хочете видалити глобальний обсяг для даного запиту, ви можете використовувати`withoutGlobalScope`метод. Метод приймає ім'я класу глобальної Scopes дії як єдиний аргумент:

    User::withoutGlobalScope(AgeScope::class)->get();

Або якщо ви визначили глобальний обсяг за допомогою Закриття:

    User::withoutGlobalScope('age')->get();

Якщо ви хочете видалити кілька або навіть усі глобальні Scopes, ви можете скористатися`withoutGlobalScopes`метод:

    // Remove all of the global scopes...
    User::withoutGlobalScopes()->get();

    // Remove some of the global scopes...
    User::withoutGlobalScopes([
        FirstScope::class, SecondScope::class
    ])->get();

<a name="local-scopes"></a>

### Місцеві Scopes застосування

Локальні Scopes дозволяють вам визначити загальні набори обмежень, які ви можете легко використовувати повторно у своїй програмі. Наприклад, можливо, вам доведеться часто отримувати всіх користувачів, які вважаються "популярними". Щоб визначити область, додайте до методу Eloquentї моделі префікс`scope`.

Scopes завжди повинні повертати екземпляр конструктора запитів:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Scope a query to only include popular users.
         *
         * @param  \Illuminate\Database\Eloquent\Builder  $query
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopePopular($query)
        {
            return $query->where('votes', '>', 100);
        }

        /**
         * Scope a query to only include active users.
         *
         * @param  \Illuminate\Database\Eloquent\Builder  $query
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeActive($query)
        {
            return $query->where('active', 1);
        }
    }

<a name="utilizing-a-local-scope"></a>

#### Використання локального обсягу

Після того, як область визначена, ви можете викликати методи Scopes при запиті моделі. Однак ви не повинні включати`scope`префікс при виклику методу. Ви навіть можете прив'язувати дзвінки до різних областей, наприклад:

    $users = App\Models\User::popular()->active()->orderBy('created_at')->get();

Поєднання декількох Eloquent модельних областей через`or`оператор запиту може вимагати використання зворотних викликів Closure:

    $users = App\Models\User::popular()->orWhere(function (Builder $query) {
        $query->active();
    })->get();

Однак, оскільки це може бути громіздким, Laravel забезпечує "вищий порядок"`orWhere`метод, який дозволяє вільно зв’язати ці Scopes дії без використання Закриття:

    $users = App\Models\User::popular()->orWhere->active()->get();

<a name="dynamic-scopes"></a>

#### Динамічні Scopes

Іноді вам може знадобитися визначити область, яка приймає параметри. Для початку просто додайте додаткові параметри до Scopes застосування. Параметри Scopes слід визначати після`$query`параметр:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Scope a query to only include users of a given type.
         *
         * @param  \Illuminate\Database\Eloquent\Builder  $query
         * @param  mixed  $type
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeOfType($query, $type)
        {
            return $query->where('type', $type);
        }
    }

Тепер ви можете передавати параметри під час виклику Scopes:

    $users = App\Models\User::ofType('admin')->get();

<a name="comparing-models"></a>

## Порівняння моделей

Іноді може знадобитися визначити, чи є дві моделі "однаковими".`is`метод може бути використаний для швидкої перевірки того, що дві моделі мають однаковий підключення первинного ключа, таблиці та бази даних:

    if ($post->is($anotherPost)) {
        //
    }

`is`метод також доступний при використанні`belongsTo`,`hasOne`,`morphTo`, і`morphOne`зв'язки. Цей метод особливо корисний, коли ви хочете порівняти пов’язану модель, не видаючи запит на отримання цієї моделі:

    if ($post->author()->is($user)) {
        //
    }

<a name="events"></a>

## Події

Eloquent моделі запускають декілька подій, що дозволяє зачепити такі моменти в життєвому циклі моделі:`retrieved`,`creating`,`created`,`updating`,`updated`,`saving`,`saved`,`deleting`,`deleted`,`restoring`,`restored`,`replicating`. Події дозволяють легко виконувати код кожного разу, коли певний клас моделі зберігається або оновлюється в базі даних. Кожна подія отримує екземпляр моделі через її конструктор.

`retrieved`подія спрацює, коли існуюча модель буде отримана з бази даних. Коли нова модель зберігається вперше, файл`creating`і`created`події відбуватимуться.`updating`/`updated`події запускатимуться, коли існуючу модель буде змінено та`save`метод називається.`saving`/`saved`події запускатимуться під час створення або оновлення моделі.

> {note} Під час масового оновлення або видалення через Eloquent,`saved`,`updated`,`deleting`, і`deleted`події моделі не будуть запускатись для моделей, що зазнали впливу. Це пов’язано з тим, що моделі фактично ніколи не отримуються під час масового оновлення чи видалення.

Для початку визначте a`$dispatchesEvents`властивість на вашій Eloquentй моделі, яке відображає різні точки життєвого циклу моделі Eloquentго до вашого власного[урокові заходи](/docs/{{version}}/events):

    <?php

    namespace App\Models;

    use App\Events\UserDeleted;
    use App\Events\UserSaved;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * The event map for the model.
         *
         * @var array
         */
        protected $dispatchesEvents = [
            'saved' => UserSaved::class,
            'deleted' => UserDeleted::class,
        ];
    }

Після визначення та відображення ваших Eloquent подій ви можете скористатися ними[Listeners подій](https://laravel.com/docs/{{version}}/events#defining-listeners)для обробки подій.

<a name="events-using-closures"></a>

### Використання замикань

Замість використання власних класів подій ви можете зареєструвати закриття, які виконуються під час запуску різних модельних подій. Як правило, ви повинні реєструвати ці закриття в`booted`метод вашої моделі:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The "booted" method of the model.
         *
         * @return void
         */
        protected static function booted()
        {
            static::created(function ($user) {
                //
            });
        }
    }

Якщо потрібно, ви можете використовувати[анонімні прослуховувачі подій у черзі](/docs/{{version}}/events#queuable-anonymous-event-listeners)при реєстрації модельних подій. Це дасть вказівку Laravel виконати прослуховувач події моделі за допомогою[черга](/docs/{{version}}/queues):

    use function Illuminate\Events\queueable;

    static::created(queueable(function ($user) {
        //
    }));

<a name="observers"></a>

### Observers(Спостерігачі)

<a name="defining-observers"></a>

#### Визначення Observers(Спостерігачі)в

Якщо ви слухаєте багато подій за даною моделлю, ви можете використовувати Observers(Спостерігачі)в, щоб згрупувати всіх своїх Listeners в один клас. Класи Observers(Спостерігачі)в мають імена методів, що відображають Eloquent події, які ви хочете послухати. Кожен із цих методів отримує модель як єдиний аргумент.`make:observer`Команда ремісників - це найпростіший спосіб створити новий клас Observers(Спостерігачі)в:

    php artisan make:observer UserObserver --model=User

Ця команда помістить нового спостерігача у ваш`App/Observers`каталог. Якщо цього каталогу не існує, Artisan створить його для вас. Ваш свіжий спостерігач буде виглядати наступним чином:

    <?php

    namespace App\Observers;

    use App\Models\User;

    class UserObserver
    {
        /**
         * Handle the User "created" event.
         *
         * @param  \App\Models\User  $user
         * @return void
         */
        public function created(User $user)
        {
            //
        }

        /**
         * Handle the User "updated" event.
         *
         * @param  \App\Models\User  $user
         * @return void
         */
        public function updated(User $user)
        {
            //
        }

        /**
         * Handle the User "deleted" event.
         *
         * @param  \App\Models\User  $user
         * @return void
         */
        public function deleted(User $user)
        {
            //
        }

        /**
         * Handle the User "forceDeleted" event.
         *
         * @param  \App\Models\User  $user
         * @return void
         */
        public function forceDeleted(User $user)
        {
            //
        }
    }

Щоб зареєструвати спостерігача, використовуйте`observe`метод на моделі, яку ви хочете спостерігати. Ви можете зареєструвати Observers(Спостерігачі)в у`boot`методу одного з ваших постачальників послуг. У цьому прикладі ми зареєструємо спостерігача в`AppServiceProvider`:

    <?php

    namespace App\Providers;

    use App\Observers\UserObserver;
    use App\Models\User;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
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
            User::observe(UserObserver::class);
        }
    }

<a name="muting-events"></a>

### Muting подій

Можливо, ви часом захочете тимчасово "вимкнути звук" усіх подій, які запускає модель. Ви можете досягти цього за допомогою`withoutEvents`метод.`withoutEvents`метод приймає Закриття як єдиний аргумент. Будь-який код, виконаний в рамках цього Закриття, не запускатиме події моделі. Наприклад, наступне завантажить і видалить файл`App\Models\User`екземпляр без запуску будь-яких модельних подій. Будь-яке значення, повернене даним Закриттям, буде повернене`withoutEvents`метод:

    use App\Models\User;

    $user = User::withoutEvents(function () use () {
        User::findOrFail(1)->delete();

        return User::find(2);
    });

<a name="saving-a-single-model-without-events"></a>

#### Збереження єдиної моделі без подій

Іноді вам може знадобитися "зберегти" дану модель, не викликаючи жодних подій. Ви можете досягти цього за допомогою`saveQuietly`метод:

    $user = User::findOrFail(1);

    $user->name = 'Victoria Faith';

    $user->saveQuietly();
