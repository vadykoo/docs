# Клієнт HTTP

-   [Вступ](#introduction)
-   [View запитів](#making-requests)
    -   [Запит даних](#request-data)
    -   [Заголовки](#headers)
    -   [Аутентифікація](#authentication)
    -   [Timeout](#timeout)
    -   [Повторні спроби](#retries)
    -   [Обробка помилок](#error-handling)
    -   [Options Guzzle](#guzzle-options)
-   [Тестування](#testing)
    -   [Фейкові відповіді](#faking-responses)
    -   [Перевірка запитів](#inspecting-requests)

<a name="introduction"></a>

## Вступ

Laravel забезпечує виразний, мінімальний API навколо[Guzzle HTTP-клієнт](http://docs.guzzlephp.org/en/stable/), що дозволяє швидко робити вихідні HTTP-запити для спілкування з іншими веб-програмами. Обгортка Laravel навколо Guzzle орієнтована на найпоширеніші випадки використання та чудовий досвід розробника.

Перед початком роботи переконайтесь, що ви встановили пакет Guzzle як залежність вашої програми. За замовчуванням Laravel автоматично включає таку залежність:

    composer require guzzlehttp/guzzle

<a name="making-requests"></a>

## View запитів

Для надсилання запитів ви можете використовувати`get`,`post`,`put`,`patch`, і`delete`методи. Спочатку розглянемо, як скласти базовий`GET`запит:

    use Illuminate\Support\Facades\Http;

    $response = Http::get('http://example.com');

`get`метод повертає екземпляр`Illuminate\Http\Client\Response`, який надає різноманітні методи, які можуть бути використані для перевірки реакції:

    $response->body() : string;
    $response->json() : array|mixed;
    $response->status() : int;
    $response->ok() : bool;
    $response->successful() : bool;
    $response->failed() : bool;
    $response->serverError() : bool;
    $response->clientError() : bool;
    $response->header($header) : string;
    $response->headers() : array;

`Illuminate\Http\Client\Response`об'єкт також реалізує PHP`ArrayAccess`інтерфейс, що дозволяє отримати доступ до даних відповідей JSON безпосередньо у відповіді:

    return Http::get('http://example.com/users/1')['name'];

<a name="request-data"></a>

### Запит даних

Звичайно, це часто при використанні`POST`,`PUT`, і`PATCH`щоб надіслати додаткові дані разом із вашим запитом. Отже, ці методи приймають масив даних як другий аргумент. За замовчуванням дані надсилатимуться за допомогою`application/json`тип вмісту:

    $response = Http::post('http://example.com/users', [
        'name' => 'Steve',
        'role' => 'Network Administrator',
    ]);

<a name="get-request-query-parameters"></a>

#### ОТРИМАТИ параметри запиту запиту

При виготовленні`GET`запити, ви можете або додати рядок запиту до URL-адреси, або передати масив пар ключ / значення як другий аргумент до`get`метод:

    $response = Http::get('http://example.com/users', [
        'name' => 'Taylor',
        'page' => 1,
    ]);

<a name="sending-form-url-encoded-requests"></a>

#### Надсилання зашифрованих запитів URL-адреси форми

Якщо ви хочете надіслати дані за допомогою`application/x-www-form-urlencoded`типу вмісту, вам слід зателефонувати до`asForm`метод перед Viewм запиту:

    $response = Http::asForm()->post('http://example.com/users', [
        'name' => 'Sara',
        'role' => 'Privacy Consultant',
    ]);

<a name="sending-a-raw-request-body"></a>

#### Надсилання основного запиту

Ви можете використовувати`withBody`метод, якщо ви хочете надати необроблений текст запиту під час надсилання запиту:

    $response = Http::withBody(
        base64_encode($photo), 'image/jpeg'
    )->post('http://example.com/photo');

<a name="multi-part-requests"></a>

#### Багатокомпонентні запити

Якщо ви хочете надіслати файли як багатокомпонентні запити, вам слід зателефонувати до`attach`метод перед Viewм запиту. Цей метод приймає ім'я файлу та його вміст. За бажанням ви можете надати третій аргумент, який буде вважатись ім’ям файлу:

    $response = Http::attach(
        'attachment', file_get_contents('photo.jpg'), 'photo.jpg'
    )->post('http://example.com/attachments');

Замість передачі необробленого вмісту файлу ви також можете передати потоковий ресурс:

    $photo = fopen('photo.jpg', 'r');

    $response = Http::attach(
        'attachment', $photo, 'photo.jpg'
    )->post('http://example.com/attachments');

<a name="headers"></a>

### Заголовки

Заголовки можна додавати до запитів за допомогою`withHeaders`метод. Це`withHeaders`метод приймає масив пар ключ / значення:

    $response = Http::withHeaders([
        'X-First' => 'foo',
        'X-Second' => 'bar'
    ])->post('http://example.com/users', [
        'name' => 'Taylor',
    ]);

<a name="authentication"></a>

### Аутентифікація

Ви можете вказати основні та дайджест-ідентифікаційні дані, використовуючи`withBasicAuth`і`withDigestAuth`методів, відповідно:

    // Basic authentication...
    $response = Http::withBasicAuth('taylor@laravel.com', 'secret')->post(...);

    // Digest authentication...
    $response = Http::withDigestAuth('taylor@laravel.com', 'secret')->post(...);

<a name="bearer-tokens"></a>

#### Жетони на пред'явника

Якщо ви хочете швидко додати`Authorization`Заголовок токена на пред'явника до запиту, ви можете використовувати`withToken`метод:

    $response = Http::withToken('token')->post(...);

<a name="timeout"></a>

### Timeout

`timeout`метод може бути використаний для вказівки максимальної кількості секунд очікування відповіді:

    $response = Http::timeout(3)->get(...);

Якщо вказаний тайм-аут перевищено, екземпляр`Illuminate\Http\Client\ConnectionException`буде кинуто.

<a name="retries"></a>

### Повторні спроби

Якщо ви хочете, щоб клієнт HTTP автоматично повторив запит, якщо сталася помилка клієнта або сервера, ви можете використовувати`retry`метод.`retry`метод приймає два аргументи: кількість спроб запиту та кількість мілісекунд, протягом яких Laravel повинен чекати між спробами:

    $response = Http::retry(3, 100)->post(...);

Якщо всі запити не вдаються, екземпляр`Illuminate\Http\Client\RequestException`буде кинуто.

<a name="error-handling"></a>

### Обробка помилок

На відміну від поведінки Guzzle за замовчуванням, клієнтська обгортка Laravel не створює винятків щодо помилок клієнта або сервера (`400`і`500`відповіді рівня від серверів). Ви можете визначити, чи була повернута одна з цих помилок за допомогою`successful`,`clientError`, або`serverError`методи:

    // Determine if the status code was >= 200 and < 300...
    $response->successful();

    // Determine if the status code was >= 400...
    $response->failed();

    // Determine if the response has a 400 level status code...
    $response->clientError();

    // Determine if the response has a 500 level status code...
    $response->serverError();

<a name="throwing-exceptions"></a>

#### Кидання винятків

Якщо у вас є екземпляр відповіді і ви хочете викинути екземпляр`Illuminate\Http\Client\RequestException`якщо відповідь - помилка клієнта або сервера, ви можете використовувати`throw`метод:

    $response = Http::post(...);

    // Throw an exception if a client or server error occurred...
    $response->throw();

    return $response['user']['id'];

`Illuminate\Http\Client\RequestException`інстанція має паблік`$response`властивість, що дозволить перевірити повернуту відповідь.

`throw`метод повертає екземпляр відповіді, якщо помилки не сталося, дозволяючи зв'язати інші операції з`throw`метод:

    return Http::post(...)->throw()->json();

Якщо ви хочете виконати деяку додаткову логіку до того, як буде вилучено виняток, ви можете передати Закриття в`throw`метод. Виняток буде автоматично видано після виклику Закриття, тому Вам не потрібно повторно викидати виняток із Закриття:

    return Http::post(...)->throw(function ($response, $e) {
        //
    })->json();

<a name="guzzle-options"></a>

### Options Guzzle

Ви можете вказати додаткові[Параметри запиту Guzzle](http://docs.guzzlephp.org/en/stable/request-options.html)за допомогою`withOptions`метод.`withOptions`метод приймає масив пар ключ / значення:

    $response = Http::withOptions([
        'debug' => true,
    ])->get('http://example.com/users');

<a name="testing"></a>

## Тестування

Багато служб Laravel забезпечують функціональність, яка допоможе вам легко та виразно писати тести, і обгортка HTTP від ​​Laravel не є винятком.`Http`фасадні`fake`Метод дозволяє вказувати клієнтові HTTP повертати тактові / фіктивні відповіді під час надсилання запитів.

<a name="faking-responses"></a>

### Фейкові відповіді

Наприклад, щоб доручити клієнту HTTP повертати порожнє,`200`відповіді коду стану на кожен запит, ви можете зателефонувати до`fake`метод без аргументів:

    use Illuminate\Support\Facades\Http;

    Http::fake();

    $response = Http::post(...);

> {note} При підробці запитів клієнтське програмне забезпечення HTTP не виконується. Ви повинні визначити очікування щодо фальшивих відповідей, ніби це проміжне програмне забезпечення працювало правильно.

<a name="faking-specific-urls"></a>

#### Підробка конкретних URL-адрес

Крім того, ви можете передати масив до`fake`метод. Ключі масиву повинні представляти шаблони URL-адрес, які ви хочете підробити, та відповідні відповіді.`*`символ може використовуватися як символ підстановки. Будь-які запити на URL-адреси, які не були сфальсифікованими, насправді будуть виконані. Ви можете використовувати`response`метод побудови заглушених / підроблених відповідей для цих кінцевих точок:

    Http::fake([
        // Stub a JSON response for GitHub endpoints...
        'github.com/*' => Http::response(['foo' => 'bar'], 200, ['Headers']),

        // Stub a string response for Google endpoints...
        'google.com/*' => Http::response('Hello World', 200, ['Headers']),
    ]);

Якщо ви хочете вказати резервний шаблон URL-адреси, який буде заглушати всі нерівні URL-адреси, ви можете використовувати один`*`характер:

    Http::fake([
        // Stub a JSON response for GitHub endpoints...
        'github.com/*' => Http::response(['foo' => 'bar'], 200, ['Headers']),

        // Stub a string response for all other endpoints...
        '*' => Http::response('Hello World', 200, ['Headers']),
    ]);

<a name="faking-response-sequences"></a>

#### Підроблені послідовності відповідей

Іноді може знадобитися вказати, що одна URL-адреса повинна повертати ряд фальшивих відповідей у ​​певному порядку. Ви можете досягти цього за допомогою`Http::sequence`метод побудови відповідей:

    Http::fake([
        // Stub a series of responses for GitHub endpoints...
        'github.com/*' => Http::sequence()
                                ->push('Hello World', 200)
                                ->push(['foo' => 'bar'], 200)
                                ->pushStatus(404),
    ]);

Коли всі відповіді в послідовності відповідей будуть використані, будь-які подальші запити змусять послідовність відповідей викликати виняток. Якщо ви хочете вказати відповідь за замовчуванням, яку слід повертати, коли послідовність порожня, ви можете використовувати`whenEmpty`метод:

    Http::fake([
        // Stub a series of responses for GitHub endpoints...
        'github.com/*' => Http::sequence()
                                ->push('Hello World', 200)
                                ->push(['foo' => 'bar'], 200)
                                ->whenEmpty(Http::response()),
    ]);

Якщо ви хочете сфальсифікувати послідовність відповідей, але вам не потрібно вказувати конкретний шаблон URL-адреси, який слід сфальсифікувати, ви можете використовувати`Http::fakeSequence`метод:

    Http::fakeSequence()
            ->push('Hello World', 200)
            ->whenEmpty(Http::response());

<a name="fake-callback"></a>

#### Підроблений зворотний дзвінок

Якщо вам потрібна більш складна логіка для визначення того, які відповіді повертати для певних кінцевих точок, ви можете передати зворотний дзвінок до`fake`метод. Цей зворотний дзвінок отримає екземпляр`Illuminate\Http\Client\Request`і повинен повернути екземпляр відповіді:

    Http::fake(function ($request) {
        return Http::response('Hello World', 200);
    });

<a name="inspecting-requests"></a>

### Перевірка запитів

Підробляючи відповіді, ви можете іноді побажати перевірити запити, які отримує клієнт, щоб переконатися, що ваша програма надсилає правильні дані або заголовки. Ви можете досягти цього, зателефонувавши до`Http::assertSent`метод після виклику`Http::fake`.

The `assertSent`метод приймає зворотний виклик, який отримає`Illuminate\Http\Client\Request`екземпляр і повинен повернути логічне значення, яке вказує, чи відповідає запит вашим очікуванням. Щоб тест міг пройти, повинен бути виданий принаймні один запит, що відповідає заданим очікуванням:

    Http::fake();

    Http::withHeaders([
        'X-First' => 'foo',
    ])->post('http://example.com/users', [
        'name' => 'Taylor',
        'role' => 'Developer',
    ]);

    Http::assertSent(function ($request) {
        return $request->hasHeader('X-First', 'foo') &&
               $request->url() == 'http://example.com/users' &&
               $request['name'] == 'Taylor' &&
               $request['role'] == 'Developer';
    });

Якщо потрібно, ви можете стверджувати, що конкретний запит не було надіслано за допомогою`assertNotSent`метод:

    Http::fake();

    Http::post('http://example.com/users', [
        'name' => 'Taylor',
        'role' => 'Developer',
    ]);

    Http::assertNotSent(function (Request $request) {
        return $request->url() === 'http://example.com/posts';
    });

Або, якщо ви хочете стверджувати, що жодних запитів не надсилали, ви можете використовувати`assertNothingSent`метод:

    Http::fake();

    Http::assertNothingSent();
