# Генерація URL-адрес

-   [Вступ](#introduction)
-   [Основи](#the-basics)
    -   [Створення базових URL-адрес](#generating-basic-urls)
    -   [Доступ до поточної URL-адреси](#accessing-the-current-url)
-   [URL-адреси для іменованих маршрутів](#urls-for-named-routes)
    -   [Підписані URL-адреси](#signed-urls)
-   [URL-адреси для дій контролера](#urls-for-controller-actions)
-   [Значення за замовчуванням](#default-values)

<a name="introduction"></a>

## Вступ

Laravel надає кілька помічників, які допоможуть вам створити URL-адреси для вашої програми. Вони в основному корисні при побудові посилань у ваших шаблонах та відповідях API або при генерації переадресаційних відповідей на іншу частину вашої програми.

<a name="the-basics"></a>

## Основи

<a name="generating-basic-urls"></a>

### Створення базових URL-адрес

`url`helper може використовуватися для створення довільних URL-адрес для вашої програми. Створена URL-адреса автоматично використовуватиме схему (HTTP або HTTPS) та хост із поточного запиту:

    $post = App\Models\Post::find(1);

    echo url("/posts/{$post->id}");

    // http://example.com/posts/1

<a name="accessing-the-current-url"></a>

### Доступ до поточної URL-адреси

Якщо шлях до`url`помічник, ан`Illuminate\Routing\UrlGenerator`повертається екземпляр, що дозволяє отримати доступ до інформації про поточну URL-адресу:

    // Get the current URL without the query string...
    echo url()->current();

    // Get the current URL including the query string...
    echo url()->full();

    // Get the full URL for the previous request...
    echo url()->previous();

Кожен із цих методів також може бути доступний через`URL`[фасад](/docs/{{version}}/facades):

    use Illuminate\Support\Facades\URL;

    echo URL::current();

<a name="urls-for-named-routes"></a>

## URL-адреси для іменованих маршрутів

`route`helper може використовуватися для генерації URL-адрес до названих маршрутів. Іменовані маршрути дозволяють створювати URL-адреси без прив’язки до фактичної URL-адреси, визначеної на маршруті. Отже, якщо URL-адреса маршруту змінюється, ніяких змін у вашій вносити не потрібно`route` function calls. For example, imagine your application contains a route defined like the following:

    Route::get('/post/{post}', function () {
        //
    })->name('post.show');

Щоб сформувати URL-адресу цього маршруту, ви можете використовувати`route`помічник ось так:

    echo route('post.show', ['post' => 1]);

    // http://example.com/post/1

Будь-які додаткові параметри масиву, які не відповідають параметрам визначення маршруту, будуть додані до рядка запиту URL-адреси:

    echo route('post.show', ['post' => 1, 'search' => 'rocket']);

    // http://example.com/post/1?search=rocket

Ви часто створюватимете URL-адреси за допомогою первинного ключа[Красномовні моделі](/docs/{{version}}/eloquent). З цієї причини ви можете передавати моделі Eloquent як значення параметрів.`route` helper will automatically extract the model's primary key:

    echo route('post.show', ['post' => $post]);

`route`helper також може використовуватися для генерації URL-адрес для маршрутів з декількома параметрами:

    Route::get('/post/{post}/comment/{comment}', function () {
        //
    })->name('comment.show');

    echo route('comment.show', ['post' => 1, 'comment' => 3]);

    // http://example.com/post/1/comment/3

<a name="signed-urls"></a>

### Підписані URL-адреси

Laravel дозволяє легко створювати "підписані" URL-адреси до названих маршрутів. Ці URL-адреси мають хеш "підпису", доданий до рядка запиту, що дозволяє Laravel перевірити, що URL-адреса не була змінена з моменту її створення. Підписані URL-адреси особливо корисні для маршрутів, які є загальнодоступними, проте потребують рівня захисту від маніпулювання URL-адресами.

Наприклад, ви можете використовувати підписані URL-адреси для реалізації загальнодоступного посилання "скасувати підписку", яке надсилається вашим клієнтам електронною поштою. Щоб створити підписану URL-адресу для названого маршруту, використовуйте`signedRoute`метод`URL`фасад:

    use Illuminate\Support\Facades\URL;

    return URL::signedRoute('unsubscribe', ['user' => 1]);

Якщо ви хочете створити тимчасову підписану URL-адресу маршруту, термін дії якої закінчується, ви можете використовувати`temporarySignedRoute`метод:

    use Illuminate\Support\Facades\URL;

    return URL::temporarySignedRoute(
        'unsubscribe', now()->addMinutes(30), ['user' => 1]
    );

<a name="validating-signed-route-requests"></a>

#### Перевірка підписаних запитів маршруту

Щоб переконатися, що вхідний запит має дійсний підпис, вам слід зателефонувати до`hasValidSignature`метод на вхідний`Request`:

    use Illuminate\Http\Request;

    Route::get('/unsubscribe/{user}', function (Request $request) {
        if (! $request->hasValidSignature()) {
            abort(401);
        }

        // ...
    })->name('unsubscribe');

Крім того, ви можете призначити`Illuminate\Routing\Middleware\ValidateSignature`Middlware до маршруту. Якщо його ще немає, слід призначити цьому проміжному програмному забезпеченню ключ у вашому ядрі HTTP`routeMiddleware`масив:

    /**
     * The application's route middleware.
     *
     * These middleware may be assigned to groups or used individually.
     *
     * @var array
     */
    protected $routeMiddleware = [
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
    ];

Після реєстрації проміжного програмного забезпечення у своєму ядрі ви можете додати його до маршруту. Якщо вхідний запит не має дійсного підпису, Middlware автоматично поверне a`403`відповідь на помилку:

    Route::post('/unsubscribe/{user}', function (Request $request) {
        // ...
    })->name('unsubscribe')->middleware('signed');

<a name="urls-for-controller-actions"></a>

## URL-адреси для дій контролера

`action`функція генерує URL-адресу для даної дії контролера:

    use App\Http\Controllers\HomeController;

    $url = action([HomeController::class, 'index']);

Якщо метод контролера приймає параметри маршруту, ви можете передати їх як другий аргумент функції:

    $url = action([UserController::class, 'profile'], ['id' => 1]);

<a name="default-values"></a>

## Значення за замовчуванням

У деяких програмах, можливо, ви захочете вказати значення за замовчуванням для певних параметрів URL-адрес. Наприклад, уявіть, що багато ваших маршрутів визначають a`{locale}`параметр:

    Route::get('/{locale}/posts', function () {
        //
    })->name('post.index');

Завжди здавати`locale`кожного разу, коли ви телефонуєте на`route`помічник. Отже, ви можете використовувати`URL::defaults`метод для визначення значення за замовчуванням для цього параметра, яке завжди застосовуватиметься під час поточного запиту. Можливо, ви захочете викликати цей метод із[маршрут проміжного програмного забезпечення](/docs/{{version}}/middleware#assigning-middleware-to-routes) so that you have access to the current request:

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Support\Facades\URL;

    class SetDefaultLocaleForUrls
    {
        public function handle($request, Closure $next)
        {
            URL::defaults(['locale' => $request->user()->locale]);

            return $next($request);
        }
    }

Одного разу значення за замовчуванням для`locale`Параметр встановлено, вам більше не потрібно передавати його значення при генерації URL-адрес через`route`помічник.

<a name="url-defaults-middleware-priority"></a>

#### URL за замовчуванням та пріоритет проміжного програмного забезпечення

Встановлення значень за замовчуванням URL може перешкодити обробці Laravel неявних прив'язок моделі. Тому вам слід[розставити пріоритети на Middlware](https://laravel.com/docs/{{version}}/middleware#sorting-middleware)що встановлює за замовчуванням URL-адреси, які будуть виконуватися перед власними Laravel`SubstituteBindings`Middlware. Ви можете досягти цього, переконавшись, що ваше Middlware відбувається до`SubstituteBindings`Middlware в`$middlewarePriority`властивість ядра HTTP вашої програми.

`$middlewarePriority`властивість визначається в основі`Illuminate\Foundation\Http\Kernel`клас. Ви можете скопіювати його визначення з цього класу та перезаписати його в ядро ​​HTTP програми, щоб змінити його:

    /**
     * The priority-sorted list of middleware.
     *
     * This forces non-global middleware to always be in the given order.
     *
     * @var array
     */
    protected $middlewarePriority = [
        // ...
         \App\Http\Middleware\SetDefaultLocaleForUrls::class,
         \Illuminate\Routing\Middleware\SubstituteBindings::class,
         // ...
    ];
