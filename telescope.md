# Телескоп Laravel

-   [Вступ](#introduction)
-   [Встановлення](#installation)
    -   [Конфігурація](#configuration)
    -   [Обрізання даних](#data-pruning)
    -   [Налаштування міграції](#migration-customization)
    -   [Авторизація інформаційної панелі](#dashboard-authorization)
-   [Модернізація телескопа](#upgrading-telescope)
-   [Фільтрування](#filtering)
    -   [Записи](#filtering-entries)
    -   [Партії](#filtering-batches)
-   [Позначення](#tagging)
-   [Доступні спостерігачі](#available-watchers)
    -   [Пакетний спостерігач](#batch-watcher)
    -   [Кеш-спостерігач](#cache-watcher)
    -   [Командний спостерігач](#command-watcher)
    -   [Смітник спостерігач](#dump-watcher)
    -   [Спостерігач за подіями](#event-watcher)
    -   [Надзвичайний спостерігач](#exception-watcher)
    -   [Ворота спостерігач](#gate-watcher)
    -   [Job Watcher](#job-watcher)
    -   [Запис журналу](#log-watcher)
    -   [Поштовий спостерігач](#mail-watcher)
    -   [Модель спостерігача](#model-watcher)
    -   [Спостерігач сповіщень](#notification-watcher)
    -   [Запит спостерігача](#query-watcher)
    -   [Редіс Ватчер](#redis-watcher)
    -   [Запит спостерігача](#request-watcher)
    -   [Графік спостерігача](#schedule-watcher)
    -   [Переглянути спостерігач](#view-watcher)
-   [Відображення аватарів користувачів](#displaying-user-avatars)

<a name="introduction"></a>

## Вступ

Телескоп Laravel - це елегантний помічник з налагодження для фреймворку Laravel. Телескоп забезпечує розуміння запитів, що надходять у вашу програму, винятків, записів журналу, запитів до бази даних, завдань у черзі, пошти, сповіщень, операцій кешування, запланованих завдань, дампів змінних тощо. Телескоп є чудовим супутником вашого місцевого середовища для розвитку Laravel.

<img src="https://laravel.com/img/docs/telescope-example.png">

<a name="installation"></a>

## Встановлення

Ви можете використовувати Composer для встановлення телескопа у ваш проект Laravel:

    composer require laravel/telescope

Після встановлення Telescope опублікуйте його активи за допомогою`telescope:install`Artisan командування. Після встановлення телескопа вам слід також запустити`migrate`команда:

    php artisan telescope:install

    php artisan migrate

<a name="installing-only-in-specific-environments"></a>

### Встановлення лише в певних середовищах

Якщо ви плануєте використовувати Телескоп лише для сприяння вашому місцевому розвитку, Ви можете встановити Телескоп за допомогою`--dev`прапор:

    composer require laravel/telescope --dev

Після бігу`telescope:install`, слід видалити`TelescopeServiceProvider`реєстрація постачальника послуг від вашого`app`файл конфігурації. Натомість зареєструйте постачальника послуг вручну в`register`метод вашого`AppServiceProvider`:

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        if ($this->app->environment('local')) {
            $this->app->register(\Laravel\Telescope\TelescopeServiceProvider::class);
            $this->app->register(TelescopeServiceProvider::class);
        }
    }

Ви також повинні запобігти існуванню пакета Телескоп[автоматично виявлено](/docs/{{version}}/packages#package-discovery)додавши до вашого`composer.json`файл:

    "extra": {
        "laravel": {
            "dont-discover": [
                "laravel/telescope"
            ]
        }
    },

<a name="migration-customization"></a>

### Налаштування міграції

Якщо ви не збираєтесь використовувати міграції за замовчуванням у Телескопі, вам слід зателефонувати до`Telescope::ignoreMigrations`метод у`register`метод вашого`AppServiceProvider`. Ви можете експортувати міграції за замовчуванням за допомогою`php artisan vendor:publish --tag=telescope-migrations`команди.

<a name="configuration"></a>

### Конфігурація

Після публікації активів Телескопа його основний файл конфігурації буде розміщений за адресою`config/telescope.php`. Цей конфігураційний файл дозволяє вам налаштувати параметри вашого спостерігача, і кожен параметр конфігурації включає опис його призначення, тому обов’язково ретельно вивчіть цей файл.

За бажанням ви можете вимкнути збір даних телескопа повністю, використовуючи`enabled`варіант конфігурації:

    'enabled' => env('TELESCOPE_ENABLED', true),

<a name="data-pruning"></a>

### Обрізання даних

Без обрізки`telescope_entries`таблиця може накопичувати записи дуже швидко. Щоб пом'якшити це, слід запланувати`telescope:prune`Команда ремісників бігати щодня:

    $schedule->command('telescope:prune')->daily();

За замовчуванням усі записи старші за 24 години будуть обрізані. Ви можете використовувати`hours`опція при виклику команди, щоб визначити, як довго зберігати дані телескопа. Наприклад, наступна команда видалить усі записи, створені понад 48 годин тому:

    $schedule->command('telescope:prune --hours=48')->daily();

<a name="dashboard-authorization"></a>

### Авторизація інформаційної панелі

Телескоп виставляє приладову панель на`/telescope`. За замовчуванням ви зможете отримати доступ до цієї інформаційної панелі лише в`local`навколишнє середовище. В межах вашого`app/Providers/TelescopeServiceProvider.php`файл, є файл`gate`метод. Ця авторизація контролює доступ до телескопа в**немісцевий**середовищах. Ви можете змінювати ці ворота за необхідності, щоб обмежити доступ до вашої установки телескопа:

    /**
     * Register the Telescope gate.
     *
     * This gate determines who can access Telescope in non-local environments.
     *
     * @return void
     */
    protected function gate()
    {
        Gate::define('viewTelescope', function ($user) {
            return in_array($user->email, [
                'taylor@laravel.com',
            ]);
        });
    }

> {note} Переконайтеся, що ви змінили`APP_ENV`середовище змінна до`production`у вашому виробничому середовищі. В іншому випадку установка вашого телескопа буде загальнодоступною.

<a name="upgrading-telescope"></a>

## Модернізація телескопа

Під час оновлення до нової основної версії телескопа важливо уважно переглянути його[посібник з оновлення](https://github.com/laravel/telescope/blob/master/UPGRADE.md).

Крім того, під час оновлення до будь-якої нової версії телескопа, вам слід повторно опублікувати активи телескопа:

    php artisan telescope:publish

Щоб оновити об’єкти та уникнути проблем із майбутніми оновленнями, ви можете додати файл`telescope:publish`команда до`post-update-cmd`сценарії у вашому додатку`composer.json`файл:

    {
        "scripts": {
            "post-update-cmd": [
                "@php artisan telescope:publish --ansi"
            ]
        }
    }

<a name="filtering"></a>

## Фільтрування

<a name="filtering-entries"></a>

### Записи

Ви можете відфільтрувати дані, записані телескопом, через`filter`зворотний дзвінок, зареєстрований у вашому`TelescopeServiceProvider`. За замовчуванням цей зворотний виклик записує всі дані в`local`середовище та винятки, невдалі завдання, заплановані завдання та дані з відстежуваними тегами у всіх інших середовищах:

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->hideSensitiveRequestDetails();

        Telescope::filter(function (IncomingEntry $entry) {
            if ($this->app->environment('local')) {
                return true;
            }

            return $entry->isReportableException() ||
                $entry->isFailedJob() ||
                $entry->isScheduledTask() ||
                $entry->hasMonitoredTag();
        });
    }

<a name="filtering-batches"></a>

### Партії

Поки`filter`зворотній дзвінок фільтрує дані для окремих записів, ви можете використовувати`filterBatch`метод реєстрації зворотного виклику, який фільтрує всі дані для даного запиту або консольної команди. Якщо зворотний дзвінок повернеться`true`, всі записи записані телескопом:

    use Illuminate\Support\Collection;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->hideSensitiveRequestDetails();

        Telescope::filterBatch(function (Collection $entries) {
            if ($this->app->environment('local')) {
                return true;
            }

            return $entries->contains(function ($entry) {
                return $entry->isReportableException() ||
                    $entry->isFailedJob() ||
                    $entry->isScheduledTask() ||
                    $entry->hasMonitoredTag();
                });
        });
    }

<a name="tagging"></a>

## Позначення

Телескоп дозволяє шукати записи за "тегом". Часто теги - це красномовні імена класів моделі або автентифіковані ідентифікатори користувачів, які Телескоп автоматично додає до записів. Іноді вам може знадобитися приєднати власні власні теги до записів. Для цього ви можете використовувати`Telescope::tag`метод.`tag`метод приймає зворотний виклик, який повинен повертати масив тегів. Теги, повернуті зворотним викликом, будуть об’єднані з будь-якими тегами, які телескоп автоматично приєднає до запису. Вам слід зателефонувати до`tag`метод у вашому`TelescopeServiceProvider`:

    use Laravel\Telescope\Telescope;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->hideSensitiveRequestDetails();

        Telescope::tag(function (IncomingEntry $entry) {
            if ($entry->type === 'request') {
                return ['status:'.$entry->content['response_status']];
            }

            return [];
        });
     }

<a name="available-watchers"></a>

## Доступні спостерігачі

Спостерігачі телескопів збирають дані програми, коли виконується запит або команда консолі. Ви можете налаштувати список спостерігачів, які ви хотіли б увімкнути в своєму`config/telescope.php`файл конфігурації:

    'watchers' => [
        Watchers\CacheWatcher::class => true,
        Watchers\CommandWatcher::class => true,
        ...
    ],

Деякі спостерігачі також дозволяють надавати додаткові параметри налаштування:

    'watchers' => [
        Watchers\QueryWatcher::class => [
            'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
            'slow' => 100,
        ],
        ...
    ],

<a name="batch-watcher"></a>

### Пакетний спостерігач

Переглядач пакетів записує інформацію про партії, що знаходяться в черзі, включаючи інформацію про роботу та з'єднання.

<a name="cache-watcher"></a>

### Кеш-спостерігач

Переглядач кешу записує дані, коли клавіша кешу потрапляє, пропускає, оновлює та забуває.

<a name="command-watcher"></a>

### Командний спостерігач

Командний спостерігач записує аргументи, параметри, код виходу та вихід, коли виконується команда Artisan. Якщо ви хочете виключити певні команди із запису спостерігачем, ви можете вказати команду в`ignore`варіант у вашому`config/telescope.php`файл:

    'watchers' => [
        Watchers\CommandWatcher::class => [
            'enabled' => env('TELESCOPE_COMMAND_WATCHER', true),
            'ignore' => ['key:generate'],
        ],
        ...
    ],

<a name="dump-watcher"></a>

### Смітник спостерігач

Спостерігач дампів записує та відображає ваші дамп змінних у телескопі. При використанні Laravel змінні можуть бути скинуті за допомогою глобальної`dump`функція. Вкладка спостерігача дампа повинна бути відкрита у браузері, щоб відбулося записування, інакше спостерігачі ігнорують дампи.

<a name="event-watcher"></a>

### Спостерігач за подіями

Спостерігач подій реєструє корисне навантаження, слухачі та дані трансляції для будь-яких подій, надісланих вашим додатком. Внутрішні події фреймворку Laravel ігноруються спостерігачем подій.

<a name="exception-watcher"></a>

### Надзвичайний спостерігач

Диспетчер винятків реєструє дані та трасування стека для будь-яких звітів, що підлягають звітуванню, які створює ваша програма.

<a name="gate-watcher"></a>

### Ворота спостерігач

Переглядач шлюзу реєструє дані та результати перевірок воріт та політики вашим додатком. Якщо ви хочете виключити певні здібності з реєстрації спостерігачем, ви можете вказати їх у`ignore_abilities`варіант у вашому`config/telescope.php`файл:

    'watchers' => [
        Watchers\GateWatcher::class => [
            'enabled' => env('TELESCOPE_GATE_WATCHER', true),
            'ignore_abilities' => ['viewNova'],
        ],
        ...
    ],

<a name="job-watcher"></a>

### Job Watcher

Диспетчер вакансій реєструє дані та стан будь-яких вакансій, надісланих вашим додатком.

<a name="log-watcher"></a>

### Запис журналу

Переглядач журналів записує дані журналу для будь-яких журналів, написаних вашою програмою.

<a name="mail-watcher"></a>

### Поштовий спостерігач

Програма для перегляду пошти дозволяє переглядати в браузері попередній перегляд електронних листів разом із відповідними даними. Ви також можете завантажити електронне повідомлення як`.eml`файл.

<a name="model-watcher"></a>

### Модель спостерігача

Модель спостерігача записує зміни моделі, коли Eloquent`created`,`updated`,`restored`, або`deleted`подія відправляється. Ви можете вказати, які події моделі слід записувати за допомогою спостерігачів`events`варіант:

    'watchers' => [
        Watchers\ModelWatcher::class => [
            'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
            'events' => ['eloquent.created*', 'eloquent.updated*'],
        ],
        ...
    ],

<a name="notification-watcher"></a>

### Спостерігач сповіщень

Переглядач сповіщень реєструє всі сповіщення, надіслані вашою програмою. Якщо сповіщення спрацьовує електронною поштою, і у вас увімкнено функцію спостереження за поштою, електронна пошта також буде доступна для попереднього перегляду на екрані програми перегляду пошти.

<a name="query-watcher"></a>

### Запит спостерігача

Переглядач запитів реєструє необроблений SQL, прив’язки та час виконання для всіх запитів, які виконуються вашим додатком. Також спостерігач позначає будь-які запити повільнішими за 100 мс як`slow`. Ви можете налаштувати поріг повільного запиту, використовуючи спостереження`slow`варіант:

    'watchers' => [
        Watchers\QueryWatcher::class => [
            'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
            'slow' => 50,
        ],
        ...
    ],

<a name="redis-watcher"></a>

### Редіс Ватчер

Переглядач Redis записує всі команди Redis, виконані вашою програмою. Якщо ви використовуєте Redis для кешування, команди кешування також будуть записані Redis Watcher.

<a name="request-watcher"></a>

### Запит спостерігача

Переглядач запитів реєструє запит, заголовки, сеанс та дані відповідей, пов’язані з будь-якими запитами, обробленими додатком. Ви можете обмежити свої дані відповідей через`size_limit`(у КБ) варіант:

    'watchers' => [
        Watchers\RequestWatcher::class => [
            'enabled' => env('TELESCOPE_REQUEST_WATCHER', true),
            'size_limit' => env('TELESCOPE_RESPONSE_SIZE_LIMIT', 64),
        ],
        ...
    ],

<a name="schedule-watcher"></a>

### Графік спостерігача

Диспетчер розкладів записує команди та результати будь-яких запланованих завдань, що виконуються вашим додатком.

<a name="view-watcher"></a>

### Переглянути спостерігач

Переглядач переглядів реєструє ім’я подання, шлях, дані та "композитори", що використовуються під час рендерингу переглядів.

<a name="displaying-user-avatars"></a>

## Відображення аватарів користувачів

На інформаційній панелі телескопа відображається аватар користувача для користувача, який увійшов у систему при збереженні заданого запису. За замовчуванням Telescope отримує аватари за допомогою веб-служби Gravatar. Однак ви можете налаштувати URL-адресу аватару, зареєструвавши зворотний дзвінок у своєму`TelescopeServiceProvider`. Зворотний дзвінок отримає ідентифікатор користувача та адресу електронної пошти та повинен повернути URL-адресу зображення аватара користувача:

    use App\Models\User;
    use Laravel\Telescope\Telescope;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        Telescope::avatar(function ($id, $email) {
            return '/avatars/'.User::find($id)->avatar_path;
        });
    }
