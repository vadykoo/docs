# Паспорт Laravel

-   [Вступ](#introduction)
-   [Оновлення паспорта](#upgrading)
-   [Встановлення](#installation)
    -   [Розгортання паспорта](#deploying-passport)
    -   [Налаштування міграції](#migration-customization)
-   [Конфігурація](#configuration)
    -   [Клієнтське таємне хешування](#client-secret-hashing)
    -   [Життєві терміни](#token-lifetimes)
    -   [Заміна стандартних моделей](#overriding-default-models)
-   [Видача токенів доступу](#issuing-access-tokens)
    -   [Управління клієнтами](#managing-clients)
    -   [Запит жетонів](#requesting-tokens)
    -   [Освіжаючі жетони](#refreshing-tokens)
    -   [Відкликання жетонів](#revoking-tokens)
    -   [Очищення токенів](#purging-tokens)
-   [Код дозволу Надання з PKCE](#code-grant-pkce)
    -   [Створення клієнта](#creating-a-auth-pkce-grant-client)
    -   [Запит жетонів](#requesting-auth-pkce-grant-tokens)
-   [Токени надання пароля](#password-grant-tokens)
    -   [Створення клієнта надання пароля](#creating-a-password-grant-client)
    -   [Запит жетонів](#requesting-password-grant-tokens)
    -   [Запит на всі сфери](#requesting-all-scopes)
    -   [Налаштування постачальника користувачів](#customizing-the-user-provider)
    -   [Налаштування поля імені користувача](#customizing-the-username-field)
    -   [Налаштування перевірки пароля](#customizing-the-password-validation)
-   [Неявні жетони надання](#implicit-grant-tokens)
-   [Повноваження клієнта надають токени](#client-credentials-grant-tokens)
-   [Токени особистого доступу](#personal-access-tokens)
    -   [Створення клієнта персонального доступу](#creating-a-personal-access-client)
    -   [Керування токенами особистого доступу](#managing-personal-access-tokens)
-   [Захист маршрутів](#protecting-routes)
    -   [Через Middleware](#via-middleware)
    -   [Передача маркера доступу](#passing-the-access-token)
-   [Токени](#token-scopes)
    -   [Визначення обсягу](#defining-scopes)
    -   [Область за замовчуванням](#default-scope)
    -   [Присвоєння сфери дії маркерам](#assigning-scopes-to-tokens)
    -   [Перевірка обсягу](#checking-scopes)
-   [Споживання вашого API за допомогою JavaScript](#consuming-your-api-with-javascript)
-   [Події](#events)
-   [Тестування](#testing)

<a name="introduction"></a>

## Вступ

Laravel вже дозволяє легко виконувати автентифікацію за допомогою традиційних форм входу, але як щодо API? API зазвичай використовують маркери для автентифікації користувачів і не підтримують стан сеансу між запитами. Laravel робить автентифікацію API легкою, використовуючи Laravel Passport, який забезпечує повну реалізацію сервера OAuth2 для вашого додатку Laravel за лічені хвилини. Паспорт будується поверх[Сервер OAuth2 ліги](https://github.com/thephpleague/oauth2-server)що підтримують Енді Міллінгтон і Саймон Хемп.

> {note} Ця документація передбачає, що ви вже знайомі з OAuth2. Якщо ви нічого не знаєте про OAuth2, подумайте про ознайомлення із загальним[термінологія](https://oauth2.thephpleague.com/terminology/)та особливості OAuth2 перед продовженням.

<a name="upgrading"></a>

## Оновлення паспорта

Під час оновлення до нової основної версії Passport важливо уважно переглянути[посібник з оновлення](https://github.com/laravel/passport/blob/master/UPGRADE.md).

<a name="installation"></a>

## Встановлення

Для початку встановіть Passport через менеджер пакетів Composer:

    composer require laravel/passport

Постачальник послуг Passport реєструє власний каталог міграції баз даних із фреймворком, тому вам слід перенести базу даних після встановлення пакету. Міграція Passport створить таблиці, необхідні вашій програмі для зберігання клієнтів та доступу до маркерів:

    php artisan migrate

Далі вам слід запустити`passport:install`команди. Ця команда створить ключі шифрування, необхідні для створення маркерів безпечного доступу. Крім того, команда створить клієнтів "особистий доступ" та "надання пароля", які будуть використовуватися для генерації маркерів доступу:

    php artisan passport:install

> {tip} Якщо ви хочете використовувати UUID як значення первинного ключа паспорта`Client`модель замість автоматичного збільшення цілих чисел, встановіть Passport за допомогою[`uuids`варіант](#client-uuids).

Після запуску`passport:install`додайте команду`Laravel\Passport\HasApiTokens`риса до вашого`App\Models\User`модель. Ця ознака надасть кілька допоміжних методів для вашої моделі, які дозволять вам перевірити маркер і сфери дії аутентифікованого користувача:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Factories\HasFactory
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Laravel\Passport\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, HasFactory, Notifiable;
    }

Далі вам слід зателефонувати до`Passport::routes`метод у межах`boot`метод вашого`AuthServiceProvider`. Цей метод реєструє маршрути, необхідні для видачі маркерів доступу та анулювання маркерів доступу, клієнтів та персональних маркерів доступу:

    <?php

    namespace App\Providers;

    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    use Illuminate\Support\Facades\Gate;
    use Laravel\Passport\Passport;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * The policy mappings for the application.
         *
         * @var array
         */
        protected $policies = [
            'App\Models\Model' => 'App\Policies\ModelPolicy',
        ];

        /**
         * Register any authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Passport::routes();
        }
    }

Нарешті, у вашому`config/auth.php`конфігураційний файл, вам слід встановити файл`driver`варіант`api`перевірка автентичності до`passport`. Це дозволить вашій заявці використовувати паспорт`TokenGuard`при автентифікації вхідних запитів API:

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'passport',
            'provider' => 'users',
        ],
    ],

<a name="client-uuids"></a>

#### Клієнтські UUID

Ви можете запустити`passport:install`команда за допомогою`--uuids`варіант присутній. Цей прапор повідомляє Passport, що ви хочете використовувати UUID замість автоматичного збільшення цілих чисел як Passport`Client`значення первинного ключа моделі. Після запуску`passport:install`команда за допомогою`--uuids`Вам буде надано додаткові вказівки щодо вимкнення міграції Passport за замовчуванням:

    php artisan passport:install --uuids

<a name="deploying-passport"></a>

### Розгортання паспорта

Розгортаючи Passport на своїх виробничих серверах вперше, вам, ймовірно, доведеться запустити`passport:keys`команди. Ця команда генерує ключі шифрування, необхідні паспорту для генерації маркера доступу. Створені ключі, як правило, не зберігаються в контролі джерела:

    php artisan passport:keys

При необхідності ви можете визначити шлях, з якого слід завантажувати ключі Passport. Ви можете використовувати`Passport::loadKeysFrom`метод для досягнення цього:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::loadKeysFrom('/secret-keys/oauth');
    }

Крім того, ви можете опублікувати файл конфігурації Passport за допомогою`php artisan vendor:publish --tag=passport-config`, який надасть можливість завантажити ключі шифрування зі змінних середовища:

    PASSPORT_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
    <private key here>
    -----END RSA PRIVATE KEY-----"

    PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----
    <public key here>
    -----END PUBLIC KEY-----"

<a name="migration-customization"></a>

### Налаштування міграції

Якщо ви не збираєтеся використовувати міграції Passport за замовчуванням, вам слід зателефонувати до`Passport::ignoreMigrations`метод у`register`метод вашого`AppServiceProvider`. Ви можете експортувати міграції за замовчуванням за допомогою`php artisan vendor:publish --tag=passport-migrations`.

<a name="configuration"></a>

## Конфігурація

<a name="client-secret-hashing"></a>

### Клієнтське таємне хешування

Якщо ви хочете, щоб секрети вашого клієнта були хешовані при зберіганні у вашій базі даних, вам слід зателефонувати до`Passport::hashClientSecrets`метод у`boot`метод вашого`AppServiceProvider`:

    Passport::hashClientSecrets();

Після ввімкнення всі ваші секрети клієнта відображатимуться лише один раз, коли ваш клієнт буде створений. Оскільки секретне значення клієнтського текстового тексту ніколи не зберігається в базі даних, відновити його у разі втрати неможливо.

<a name="token-lifetimes"></a>

### Життєві терміни

За замовчуванням Passport видає довговічні маркери доступу, термін дії яких закінчується через рік. Якщо ви хочете налаштувати довший / коротший термін служби маркера, ви можете використовувати`tokensExpireIn`,`refreshTokensExpireIn`, і`personalAccessTokensExpireIn`методи. Ці методи слід викликати з`boot`метод вашого`AuthServiceProvider`:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::tokensExpireIn(now()->addDays(15));

        Passport::refreshTokensExpireIn(now()->addDays(30));

        Passport::personalAccessTokensExpireIn(now()->addMonths(6));
    }

> {примітка}`expires_at`стовпці таблиць бази даних Passport доступні лише для читання та лише для відображення. Під час випуску токенів Passport зберігає інформацію про термін дії у підписаних та зашифрованих токенах. Якщо вам потрібно визнати маркер недійсним, його слід відкликати.

<a name="overriding-default-models"></a>

### Заміна стандартних моделей

Ви можете розширити моделі, які використовуються внутрішньо Passport:

    use Laravel\Passport\Client as PassportClient;

    class Client extends PassportClient
    {
        // ...
    }

Потім ви можете доручити Passport використовувати власні моделі через`Passport`клас:

    use App\Models\Passport\AuthCode;
    use App\Models\Passport\Client;
    use App\Models\Passport\PersonalAccessClient;
    use App\Models\Passport\Token;

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::useTokenModel(Token::class);
        Passport::useClientModel(Client::class);
        Passport::useAuthCodeModel(AuthCode::class);
        Passport::usePersonalAccessClientModel(PersonalAccessClient::class);
    }

<a name="issuing-access-tokens"></a>

## Видача токенів доступу

Використання OAuth2 з кодами авторизації - це те, як більшість розробників знайомі з OAuth2. При використанні кодів авторизації клієнтська програма перенаправляє користувача на ваш сервер, де він або схвалює, або відхиляє запит на видачу клієнту маркера доступу.

<a name="managing-clients"></a>

### Управління клієнтами

По-перше, розробники, що створюють додатки, які повинні взаємодіяти з API вашого додатка, повинні зареєструвати свою програму у вашій, створивши "клієнт". Як правило, це полягає у наданні назви їхньої програми та URL-адреси, на яку ваша програма може переспрямовувати після того, як користувачі затвердять їхній запит на авторизацію.

<a name="the-passportclient-command"></a>

#### `passport:client`Команда

Найпростіший спосіб створити клієнта - використання`passport:client`Artisan командування. Ця команда може бути використана для створення власних клієнтів для тестування функціональності OAuth2. Коли ви запускаєте`client`команда, Passport запропонує вам отримати додаткову інформацію про вашого клієнта та надасть ідентифікатор клієнта та секрет:

    php artisan passport:client

**Переспрямування URL-адрес**

Якщо ви хочете дозволити для свого клієнта декілька URL-адрес для переадресації, ви можете вказати їх за допомогою списку, розділеного комами, коли запит на URL-адресу`passport:client`команда:

    http://example.com/callback,http://examplefoo.com/callback

> {note} Будь-яка URL-адреса, яка містить коми, повинна бути закодована.

<a name="clients-json-api"></a>

#### API JSON

Оскільки ваші користувачі не зможуть використовувати`client`команда Passport надає JSON API, який ви можете використовувати для створення клієнтів. Це економить вам проблеми з необхідністю вручну кодувати контролери для створення, оновлення та видалення клієнтів.

Однак вам потрібно буде з'єднати JSON API Passport зі своїм власним інтерфейсом, щоб забезпечити інформаційну панель для ваших користувачів для управління своїми клієнтами. Нижче ми розглянемо всі кінцеві точки API для управління клієнтами. Для зручності ми будемо використовувати[Аксіос](https://github.com/axios/axios)для демонстрації надсилання HTTP-запитів до кінцевих точок.

API JSON охороняється`web`і`auth`Middlware; тому його можна викликати лише з вашої власної програми. Його не можна викликати із зовнішнього джерела.

<a name="get-oauthclients"></a>

#### `GET /oauth/clients`

Цей маршрут повертає всіх клієнтів для автентифікованого користувача. Це в першу чергу корисно для переліку всіх клієнтів користувача, щоб вони могли їх редагувати або видаляти:

    axios.get('/oauth/clients')
        .then(response => {
            console.log(response.data);
        });

<a name="post-oauthclients"></a>

#### `POST /oauth/clients`

Цей маршрут використовується для створення нових клієнтів. Для цього потрібні дві частини даних: дані клієнта`name`і a`redirect`URL.`redirect`URL-адреса - це те, куди користувач буде перенаправлений після схвалення чи відмови у запиті на авторизацію.

Коли клієнт створюється, йому видається ідентифікатор клієнта та секрет клієнта. Ці значення будуть використовуватися при запиті маркерів доступу з вашої програми. Маршрут створення клієнта поверне новий екземпляр клієнта:

    const data = {
        name: 'Client Name',
        redirect: 'http://example.com/callback'
    };

    axios.post('/oauth/clients', data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // List errors on response...
        });

<a name="put-oauthclientsclient-id"></a>

#### `PUT /oauth/clients/{client-id}`

Цей маршрут використовується для оновлення клієнтів. Для цього потрібні дві частини даних: дані клієнта`name`і a`redirect`URL.`redirect`URL-адреса - це те, куди користувач буде перенаправлений після схвалення чи відмови у запиті на авторизацію. Маршрут поверне оновлений екземпляр клієнта:

    const data = {
        name: 'New Client Name',
        redirect: 'http://example.com/callback'
    };

    axios.put('/oauth/clients/' + clientId, data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // List errors on response...
        });

<a name="delete-oauthclientsclient-id"></a>

#### `DELETE /oauth/clients/{client-id}`

Цей маршрут використовується для видалення клієнтів:

    axios.delete('/oauth/clients/' + clientId)
        .then(response => {
            //
        });

<a name="requesting-tokens"></a>

### Запит жетонів

<a name="requesting-tokens-redirecting-for-authorization"></a>

#### Перенаправлення на авторизацію

Після створення клієнта розробники можуть використовувати його ідентифікатор та секрет для запиту коду авторизації та маркера доступу з вашої програми. По-перше, програма-споживач повинна зробити запит на перенаправлення до програми`/oauth/authorize`маршрут так:

    Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => '',
            'state' => $state,
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

> {tip} Пам'ятайте,`/oauth/authorize`маршрут вже визначений`Passport::routes`метод. Вам не потрібно вручну визначати цей маршрут.

<a name="approving-the-request"></a>

#### Затвердження запиту

При отриманні запитів на авторизацію Passport автоматично відображає користувачеві шаблон, що дозволяє йому схвалити або відхилити запит на авторизацію. Якщо вони затвердять запит, вони будуть перенаправлені назад до`redirect_uri`що було вказано додатком-споживачем.`redirect_uri`повинен відповідати`redirect`URL-адреса, вказана під час створення клієнта.

Якщо ви хочете налаштувати екран затвердження авторизації, ви можете опублікувати подання Passport за допомогою`vendor:publish`Artisan командування. Опубліковані погляди будуть розміщені в`resources/views/vendor/passport`:

    php artisan vendor:publish --tag=passport-views

Іноді вам може знадобитися пропустити запит авторизації, наприклад, під час авторизації власного клієнта. Ви можете досягти цього до[розширення`Client`модель](#overriding-default-models)та визначення а`skipsAuthorization`метод. Якщо`skipsAuthorization`повертається`true`клієнт буде схвалений, а користувач буде перенаправлений назад до`redirect_uri`негайно:

    <?php

    namespace App\Models\Passport;

    use Laravel\Passport\Client as BaseClient;

    class Client extends BaseClient
    {
        /**
         * Determine if the client should skip the authorization prompt.
         *
         * @return bool
         */
        public function skipsAuthorization()
        {
            return $this->firstParty();
        }
    }

<a name="requesting-tokens-converting-authorization-codes-to-access-tokens"></a>

#### Перетворення кодів авторизації для доступу до маркерів

Якщо користувач схвалить запит на авторизацію, він буде перенаправлений назад до програми-споживача. Спочатку споживач повинен перевірити`state`параметра проти значення, яке зберігалося до перенаправлення. Якщо параметр стану відповідає споживачеві, слід видати a`POST`запит до вашої програми на запит маркера доступу. Запит повинен містити код авторизації, виданий вашим додатком, коли користувач схвалив запит авторизації. У цьому прикладі ми будемо використовувати бібліотеку Guzzle HTTP для створення`POST`запит:

    Route::get('/callback', function (Request $request) {
        $state = $request->session()->pull('state');

        throw_unless(
            strlen($state) > 0 && $state === $request->state,
            InvalidArgumentException::class
        );

        $http = new GuzzleHttp\Client;

        $response = $http->post('http://your-app.com/oauth/token', [
            'form_params' => [
                'grant_type' => 'authorization_code',
                'client_id' => 'client-id',
                'client_secret' => 'client-secret',
                'redirect_uri' => 'http://example.com/callback',
                'code' => $request->code,
            ],
        ]);

        return json_decode((string) $response->getBody(), true);
    });

Це`/oauth/token`route поверне відповідь JSON, що містить`access_token`,`refresh_token`, і`expires_in`атрибути.`expires_in`атрибут містить кількість секунд до закінчення терміну дії маркера доступу.

> {tip} Подобається`/oauth/authorize`маршрут,`/oauth/token`маршрут для вас визначений`Passport::routes`метод. Не потрібно вручну визначати цей маршрут. За замовчуванням цей маршрут регулюється за допомогою налаштувань`ThrottleRequests`Middlware.

<a name="tokens-json-api"></a>

#### API JSON

Паспорт також включає API JSON для управління авторизованими маркерами доступу. Ви можете поєднати це зі своїм власним інтерфейсом, щоб запропонувати своїм користувачам інформаційну панель для управління маркерами доступу. Для зручності ми будемо використовувати[Аксіос](https://github.com/mzabriskie/axios)для демонстрації надсилання HTTP-запитів до кінцевих точок. API JSON охороняється`web`і`auth`Middlware; тому його можна викликати лише з вашої власної програми.

<a name="get-oauthtokens"></a>

#### `GET /oauth/tokens`

Цей маршрут повертає всі авторизовані маркери доступу, які створив аутентифікований користувач. Це в першу чергу корисно для переліку всіх токенів користувача, щоб вони могли їх відкликати:

    axios.get('/oauth/tokens')
        .then(response => {
            console.log(response.data);
        });

<a name="delete-oauthtokenstoken-id"></a>

#### `DELETE /oauth/tokens/{token-id}`

Цей маршрут може бути використаний для скасування авторизованих маркерів доступу та пов’язаних з ними маркерів оновлення:

    axios.delete('/oauth/tokens/' + tokenId);

<a name="refreshing-tokens"></a>

### Освіжаючі жетони

Якщо ваша програма видає короткочасні маркери доступу, користувачам потрібно буде оновити свої маркери доступу за допомогою маркера оновлення, який був наданий їм під час видачі маркера доступу. У цьому прикладі ми будемо використовувати бібліотеку Guzzle HTTP для оновлення маркера:

    $http = new GuzzleHttp\Client;

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'refresh_token',
            'refresh_token' => 'the-refresh-token',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'scope' => '',
        ],
    ]);

    return json_decode((string) $response->getBody(), true);

Це`/oauth/token`route поверне відповідь JSON, що містить`access_token`,`refresh_token`, і`expires_in`атрибути.`expires_in`атрибут містить кількість секунд до закінчення терміну дії маркера доступу.

<a name="revoking-tokens"></a>

### Відкликання жетонів

Ви можете відкликати маркер, використовуючи`revokeAccessToken`метод на`TokenRepository`. Ви можете відкликати маркери оновлення маркера за допомогою`revokeRefreshTokensByAccessTokenId`метод на`RefreshTokenRepository`:

    $tokenRepository = app('Laravel\Passport\TokenRepository');
    $refreshTokenRepository = app('Laravel\Passport\RefreshTokenRepository');

    // Revoke an access token...
    $tokenRepository->revokeAccessToken($tokenId);

    // Revoke all of the token's refresh tokens...
    $refreshTokenRepository->revokeRefreshTokensByAccessTokenId($tokenId);

<a name="purging-tokens"></a>

### Очищення токенів

Коли маркери скасовано або закінчився термін їх дії, можливо, ви захочете видалити їх із бази даних. Паспорт поставляється з командою, яка може зробити це за вас:

    # Purge revoked and expired tokens and auth codes...
    php artisan passport:purge

    # Only purge revoked tokens and auth codes...
    php artisan passport:purge --revoked

    # Only purge expired tokens and auth codes...
    php artisan passport:purge --expired

Ви також можете налаштувати a[запланована робота](/docs/{{version}}/scheduling)у вашій консолі`Kernel`клас для автоматичного обрізання ваших токенів за розкладом:

    /**
     * Define the application's command schedule.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->command('passport:purge')->hourly();
    }

<a name="code-grant-pkce"></a>

## Код дозволу Надання з PKCE

Надання коду авторизації з «Ключем доказу для обміну кодами» (PKCE) - це надійний спосіб автентифікації односторінкових програм або власних програм для доступу до вашого API. Цей грант слід використовувати, коли ви не можете гарантувати, що таємниця клієнта буде зберігатися конфіденційно або для того, щоб зменшити загрозу перехоплення зловмисником коду авторизації. Поєднання "перевірявача коду" та "виклику коду" замінює клієнтський секрет при обміні кодом авторизації на маркер доступу.

<a name="creating-a-auth-pkce-grant-client"></a>

### Створення клієнта

Перш ніж ваша програма зможе видавати маркери через надання дозволу коду авторизації за допомогою PKCE, вам потрібно буде створити клієнт із підтримкою PKCE. Ви можете зробити це за допомогою`passport:client`команда за допомогою`--public`варіант:

    php artisan passport:client --public

<a name="requesting-auth-pkce-grant-tokens"></a>

### Запит жетонів

<a name="code-verifier-code-challenge"></a>

#### Верифікатор коду та виклик коду

Оскільки цей дозвіл на авторизацію не надає таємницю клієнта, розробникам потрібно буде згенерувати комбінацію перевірителя коду та виклику коду, щоб запросити маркер.

Код, що перевіряє, повинен бути випадковим рядком із 43 до 128 символів, що містить літери, цифри та`"-"`,`"."`,`"_"`,`"~"`, як визначено в[Специфікація RFC 7636](https://tools.ietf.org/html/rfc7636).

Викликом коду має бути кодований рядок Base64 із символами, захищеними URL-адресами та іменами файлів. Відставання`'='`символи слід видаляти, а розривів рядків, пробілів та інших додаткових символів не повинно бути.

    $encoded = base64_encode(hash('sha256', $code_verifier, true));

    $codeChallenge = strtr(rtrim($encoded, '='), '+/', '-_');

<a name="code-grant-pkce-redirecting-for-authorization"></a>

#### Перенаправлення на авторизацію

Після створення клієнта ви можете використовувати ідентифікатор клієнта та згенерований код перевірки та виклик коду для запиту коду авторизації та маркера доступу з вашої програми. По-перше, програма-споживач повинна зробити запит на перенаправлення до програми`/oauth/authorize`маршрут:

    Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $request->session()->put('code_verifier', $code_verifier = Str::random(128));

        $codeChallenge = strtr(rtrim(
            base64_encode(hash('sha256', $code_verifier, true))
        , '='), '+/', '-_');

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => '',
            'state' => $state,
            'code_challenge' => $codeChallenge,
            'code_challenge_method' => 'S256',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

<a name="code-grant-pkce-converting-authorization-codes-to-access-tokens"></a>

#### Перетворення кодів авторизації для доступу до маркерів

Якщо користувач схвалить запит на авторизацію, він буде перенаправлений назад до програми-споживача. Споживач повинен перевірити`state`параметра проти значення, яке зберігалося до перенаправлення, як у стандартному наданні коду авторизації.

Якщо параметр стану відповідає, споживач повинен видати a`POST`запит до вашої програми на запит маркера доступу. Запит повинен містити код авторизації, виданий вашим додатком, коли користувач схвалив запит авторизації, разом із спочатку сформованим засобом перевірки коду:

    Route::get('/callback', function (Request $request) {
        $state = $request->session()->pull('state');

        $codeVerifier = $request->session()->pull('code_verifier');

        throw_unless(
            strlen($state) > 0 && $state === $request->state,
            InvalidArgumentException::class
        );

        $response = (new GuzzleHttp\Client)->post('http://your-app.com/oauth/token', [
            'form_params' => [
                'grant_type' => 'authorization_code',
                'client_id' => 'client-id',
                'redirect_uri' => 'http://example.com/callback',
                'code_verifier' => $codeVerifier,
                'code' => $request->code,
            ],
        ]);

        return json_decode((string) $response->getBody(), true);
    });

<a name="password-grant-tokens"></a>

## Токени надання пароля

Надання пароля OAuth2 дозволяє вашим стороннім клієнтам, таким як мобільний додаток, отримати маркер доступу, використовуючи адресу електронної пошти / ім’я користувача та пароль. Це дозволяє безпечно видавати маркери доступу своїм стороннім клієнтам, не вимагаючи, щоб користувачі проходили весь потік перенаправлення коду авторизації OAuth2.

<a name="creating-a-password-grant-client"></a>

### Створення клієнта надання пароля

Перш ніж ваша програма зможе видавати маркери за допомогою надання пароля, вам потрібно буде створити клієнт надання пароля. Ви можете зробити це за допомогою`passport:client`команда за допомогою`--password`варіант. Якщо ви вже запустили`passport:install`команда, вам не потрібно запускати цю команду:

    php artisan passport:client --password

<a name="requesting-password-grant-tokens"></a>

### Запит жетонів

Після створення клієнта надання пароля ви можете подати запит на маркер доступу, видавши файл`POST`запит до`/oauth/token`маршрут із адресою електронної пошти та паролем користувача. Пам’ятайте, цей маршрут вже зареєстровано`Passport::routes`методу, тому немає необхідності визначати його вручну. Якщо запит буде успішним, ви отримаєте`access_token`і`refresh_token`у відповіді JSON із сервера:

    $http = new GuzzleHttp\Client;

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'password',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'username' => 'taylor@laravel.com',
            'password' => 'my-password',
            'scope' => '',
        ],
    ]);

    return json_decode((string) $response->getBody(), true);

> {tip} Пам’ятайте, маркери доступу за замовчуванням довговічні. Тим не менш, ти вільний[налаштуйте максимальний термін служби маркера доступу](#configuration)при необхідності.

<a name="requesting-all-scopes"></a>

### Запит на всі сфери

Використовуючи надання пароля або надання клієнтських ідентифікаційних даних, ви можете авторизувати маркер для всіх областей, що підтримуються вашою програмою. Ви можете зробити це, запитувавши файл`*`сфера застосування. Якщо ви запитуєте`*`сфера застосування`can`метод на екземплярі маркера завжди повертається`true`. Цей обсяг може бути призначений лише маркеру, який видається за допомогою`password`або`client_credentials`грант:

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'password',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'username' => 'taylor@laravel.com',
            'password' => 'my-password',
            'scope' => '*',
        ],
    ]);

<a name="customizing-the-user-provider"></a>

### Налаштування постачальника користувачів

Якщо у вашій програмі використовується більше одного[аутентифікація постачальника користувачів](/docs/{{version}}/authentication#introduction), Ви можете вказати, якого постачальника користувачів використовує клієнт, що надає пароль, вказавши a`--provider`опція при створенні клієнта через`artisan passport:client --password`команди. Вказане ім'я постачальника має відповідати дійсному постачальнику, визначеному у вашому`config/auth.php`файл конфігурації. Тоді можна[захистити свій маршрут за допомогою проміжного програмного забезпечення](#via-middleware)забезпечити авторизацію лише користувачів із зазначеного постачальника послуг охорони.

<a name="customizing-the-username-field"></a>

### Налаштування поля імені користувача

Під час автентифікації за допомогою надання пароля Passport буде використовувати`email`атрибут вашої моделі як "ім'я користувача". Однак ви можете налаштувати цю поведінку, визначивши a`findForPassport`метод на вашій моделі:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Laravel\Passport\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;

        /**
         * Find the user instance for the given username.
         *
         * @param  string  $username
         * @return \App\Models\User
         */
        public function findForPassport($username)
        {
            return $this->where('username', $username)->first();
        }
    }

<a name="customizing-the-password-validation"></a>

### Налаштування перевірки пароля

Під час автентифікації за допомогою надання пароля Passport буде використовувати`password`атрибут вашої моделі для перевірки даного пароля. Якщо у вашій моделі немає`password`атрибут або ви хочете налаштувати логіку перевірки пароля, ви можете визначити a`validateForPassportPasswordGrant`метод на вашій моделі:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Support\Facades\Hash;
    use Laravel\Passport\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;

        /**
         * Validate the password of the user for the Passport password grant.
         *
         * @param  string  $password
         * @return bool
         */
        public function validateForPassportPasswordGrant($password)
        {
            return Hash::check($password, $this->password);
        }
    }

<a name="implicit-grant-tokens"></a>

## Неявні жетони надання

Явне надання подібне до дозволу коду авторизації; однак маркер повертається клієнту без обміну кодом авторизації. Цей грант найчастіше використовується для JavaScript або мобільних додатків, де облікові дані клієнта неможливо надійно зберегти. Щоб увімкнути надання, зателефонуйте на`enableImplicitGrant`метод у вашому`AuthServiceProvider`:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::enableImplicitGrant();
    }

Після ввімкнення гранту розробники можуть використовувати свій ідентифікатор клієнта для запиту маркера доступу з вашої програми. Програма-споживач повинна зробити запит на перенаправлення до вашої програми`/oauth/authorize`маршрут так:

    Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'token',
            'scope' => '',
            'state' => $state,
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

> {tip} Пам'ятайте,`/oauth/authorize`маршрут вже визначений`Passport::routes`метод. Вам не потрібно вручну визначати цей маршрут.

<a name="client-credentials-grant-tokens"></a>

## Повноваження клієнта надають токени

Надання облікових даних клієнта підходить для автентифікації машини на машину. Наприклад, ви можете використовувати цей грант у запланованому завданні, яке виконує завдання технічного обслуговування через API.

Перш ніж ваша програма зможе видавати токени за допомогою надання клієнтських облікових даних, вам потрібно буде створити клієнтський сертифікат для надання клієнта. Ви можете зробити це за допомогою`--client`варіант`passport:client`команда:

    php artisan passport:client --client

Далі, щоб використовувати цей тип гранту, вам потрібно додати`CheckClientCredentials`Middlware для`$routeMiddleware`власність вашого`app/Http/Kernel.php`файл:

    use Laravel\Passport\Http\Middleware\CheckClientCredentials;

    protected $routeMiddleware = [
        'client' => CheckClientCredentials::class,
    ];

Потім приєднайте Middlware до маршруту:

    Route::get('/orders', function (Request $request) {
        ...
    })->middleware('client');

Щоб обмежити доступ до маршруту певними областями, ви можете надати список необхідних областей, розділених комами, при додаванні`client`Middlware до маршруту:

    Route::get('/orders', function (Request $request) {
        ...
    })->middleware('client:check-status,your-scope');

<a name="retrieving-tokens"></a>

### Отримання жетонів

Щоб отримати маркер за допомогою цього типу надання, зробіть запит до`oauth/token`кінцева точка:

    $guzzle = new GuzzleHttp\Client;

    $response = $guzzle->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'client_credentials',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'scope' => 'your-scope',
        ],
    ]);

    return json_decode((string) $response->getBody(), true)['access_token'];

<a name="personal-access-tokens"></a>

## Токени особистого доступу

Іноді ваші користувачі можуть захотіти видати собі маркери доступу, не проходячи звичайний потік перенаправлення коду авторизації. Дозвіл користувачам видавати маркери собі через інтерфейс вашого додатка може бути корисним для того, щоб дозволити користувачам експериментувати з вашим API, або може слугувати більш простим підходом до видачі маркерів доступу загалом.

<a name="creating-a-personal-access-client"></a>

### Створення клієнта персонального доступу

Перш ніж ваша програма зможе видавати маркери особистого доступу, вам потрібно буде створити клієнт персонального доступу. Ви можете зробити це за допомогою`passport:client`команда за допомогою`--personal`варіант. Якщо ви вже запустили`passport:install`команда, вам не потрібно запускати цю команду:

    php artisan passport:client --personal

Створивши клієнт вашого особистого доступу, помістіть ідентифікатор клієнта та секретне значення у тексті вашого додатка`.env`файл:

    PASSPORT_PERSONAL_ACCESS_CLIENT_ID=client-id-value
    PASSPORT_PERSONAL_ACCESS_CLIENT_SECRET=unhashed-client-secret-value

<a name="managing-personal-access-tokens"></a>

### Керування токенами особистого доступу

Після створення клієнта особистого доступу ви можете видавати маркери для даного користувача за допомогою`createToken`метод на`User`екземпляр моделі.`createToken`Метод приймає ім'я маркера як перший аргумент та необов'язковий масив[сфери дії](#token-scopes)як другий аргумент:

    $user = App\Models\User::find(1);

    // Creating a token without scopes...
    $token = $user->createToken('Token Name')->accessToken;

    // Creating a token with scopes...
    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="personal-access-tokens-json-api"></a>

#### API JSON

Паспорт також включає JSON API для управління маркерами особистого доступу. Ви можете поєднати це зі своїм власним інтерфейсом, щоб запропонувати своїм користувачам інформаційну панель для управління маркерами особистого доступу. Нижче ми розглянемо всі кінцеві точки API для управління маркерами особистого доступу. Для зручності ми будемо використовувати[Аксіос](https://github.com/mzabriskie/axios)для демонстрації надсилання HTTP-запитів до кінцевих точок.

API JSON охороняється`web`і`auth`Middlware; тому його можна викликати лише з вашої власної програми. Його не можна викликати із зовнішнього джерела.

<a name="get-oauthscopes"></a>

#### `GET /oauth/scopes`

Цей маршрут повертає всі[сфери дії](#token-scopes)визначено для вашої програми. Ви можете використовувати цей маршрут для переліку областей, які користувач може призначити особистому маркеру доступу:

    axios.get('/oauth/scopes')
        .then(response => {
            console.log(response.data);
        });

<a name="get-oauthpersonal-access-tokens"></a>

#### `GET /oauth/personal-access-tokens`

Цей маршрут повертає всі маркери особистого доступу, які створив аутентифікований користувач. Це в першу чергу корисно для переліку всіх токенів користувача, щоб вони могли їх редагувати або відкликати:

    axios.get('/oauth/personal-access-tokens')
        .then(response => {
            console.log(response.data);
        });

<a name="post-oauthpersonal-access-tokens"></a>

#### `POST /oauth/personal-access-tokens`

Цей маршрут створює нові маркери особистого доступу. Для цього потрібні дві частини даних: маркер`name`та`scopes`що слід призначити маркеру:

    const data = {
        name: 'Token Name',
        scopes: []
    };

    axios.post('/oauth/personal-access-tokens', data)
        .then(response => {
            console.log(response.data.accessToken);
        })
        .catch (response => {
            // List errors on response...
        });

<a name="delete-oauthpersonal-access-tokenstoken-id"></a>

#### `DELETE /oauth/personal-access-tokens/{token-id}`

Цей маршрут може бути використаний для відкликання жетонів особистого доступу:

    axios.delete('/oauth/personal-access-tokens/' + tokenId);

<a name="protecting-routes"></a>

## Захист маршрутів

<a name="via-middleware"></a>

### Через Middleware

Паспорт включає[аутентифікаційний охоронець](/docs/{{version}}/authentication#adding-custom-guards)що перевірить маркери доступу на вхідні запити. Після налаштування`api`охоронець для використання`passport`драйвера, вам потрібно лише вказати`auth:api`Middlware на будь-яких маршрутах, які вимагають дійсного маркера доступу:

    Route::get('/user', function () {
        //
    })->middleware('auth:api');

<a name="multiple-authentication-guards"></a>

#### Кілька захисників автентифікації

Якщо ваша програма автентифікує різні типи користувачів, які, можливо, використовують абсолютно різні моделі Eloquent, вам, ймовірно, доведеться визначити конфігурацію охорони для кожного типу постачальника користувачів у вашій програмі. Це дозволяє захистити запити, призначені для конкретних постачальників користувачів. Наприклад, враховуючи наступну конфігурацію охорони`config/auth.php`файл конфігурації:

    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],

    'api-customers' => [
        'driver' => 'passport',
        'provider' => 'customers',
    ],

Наступний маршрут буде використовувати`api-customers`гвардія, яка використовує`customers`постачальника користувачів для автентифікації вхідних запитів:

    Route::get('/customer', function () {
        //
    })->middleware('auth:api-customers');

> {tip} Щоб отримати додаткову інформацію про використання кількох постачальників послуг із Passport, зверніться до[документація про надання пароля](#customizing-the-user-provider).

<a name="passing-the-access-token"></a>

### Передача маркера доступу

Під час виклику маршрутів, захищених Passport, споживачі API вашого додатку повинні вказати свій маркер доступу як`Bearer`лексема в`Authorization`заголовок їх запиту. Наприклад, під час використання бібліотеки Guzzle HTTP&#x3A;

    $response = $client->request('GET', '/api/user', [
        'headers' => [
            'Accept' => 'application/json',
            'Authorization' => 'Bearer '.$accessToken,
        ],
    ]);

<a name="token-scopes"></a>

## Токени

Області дозволяють вашим клієнтам API запитувати певний набір дозволів під час запиту авторизації для доступу до облікового запису. Наприклад, якщо ви створюєте додаток для електронної комерції, не всі споживачі API потребуватимуть можливості робити замовлення. Натомість ви можете дозволити споживачам вимагати лише дозволу на доступ до статусів відвантаження замовлення. Іншими словами, сфери дії дозволяють користувачам вашого додатку обмежувати дії, які сторонні програми можуть виконувати від їх імені.

<a name="defining-scopes"></a>

### Визначення обсягу

Ви можете визначити область застосування вашого API за допомогою`Passport::tokensCan`метод у`boot`метод вашого`AuthServiceProvider`.`tokensCan`метод приймає масив імен областей та описів області. Опис області може бути будь-яким, що забажаєте, і відображатиметься користувачам на екрані затвердження авторизації:

    use Laravel\Passport\Passport;

    Passport::tokensCan([
        'place-orders' => 'Place orders',
        'check-status' => 'Check order status',
    ]);

<a name="default-scope"></a>

### Область за замовчуванням

Якщо клієнт не вимагає будь-яких конкретних областей застосування, ви можете налаштувати сервер Passport на приєднання до маркера області за замовчуванням за допомогою`setDefaultScope`метод. Як правило, вам слід викликати цей метод із`boot`метод вашого`AuthServiceProvider`:

    use Laravel\Passport\Passport;

    Passport::setDefaultScope([
        'check-status',
        'place-orders',
    ]);

<a name="assigning-scopes-to-tokens"></a>

### Присвоєння сфери дії маркерам

<a name="when-requesting-authorization-codes"></a>

#### Під час запиту кодів авторизації

Запитуючи маркер доступу з використанням дозволу коду авторизації, споживачі повинні вказати бажаний обсяг як`scope`параметр рядка запиту.`scope`параметром повинен бути розділений пробілом список областей:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => 'place-orders check-status',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

<a name="when-issuing-personal-access-tokens"></a>

#### При видачі токенів особистого доступу

Якщо ви видаєте маркери особистого доступу за допомогою`User`моделі`createToken`методу, ви можете передати масив бажаних областей дії як другий аргумент методу:

    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="checking-scopes"></a>

### Перевірка обсягу

Паспорт включає два проміжні програми, які можна використовувати для перевірки автентичності вхідного запиту за допомогою маркера, якому було надано задану область дії. Для початку додайте наступне Middlware до`$routeMiddleware`власність вашого`app/Http/Kernel.php`файл:

    'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
    'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,

<a name="check-for-all-scopes"></a>

#### Перевірте всі сфери

`scopes`Middlware може бути призначене маршруту для перевірки наявності маркера доступу вхідного запиту_всі_з перелічених областей застосування:

    Route::get('/orders', function () {
        // Access token has both "check-status" and "place-orders" scopes...
    })->middleware(['auth:api', 'scopes:check-status,place-orders']);

<a name="check-for-any-scopes"></a>

#### Перевірте будь-які сфери застосування

`scope`Middlware може бути призначене маршруту для перевірки наявності маркера доступу вхідного запиту_принаймні один_з перелічених областей застосування:

    Route::get('/orders', function () {
        // Access token has either "check-status" or "place-orders" scope...
    })->middleware(['auth:api', 'scope:check-status,place-orders']);

<a name="checking-scopes-on-a-token-instance"></a>

#### Перевірка обсягу на екземплярі маркера

Після того, як запит автентифікації маркера доступу увійшов у вашу програму, ви все ще можете перевірити, чи має маркер задану область застосування, використовуючи`tokenCan`метод на аутентифікованому`User`примірник:

    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        if ($request->user()->tokenCan('place-orders')) {
            //
        }
    });

<a name="additional-scope-methods"></a>

#### Додаткові методи застосування

`scopeIds`метод поверне масив усіх визначених ідентифікаторів / імен:

    Laravel\Passport\Passport::scopeIds();

`scopes`Метод повертає масив усіх визначених областей як екземпляри`Laravel\Passport\Scope`:

    Laravel\Passport\Passport::scopes();

`scopesFor`метод поверне масив`Laravel\Passport\Scope`екземпляри, що відповідають заданим ідентифікаторам / іменам:

    Laravel\Passport\Passport::scopesFor(['place-orders', 'check-status']);

Ви можете визначити, чи визначена дана область застосування, використовуючи`hasScope`метод:

    Laravel\Passport\Passport::hasScope('place-orders');

<a name="consuming-your-api-with-javascript"></a>

## Споживання вашого API за допомогою JavaScript

Створюючи API, може бути надзвичайно корисно мати можливість використовувати ваш власний API із вашої програми JavaScript. Цей підхід до розробки API дозволяє вашому власному додатку використовувати той самий API, яким ви ділитесь зі світом. Той самий API може використовуватися вашою веб-програмою, мобільними програмами, сторонніми програмами та будь-якими SDK, які ви можете публікувати на різних менеджерах пакетів.

Як правило, якщо ви хочете використовувати ваш API із вашої програми JavaScript, вам доведеться вручну надіслати маркер доступу до програми та передати його з кожним запитом до вашої програми. Однак Passport включає Middlware, яке може це впоратись із вами. Все, що вам потрібно зробити, це додати`CreateFreshApiToken`Middlware для вашого`web`група проміжного програмного забезпечення у вашому`app/Http/Kernel.php`файл:

    'web' => [
        // Other middleware...
        \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
    ],

> {note} Ви повинні переконатися, що`CreateFreshApiToken`Middlware є останнім проміжним програмним забезпеченням, переліченим у вашому стеку проміжного програмного забезпечення.

До цього проміжного програмного забезпечення Passport буде додано файл`laravel_token`cookie до ваших вихідних відповідей. Цей файл cookie містить зашифрований JWT, який Passport буде використовувати для автентифікації запитів API із вашої програми JavaScript. Термін служби JWT дорівнює вашому`session.lifetime`значення конфігурації. Тепер ви можете надсилати запити до API вашого додатка, не передаючи явно маркер доступу:

    axios.get('/api/user')
        .then(response => {
            console.log(response.data);
        });

<a name="customizing-the-cookie-name"></a>

#### Налаштування назви файлу cookie

Якщо потрібно, ви можете налаштувати`laravel_token`ім'я файлу cookie з використанням`Passport::cookie`метод. Як правило, цей метод слід викликати з`boot`метод вашого`AuthServiceProvider`:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::cookie('custom_name');
    }

<a name="csrf-protection"></a>

#### Захист CSRF

Використовуючи цей метод автентифікації, вам потрібно буде переконатися, що у ваші запити включено дійсний заголовок токена CSRF. Підмостки за замовчуванням JavaScript включають екземпляр Axios, який автоматично використовуватиме зашифрований`XSRF-TOKEN`значення файлу cookie для відправки`X-XSRF-TOKEN`заголовок на запити того самого походження.

> {tip} Якщо ви вирішите надіслати файл`X-CSRF-TOKEN`заголовок замість`X-XSRF-TOKEN`, вам потрібно буде використовувати незашифрований маркер, наданий`csrf_token()`.

<a name="events"></a>

## Події

Passport викликає події при видачі маркерів доступу та оновлення маркерів. Ви можете використовувати ці події, щоб обрізати або скасувати інші маркери доступу у вашій базі даних. Ви можете приєднати слухачів до цих подій у програмі своєї програми`EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Laravel\Passport\Events\AccessTokenCreated' => [
            'App\Listeners\RevokeOldTokens',
        ],

        'Laravel\Passport\Events\RefreshTokenCreated' => [
            'App\Listeners\PruneOldTokens',
        ],
    ];

<a name="testing"></a>

## Тестування

Паспорт`actingAs`метод може бути використаний для вказівки поточного аутентифікованого користувача, а також його обсягу. Перший аргумент, наданий`actingAs`метод є екземпляром користувача, а другий - масивом областей, які повинні бути надані маркеру користувача:

    use App\Models\User;
    use Laravel\Passport\Passport;

    public function testServerCreation()
    {
        Passport::actingAs(
            User::factory()->create(),
            ['create-servers']
        );

        $response = $this->post('/api/create-server');

        $response->assertStatus(201);
    }

Паспорт`actingAsClient`метод може бути використаний для вказівки поточного аутентифікованого клієнта, а також його обсягу. Перший аргумент, наданий`actingAsClient`метод є екземпляром клієнта, а другий - масивом областей, які повинні бути надані маркеру клієнта:

    use Laravel\Passport\Client;
    use Laravel\Passport\Passport;

    public function testGetOrders()
    {
        Passport::actingAsClient(
            Client::factory()->create(),
            ['check-status']
        );

        $response = $this->get('/api/orders');

        $response->assertStatus(200);
    }
