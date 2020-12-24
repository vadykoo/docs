git 6823f659ab97ee3144a632d440569cb0ccd85a45

---

# Консоль Artisan

-   [Вступ](#introduction)
    -   [Tinker (REPL)](#tinker)
-   [Написання команд](#writing-commands)
    -   [Генерація команд](#generating-commands)
    -   [Структура команд](#command-structure)
    -   [Команди замикання](#closure-commands)
-   [Визначення вхідних параметрів](#defining-input-expectations)
    -   [Аргументи](#arguments)
    -   [Опції](#options)
    -   [Вхідні масиви](#input-arrays)
    -   [Вхідні описи](#input-descriptions)
-   [Введення / виведення](#command-io)
    -   [Отримання даних вводу](#retrieving-input)
    -   [Запрошення на введення (Prompting For Input)](#prompting-for-input)
    -   [Вихідні дані](#writing-output)
-   [Реєстрація команд](#registering-commands)
-   [Програмне виконання команд](#programmatically-executing-commands)
    -   [Виклик команд з інших команд](#calling-commands-from-other-commands)
-   [Налаштування заглушки](#stub-customization)
-   [Події](#events)

<a name="introduction"></a>

## Вступ
 
Artisan - це інтерфейс командного рядка, що входить до складу Laravel. Він знаходится у корені вашого додатку як скрипт `artisan` і надає ряд корисних команд, які можуть допомогти вам під час створення вашого додатку. Щоб переглянути список усіх доступних команд Artisan, використовуйте команду `list`:

    php artisan list

Кожна команда також включає "довідку", яка відображає та описує доступні аргументи та опції команди. Щоб переглянути екран довідки, перед назвою команди слід вказати `help`:

    php artisan help migrate

<a name="laravel-sail"></a>

#### Laravel Sail
Якщо ви використовуєте [Laravel Sail](/docs/{{version}}/sail) як ваше локальне середовище розробки, не забувайте використовувати командний рядок `sail` для виклику команд Artisan. Sail виконуватиме ваші Artisan команди у контейнерах Docker вашого додатку.

    ./sail artisan list

<a name="tinker"></a>

### Tinker (REPL)

Laravel Tinker - це потужний REPL для фреймворку Laravel на базі пакету [PsySH](https://github.com/bobthecow/psysh).

<a name="installation"></a>

#### Встановлення

Усі додатки Laravel включають Tinker за замовчуванням. Однак, ви можете встановити його вручну, якщо він був видалений, використовуючи Composer:

    composer require laravel/tinker

> {tip} Шукаєте графічний інтерфейс для взаємодії з вашим додатком Laravel? Використайте [Tinkerwell](https://tinkerwell.app)!

<a name="usage"></a>

#### Використання

Tinker дозволяє взаємодіяти з усім вашим додатком Laravel у командному рядку, включаючи Eloquent ORM, jobs, events тощо. Щоб увійти в середовище Tinker, запустіть команду Artisan `tinker`:

    php artisan tinker

Ви можете опублікувати файл конфігурації Tinker за допомогою команди `vendor:publish`:

    php artisan vendor:publish --provider="Laravel\Tinker\TinkerServiceProvider"

> {примітка} При розміщенні завдання в черзі, хелпер функція `dispatch` та метод `dispatch` в класі `Dispatchable` залежать від збирачя сміття. Тому, використовуючи Tinker, слід використовувати `Bus::dispatch` або `Queue::push` щоб виконати dispatch jobs.

<a name="command-whitelist"></a>

#### Білий список команд

Tinker використовує білий список, щоб визначити, які команди Artisan дозволяється запускати в його оболонці. За замовчуванням, ви можете запустити команди `clear-compiled`,`down`,`env`,`inspire`,`migrate`,`optimize`,`up`. Якщо ви хочете додати до білого списку більше команд, ви можете додати їх до масиву `commands` у вашому файлі конфігурації `tinker.php`:

    'commands' => [
        // App\Console\Commands\ExampleCommand::class,
    ],

<a name="classes-that-should-not-be-aliased"></a>

#### Класи, які не мають бути аліасними

Як правило, Tinker автоматично створює псевдоніми (аліаси) для класів, коли вони потрібні в Tinker. Тим не менш, ви можете заборонити створювати аліаси для деяких класів. Для цього перерахуйте класи в `dont_alias` масиві вашого `tinker.php` файла конфігурації:

    'dont_alias' => [
        App\Models\User::class,
    ],

<a name="writing-commands"></a>

## Написання команд

На додаток до команд, наданих Artisan, ви також можете створити власні команди. Команди зазвичай зберігаються в директорії `app/Console/Commands`. Однак, ви можете обрати власне місце зберігання, за умови, що ваші команди може завантажувати Composer.

<a name="generating-commands"></a>

### Генерація команд

Для створення нової команди, використовуйте Artisan команду `make:command`. Ця команда створить новий клас команди у директорії `app/Console/Commands`. Не хвилюйтеся, якщо ця директорія не існує у вашому додатку, оскільки вона буде створена під час першого запуску Artisan команди `make:command`:

    php artisan make:command SendEmails

<a name="command-structure"></a>

### Структура команд

Після генерації команди вам слід визначити відповідні властивості класів `signature` і `description`. Ці властивості будуть використовуватися при відображенні вашої команди на `list` екрані. Властивість `signature` також дозволяє вам визначити [your command's input expectations](#defining-input-expectations). Метод
`handle` буде викликатися при виконанні вашої команди. Ви можете розмістити свою логіку команди у цьому методі.

Розглянемо приклад команди. Зверніть увагу, що ми можемо додавати будь-які залежності через командний метод `handle`. Laravel [сервіс контейнер](/docs/{{version}}/container) автоматично впровадить усі типізовані залежності:

   <?php

    namespace App\Console\Commands;

    use App\Models\User;
    use App\Support\DripEmailer;
    use Illuminate\Console\Command;

    class SendEmails extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'mail:send {user}';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Send a marketing email to a user';

        /**
         * Create a new command instance.
         *
         * @return void
         */
        public function __construct()
        {
            parent::__construct();
        }

        /**
         * Execute the console command.
         *
         * @param  \App\Support\DripEmailer  $drip
         * @return mixed
         */
        public function handle(DripEmailer $drip)
        {
            $drip->send(User::find($this->argument('user')));
        }
    }

> {tip} Для кращого повторного використання коду корисно зберігати консольні команди простими та дозволяти їм звертатися до сервісів додатку для виконання своїх завдань. У наведеному нижче прикладі зауважте, що ми впроваджуємо сервісний клас, щоб зробити "важку роботу" надсилання електронних листів.

<a name="closure-commands"></a>

### Команди замикання

Команди на основі замикання - це альтернативний спосіб визначення консольних команд, як класів. Так само, як замикання маршрутів є альтернативою контролерам. Розглядайте команди замикання як альтернативу класам команд. У методі `commands` вашого файлу `app/Console/Kernel.php` Laravel завантажує файл `routes/console.php`:

    /**
     * Register the Closure based commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        require base_path('routes/console.php');
    }

Хоча цей файл не визначає маршрути HTTP, він визначає консоль на основі точок входу (маршрути) у ваш додаток. У цьому файлі ви можете визначити всі замикання на основі консольних команд за допомогою методу `Artisan::command`. Метод `command` приймає два аргументи: [command signature](#defining-input-expectations) та замикання, яке у свою чергу приймає аргументи команди та опцій:

    Artisan::command('mail:send {user}', function ($user) {
        $this->info("Sending email to: {$user}!");
    });

Замикання пов’язане з екземпляром бахової команди. Тому ви маєте повний доступ до всіх допоміжних методів, які можуть бути доступні при використанні класу.

<a name="type-hinting-dependencies"></a>

#### Типізовані (Type-Hinting) залежноості

Крім отримання аргументів і опцій вашої команди, команди замикання можуть також приймати типізовані залежності, які може вирішити сервіс контейнер [service container](/docs/{{version}}/container):

    use App\Models\User;
    use App\Support\DripEmailer;

    Artisan::command('mail:send {user}', function (DripEmailer $drip, $user) {
        $drip->send(User::find($user));
    });

<a name="closure-command-descriptions"></a>

#### Описи команд замикання

Визначаючи команду на основі замикання, ви можете використовувати метод `purpose` щоб додати опис до команди. Цей опис відображатиметься під час запуску команд `php artisan list` або `php artisan help`:

    Artisan::command('mail:send {user}', function ($user) {
        // ...
    })->purpose('Send a marketing email to a user');

<a name="defining-input-expectations"></a>

## Визначення вхідних параметрів

Під час написання консольних команд зазвичай збирають вхідні дані користувача за допомогою аргументів або опцій. Laravel робить дуже зручним визначення вхідних даних, які ви очікуєте від користувача за допомогою властивості `signature` ваших команд. Властивість `signature` дозволяє визначити ім'я, аргументи та опції команди в єдиному виразному синтаксисі, подібному до маршруту.

<a name="arguments"></a>

### Аргументи

Усі надані користувачем аргументи та опції обертаються фігурними дужками. У наступному прикладі команда визначає один обов'язковий аргумент: `user`:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

Ви також можете зробити аргументи необов'язковими та визначити значення за замовчуванням для аргументів:

    // Optional argument...
    email:send {user?}

    // Optional argument with default value...
    email:send {user=foo}

<a name="options"></a>

### Опції

Опції, як і аргументи, є іншою формою введення данних користувачем. Опції мають префікс двох дефісів (`--`), коли вони вказані в командному рядку. Існує два типи опцій: ті, що отримують значення, і ті, які не отримують. Опції, які не отримують значення, служать логічним "перемикачем". Давайте подивимось на приклад цього типу опцій:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';

У цьому прикладі перемикач `--queue` можна вказати при виклику команди Artisan. Якщо перемикач `--queue` передано, значення опції буде `true`. В іншому випадку, значення буде `false`:

    php artisan email:send 1 --queue

<a name="options-with-values"></a>

#### Опції зі значеннями

Далі, давайте розглянемо опцію, яка очікує значення. Якщо користувач повинен вказати значення для опції, додайте суфікс `=` до назви опції.

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';

У цьому прикладі користувач може передати значення для опції таким чином. Якщо опция не вказана при викликанні команди, її значення буде `null`:

    php artisan email:send 1 --queue=default

Ви можете присвоїти значення за замовчуванням для опцій, вказавши значення за замовчуванням після назви опції. Якщо користувач не передає значення опції, буде використано значення за замовчуванням:

     mail:send {user} {--queue=default}

<a name="option-shortcuts"></a>

#### Скорочення опцій

Щоб скоротити опцію при її визначені, ви можете вказати скорочення перед назвою опції та використовувати роздільний знак `|`, щоб відокремити скорочення від повної назви опції:

    mail:send {user} {--Q|queue}

<a name="input-arrays"></a>

### Вхідні масиви

Якщо ви хочете визначити аргументи або опції, які очикують кілька вхідних значень, ви можете використовувати символ `*`. Спочатку розглянемо приклад, який визначає аргумент масиву:

    mail:send {user*}

При виклику цього методу аргументи `user` можуть передаватися послідовно в командний рядок. Наприклад, наступна команда встановить значення `user` у масив з його значеннями `foo` та `bar`:

    php artisan mail:send foo bar

<a name="option-arrays"></a>

#### Масиви опцій

При визначенні опції, яка очікує кілька вхідних значень, кожне значення опціїї, передане команді, має мати префікс з ім'ям опції:

    mail:send {user} {--id=*}

    php artisan mail:send --id=1 --id=2

<a name="input-descriptions"></a>

### Вхідні описи

Ви можете призначити описи вхідним аргументам і опціям, відокремивши  ім'я аргументу від опису двокрапкою. Якщо вам потрібно трохи додаткового місця для визначення вашої команди, сміливо розподіляйте визначення по кількох рядках:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'mail:send
                            {user : The ID of the user}
                            {--queue= : Whether the job should be queued}';

<a name="command-io"></a>

## Введення / виведення

<a name="retrieving-input"></a>

### Отримання даних вводу

Під час виконання команди, вам, ймовірно, потрібно буде отримати доступ до значень аргументів та опцій, прийнятих вашою командою. Для цього ви можете використовувати методи `argument` та `option`. Якщо якогось аргумента чи опції не існує, ви отримаєте `null`:

    /**
     * Execute the console command.
     *
     * @return int
     */
    public function handle()
    {
        $userId = $this->argument('user');

        //
    }

Якщо вам потрібно отримати всі аргументи я к`array`, викличте метод `arguments`:

    $arguments = $this->arguments();

Опції можна отримати так само легко, як і аргументи, використовуючи метод `option`. Щоб отримати всі опції як масив, викличте метод `options`:

    // Retrieve a specific option...
    $queueName = $this->option('queue');

    // Retrieve all options as an array...
    $options = $this->options();

<a name="prompting-for-input"></a>

### Запрошення на введення (Prompting For Input)

Окрім відображення вихідних даних, ви також можете попросити користувача ввести дані під час виконання вашої команди. Метод `ask` викличе діалог, прийме введення від користувача, а потім поверне введення користувача назад до вашої команди:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('What is your name?');
    }

Метод `secret` подібний до `ask`, але введені користувачем дані не будуть видимі для них під час введення. Цей метод корисний при запиті на конфіденційну інформацію, таку як пароль:

    $password = $this->secret('What is the password?');

<a name="asking-for-confirmation"></a>

#### Запит підтвердження

Якщо вам потрібно попросити у користувача просте підтвердження "так чи ні", ви можете використовувати метод `confirm`. За замовчуванням цей метод поверне `false`. Однак, якщо користувач входить `y` або `yes` у відповідь на запит, то метод поверне `true`.

    if ($this->confirm('Do you wish to continue?')) {
        //
    }

При потребі, ви можете вказати, що запит підтвердження повинен повертати `true` за замовчуванням, передавши `true` як другий аргумент в метод `confirm`:

    if ($this->confirm('Do you wish to continue?', true)) {
        //
    }

<a name="auto-completion"></a>

#### Автозаповнення

Метод `anticipate` може бути використаний для забезпечення автоматичного заповнення можливих варіантів. Користувач все ще може вибрати будь-яку відповідь, незалежно від підказок автоматичного заповнення:

$name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

Крім того, ви можете передати замикання як другий аргумент методу `anticipate`. Замикання буде викликатися кожного разу, коли користувач набирає введений символ. Замикання повинно приймати параметр рядка, що містить введені користувачем дані, і повертати масив параметрів для автозавершення:

    $name = $this->anticipate('What is your address?', function ($input) {
        // Return auto-completion options...
    });

<a name="multiple-choice-questions"></a>

#### Запитання з декількома варіантами вибору

Якщо вам потрібно надати користувачеві заздалегідь визначений набір варіантів, ви можете використовувати метод `choice`. Ви можете встановити індекс масиву значення за замовчуванням, яке повертається, якщо не вибрано жодної опції:

    $name = $this->choice(
        'What is your name?',
        ['Taylor', 'Dayle'],
        $defaultIndex
    );

Крім того, метод `choice` приймає необов’язкові четвертий та п’ятий аргументи для визначення максимальної кількості спроб вибрати дійсну відповідь та чи дозволено кілька виділень:

    $name = $this->choice(
        'What is your name?',
        ['Taylor', 'Dayle'],
        $defaultIndex,
        $maxAttempts = null,
        $allowMultipleSelections = false
    );

<a name="writing-output"></a>

### Вихідні дані

Щоб надіслати вихід на консоль, використовуйте методи `line`,`info`,`comment`,`question`, `error`. Кожен із цих методів використовуватиме відповідні кольори ANSI для свого призначення. Наприклад, давайте відобразимо загальну інформацію для користувача. Як правило, метод `info` відображатиметься в консолі у вигляді зеленого тексту:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        // ...

        $this->info('The command was successful!');
    }

Щоб відобразити повідомлення про помилку, використовуйте метод `error`. Текст повідомлення про помилку зазвичай відображається червоним:

    $this->error('Something went wrong!');

Якщо ви хочете відобразити простий, незабарвлений текст, використовуйте метод `line`:

    $this->line('Display this on the screen');

Ви можете використовувати метод `newLine` для відображення порожнього рядка:

    $this->newLine();

    // Write three blank lines...
    $this->newLine(3);

<a name="table-layouts"></a>

#### Макети таблиць

Метод `table` полегшує правильне форматування декількох рядків / стовпців даних. Все, що вам потрібно зробити, це вказати імена стовпців та дані для таблиці. Laravel автоматично обчислитить відповідну для вас ширину та висоту таблиці:

    use App\Models\User;

    $this->table(
        ['Name', 'Email'],
        User::all(['name', 'email'])->toArray()
    );

<a name="progress-bars"></a>

#### Прогрес бари

Для довготривалих завдань може бути корисно показати індикатор прогресу. Використовуючи метод `withProgressBar`, Laravel відобразить прогрес бар і буде просувати прогрес для кожної ітерації за заданим значенням, яке можна повторити:

    use App\Models\User;

    $users = $this->withProgressBar(User::all(), function ($user) {
        $this->performTask($user);
    });

Іноді вам може знадобитися більше ручного контролю над тим, як просувається індикатор прогресу. По-перше, визначте загальну кількість кроків, яку процес буде повторювати. Потім просувайте індикатор виконання після обробки кожного елемента:

    $users = App\Models\User::all();

    $bar = $this->output->createProgressBar(count($users));

    $bar->start();

    foreach ($users as $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

> {tip} Щоб отримати додаткові параметри, перегляньте[Symfony Progress Bar component documentation](https://symfony.com/doc/current/components/console/helpers/progressbar.html).

<a name="registering-commands"></a>

## Реєстрація команд

Усі ваші консольні команди реєструються в класі `App\Console\Kernel` вашого застосунку, який є вашим "консольним ядром". У методі `command` цього класу ви побачите виклик методу ядра `load`. Метод `load` буде сканувати директорію `app/Console/Commands` і автоматично реєструвати кожну команду, що вона містить, у Artisan. Ви навіть можете робити додаткові виклики методу `load` для сканування інших каталогів на наявність команд Artisan:


/**
     * Register the commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');
        $this->load(__DIR__.'/../Domain/Orders/Commands');

        // ...
    }

Ви також можете вручну реєструвати команди, додаючи назву класу до властивості `$commands` вашого класу `App\Console\Kernel`. Коли Artisan завантажиться, усі команди, перелічені в цій властивості, будуть вирішені за допомогою [service container](/docs/{{version}}/container) та зареєстровані у Artisan:

    protected $commands = [
        Commands\SendEmails::class
    ];

<a name="programmatically-executing-commands"></a>

## Програмне виконання команд

Іноді, можливо, ви захочете виконати команду Artisan поза CLI. Наприклад, ви можете запустити команду Artisan з маршруту або контролера. Ви можете використовувати метод `call` фасаду `Artisan` для досягнення цього. Метод `call` приймає як ім'я сігнатури класу або клас команди як перший аргумент, так і масив параметрів команди як другий аргумент. Буде повернено код виходу:

    use Illuminate\Support\Facades\Artisan;

    Route::post('/user/{user}/mail', function ($user) {
        $exitCode = Artisan::call('mail:send', [
            'user' => $user, '--queue' => 'default'
        ]);

        //
    });

Альтернативно, ви можете передати всю команду Artisan в метод `call` як рядок:

    Artisan::call('mail:send 1 --queue=default');

<a name="passing-array-values"></a>

#### Передавання значень масиву

Якщо ваша команда визначає опцію, яка приймає масив, ви можете передати масив значень цьому параметру:

    use Illuminate\Support\Facades\Artisan;

    Route::post('/mail', function () {
        $exitCode = Artisan::call('mail:send', [
            '--id' => [5, 13]
        ]);
    });

<a name="passing-boolean-values"></a>

#### Передача булевих значень

Якщо вам потрібно вказати значення параметра, який не приймає рядкові значення, наприклад, прапорець `--force` у команді `migrate:refresh`, ви повинні передати значення `true` або `false`:

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

<a name="queueing-artisan-commands"></a>

#### Черги Artisan команд

Використовуючи метод `queue` фасаду `Artisan` ви навіть можете поставити в чергу команди Artisan, щоб вони оброблялися у фоновому режимі через [queue workers](/docs/{{version}}/queues). Перед використанням цього методу переконайтесь, що ви сконфігурували вашу чергу та запускаете слухача черги: 

    use Illuminate\Support\Facades\Artisan;

    Route::post('/user/{user}/mail', function ($user) {
        Artisan::queue('mail:send', [
            'user' => $user, '--queue' => 'default'
        ]);

        //
    });

Використовуючи методи `onConnection` та `onQueue`, ви можете вказати з'єднання або чергу, до якої Artisan команда повинна ссилатися:

    Artisan::queue('mail:send', [
        'user' => 1, '--queue' => 'default'
    ])->onConnection('redis')->onQueue('commands');

<a name="calling-commands-from-other-commands"></a>

### Виклик команд з інших команд

Іноді вам може знадобитися викликати інші команди з існуючої команди Artisan. Ви можете зробити це за допомогою методу `call`. Метод `call` приймає ім'я команди та масив аргументів/опцій команди:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->call('mail:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    }

Якщо ви хочете викликати іншу консольну команду та заблокувати всі її результати, ви можете використовувати метод `callSilent`. Метод `callSilent` має ту самц сігнатуру, що і метод `call`:

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

<a name="stub-customization"></a>

## Налаштування заглушки

Консольні команди Artisan `make` використовуються для створення різноманітних класів, таких як контролери, завдання, міграції та тести. Ці класи генеруються за допомогою  файлів "заглушок", які заповнюються значеннями на основі вашого вводу. Однак, вам може знадобитися внести невеликі зміни у Artisan файли. Для цього ви можете використовувати команду `stub:publish` для публікації найпоширеніших заглушок вашого застосунку для їх кастомізації:

    php artisan stub:publish

Опубліковані заглушки будуть розташовані в межах кореневої директрії `stubs`. Будь-які зміни, які ви внесете до цих заглушок, будуть відображені, коли ви створюєте відповідні класи за допомогою Artisan команди `make`.

<a name="events"></a>

## Події

Artisan відправляє три події під час виконання команд: `Illuminate\Console\Events\ArtisanStarting`,`Illuminate\Console\Events\CommandStarting`, і `Illuminate\Console\Events\CommandFinished`. Подія `ArtisanStarting` надсилається негайно, як тільки Artisan починає працювати. Далі безпосередньо перед запуском команди відправляється подія `CommandStarting`. Нарешті, відправляється подія `CommandFinished`, як тільки команда закінчує виконуватись.
