# Запити HTTP

-   [Доступ до запиту](#accessing-the-request)
    -   [Шлях і метод запиту](#request-path-and-method)
    -   [Запити PSR-7](#psr7-requests)
-   [Обрізання та нормалізація введення](#input-trimming-and-normalization)
-   [Отримання вводу](#retrieving-input)
    -   [Старий вхід](#old-input)
    -   [Печиво](#cookies)
-   [Файли](#files)
    -   [Отримання завантажених файлів](#retrieving-uploaded-files)
    -   [Зберігання завантажених файлів](#storing-uploaded-files)
-   [Налаштування надійних проксі](#configuring-trusted-proxies)

<a name="accessing-the-request"></a>

## Доступ до запиту

Щоб отримати екземпляр поточного запиту HTTP за допомогою ін'єкції залежностей, слід ввести натяк на`Illuminate\Http\Request`клас за вашим методом контролера. Екземпляр вхідного запиту буде автоматично введений[службовий контейнер](/docs/{{version}}/container):

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Store a new user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $name = $request->input('name');

            //
        }
    }

<a name="dependency-injection-route-parameters"></a>

#### Параметри ін’єкції залежності та маршруту

Якщо ваш метод контролера також очікує введення від параметра маршруту, вам слід вказати параметри маршруту після інших залежностей. Наприклад, якщо ваш маршрут визначено так:

    use App\Http\Controllers\UserController;

    Route::put('user/{id}', [UserController::class, 'update']);

Ви все ще можете навести натяк на`Illuminate\Http\Request`і отримати доступ до параметра маршруту`id`визначивши метод свого контролера наступним чином:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Update the specified user.
         *
         * @param  Request  $request
         * @param  string  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }

<a name="accessing-the-request-via-route-closures"></a>

#### Доступ до запиту через перекриття маршрутів

Ви також можете ввести натяк на`Illuminate\Http\Request`клас на маршруті Закриття. Службовий контейнер автоматично вводить вхідний запит в Закриття при його виконанні:

    use Illuminate\Http\Request;

    Route::get('/', function (Request $request) {
        //
    });

<a name="request-path-and-method"></a>

### Шлях і метод запиту

`Illuminate\Http\Request`екземпляр надає різноманітні методи для перевірки HTTP-запиту для вашої програми та розширює`Symfony\Component\HttpFoundation\Request`клас. Нижче ми обговоримо кілька найважливіших методів.

<a name="retrieving-the-request-path"></a>

#### Отримання шляху запиту

`path`метод повертає інформацію про шлях запиту. Отже, якщо вхідний запит націлений на`http://domain.com/foo/bar`,`path`метод повернеться`foo/bar`:

    $uri = $request->path();

`is`метод дозволяє перевірити, що шлях вхідного запиту відповідає заданому шаблону. Ви можете використовувати`*`символ як підстановочний знак при використанні цього методу:

    if ($request->is('admin/*')) {
        //
    }

<a name="retrieving-the-request-url"></a>

#### Отримання URL-адреси запиту

Щоб отримати повну URL-адресу вхідного запиту, ви можете використовувати`url`або`fullUrl`методи.`url`метод поверне URL-адресу без рядка запиту, тоді як`fullUrl`метод включає рядок запиту:

    // Without Query String...
    $url = $request->url();

    // With Query String...
    $url = $request->fullUrl();

<a name="retrieving-the-request-method"></a>

#### Отримання методу запиту

`method`метод поверне для запиту дієслово HTTP. Ви можете використовувати`isMethod`спосіб перевірити, що дієслово HTTP відповідає заданому рядку:

    $method = $request->method();

    if ($request->isMethod('post')) {
        //
    }

<a name="psr7-requests"></a>

### Запити PSR-7

[Стандарт PSR-7](https://www.php-fig.org/psr/psr-7/)визначає інтерфейси для повідомлень HTTP, включаючи запити та відповіді. Якщо ви хочете отримати екземпляр запиту PSR-7 замість запиту Laravel, спочатку потрібно встановити кілька бібліотек. Laravel використовує_Міст повідомлень HTTP Symfony_компонент для перетворення типових запитів та відповідей Laravel у реалізації, сумісні з PSR-7:

    composer require symfony/psr-http-message-bridge
    composer require nyholm/psr7

Після встановлення цих бібліотек ви можете отримати запит PSR-7, натякнувши тип інтерфейсу запиту на ваш маршрут Закриття або метод контролера:

    use Psr\Http\Message\ServerRequestInterface;

    Route::get('/', function (ServerRequestInterface $request) {
        //
    });

> {tip} Якщо ви повернете екземпляр відповіді PSR-7 з маршруту або контролера, він автоматично буде перетворений назад у екземпляр відповіді Laravel і відображатиметься в рамках.

<a name="input-trimming-and-normalization"></a>

## Обрізання та нормалізація введення

За замовчуванням Laravel включає`TrimStrings`і`ConvertEmptyStringsToNull`Middlware у глобальному стеку проміжного програмного забезпечення вашої програми. Це Middlware перелічено в стеку`App\Http\Kernel`клас. Це Middlware автоматично обрізає всі поля вхідних рядків у запиті, а також перетворить будь-які порожні поля рядків у`null`. Це дозволяє вам не турбуватися про ці проблеми з нормалізацією у ваших маршрутах та контролерах.

Якщо ви хочете відключити цю поведінку, ви можете видалити два проміжні програми зі стеку проміжного програмного забезпечення вашої програми, видаливши їх із`$middleware`власність вашого`App\Http\Kernel`клас.

<a name="retrieving-input"></a>

## Отримання вводу

<a name="retrieving-all-input-data"></a>

#### Отримання всіх вхідних даних

Ви також можете отримати всі вхідні дані як`array`за допомогою`all`метод:

    $input = $request->all();

<a name="retrieving-an-input-value"></a>

#### Отримання вхідного значення

За допомогою кількох простих методів ви можете отримати доступ до всіх введених користувачем даних із вашого`Illuminate\Http\Request`екземпляр, не турбуючись про те, яке дієслово HTTP було використано для запиту. Незалежно від дієслова HTTP,`input`метод може бути використаний для отримання вводу користувача:

    $name = $request->input('name');

Ви можете передати значення за замовчуванням як другий аргумент в`input`метод. Це значення буде повернуто, якщо запитане вхідне значення відсутнє у запиті:

    $name = $request->input('name', 'Sally');

Під час роботи з формами, що містять вхідні дані масиву, використовуйте позначення «крапка» для доступу до масивів:

    $name = $request->input('products.0.name');

    $names = $request->input('products.*.name');

Ви можете зателефонувати до`input`метод без будь-яких аргументів для отримання всіх вхідних значень як асоціативний масив:

    $input = $request->input();

<a name="retrieving-input-from-the-query-string"></a>

#### Отримання вхідних даних із рядка запиту

Поки`input`метод отримує значення з усього корисного набору запиту (включаючи рядок запиту),`query`метод отримуватиме лише значення із рядка запиту:

    $name = $request->query('name');

Якщо запитаних даних значення рядка запиту немає, буде повернено другий аргумент цього методу:

    $name = $request->query('name', 'Helen');

Ви можете зателефонувати до`query`метод без будь-яких аргументів, щоб отримати всі значення рядка запиту як асоціативний масив:

    $query = $request->query();

<a name="retrieving-input-via-dynamic-properties"></a>

#### Отримання вводу через динамічні властивості

You may also access user input using dynamic properties on the `Illuminate\Http\Request`інстанції. Наприклад, якщо одна з форм вашої програми містить`name`поле, ви можете отримати доступ до значення поля таким чином:

    $name = $request->name;

При використанні динамічних властивостей Laravel спочатку шукатиме значення параметра в корисному навантаженні запиту. Якщо його немає, Laravel буде шукати поле в параметрах маршруту.

<a name="retrieving-json-input-values"></a>

#### Отримання вхідних значень JSON

Надсилаючи запити JSON до вашої програми, ви можете отримати доступ до даних JSON через`input`метод, доки`Content-Type`Заголовок запиту правильно встановлений`application/json`. Ви навіть можете використовувати синтаксис "крапка", щоб копатись у масивах JSON:

    $name = $request->input('user.name');

<a name="retrieving-boolean-input-values"></a>

#### Отримання булевих вхідних значень

При роботі з елементами HTML, такими як прапорці, ваш додаток може отримувати "істинні" значення, які насправді є рядками. Наприклад, "true" або "on". Для зручності ви можете використовувати`boolean`метод отримати ці значення як логічні.`boolean`метод повертає`true`для 1, "1", true, "true", "on" та "yes". Усі інші значення повернуться`false`:

    $archived = $request->boolean('archived');

<a name="retrieving-a-portion-of-the-input-data"></a>

#### Отримання частини вхідних даних

Якщо вам потрібно отримати підмножину вхідних даних, ви можете використовувати`only`і`except`методи. Обидва ці методи приймають один`array`або динамічний список аргументів:

    $input = $request->only(['username', 'password']);

    $input = $request->only('username', 'password');

    $input = $request->except(['credit_card']);

    $input = $request->except('credit_card');

> {tip} The`only`метод повертає всі пари ключ / значення, які ви запитуєте; однак він не поверне пари ключ / значення, яких немає у запиті.

<a name="determining-if-an-input-value-is-present"></a>

#### Визначення наявності вхідного значення

Ви повинні використовувати`has`метод, щоб визначити, чи є значення в запиті.`has`метод повертає`true`якщо значення присутнє на запит:

    if ($request->has('name')) {
        //
    }

Коли дається масив,`has`метод визначає, чи є всі вказані значення:

    if ($request->has(['name', 'email'])) {
        //
    }

`whenHas`метод виконає заданий зворотний виклик, якщо у запиті присутнє значення:

    $request->whenHas('name', function ($input) {
        //
    });

`hasAny`метод повертає`true`якщо є якесь із зазначених значень:

    if ($request->hasAny(['name', 'email'])) {
        //
    }

Якщо ви хочете визначити, чи є значення у запиті і не є порожнім, ви можете використовувати`filled`метод:

    if ($request->filled('name')) {
        //
    }

`whenFilled`метод виконає даний зворотний виклик, якщо в запиті присутнє значення, яке не є порожнім:

    $request->whenFilled('name', function ($input) {
        //
    });

Щоб визначити, чи відсутній даний ключ у запиті, ви можете використовувати`missing`метод:

    if ($request->missing('name')) {
        //
    }

<a name="old-input"></a>

### Старий вхід

Laravel дозволяє зберегти вхідні дані одного запиту під час наступного запиту. Ця функція особливо корисна для повторного заповнення форм після виявлення помилок перевірки. Однак, якщо ви використовуєте Laravel's[функції перевірки](/docs/{{version}}/validation), навряд чи вам доведеться використовувати ці методи вручну, оскільки деякі вбудовані засоби перевірки Laravel викликатимуть їх автоматично.

<a name="flashing-input-to-the-session"></a>

#### Миготливий вхід на сесію

`flash`метод на`Illuminate\Http\Request`class буде блимати поточним входом до[сесія](/docs/{{version}}/session)щоб він був доступний під час наступного запиту користувача до програми:

    $request->flash();

Ви також можете використовувати`flashOnly`і`flashExcept`методи передачі підмножини даних запиту до сеансу. Ці методи корисні для уникнення конфіденційної інформації, такої як паролі, поза сеансом:

    $request->flashOnly(['username', 'email']);

    $request->flashExcept('password');

<a name="flashing-input-then-redirecting"></a>

#### Миготливе введення та перенаправлення

Оскільки вам часто захочеться прошивати введення в сеанс, а потім перенаправляти на попередню сторінку, ви можете легко прив’язати прошивку введення до перенаправлення за допомогою`withInput`метод:

    return redirect('form')->withInput();

    return redirect('form')->withInput(
        $request->except('password')
    );

<a name="retrieving-old-input"></a>

#### Отримання старого вводу

Щоб отримати блимаючий вхід з попереднього запиту, використовуйте`old`метод на`Request`інстанції.`old`метод витягне попередньо прошиті вхідні дані з[сесія](/docs/{{version}}/session):

    $username = $request->old('username');

Laravel також забезпечує глобальний`old`помічник. Якщо ви відображаєте старий вхід у форматі[Blade шаблон](/docs/{{version}}/blade), зручніше використовувати`old`помічник. Якщо для даного поля не існує старого вводу,`null`буде повернено:

    <input type="text" name="username" value="{{ old('username') }}">

<a name="cookies"></a>

### Печиво

<a name="retrieving-cookies-from-requests"></a>

#### Отримання файлів cookie із запитів

Усі файли cookie, створені фреймворком Laravel, зашифровані та підписані кодом автентифікації, тобто вони вважатимуться недійсними, якщо їх змінив клієнт. Щоб отримати значення файлу cookie із запиту, використовуйте`cookie`метод на`Illuminate\Http\Request`примірник:

    $value = $request->cookie('name');

Крім того, ви можете використовувати`Cookie`фасад для доступу до значень файлів cookie:

    use Illuminate\Support\Facades\Cookie;

    $value = Cookie::get('name');

<a name="attaching-cookies-to-responses"></a>

#### Прикріплення файлів cookie до відповідей

Ви можете додати файл cookie до вихідного`Illuminate\Http\Response`екземпляр за допомогою`cookie`метод. Вам слід передати ім'я, значення та кількість хвилин, коли файл cookie повинен вважатися дійсним для цього методу:

    return response('Hello World')->cookie(
        'name', 'value', $minutes
    );

`cookie`метод також приймає ще кілька аргументів, які використовуються рідше. Як правило, ці аргументи мають ту саму мету та значення, що й аргументи, які будуть надані рідному PHP[setcookie](https://secure.php.net/manual/en/function.setcookie.php)метод:

    return response('Hello World')->cookie(
        'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
    );

Крім того, ви можете використовувати`Cookie`фасад до файлів cookie "в черзі" для прикріплення до вихідної відповіді вашої програми.`queue`метод приймає a`Cookie`екземпляр або аргументи, необхідні для створення a`Cookie`інстанції. Ці файли cookie буде додано до вихідної відповіді до того, як вона буде надіслана браузеру:

    Cookie::queue(Cookie::make('name', 'value', $minutes));

    Cookie::queue('name', 'value', $minutes);

<a name="generating-cookie-instances"></a>

#### Створення екземплярів файлів cookie

Якщо ви хочете створити файл`Symfony\Component\HttpFoundation\Cookie`екземпляр, який може бути наданий екземпляру відповіді пізніше, ви можете використовувати глобальний`cookie`помічник. Цей файл cookie не буде відправлений назад клієнту, якщо він не приєднаний до екземпляра відповіді:

    $cookie = cookie('name', 'value', $minutes);

    return response('Hello World')->cookie($cookie);

<a name="expiring-cookies-early"></a>

#### Термін дії кукі-файлів закінчується достроково

Ви можете видалити файл cookie, закінчивши термін його дії через`forget`метод`Cookie`фасад:

    Cookie::queue(Cookie::forget('name'));

Крім того, ви можете приєднати прострочений файл cookie до екземпляра відповіді:

    $cookie = Cookie::forget('name');

    return response('Hello World')->withCookie($cookie);

<a name="files"></a>

## Файли

<a name="retrieving-uploaded-files"></a>

### Отримання завантажених файлів

Ви можете отримати доступ до завантажених файлів із`Illuminate\Http\Request`екземпляр за допомогою`file`методом або з використанням динамічних властивостей.`file`метод повертає екземпляр`Illuminate\Http\UploadedFile`клас, який розширює PHP`SplFileInfo`клас і надає різноманітні методи взаємодії з файлом:

    $file = $request->file('photo');

    $file = $request->photo;

Ви можете визначити, чи присутній файл у запиті, використовуючи`hasFile`метод:

    if ($request->hasFile('photo')) {
        //
    }

<a name="validating-successful-uploads"></a>

#### Перевірка успішних завантажень

Окрім перевірки наявності файлу, ви можете перевірити, чи не було проблем із завантаженням файлу через`isValid`метод:

    if ($request->file('photo')->isValid()) {
        //
    }

<a name="file-paths-extensions"></a>

#### Шляхи до файлів та розширення

`UploadedFile`клас також містить методи доступу до повноцінного шляху до файлу та його розширення.`extension`метод спробує вгадати розширення файлу на основі його вмісту. Це розширення може відрізнятися від розширення, наданого клієнтом:

    $path = $request->photo->path();

    $extension = $request->photo->extension();

<a name="other-file-methods"></a>

#### Інші методи файлів

Існує безліч інших методів, доступних на`UploadedFile`екземпляри. Перевірте[Документація API для класу](https://api.symfony.com/master/Symfony/Component/HttpFoundation/File/UploadedFile.html)для отримання додаткової інформації щодо цих методів.

<a name="storing-uploaded-files"></a>

### Зберігання завантажених файлів

Для зберігання завантаженого файлу, як правило, ви використовуєте один із налаштованих[файлові системи](/docs/{{version}}/filesystem).`UploadedFile`клас має`store`метод, який перемістить завантажений файл на один із ваших дисків, який може бути місцем у вашій локальній файловій системі або навіть місцем зберігання у хмарі, як Amazon S3.

`store`метод приймає шлях, де файл повинен зберігатися, відносно налаштованого кореневого каталогу файлової системи. Цей шлях не повинен містити ім'я файлу, оскільки унікальний ідентифікатор буде автоматично створений, щоб служити ім'ям файлу.

`store`метод також приймає необов'язковий другий аргумент для імені диска, який слід використовувати для зберігання файлу. Метод повертає шлях до файлу відносно кореня диска:

    $path = $request->photo->store('images');

    $path = $request->photo->store('images', 's3');

Якщо ви не хочете, щоб ім'я файлу створювалося автоматично, ви можете використовувати файл`storeAs`метод, який приймає шлях, ім'я файлу та ім'я диска як свої аргументи:

    $path = $request->photo->storeAs('images', 'filename.jpg');

    $path = $request->photo->storeAs('images', 'filename.jpg', 's3');

<a name="configuring-trusted-proxies"></a>

## Налаштування надійних проксі

Під час запуску ваших програм за допомогою балансувача навантаження, який припиняє дію сертифікатів TLS / SSL, ви можете помітити, що ваша програма іноді не генерує посилання HTTPS. Зазвичай це тому, що ваша програма перенаправляє трафік з балансувача навантаження на порт 80 і не знає, що повинна генерувати безпечні посилання.

Для вирішення цього питання ви можете використовувати`App\Http\Middleware\TrustProxies`Middlware, що входить до вашої програми Laravel, що дозволяє швидко налаштувати балансири навантаження або проксі-сервери, яким ваша програма повинна довіряти. Ваші довірені проксі-сервери повинні бути вказані як масив на`$proxies`властивість цього проміжного програмного забезпечення. Окрім налаштування надійних проксі-серверів, ви можете налаштувати і проксі-сервер`$headers`якому слід довіряти:

    <?php

    namespace App\Http\Middleware;

    use Fideloper\Proxy\TrustProxies as Middleware;
    use Illuminate\Http\Request;

    class TrustProxies extends Middleware
    {
        /**
         * The trusted proxies for this application.
         *
         * @var string|array
         */
        protected $proxies = [
            '192.168.1.1',
            '192.168.1.2',
        ];

        /**
         * The headers that should be used to detect proxies.
         *
         * @var int
         */
        protected $headers = Request::HEADER_X_FORWARDED_ALL;
    }

> {tip} Якщо ви використовуєте AWS Elastic Balancing Load, ваш`$headers`значення має бути`Request::HEADER_X_FORWARDED_AWS_ELB`. Для отримання додаткової інформації про константи, які можуть бути використані в`$headers`, перегляньте документацію Symfony щодо[довірливі довірені особи](https://symfony.com/doc/current/deployment/proxies.html).

<a name="trusting-all-proxies"></a>

#### Довіра всім довіреним особам

Якщо ви використовуєте Amazon AWS або іншого "хмарного" постачальника балансирів навантаження, ви можете не знати IP-адреси своїх фактичних балансирів. У цьому випадку ви можете використовувати`*`довіряти всім довіреним особам:

    /**
     * The trusted proxies for this application.
     *
     * @var string|array
     */
    protected $proxies = '*';
