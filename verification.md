# Підтвердження електронної пошти

-   [Вступ](#introduction)
    -   [Підготовка моделі](#model-preparation)
    -   [Підготовка бази даних](#database-preparation)
-   [Routing](#verification-routing)
    -   [Повідомлення про підтвердження електронної пошти](#the-email-verification-notice)
    -   [Обробник підтвердження електронної пошти](#the-email-verification-handler)
    -   [Повторна відправка електронного листа для підтвердження](#resending-the-verification-email)
    -   [Захист маршрутів](#protecting-routes)
-   [Події](#events)

<a name="introduction"></a>

## Вступ

Багато веб-додатків вимагають від користувачів підтвердження своїх електронних адрес перед використанням програми. Замість того, щоб змушувати вас повторно впроваджувати це в кожній програмі, Laravel пропонує зручні методи надсилання та перевірки запитів на перевірку електронної пошти.

> {tip} Хочете швидко розпочати? Встановити[Laravel Jetstream](https://jetstream.laravel.com)у свіжому додатку Laravel. Після міграції бази даних перейдіть до браузера`/register`або будь-яку іншу URL-адресу, призначену вашій програмі. Jetstream подбає про побудову всієї вашої системи автентифікації, включаючи підтримку перевірки електронної пошти!

<a name="model-preparation"></a>

### Підготовка моделі

Для початку переконайтеся, що ваш`App\Models\User`модель реалізує`Illuminate\Contracts\Auth\MustVerifyEmail`контракт:

    <?php

    namespace App\Models;

    use Illuminate\Contracts\Auth\MustVerifyEmail;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable implements MustVerifyEmail
    {
        use Notifiable;

        // ...
    }

Як тільки цей інтерфейс буде додано до вашої моделі, нещодавно зареєстрованим користувачам автоматично буде надіслано електронне повідомлення із посиланням для підтвердження електронної пошти. Як ви можете бачити, вивчивши ваш`EventServiceProvider`, Laravel вже містить a`SendEmailVerificationNotification`[слухач](/docs/{{version}}/events)що додається до`Illuminate\Auth\Events\Registered`подія.

Якщо ви вручну здійснюєте реєстрацію в своїй заявці, а не використовуєте[Laravel Jetstream](https://jetstream.laravel.com), ви повинні переконатися, що ви відправляєте`Illuminate\Auth\Events\Registered`подія після успішної реєстрації користувача:

    use Illuminate\Auth\Events\Registered;

    event(new Registered($user));

<a name="database-preparation"></a>

### Підготовка бази даних

Далі, ваш`user`таблиця повинна містити`email_verified_at`для збереження дати та часу підтвердження адреси електронної пошти. За замовчуванням`users`міграція таблиці, що входить до фреймворку Laravel, уже містить цей стовпець. Отже, все, що вам потрібно зробити, це запустити міграції бази даних:

    php artisan migrate

<a name="verification-routing"></a>

## Routing

Щоб правильно здійснити перевірку електронної пошти, потрібно визначити три маршрути. По-перше, потрібен маршрут для відображення повідомлення користувачеві про те, що він повинен натиснути посилання для підтвердження електронної пошти в електронному листі для підтвердження, який Laravel надіслав їм після реєстрації. По-друге, потрібен буде маршрут для обробки запитів, що генеруються, коли користувач натискає посилання для підтвердження електронної пошти в електронному листі. По-третє, потрібен буде маршрут для повторної відправки посилання для підтвердження, якщо користувач випадково втратить перший.

<a name="the-email-verification-notice"></a>

### Повідомлення про підтвердження електронної пошти

Як вже згадувалося раніше, слід визначити маршрут, який поверне View з вказівкою користувачеві натиснути посилання для підтвердження електронної пошти, надіслане їм електронною поштою від Laravel. Це представлення відображатиметься користувачам, коли вони намагатимуться отримати доступ до інших частин програми, не попередньо перевіривши свою електронну адресу. Пам’ятайте, що посилання автоматично надсилається користувачеві, поки ваш`App\Models\User`модель реалізує`MustVerifyEmail`інтерфейс:

    Route::get('/email/verify', function () {
        return view('auth.verify-email');
    })->middleware(['auth'])->name('verification.notice');

Маршрут, який повертає повідомлення про перевірку електронної пошти, повинен бути названий`verification.notice`. Важливо, щоб маршруту було присвоєно саме цю назву, оскільки`verified`проміжне програмне забезпечення[входить до складу Laravel](#protecting-routes)автоматично перенаправить на цю назву маршруту, якщо користувач не підтвердив свою адресу електронної пошти.

> {tip} При ручному впровадженні перевірки електронної пошти вам потрібно самостійно визначити вміст View повідомлення про перевірку. Якщо вам потрібні будівельні ліси, що включають усі необхідні View автентифікації та перевірки, відвідайте[Laravel Jetstream](https://jetstream.laravel.com).

<a name="the-email-verification-handler"></a>

### Обробник підтвердження електронної пошти

Далі нам потрібен маршрут, який оброблятиме запити, що генеруються, коли користувач натискає посилання для підтвердження електронної пошти, яке було надіслане йому електронною поштою. Цей маршрут слід назвати`verification.verify`і присвоїти`auth`і`signed`проміжні засоби:

    use Illuminate\Foundation\Auth\EmailVerificationRequest;
    use Illuminate\Http\Request;

    Route::get('/email/verify/{id}/{hash}', function (EmailVerificationRequest $request) {
        $request->fulfill();

        return redirect('/home');
    })->middleware(['auth', 'signed'])->name('verification.verify');

Перш ніж рухатися далі, давайте детальніше розглянемо цей маршрут. По-перше, ви помітите, що ми використовуємо`EmailVerificationRequest`тип запиту замість типового`Illuminate\Http\Request`екземпляр.`EmailVerificationRequest`є[запит форми](/docs/{{version}}/validation#form-request-validation)що входить до складу Laravel. Цей запит подбає про автоматичну перевірку запиту`id`і`hash`параметри.

Далі ми можемо перейти безпосередньо до виклику`fulfill`метод за запитом. Цей метод буде викликати файл`markEmailAsVerified`метод для автентифікованого користувача та відправити файл`Illuminate\Auth\Events\Verified`подія.`markEmailAsVerified`метод доступний за замовчуванням`App\Models\User`модель через`Illuminate\Foundation\Auth\User`базовий клас. Після підтвердження електронної адреси користувача ви можете перенаправити їх куди завгодно.

<a name="resending-the-verification-email"></a>

### Повторна відправка електронного листа для підтвердження

Іноді користувач може неправильно або випадково видалити електронну адресу для підтвердження електронної адреси. Щоб це зробити, ви можете визначити маршрут, щоб дозволити користувачеві вимагати повторної перевірки електронного листа. Потім ви можете зробити запит на цей маршрут, розмістивши просту кнопку View форми у вашому[перегляд повідомлення про перевірку](#the-email-verification-notice):

    use Illuminate\Http\Request;

    Route::post('/email/verification-notification', function (Request $request) {
        $request->user()->sendEmailVerificationNotification();

        return back()->with('status', 'verification-link-sent');
    })->middleware(['auth', 'throttle:6,1'])->name('verification.send');

<a name="protecting-routes"></a>

### Захист маршрутів

[Маршрутизуйте проміжне програмне забезпечення](/docs/{{version}}/middleware)може використовуватися, щоб дозволити лише перевіреним користувачам доступ до заданого маршруту. Laravel кораблі з`verified`проміжне програмне забезпечення, яке посилається на`Illuminate\Auth\Middleware\EnsureEmailIsVerified`клас. Оскільки це проміжне програмне забезпечення вже зареєстроване в ядрі HTTP вашої програми, все, що вам потрібно зробити, це приєднати проміжне програмне забезпечення до визначення маршруту:

    Route::get('profile', function () {
        // Only verified users may enter...
    })->middleware('verified');

Якщо неперевірений користувач намагається отримати доступ до маршруту, якому було призначено це проміжне програмне забезпечення, він буде автоматично перенаправлений на`verification.notice`[названий маршрут](/docs/{{version}}/routing#named-routes).

<a name="events"></a>

## Події

При використанні[Laravel Jetstream](https://jetstream.laravel.com), Відправлення Laravel[події](/docs/{{version}}/events)під час процесу перевірки електронної пошти. Якщо ви вручну обробляєте підтвердження електронної пошти для своєї програми, можливо, ви захочете вручну надіслати ці події після завершення перевірки. Ви можете приєднати слухачів до цих подій у своєму`EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Events\Verified' => [
            'App\Listeners\LogVerifiedUser',
        ],
    ];
