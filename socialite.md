# Laravel Socialite

-   [Вступ](#introduction)
-   [Оновлення Socialite](#upgrading-socialite)
-   [Встановлення](#installation)
-   [Конфігурація](#configuration)
-   [Маршрутизація](#routing)
-   [Необов’язкові параметри](#optional-parameters)
-   [Області доступу](#access-scopes)
-   [Аутентифікація без Stateless](#stateless-authentication)
-   [Отримання даних користувача](#retrieving-user-details)

<a name="introduction"></a>

## Introduction

In addition to typical, form based authentication, Laravel also provides a simple, convenient way to authenticate with OAuth providers using [Laravel Socialite](https://github.com/laravel/socialite). На даний момент Socialite підтримує автентифікацію за допомогою Facebook, Twitter, LinkedIn, Google, GitHub, GitLab та Bitbucket.

> {tip} Адаптери для інших платформ перераховані спільнотою[Постачальники соціальних мереж](https://socialiteproviders.com/)веб-сайт.

<a name="upgrading-socialite"></a>

## Оновлення Socialite

Під час оновлення до нової основної версії Socialite важливо уважно переглянути[посібник з оновлення](https://github.com/laravel/socialite/blob/master/UPGRADE.md).

<a name="installation"></a>

## Встановлення

Щоб розпочати роботу з Socialite, використовуйте Composer, щоб додати пакет до залежностей вашого проекту:

    composer require laravel/socialite

<a name="configuration"></a>

## Конфігурація

Перш ніж використовувати Socialite, вам також потрібно буде додати облікові дані служб OAuth, якими користується ваша програма. Ці облікові дані слід розмістити у вашому`config/services.php`файл конфігурації, і повинен використовувати ключ`facebook`,`twitter`,`linkedin`,`google`,`github`,`gitlab`або`bitbucket`залежно від постачальників, яких вимагає ваша програма. Наприклад:

    'github' => [
        'client_id' => env('GITHUB_CLIENT_ID'),
        'client_secret' => env('GITHUB_CLIENT_SECRET'),
        'redirect' => 'http://your-callback-url',
    ],

> {tip} Якщо`redirect`Параметр містить відносний шлях, він буде автоматично перетворений на повністю кваліфіковану URL-адресу.

<a name="routing"></a>

## Маршрутизація

Далі ви готові до автентифікації користувачів! Вам знадобляться два маршрути: один для перенаправлення користувача до постачальника OAuth, а інший для отримання зворотного дзвінка від постачальника після автентифікації. Ми отримаємо доступ до Socialite за допомогою`Socialite`фасад:

    <?php

    namespace App\Http\Controllers\Auth;

    use App\Http\Controllers\Controller;
    use Laravel\Socialite\Facades\Socialite;

    class LoginController extends Controller
    {
        /**
         * Redirect the user to the GitHub authentication page.
         *
         * @return \Illuminate\Http\Response
         */
        public function redirectToProvider()
        {
            return Socialite::driver('github')->redirect();
        }

        /**
         * Obtain the user information from GitHub.
         *
         * @return \Illuminate\Http\Response
         */
        public function handleProviderCallback()
        {
            $user = Socialite::driver('github')->user();

            // $user->token;
        }
    }

`redirect`метод дбає про відправлення користувача до постачальника OAuth, тоді як`user`метод прочитає вхідний запит і отримає інформацію про користувача від постачальника.

Вам потрібно буде визначити маршрути до методів вашого контролера:

    use App\Http\Controllers\Auth\LoginController;

    Route::get('login/github', [LoginController::class, 'redirectToProvider']);
    Route::get('login/github/callback', [LoginController::class, 'handleProviderCallback']);

<a name="optional-parameters"></a>

## Необов’язкові параметри

Ряд постачальників OAuth підтримують необов’язкові параметри у запиті на перенаправлення. Щоб включити будь-які необов’язкові параметри у запит, зателефонуйте на`with`метод з асоціативним масивом:

    return Socialite::driver('google')
        ->with(['hd' => 'example.com'])
        ->redirect();

> {note} При використанні`with`, будьте обережні, щоб не передавати зарезервовані ключові слова, такі як`state`або`response_type`.

<a name="access-scopes"></a>

## Області доступу

Перш ніж перенаправляти користувача, ви можете також додати додаткові "області" на запит, використовуючи`scopes`метод. Цей метод об’єднає всі існуючі області з тими, які ви надаєте:

    return Socialite::driver('github')
        ->scopes(['read:user', 'public_repo'])
        ->redirect();

Ви можете переписати всі існуючі області, використовуючи`setScopes`метод:

    return Socialite::driver('github')
        ->setScopes(['read:user', 'public_repo'])
        ->redirect();

<a name="stateless-authentication"></a>

## Аутентифікація без Stateless

`stateless`метод може бути використаний для вимкнення перевірки стану сеансу. Це корисно при додаванні соціальної автентифікації до API:

    return Socialite::driver('google')->stateless()->user();

> {note} Аутентифікація без Stateless недоступна для драйвера Twitter, який використовує OAuth 1.0 для автентифікації.

<a name="retrieving-user-details"></a>

## Отримання даних користувача

Отримавши примірник користувача, ви можете отримати ще кілька деталей про користувача:

    $user = Socialite::driver('github')->user();

    // OAuth Two Providers
    $token = $user->token;
    $refreshToken = $user->refreshToken; // not always provided
    $expiresIn = $user->expiresIn;

    // OAuth One Providers
    $token = $user->token;
    $tokenSecret = $user->tokenSecret;

    // All Providers
    $user->getId();
    $user->getNickname();
    $user->getName();
    $user->getEmail();
    $user->getAvatar();

<a name="retrieving-user-details-from-a-token-oauth2"></a>

#### Отримання даних користувача з маркера (OAuth2)

Якщо у вас вже є дійсний маркер доступу для користувача, ви можете отримати його дані за допомогою`userFromToken`метод:

    $user = Socialite::driver('github')->userFromToken($token);

<a name="retrieving-user-details-from-a-token-and-secret-oauth1"></a>

#### Отримання даних користувача з маркера та секрету (OAuth1)

Якщо ви вже маєте дійсну пару маркера / секрету для користувача, ви можете отримати його дані за допомогою`userFromTokenAndSecret`метод:

    $user = Socialite::driver('twitter')->userFromTokenAndSecret($token, $secret);
