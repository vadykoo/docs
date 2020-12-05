# База даних: Seeds

-   [Вступ](#introduction)
-   [Написання Seeders](#writing-seeders)
    -   [Використання модельних Factories](#using-model-factories)
    -   [Виклик додаткових Seeders](#calling-additional-seeders)
-   [Запуск Seeders](#running-seeders)

<a name="introduction"></a>

## Вступ

Laravel включає простий метод розподілу бази даних з тестовими даними за допомогою класів Seeds. Усі класи Seeds зберігаються в`database/seeders`каталог. Класи Seeds можуть мати будь-яке ім'я, яке ви бажаєте, але, мабуть, слід дотримуватися деяких розумних домовленостей, таких як`UserSeeder`тощо. За замовчуванням a`DatabaseSeeder`клас визначений для вас. З цього класу ви можете використовувати`call`метод для запуску інших класів Seeds, що дозволяє контролювати порядок висіву.

<a name="writing-seeders"></a>

## Написання Seeders

Щоб створити сіялку, виконайте`make:seeder`[artisan командування](/docs/{{version}}/artisan). Усі сівалки, створені в рамках, будуть розміщені в`database/seeders`каталог:

    php artisan make:seeder UserSeeder

Клас сівалки містить лише один метод за замовчуванням:`run`. Цей метод викликається, коли`db:seed`[artisan командування](/docs/{{version}}/artisan)виконується. У межах`run`методом, ви можете вставити дані у свою базу даних, як завгодно. Ви можете використовувати[конструктор запитів](/docs/{{version}}/queries)щоб вручну вставити дані, або ви можете використовувати[Eloquent фабрики моделей](/docs/{{version}}/database-testing#writing-factories).

> {порада}[Захист від масового призначення](/docs/{{version}}/eloquent#mass-assignment)автоматично вимикається під час засівання бази даних.

Як приклад, давайте змінимо за замовчуванням`DatabaseSeeder`клас і додайте оператор вставки бази даних до`run`метод:

    <?php

    namespace Database\Seeders;

    use Illuminate\Database\Seeder;
    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\Facades\Hash;
    use Illuminate\Support\Str;

    class DatabaseSeeder extends Seeder
    {
        /**
         * Run the database seeders.
         *
         * @return void
         */
        public function run()
        {
            DB::table('users')->insert([
                'name' => Str::random(10),
                'email' => Str::random(10).'@gmail.com',
                'password' => Hash::make('password'),
            ]);
        }
    }

> {tip} Ви можете ввести натяк на будь-які залежності, які вам потрібні в межах`run`підпис методу. Вони будуть автоматично вирішені через Laravel[службовий контейнер](/docs/{{version}}/container).

<a name="using-model-factories"></a>

### Використання модельних Factories

Звичайно, вручну вказувати атрибути для кожного Seeds моделі є громіздким. Натомість ви можете використовувати[модельні Factory](/docs/{{version}}/database-testing#writing-factories)для зручного генерування великої кількості записів бази даних. Спочатку перегляньте[типова Factoryька документація](/docs/{{version}}/database-testing#writing-factories)навчитися визначати свої Factory.

Наприклад, давайте створимо 50 користувачів і приєднаємо зв'язок з кожним користувачем:

    use App\Models\User;

    /**
     * Run the database seeders.
     *
     * @return void
     */
    public function run()
    {
        User::factory()
                ->times(50)
                ->hasPosts(1)
                ->create();
    }

<a name="calling-additional-seeders"></a>

### Виклик додаткових Seeders

У межах`DatabaseSeeder`клас, ви можете використовувати`call`метод для виконання додаткових класів Seeds. Використання`call`Метод дозволяє розбити базу даних на кілька файлів, щоб жоден клас посівної машини не став надзвичайно великим. Введіть назву класу сівалки, який ви хочете запустити:

    /**
     * Run the database seeders.
     *
     * @return void
     */
    public function run()
    {
        $this->call([
            UserSeeder::class,
            PostSeeder::class,
            CommentSeeder::class,
        ]);
    }

<a name="running-seeders"></a>

## Запуск Seeders

Ви можете використовувати`db:seed`Команда Artisan для створення вашої бази даних. За замовчуванням`db:seed`команда запускає`DatabaseSeeder`клас, який можна використовувати для виклику інших класів Seeds. Однак ви можете використовувати`--class`можливість вказати конкретний клас сівалки для запуску окремо:

    php artisan db:seed

    php artisan db:seed --class=UserSeeder

Ви також можете створити базу даних за допомогою`migrate:fresh`команда, яка скине всі таблиці та повторно виконає всі ваші міграції. Ця команда корисна для повного відновлення бази даних:

    php artisan migrate:fresh --seed

<a name="forcing-seeding-production"></a>

#### Примушення Seeders до запуску у виробництво

Деякі операції висіву можуть призвести до зміни чи втрати даних. Щоб захистити вас від запуску команд висіву проти вашої виробничої бази даних, вам буде запропоновано підтвердження перед тим, як сівалки будуть виконані. Щоб змусити сівалки працювати без підказки, використовуйте`--force`прапор:

    php artisan db:seed --force
