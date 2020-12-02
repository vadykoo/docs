# Тестування бази даних

-   [Вступ](#introduction)
-   [Скидання бази даних після кожного тесту](#resetting-the-database-after-each-test)
-   [Створення фабрик](#creating-factories)
-   [Написання фабрик](#writing-factories)
    -   [Заводські штати](#factory-states)
    -   [Фабричні зворотні дзвінки](#factory-callbacks)
-   [Використання заводів](#using-factories)
    -   [Створення моделей](#creating-models)
    -   [Персистуючі моделі](#persisting-models)
    -   [Послідовності](#sequences)
-   [Заводські відносини](#factory-relationships)
    -   [Відносини у межах визначень](#relationships-within-definition)
    -   [Має багато стосунків](#has-many-relationships)
    -   [Належить до стосунків](#belongs-to-relationships)
    -   [Багато до багатьох відносин](#many-to-many-relationships)
    -   [Поліморфні відносини](#polymorphic-relationships)
-   [Використання насіння](#using-seeds)
-   [Доступні твердження](#available-assertions)

<a name="introduction"></a>

## Вступ

Laravel пропонує безліч корисних інструментів для спрощення тестування програм, керованих базами даних. По-перше, ви можете використовувати`assertDatabaseHas`помічник стверджувати, що дані існують у базі даних, що відповідають заданому набору критеріїв. Наприклад, якщо ви хочете перевірити, чи є запис у файлі`users`таблиця з`email`значення`sally@example.com`, ви можете зробити наступне:

    public function testDatabase()
    {
        // Make call to application...

        $this->assertDatabaseHas('users', [
            'email' => 'sally@example.com',
        ]);
    }

Ви також можете використовувати`assertDatabaseMissing`помічник стверджувати, що дані не існують у базі даних.

`assertDatabaseHas`метод та інші подібні помічники для зручності. Ви можете використовувати будь-який із вбудованих методів твердження PHPUnit для доповнення тестів функцій.

<a name="resetting-the-database-after-each-test"></a>

## Скидання бази даних після кожного тесту

Часто корисно скидати базу даних після кожного тесту, щоб дані попереднього тесту не перешкоджали подальшим тестам.`RefreshDatabase`trait використовує найбільш оптимальний підхід до міграції тестової бази даних, залежно від того, використовуєте ви базу даних в пам'яті або традиційну базу даних. Використовуйте ознаку у своєму тестовому класі, і все буде оброблено для вас:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        use RefreshDatabase;

        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->get('/');

            // ...
        }
    }

<a name="creating-factories"></a>

## Створення фабрик

Під час тестування вам може знадобитися вставити кілька записів у вашу базу даних перед виконанням тесту. Замість того, щоб вручну вказувати значення кожного стовпця під час створення цих тестових даних, Laravel дозволяє визначити набір атрибутів за замовчуванням для кожного з ваших[Красномовні моделі](/docs/{{version}}/eloquent)за допомогою модельних фабрик.

Щоб створити фабрику, використовуйте`make:factory`[Ремісниче командування](/docs/{{version}}/artisan):

    php artisan make:factory PostFactory

Нова фабрика буде розміщена у вашому`database/factories`каталог.

`--model`Опція може використовуватися для позначення назви моделі, створеної на заводі. Цей параметр попередньо заповнить згенерований заводський файл заданою моделлю:

    php artisan make:factory PostFactory --model=Post

<a name="writing-factories"></a>

## Написання фабрик

Для початку погляньте на`database/factories/UserFactory.php`файл у вашій заявці. Зразу, цей файл містить таке заводське визначення:

    namespace Database\Factories;

    use App\Models\User;
    use Illuminate\Database\Eloquent\Factories\Factory;
    use Illuminate\Support\Str;

    class UserFactory extends Factory
    {
        /**
         * The name of the factory's corresponding model.
         *
         * @var string
         */
        protected $model = User::class;

        /**
         * Define the model's default state.
         *
         * @return array
         */
        public function definition()
        {
            return [
                'name' => $this->faker->name,
                'email' => $this->faker->unique()->safeEmail,
                'email_verified_at' => now(),
                'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
                'remember_token' => Str::random(10),
            ];
        }
    }

Як бачимо, у найосновнішому вигляді фабрики - це класи, які розширюють базовий заводський клас Laravel і визначають a`model`майно і`definition`метод.`definition`Метод повертає набір значень атрибутів за замовчуванням, які слід застосовувати при створенні моделі за допомогою заводу.

Через`faker`власності, фабрики мають доступ до[Факер](https://github.com/FakerPHP/Faker)Бібліотека PHP, яка дозволяє зручно генерувати різні види випадкових даних для тестування.

> {tip} Ви можете встановити локаль Faker, додавши a`faker_locale`варіант для вашого`config/app.php`файл конфігурації.

<a name="factory-states"></a>

### Заводські штати

Методи державного маніпулювання дозволяють визначити дискретні модифікації, які можна застосувати до ваших модельних заводів у будь-якій комбінації. Наприклад, ваш`User`модель може мати`suspended`стан, який змінює одне зі значень атрибута за замовчуванням. Ви можете визначити свої перетворення стану, використовуючи базові заводи`state`метод. Ви можете називати свій метод стану як завгодно. Зрештою, це лише типовий метод PHP. Наданий зворотний виклик маніпуляції станом отримає масив необроблених атрибутів, визначених для заводу, і повинен повернути масив атрибутів для модифікації:

    /**
     * Indicate that the user is suspended.
     *
     * @return \Illuminate\Database\Eloquent\Factories\Factory
     */
    public function suspended()
    {
        return $this->state(function (array $attributes) {
            return [
                'account_status' => 'suspended',
            ];
        });
    }

<a name="factory-callbacks"></a>

### Фабричні зворотні дзвінки

Заводські зворотні дзвінки реєструються за допомогою`afterMaking`і`afterCreating`методи і дозволяють виконувати додаткові завдання після виготовлення або створення моделі. Вам слід зареєструвати ці зворотні дзвінки, визначивши a`configure`метод на заводському класі. Цей метод буде автоматично викликаний Laravel, коли буде створено екземпляр заводу:

    namespace Database\Factories;

    use App\Models\User;
    use Illuminate\Database\Eloquent\Factories\Factory;
    use Illuminate\Support\Str;

    class UserFactory extends Factory
    {
        /**
         * The name of the factory's corresponding model.
         *
         * @var string
         */
        protected $model = User::class;

        /**
         * Configure the model factory.
         *
         * @return $this
         */
        public function configure()
        {
            return $this->afterMaking(function (User $user) {
                //
            })->afterCreating(function (User $user) {
                //
            });
        }

        // ...
    }

<a name="using-factories"></a>

## Використання заводів

<a name="creating-models"></a>

### Створення моделей

Визначивши свої заводи, ви можете використовувати статику`factory`метод, передбачений`Illuminate\Database\Eloquent\Factories\HasFactory`риси на ваших красномовних моделях, щоб створити екземпляр заводського екземпляра для цієї моделі:

    namespace App\Models;

    use Illuminate\Database\Eloquent\Factories\HasFactory;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        use HasFactory;
    }

Давайте розглянемо кілька прикладів створення моделей. Спочатку ми використаємо`make`метод створення моделей без збереження їх у базі даних:

    use App\Models\User;

    public function testDatabase()
    {
        $user = User::factory()->make();

        // Use model in tests...
    }

You may create a collection of many models using the `count`метод:

    // Create three App\Models\User instances...
    $users = User::factory()->count(3)->make();

`HasFactory`особливості`factory`Метод буде використовувати конвенції для визначення належної фабрики для моделі. Зокрема, метод буде шукати фабрику в`Database\Factories`простір імен, який має ім'я класу, що відповідає імені моделі і суфіксом якого є`Factory`. Якщо ці умови не стосуються конкретного додатка чи заводу, ви можете замінити`newFactory`метод на вашій моделі, щоб повернути екземпляр відповідного заводу моделі безпосередньо:

    /**
     * Create a new factory instance for the model.
     *
     * @return \Illuminate\Database\Eloquent\Factories\Factory
     */
    protected static function newFactory()
    {
        return \Database\Factories\Administration\FlightFactory::new();
    }

<a name="applying-states"></a>

#### Держави, що застосовують

Ви також можете застосувати будь-який із своїх[штатів](#factory-states)до моделей. Якщо ви хочете застосувати до моделей кілька перетворень стану, ви можете просто викликати методи стану безпосередньо:

    $users = User::factory()->count(5)->suspended()->make();

<a name="overriding-attributes"></a>

#### Заміна атрибутів

Якщо ви хочете замінити деякі значення за замовчуванням для своїх моделей, ви можете передати масив значень в`make`метод. Тільки зазначені значення будуть замінені, тоді як для решти значень залишаються встановлені значення за замовчуванням, як це вказано на заводі:

    $user = User::factory()->make([
        'name' => 'Abigail Otwell',
    ]);

Крім того,`state`метод може бути викликаний безпосередньо на заводському екземплярі для виконання вбудованого перетворення стану:

    $user = User::factory()->state([
        'name' => 'Abigail Otwell',
    ])->make();

> {порада}[Захист від масового призначення](/docs/{{version}}/eloquent#mass-assignment)автоматично вимикається при створенні моделей на заводах.

<a name="persisting-models"></a>

### Персистуючі моделі

`create` method creates model instances and persists them to the database using Eloquent's `save`метод:

    use App\Models\User;

    public function testDatabase()
    {
        // Create a single App\Models\User instance...
        $user = User::factory()->create();

        // Create three App\Models\User instances...
        $users = User::factory()->count(3)->create();

        // Use model in tests...
    }

Ви можете замінити атрибути на моделі, передавши масив атрибутів в`create`метод:

    $user = User::factory()->create([
        'name' => 'Abigail',
    ]);

<a name="sequences"></a>

### Послідовності

Іноді вам може знадобитися чергувати значення даного атрибута моделі для кожної створеної моделі. Ви можете досягти цього, визначивши перетворення держави як`Sequence`екземпляр. Наприклад, ми можемо побажати чергувати значення`admin`стовпець на a`User`модель між`Y`і`N`для кожного створеного користувача:

    use App\Models\User;
    use Illuminate\Database\Eloquent\Factories\Sequence;

    $users = User::factory()
                    ->count(10)
                    ->state(new Sequence(
                        ['admin' => 'Y'],
                        ['admin' => 'N'],
                    ))
                    ->create();

У цьому прикладі буде створено п'ять користувачів із`admin`значення`Y`і п'ять користувачів будуть створені з`admin`значення`N`.

<a name="factory-relationships"></a>

## Заводські відносини

<a name="relationships-within-definition"></a>

### Відносини у межах визначень

Ви можете прив’язати стосунки до моделей у своїх заводських визначеннях. Наприклад, якщо ви хочете створити новий`User`при створенні`Post`, ви можете зробити наступне:

    use App\Models\User;

    /**
     * Define the model's default state.
     *
     * @return array
     */
    public function definition()
    {
        return [
            'user_id' => User::factory(),
            'title' => $this->faker->title,
            'content' => $this->faker->paragraph,
        ];
    }

Якщо стовпці відношення залежать від заводу, який його визначає, ви можете надати зворотний виклик, який приймає обчислюваний масив атрибутів:

    /**
     * Define the model's default state.
     *
     * @return array
     */
    public function definition()
    {
        return [
            'user_id' => User::factory(),
            'user_type' => function (array $attributes) {
                return User::find($attributes['user_id'])->type;
            },
            'title' => $this->faker->title,
            'content' => $this->faker->paragraph,
        ];
    }

<a name="has-many-relationships"></a>

### Має багато стосунків

Далі, давайте дослідимо побудову красномовних зв’язків між моделями, використовуючи вільні фабричні методи Laravel. По-перше, припустимо, що наш додаток має`User`модель та a`Post`модель. Крім того, припустимо, що`User`модель визначає a`hasMany`відносини з`Post`. Ми можемо створити користувача, який має три дописи за допомогою`has`метод, що надається заводом.`has`метод приймає заводський екземпляр:

    use App\Models\Post;
    use App\Models\User;

    $user = User::factory()
                ->has(Post::factory()->count(3))
                ->create();

By convention, when passing a `Post`модель до`has`методом, Laravel припустить, що`User`модель повинна мати a`posts`метод, що визначає взаємозв'язок. За необхідності ви можете чітко вказати назву відносин, якими ви хочете маніпулювати:

    $user = User::factory()
                ->has(Post::factory()->count(3), 'posts')
                ->create();

Звичайно, ви можете виконувати державні маніпуляції на відповідних моделях. Крім того, ви можете пройти перетворення стану на основі закриття, якщо зміна стану вимагає доступу до батьківської моделі:

    $user = User::factory()
                ->has(
                    Post::factory()
                            ->count(3)
                            ->state(function (array $attributes, User $user) {
                                return ['user_type' => $user->type];
                            })
                )
                ->create();

<a name="has-many-relationships-using-magic-methods"></a>

#### Використання магічних методів

Для зручності ви можете використовувати фабричні методи магічних відносин для визначення відносин. Наприклад, у наступному прикладі буде використано конвенцію, щоб визначити, що відповідні моделі слід створювати за допомогою`posts`метод відносин на`User`модель:

    $user = User::factory()
                ->hasPosts(3)
                ->create();

Використовуючи магічні методи для створення фабричних відносин, ви можете передати масив атрибутів, щоб замінити їх на відповідні моделі:

    $user = User::factory()
                ->hasPosts(3, [
                    'published' => false,
                ])
                ->create();

Ви можете надати перетворення стану на основі закриття, якщо зміна стану вимагає доступу до батьківської моделі:

    $user = User::factory()
                ->hasPosts(3, function (array $attributes, User $user) {
                    return ['user_type' => $user->type];
                })
                ->create();

<a name="belongs-to-relationships"></a>

### Належить до стосунків

Тепер, коли ми дослідили, як побудувати відносини "має багато" за допомогою фабрик, давайте дослідимо зворотний зв'язок.`for`метод може бути використаний для визначення моделі, якій належать створені на заводі моделі. Наприклад, ми можемо створити три`Post`екземпляри моделі, що належать одному користувачеві:

    use App\Models\Post;
    use App\Models\User;

    $posts = Post::factory()
                ->count(3)
                ->for(User::factory()->state([
                    'name' => 'Jessica Archer',
                ]))
                ->create();

<a name="belongs-to-relationships-using-magic-methods"></a>

#### Використання магічних методів

Для зручності ви можете використовувати фабричні методи магічних відносин для визначення відносин "належить". Наприклад, у наступному прикладі буде використано конвенцію, щоб визначити, що три посади повинні належати до`user`відносини на`Post`модель:

    $posts = Post::factory()
                ->count(3)
                ->forUser([
                    'name' => 'Jessica Archer',
                ])
                ->create();

<a name="many-to-many-relationships"></a>

### Багато до багатьох відносин

Люблю[має багато стосунків](#has-many-relationships), відносини "багато до багатьох" можуть бути створені за допомогою`has`метод:

    use App\Models\Role;
    use App\Models\User;

    $users = User::factory()
                ->has(Role::factory()->count(3))
                ->create();

<a name="pivot-table-attributes"></a>

#### Атрибути зведеної таблиці

Якщо вам потрібно визначити атрибути, які слід встановити у зведеній / проміжній таблиці, що пов'язує моделі, ви можете використовувати`hasAttached`метод. Цей метод приймає масив імен та значень атрибутів зведеної таблиці як другий аргумент:

    use App\Models\Role;
    use App\Models\User;

    $users = User::factory()
                ->hasAttached(
                    Role::factory()->count(3),
                    ['active' => true]
                )
                ->create();

Ви можете надати перетворення стану на основі закриття, якщо зміна стану вимагає доступу до відповідної моделі:

    $users = User::factory()
                ->hasAttached(
                    Role::factory()
                        ->count(3)
                        ->state(function (array $attributes, User $user) {
                            return ['name' => $user->name.' Role'];
                        }),
                    ['active' => true]
                )
                ->create();

<a name="many-to-many-relationships-using-magic-methods"></a>

#### Використання магічних методів

Для зручності ви можете використовувати фабричні методи магічних відносин, щоб визначити багато-багато відносин. Наприклад, у наступному прикладі буде використано конвенцію, щоб визначити, що відповідні моделі слід створювати за допомогою`roles`метод відносин на`User`модель:

    $users = User::factory()
                ->hasRoles(1, [
                    'name' => 'Editor'
                ])
                ->create();

<a name="polymorphic-relationships"></a>

### Поліморфні відносини

[Поліморфні зв’язки](/docs/{{version}}/eloquent-relationships#polymorphic-relationships)також можуть бути створені за допомогою заводів. Поліморфні відносини "морф багато" створюються так само, як типові відносини "має багато". Наприклад, якщо a`Post`модель має`morphMany`відносини з а`Comment`модель:

    use App\Models\Post;

    $post = Post::factory()->hasComments(3)->create();

<a name="morph-to-relationships"></a>

#### Морф до відносин

Магічні методи не можуть використовуватися для створення`morphTo`стосунки. Натомість,`for`метод повинен використовуватися безпосередньо, а назва відносин має бути чітко вказана. Наприклад, уявіть, що`Comment`модель має`commentable`метод, що визначає a`morphTo`відносини. У цій ситуації ми можемо створити три коментарі, які належать до однієї публікації, використовуючи`for`метод безпосередньо:

    $comments = Comment::factory()->count(3)->for(
        Post::factory(), 'commentable'
    )->create();

<a name="polymorphic-many-to-many-relationships"></a>

#### Поліморфні відносини багато-багато-багато

Поліморфні відносини "багато-до-багатьох" можуть бути створені так само, як неполіморфні відносини "багато до багатьох":

    use App\Models\Tag;
    use App\Models\Video;

    $videos = Video::factory()
                ->hasAttached(
                    Tag::factory()->count(3),
                    ['public' => true]
                )
                ->create();

Звичайно, магія`has`метод також може бути використаний для створення поліморфних зв'язків "багато до багатьох":

    $videos = Video::factory()
                ->hasTags(3, ['public' => true])
                ->create();

<a name="using-seeds"></a>

## Використання насіння

Якщо ви хочете використовувати[сівалки баз даних](/docs/{{version}}/seeding)щоб заповнити базу даних під час тестування функцій, ви можете використовувати`seed`метод. За замовчуванням`seed`метод поверне файл`DatabaseSeeder`, який повинен стратити всіх інших ваших сівалок. Крім того, ви передаєте конкретну назву класу сівалки в`seed`метод:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use OrderStatusSeeder;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        use RefreshDatabase;

        /**
         * Test creating a new order.
         *
         * @return void
         */
        public function testCreatingANewOrder()
        {
            // Run the DatabaseSeeder...
            $this->seed();

            // Run a single seeder...
            $this->seed(OrderStatusSeeder::class);

            // ...
        }
    }

Alternatively, you may instruct the `RefreshDatabase`ознака для автоматичного залучення бази даних перед кожним тестом. Ви можете досягти цього, визначивши a`$seed`властивість у вашому тестовому класі:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Indicates whether the database should be seeded before each test.
         *
         * @var bool
         */
        protected $seed = true;
        
        // ...
    }

<a name="available-assertions"></a>

## Доступні твердження

Laravel пропонує кілька тверджень бази даних для вашого[PHPUnit](https://phpunit.de/)тести функцій:

| Метод                                                | Опис                                                                   |
| ---------------------------------------------------- | ---------------------------------------------------------------------- |
| `$this->assertDatabaseCount($table, int $count);`    | Стверджуйте, що таблиця в базі даних містить задану кількість записів. |
| `$this->assertDatabaseHas($table, array $data);`     | Стверджуйте, що таблиця в базі даних містить дані.                     |
| `$this->assertDatabaseMissing($table, array $data);` | Стверджуйте, що таблиця в базі даних не містить даних.                 |
| `$this->assertDeleted($table, array $data);`         | Стверджуйте, що даний запис було видалено.                             |
| `$this->assertSoftDeleted($table, array $data);`     | Стверджуйте, що даний запис було видалено м’яко.                       |

Для зручності ви можете передати модель`assertDeleted`і`assertSoftDeleted`Помічники для затвердження запису були видалені або видалені з бази даних відповідно на основі первинного ключа моделі.

Наприклад, якщо ви використовуєте фабрику моделей у своєму тесті, ви можете передати цю модель одному з таких помічників, щоб протестувати вашу програму, щоб правильно видалити запис із бази даних:

    public function testDatabase()
    {
        $user = User::factory()->create();

        // Make call to application...

        $this->assertDeleted($user);
    }
