# Захист CSRF

-   [Вступ](#csrf-introduction)
-   [Без урахування URI](#csrf-excluding-uris)
-   [X-CSRF-Token](#csrf-x-csrf-token)
-   [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>

## Вступ

Laravel дозволяє легко захистити вашу програму від[міжсайтовий підробку запиту](https://en.wikipedia.org/wiki/Cross-site_request_forgery)(CSRF) атаки. Підроблені міжсайтові запити - це різновид зловмисного використання, за допомогою якого несанкціоновані команди виконуються від імені автентифікованого користувача.

Laravel автоматично генерує "маркер" CSRF для кожного активного сеансу користувача, керованого додатком. Цей маркер використовується для перевірки того, що автентифікований користувач є тим, хто насправді робить запити до програми.

Кожного разу, коли ви визначаєте форму HTML у своїй програмі, ви повинні включити приховане поле маркера CSRF у форму, щоб проміжне програмне забезпечення захисту CSRF могло перевірити запит. Ви можете використовувати`@csrf`Директива Blade для створення поля маркера:

    <form method="POST" action="/profile">
        @csrf
        ...
    </form>

`VerifyCsrfToken`[проміжне програмне забезпечення](/docs/{{version}}/middleware), що входить до`web`група проміжного програмного забезпечення, автоматично перевірить, що маркер у введеному запиті відповідає маркеру, що зберігається в сеансі.

<a name="csrf-tokens-javascript"></a>

#### CSRF токени та JavaScript

При створенні програм, керованих JavaScript, зручно мати, щоб ваша HTTP-бібліотека JavaScript автоматично приєднувала маркер CSRF до кожного вихідного запиту. За замовчуванням бібліотека Axios HTTP, надана в`resources/js/bootstrap.js`файл автоматично надсилає файл`X-XSRF-TOKEN`заголовок, використовуючи значення зашифрованого`XSRF-TOKEN` cookie. If you are not using this library, you will need to manually configure this behavior for your application.

<a name="csrf-excluding-uris"></a>

## Виключення URI із захисту CSRF

Іноді вам може знадобитися виключити набір URI із захисту CSRF. Наприклад, якщо ви використовуєте[Смуга](https://stripe.com)для обробки платежів і використання їх системи webhook вам потрібно буде виключити ваш маршрут обробника веб-хука із захисту CSRF, оскільки Stripe не знатиме, який маркер CSRF надсилати на ваші маршрути.

Як правило, ви повинні розміщувати такі маршрути за межами`web`проміжне програмне забезпечення, яке`RouteServiceProvider`стосується всіх маршрутів у`routes/web.php`файл. Однак ви можете також виключити маршрути, додавши їх URI до`$except` property of the `VerifyCsrfToken`проміжне програмне забезпечення:

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as Middleware;

    class VerifyCsrfToken extends Middleware
    {
        /**
         * The URIs that should be excluded from CSRF verification.
         *
         * @var array
         */
        protected $except = [
            'stripe/*',
            'http://example.com/foo/bar',
            'http://example.com/foo/*',
        ];
    }

> {tip} Проміжне програмне забезпечення CSRF автоматично вимикається, коли[запущені тести](/docs/{{version}}/testing).

<a name="csrf-x-csrf-token"></a>

## X-CSRF-TOKEN

На додаток до перевірки маркера CSRF як параметра POST, файл`VerifyCsrfToken`проміжне програмне забезпечення також перевірить наявність`X-CSRF-TOKEN`заголовок запиту. Наприклад, ви можете зберегти маркер у HTML`meta`тег:

    <meta name="csrf-token" content="{{ csrf_token() }}">

Потім, як тільки ви створили`meta`тегу, ви можете доручити бібліотеці, як jQuery, автоматично додавати маркер до всіх заголовків запиту. Це забезпечує простий, зручний захист CSRF для ваших програм на базі AJAX:

    $.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
    });

<a name="csrf-x-xsrf-token"></a>

## X-XSRF-ЛОКЕН

Laravel stores the current CSRF token in an encrypted `XSRF-TOKEN`cookie, який включений до кожної відповіді, сформованої фреймворком. Ви можете використовувати значення cookie, щоб встановити`X-XSRF-TOKEN`заголовок запиту.

Цей файл cookie в основному надсилається для зручності, оскільки деякі фреймворки та бібліотеки JavaScript, такі як Angular та Axios, автоматично розміщують своє значення в`X-XSRF-TOKEN`заголовок на запити того самого походження.

> {tip} За замовчуванням`resources/js/bootstrap.js`Файл містить бібліотеку Axios HTTP, яка автоматично надішле це для вас.
