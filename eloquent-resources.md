# Eloquent: Ресурси API

-   [Вступ](#introduction)
-   [Генерування ресурсів](#generating-resources)
-   [Огляд концепції](#concept-overview)
    -   [Колекції ресурсів](#resource-collections)
-   [Написання ресурсів](#writing-resources)
    -   [Перенесення даних](#data-wrapping)
    -   [Пагінація](#pagination)
    -   [Умовні атрибути](#conditional-attributes)
    -   [Умовні зв'язки](#conditional-relationships)
    -   [Додавання метаданих](#adding-meta-data)
-   [Відповіді ресурсів](#resource-responses)

<a name="introduction"></a>

## Вступ

Створюючи API, вам може знадобитися рівень трансформації, який знаходиться між вашими Eloquent моделями та відповідями JSON, які фактично повертаються користувачам вашого додатка. Класи ресурсів Laravel дозволяють виразно та легко трансформувати свої моделі та колекції моделей у JSON.

<a name="generating-resources"></a>

## Генерування ресурсів

Для створення класу ресурсів ви можете використовувати`make:resource`Ремісниче командування. За замовчуванням ресурси будуть розміщені в`app/Http/Resources`каталог вашої програми. Ресурси розширюють`Illuminate\Http\Resources\Json\JsonResource`клас:

    php artisan make:resource User

<a name="generating-resource-collections"></a>

#### Колекції ресурсів

На додаток до створення ресурсів, які трансформують окремі моделі, ви можете генерувати ресурси, які відповідають за перетворення колекцій моделей. Це дозволяє вашій відповіді включати посилання та іншу метаінформацію, яка стосується всієї колекції даного ресурсу.

Для створення колекції ресурсів слід використовувати`--collection`прапор при створенні ресурсу. Або, включаючи слово`Collection`в назві ресурсу вкаже Laravel, що він повинен створити ресурс колекції. Колекційні ресурси розширюють`Illuminate\Http\Resources\Json\ResourceCollection`клас:

    php artisan make:resource User --collection

    php artisan make:resource UserCollection

<a name="concept-overview"></a>

## Огляд концепції

> {tip} Це огляд високого рівня ресурсів та колекцій ресурсів. Вам настійно рекомендується прочитати інші розділи цієї документації, щоб глибше зрозуміти налаштування та потужність, пропоновані вам ресурсами.

Перш ніж заглибитися у всі варіанти, доступні для вас під час написання ресурсів, давайте спочатку подивимось на високому рівні, як ресурси використовуються в Laravel. Клас ресурсів представляє єдину модель, яку потрібно перетворити на структуру JSON. Наприклад, ось просте`User`клас ресурсу:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class User extends JsonResource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

Кожен клас ресурсів визначає a`toArray`метод, який повертає масив атрибутів, які слід перетворити в JSON під час надсилання відповіді. Зверніть увагу, що ми можемо отримати доступ до властивостей моделі безпосередньо з`$this`змінна. Це пов’язано з тим, що клас ресурсу автоматично надаватиме доступ до властивостей і методів до базової моделі для зручного доступу. Після визначення ресурсу його можна повернути з маршруту або контролера:

    use App\Http\Resources\User as UserResource;
    use App\Models\User;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

<a name="resource-collections"></a>

### Колекції ресурсів

Якщо ви повертаєте колекцію ресурсів або сторінкову відповідь, ви можете використовувати файл`collection`метод при створенні екземпляра ресурсу у вашому маршруті або контролері:

    use App\Http\Resources\User as UserResource;
    use App\Models\User;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

Зауважте, що це не дозволяє додавати метадані, які, можливо, доведеться повернути разом із колекцією. Якщо ви хочете налаштувати відповідь на колекцію ресурсів, ви можете створити виділений ресурс для представлення колекції:

    php artisan make:resource UserCollection

Після створення класу збору ресурсів ви можете легко визначити будь-які метадані, які повинні бути включені у відповідь:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }

Після визначення вашої колекції ресурсів вона може бути повернута з маршруту або контролера:

    use App\Http\Resources\UserCollection;
    use App\Models\User;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

<a name="preserving-collection-keys"></a>

#### Збереження ключів колекції

Повертаючи колекцію ресурсів із маршруту, Laravel скидає ключі колекції, щоб вони мали простий числовий порядок. Однак ви можете додати a`preserveKeys`властивість до вашого класу ресурсів із зазначенням, чи слід зберігати ключі колекції:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class User extends JsonResource
    {
        /**
         * Indicates if the resource's collection keys should be preserved.
         *
         * @var bool
         */
        public $preserveKeys = true;
    }

Коли`preserveKeys`для властивості встановлено значення`true`, ключі колекції збережуться:

    use App\Http\Resources\User as UserResource;
    use App\Models\User;

    Route::get('/user', function () {
        return UserResource::collection(User::all()->keyBy->id);
    });

<a name="customizing-the-underlying-resource-class"></a>

#### Налаштування базового класу ресурсів

Як правило,`$this->collection`властивість колекції ресурсів автоматично заповнюється результатом зіставлення кожного елемента колекції з її особливим класом ресурсів. Клас ресурсу в однині вважається назвою класу колекції без кінцевого результату`Collection`рядок.

Наприклад,`UserCollection`спробує відобразити дані екземплярів користувача в`User`ресурс. Щоб налаштувати цю поведінку, ви можете замінити`$collects`властивість вашої колекції ресурсів:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * The resource that this resource collects.
         *
         * @var string
         */
        public $collects = 'App\Http\Resources\Member';
    }

<a name="writing-resources"></a>

## Написання ресурсів

> {tip} Якщо ви ще не читали[огляд концепції](#concept-overview), вам настійно рекомендується це зробити, перш ніж продовжувати роботу з цією документацією.

По суті, ресурси прості. Їм потрібно лише перетворити дану модель на масив. Отже, кожен ресурс містить a`toArray`метод, який переводить атрибути вашої моделі в масив, зручний для API, який можна повернути вашим користувачам:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class User extends JsonResource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

Після визначення ресурсу його можна повернути безпосередньо з маршруту або контролера:

    use App\Http\Resources\User as UserResource;
    use App\Models\User;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

<a name="relationships"></a>

#### зв'язки

Якщо ви хочете включити відповідні ресурси у свою відповідь, ви можете додати їх до масиву, який повертає ваш`toArray`метод. У цьому прикладі ми будемо використовувати`Post`ресурсів`collection`спосіб додати дописи користувача в блозі до відповіді ресурсу:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => PostResource::collection($this->posts),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

> {tip} Якщо ви хочете включити зв'язки лише тоді, коли вони вже завантажені, перегляньте документацію на[умовні зв'язки](#conditional-relationships).

<a name="writing-resource-collections"></a>

#### Колекції ресурсів

Поки ресурси перекладають одну модель у масив, колекції ресурсів перетворюють колекцію моделей у масив. Не обов'язково визначати клас збору ресурсів для кожного з типів вашої моделі, оскільки всі ресурси надають файл`collection`метод генерування "спеціальної" колекції ресурсів на льоту:

    use App\Http\Resources\User as UserResource;
    use App\Models\User;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

Однак, якщо вам потрібно налаштувати метадані, що повертаються разом із колекцією, потрібно буде визначити колекцію ресурсів:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }

Як і окремі ресурси, колекції ресурсів можуть повертатися безпосередньо з маршрутів або контролерів:

    use App\Http\Resources\UserCollection;
    use App\Models\User;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

<a name="data-wrapping"></a>

### Перенесення даних

За замовчуванням ваш найвіддаленіший ресурс загортається у файл`data`ключ, коли відповідь ресурсу перетворюється на JSON. Так, наприклад, типова відповідь на збір ресурсів виглядає так:

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ]
    }

Якщо ви хочете використовувати спеціальний ключ замість`data`, Ви можете визначити a`$wrap`атрибут класу ресурсів:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class User extends JsonResource
    {
        /**
         * The "data" wrapper that should be applied.
         *
         * @var string
         */
        public static $wrap = 'user';
    }

Якщо ви хочете відключити обтікання самого зовнішнього ресурсу, ви можете використовувати`withoutWrapping`метод на базовому класі ресурсів. Як правило, вам слід викликати цей метод з вашого`AppServiceProvider`або інший[постачальник послуг](/docs/{{version}}/providers)що завантажується при кожному запиті до вашої програми:

    <?php

    namespace App\Providers;

    use Illuminate\Http\Resources\Json\JsonResource;
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
            JsonResource::withoutWrapping();
        }
    }

> {примітка}`withoutWrapping`метод впливає лише на крайню відповідь і не видаляє`data`ключі, які ви вручну додаєте до власних колекцій ресурсів.

<a name="wrapping-nested-resources"></a>

### Обтікання вкладених ресурсів

Ви маєте повну свободу визначати, як стосуватимуться зв'язки вашого ресурсу. Якщо ви хочете, щоб усі колекції ресурсів були загорнуті в`data`ключ, незалежно від їх вкладеності, ви повинні визначити клас збору ресурсів для кожного ресурсу і повернути колекцію в межах`data`ключ.

Можливо, вам цікаво, чи це не призведе до того, що ваш найвіддаленіший ресурс буде обгорнуто надвоє`data`клавіші. Не хвилюйтеся, Laravel ніколи не дозволить, щоб ваші ресурси випадково були подвійно загорнуті, тому вам не потрібно турбуватися про рівень вкладеності колекції ресурсів, яку ви перетворюєте:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class CommentsCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return ['data' => $this->collection];
        }
    }

<a name="data-wrapping-and-pagination"></a>

### Упаковка даних та пагінація

Коли повертає пагіновані колекції у відповідь ресурсу, Laravel оберне дані ваших ресурсів у файл`data`клавішу, навіть якщо`withoutWrapping`метод був названий. Це тому, що пагіновані відповіді завжди містять`meta`і`links`ключі з інформацією про стан пагінатора:

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ],
        "links":{
            "first": "http://example.com/pagination?page=1",
            "last": "http://example.com/pagination?page=1",
            "prev": null,
            "next": null
        },
        "meta":{
            "current_page": 1,
            "from": 1,
            "last_page": 1,
            "path": "http://example.com/pagination",
            "per_page": 15,
            "to": 10,
            "total": 10
        }
    }

<a name="pagination"></a>

### Пагінація

Ви завжди можете передати примірник пагінатора в`collection`методу ресурсу або до власної колекції ресурсів:

    use App\Http\Resources\UserCollection;
    use App\Models\User;

    Route::get('/users', function () {
        return new UserCollection(User::paginate());
    });

Сторінкові відповіді завжди містять`meta`і`links`ключі з інформацією про стан пагінатора:

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ],
        "links":{
            "first": "http://example.com/pagination?page=1",
            "last": "http://example.com/pagination?page=1",
            "prev": null,
            "next": null
        },
        "meta":{
            "current_page": 1,
            "from": 1,
            "last_page": 1,
            "path": "http://example.com/pagination",
            "per_page": 15,
            "to": 10,
            "total": 10
        }
    }

