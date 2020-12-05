# Laravel Horizon

-   [Вступ](#introduction)
-   [Встановлення](#installation)
    -   [Конфігурація](#configuration)
    -   [Авторизація інформаційної панелі](#dashboard-authorization)
-   [Оновлення Horizon](#upgrading-horizon)
-   [Runningгоризонт](#running-horizon)
    -   [Розгортання Horizon](#deploying-horizon)
-   [Теги](#tags)
-   [Повідомлення](#notifications)
-   [Метрики](#metrics)
-   [Видалення невдалих завдань](#deleting-failed-jobs)
-   [Очищення Jobs із черг](#clearing-jobs-from-queues)

<a name="introduction"></a>

## Вступ

Horizon забезпечує чудову інформаційну панель та конфігурацію, керовану кодом, для ваших черг Redis, що працюють від Laravel. Horizon дозволяє легко відстежувати ключові показники вашої системи черги, такі як пропускна здатність роботи, час роботи та збої в роботі.

Вся ваша конфігурація працівника зберігається в одному простому конфігураційному файлі, що дозволяє вашій конфігурації залишатися під контролем Source, де вся ваша команда може співпрацювати.

<img src="https://laravel.com/img/docs/horizon-example.png">

<a name="installation"></a>

## Встановлення

> {note} Ви повинні переконатися, що для з’єднання з чергою встановлено значення`redis`у вашому`queue`файл конфігурації.

Ви можете використовувати Composer для встановлення Horizon у свій проект Laravel:

    composer require laravel/horizon

Після встановлення Horizon опублікуйте його Assets за допомогою`horizon:install`Команда ремісників:

    php artisan horizon:install

<a name="configuration"></a>

### Конфігурація

Після публікації активів Horizon його основний конфігураційний файл буде розміщений за адресою`config/horizon.php`. Цей конфігураційний файл дозволяє налаштувати параметри вашого працівника, і кожен параметр конфігурації включає опис його призначення, тому обов’язково ретельно вивчіть цей файл.

> {note} Ви повинні переконатися, що`environments`частина вашого`horizon`Файл конфігурації містить запис для кожного середовища, в якому ви плануєте запустити Horizon.

<a name="balance-options"></a>

#### Варіанти балансу

Horizon дозволяє вибрати з трьох стратегій балансування:`simple`,`auto`, і`false`.`simple`стратегія, яка є конфігураційним файлом за замовчуванням, розподіляє майбутні завдання рівномірно між процесами:

    'balance' => 'simple',

`auto`Стратегія регулює кількість робочих процесів у черзі на основі поточного навантаження в черзі. Наприклад, якщо ваш`notifications`черга має 1000 Jobs очікування, поки ваш`render`Черга порожня, Horizon виділить для вас більше працівників`notifications`чергу, поки вона не порожня. Коли`balance`для параметра встановлено значення`false`, буде використана поведінка Laravel за замовчуванням, яка обробляє черги в тому порядку, як вони вказані у вашій конфігурації.

При використанні`auto`стратегію, ви можете визначити`minProcesses`і`maxProcesses`параметри конфігурації для контролю мінімальної та максимальної кількості процесів Horizon повинен масштабуватися вгору та вниз до:

    'environments' => [
        'production' => [
            'supervisor-1' => [
                'connection' => 'redis',
                'queue' => ['default'],
                'balance' => 'auto',
                'minProcesses' => 1,
                'maxProcesses' => 10,
                'balanceMaxShift' => 1,
                'balanceCooldown' => 3,
                'tries' => 3,
            ],
        ],
    ],

`balanceMaxShift`і`balanceCooldown`значення конфігурації, щоб визначити, наскільки швидко Horizon буде масштабуватися, щоб задовольнити попит працівників У наведеному вище прикладі кожні три секунди буде створено або знищено максимум один новий процес. Ви можете налаштувати ці значення за необхідності залежно від потреб вашої програми.

<a name="job-trimming"></a>

#### Обрізка вакансій

`horizon`Файл конфігурації дозволяє вам визначити, як довго зберігатимуться останні та невдалі завдання (у хвилинах). За замовчуванням останні завдання зберігаються одну годину, а невдалі - тиждень:

    'trim' => [
        'recent' => 60,
        'failed' => 10080,
    ],

<a name="dashboard-authorization"></a>

### Авторизація інформаційної панелі

Horizon виставляє інформаційну панель на`/horizon`. За замовчуванням ви зможете отримати доступ до цієї інформаційної панелі лише в`local`навколишнє середовище. В межах вашого`app/Providers/HorizonServiceProvider.php`файл, є файл`gate`метод. Ця авторизація контролює доступ до Horizon в**неLocal**середовищах. Ви можете змінювати ці ворота за необхідності, щоб обмежити доступ до вашої інсталяції Horizon:

    /**
     * Register the Horizon gate.
     *
     * This gate determines who can access Horizon in non-local environments.
     *
     * @return void
     */
    protected function gate()
    {
        Gate::define('viewHorizon', function ($user) {
            return in_array($user->email, [
                'taylor@laravel.com',
            ]);
        });
    }

> {note} Пам'ятайте, що Laravel вводить_автентифікований_користувача до Воріт автоматично. Якщо ваш додаток забезпечує захист Horizon іншим способом, наприклад, обмеженнями IP, тоді вашим користувачам Horizon може не знадобитися "входити". Тому вам потрібно буде змінитися`function ($user)`вище до`function ($user = null)`змусити Laravel не вимагати автентифікації.

<a name="upgrading-horizon"></a>

## Оновлення Horizon

Під час оновлення до нової основної версії Horizon важливо уважно переглянути[посібник з оновлення](https://github.com/laravel/horizon/blob/master/UPGRADE.md).

Крім того, під час оновлення до будь-якої нової версії Horizon, вам слід повторно опублікувати Assets Horizon:

    php artisan horizon:publish

Щоб оновити ресурси та уникнути проблем із майбутніми оновленнями, ви можете додати команду до`post-update-cmd`скрипти у вашому`composer.json`файл:

    {
        "scripts": {
            "post-update-cmd": [
                "@php artisan horizon:publish --ansi"
            ]
        }
    }

<a name="running-horizon"></a>

## Runningгоризонт

Після налаштування ваших робітників у`config/horizon.php`конфігураційний файл, ви можете запустити Horizon за допомогою`horizon`artisan командування. Ця одна команда розпочне всі налаштовані вами працівники:

    php artisan horizon

Ви можете призупинити процес Horizon і доручити йому продовжувати обробку завдань за допомогою`horizon:pause`і`horizon:continue`Ремісничі команди:

    php artisan horizon:pause

    php artisan horizon:continue

Ви також можете призупинити та продовжити роботу певних керівників Horizon (групи працівників), використовуючи`horizon:pause-supervisor`і`horizon:continue-supervisor`Ремісничі команди:

    php artisan horizon:pause-supervisor supervisor-1

    php artisan horizon:continue-supervisor supervisor-1

Ви можете перевірити поточний стан процесу Horizon за допомогою`horizon:status`Команда ремісників:

    php artisan horizon:status

Ви можете витончено завершити головний процес Horizon на вашому комп'ютері за допомогою`horizon:terminate`artisan командування. Усі завдання, які Horizon обробляє в даний час, будуть виконані, а потім Horizon вийде:

    php artisan horizon:terminate

<a name="deploying-horizon"></a>

### Розгортання Horizon

Якщо ви розгортаєте Horizon на реальному сервері, вам слід налаштувати монітор процесу для моніторингу`php artisan horizon`команду та перезапустіть її, якщо вона несподівано вийде. Розгортаючи свіжий код на своєму сервері, вам потрібно буде доручити основному процесу Horizon завершити процес, щоб його можна було перезапустити монітором процесу та отримати зміни коду.

<a name="installing-supervisor"></a>

#### Встановлення Supervisor

Supervisor - це монітор процесів для операційної системи Linux, який автоматично перезапустить ваш`horizon`обробляти, якщо він не вдається. Щоб встановити Supervisor на Ubuntu, ви можете використати таку команду:

    sudo apt-get install supervisor

> {tip} Якщо налаштування супервізора самостійно здається вражаючим, розгляньте можливість використання[Кузня Laravel](https://forge.laravel.com), який автоматично встановить і налаштує Supervisor для ваших проектів Laravel.

<a name="supervisor-configuration"></a>

#### Конфігурація супервізора

Файли конфігурації супервізора зазвичай зберігаються в`/etc/supervisor/conf.d`каталог. У цьому каталозі ви можете створити будь-яку кількість файлів конфігурації, які вказують керівнику, як слід контролювати ваші процеси. Наприклад, давайте створимо файл`horizon.conf`файл, який запускається та контролює a`horizon`процес:

    [program:horizon]
    process_name=%(program_name)s
    command=php /home/forge/app.com/artisan horizon
    autostart=true
    autorestart=true
    user=forge
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/horizon.log
    stopwaitsecs=3600

> {note} Ви повинні переконатися, що значення`stopwaitsecs`більше, ніж кількість секунд, витрачених вашим найдовшим завданням. В іншому випадку Супервайзер може вбити завдання до його обробки.

<a name="starting-supervisor"></a>

#### Початковий керівник

Після створення файлу конфігурації ви можете оновити конфігурацію супервізора та запустити процеси, використовуючи такі команди:

    sudo supervisorctl reread

    sudo supervisorctl update

    sudo supervisorctl start horizon

Щоб отримати додаткову інформацію про наглядача, зверніться до[Документація керівника](http://supervisord.org/index.html).

<a name="tags"></a>

## Теги

Horizon дозволяє призначати "теги" завданням, включаючи доступність, трансляції подій, Notification та прослуховувачі подій у черзі. Насправді Horizon буде розумно та автоматично додавати теги до більшості завдань залежно від Eloquent моделей, які приєднані до завдання. Наприклад, подивіться на таку роботу:

    <?php

    namespace App\Jobs;

    use App\Models\Video;
    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Queue\SerializesModels;

    class RenderVideo implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        /**
         * The video instance.
         *
         * @var \App\Models\Video
         */
        public $video;

        /**
         * Create a new job instance.
         *
         * @param  \App\Models\Video  $video
         * @return void
         */
        public function __construct(Video $video)
        {
            $this->video = $video;
        }

        /**
         * Execute the job.
         *
         * @return void
         */
        public function handle()
        {
            //
        }
    }

Якщо ця робота поставлена ​​в чергу з`App\Models\Video`екземпляр, який має`id`з`1`, він автоматично отримає тег`App\Models\Video:1`. Це тому, що Horizon вивчить властивості завдання для будь-яких Eloquent моделей. Якщо знайдені Eloquent моделі, Horizon розумно позначить завдання, використовуючи назву класу моделі та первинний ключ:

    $video = App\Models\Video::find(1);

    App\Jobs\RenderVideo::dispatch($video);

<a name="manually-tagging"></a>

#### Позначення вручну

Якщо ви хочете вручну визначити теги для одного з об'єктів, що знаходяться в черзі, ви можете визначити файл`tags`метод на класі:

    class RenderVideo implements ShouldQueue
    {
        /**
         * Get the tags that should be assigned to the job.
         *
         * @return array
         */
        public function tags()
        {
            return ['render', 'video:'.$this->video->id];
        }
    }

<a name="notifications"></a>

## Повідомлення

> **Примітка:**Налаштовуючи Horizon для надсилання сповіщень Slack або SMS, слід переглянути[передумови для відповідного драйвера сповіщень](/docs/{{version}}/notifications).

Якщо ви хочете отримати повідомлення, коли одна з ваших черг має тривалий час очікування, ви можете скористатися`Horizon::routeMailNotificationsTo`,`Horizon::routeSlackNotificationsTo`, і`Horizon::routeSmsNotificationsTo`методи. Ви можете викликати ці методи з додатків`HorizonServiceProvider`:

    Horizon::routeMailNotificationsTo('example@example.com');
    Horizon::routeSlackNotificationsTo('slack-webhook-url', '#channel');
    Horizon::routeSmsNotificationsTo('15556667777');

<a name="configuring-notification-wait-time-thresholds"></a>

#### Налаштування порогів часу очікування Notification

Ви можете налаштувати, скільки секунд вважається "довгим очікуванням" у вашому`config/horizon.php`файл конфігурації.`waits`Параметр конфігурації в цьому файлі дозволяє контролювати поріг довгого очікування для кожної комбінації з'єднання / черги:

    'waits' => [
        'redis:default' => 60,
        'redis:critical,high' => 90,
    ],

<a name="metrics"></a>

## Метрики

Horizon включає інформаційну панель метрик, яка надає інформацію про вашу роботу та час очікування та пропускну здатність. Для заповнення цієї інформаційної панелі слід налаштувати Horizon`snapshot`Команда Artisan запускатись кожні п’ять хвилин через додатки[планувальник](/docs/{{version}}/scheduling):

    /**
     * Define the application's command schedule.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->command('horizon:snapshot')->everyFiveMinutes();
    }

<a name="deleting-failed-jobs"></a>

## Видалення невдалих завдань

Якщо ви хочете видалити невдале завдання, ви можете використовувати`horizon:forget`команди.`horizon:forget`команда приймає ідентифікатор невдалого завдання як єдиний аргумент:

    php artisan horizon:forget 5

<a name="clearing-jobs-from-queues"></a>

## Очищення Jobs із черг

Якщо ви хочете видалити всі завдання із черги за замовчуванням, ви можете зробити це за допомогою`horizon:clear`Команда ремісників:

    php artisan horizon:clear

Ви також можете надати`queue`можливість видалення завдань із певної черги:

    php artisan horizon:clear --queue=emails
