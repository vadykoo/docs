# Лісозаготівля

-   [Вступ](#introduction)
-   [Конфігурація](#configuration)
    -   [Будівництво Log Stacks](#building-log-stacks)
-   [Написання журнальних повідомлень](#writing-log-messages)
    -   [Запис на певні канали](#writing-to-specific-channels)
-   [Розширена настройка каналу Monolog](#advanced-monolog-channel-customization)
    -   [Налаштування Monolog для каналів](#customizing-monolog-for-channels)
    -   [Створення каналів обробки монологів](#creating-monolog-handler-channels)
    -   [Створення каналів на Factories](#creating-channels-via-factories)

<a name="introduction"></a>

## Вступ

Щоб допомогти вам дізнатись більше про те, що відбувається у вашій програмі, Laravel надає надійні служби реєстрації, які дозволяють реєструвати повідомлення у файли, журнал системних помилок і навіть у Slack, щоб повідомити всю вашу команду.

Під капотом Laravel використовує[Монолог](https://github.com/Seldaek/monolog)бібліотека, яка забезпечує підтримку різноманітних потужних обробників журналів. Laravel робить зручним налаштування цих обробників, дозволяючи змішувати та поєднувати їх, щоб налаштувати обробку журналів програми.

<a name="configuration"></a>

## Конфігурація

Вся конфігурація системи реєстрації вашої програми розміщена в`config/logging.php`файл конфігурації. Цей файл дозволяє вам налаштувати канали журналу програми, тому перегляньте кожен з доступних каналів та їх варіанти. Нижче ми розглянемо кілька загальних варіантів.

За замовчуванням Laravel буде використовувати`stack`канал при реєстрації повідомлень.`stack`Канал використовується для об'єднання кількох каналів журналу в один канал. Щоб отримати додаткову інформацію про будівельні стеки, перегляньте[документація нижче](#building-log-stacks).

<a name="configuring-the-channel-name"></a>

#### Налаштування назви каналу

За замовчуванням у Monolog створюється екземпляр із "назвою каналу", яке відповідає поточному середовищу, наприклад`production`або`local`. Щоб змінити це значення, додайте a`name`варіант конфігурації вашого каналу:

    'stack' => [
        'driver' => 'stack',
        'name' => 'channel-name',
        'channels' => ['single', 'slack'],
    ],

<a name="available-channel-drivers"></a>

#### Доступні драйвери каналів

| Ім'я         | Опис                                                                                           |
| ------------ | ---------------------------------------------------------------------------------------------- |
| `stack`      | Обгортка для полегшення створення "багатоканальних" каналів                                    |
| `single`     | Канал реєстратора на основі одного файлу або шляху (`StreamHandler`)                           |
| `daily`      | A`RotatingFileHandler`драйвер Monolog, який обертається щодня                                  |
| `slack`      | A`SlackWebhookHandler`драйвер Monolog на базі                                                  |
| `papertrail` | A`SyslogUdpHandler`драйвер Monolog на базі                                                     |
| `syslog`     | A`SyslogHandler`драйвер Monolog на базі                                                        |
| `errorlog`   | A`ErrorLogHandler`драйвер Monolog на базі                                                      |
| `monolog`    | Заводський драйвер Monolog, який може використовувати будь-який підтримуваний обробник Monolog |
| `custom`     | Драйвер, який викликає вказану фабрику для створення каналу                                    |

> {tip} Ознайомтеся з документацією на[розширена настройка каналу](#advanced-monolog-channel-customization)щоб дізнатись більше про`monolog`і`custom`водіїв.

<a name="configuring-the-single-and-daily-channels"></a>

#### Налаштування одинарного та щоденного каналів

`single`і`daily`канали мають три додаткові параметри конфігурації:`bubble`,`permission`, і`locking`.

| Ім'я         | Опис                                                                   | За замовчуванням |
| ------------ | ---------------------------------------------------------------------- | ---------------- |
| `bubble`     | Вказує, чи повинні повідомлення надходити на інші канали після обробки | `true`           |
| `permission` | Дозволи файлу журналу                                                  | `0644`           |
| `locking`    | Спроба заблокувати файл журналу перед записом до нього                 | `false`          |

<a name="configuring-the-papertrail-channel"></a>

#### Налаштування каналу Papertrail

`papertrail`канал вимагає`url`і`port`параметри конфігурації. Ви можете отримати ці значення з[Папертрейл](https://help.papertrailapp.com/kb/configuration/configuring-centralized-logging-from-php-apps/#send-events-from-php-app).

<a name="configuring-the-slack-channel"></a>

#### Налаштування каналу Slack

`slack`канал вимагає a`url`варіант конфігурації. Ця URL-адреса повинна відповідати URL-адресі для[вхідний вебхук](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks)які ви налаштували для своєї команди Slack. За замовчуванням Slack отримуватиме журнали лише на`critical`рівень і вище; однак ви можете налаштувати це в`logging`файл конфігурації.

<a name="building-log-stacks"></a>

### Будівництво Log Stacks

Як уже зазначалося,`stack`Драйвер дозволяє об'єднати кілька каналів в один журнал. Щоб проілюструвати, як використовувати стеки журналів, давайте розглянемо приклад конфігурації, який ви можете побачити у виробничому додатку:

    'channels' => [
        'stack' => [
            'driver' => 'stack',
            'channels' => ['syslog', 'slack'],
        ],

        'syslog' => [
            'driver' => 'syslog',
            'level' => 'debug',
        ],

        'slack' => [
            'driver' => 'slack',
            'url' => env('LOG_SLACK_WEBHOOK_URL'),
            'username' => 'Laravel Log',
            'emoji' => ':boom:',
            'level' => 'critical',
        ],
    ],

Давайте розберемо цю конфігурацію. По-перше, зверніть увагу на наш`stack`канал агрегує два інших канали через свій`channels`варіант:`syslog`і`slack`. Отже, при реєстрації повідомлень обидва ці канали матимуть можливість реєструвати повідомлення.

<a name="log-levels"></a>

#### Рівні журналу

Зверніть увагу на`level`параметр конфігурації, присутній на`syslog`і`slack`конфігурації каналів у прикладі вище. Цей параметр визначає мінімальний "рівень", яким повинно бути повідомлення, щоб канал міг реєструватися. Monolog, який надає послуги реєстрації Laravel, пропонує всі рівні журналу, визначені в[Специфікація RFC 5424](https://tools.ietf.org/html/rfc5424):**надзвичайна ситуація**,**попередження**,**критичний**,**помилка**,**увага**,**повідомлення**,**інформація**, і**відлагоджувати**.

Отже, уявімо, що ми реєструємо повідомлення за допомогою`debug`метод:

    Log::debug('An informational message.');

Враховуючи нашу конфігурацію,`syslog`канал запише повідомлення в системний журнал; однак, оскільки повідомлення про помилку немає`critical`або вище, він не буде надісланий Slack. Однак, якщо ми реєструємо`emergency`повідомлення, воно буде надіслано як до системного журналу, так і до Slack, оскільки`emergency`рівень перевищує наш мінімальний поріг рівня для обох каналів:

    Log::emergency('The system is down!');

<a name="writing-log-messages"></a>

## Написання журнальних повідомлень

Ви можете записувати інформацію до журналів, використовуючи`Log`[фасад](/docs/{{version}}/facades). Як зазначалося раніше, реєстратор забезпечує вісім рівнів реєстрації, визначених у[Специфікація RFC 5424](https://tools.ietf.org/html/rfc5424):**надзвичайна ситуація**,**попередження**,**критичний**,**помилка**,**увага**,**повідомлення**,**інформація**і**відлагоджувати**:

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

Отже, ви можете зателефонувати будь-якому з цих методів, щоб записати повідомлення на відповідний рівень. За замовчуванням повідомлення буде записано на канал журналу за замовчуванням, як налаштовано вашим`config/logging.php`файл конфігурації:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\User;
    use Illuminate\Support\Facades\Log;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            Log::info('Showing user profile for user: '.$id);

            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

<a name="contextual-information"></a>

#### Контекстна інформація

Масив контекстних даних також може бути переданий методам журналу. Ці контекстуальні дані будуть відформатовані та відображені з повідомленням журналу:

    Log::info('User failed to login.', ['id' => $user->id]);

<a name="writing-to-specific-channels"></a>

### Запис на певні канали

Іноді вам може знадобитися зареєструвати повідомлення на каналі, відмінному від стандартного для вашої програми. Ви можете використовувати`channel`метод на`Log`Facade для отримання та входу в будь-який канал, визначений у вашому файлі конфігурації:

    Log::channel('slack')->info('Something happened!');

Якщо ви хочете створити стек реєстрації на вимогу, що складається з декількох каналів, ви можете використовувати`stack`метод:

    Log::stack(['single', 'slack'])->info('Something happened!');

<a name="advanced-monolog-channel-customization"></a>

## Розширена настройка каналу Monolog

<a name="customizing-monolog-for-channels"></a>

### Налаштування Monolog для каналів

Іноді вам може знадобитися повний контроль над тим, як Monolog налаштовано на існуючий канал. Наприклад, ви можете налаштувати власний Monolog`FormatterInterface`реалізація для обробників даного каналу.

Для початку визначте a`tap`масив конфігурації каналу.`tap`масив повинен містити перелік класів, які повинні мати можливість налаштувати (або "зачепити") примірник Monolog після його створення:

    'single' => [
        'driver' => 'single',
        'tap' => [App\Logging\CustomizeFormatter::class],
        'path' => storage_path('logs/laravel.log'),
        'level' => 'debug',
    ],

Після налаштування`tap`на вашому каналі, ви готові визначити клас, який буде налаштовувати ваш екземпляр Monolog. Цей клас потребує лише одного методу:`__invoke`, який отримує`Illuminate\Log\Logger`екземпляр.`Illuminate\Log\Logger`instance проксіє всі виклики методів до базового екземпляра Monolog:

    <?php

    namespace App\Logging;

    use Monolog\Formatter\LineFormatter;

    class CustomizeFormatter
    {
        /**
         * Customize the given logger instance.
         *
         * @param  \Illuminate\Log\Logger  $logger
         * @return void
         */
        public function __invoke($logger)
        {
            foreach ($logger->getHandlers() as $handler) {
                $handler->setFormatter(new LineFormatter(
                    '[%datetime%] %channel%.%level_name%: %message% %context% %extra%'
                ));
            }
        }
    }

> {tip} Усі ваші класи "натискання" вирішуються[службовий контейнер](/docs/{{version}}/container), тому будь-які залежності конструктора, які вони потребують, будуть автоматично введені.

<a name="creating-monolog-handler-channels"></a>

### Створення каналів обробки монологів

Monolog має різноманітні[доступні обробники](https://github.com/Seldaek/monolog/tree/master/src/Monolog/Handler). У деяких випадках тип реєстратора, який ви хочете створити, - це просто драйвер Monolog із екземпляром конкретного обробника. Ці канали можна створити за допомогою`monolog`водій.

При використанні`monolog`водій,`handler`Параметр конфігурації використовується, щоб вказати, який обробник буде створений. За бажанням, будь-які параметри конструктора, необхідні обробнику, можуть бути вказані за допомогою`with`варіант конфігурації:

    'logentries' => [
        'driver'  => 'monolog',
        'handler' => Monolog\Handler\SyslogUdpHandler::class,
        'with' => [
            'host' => 'my.logentries.internal.datahubhost.company.com',
            'port' => '10000',
        ],
    ],

<a name="monolog-formatters"></a>

#### Monolog Formatters

При використанні`monolog`водій, Monolog`LineFormatter`буде використовуватися як форматтер за замовчуванням. Однак ви можете налаштувати тип форматування, що передається обробнику за допомогою`formatter`і`formatter_with`параметри конфігурації:

    'browser' => [
        'driver' => 'monolog',
        'handler' => Monolog\Handler\BrowserConsoleHandler::class,
        'formatter' => Monolog\Formatter\HtmlFormatter::class,
        'formatter_with' => [
            'dateFormat' => 'Y-m-d',
        ],
    ],

Якщо ви використовуєте обробник Monolog, який може забезпечити власний програму форматування, ви можете встановити значення`formatter`варіант конфігурації до`default`:

    'newrelic' => [
        'driver' => 'monolog',
        'handler' => Monolog\Handler\NewRelicHandler::class,
        'formatter' => 'default',
    ],

<a name="creating-channels-via-factories"></a>

### Створення каналів на Factories

Якщо ви хочете визначити повністю власний канал, в якому ви маєте повний контроль над створенням і конфігурацією Monolog, ви можете вказати`custom`введіть драйвер у вашому`config/logging.php`файл конфігурації. Ваша конфігурація повинна містити a`via`можливість вказати заводський клас, який буде викликаний для створення екземпляра Monolog:

    'channels' => [
        'custom' => [
            'driver' => 'custom',
            'via' => App\Logging\CreateCustomLogger::class,
        ],
    ],

Після налаштування`custom`каналу, ви готові визначити клас, який створить ваш екземпляр Monolog. Цей клас потребує лише одного методу:`__invoke`, який повинен повернути екземпляр Monolog:

    <?php

    namespace App\Logging;

    use Monolog\Logger;

    class CreateCustomLogger
    {
        /**
         * Create a custom Monolog instance.
         *
         * @param  array  $config
         * @return \Monolog\Logger
         */
        public function __invoke(array $config)
        {
            return new Logger(...);
        }
    }