<a name="conditional-attributes"></a>

### Умовні атрибути

Іноді вам може знадобитися включити атрибут у відповідь ресурсу, лише якщо виконана задана умова. Наприклад, ви можете включити значення, лише якщо поточний користувач є "адміністратором". Laravel пропонує безліч допоміжних методів, які допоможуть вам у цій ситуації.`when`метод може бути використаний для умовного додавання атрибута до відповіді ресурсу:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'secret' => $this->when(Auth::user()->isAdmin(), 'secret-value'),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

У цьому прикладі`secret`ключ буде повернуто в остаточній відповіді ресурсу, лише якщо аутентифікований користувач`isAdmin`метод повертає`true`. Якщо метод повертається`false`,`secret`ключ буде повністю видалено з відповіді ресурсу, перш ніж він буде відправлений назад клієнту.`when`метод дозволяє виразно визначити ваші ресурси, не вдаючись до умовних операторів при побудові масиву.

`when`метод також приймає Закриття як другий аргумент, дозволяючи обчислити отримане значення лише за умови, що задана умова`true`:

    'secret' => $this->when(Auth::user()->isAdmin(), function () {
        return 'secret-value';
    }),

<a name="merging-conditional-attributes"></a>

#### Об’єднання умовних атрибутів

Іноді у вас може бути кілька атрибутів, які повинні бути включені у відповідь ресурсу на основі однієї і тієї ж умови. У цьому випадку ви можете використовувати`mergeWhen`метод включати атрибути у відповідь лише тоді, коли задана умова є`true`:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            $this->mergeWhen(Auth::user()->isAdmin(), [
                'first-secret' => 'value',
                'second-secret' => 'value',
            ]),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

