# Laravel святий

-   [Вступ](#introduction)
    -   [Як це працює](#how-it-works)
-   [Встановлення](#installation)
-   [Конфігурація](#configuration)
    -   [Заміна стандартних моделей](#overriding-default-models)
-   [Аутентифікація маркера API](#api-token-authentication)
    -   [Видача токенів API](#issuing-api-tokens)
    -   [Можливості жетонів](#token-abilities)
    -   [Захист маршрутів](#protecting-routes)
    -   [Відкликання жетонів](#revoking-tokens)
-   [Аутентифікація SPA](#spa-authentication)
    -   [Конфігурація](#spa-configuration)
    -   [Автентифікація](#spa-authenticating)
    -   [Захист маршрутів](#protecting-spa-routes)
    -   [Авторизація приватних трансляційних каналів](#authorizing-private-broadcast-channels)
-   [Аутентифікація мобільних додатків](#mobile-application-authentication)
    -   [Видача токенів API](#issuing-mobile-api-tokens)
    -   [Захист маршрутів](#protecting-mobile-api-routes)
    -   [Відкликання жетонів](#revoking-mobile-api-tokens)
-   [Тестування](#testing)

<a name="introduction"></a>

## Вступ

Laravel Sanctum пропонує легку систему автентифікації для SPA (додатків на одній сторінці), мобільних додатків та простих API на основі токенів. Sanctum дозволяє кожному користувачеві вашого додатку генерувати кілька маркерів API для свого облікового запису. Цим маркерам можуть бути надані здібності / сфери дії, які визначають, які дії маркерам дозволено виконувати.

<a name="how-it-works"></a>

### Як це працює

Laravel Sanctum існує для вирішення двох окремих проблем.

<a name="how-it-works-api-tokens"></a>

#### Маркери API

По-перше, це простий пакет для видачі маркерів API своїм користувачам без ускладнення OAuth. Ця функція натхнена GitHub "маркерами доступу". Наприклад, уявіть, що у "налаштуваннях облікового запису" вашої програми є екран, на якому користувач може генерувати маркер API для свого облікового запису. Ви можете використовувати Sanctum для створення та управління цими маркерами. Як правило, ці маркери мають дуже тривалий термін дії (роки), але користувач може в будь-який час скасувати їх вручну.

Laravel Sanctum пропонує цю функцію, зберігаючи токени API користувача в одній таблиці бази даних та аутентифікуючи вхідні запити через`Authorization`заголовок, який повинен містити дійсний маркер API.

<a name="how-it-works-spa-authentication"></a>

#### Аутентифікація SPA

По-друге, Sanctum існує, щоб запропонувати простий спосіб автентифікації односторінкових програм (SPA), які потребують зв’язку з API, що працює на основі Laravel. Ці SPA можуть існувати в тому ж сховищі, що і ваша програма Laravel, або можуть бути абсолютно окремими сховищами, такими як SPA, створений за допомогою Vue CLI.

Для цієї функції Sanctum не використовує жодних жетонів. Натомість Sanctum використовує вбудовані служби аутентифікації сеансів на основі файлів cookie Laravel. Це забезпечує переваги захисту CSRF, аутентифікації сеансу, а також захищає від витоку облікових даних автентифікації через XSS. Sanctum намагатиметься автентифікуватись за допомогою файлів cookie лише тоді, коли вхідний запит надходить із вашого власного інтерфейсу SPA.

> {tip} Цілком чудово використовувати Sanctum лише для автентифікації маркера API або лише для аутентифікації SPA. Те, що ви використовуєте Sanctum, не означає, що вам потрібно використовувати обидві функції, які він пропонує.

<a name="installation"></a>

## Встановлення

Ви можете встановити Laravel Sanctum через Composer:

    composer require laravel/sanctum

Далі слід опублікувати файли конфігурації та міграції Sanctum за допомогою`vendor:publish`artisan командування.`sanctum`файл конфігурації буде розміщений у вашому`config`каталог:

    php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"

Нарешті, слід запустити міграції бази даних. Sanctum створить одну таблицю бази даних, в якій зберігатимуться маркери API:

    php artisan migrate

Далі, якщо ви плануєте використовувати Sanctum для аутентифікації SPA, вам слід додати Middlware Sanctum до вашого`api`група проміжного програмного забезпечення у вашому`app/Http/Kernel.php`файл:

    'api' => [
        \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],

<a name="migration-customization"></a>

#### Налаштування міграції

Якщо ви не збираєтеся використовувати стандартні міграції Sanctum, вам слід зателефонувати до`Sanctum::ignoreMigrations`метод у`register`метод вашого`AppServiceProvider`. Ви можете експортувати міграції за замовчуванням за допомогою`php artisan vendor:publish --tag=sanctum-migrations`.

<a name="configuration"></a>

## Конфігурація

<a name="overriding-default-models"></a>

### Заміна стандартних моделей

Ви можете продовжити`PersonalAccessToken`модель, що використовується внутрішньо Sanctum:

    use Laravel\Sanctum\PersonalAccessToken as SanctumPersonalAccessToken;

    class PersonalAccessToken extends SanctumPersonalAccessToken
    {
        // ...
    }

Тоді ви можете доручити Sanctum використовувати вашу власну модель через`usePersonalAccessTokenModel`метод, наданий Sanctum. Як правило, ви повинні викликати цей метод у`boot`методу одного з ваших постачальників послуг:

    use App\Models\Passport\PersonalAccessToken;
    use Laravel\Sanctum\Sanctum;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Sanctum::usePersonalAccessTokenModel(PersonalAccessToken::class);
    }

<a name="api-token-authentication"></a>

## Аутентифікація маркера API

> {tip} Не слід використовувати маркери API для автентифікації власного власного SPA. Натомість використовуйте вбудований Sanctum[SPA аутентифікація](#spa-authentication).

<a name="issuing-api-tokens"></a>

### Видача токенів API

Sanctum дозволяє видавати маркери API / маркери особистого доступу, які можуть використовуватися для автентифікації запитів API. Під час надсилання запитів за допомогою маркерів API, маркер повинен бути включений у`Authorization`заголовок як a`Bearer`маркер.

Щоб розпочати видачу токенів для користувачів, ваша модель Користувач повинна використовувати`HasApiTokens`риса:

    use Laravel\Sanctum\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, HasFactory, Notifiable;
    }

Щоб видати маркер, ви можете використовувати`createToken`метод.`createToken`метод повертає a`Laravel\Sanctum\NewAccessToken`екземпляр. Маркери API хешуються за допомогою хешування SHA-256 перед збереженням у вашій базі даних, але ви можете отримати доступ до простого текстового значення маркера за`plainTextToken`власність`NewAccessToken`екземпляр. Ви повинні показати це значення користувачеві одразу після створення маркера:

    $token = $user->createToken('token-name');

    return $token->plainTextToken;

Ви можете отримати доступ до всіх токенів користувача за допомогою`tokens`Красномовні відносини, надані`HasApiTokens`риса:

    foreach ($user->tokens as $token) {
        //
    }

<a name="token-abilities"></a>

### Можливості жетонів

Sanctum дозволяє призначати маркерам "здібності", подібні до "областей" OAuth. Ви можете передати масив рядкових здібностей як другий аргумент в`createToken`метод:

    return $user->createToken('token-name', ['server:update'])->plainTextToken;

При обробці вхідного запиту, автентифікованого Sanctum, ви можете визначити, чи має маркер задану здатність, використовуючи`tokenCan`метод:

    if ($user->tokenCan('server:update')) {
        //
    }

> {tip} Для зручності,`tokenCan`метод завжди повертається`true`якщо вхідний аутентифікований запит надійшов від вашого основного SPA, і ви використовуєте вбудований Sanctum[SPA аутентифікація](#spa-authentication).

<a name="protecting-routes"></a>

### Захист маршрутів

Щоб захистити маршрути, щоб усі вхідні запити мали бути автентифікованими, слід додати файл`sanctum`захист автентифікації до ваших маршрутів API у вашому`routes/api.php`файл. Цей охоронець гарантує, що вхідні запити аутентифікуються як аутентифіковані запити з вашого SPA або містять дійсний заголовок маркера API, якщо запит надходить від третьої сторони:

    Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
        return $request->user();
    });

<a name="revoking-tokens"></a>

### Відкликання жетонів

Ви можете "відкликати" маркери, видаливши їх зі своєї бази даних за допомогою`tokens`відносини, які забезпечуються`HasApiTokens`риса:

    // Revoke all tokens...
    $user->tokens()->delete();

    // Revoke the user's current token...
    $request->user()->currentAccessToken()->delete();

    // Revoke a specific token...
    $user->tokens()->where('id', $id)->delete();

<a name="spa-authentication"></a>

## Аутентифікація SPA

Sanctum існує, щоб запропонувати простий спосіб автентифікації односторінкових програм (SPA), які потребують зв’язку з API, що працює на основі Laravel. Ці SPA можуть існувати в тому ж сховищі, що і ваша програма Laravel, або можуть бути абсолютно окремими сховищами, такими як SPA, створений за допомогою Vue CLI.

Для цієї функції Sanctum не використовує жодних жетонів. Натомість Sanctum використовує вбудовані служби аутентифікації сеансів на основі файлів cookie Laravel. Це забезпечує переваги захисту CSRF, аутентифікації сеансу, а також захищає від витоку облікових даних автентифікації через XSS. Sanctum намагатиметься автентифікуватись за допомогою файлів cookie лише тоді, коли вхідний запит надходить із вашого власного інтерфейсу SPA.

> {note} Для автентифікації ваші SPA та API мають спільний домен верхнього рівня. Однак вони можуть розміщуватися на різних субдоменах.

<a name="spa-configuration"></a>

### Конфігурація

<a name="configuring-your-first-party-domains"></a>

#### Налаштування основних доменів

По-перше, вам слід налаштувати, з яких доменів ваш SPA буде робити запити. Ви можете налаштувати ці домени за допомогою`stateful`варіант конфігурації у вашому`sanctum`файл конфігурації. Цей параметр конфігурації визначає, які домени підтримуватимуть аутентифікацію, що відповідає стану, за допомогою файлів cookie сеансу Laravel під час надсилання запитів до вашого API.

> {note} Якщо ви отримуєте доступ до своєї програми за URL-адресою, яка включає порт (`127.0.0.1:8000`), ви повинні переконатися, що ви включили номер порту до домену.

<a name="sanctum-middleware"></a>

#### Санктум Посуд

Далі слід додати Middlware Sanctum до вашого`api`група проміжного програмного забезпечення у вашому`app/Http/Kernel.php`файл. Це Middlware відповідає за те, щоб вхідні запити з вашого SPA могли автентифікуватися за допомогою сеансових файлів cookie Laravel, дозволяючи при цьому запитам від третіх сторін або мобільних додатків проводити автентифікацію за допомогою маркерів API:

    use Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful;

    'api' => [
        EnsureFrontendRequestsAreStateful::class,
        'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],

<a name="cors-and-cookies"></a>

#### CORS & Cookies

Якщо у вас виникають проблеми з автентифікацією вашої програми із SPA, яка виконується на окремому субдомені, ви, ймовірно, неправильно налаштували параметри CORS (спільне використання ресурсів) або сеансові файли cookie.

Ви повинні переконатися, що конфігурація CORS вашої програми повертає файл`Access-Control-Allow-Credentials`заголовок зі значенням`True`встановивши`supports_credentials`у вашому додатку`cors`файл конфігурації до`true`.

Крім того, слід увімкнути`withCredentials`варіант на вашому глобальному`axios`екземпляр. Як правило, це слід виконувати у вашому`resources/js/bootstrap.js`файл:

    axios.defaults.withCredentials = true;

Нарешті, слід переконатись, що конфігурація доменного файлу cookie сеансу програми підтримує будь-який субдомен вашого кореневого домену. Ви можете зробити це, додавши домену префікс ведучого`.`в межах вашого`session`файл конфігурації:

    'domain' => '.domain.com',

<a name="spa-authenticating"></a>

### Автентифікація

Для автентифікації вашого SPA, сторінка входу вашого SPA повинна спочатку зробити запит до`/sanctum/csrf-cookie`маршрут для ініціалізації захисту CSRF для програми:

    axios.get('/sanctum/csrf-cookie').then(response => {
        // Login...
    });

Під час цього запиту Laravel встановить`XSRF-TOKEN`файл cookie, що містить поточний маркер CSRF. Потім цей маркер повинен бути переданий у`X-XSRF-TOKEN`заголовок на наступні запити, які бібліотеки, такі як Axios та Angular HttpClient, автоматично роблять для вас.

Після ініціалізації захисту CSRF ви повинні зробити a`POST`запит до типового Laravel`/login`маршрут. Це`/login`маршрут може бути наданий`laravel/jetstream`[риштування для автентифікації](/docs/{{version}}/authentication#introduction)пакет.

Якщо запит на вхід буде успішним, ви пройдете автентифікацію, а наступні запити до ваших маршрутів API будуть автоматично автентифіковані за допомогою сеансового файлу cookie, який сервер Laravel видав вашому клієнту.

> {tip} Ви можете написати своє власне`/login`кінцева точка; однак вам слід переконатись, що він автентифікує користувача за допомогою стандарту,[послуги аутентифікації на основі сеансів, які надає Laravel](/docs/{{version}}/authentication#authenticating-users).

<a name="protecting-spa-routes"></a>

### Захист маршрутів

Щоб захистити маршрути, щоб усі вхідні запити мали бути автентифікованими, слід додати файл`sanctum`захист автентифікації до ваших маршрутів API у вашому`routes/api.php`файл. Цей охоронець гарантує, що вхідні запити аутентифікуються як аутентифіковані запити з вашого SPA або містять дійсний заголовок маркера API, якщо запит надходить від третьої сторони:

    Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
        return $request->user();
    });

<a name="authorizing-private-broadcast-channels"></a>

### Авторизація приватних трансляційних каналів

Якщо ваш SPA потребує автентифікації з[приватні / присутні канали мовлення](/docs/{{version}}/broadcasting#authorizing-channels), вам слід розмістити`Broadcast::routes`виклик методу у вашому`routes/api.php`файл:

    Broadcast::routes(['middleware' => ['auth:sanctum']]);

Далі, для того, щоб запити авторизації Pusher мали успіх, вам потрібно буде надати спеціальний Pusher`authorizer`при ініціалізації[Laravel Echo](/docs/{{version}}/broadcasting#installing-laravel-echo). Це дозволяє вашій програмі налаштувати Pusher на використання`axios`екземпляр, тобто[належним чином налаштовано для міждоменних запитів](#cors-and-cookies):

    window.Echo = new Echo({
        broadcaster: "pusher",
        cluster: process.env.MIX_PUSHER_APP_CLUSTER,
        encrypted: true,
        key: process.env.MIX_PUSHER_APP_KEY,
        authorizer: (channel, options) => {
            return {
                authorize: (socketId, callback) => {
                    axios.post('/api/broadcasting/auth', {
                        socket_id: socketId,
                        channel_name: channel.name
                    })
                    .then(response => {
                        callback(false, response.data);
                    })
                    .catch(error => {
                        callback(true, error);
                    });
                }
            };
        },
    })

<a name="mobile-application-authentication"></a>

## Аутентифікація мобільних додатків

Ви можете використовувати маркери Sanctum для автентифікації запитів вашого мобільного додатка до вашого API. Процес автентифікації запитів мобільних додатків подібний до автентифікації сторонніх запитів API; однак є невеликі відмінності в тому, як ви будете видавати маркери API.

<a name="issuing-mobile-api-tokens"></a>

### Видача токенів API

Для початку створіть маршрут, який приймає електронну адресу користувача / ім’я користувача, пароль та ім’я пристрою, а потім обмінює ці облікові дані на новий маркер Sanctum. Кінцева точка поверне простотекстовий маркер Sanctum, який потім може бути збережений на мобільному пристрої та використаний для додаткових запитів API:

    use App\Models\User;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use Illuminate\Validation\ValidationException;

    Route::post('/sanctum/token', function (Request $request) {
        $request->validate([
            'email' => 'required|email',
            'password' => 'required',
            'device_name' => 'required',
        ]);

        $user = User::where('email', $request->email)->first();

        if (! $user || ! Hash::check($request->password, $user->password)) {
            throw ValidationException::withMessages([
                'email' => ['The provided credentials are incorrect.'],
            ]);
        }

        return $user->createToken($request->device_name)->plainTextToken;
    });

Коли мобільний пристрій використовує маркер для надсилання запиту API до вашої програми, він повинен передати маркер у`Authorization`заголовок як a`Bearer`маркер.

> {tip} Під час випуску токенів для мобільного додатку ви також можете вказати[символічні здібності](#token-abilities)

<a name="protecting-mobile-api-routes"></a>

### Захист маршрутів

Як було задокументовано раніше, ви можете захистити маршрути, щоб усі вхідні запити повинні бути автентифіковані, додавши`sanctum`автентифікаційний охоронець маршрутів. Зазвичай ви прикріплюєте цю охорону до маршрутів, визначених у вашому`routes/api.php`файл:

    Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
        return $request->user();
    });

<a name="revoking-mobile-api-tokens"></a>

### Відкликання жетонів

Щоб дозволити користувачам відкликати маркери API, видані на мобільні пристрої, ви можете вказати їх за іменами, разом із кнопкою «Відкликати», у розділі «Налаштування облікового запису» в інтерфейсі веб-програми. Коли користувач натискає кнопку "Відкликати", ви можете видалити маркер з бази даних. Пам'ятайте, ви можете отримати доступ до маркерів API користувача через`tokens`відносини, передбачені`HasApiTokens`риса:

    // Revoke all tokens...
    $user->tokens()->delete();

    // Revoke a specific token...
    $user->tokens()->where('id', $id)->delete();

<a name="testing"></a>

## Тестування

Під час тестування`Sanctum::actingAs`метод може бути використаний для автентифікації користувача та вказівки, які можливості надаються їх токену:

    use App\Models\User;
    use Laravel\Sanctum\Sanctum;

    public function test_task_list_can_be_retrieved()
    {
        Sanctum::actingAs(
            User::factory()->create(),
            ['view-tasks']
        );

        $response = $this->get('/api/task');

        $response->assertOk();
    }

Якщо ви хочете надати маркеру всі здібності, ви повинні включити`*`у списку здібностей, наданому`actingAs`метод:

    Sanctum::actingAs(
        User::factory()->create(),
        ['*']
    );
