# Скидання паролів

-   [Вступ](#introduction)
    -   [Підготовка моделі](#model-preparation)
    -   [Підготовка бази даних](#database-preparation)
-   [Routing](#routing)
    -   [Запит на посилання для скидання пароля](#requesting-the-password-reset-link)
    -   [Скидання пароля](#resetting-the-password)
-   [Налаштування](#password-customization)

<a name="introduction"></a>

## Вступ

Більшість веб-програм надають користувачам можливість скинути свої забуті паролі. Замість того, щоб змушувати вас повторно впроваджувати це в кожній програмі, Laravel пропонує зручні методи надсилання нагадувань про паролі та скидання паролів.

> {tip} Хочете швидко розпочати? Встановити[Laravel Jetstream](https://jetstream.laravel.com)у свіжому додатку Laravel. Після міграції бази даних перейдіть до браузера`/register`або будь-яка інша URL-адреса, призначена вашій програмі. Jetstream подбає про побудову всієї вашої системи автентифікації, включаючи скидання паролів!

<a name="model-preparation"></a>

### Підготовка моделі

Перш ніж використовувати функції скидання пароля Laravel, ваш`App\Models\User`модель повинна використовувати`Illuminate\Notifications\Notifiable`риса. Як правило, ця риса автоматично включається за замовчуванням`App\Models\User`модель, що входить до комплекту Laravel.

Далі переконайтеся, що ваш`App\Models\User`модель реалізує`Illuminate\Contracts\Auth\CanResetPassword`контракт.`App\Models\User`Модель, що входить до фреймворку, вже реалізує цей інтерфейс і використовує`Illuminate\Auth\Passwords\CanResetPassword`ознака включати методи, необхідні для реалізації інтерфейсу.

<a name="database-preparation"></a>

### Підготовка бази даних

Потрібно створити таблицю, щоб зберігати маркери скидання пароля вашої програми. Міграція для цієї таблиці включена до інсталяції Laravel за замовчуванням, тому вам потрібно лише перенести вашу базу даних, щоб створити цю таблицю:

    php artisan migrate

<a name="routing"></a>

## Routing

Щоб належним чином реалізувати підтримку, яка дозволяє користувачам скидати свої паролі, нам потрібно буде визначити кілька маршрутів. По-перше, нам знадобиться пара маршрутів для обробки, що дозволяють користувачеві запитувати посилання для скидання пароля через свою електронну адресу. По-друге, нам знадобиться пара маршрутів для фактичного скидання пароля після того, як користувач відвідає посилання для скидання пароля, яке йому надіслано електронною поштою.

<a name="requesting-the-password-reset-link"></a>

### Запит на посилання для скидання пароля

<a name="the-password-reset-link-request-form"></a>

#### Форма запиту на посилання для скидання пароля

Спочатку ми визначимо маршрути, необхідні для запиту посилань на скидання пароля. Для початку ми визначимо маршрут, який повертає View з формою запиту на посилання для скидання пароля:

    Route::get('/forgot-password', function () {
        return view('auth.forgot-password');
    })->middleware(['guest'])->name('password.request');

Шаблон, який повертається цим маршрутом, повинен мати форму, що містить`email`поле, яке дозволить користувачеві запитувати посилання на скидання пароля для даної адреси електронної пошти.

<a name="password-reset-link-handling-the-form-submission"></a>

#### Обробка View форми

Далі ми визначимо, що маршрут буде обробляти запит форми з View "забутий пароль". Цей маршрут буде відповідальним за перевірку електронної адреси та надсилання запиту на скидання пароля відповідному користувачеві:

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Password;

    Route::post('/forgot-password', function (Request $request) {
        $request->validate(['email' => 'required|email']);

        $status = Password::sendResetLink(
            $request->only('email')
        );

        return $status === Password::RESET_LINK_SENT
                    ? back()->with(['status' => __($status)])
                    : back()->withErrors(['email' => __($status)]);
    })->middleware(['guest'])->name('password.email');

Перш ніж рухатися далі, давайте детальніше розглянемо цей маршрут. По-перше, запит`email`атрибут перевірено. Далі ми використовуватимемо вбудований "посередник паролів" Laravel (через`Password`фасад), щоб надіслати користувачеві посилання на скидання пароля. Брокер паролів подбає про те, щоб отримати користувача за даним полем (в даному випадку електронною адресою) і надіслати користувачеві посилання для скидання пароля через вбудований Laravel[система Notification](/docs/{{version}}/notifications).

`sendResetLink`метод повертає "статус" кулі. Цей статус може бути перекладений за допомогою Laravel[локалізація](/docs/{{version}}/localization)помічники для того, щоб показати зручне повідомлення користувачеві про стан його запиту. Переклад статусу скидання пароля визначається вашим додатком`resources/lang/{lang}/passwords.php`мовний файл. Запис для кожного можливого значення зливу стану знаходиться в межах`passwords`мовний файл.

> {tip} При ручному виконанні скидання пароля потрібно визначити вміст Views та маршрутів самостійно. Якщо ви хочете риштування, яке включає всю необхідну логіку автентифікації та перевірки, перевірте[Laravel Jetstream](https://jetstream.laravel.com).

<a name="resetting-the-password"></a>

### Скидання пароля

<a name="the-password-reset-form"></a>

#### Форма скидання пароля

Далі ми визначимо маршрути, необхідні для фактичного скидання пароля після того, як користувач натисне посилання для скидання пароля, яке було надіслане йому електронною поштою та надає новий пароль. Спочатку визначимо маршрут, який буде відображати форму скидання пароля, яка відображається, коли користувач натискає посилання скидання пароля. Цей маршрут отримає`token`параметр, який ми будемо використовувати пізніше для перевірки запиту на скидання пароля:

    Route::get('/reset-password/{token}', function ($token) {
        return view('auth.reset-password', ['token' => $token]);
    })->middleware(['guest'])->name('password.reset');

Вигляд, який повертається цим маршрутом, повинен мати форму, що містить`email`поле, a`password`поле, a`password_confirmation`поле, і приховане`token`поле, яке повинно містити значення секретного маркера, отриманого нашим маршрутом.

<a name="password-reset-handling-the-form-submission"></a>

#### Обробка View форми

Звичайно, нам потрібно визначити маршрут, який фактично обробляє View форми для скидання пароля. Цей маршрут буде відповідальним за перевірку вхідного запиту та оновлення пароля користувача в базі даних:

    use Illuminate\Auth\Events\PasswordReset;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use Illuminate\Support\Facades\Password;
    use Illuminate\Support\Str;

    Route::post('/reset-password', function (Request $request) {
        $request->validate([
            'token' => 'required',
            'email' => 'required|email',
            'password' => 'required|min:8|confirmed',
        ]);

        $status = Password::reset(
            $request->only('email', 'password', 'password_confirmation', 'token'),
            function ($user, $password) use ($request) {
                $user->forceFill([
                    'password' => Hash::make($password)
                ])->save();

                $user->setRememberToken(Str::random(60));

                event(new PasswordReset($user));
            }
        );

        return $status == Password::PASSWORD_RESET
                    ? redirect()->route('login')->with('status', __($status))
                    : back()->withErrors(['email' => __($status)]);
    })->middleware(['guest'])->name('password.update');

Перш ніж рухатися далі, давайте детальніше розглянемо цей маршрут. По-перше, запит`token`,`email`, і`password`атрибути перевірені. Далі ми використовуватимемо вбудований "посередник паролів" Laravel (через`Password`фасад) для перевірки облікових даних запиту на скидання пароля.

Якщо маркер, адреса електронної пошти та пароль, надані посереднику паролів, є дійсними, Закриття передається до`reset`буде викликаний метод. В рамках цього Закриття, яке отримує екземпляр користувача та пароль простого тексту, ми можемо оновити пароль користувача в базі даних.

`reset`метод повертає "статус" кулі. Цей статус може бути перекладений за допомогою Laravel[локалізація](/docs/{{version}}/localization)помічники для того, щоб показати зручне повідомлення користувачеві про стан його запиту. Переклад статусу скидання пароля визначається вашим додатком`resources/lang/{lang}/passwords.php`мовний файл. Запис для кожного можливого значення зливу стану знаходиться в межах`passwords`мовний файл.

<a name="password-customization"></a>

## Налаштування

<a name="reset-link-customization"></a>

#### Скинути налаштування посилання

Ви можете налаштувати URL-адресу посилання для скидання пароля за допомогою`createUrlUsing`метод, передбачений`ResetPassword`клас повідомлення. Цей метод приймає Закриття, яке отримує екземпляр користувача, який отримує повідомлення, а також маркер посилання на скидання пароля. Як правило, вам слід викликати цей метод у постачальника послуг`boot`метод:

    use Illuminate\Auth\Notifications\ResetPassword;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        ResetPassword::createUrlUsing(function ($notifiable, string $token) {
            return 'https://example.com/auth/reset-password?token='.$token;
        });
    }

<a name="reset-email-customization"></a>

#### Скинути налаштування електронної пошти

Ви можете легко змінити клас Notification, який використовується для надсилання користувачеві посилання на скидання пароля. Для початку перевизначте`sendPasswordResetNotification`метод на вашому`User`модель. За допомогою цього методу ви можете надсилати Notification, використовуючи будь-який вибраний вами клас сповіщень. Скидання пароля`$token`є першим аргументом, отриманим методом:

    /**
     * Send the password reset notification.
     *
     * @param  string  $token
     * @return void
     */
    public function sendPasswordResetNotification($token)
    {
        $this->notify(new ResetPasswordNotification($token));
    }
