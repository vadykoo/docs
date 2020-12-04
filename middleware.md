# Middleware

-   [Вступ](#introduction)
-   [Визначення проміжного програмного забезпечення](#defining-middleware)
-   [Реєстрація проміжного програмного забезпечення](#registering-middleware)
    -   [Глобальне ПЗ](#global-middleware)
    -   [Призначення проміжного програмного забезпечення маршрутам](#assigning-middleware-to-routes)
    -   [Групи проміжного програмного забезпечення](#middleware-groups)
    -   [Сортування проміжного програмного забезпечення](#sorting-middleware)
-   [Параметри проміжного програмного забезпечення](#middleware-parameters)
-   [Можливе проміжне програмне забезпечення](#terminable-middleware)

<a name="introduction"></a>

## Вступ

Middleware забезпечує зручний механізм фільтрації HTTP-запитів, що надходять у вашу програму. Наприклад, Laravel включає проміжне програмне забезпечення, яке підтверджує автентичність користувача вашого додатка. Якщо користувач не аутентифікований, проміжне програмне забезпечення перенаправить користувача на екран входу. Однак, якщо користувача аутентифіковано, проміжне програмне забезпечення дозволить запиту перейти далі до програми.

Можна створити додаткове проміжне програмне забезпечення для виконання різноманітних завдань, крім автентифікації. Middleware CORS може бути відповідальним за додавання відповідних заголовків до всіх відповідей, що залишають вашу програму. Middleware для реєстрації може реєструвати всі вхідні запити до вашої програми.

У фреймворк Laravel входить кілька проміжних програм, включаючи проміжне програмне забезпечення для автентифікації та захисту CSRF. Всі ці проміжні програми знаходяться в`app/Http/Middleware`каталог.

<a name="defining-middleware"></a>

## Визначення проміжного програмного забезпечення

Щоб створити нове проміжне програмне забезпечення, використовуйте`make:middleware`Команда ремісників:

    php artisan make:middleware CheckAge

Ця команда розмістить нову`CheckAge`клас у вашому`app/Http/Middleware`каталог. У цьому проміжному програмному забезпеченні ми дозволимо доступ до маршруту лише за умови, що надано`age`більше 200. В іншому випадку ми перенаправляємо користувачів назад на`home`Ненависть:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class CheckAge
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            if ($request->age <= 200) {
                return redirect('home');
            }

            return $next($request);
        }
    }

Як бачите, якщо дане`age`менше або дорівнює`200`, проміжне програмне забезпечення поверне клієнту перенаправлення HTTP; в іншому випадку запит буде передано далі в заявку. Щоб передати запит глибше в програму (дозволивши проміжне програмне забезпечення "пройти"), зателефонуйте на`$next`зворотний дзвінок за допомогою`$request`.

Краще уявити проміжне програмне забезпечення, оскільки низка "шарів" HTTP-запитів повинна пройти, перш ніж вони потраплять у вашу програму. Кожен шар може розглянути запит і навіть повністю відхилити його.

> {tip} Усі проміжні програми вирішуються через[службовий контейнер](/docs/{{version}}/container), тож ви можете навести натяк на будь-які залежності, які вам потрібні в конструкторі проміжного програмного забезпечення.

<a name="before-after-middleware"></a>

#### До та після проміжного програмного забезпечення

Запуск проміжного програмного забезпечення до або після запиту залежить від самого проміжного програмного забезпечення. Наприклад, наступне проміжне програмне забезпечення виконує певне завдання**раніше**запит обробляється додатком:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class BeforeMiddleware
    {
        public function handle($request, Closure $next)
        {
            // Perform action

            return $next($request);
        }
    }

Однак це проміжне програмне забезпечення виконало б своє завдання**після**запит обробляється додатком:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class AfterMiddleware
    {
        public function handle($request, Closure $next)
        {
            $response = $next($request);

            // Perform action

            return $response;
        }
    }

<a name="registering-middleware"></a>

## Реєстрація проміжного програмного забезпечення

<a name="global-middleware"></a>

### Глобальне ПЗ

Якщо ви хочете, щоб проміжне програмне забезпечення працювало під час кожного запиту HTTP до вашої програми, перелічіть клас проміжного програмного забезпечення в`$middleware`власність вашого`app/Http/Kernel.php`клас.

<a name="assigning-middleware-to-routes"></a>

### Призначення проміжного програмного забезпечення маршрутам

Якщо ви хочете призначити проміжне програмне забезпечення для певних маршрутів, спочатку слід призначити проміжне програмне забезпечення ключем у вашому`app/Http/Kernel.php`файл. За замовчуванням`$routeMiddleware`властивість цього класу містить записи проміжного програмного забезпечення, що входить до складу Laravel. Щоб додати свій власний, додайте його до цього списку та призначте йому вибраний вами ключ:

    // Within App\Http\Kernel Class...

    protected $routeMiddleware = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
        'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
    ];

Після того, як проміжне програмне забезпечення буде визначено в ядрі HTTP, ви можете використовувати`middleware`метод призначення маршрутного проміжного програмного забезпечення:

    Route::get('admin/profile', function () {
        //
    })->middleware('auth');

Ви також можете призначити маршруту кілька проміжних програм:

    Route::get('/', function () {
        //
    })->middleware('first', 'second');

Призначаючи проміжне програмне забезпечення, ви також можете передати повну назву класу:

    use App\Http\Middleware\CheckAge;

    Route::get('admin/profile', function () {
        //
    })->middleware(CheckAge::class);

Призначаючи проміжне програмне забезпечення групі маршрутів, іноді може знадобитися запобігти застосуванню проміжного програмного забезпечення до окремого маршруту в групі. Ви можете досягти цього за допомогою`withoutMiddleware`метод:

    use App\Http\Middleware\CheckAge;

    Route::middleware([CheckAge::class])->group(function () {
        Route::get('/', function () {
            //
        });

        Route::get('admin/profile', function () {
            //
        })->withoutMiddleware([CheckAge::class]);
    });

`withoutMiddleware`метод може видалити лише проміжне програмне забезпечення маршруту і не застосовується до[глобальне проміжне програмне забезпечення](#global-middleware).

<a name="middleware-groups"></a>

### Групи проміжного програмного забезпечення

Іноді вам може знадобитися згрупувати кілька проміжних програм за допомогою однієї клавіші, щоб полегшити їх призначення для маршрутів. Ви можете зробити це за допомогою`$middlewareGroups`властивість вашого ядра HTTP.

Нестандартно приходить Laravel`web`і`api`групи проміжного програмного забезпечення, що містять загальне проміжне програмне забезпечення, яке ви можете застосувати до свого веб-інтерфейсу та маршрутів API:

    /**
     * The application's route middleware groups.
     *
     * @var array
     */
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            // \Illuminate\Session\Middleware\AuthenticateSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],

        'api' => [
            'throttle:api',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],
    ];

Групи проміжного програмного забезпечення можуть бути призначені для маршрутів та дій контролера, використовуючи той самий синтаксис, що і окремі проміжні програми. Знову ж таки, проміжні групи роблять зручнішим призначати багато проміжних програм до маршруту одночасно:

    Route::get('/', function () {
        //
    })->middleware('web');

    Route::group(['middleware' => ['web']], function () {
        //
    });

    Route::middleware(['web', 'subscribed'])->group(function () {
        //
    });

> {tip} Нестандартно,`web`група проміжного програмного забезпечення автоматично застосовується до вашого`routes/web.php`файл`RouteServiceProvider`.

<a name="sorting-middleware"></a>

### Сортування проміжного програмного забезпечення

Рідко вам може знадобитися проміжне програмне забезпечення для виконання в певному порядку, але не мати контролю над своїм замовленням, коли вони призначені маршруту. У цьому випадку ви можете вказати пріоритет проміжного програмного забезпечення за допомогою`$middlewarePriority`власність вашого`app/Http/Kernel.php`файл:

    /**
     * The priority-sorted list of middleware.
     *
     * This forces non-global middleware to always be in the given order.
     *
     * @var array
     */
    protected $middlewarePriority = [
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Contracts\Auth\Middleware\AuthenticatesRequests::class,
        \Illuminate\Routing\Middleware\ThrottleRequests::class,
        \Illuminate\Session\Middleware\AuthenticateSession::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \Illuminate\Auth\Middleware\Authorize::class,
    ];

<a name="middleware-parameters"></a>

## Параметри проміжного програмного забезпечення

Middleware також може отримувати додаткові параметри. Наприклад, якщо вашій програмі потрібно виконати перевірку, що автентифікований користувач має задану "роль" перед виконанням певної дії, ви можете створити`CheckRole`проміжне програмне забезпечення, яке отримує ім'я ролі як додатковий аргумент.

Додаткові параметри проміжного програмного забезпечення будуть передані проміжному програмному забезпеченню після`$next`аргумент:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class CheckRole
    {
        /**
         * Handle the incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @param  string  $role
         * @return mixed
         */
        public function handle($request, Closure $next, $role)
        {
            if (! $request->user()->hasRole($role)) {
                // Redirect...
            }

            return $next($request);
        }

    }

Параметри проміжного програмного забезпечення можуть бути вказані при визначенні маршруту, відокремлюючи ім'я та параметри проміжного програмного забезпечення за допомогою a`:`. Кілька параметрів слід розділяти комами:

    Route::put('post/{id}', function ($id) {
        //
    })->middleware('role:editor');

<a name="terminable-middleware"></a>

## Можливе проміжне програмне забезпечення

Іноді проміжне програмне забезпечення, можливо, доведеться виконати якусь роботу після того, як відповідь HTTP надіслано браузеру. Якщо ви визначите`terminate`на проміжному програмному забезпеченні, а веб-сервер використовує FastCGI,`terminate`метод буде автоматично викликаний після того, як відповідь буде надіслана браузеру:

    <?php

    namespace Illuminate\Session\Middleware;

    use Closure;

    class StartSession
    {
        public function handle($request, Closure $next)
        {
            return $next($request);
        }

        public function terminate($request, $response)
        {
            // Store the session data...
        }
    }

`terminate`метод повинен отримувати як запит, так і відповідь. Після того, як ви визначили проміжне програмне забезпечення, яке можна термінувати, вам слід додати його до списку маршрутних або глобальних проміжних програм у`app/Http/Kernel.php`файл.

При дзвінку в`terminate`для вашого проміжного програмного забезпечення, Laravel вирішить новий екземпляр проміжного програмного забезпечення з[службовий контейнер](/docs/{{version}}/container). Якщо ви хочете використовувати той самий екземпляр проміжного програмного забезпечення, коли файл`handle`і`terminate`викликаються методи, зареєструйте проміжне програмне забезпечення з контейнером, використовуючи контейнери`singleton`метод. Зазвичай це слід робити в`register`метод вашого`AppServiceProvider.php`:

    use App\Http\Middleware\TerminableMiddleware;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(TerminableMiddleware::class);
    }
