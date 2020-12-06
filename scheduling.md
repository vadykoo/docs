# Планування завдань

-   [Вступ](#introduction)
-   [Визначення графіків](#defining-schedules)
    -   [Планування команд ремісників](#scheduling-artisan-commands)
    -   [Планування роботи в черзі](#scheduling-queued-jobs)
    -   [Планування команд оболонки](#scheduling-shell-commands)
    -   [Заплануйте параметри частоти](#schedule-frequency-options)
    -   [Часові зони](#timezones)
    -   [Запобігання перекриванню завдань](#preventing-task-overlaps)
    -   [Запуск завдань на одному сервері](#running-tasks-on-one-server)
    -   [Фонові завдання](#background-tasks)
    -   [Режим обслуговування](#maintenance-mode)
-   [Вихід завдання](#task-output)
-   [Гачки завдань](#task-hooks)

<a name="introduction"></a>

## Вступ

Раніше ви могли створювати запис Cron для кожного завдання, яке вам потрібно було запланувати на своєму сервері. Однак це може швидко стати проблемою, оскільки ваш графік завдань більше не контролюється джерелом, і ви повинні SSH на своєму сервері, щоб додати додаткові записи Cron.

Планувальник команд Laravel дозволяє вам вільно та виразно визначити свій графік команд у самому Laravel. При використанні планувальника на вашому сервері потрібен лише один запис Cron. Розклад ваших завдань визначений у`app/Console/Kernel.php`файлів`schedule`метод. Щоб допомогти вам розпочати, у методі визначено простий приклад.

<a name="starting-the-scheduler"></a>

### Запуск планувальника

При використанні планувальника вам потрібно лише додати наступний запис Cron на ваш сервер. Якщо ви не знаєте, як додати записи Cron на свій сервер, подумайте про використання такої послуги, як[Кузня Laravel](https://forge.laravel.com)який може керувати записами Cron для вас:

    * * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1

Цей Cron щохвилини телефонуватиме до планувальника команд Laravel. Коли`schedule:run`команда виконується, Laravel оцінить ваші заплановані завдання та виконає відповідні завдання.

<a name="starting-the-scheduler-locally"></a>

### Запуск планувальника локально

Як правило, ви не додаєте запис планувальника Cron до вашої локальної машини розробки. Натомість ви можете використовувати`schedule:work`Artisan командування. Ця команда буде виконуватися на передньому плані та викликати планувальник щохвилини, поки ви не вийдете з команди:

    php artisan schedule:work

<a name="defining-schedules"></a>

## Визначення графіків

Ви можете визначити всі заплановані завдання в`schedule`метод`App\Console\Kernel`клас. Для початку давайте розглянемо приклад планування завдання. У цьому прикладі ми заплануємо a`Closure`дзвонити щодня опівночі. У межах`Closure`ми виконаємо запит до бази даних для очищення таблиці:

    <?php

    namespace App\Console;

    use Illuminate\Console\Scheduling\Schedule;
    use Illuminate\Foundation\Console\Kernel as ConsoleKernel;
    use Illuminate\Support\Facades\DB;

    class Kernel extends ConsoleKernel
    {
        /**
         * The Artisan commands provided by your application.
         *
         * @var array
         */
        protected $commands = [
            //
        ];

        /**
         * Define the application's command schedule.
         *
         * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
         * @return void
         */
        protected function schedule(Schedule $schedule)
        {
            $schedule->call(function () {
                DB::table('recent_users')->delete();
            })->daily();
        }
    }

Окрім планування за допомогою Closures, ви також можете використовувати[викликаються об'єкти](https://secure.php.net/manual/en/language.oop5.magic.php#object.invoke). Невідкличні об'єкти - це прості класи PHP, які містять`__invoke`метод:

    $schedule->call(new DeleteRecentUsers)->daily();

<a name="scheduling-artisan-commands"></a>

### Планування команд ремісників

На додаток до планування дзвінків на закриття, ви також можете запланувати[Ремісничі команди](/docs/{{version}}/artisan)та команди операційної системи. Наприклад, ви можете використовувати`command`спосіб запланувати команду Artisan, використовуючи або назву команди, або клас:

    $schedule->command('emails:send Taylor --force')->daily();

    $schedule->command(EmailsCommand::class, ['Taylor', '--force'])->daily();

<a name="scheduling-queued-jobs"></a>

### Планування роботи в черзі

`job`метод може бути використаний для планування a[робота в черзі](/docs/{{version}}/queues). Цей метод забезпечує зручний спосіб планування робіт без використання`call`спосіб вручну створити Закриття для черги на завдання:

    $schedule->job(new Heartbeat)->everyFiveMinutes();

    // Dispatch the job to the "heartbeats" queue...
    $schedule->job(new Heartbeat, 'heartbeats')->everyFiveMinutes();

<a name="scheduling-shell-commands"></a>

### Планування команд оболонки

`exec`метод може бути використаний для видачі команди операційній системі:

    $schedule->exec('node /home/forge/script.js')->daily();

<a name="schedule-frequency-options"></a>

### Заплануйте параметри частоти

Існує безліч графіків, які ви можете призначити для свого завдання:

| Метод                             | Опис                                                    |
| --------------------------------- | ------------------------------------------------------- |
| `->cron('* * * * *');`            | Запускає завдання за власним розкладом Cron            |
| `->everyMinute();`                | Запускає завдання щохвилини                            |
| `->everyTwoMinutes();`            | Запускає завдання кожні дві хвилини                    |
| `->everyThreeMinutes();`          | Запускає завдання кожні три хвилини                    |
| `->everyFourMinutes();`           | Запускає завдання кожні чотири хвилини                 |
| `->everyFiveMinutes();`           | Запускає завдання кожні п’ять хвилин                   |
| `->everyTenMinutes();`            | Запускає завдання кожні десять хвилин                  |
| `->everyFifteenMinutes();`        | Запускає завдання кожні п’ятнадцять хвилин             |
| `->everyThirtyMinutes();`         | Запускає завдання кожні тридцять хвилин                |
| `->hourly();`                     | Запускає завдання щогодини                             |
| `->hourlyAt(17);`                 | Запускає завдання кожну годину о 17 хвилинах за годину |
| `->everyTwoHours();`              | Запускає завдання кожні дві години                     |
| `->everyThreeHours();`            | Запускає завдання кожні три години                     |
| `->everyFourHours();`             | Запускає завдання кожні чотири години                  |
| `->everySixHours();`              | Запускає завдання кожні шість годин                    |
| `->daily();`                      | Запускає завдання щодня опівночі                       |
| `->dailyAt('13:00');`             | Запускає завдання щодня о 13:00                        |
| `->twiceDaily(1, 13);`            | Запускає завдання щодня о 1:00 та 13:00                |
| `->weekly();`                     | Запускає завдання щонеділі о 00:00                     |
| `->weeklyOn(1, '8:00');`          | Запускає завдання щотижня в понеділок о 8:00           |
| `->monthly();`                    | Запускає завдання в перший день кожного місяця о 00:00 |
| `->monthlyOn(4, '15:00');`        | Запускає завдання щомісяця 4-го о 15:00                |
| `->twiceMonthly(1, 16, '13:00');` | Запускає завдання щомісяця 1-го та 16-го о 13:00       |
| `->lastDayOfMonth('15:00');`      | Запускає завдання в останній день місяця о 15:00       |
| `->quarterly();`                  | Запускає завдання в перший день кожної чверті о 00:00  |
| `->yearly();`                     | Запускає завдання в перший день кожного року о 00:00   |
| `->yearlyOn(6, 1, '17:00');`      | Запускає завдання щороку 1 червня о 17:00              |
| `->timezone('America/New_York');` | Встановіть часовий пояс                                 |

Ці методи можуть поєднуватися з додатковими обмеженнями для створення ще більш точно налаштованих графіків, які виконуються лише в певні дні тижня. Наприклад, щоб запланувати команду для запуску щотижня в понеділок:

    // Run once per week on Monday at 1 PM...
    $schedule->call(function () {
        //
    })->weekly()->mondays()->at('13:00');

    // Run hourly from 8 AM to 5 PM on weekdays...
    $schedule->command('foo')
              ->weekdays()
              ->hourly()
              ->timezone('America/Chicago')
              ->between('8:00', '17:00');

Нижче наведено перелік додаткових обмежень розкладу:

| Метод                                    | Опис                                                       |                                    |
| ---------------------------------------- | ---------------------------------------------------------- | ---------------------------------- |
| `->weekdays();`                          | Обмежте завдання буднями                                   |                                    |
| `->weekends();`                          | Обмежте завдання вихідними                                 |                                    |
| `->sundays();`                           | Обмежте завдання неділею                                   |                                    |
| `->mondays();`                           | Обмежте завдання понеділком                                |                                    |
| `->tuesdays();`                          | Обмежте завдання вівторком                                 |                                    |
| `->wednesdays();`                        | Обмежте завдання середою                                   |                                    |
| `->thursdays();`                         | Обмежте завдання четвергом                                 |                                    |
| `->fridays();`                           | Обмежте завдання п’ятницею                                 |                                    |
| `->saturdays();`                         | Обмежте завдання суботою                                   |                                    |
| \`-> дні (масив                          | змішаний); \`                                              | Обмежте завдання конкретними днями |
| `->between($startTime, $endTime);`       | Обмежте завдання для запуску між часом початку та кінця    |                                    |
| `->unlessBetween($startTime, $endTime);` | Обмежте завдання не запускатись між часом початку та кінця |                                    |
| `->when(Closure);`                       | Обмежте завдання на основі перевірки істини                |                                    |
| `->environments($env);`                  | Обмежте завдання конкретним середовищем                    |                                    |

<a name="day-constraints"></a>

#### Денні обмеження

`days`метод може бути використаний для обмеження виконання завдання певними днями тижня. Наприклад, ви можете запланувати команду для запуску щогодини в неділю та середу:

    $schedule->command('reminders:send')
                    ->hourly()
                    ->days([0, 3]);

<a name="between-time-constraints"></a>

#### Між часовими обмеженнями

`between`метод може бути використаний для обмеження виконання завдання на основі часу доби:

    $schedule->command('reminders:send')
                        ->hourly()
                        ->between('7:00', '22:00');

Подібним чином`unlessBetween`метод може бути використаний для виключення виконання завдання протягом певного періоду:

    $schedule->command('reminders:send')
                        ->hourly()
                        ->unlessBetween('23:00', '4:00');

<a name="truth-test-constraints"></a>

#### Обмеження перевірки істини

`when`метод може бути використаний для обмеження виконання завдання на основі результату даного тесту на істинність. Іншими словами, якщо дане`Closure`повертається`true`, завдання буде виконуватися до тих пір, поки жодні інші обмежувальні умови не заважають виконувати завдання:

    $schedule->command('emails:send')->daily()->when(function () {
        return true;
    });

`skip`Метод може розглядатися як зворотний`when`. Якщо`skip`метод повертає`true`, заплановане завдання не буде виконано:

    $schedule->command('emails:send')->daily()->skip(function () {
        return true;
    });

При використанні прикутий`when`методів, запланована команда виконуватиметься, лише якщо все`when`умови повертаються`true`.

<a name="environment-constraints"></a>

#### Обмеження довкілля

`environments`метод може бути використаний для виконання завдань лише в заданих середовищах:

    $schedule->command('emails:send')
                ->daily()
                ->environments(['staging', 'production']);

<a name="timezones"></a>

### Часові зони

Використання`timezone`методом, ви можете вказати, що час запланованого завдання повинен інтерпретуватися в межах заданого часового поясу:

    $schedule->command('report:generate')
             ->timezone('America/New_York')
             ->at('02:00')

Якщо ви призначаєте однаковий часовий пояс усім запланованим завданням, можливо, ви захочете визначити`scheduleTimezone`метод у вашому`app/Console/Kernel.php`файл. Цей метод повинен повертати часовий пояс за замовчуванням, який повинен бути призначений для всіх запланованих завдань:

    /**
     * Get the timezone that should be used by default for scheduled events.
     *
     * @return \DateTimeZone|string|null
     */
    protected function scheduleTimezone()
    {
        return 'America/Chicago';
    }

> {note} Пам’ятайте, що деякі часові пояси використовують перехід на літній час. Коли відбуваються зміни переходу на літній час, заплановане завдання може виконуватися двічі або навіть не виконуватися взагалі. З цієї причини ми рекомендуємо уникати планування часових поясів, коли це можливо.

<a name="preventing-task-overlaps"></a>

### Запобігання перекриванню завдань

За замовчуванням заплановані завдання будуть виконуватися, навіть якщо попередній екземпляр завдання все ще працює. Щоб запобігти цьому, ви можете використовувати`withoutOverlapping`метод:

    $schedule->command('emails:send')->withoutOverlapping();

У цьому прикладі`emails:send`[Artisan командування](/docs/{{version}}/artisan)буде запускатися щохвилини, якщо вона ще не працює.`withoutOverlapping`метод особливо корисний, якщо у вас є завдання, які різко різняться за часом виконання, не даючи вам точно передбачити, скільки часу триватиме дане завдання.

При необхідності ви можете вказати, скільки хвилин має пройти до закінчення терміну дії "без перекриття". За замовчуванням блокування закінчується через 24 години:

    $schedule->command('emails:send')->withoutOverlapping(10);

<a name="running-tasks-on-one-server"></a>

### Запуск завдань на одному сервері

> {note} Щоб використовувати цю функцію, ваша програма повинна використовувати`database`,`memcached`, або`redis`кеш-драйвер як драйвер кешу вашого додатка за замовчуванням. Крім того, усі сервери повинні взаємодіяти з одним сервером центральної кеш-пам’яті.

Якщо ваша програма працює на декількох серверах, ви можете обмежити заплановане завдання лише на одному сервері. Наприклад, припустимо, що у вас є заплановане завдання, яке генерує новий звіт щоп’ятниці ввечері. Якщо планувальник завдань працює на трьох робочих серверах, заплановане завдання буде виконуватися на всіх трьох серверах і тричі генеруватиме звіт. Не добре!

Щоб вказати, що завдання має виконуватися лише на одному сервері, використовуйте`onOneServer`метод при визначенні запланованого завдання. Перший сервер, який отримає завдання, забезпечить атомний замок завдання, щоб запобігти одночасному виконанню того самого завдання на інших серверах:

    $schedule->command('report:generate')
                    ->fridays()
                    ->at('17:00')
                    ->onOneServer();

<a name="background-tasks"></a>

### Фонові завдання

За замовчуванням кілька команд, запланованих одночасно, будуть виконуватися послідовно. Якщо у вас є тривалі команди, це може спричинити запуск наступних команд набагато пізніше, ніж передбачалося. Якщо ви хочете запускати команди у фоновому режимі, щоб усі вони могли працювати одночасно, ви можете використовувати`runInBackground`метод:

    $schedule->command('analytics:report')
             ->daily()
             ->runInBackground();

> {примітка}`runInBackground`метод може використовуватися лише при плануванні завдань через`command`і`exec`методи.

<a name="maintenance-mode"></a>

### Режим обслуговування

Заплановані завдання Laravel не запускатимуться, коли Laravel працює[режим обслуговування](/docs/{{version}}/configuration#maintenance-mode), оскільки ми не хочемо, щоб ваші завдання заважали незавершеному обслуговуванню, яке ви можете виконувати на своєму сервері. Однак, якщо ви хочете змусити запустити завдання навіть у режимі обслуговування, ви можете використовувати`evenInMaintenanceMode`метод:

    $schedule->command('emails:send')->evenInMaintenanceMode();

<a name="task-output"></a>

## Вихід завдання

Планувальник Laravel надає кілька зручних методів для роботи з результатом, що генерується запланованими завданнями. По-перше, за допомогою`sendOutputTo`методом, ви можете надіслати вихідні дані у файл для подальшої перевірки:

    $schedule->command('emails:send')
             ->daily()
             ->sendOutputTo($filePath);

Якщо ви хочете додати вихідні дані до даного файлу, ви можете використовувати файл`appendOutputTo`метод:

    $schedule->command('emails:send')
             ->daily()
             ->appendOutputTo($filePath);

Використання`emailOutputTo`Ви можете надіслати вихідні дані на електронну адресу за вашим вибором. Перш ніж надсилати вихідні дані завдання електронною поштою, слід налаштувати Laravel[послуги електронної пошти](/docs/{{version}}/mail):

    $schedule->command('foo')
             ->daily()
             ->sendOutputTo($filePath)
             ->emailOutputTo('foo@example.com');

Якщо ви хочете надіслати вихідні дані лише електронною поштою, якщо команда не вдається, використовуйте`emailOutputOnFailure`метод:

    $schedule->command('foo')
             ->daily()
             ->emailOutputOnFailure('foo@example.com');

> {примітка}`emailOutputTo`,`emailOutputOnFailure`,`sendOutputTo`, і`appendOutputTo`методи є ексклюзивними для`command`і`exec`методи.

<a name="task-hooks"></a>

## Гачки завдань

Використання`before`і`after`методами, ви можете вказати код, який буде виконуватися до і після завершення запланованого завдання:

    $schedule->command('emails:send')
             ->daily()
             ->before(function () {
                 // Task is about to start...
             })
             ->after(function () {
                 // Task is complete...
             });

`onSuccess`і`onFailure`Методи дозволяють вказати код, який буде виконуватися, якщо заплановане завдання вдається або не вдається:

    $schedule->command('emails:send')
             ->daily()
             ->onSuccess(function () {
                 // The task succeeded...
             })
             ->onFailure(function () {
                 // The task failed...
             });

Якщо вихідні дані доступні з вашої команди, ви можете отримати доступ до них у вашому`after`,`onSuccess`або`onFailure`гачки шляхом набору тексту`Illuminate\Support\Stringable`екземпляр як`$output`у визначенні закриття вашого гачка:

    use Illuminate\Support\Stringable;

    $schedule->command('emails:send')
             ->daily()
             ->onSuccess(function (Stringable $output) {
                 // The task succeeded...
             })
             ->onFailure(function (Stringable $output) {
                 // The task failed...
             });

<a name="pinging-urls"></a>

#### Пінг URL-адрес

Використання`pingBefore`і`thenPing`методів, планувальник може автоматично пінгувати дану URL-адресу до або після завершення завдання. Цей метод корисний для сповіщення зовнішньої служби, наприклад[Laravel Send](https://envoyer.io), що ваше заплановане завдання розпочинається або завершило виконання:

    $schedule->command('emails:send')
             ->daily()
             ->pingBefore($url)
             ->thenPing($url);

`pingBeforeIf`і`thenPingIf`Методи можуть бути використані для пінгування даної URL-адреси, лише якщо задана умова`true`:

    $schedule->command('emails:send')
             ->daily()
             ->pingBeforeIf($condition, $url)
             ->thenPingIf($condition, $url);

`pingOnSuccess`і`pingOnFailure`Методи можуть бути використані для пінгування даної URL-адреси, лише якщо завдання успішне або не вдається:

    $schedule->command('emails:send')
             ->daily()
             ->pingOnSuccess($successUrl)
             ->pingOnFailure($failureUrl);

Для всіх методів ping потрібна бібліотека Guzzle HTTP. Ви можете додати Guzzle до свого проекту за допомогою менеджера пакетів Composer:

    composer require guzzlehttp/guzzle