Знову ж таки, якщо задана умова є`false`, ці атрибути будуть повністю видалені з відповіді ресурсу до того, як вони будуть надіслані клієнту.

> {примітка}`mergeWhen`метод не слід використовувати в масивах, що поєднують рядкові та числові клавіші. Крім того, його не слід використовувати в масивах з цифровими клавішами, які не впорядковані послідовно.

<a name="conditional-relationships"></a>

### Умовні зв'язки

На додаток до умовно завантажувальних атрибутів, ви можете умовно включити зв'язки у відповіді вашого ресурсу на основі того, чи зв'язки вже завантажені в модель. Це дозволяє вашому контролеру вирішувати, які зв'язки слід завантажувати в модель, і ваш ресурс може легко включати їх лише тоді, коли вони насправді завантажені.

Зрештою, це полегшує уникнення проблем із запитом "N + 1" у ваших ресурсах.`whenLoaded`метод може бути використаний для умовного завантаження зв'язків. Щоб уникнути непотрібного завантаження зв'язків, цей метод приймає ім’я зв'язки замість самого відношення:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => PostResource::collection($this->whenLoaded('posts')),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

У цьому прикладі, якщо зв'язки не завантажені, файл`posts`ключ буде повністю вилучено з відповіді ресурсу до того, як його буде надіслано клієнту.

