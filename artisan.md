git 6823f659ab97ee3144a632d440569cb0ccd85a45

---

# Консоль Artisan

-   [Вступ](#introduction)
    -   [Tinker (REPL)](#tinker)
-   [Написання команд](#writing-commands)
    -   [Генерація команд](#generating-commands)
    -   [Структура команд](#command-structure)
    -   [Команди Closure](#closure-commands)
-   [Визначення вхідних очікувань](#defining-input-expectations)
    -   [Аргументи](#arguments)
    -   [Варіанти](#options)
    -   [Вхідні масиви](#input-arrays)
    -   [Вхідні описи](#input-descriptions)
-   [Командне введення / виведення](#command-io)
    -   [Отримання вхідних даних](#retrieving-input)
    -   [Прохання ввести дані](#prompting-for-input)
    -   [Вивід даних](#writing-output)
-   [Реєстрація команд](#registering-commands)
-   [Програмне виконання команд](#programmatically-executing-commands)
    -   [Виклик команд з інших команд](#calling-commands-from-other-commands)
-   [Налаштування заглушок](#stub-customization)
-   [Події](#events)

<a name="introduction"></a>

## Вступ

Artisan - це інтерфейс командного рядка, що входить до складу Laravel. Він надає ряд корисних команд, які можуть допомогти вам під час створення вашого сайту. Щоб переглянути список усіх доступних команд Artisan, ви можете використовувати`list`команду:

    php artisan list

Кожна команда також включає "довідку", яка відображає та описує доступні аргументи та параметри команди. Щоб переглянути екран довідки, перед назвою команди натисніть`help`:

    php artisan help migrate

<a name="tinker"></a>

### Тинкер (REPL)

Laravel Tinker - це потужний REPL для фреймворку Laravel, який працює на[PsySH](https://github.com/bobthecow/psysh)пакеті.

<a name="installation"></a>

#### Встановлення

Усі програми Laravel включають Tinker за замовчуванням. Однак ви можете встановити його вручну, якщо потрібно, використовуючи Composer:

    composer require laravel/tinker

> {tip} Шукаєте графічний інтерфейс для взаємодії з вашим додатком Laravel? Використайте[Тінкервелл](https://tinkerwell.app)!

<a name="usage"></a>

#### Використання

Tinker дозволяє взаємодіяти з усім вашим додатком Laravel у командному рядку, включаючи Eloquent ORM, jobs, events тощо. Щоб увійти в середовище Tinker, запустіть`tinker`Команду Artisan:

    php artisan tinker

Ви можете опублікувати файл конфігурації Tinker за допомогою`vendor:publish`команда:

    php artisan vendor:publish --provider="Laravel\Tinker\TinkerServiceProvider"

> {примітка}`dispatch`допоміжна функція та`dispatch`метод в`Dispatchable`клас залежить від garbage collection, щоб розмістити завдання в черзі. Тому, використовуючи tinker, слід використовувати`Bus::dispatch`або`Queue::push` щоб виконати dispatch jobs.

<a name="command-whitelist"></a>

#### Білий список команд

Tinker використовує білий список, щоб визначити, які команди Artisan дозволяється запускати в його оболонці. За замовчуванням ви можете запустити`clear-compiled`,`down`,`env`,`inspire`,`migrate`,`optimize`, і`up`команди. Якщо ви хочете додати до білого списку більше команд, ви можете додати їх до`commands`масив у вашому`tinker.php`файл конфігурації:

    'commands' => [
        // App\Console\Commands\ExampleCommand::class,
    ],

<a name="classes-that-should-not-be-aliased"></a>

#### Класи, які не мають бути "Aliased"

Як правило, Tinker автоматично псевдонімить(aliases) класи, коли вони потрібні в Tinker. Тим не менш, ви можете заборонити деякі класи. Ви можете досягти цього, перелічивши класи в`dont_alias`масиві вашого`tinker.php`файла конфігурації:

    'dont_alias' => [
        App\Models\User::class,
    ],

<a name="writing-commands"></a>

## Написання команд

На додаток до команд, наданих Artisan, ви також можете створити власні custom команди. Команди зазвичай зберігаються в`app/Console/Commands`каталозі; однак ви можете вибрати власне місце зберігання, доки ваші команди може завантажувати Composer.

<a name="generating-commands"></a>

### Створення команд

Щоб створити нову команду, використовуйте`make:command`Ремісниче командування. Ця команда створить новий клас команд у`app/Console/Commands`каталог. Не хвилюйтеся, якщо цей каталог не існує у вашій програмі, оскільки він буде створений під час першого запуску`make:command`Ремісниче командування. Створена команда включатиме набір властивостей та методів за замовчуванням, які присутні у всіх командах:

    php artisan make:command SendEmails

<a name="command-structure"></a>

### Структура команд

Після генерації команди ви повинні заповнити`signature`і`description`властивості класу, які будуть використовуватися при відображенні вашої команди на`list`екран.`handle`метод буде викликаний при виконанні вашої команди. Ви можете розмістити свою логіку команд у цьому методі.

> {tip} Для кращого повторного використання коду корисно зберігати команди консолі легкими та дозволяти їм переходити до служб додатків для виконання своїх завдань. У наведеному нижче прикладі зауважте, що ми вводимо клас обслуговування, щоб зробити важку роботу під час надсилання електронних листів.

Давайте подивимось на приклад команди. Зверніть увагу, що ми можемо вводити в команду будь-які потрібні нам залежності`handle`метод. Ларавель[службовий контейнер](/docs/{{version}}/container)автоматично введе всі залежності, наведені у підписі цього методу:

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
        protected $signature = 'email:send {user}';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Send drip e-mails to a user';

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

<a name="closure-commands"></a>

### Команди закриття

Команди на основі закриття дають альтернативу визначенню консольних команд як класів. Так само, як замикання маршрутів є альтернативою контролерам, подумайте про командні замикання як про альтернативу класам команд. У межах`commands`метод вашого`app/Console/Kernel.php`файл, Laravel завантажує файл`routes/console.php`файл:

    /**
     * Register the Closure based commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        require base_path('routes/console.php');
    }

Хоча цей файл не визначає маршрути HTTP, він визначає точки входу (маршрути) на основі консолі у вашу програму. У цьому файлі ви можете визначити всі маршрути, засновані на закритті, за допомогою`Artisan::command`метод.`command`метод приймає два аргументи:[підпис команди](#defining-input-expectations)та Закриття, яке отримує аргументи команд та параметри:

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    });

Закриття прив’язане до базового екземпляра команди, тому ви маєте повний доступ до всіх допоміжних методів, до яких ви, як правило, зможете отримати доступ до повного класу команд.

<a name="type-hinting-dependencies"></a>

#### Залежності підказки типу

На додаток до отримання аргументів і параметрів вашої команди, Закриття команд може також натякати на додаткові залежності, які ви хотіли б вирішити з[службовий контейнер](/docs/{{version}}/container):

    use App\Models\User;
    use App\Support\DripEmailer;

    Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
        $drip->send(User::find($user));
    });

<a name="closure-command-descriptions"></a>

#### Описи команд закриття

Визначаючи команду на основі закриття, ви можете використовувати`describe`метод, щоб додати опис до команди. Цей опис відображатиметься під час запуску`php artisan list`або`php artisan help`команди:

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    })->describe('Build the project');

<a name="defining-input-expectations"></a>

## Визначення споживчих очікувань

Під час написання консольних команд звичайно збирати вхідні дані користувача за допомогою аргументів або параметрів. Laravel робить дуже зручним визначення вхідних даних, які ви очікуєте від користувача за допомогою`signature`властивість ваших команд.`signature`властивість дозволяє визначити ім'я, аргументи та параметри команди в єдиному, виразному синтаксисі, подібному до маршруту.

<a name="arguments"></a>

### Аргументи

Усі надані користувачем аргументи та параметри обертаються фігурними дужками. У наступному прикладі команда визначає один**вимагається**аргумент:`user`:

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

### Варіанти

Параметри, як і аргументи, є іншою формою введення користувачем. Параметри мають префікс двох дефісів (`--`), коли вони вказані в командному рядку. Існує два типи варіантів: ті, що отримують цінність, і ті, які не отримують. Параметри, які не отримують значення, служать логічним "перемикачем". Давайте подивимось на приклад цього типу опціону:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';

У цьому прикладі`--queue`switch можна вказати при виклику команди Artisan. Якщо`--queue`перемикач передано, значення опції буде`true`. В іншому випадку значення буде`false`:

    php artisan email:send 1 --queue

<a name="options-with-values"></a>

#### Параметри зі значеннями

Далі, давайте розглянемо варіант, який очікує значення. Якщо користувач повинен вказати значення для параметра, додайте ім'я параметра до символу`=`знак:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';

У цьому прикладі користувач може передати значення для параметра таким чином:

    php artisan email:send 1 --queue=default

Ви можете присвоїти значенням параметрів за замовчуванням, вказавши значення за замовчуванням після імені параметра. Якщо користувач не передає значення опції, буде використано значення за замовчуванням:

    email:send {user} {--queue=default}

<a name="option-shortcuts"></a>

#### Комбінації клавіш

Щоб призначити ярлик при визначенні параметра, ви можете вказати його перед назвою параметра та використовувати | роздільник, щоб відокремити ярлик від повного імені опції:

    email:send {user} {--Q|queue}

<a name="input-arrays"></a>

### Вхідні масиви

Якщо ви хочете визначити аргументи або параметри, щоб очікувати введення масиву, ви можете використовувати`*`характер. Спочатку розглянемо приклад, який визначає аргумент масиву:

    email:send {user*}

При виклику цього методу`user`аргументи можуть передаватися в командний рядок. Наприклад, наступна команда встановить значення`user`до`['foo', 'bar']`:

    php artisan email:send foo bar

При визначенні параметра, який очікує введення масиву, кожне значення параметра, передане команді, має мати префікс з ім'ям параметра:

    email:send {user} {--id=*}

    php artisan email:send --id=1 --id=2

<a name="input-descriptions"></a>

### Вхідні описи

Ви можете призначити описи вхідним аргументам і параметрам, відокремивши параметр від опису двокрапкою. Якщо вам потрібно трохи додаткового місця для визначення вашої команди, сміливо розподіляйте визначення по кількох рядках:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send
                            {user : The ID of the user}
                            {--queue= : Whether the job should be queued}';

<a name="command-io"></a>

## Командне введення / виведення

<a name="retrieving-input"></a>

### Отримання вводу

Поки ваша команда виконується, вам, очевидно, потрібно буде отримати доступ до значень аргументів та параметрів, прийнятих вашою командою. Для цього ви можете використовувати`argument`і`option`методи:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $userId = $this->argument('user');

        //
    }

Якщо вам потрібно отримати всі аргументи як`array`, зателефонуйте на`arguments`метод:

    $arguments = $this->arguments();

Параметри можна отримати так само легко, як і аргументи, використовуючи`option`метод. Щоб отримати всі опції як масив, зателефонуйте на`options`метод:

    // Retrieve a specific option...
    $queueName = $this->option('queue');

    // Retrieve all options...
    $options = $this->options();

Якщо аргумент або параметр не існує,`null`буде повернено.

<a name="prompting-for-input"></a>

### Запрошення на введення

Окрім відображення вихідних даних, ви також можете попросити користувача надати введення під час виконання вашої команди.`ask`метод запропонує користувачеві задане питання, прийме їх введення, а потім поверне введення користувача назад до вашої команди:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('What is your name?');
    }

`secret`метод подібний до`ask`, але введені користувачем дані не будуть видимі для них під час введення тексту в консолі. Цей метод корисний при запиті на конфіденційну інформацію, таку як пароль:

    $password = $this->secret('What is the password?');

<a name="asking-for-confirmation"></a>

#### Прохання про підтвердження

Якщо вам потрібно попросити у користувача просте підтвердження, ви можете використовувати`confirm`метод. За замовчуванням цей метод повернеться`false`. Однак, якщо користувач входить`y`або`yes`у відповідь на запит метод повернеться`true`.

    if ($this->confirm('Do you wish to continue?')) {
        //
    }

<a name="auto-completion"></a>

#### Автозавершення

`anticipate`метод може бути використаний для забезпечення автоматичного заповнення для можливого вибору. Користувач все ще може вибрати будь-яку відповідь, незалежно від підказок автоматичного заповнення:

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

Крім того, ви можете передати Закриття як другий аргумент`anticipate`метод. Закриття буде викликатися кожного разу, коли користувач набирає введений символ. Закриття повинно приймати параметр рядка, що містить введені користувачем дані, і повертати масив параметрів для автозавершення:

    $name = $this->anticipate('What is your name?', function ($input) {
        // Return auto-completion options...
    });

<a name="multiple-choice-questions"></a>

#### Запитання з декількома варіантами вибору

Якщо вам потрібно надати користувачеві заздалегідь визначений набір варіантів, ви можете використовувати`choice`метод. Ви можете встановити індекс масиву значення за замовчуванням, яке повертається, якщо не вибрано жодної опції:

    $name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $defaultIndex);

Крім того,`choice`метод приймає необов’язкові четвертий та п’ятий аргументи для визначення максимальної кількості спроб вибрати дійсну відповідь та чи дозволено кілька виділень:

    $name = $this->choice(
        'What is your name?',
        ['Taylor', 'Dayle'],
        $defaultIndex,
        $maxAttempts = null,
        $allowMultipleSelections = false
    );

<a name="writing-output"></a>

### Вихідні дані

Щоб надіслати вихід на консоль, використовуйте`line`,`info`,`comment`,`question`і`error`методи. Кожен із цих методів використовуватиме відповідні кольори ANSI для свого призначення. Наприклад, давайте відобразимо загальну інформацію для користувача. Як правило,`info`метод відображатиметься в консолі у вигляді зеленого тексту:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('Display this on the screen');
    }

Щоб відобразити повідомлення про помилку, використовуйте`error`метод. Текст повідомлення про помилку зазвичай відображається червоним:

    $this->error('Something went wrong!');

Якщо ви хочете відобразити простий, кольоровий вихід консолі, використовуйте`line`метод:

    $this->line('Display this on the screen');

Ви можете використовувати`newLine`спосіб відображення порожнього рядка:

    $this->newLine();

    // Write three blank lines...
    $this->newLine(3);

<a name="table-layouts"></a>

#### Макети таблиць

`table`метод полегшує правильне форматування декількох рядків / стовпців даних. Просто передайте в заголовки та рядки метод. Ширина та висота будуть динамічно обчислюватися на основі даних:

    $headers = ['Name', 'Email'];

    $users = App\Models\User::all(['name', 'email'])->toArray();

    $this->table($headers, $users);

<a name="progress-bars"></a>

#### Бари прогресу

Для довготривалих завдань може бути корисно показати індикатор прогресу. Використовуючи вихідний об'єкт, ми можемо запускати, просувати і зупиняти індикатор прогресу. По-перше, визначте загальну кількість кроків, які процес буде повторювати. Потім просуньте індикатор виконання після обробки кожного елемента:

    $users = App\Models\User::all();

    $bar = $this->output->createProgressBar(count($users));

    $bar->start();

    foreach ($users as $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

Щоб отримати додаткові параметри, перегляньте[Документація компонента Symfony Progress Bar](https://symfony.com/doc/current/components/console/helpers/progressbar.html).

<a name="registering-commands"></a>

## Реєстрація команд

Через`load`виклику методу у вашому консольному ядрі`commands`метод, усі команди в`app/Console/Commands`каталог буде автоматично зареєстрований у Artisan. Насправді ви можете безкоштовно телефонувати до`load`метод сканування інших каталогів на наявність команд Artisan:

    /**
     * Register the commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');
        $this->load(__DIR__.'/MoreCommands');

        // ...
    }

Ви також можете вручну реєструвати команди, додаючи назву класу до`$commands`власність вашого`app/Console/Kernel.php`файл. Коли Artisan завантажиться, усі команди, перелічені в цьому властивості, будуть вирішені[службовий контейнер](/docs/{{version}}/container)та зареєстрований у Artisan:

    protected $commands = [
        Commands\SendEmails::class
    ];

<a name="programmatically-executing-commands"></a>

## Програмне виконання команд

Іноді, можливо, ви захочете виконати команду Artisan поза CLI. Наприклад, ви можете запустити команду Artisan з маршруту або контролера. Ви можете використовувати`call`метод на`Artisan`фасад для досягнення цього.`call`метод приймає як ім'я або клас команди як перший аргумент, так і масив параметрів команди як другий аргумент. Буде повернено код виходу:

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

Крім того, ви можете передати всю команду Artisan команді`call`метод як рядок:

    Artisan::call('email:send 1 --queue=default');

Використання`queue`метод на`Artisan`фасаду, ви навіть можете поставити в чергу команди Artisan, щоб вони оброблялися у фоновому режимі вашими[чергові працівники](/docs/{{version}}/queues). Перш ніж використовувати цей метод, переконайтеся, що ви налаштували свою чергу і запускаєте прослуховувач черги:

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

Ви також можете вказати підключення або чергу, куди слід надіслати команду Artisan:

    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ])->onConnection('redis')->onQueue('commands');

<a name="passing-array-values"></a>

#### Передавання значень масиву

Якщо ваша команда визначає опцію, яка приймає масив, ви можете передати масив значень цьому параметру:

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--id' => [5, 13]
        ]);
    });

<a name="passing-boolean-values"></a>

#### Передача булевих значень

Якщо вам потрібно вказати значення параметра, який не приймає рядкові значення, наприклад`--force`прапор на`migrate:refresh`команду, ви повинні пройти`true`або`false`:

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

<a name="calling-commands-from-other-commands"></a>

### Виклик команд з інших команд

Іноді вам може знадобитися викликати інші команди з існуючої команди Artisan. Ви можете зробити це за допомогою`call`метод. Це`call`метод приймає ім'я команди та масив параметрів команди:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    }

Якщо ви хочете викликати іншу команду консолі та придушити всі її результати, ви можете використовувати`callSilent`метод.`callSilent`метод має той самий підпис, що і`call`метод:

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

<a name="stub-customization"></a>

## Налаштування заглушки

Консоль Artisan`make`команди використовуються для створення різноманітних класів, таких як контролери, завдання, міграції та тести. Ці класи генеруються за допомогою "заглушених" файлів, які заповнюються значеннями на основі вашого вводу. Однак іноді вам може знадобитися внести невеликі зміни у файли, створені Artisan. Для цього ви можете використовувати`stub:publish`команда для публікації найпоширеніших заглушок для налаштування:

    php artisan stub:publish

Опубліковані заглушки будуть розташовані в межах`stubs`в кореневій частині вашої програми. Будь-які зміни, які ви внесете до цих заглушок, будуть відображені, коли ви створюєте відповідні класи за допомогою Artisan`make`команди.

<a name="events"></a>

## Події

Artisan відправляє три події під час виконання команд:`Illuminate\Console\Events\ArtisanStarting`,`Illuminate\Console\Events\CommandStarting`, і`Illuminate\Console\Events\CommandFinished`.`ArtisanStarting`подія надсилається негайно, коли Artisan починає працювати. Далі,`CommandStarting`подія відправляється безпосередньо перед запуском команди. Нарешті,`CommandFinished`подія відправляється, як тільки команда закінчує виконуватись.
