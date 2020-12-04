# Постачальники послуг

-   [Вступ](#introduction)
-   [Написання послуг постачальниками](#writing-service-providers)
    -   [Метод реєстрації](#the-register-method)
    -   [Метод завантаження](#the-boot-method)
-   [Реєстрація постачальників](#registering-providers)
-   [Відстрочені постачальники](#deferred-providers)

<a name="introduction"></a>

## Вступ

Постачальники послуг є центральним місцем усього завантаження програм Laravel. Ваша власна програма, а також усі основні послуги Laravel завантажуються через постачальників послуг.

Але що ми маємо на увазі під "завантаженим"? Загалом ми маємо на увазі**реєстрація**речі, включаючи реєстрацію прив’язок контейнерів служб, прослуховувачів подій, проміжне програмне забезпечення та навіть маршрути. Постачальники послуг є центральним місцем для налаштування вашої програми.

Якщо ви відкриєте`config/app.php`файл, що входить до складу Laravel, ви побачите файл`providers`масив. Це всі класи постачальника послуг, які будуть завантажені для вашої програми. Зверніть увагу, що багато з них є "відкладеними" провайдерами, тобто вони не завантажуються при кожному запиті, а лише тоді, коли послуги, які вони надають, насправді потрібні.

У цьому огляді ви дізнаєтеся, як писати власних постачальників послуг та реєструвати їх у своїй програмі Laravel.

<a name="writing-service-providers"></a>

## Написання послуг постачальниками

Усі постачальники послуг розширюють`Illuminate\Support\ServiceProvider`клас. Більшість постачальників послуг містять`register`і a`boot`метод. У межах`register`метод, ви повинні**лише прив'язувати речі до[службовий контейнер](/docs/{{version}}/container)**. Ви ніколи не повинні намагатися зареєструвати будь-які прослуховувачі подій, маршрути чи будь-яку іншу функціональну можливість у межах`register`метод.

CLIS Artisan може створити нового постачальника через`make:provider`команда:

    php artisan make:provider RiakServiceProvider

<a name="the-register-method"></a>

### Метод реєстрації

Як зазначалося раніше, в межах`register`метод, вам слід лише прив'язувати речі до[службовий контейнер](/docs/{{version}}/container). Ви ніколи не повинні намагатися зареєструвати будь-які прослуховувачі подій, маршрути чи будь-яку іншу функціональну можливість у межах`register`метод. В іншому випадку ви можете випадково використовувати послугу, яку надає постачальник послуг, який ще не завантажений.

Давайте поглянемо на основного постачальника послуг. У будь-якому з методів вашого постачальника послуг ви завжди маєте доступ до`$app`властивість, що забезпечує доступ до контейнера служби:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Riak\Connection;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection(config('riak'));
            });
        }
    }

Цей постачальник послуг визначає лише a`register`і використовує цей метод для визначення реалізації`Riak\Connection`в контейнері для обслуговування. Якщо ви не розумієте, як працює службовий контейнер, перевірте[його документація](/docs/{{version}}/container).

<a name="the-bindings-and-singletons-properties"></a>

#### `bindings`І`singletons`Властивості

Якщо ваш постачальник послуг реєструє багато простих прив'язок, можливо, ви захочете використовувати`bindings`і`singletons`властивості, а не вручну реєструвати кожну прив'язку контейнера. Коли постачальник послуг завантажується фреймворком, він автоматично перевіряє наявність цих властивостей та реєструє їх прив’язки:

    <?php

    namespace App\Providers;

    use App\Contracts\DowntimeNotifier;
    use App\Contracts\ServerProvider;
    use App\Services\DigitalOceanServerProvider;
    use App\Services\PingdomDowntimeNotifier;
    use App\Services\ServerToolsProvider;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * All of the container bindings that should be registered.
         *
         * @var array
         */
        public $bindings = [
            ServerProvider::class => DigitalOceanServerProvider::class,
        ];

        /**
         * All of the container singletons that should be registered.
         *
         * @var array
         */
        public $singletons = [
            DowntimeNotifier::class => PingdomDowntimeNotifier::class,
            ServerProvider::class => ServerToolsProvider::class,
        ];
    }

<a name="the-boot-method"></a>

### Метод завантаження

Отже, що, якщо нам потрібно зареєструвати a[погляд композитора](/docs/{{version}}/views#view-composers)у нашого постачальника послуг? Це слід робити в межах`boot`метод.**Цей метод викликається після реєстрації всіх інших постачальників послуг**, тобто ви маєте доступ до всіх інших служб, які були зареєстровані в рамках:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            view()->composer('view', function () {
                //
            });
        }
    }

<a name="boot-method-dependency-injection"></a>

#### Ін'єкція залежності методу завантаження

Ви можете ввести натяк на залежності для постачальника послуг`boot`метод.[службовий контейнер](/docs/{{version}}/container) will automatically inject any dependencies you need:

    use Illuminate\Contracts\Routing\ResponseFactory;

    public function boot(ResponseFactory $response)
    {
        $response->macro('caps', function ($value) {
            //
        });
    }

<a name="registering-providers"></a>

## Реєстрація постачальників

Усі постачальники послуг зареєстровані в`config/app.php`файл конфігурації. Цей файл містить`providers`масив, де ви можете перерахувати імена класів своїх постачальників послуг. За замовчуванням у цьому масиві перелічено набір основних постачальників послуг Laravel. Ці постачальники завантажують основні компоненти Laravel, такі як пошта, черга, кеш-пам'ять та інші.

Щоб зареєструвати свого постачальника, додайте його до масиву:

    'providers' => [
        // Other Service Providers

        App\Providers\ComposerServiceProvider::class,
    ],

<a name="deferred-providers"></a>

## Відстрочені постачальники

Якщо ваш постачальник послуг**тільки**реєстрація прив'язок в[службовий контейнер](/docs/{{version}}/container), Ви можете відкласти його реєстрацію доти, доки насправді не буде потрібно одне із зареєстрованих прив'язок. Відкладання завантаження такого постачальника покращить продуктивність вашої програми, оскільки вона не завантажується з файлової системи на кожен запит.

Laravel складає та зберігає перелік усіх послуг, що надаються відкладеними постачальниками послуг, разом із назвою класу свого постачальника послуг. Тоді лише при спробі вирішити одну з цих служб Laravel завантажує постачальника послуг.

Щоб відкласти завантаження провайдера, застосуйте`\Illuminate\Contracts\Support\DeferrableProvider`інтерфейс та визначте a`provides`метод.`provides`метод повинен повертати прив'язки контейнера служби, зареєстровані постачальником:

    <?php

    namespace App\Providers;

    use Illuminate\Contracts\Support\DeferrableProvider;
    use Illuminate\Support\ServiceProvider;
    use Riak\Connection;

    class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
    {
        /**
         * Register any application services.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection($app['config']['riak']);
            });
        }

        /**
         * Get the services provided by the provider.
         *
         * @return array
         */
        public function provides()
        {
            return [Connection::class];
        }
    }