<a name="conditional-pivot-information"></a>

#### Інформація про умовний зведення

Окрім умовного включення інформації про зв’язок у відповіді ваших ресурсів, ви можете умовно включити дані з проміжних таблиць взаємозв’язків багато-до-багатьох, використовуючи`whenPivotLoaded`метод.`whenPivotLoaded`Метод приймає ім'я зведеної таблиці як перший аргумент. Другим аргументом має бути Закриття, яке визначає значення, яке повертається, якщо в моделі доступна інформація про зведення:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'expires_at' => $this->whenPivotLoaded('role_user', function () {
                return $this->pivot->expires_at;
            }),
        ];
    }

Якщо у вашій проміжній таблиці використовується аксесуар, відмінний від`pivot`, ви можете використовувати`whenPivotLoadedAs`метод:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'expires_at' => $this->whenPivotLoadedAs('subscription', 'role_user', function () {
                return $this->subscription->expires_at;
            }),
        ];
    }

<a name="adding-meta-data"></a>

### Додавання метаданих

Деякі стандарти JSON API вимагають додавання метаданих у відповіді вашого ресурсу та колекцій ресурсів. Сюди часто входять такі речі, як`links`до ресурсу або пов’язаних ресурсів, або метаданих про сам ресурс. Якщо вам потрібно повернути додаткові метадані про ресурс, включіть їх до свого`toArray`метод. Наприклад, ви можете включити`link` information when transforming a resource collection:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }

Повертаючи додаткові метадані з ваших ресурсів, вам ніколи не доведеться турбуватися про випадкове перевизначення`links`або`meta`клавіші, які автоматично додає Laravel при поверненні сторінкових відповідей. Будь-які додаткові`links`Ви визначите, що буде об'єднано з посиланнями, наданими пагінатором.

<a name="top-level-meta-data"></a>

#### Метадані верхнього рівня

Іноді, можливо, ви захочете включити певні метадані з відповіддю ресурсу, лише якщо ресурс є тим зовнішнім ресурсом, який повертається. Як правило, це включає метаінформацію про відповідь у цілому. Щоб визначити ці метадані, додайте a`with`метод до вашого класу ресурсів. Цей метод повинен повертати масив метаданих, який буде включено до відповіді ресурсу лише тоді, коли ресурс є самим зовнішнім ресурсом, який відображається:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return parent::toArray($request);
        }

        /**
         * Get additional data that should be returned with the resource array.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function with($request)
        {
            return [
                'meta' => [
                    'key' => 'value',
                ],
            ];
        }
    }

<a name="adding-meta-data-when-constructing-resources"></a>

#### Додавання метаданих при побудові ресурсів

Ви також можете додавати дані верхнього рівня під час створення екземплярів ресурсів у своєму маршруті або контролері.`additional`метод, доступний на всіх ресурсах, приймає масив даних, які слід додати до відповіді ресурсу:

    return (new UserCollection(User::all()->load('roles')))
                    ->additional(['meta' => [
                        'key' => 'value',
                    ]]);

<a name="resource-responses"></a>

## Відповіді ресурсів

Як ви вже читали, ресурси можуть повертатися безпосередньо з маршрутів та контролерів:

    use App\Http\Resources\User as UserResource;
    use App\Models\User;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

Однак іноді вам може знадобитися налаштувати вихідну відповідь HTTP, перш ніж вона буде надіслана клієнту. Є два шляхи досягнення цього. По-перше, ви можете ланцюжок`response`метод на ресурс. Цей метод поверне файл`Illuminate\Http\JsonResponse`екземпляр, що дозволяє вам повністю контролювати заголовки відповідей:

    use App\Http\Resources\User as UserResource;
    use App\Models\User;

    Route::get('/user', function () {
        return (new UserResource(User::find(1)))
                    ->response()
                    ->header('X-Value', 'True');
    });

Крім того, ви можете визначити a`withResponse`метод у самому ресурсі. Цей метод буде викликаний, коли ресурс повертається як найвіддаленіший ресурс у відповіді:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class User extends JsonResource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
            ];
        }

        /**
         * Customize the outgoing response for the resource.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Illuminate\Http\Response  $response
         * @return void
         */
        public function withResponse($request, $response)
        {
            $response->header('X-Value', 'True');
        }
    }
