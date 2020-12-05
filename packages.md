# Розробка пакетів

-   [Вступ](#introduction)
    -   [Примітка про фасади](#a-note-on-facades)
-   [Відкриття пакету](#package-discovery)
-   [Service Providers](#service-providers)
-   [Ресурси](#resources)
    -   [Конфігурація](#configuration)
    -   [Міграції](#migrations)
    -   [Маршрути](#routes)
    -   [Переклади](#translations)
    -   [Перегляди](#views)
    -   [Перегляд компонентів](#view-components)
-   [Команди](#commands)
-   [Public Assets](#public-assets)
-   [Публікація файлових груп](#publishing-file-groups)

<a name="introduction"></a>

## Вступ

Пакети - це основний спосіб додати функціональність до Laravel. Пакети можуть бути чим завгодно, від чудового способу роботи з такими датами, як[Вуглець](https://github.com/briannesbitt/Carbon), або цілий фреймворк тестування BDD, наприклад[Бехат](https://github.com/Behat/Behat).

Існують різні типи пакетів. Деякі пакети є автономними, тобто вони працюють з будь-яким фреймворком PHP. Carbon та Behat - це приклади окремих пакетів. Будь-який із цих пакетів можна використовувати разом із Laravel, запитуючи їх у своєму`composer.json`файл.

З іншого боку, інші упаковки спеціально призначені для використання з Laravel. Ці пакети можуть мати маршрути, контролери, View та конфігурацію, спеціально призначені для вдосконалення програми Laravel. Цей посібник насамперед охоплює розробку тих пакетів, які є специфічними для Laravel.

<a name="a-note-on-facades"></a>

### Примітка про фасади

При написанні заявки Laravel, як правило, не має значення, використовуєте ви контракти чи фасади, оскільки обидва вони забезпечують фактично рівні рівні перевірки. Однак під час написання пакетів ваш пакет, як правило, не матиме доступу до всіх помічників тестування Laravel. Якщо ви хочете мати можливість писати свої тести пакетів так, ніби вони існують у типовому додатку Laravel, ви можете використовувати[Оркестровий випробувальний стенд](https://github.com/orchestral/testbench)пакет.

<a name="package-discovery"></a>

## Відкриття пакету

У програмі Laravel`config/app.php`файл конфігурації, файл`providers`Параметр визначає список Vendor послуг, які повинен завантажити Laravel. Коли хтось встановлює ваш пакет, ви зазвичай хочете, щоб ваш постачальник послуг був включений до цього списку. Замість того, щоб вимагати від користувачів вручну додавати свого постачальника послуг до списку, ви можете визначити цього постачальника в`extra`розділ вашого пакета`composer.json`файл. Окрім Vendor послуг, ви можете також перерахувати будь-яких[фасади](/docs/{{version}}/facades)Ви хочете бути зареєстрованим:

    "extra": {
        "laravel": {
            "providers": [
                "Barryvdh\\Debugbar\\ServiceProvider"
            ],
            "aliases": {
                "Debugbar": "Barryvdh\\Debugbar\\Facade"
            }
        }
    },

Після того, як ваш пакет буде налаштований на виявлення, Laravel автоматично реєструватиме своїх Vendor послуг та фасади після його встановлення, створюючи зручний досвід встановлення для користувачів вашого пакету.

<a name="opting-out-of-package-discovery"></a>

### Відмова від виявлення пакунків

Якщо ви є споживачем пакета і хочете відключити виявлення пакета для пакету, ви можете вказати ім'я пакета в`extra`розділ вашої програми`composer.json`файл:

    "extra": {
        "laravel": {
            "dont-discover": [
                "barryvdh/laravel-debugbar"
            ]
        }
    },

Ви можете вимкнути виявлення пакунків для всіх пакетів, що використовують`*`символу всередині вашої програми`dont-discover`директива:

    "extra": {
        "laravel": {
            "dont-discover": [
                "*"
            ]
        }
    },

<a name="service-providers"></a>

## Service Providers

[Service Providers](/docs/{{version}}/providers)є точками зв’язку між вашим пакетом та Laravel. Постачальник послуг відповідає за прив'язку речей до Laravel[службовий контейнер](/docs/{{version}}/container)та інформування Laravel куди завантажувати ресурси пакету, такі як Views, конфігурація та файли локалізації.

Постачальник послуг розширює`Illuminate\Support\ServiceProvider`клас і містить два методи:`register`і`boot`. Основа`ServiceProvider`клас знаходиться в`illuminate/support`Пакет Composer, який слід додати до залежностей власного пакета. Щоб дізнатись більше про структуру та призначення Vendor послуг, ознайомтесь[їх документація](/docs/{{version}}/providers).

<a name="resources"></a>

## Ресурси

<a name="configuration"></a>

### Конфігурація

Як правило, вам потрібно буде опублікувати файл конфігурації пакета у власному додатку`config`каталог. Це дозволить користувачам вашого пакету легко замінити ваші параметри конфігурації за замовчуванням. Щоб дозволити публікацію файлів конфігурації, зателефонуйте на номер`publishes`метод з`boot`спосіб вашого постачальника послуг:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/path/to/config/courier.php' => config_path('courier.php'),
        ]);
    }

Тепер, коли користувачі вашого пакету виконують Laravel`vendor:publish`команда, ваш файл буде скопійовано до вказаного місця публікації. Після публікації вашої конфігурації її значення можна отримати як будь-який інший конфігураційний файл:

    $value = config('courier.option');

> {note} Не слід визначати Закриття у файлах конфігурації. Вони не можуть бути правильно серіалізовані, коли користувачі виконують`config:cache`artisan командування.

<a name="default-package-configuration"></a>

#### Конфігурація пакета за замовчуванням

Ви також можете об'єднати власний файл конфігурації пакета з опублікованою копією програми. Це дозволить вашим користувачам визначати лише ті параметри, які вони дійсно хочуть замінити в опублікованій копії конфігурації. Щоб об'єднати конфігурації, використовуйте`mergeConfigFrom`методу, який надається постачальником послуг`register`метод:

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->mergeConfigFrom(
            __DIR__.'/path/to/config/courier.php', 'courier'
        );
    }

> {note} Цей метод об’єднує лише перший рівень масиву конфігурації. Якщо ваші користувачі частково визначають багатовимірний масив конфігурації, відсутні параметри не будуть об’єднані.

<a name="routes"></a>

### Маршрути

Якщо ваш пакет містить маршрути, ви можете завантажити їх за допомогою`loadRoutesFrom`метод. Цей метод автоматично визначає, чи маршрути програми кешовані, і не завантажує ваш файл маршрутів, якщо маршрути вже кешовані:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadRoutesFrom(__DIR__.'/routes.php');
    }

<a name="migrations"></a>

### Міграції

Якщо ваш пакет містить[міграції баз даних](/docs/{{version}}/migrations), ви можете використовувати`loadMigrationsFrom`спосіб повідомити Laravel, як їх завантажувати.`loadMigrationsFrom`метод приймає шлях до міграцій вашого пакету як єдиний аргумент:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadMigrationsFrom(__DIR__.'/path/to/migrations');
    }

Після реєстрації міграцій вашого пакету вони автоматично запускатимуться, коли`php artisan migrate`команда виконується. Вам не потрібно експортувати їх до основної програми`database/migrations`каталог.

<a name="translations"></a>

### Переклади

Якщо ваш пакет містить[файли перекладу](/docs/{{version}}/localization), ви можете використовувати`loadTranslationsFrom`спосіб повідомити Laravel, як їх завантажувати. Наприклад, якщо ваш пакет названий`courier`, вам слід додати наступне до свого постачальника послуг`boot`метод:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
    }

На переклади пакетів посилаються за допомогою`package::file.line`синтаксична конвенція. Отже, ви можете завантажити`courier`пакет`welcome`рядок від`messages`файл приблизно так:

    echo trans('courier::messages.welcome');

<a name="publishing-translations"></a>

#### Публікація перекладів

Якщо ви хочете опублікувати переклади вашого пакету до програми`resources/lang/vendor`в каталозі, Ви можете використовувати каталог постачальника послуг`publishes`метод.`publishes`метод приймає масив шляхів до пакунків та бажані місця їх публікації. Наприклад, опублікувати файли перекладу для`courier`Ви можете зробити наступне:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');

        $this->publishes([
            __DIR__.'/path/to/translations' => resource_path('lang/vendor/courier'),
        ]);
    }

Тепер, коли користувачі вашого пакету виконують Laravel`vendor:publish`Команда Artisan, переклади вашого пакету будуть опубліковані до вказаного місця публікації.

<a name="views"></a>

### Перегляди

Зареєструвати пакунки[погляди](/docs/{{version}}/views)з Laravel, вам потрібно сказати Laravel, де розташовані види. Ви можете зробити це за допомогою постачальника послуг`loadViewsFrom`метод.`loadViewsFrom`метод приймає два аргументи: шлях до шаблонів View та ім'я вашого пакета. Наприклад, якщо ім’я вашого пакета -`courier`, Ви б додали наступне до свого постачальника послуг`boot`метод:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
    }

На View пакетів посилаються за допомогою`package::view`синтаксична конвенція. Отже, як тільки ваш шлях перегляду зареєстрований у постачальника послуг, ви можете завантажити`admin`вид з боку`courier`пакет приблизно так:

    Route::get('admin', function () {
        return view('courier::admin');
    });

<a name="overriding-package-views"></a>

#### Заміна переглядів пакетів

Коли ви використовуєте`loadViewsFrom`методу, Laravel фактично реєструє два місця для ваших поглядів: додаток`resources/views/vendor`і вказаний вами каталог. Отже, використовуючи`courier`Наприклад, Laravel спочатку перевірить, чи розробник надав власну версію View в`resources/views/vendor/courier`. Потім, якщо View не було налаштовано, Laravel здійснить пошук у каталозі перегляду пакетів, який ви вказали у своєму дзвінку`loadViewsFrom`. Це полегшує користувачам пакунків можливість налаштовувати / перевизначати View вашого пакета.

<a name="publishing-views"></a>

#### Публікація поглядів

Якщо ви хочете зробити свої погляди доступними для публікації в програмі`resources/views/vendor`в каталозі, Ви можете використовувати каталог постачальника послуг`publishes`метод.`publishes`метод приймає масив шляхів перегляду пакетів та їх бажані місця публікації:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');

        $this->publishes([
            __DIR__.'/path/to/views' => resource_path('views/vendor/courier'),
        ]);
    }

Тепер, коли користувачі вашого пакету виконують Laravel`vendor:publish`Команда Artisan, View вашого пакету будуть скопійовані у вказане місце публікації.

<a name="view-components"></a>

### Перегляд компонентів

Якщо ваш пакет містить[переглянути компоненти](/docs/{{version}}/blade#components), ви можете використовувати`loadViewComponentsAs`спосіб повідомити Laravel, як їх завантажувати.`loadViewComponentsAs`метод приймає два аргументи: префікс тегу для компонентів представлення та масив класу компонентів View. Наприклад, якщо префікс вашого пакета -`courier`і у вас є`Alert`і`Button`перегляду компонентів, ви б додали наступне до постачальника послуг`boot`метод:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewComponentsAs('courier', [
            Alert::class,
            Button::class,
        ]);
    }

Після того, як компоненти вашого представлення зареєстровані у постачальника послуг, ви можете посилатись на них у своєму поданні так:

    <x-courier-alert />

    <x-courier-button />

<a name="anonymous-components"></a>

#### Анонімні компоненти

Якщо ваш пакет містить анонімні компоненти, вони повинні бути розміщені в`components`каталог каталогу "View" вашого пакету (як вказано`loadViewsFrom`). Потім ви можете відтворити їх, додавши ім'я компонента до простору імен View пакета:

    <x-courier::alert />

<a name="commands"></a>

## Команди

Щоб зареєструвати команди Artisan вашого пакету в Laravel, ви можете використовувати`commands`метод. Цей метод передбачає масив імен класів команд. Після реєстрації команд ви можете виконувати їх за допомогою[Майстер CLI](/docs/{{version}}/artisan):

    /**
     * Bootstrap the application services.
     *
     * @return void
     */
    public function boot()
    {
        if ($this->app->runningInConsole()) {
            $this->commands([
                FooCommand::class,
                BarCommand::class,
            ]);
        }
    }

<a name="public-assets"></a>

## Public Assets

Ваш пакет може мати такі Assets, як JavaScript, CSS та зображення. Опублікувати ці Assets в додатку`public`довідник, скористайтеся послугами постачальника послуг`publishes`метод. У цьому прикладі ми також додамо a`public`тег групи активів, який може використовуватися для публікації груп пов’язаних активів:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/path/to/assets' => public_path('vendor/courier'),
        ], 'public');
    }

Тепер, коли користувачі вашого пакету виконують`vendor:publish`Команда активів буде скопійована до вказаного місця публікації. Оскільки вам зазвичай потрібно буде перезаписувати Assets кожного разу, коли пакет оновлюється, ви можете використовувати`--force`прапор:

    php artisan vendor:publish --tag=public --force

<a name="publishing-file-groups"></a>

## Публікація файлових груп

Можливо, ви захочете опублікувати групи ресурсів та ресурсів пакета окремо. Наприклад, ви можете дозволити своїм користувачам публікувати файли конфігурації вашого пакета, не змушуючи публікувати Assets пакета. Ви можете зробити це, позначивши їх при виклику`publishes`метод від постачальника послуг пакета. Наприклад, давайте використаємо теги, щоб визначити дві групи публікацій у`boot`метод постачальника пакетних послуг:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/../config/package.php' => config_path('package.php')
        ], 'config');

        $this->publishes([
            __DIR__.'/../database/migrations/' => database_path('migrations')
        ], 'migrations');
    }

Тепер ваші користувачі можуть публікувати ці групи окремо, посилаючись на їх теги під час запуску`vendor:publish`команда:

    php artisan vendor:publish --tag=config
