# Відповіді HTTP

-   [Створення відповідей](#creating-responses)
    -   [Додавання заголовків до відповідей](#attaching-headers-to-responses)
    -   [Прикріплення файлів cookie до відповідей](#attaching-cookies-to-responses)
    -   [Файли cookie та шифрування](#cookies-and-encryption)
-   [Перенаправлення](#redirects)
    -   [Перенаправлення на іменовані маршрути](#redirecting-named-routes)
    -   [Перенаправлення на дії контролера](#redirecting-controller-actions)
    -   [Перенаправлення на зовнішні домени](#redirecting-external-domains)
    -   [Redirecting With Flashed Session Data](#redirecting-with-flashed-session-data)
-   [Інші типи відповідей](#other-response-types)
    -   [Переглянути відповіді](#view-responses)
    -   [Відповіді JSON](#json-responses)
    -   [Завантаження файлів](#file-downloads)
    -   [Файл Відповіді](#file-responses)
-   [Макроси відповіді](#response-macros)

<a name="creating-responses"></a>

## Створення відповідей

<a name="strings-arrays"></a>

#### Струни та масиви

Усі маршрути та контролери повинні повертати відповідь, яку потрібно відправити назад у браузер користувача. Laravel пропонує кілька різних способів повернути відповіді. Найпростіша відповідь - повернення рядка з маршруту або контролера. Фреймворк автоматично перетворить рядок у повну відповідь HTTP&#x3A;

    Route::get('/', function () {
        return 'Hello World';
    });

На додаток до повернення рядків із ваших маршрутів та контролерів, ви також можете повернути масиви. Фреймворк автоматично перетворить масив у відповідь JSON:

    Route::get('/', function () {
        return [1, 2, 3];
    });

> {tip} Чи знали ви, що можете також повернутися[Красномовні колекції](/docs/{{version}}/eloquent-collections)зі своїх маршрутів чи контролерів? Вони будуть автоматично перетворені в JSON. Спробуй!

<a name="response-objects"></a>

#### Об'єкти відповіді

Як правило, ви не просто повертаєте прості рядки або масиви з ваших дій маршруту. Натомість ти повернешся повним`Illuminate\Http\Response`екземпляри або[погляди](/docs/{{version}}/views).

Повернення повного`Response`екземпляр дозволяє налаштувати код стану HTTP та заголовки відповіді. A`Response`екземпляр успадковує від`Symfony\Component\HttpFoundation\Response` class, which provides a variety of methods for building HTTP responses:

    Route::get('home', function () {
        return response('Hello World', 200)
                      ->header('Content-Type', 'text/plain');
    });

<a name="attaching-headers-to-responses"></a>

#### Додавання заголовків до відповідей

Майте на увазі, що більшість методів реагування є ланцюжковими, що дозволяє швидко будувати екземпляри відповідей. Наприклад, ви можете використовувати`header`спосіб додати серію заголовків до відповіді перед тим, як відправити її назад користувачеві:

    return response($content)
                ->header('Content-Type', $type)
                ->header('X-Header-One', 'Header Value')
                ->header('X-Header-Two', 'Header Value');

Або ви можете використовувати`withHeaders`метод, щоб вказати масив заголовків, які потрібно додати до відповіді:

    return response($content)
                ->withHeaders([
                    'Content-Type' => $type,
                    'X-Header-One' => 'Header Value',
                    'X-Header-Two' => 'Header Value',
                ]);

<a name="cache-control-middleware"></a>

#### Керування кешем Middlware

Laravel включає a`cache.headers`Middlware, яке можна використовувати для швидкого встановлення`Cache-Control`заголовок для групи маршрутів. Якщо`etag`вказаний у списку директив, хеш MD5 вмісту відповіді буде автоматично встановлений як ідентифікатор ETag:

    Route::middleware('cache.headers:public;max_age=2628000;etag')->group(function () {
        Route::get('privacy', function () {
            // ...
        });

        Route::get('terms', function () {
            // ...
        });
    });

<a name="attaching-cookies-to-responses"></a>

#### Прикріплення файлів cookie до відповідей

`cookie`метод на примірниках відповідей дозволяє легко приєднати до відповіді файли cookie. Наприклад, ви можете використовувати`cookie`метод генерувати cookie і вільно приєднувати його до екземпляра відповіді так:

    return response($content)
                    ->header('Content-Type', $type)
                    ->cookie('name', 'value', $minutes);

`cookie`метод також приймає ще кілька аргументів, які використовуються рідше. Як правило, ці аргументи мають ту саму мету та значення, що й аргументи, які будуть надані рідному PHP[setcookie](https://secure.php.net/manual/en/function.setcookie.php)метод:

    ->cookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)

Крім того, ви можете використовувати`Cookie`фасад до файлів cookie "в черзі" для прикріплення до вихідної відповіді вашої програми.`queue`метод приймає a`Cookie`екземпляр або аргументи, необхідні для створення a`Cookie`екземпляр. Ці файли cookie буде додано до вихідної відповіді до того, як її буде надіслано браузеру:

    Cookie::queue(Cookie::make('name', 'value', $minutes));

    Cookie::queue('name', 'value', $minutes);

<a name="cookies-and-encryption"></a>

#### Файли cookie та шифрування

За замовчуванням усі файли cookie, створені Laravel, шифруються та підписуються, щоб клієнт не міг їх змінити чи прочитати. Якщо ви хочете відключити шифрування для підмножини файлів cookie, створених вашою програмою, ви можете використовувати`$except`власність`App\Http\Middleware\EncryptCookies`Middlware, яке знаходиться в`app/Http/Middleware`каталог:

    /**
     * The names of the cookies that should not be encrypted.
     *
     * @var array
     */
    protected $except = [
        'cookie_name',
    ];

<a name="redirects"></a>

## Перенаправлення

Переадресаційні відповіді є прикладами`Illuminate\Http\RedirectResponse`класу та містять відповідні заголовки, необхідні для перенаправлення користувача на іншу URL-адресу. Є кілька способів генерувати a`RedirectResponse`екземпляр. Найпростіший метод - використання глобального`redirect`помічник:

    Route::get('dashboard', function () {
        return redirect('home/dashboard');
    });

Іноді вам може знадобитися перенаправити користувача у попереднє місце розташування, наприклад, коли надіслана форма недійсна. Ви можете зробити це, використовуючи глобальний`back`допоміжна функція. Оскільки ця функція використовує[сесія](/docs/{{version}}/session), переконайтеся, що маршрут викликає`back`функція використовує`web`групу проміжного програмного забезпечення або застосовує всі проміжні програми сеансу:

    Route::post('user/profile', function () {
        // Validate the request...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>

### Перенаправлення на іменовані маршрути

Коли ви телефонуєте на`redirect`помічник без параметрів, екземпляр`Illuminate\Routing\Redirector`повертається, що дозволяє вам викликати будь-який метод на`Redirector`екземпляр. Наприклад, для створення`RedirectResponse`до вказаного маршруту, ви можете використовувати`route`метод:

    return redirect()->route('login');

Якщо ваш маршрут має параметри, ви можете передати їх як другий аргумент до`route` method:

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', ['id' => 1]);

<a name="populating-parameters-via-eloquent-models"></a>

#### Заповнення параметрів за допомогою Eloquent моделей

Якщо ви переспрямовуєте на маршрут із параметром "ID", який заповнюється з моделі Eloquent, ви можете передати саму модель. Ідентифікатор буде витягнуто автоматично:

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', [$user]);

Якщо ви хочете налаштувати значення, яке розміщується в параметрі маршруту, ви можете вказати стовпець у визначенні параметра маршруту (`profile/{id:slug}`) або ви можете замінити`getRouteKey`метод на вашій красномовній моделі:

    /**
     * Get the value of the model's route key.
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>

### Перенаправлення на дії контролера

Ви також можете генерувати переспрямування на[дії контролера](/docs/{{version}}/controllers). Для цього передайте контролер та ім'я дії в`action`метод:

    use App\Http\Controllers\HomeController;

    return redirect()->action([HomeController::class, 'index']);

Якщо ваш маршрут контролера вимагає параметрів, ви можете передати їх як другий аргумент до`action`метод:

    return redirect()->action(
        [UserController::class, 'profile'], ['id' => 1]
    );

<a name="redirecting-external-domains"></a>

### Перенаправлення на зовнішні домени

Іноді вам може знадобитися перенаправити на домен за межами вашої програми. Ви можете зробити це, зателефонувавши до`away`метод, який створює a`RedirectResponse`без будь-якого додаткового кодування, перевірки чи перевірки URL-адрес:

    return redirect()->away('https://www.google.com');

<a name="redirecting-with-flashed-session-data"></a>

### Переадресація за допомогою прошитих даних сеансу

Перенаправлення на нову URL-адресу та[перепрошивка даних до сеансу](/docs/{{version}}/session#flash-data)зазвичай робляться одночасно. Як правило, це робиться після успішного виконання дії під час прошивки повідомлення про успіх до сеансу. Для зручності ви можете створити файл`RedirectResponse`дані екземпляра та флеш-пам’яті для сеансу в одному, вільному ланцюжку методів:

    Route::post('user/profile', function () {
        // Update the user's profile...

        return redirect('dashboard')->with('status', 'Profile updated!');
    });

Після перенаправлення користувача ви можете відобразити блимає повідомлення з[сесія](/docs/{{version}}/session). Наприклад, використовуючи[Леза синтаксису](/docs/{{version}}/blade):

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif

<a name="other-response-types"></a>

## Інші типи відповідей

`response`helper може використовуватися для генерації інших типів екземплярів відповідей. Коли`response`helper викликається без аргументів, реалізація`Illuminate\Contracts\Routing\ResponseFactory`[контракт](/docs/{{version}}/contracts)повертається. Цей контракт надає кілька корисних методів для отримання відповідей.

<a name="view-responses"></a>

### Переглянути відповіді

Якщо вам потрібен контроль над статусом відповіді та заголовками, але вам також потрібно повернути a[вид](/docs/{{version}}/views)як зміст відповіді слід використовувати`view`метод:

    return response()
                ->view('hello', $data, 200)
                ->header('Content-Type', $type);

Звичайно, якщо вам не потрібно передавати власний код стану HTTP або власні заголовки, вам слід використовувати глобальний`view`допоміжна функція.

<a name="json-responses"></a>

### Відповіді JSON

The `json`метод автоматично встановить`Content-Type`заголовок до`application/json`, а також перетворити даний масив у JSON за допомогою`json_encode`Функція PHP:

    return response()->json([
        'name' => 'Abigail',
        'state' => 'CA',
    ]);

Якщо ви хочете створити відповідь JSONP, ви можете використовувати`json`метод у поєднанні з`withCallback`метод:

    return response()
                ->json(['name' => 'Abigail', 'state' => 'CA'])
                ->withCallback($request->input('callback'));

<a name="file-downloads"></a>

### Завантаження файлів

`download`метод може бути використаний для генерації відповіді, що змушує браузер користувача завантажувати файл за вказаним шляхом.`download`метод приймає ім'я файлу як другий аргумент методу, який визначатиме ім'я файлу, яке бачить користувач, який завантажує файл. Нарешті, ви можете передати масив заголовків HTTP як третій аргумент методу:

    return response()->download($pathToFile);

    return response()->download($pathToFile, $name, $headers);

    return response()->download($pathToFile)->deleteFileAfterSend();

> {note} Symfony HttpFoundation, який керує завантаженнями файлів, вимагає, щоб завантажуваний файл мав назву файлу ASCII.

<a name="streamed-downloads"></a>

#### Потокові завантаження

Іноді вам може знадобитися перетворити рядкову відповідь даної операції на відповідь, яку можна завантажити без необхідності записувати вміст операції на диск. Ви можете використовувати`streamDownload`метод у цьому сценарії. Цей метод приймає як аргументи зворотний виклик, ім'я файлу та необов'язковий масив заголовків:

    return response()->streamDownload(function () {
        echo GitHub::api('repo')
                    ->contents()
                    ->readme('laravel', 'laravel')['contents'];
    }, 'laravel-readme.md');

<a name="file-responses"></a>

### Файл Відповіді

`file`Метод може бути використаний для відображення файлу, наприклад, зображення або PDF, безпосередньо у браузері користувача, а не для ініціювання завантаження. Цей метод приймає шлях до файлу як перший аргумент, а масив заголовків - як другий аргумент:

    return response()->file($pathToFile);

    return response()->file($pathToFile, $headers);

<a name="response-macros"></a>

## Макроси відповіді

Якщо ви хочете визначити спеціальну відповідь, яку ви можете повторно використовувати в різних своїх маршрутах та контролерах, ви можете використовувати`macro`метод на`Response`фасад. Наприклад, з a[постачальника послуг](/docs/{{version}}/providers) `boot`метод:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Response;
    use Illuminate\Support\ServiceProvider;

    class ResponseMacroServiceProvider extends ServiceProvider
    {
        /**
         * Register the application's response macros.
         *
         * @return void
         */
        public function boot()
        {
            Response::macro('caps', function ($value) {
                return Response::make(strtoupper($value));
            });
        }
    }

`macro`Функція приймає ім'я як перший аргумент, а Закриття як другий аргумент. Закриття макросу буде виконано при виклику імені макроса з`ResponseFactory`впровадження або`response`помічник:

    return response()->caps('foo');
