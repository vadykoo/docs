# Контейнер послуг

-   [Вступ](#introduction)
-   [Обв'язування](#binding)
    -   [Основи прив’язки](#binding-basics)
    -   [Прив'язка інтерфейсів до реалізацій](#binding-interfaces-to-implementations)
    -   [Контекстне прив’язування](#contextual-binding)
    -   [Пов'язувальні примітиви](#binding-primitives)
    -   [Зв'язування набраних варіадик](#binding-typed-variadics)
    -   [Позначення](#tagging)
    -   [Розширення прив’язок](#extending-bindings)
-   [Вирішуючи](#resolving)
    -   [Метод Make](#the-make-method)
    -   [Автоматична ін’єкція](#automatic-injection)
-   [Контейнерні події](#container-events)
-   [PSR-11](#psr-11)

<a name="introduction"></a>

## Вступ

Контейнер послуг Laravel - це потужний інструмент для управління залежностями класу та виконання ін'єкції залежностей. Введення залежностей - це вигадана фраза, яка по суті означає це: залежності класу "вводяться" в клас за допомогою конструктора або, в деяких випадках, методів "встановлення".

Давайте розглянемо простий приклад:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Repositories\UserRepository;
    use App\Models\User;

    class UserController extends Controller
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            $user = $this->users->find($id);

            return view('user.profile', ['user' => $user]);
        }
    }

У цьому прикладі`UserController`потрібно отримати користувачів із джерела даних. Отже, ми будемо**вводити** a service that is able to retrieve users. In this context, our `UserRepository`швидше за все використовує[Красномовний](/docs/{{version}}/eloquent)для отримання інформації про користувача з бази даних. Однак, оскільки сховище вводиться, ми можемо легко поміняти його іншою реалізацією. Ми також можемо легко "висміяти" або створити фіктивну реалізацію`UserRepository`під час тестування вашої програми.

Глибоке розуміння сервісного контейнера Laravel має важливе значення для створення потужного, великого додатка, а також для сприяння самому ядру Laravel.

<a name="binding"></a>

## Обв'язування

<a name="binding-basics"></a>

### Основи прив’язки

Майже всі прив'язки вашого контейнера служб будуть зареєстровані всередині[постачальників послуг](/docs/{{version}}/providers), тому більшість з цих прикладів продемонструють використання контейнера в цьому контексті.

> {tip} Немає необхідності прив’язувати класи до контейнера, якщо вони не залежать від будь-якого інтерфейсу. Контейнеру не потрібно вказувати, як будувати ці об'єкти, оскільки він може автоматично вирішувати ці об'єкти за допомогою відображення.

<a name="simple-bindings"></a>

#### Прості прив’язки

У постачальника послуг ви завжди маєте доступ до контейнера через`$this->app`майно. Ми можемо зареєструвати прив'язку за допомогою`bind`метод, передаючи ім'я класу або інтерфейсу, яке ми хочемо зареєструвати разом із`Closure`що повертає екземпляр класу:

    $this->app->bind('HelpSpot\API', function ($app) {
        return new \HelpSpot\API($app->make('HttpClient'));
    });

Зверніть увагу, що ми отримуємо сам контейнер як аргумент для розв'язувача. Потім ми можемо використовувати контейнер для вирішення підзалежностей об’єкта, який ми будуємо.

<a name="binding-a-singleton"></a>

#### Binding A Singleton

`singleton`метод прив'язує клас або інтерфейс до контейнера, які слід вирішувати лише один раз. Як тільки одностороннє прив'язування буде вирішено, той самий екземпляр об'єкта буде повернутий при наступних викликах до контейнера:

    $this->app->singleton('HelpSpot\API', function ($app) {
        return new \HelpSpot\API($app->make('HttpClient'));
    });

<a name="binding-instances"></a>

#### Примірники

Ви також можете прив'язати наявний екземпляр об'єкта до контейнера за допомогою`instance`метод. Даний екземпляр завжди повертається під час наступних викликів у контейнер:

    $api = new \HelpSpot\API(new HttpClient);

    $this->app->instance('HelpSpot\API', $api);

<a name="binding-interfaces-to-implementations"></a>

### Прив'язка інтерфейсів до реалізацій

Дуже потужною функцією службового контейнера є його здатність прив'язувати інтерфейс до даної реалізації. Наприклад, припустимо, що ми маємо`EventPusher`інтерфейс та a`RedisEventPusher`впровадження. Після того, як ми закодували наш`RedisEventPusher`реалізації цього інтерфейсу, ми можемо зареєструвати його в контейнері служб так:

    $this->app->bind(
        'App\Contracts\EventPusher',
        'App\Services\RedisEventPusher'
    );

Це твердження повідомляє контейнеру, що він повинен вводити файл`RedisEventPusher`коли клас потребує реалізації`EventPusher`. Тепер ми можемо ввести натяк на`EventPusher`інтерфейс у конструкторі або будь-якому іншому місці, де залежності вводяться контейнером служби:

    use App\Contracts\EventPusher;

    /**
     * Create a new class instance.
     *
     * @param  EventPusher  $pusher
     * @return void
     */
    public function __construct(EventPusher $pusher)
    {
        $this->pusher = $pusher;
    }

<a name="contextual-binding"></a>

### Контекстне прив’язування

Іноді у вас можуть бути два класи, які використовують один і той же інтерфейс, але ви хочете вводити різні реалізації в кожен клас. Наприклад, два контролери можуть залежати від різних реалізацій`Illuminate\Contracts\Filesystem\Filesystem`[контракт](/docs/{{version}}/contracts). Laravel пропонує простий, вільний інтерфейс для визначення такої поведінки:

    use App\Http\Controllers\PhotoController;
    use App\Http\Controllers\UploadController;
    use App\Http\Controllers\VideoController;
    use Illuminate\Contracts\Filesystem\Filesystem;
    use Illuminate\Support\Facades\Storage;

    $this->app->when(PhotoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('local');
              });

    $this->app->when([VideoController::class, UploadController::class])
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('s3');
              });

<a name="binding-primitives"></a>

### Пов'язувальні примітиви

Іноді у вас може бути клас, який отримує деякі введені класи, але також потребує введеного примітивного значення, такого як ціле число. Ви можете легко використовувати контекстне прив’язування для введення будь-якого значення, яке може знадобитися вашому класу:

    $this->app->when('App\Http\Controllers\UserController')
              ->needs('$variableName')
              ->give($value);

Іноді клас може залежати від масиву позначених екземплярів. Використання`giveTagged`методом, ви можете легко ввести всі прив'язки контейнера з цим тегом:

    $this->app->when(ReportAggregator::class)
        ->needs('$reports')
        ->giveTagged('reports');

<a name="binding-typed-variadics"></a>

### Зв'язування набраних варіадик

Іноді у вас може бути клас, який отримує масив набраних об'єктів, використовуючи аргумент варіадичного конструктора:

    class Firewall
    {
        protected $logger;
        protected $filters;

        public function __construct(Logger $logger, Filter ...$filters)
        {
            $this->logger = $logger;
            $this->filters = $filters;
        }
    }

Використовуючи контекстне прив'язування, ви можете вирішити цю залежність, надавши`give`метод із закриттям, що повертає масив вирішених`Filter`екземпляри:

    $this->app->when(Firewall::class)
              ->needs(Filter::class)
              ->give(function ($app) {
                    return [
                        $app->make(NullFilter::class),
                        $app->make(ProfanityFilter::class),
                        $app->make(TooLongFilter::class),
                    ];
              });

Для зручності ви також можете просто надати масив імен класів, які контейнер повинен вирішувати щоразу`Firewall`потреби`Filter`екземпляри:

    $this->app->when(Firewall::class)
              ->needs(Filter::class)
              ->give([
                  NullFilter::class,
                  ProfanityFilter::class,
                  TooLongFilter::class,
              ]);

<a name="variadic-tag-dependencies"></a>

#### Варіадичні залежності тегів

Іноді клас може мати варіатичну залежність, яка натякається на тип як даний клас (`Report ...$reports`). Використання`needs`і`giveTagged`методів, ви можете легко ввести всі прив'язки контейнера з цим тегом для даної залежності:

    $this->app->when(ReportAggregator::class)
        ->needs(Report::class)
        ->giveTagged('reports');

<a name="tagging"></a>

### Позначення

Іноді вам може знадобитися вирішити всі певні "категорії" прив'язки. Наприклад, можливо, ви створюєте агрегатор звітів, який отримує безліч різноманітних`Report`реалізації інтерфейсу. Після реєстрації`Report`реалізації, ви можете призначити їм тег за допомогою`tag`метод:

    $this->app->bind('SpeedReport', function () {
        //
    });

    $this->app->bind('MemoryReport', function () {
        //
    });

    $this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

Після позначення служб ви можете легко вирішити їх усі через`tagged`метод:

    $this->app->bind('ReportAggregator', function ($app) {
        return new ReportAggregator($app->tagged('reports'));
    });

<a name="extending-bindings"></a>

### Розширення прив’язок

`extend`метод дозволяє модифікувати вирішені послуги. Наприклад, коли послуга вирішена, ви можете запустити додатковий код для декорування або налаштування служби.`extend`Метод приймає Closure, який повинен повернути змінену службу як єдиний аргумент. Закриття отримує вирішувану послугу та екземпляр контейнера:

    $this->app->extend(Service::class, function ($service, $app) {
        return new DecoratedService($service);
    });

<a name="resolving"></a>

## Вирішуючи

<a name="the-make-method"></a>

#### `make`Метод

Ви можете використовувати`make`метод для вирішення екземпляра класу з контейнера.`make`метод приймає ім'я класу або інтерфейсу, який ви хочете вирішити:

    $api = $this->app->make('HelpSpot\API');

Якщо ви знаходитесь у місці свого коду, яке не має доступу до`$app`змінної, ви можете використовувати глобальну`resolve`помічник:

    $api = resolve('HelpSpot\API');

Якщо деякі залежності вашого класу не можна вирішити за допомогою контейнера, ви можете ввести їх, передавши їх як асоціативний масив до`makeWith`метод:

    $api = $this->app->makeWith('HelpSpot\API', ['id' => 1]);

<a name="automatic-injection"></a>

#### Автоматична ін’єкція

Як варіант, і що важливо, ви можете "натякнути" на залежність у конструкторі класу, який вирішується контейнером, включаючи[контролери](/docs/{{version}}/controllers),[слухачі подій](/docs/{{version}}/events),[проміжне програмне забезпечення](/docs/{{version}}/middleware), і більше. Крім того, ви можете вказати залежності типу`handle`метод[робочі місця в черзі](/docs/{{version}}/queues). На практиці саме так контейнер повинен вирішувати більшість ваших об’єктів.

Наприклад, ви можете ввести натяк на сховище, визначене вашим додатком, у конструкторі контролера. Сховище буде автоматично вирішено та введено в клас:

    <?php

    namespace App\Http\Controllers;

    use App\Models\Users\Repository as UserRepository;

    class UserController extends Controller
    {
        /**
         * The user repository instance.
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Show the user with the given ID.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            //
        }
    }

<a name="container-events"></a>

## Контейнерні події

Контейнер служби запускає подію кожного разу, коли він вирішує об'єкт. Ви можете прослухати цю подію за допомогою`resolving`метод:

    $this->app->resolving(function ($object, $app) {
        // Called when container resolves object of any type...
    });

    $this->app->resolving(\HelpSpot\API::class, function ($api, $app) {
        // Called when container resolves objects of type "HelpSpot\API"...
    });

Як бачите, вирішуваний об’єкт буде передано зворотному виклику, що дозволить встановити будь-які додаткові властивості об’єкта до того, як він буде наданий його споживачеві.

<a name="psr-11"></a>

## PSR-11

Службовий контейнер Laravel реалізує[PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md)інтерфейс. Таким чином, ви можете натякнути на інтерфейс контейнера PSR-11, щоб отримати екземпляр контейнера Laravel:

    use Psr\Container\ContainerInterface;

    Route::get('/', function (ContainerInterface $container) {
        $service = $container->get('Service');

        //
    });

Викликається виняток, якщо заданий ідентифікатор не вдається вирішити. Винятком буде екземпляр`Psr\Container\NotFoundExceptionInterface`якщо ідентифікатор ніколи не був прив’язаний. Якщо ідентифікатор був зв’язаний, але не вдалося вирішити, екземпляр`Psr\Container\ContainerExceptionInterface`буде кинуто.
