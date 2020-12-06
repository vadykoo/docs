# Черги

-   [Вступ](#introduction)
    -   [Зв’язки проти Черги](#connections-vs-queues)
    -   [Примітки та вимоги до драйвера](#driver-prerequisites)
-   [Створення робочих місць](#creating-jobs)
    -   [Створення класів робочих місць](#generating-job-classes)
    -   [Структура класу](#class-structure)
    -   [Унікальні робочі місця](#unique-jobs)
-   [Робота проміжного програмного забезпечення](#job-middleware)
    -   [Обмеження ставки](#rate-limiting)
    -   [Запобігання перекриванню робочих місць](#preventing-job-overlaps)
-   [Диспетчерські роботи](#dispatching-jobs)
    -   [Затримка відправки](#delayed-dispatching)
    -   [Синхронна диспетчеризація](#synchronous-dispatching)
    -   [Підключення робочих місць](#job-chaining)
    -   [Налаштування черги та підключення](#customizing-the-queue-and-connection)
    -   [Вказівка ​​максимальних спроб роботи / значень часу очікування](#max-job-attempts-and-timeout)
    -   [Обробка помилок](#error-handling)
-   [Розміщення робочих місць](#job-batching)
    -   [Визначення сумісних робочих місць](#defining-batchable-jobs)
    -   [Диспетчерські партії](#dispatching-batches)
    -   [Додавання робочих місць до пакетів](#adding-jobs-to-batches)
    -   [Огляд партій](#inspecting-batches)
    -   [Скасування пакетів](#cancelling-batches)
    -   [Пакетні відмови](#batch-failures)
-   [Закриття в черзі](#queueing-closures)
-   [Запуск програми черги](#running-the-queue-worker)
    -   [Пріоритети черги](#queue-priorities)
    -   [Робітники черги та розгортання](#queue-workers-and-deployment)
    -   [Термін дії та час очікування](#job-expirations-and-timeouts)
-   [Конфігурація супервізора](#supervisor-configuration)
-   [Робота з невдалими робочими місцями](#dealing-with-failed-jobs)
    -   [Прибирання після невдалих робіт](#cleaning-up-after-failed-jobs)
    -   [Невдалі події роботи](#failed-job-events)
    -   [Повторна спроба невдалих завдань](#retrying-failed-jobs)
    -   [Ігнорування відсутніх моделей](#ignoring-missing-models)
-   [Очищення робочих місць із черг](#clearing-jobs-from-queues)
-   [Події роботи](#job-events)

<a name="introduction"></a>

## Вступ

> {tip} Laravel тепер пропонує Horizon, чудову панель інструментів та систему конфігурації для ваших черг із підтримкою Redis. Перевірте повне[Документація горизонту](/docs/{{version}}/horizon)для отримання додаткової інформації.

Черги Laravel забезпечують уніфікований API для різноманітних серверних систем, таких як Beanstalk, Amazon SQS, Redis або навіть реляційна база даних. Черги дозволяють відкласти обробку трудомісткого завдання, наприклад, надсилання електронного листа, на пізніший час. Відкладення цих трудомістких завдань різко пришвидшує веб-запити до вашої програми.

Файл конфігурації черги зберігається в`config/queue.php`. У цьому файлі ви знайдете конфігурації підключення для кожного з драйверів черги, що входять до фреймворку, що включає базу даних,[Beanstalkd](https://beanstalkd.github.io/),[Amazon SQS](https://aws.amazon.com/sqs/),[Редіс](https://redis.io)та синхронний драйвер, який негайно виконуватиме завдання (для локального використання). A`null`також включений драйвер черги, який відкидає завдання, що стоять у черзі.

<a name="connections-vs-queues"></a>

### Зв’язки проти Черги

Перш ніж розпочати роботу з чергами Laravel, важливо зрозуміти різницю між "з'єднаннями" та "чергами". У вашому`config/queue.php`файл конфігурації, є файл`connections`варіант конфігурації. Цей параметр визначає конкретне підключення до серверної служби, такої як Amazon SQS, Beanstalk або Redis. Однак будь-яке дане підключення до черги може мати кілька "черг", які можна розглядати як різні стеки або купи завдань, що стоять у черзі.

Зверніть увагу, що кожен приклад конфігурації з'єднання в`queue`Файл конфігурації містить файл`queue`атрибут. Це черга за замовчуванням, куди будуть відправлені завдання, коли вони будуть надіслані на певне з'єднання. Іншими словами, якщо ви відправляєте роботу, не чітко визначивши, в яку чергу вона повинна бути відправлена, завдання буде розміщено в черзі, яка визначена в`queue`атрибут конфігурації з'єднання:

    // This job is sent to the default queue...
    Job::dispatch();

    // This job is sent to the "emails" queue...
    Job::dispatch()->onQueue('emails');

Деяким програмам може не знадобитися коли-небудь виштовхувати завдання в декілька черг, натомість воліючи мати одну просту чергу. Однак надсилання завдань до декількох черг може бути особливо корисним для програм, які хочуть визначити пріоритети або сегментувати обробку завдань, оскільки робочий черги Laravel дозволяє вказати, які черги він повинен обробляти за пріоритетом. Наприклад, якщо ви натискаєте завдання на`high`черзі, ви можете запустити працівника, який надає їм вищий пріоритет обробки:

    php artisan queue:work --queue=high,default

<a name="driver-prerequisites"></a>

### Примітки та вимоги до драйвера

<a name="database"></a>

#### База даних

Для того, щоб використовувати`database`драйвера черги, вам знадобиться таблиця бази даних для зберігання завдань. Щоб створити міграцію, яка створює цю таблицю, запустіть`queue:table`artisan командування. Після створення міграції ви можете перенести базу даних за допомогою`migrate`команда:

    php artisan queue:table

    php artisan migrate

<a name="redis"></a>

#### Редіс

Для того, щоб використовувати`redis`драйвера черги, вам слід налаштувати підключення до бази даних Redis у вашому`config/database.php`файл конфігурації.

**Кластер Редіс**

Якщо підключення до черги Redis використовує кластер Redis, імена черг повинні містити[хеш-тег ключа](https://redis.io/topics/cluster-spec#keys-hash-tags). Це потрібно для того, щоб усі ключі Redis для даної черги були розміщені в одному хеш-слоті:

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => '{default}',
        'retry_after' => 90,
    ],

**Блокування**

При використанні черги Redis ви можете використовувати`block_for`параметр конфігурації, щоб вказати, скільки часу драйвер повинен чекати, поки завдання стане доступним, перед ітерацією робочого циклу та повторним опитуванням бази даних Redis.

Налаштування цього значення на основі вашої черги може бути ефективнішим, ніж постійне опитування бази даних Redis для нових завдань. Наприклад, ви можете встановити значення`5`щоб вказати, що водій повинен заблокувати протягом п’яти секунд, очікуючи, поки завдання стане доступним:

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => 'default',
        'retry_after' => 90,
        'block_for' => 5,
    ],

> {note} Налаштування`block_for`до`0`призведе до того, що працівники черги блокуватимуть на невизначений час, поки не з’явиться робота. Це також запобіжить такі сигнали, як`SIGTERM`від обробки до обробки наступного завдання.

<a name="other-driver-prerequisites"></a>

#### Інші передумови драйвера

Для перелічених драйверів черги потрібні такі залежності:

<div class="content-list" markdown="1">
- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~4.0`
- Redis: `predis/predis ~1.0` or phpredis PHP extension
</div>

<a name="creating-jobs"></a>

## Створення робочих місць

<a name="generating-job-classes"></a>

### Створення класів робочих місць

За замовчуванням усі завдання, що знаходяться в черзі для вашої програми, зберігаються в`app/Jobs`каталог. Якщо`app/Jobs`каталог не існує, він буде створений під час запуску`make:job`artisan командування. Ви можете створити нову роботу в черзі за допомогою Artisan CLI:

    php artisan make:job ProcessPodcast

Створений клас реалізує файл`Illuminate\Contracts\Queue\ShouldQueue`інтерфейс, вказуючи Laravel, що завдання має бути перенесено в чергу для асинхронного запуску.

> {tip} Вакансії можуть бути налаштовані за допомогою[заглушка видавництва](/docs/{{version}}/artisan#stub-customization)

<a name="class-structure"></a>

### Структура класу

Класи роботи дуже прості, зазвичай містять лише a`handle`метод, який викликається, коли завдання обробляється чергою. Для початку давайте розглянемо приклад класу роботи. У цьому прикладі ми зробимо вигляд, що керуємо службою публікації подкастів і потребуємо обробки завантажених файлів подкастів до їх публікації:

    <?php

    namespace App\Jobs;

    use App\Models\Podcast;
    use App\Services\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Queue\SerializesModels;

    class ProcessPodcast implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        protected $podcast;

        /**
         * Create a new job instance.
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }

        /**
         * Execute the job.
         *
         * @param  AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // Process uploaded podcast...
        }
    }

У цьому прикладі зауважте, що нам вдалося передати[Красномовна модель](/docs/{{version}}/eloquent)безпосередньо в конструкторі завдання, яке знаходиться в черзі. Через`SerializesModels`риси, яку використовує робота, красномовні моделі та їх навантажені відносини будуть витончено серіалізовані та несеріалізовані під час обробки завдання. Якщо ваше завдання в черзі приймає модель Eloquent у своєму конструкторі, лише ідентифікатор моделі буде серіалізований в черзі. Коли завдання справді обробляється, система черги автоматично повторно отримує повний екземпляр моделі та її завантажені зв’язки з бази даних. Це все абсолютно прозоро для вашої програми та запобігає проблемам, які можуть виникнути при серіалізації повних екземплярів моделі Eloquent.

`handle`метод викликається, коли завдання обробляється чергою. Зверніть увагу, що ми можемо вводити залежності натяку на`handle`метод роботи. Ларавель[службовий контейнер](/docs/{{version}}/container)автоматично вводить ці залежності.

Якщо ви хочете взяти повний контроль над тим, як контейнер вводить залежності в`handle`методом, ви можете використовувати контейнер`bindMethod`метод.`bindMethod`метод приймає зворотний виклик, який отримує завдання та контейнер. В рамках зворотного виклику ви можете вільно викликати`handle`метод, як завгодно. Як правило, вам слід викликати цей метод із[постачальник послуг](/docs/{{version}}/providers):

    use App\Jobs\ProcessPodcast;

    $this->app->bindMethod(ProcessPodcast::class.'@handle', function ($job, $app) {
        return $job->handle($app->make(AudioProcessor::class));
    });

> {note} Двійкові дані, такі як вміст вихідного зображення, повинні передаватися через`base64_encode`функція перед передачею на завдання в черзі. В іншому випадку завдання може неправильно серіалізуватися в JSON при розміщенні в черзі.

<a name="handling-relationships"></a>

#### Обробка відносин

Оскільки завантажені відносини також серіалізуються, серіалізований рядок завдань може стати досить великим. Щоб запобігти серіалізації відносин, ви можете зателефонувати за номером`withoutRelations`метод на моделі при встановленні значення властивості. Цей метод поверне екземпляр моделі без завантажених зв'язків:

    /**
     * Create a new job instance.
     *
     * @param  \App\Models\Podcast  $podcast
     * @return void
     */
    public function __construct(Podcast $podcast)
    {
        $this->podcast = $podcast->withoutRelations();
    }

<a name="unique-jobs"></a>

### Унікальні робочі місця

> {note} Унікальні завдання вимагають драйвера кеш-пам’яті, який підтримує[замки](/docs/{{version}}/cache#atomic-locks).

Іноді вам може знадобитися переконатися, що в черзі в будь-який момент часу знаходиться лише один екземпляр конкретного завдання. Ви можете зробити це, застосувавши`ShouldBeUnique`інтерфейс для вашого робочого класу:

    <?php

    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Contracts\Queue\ShouldBeUnique;

    class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
    {
        ...
    }

У наведеному вище прикладі`UpdateSearchIndex`робота унікальна. Отже, нові відправлення завдання будуть проігноровані, якщо інший екземпляр завдання вже знаходиться в черзі і не закінчив обробку.

У деяких випадках вам може знадобитися визначити конкретний "ключ", який робить завдання унікальним, або ви можете вказати час очікування, після якого робота більше не залишається унікальною. Для цього ви можете визначити`uniqueId`і`uniqueFor`властивості або методи у вашому робочому класі:

    <?php

    use App\Product;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Contracts\Queue\ShouldBeUnique;

    class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
    {
        /**
         * The product instance.
         *
         * @var \App\Product
         */
        public $product;

        /**
        * The number of seconds after which the job's unique lock will be released.
        *
        * @var int
        */
        public $uniqueFor = 3600;

        /**
        * The unique ID of the job.
        *
        * @return string
        */
        public function uniqueId()
        {
            return $this->product->id;
        }
    }

У наведеному вище прикладі`UpdateSearchIndex`Завдання унікальне за ідентифікатором продукту. Отже, будь-які нові відправлення завдання з однаковим ідентифікатором товару будуть ігноруватися, поки не завершиться обробка існуючого завдання. Крім того, якщо існуюче завдання не буде оброблено протягом однієї години, унікальний замок буде звільнений, а інше завдання з тим самим унікальним ключем може бути відправлено в чергу.

<a name="unique-job-locks"></a>

#### Унікальні замки роботи

За лаштунками, коли а`ShouldBeUnique`робота відправлена, Laravel намагається придбати[замок](/docs/{{version}}/cache#atomic-locks)за допомогою`uniqueId`ключ. Якщо замок не отримано, відправка ігнорується. Це блокування звільняється, коли завдання завершує обробку або не вдається виконати всі спроби повторної спроби. За замовчуванням Laravel використовуватиме драйвер кешу за замовчуванням для отримання цього блокування. Однак, якщо ви хочете використовувати інший драйвер для отримання блокування, ви можете визначити a`uniqueVia`метод, який повертає драйвер кешу, який слід використовувати:

    use Illuminate\Support\Facades\Cache;

    class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
    {
        ...

        /**
        * Get the cache driver for the unique job lock.
        *
        * @return \Illuminate\Contracts\Cache\Repository
        */
        public function uniqueVia()
        {
            return Cache::driver('redis');
        }
    }

> {tip} Якщо потрібно лише обмежити одночасну обробку завдання, використовуйте[`WithoutOverlapping`](/docs/{{version}}/queues#preventing-job-overlaps)замість цього Middlware.

<a name="job-middleware"></a>

## Робота проміжного програмного забезпечення

Проміжне програмне забезпечення для роботи дозволяє обернути власну логіку навколо виконання завдань, що знаходяться в черзі, зменшуючи шаблон в самих робочих місцях. Наприклад, розглянемо наступне`handle`метод, який використовує функції обмеження швидкості Redis від Laravel, щоб дозволити обробляти лише одне завдання кожні п’ять секунд:

    /**
     * Execute the job.
     *
     * @return void
     */
    public function handle()
    {
        Redis::throttle('key')->block(0)->allow(1)->every(5)->then(function () {
            info('Lock obtained...');

            // Handle job...
        }, function () {
            // Could not obtain lock...

            return $this->release(5);
        });
    }

Поки цей код дійсний, структура`handle`метод стає галасливим, оскільки він захаращений логікою обмеження швидкості Редіса. Крім того, ця логіка обмеження швидкості повинна бути продубльована для будь-яких інших завдань, які ми хочемо встановити.

Замість обмеження швидкості в методі дескриптора ми могли б визначити Middlware, яке обробляє обмеження швидкості. Laravel не має розташування за замовчуванням для проміжного програмного забезпечення, тому ви можете розмістити Middlware для роботи в будь-якому місці програми. У цьому прикладі ми розмістимо Middlware в`app/Jobs/Middleware`каталог:

    <?php

    namespace App\Jobs\Middleware;

    use Illuminate\Support\Facades\Redis;

    class RateLimited
    {
        /**
         * Process the queued job.
         *
         * @param  mixed  $job
         * @param  callable  $next
         * @return mixed
         */
        public function handle($job, $next)
        {
            Redis::throttle('key')
                    ->block(0)->allow(1)->every(5)
                    ->then(function () use ($job, $next) {
                        // Lock obtained...

                        $next($job);
                    }, function () use ($job) {
                        // Could not obtain lock...

                        $job->release(5);
                    });
        }
    }

Як бачите, подобається[маршрут проміжного програмного забезпечення](/docs/{{version}}/middleware), Middlware роботи отримує оброблювану роботу та зворотний виклик, який слід викликати для продовження обробки завдання.

Після створення проміжного програмного забезпечення, вони можуть бути приєднані до завдання, повернувши їх із робочого місця`middleware`метод. Цей метод не існує на робочих місцях, встановлених платформою`make:job`Команда ремісника, тому вам потрібно буде додати її до власного визначення класу роботи:

    use App\Jobs\Middleware\RateLimited;

    /**
     * Get the middleware the job should pass through.
     *
     * @return array
     */
    public function middleware()
    {
        return [new RateLimited];
    }

<a name="rate-limiting"></a>

### Обмеження ставки

Хоча ми щойно продемонстрували, як написати власне Middlware для обмеження швидкості, Laravel включає Middlware, що обмежує швидкість, яке ви можете використовувати для оцінки обмежених завдань. Люблю[обмежувачі швидкості маршруту](/docs/{{version}}/routing#defining-rate-limiter), обмежувачі вакансій визначаються за допомогою`RateLimiter`фасадні`for`метод.

Наприклад, ви можете дозволити користувачам робити резервні копії своїх даних раз на годину, не встановлюючи такого обмеження для преміум-клієнтів. Для цього ви можете визначити a`RateLimiter`у вашому`AppServiceProvider`:

    use Illuminate\Support\Facades\RateLimiter;

    RateLimiter::for('backups', function ($job) {
        return $job->user->vipCustomer()
                    ? Limit::none()
                    : Limit::perHour(1)->by($job->user->id);
    });

Потім ви можете приєднати обмежувач швидкості до завдання резервного копіювання за допомогою`RateLimited`Middlware. Кожного разу, коли завдання перевищує обмеження швидкості, це Middlware відпускає завдання назад у чергу з відповідною затримкою залежно від тривалості обмеження швидкості.

    use Illuminate\Queue\Middleware\RateLimited;

    /**
     * Get the middleware the job should pass through.
     *
     * @return array
     */
    public function middleware()
    {
        return [new RateLimited('backups')];
    }

Повернення завдання із обмеженою швидкістю до черги все одно збільшить загальну кількість завдання`attempts`. Можливо, ви захочете налаштувати свій`tries`і`maxExceptions`властивостей класу вашої роботи відповідно. Або, можливо, ви захочете використовувати[`retryUntil`метод](#time-based-attempts)щоб визначити проміжок часу, доки робота більше не повинна виконуватися.

> {tip} Якщо ви використовуєте Redis, ви можете використовувати`RateLimitedWithRedis`Middlware, яке досконало налаштовано для Redis і є більш ефективним, ніж базове обмеження проміжного програмного забезпечення.

<a name="preventing-job-overlaps"></a>

### Запобігання перекриванню робочих місць

Laravel включає a`WithoutOverlapping`Middlware, яке дозволяє запобігати дублюванню завдань на основі довільного ключа. Це може бути корисно, коли завдання в черзі модифікує ресурс, який одночасно має змінюватись лише одним завданням.

Наприклад, уявімо, що у вас є робота з обробки відшкодування, і ви хочете, щоб запобігти дублюванню завдань відшкодування для того самого ідентифікатора замовлення. Для цього ви можете повернути файл`WithoutOverlapping`Middlware від ваших завдань з обробки відшкодування`middleware`метод:

    use Illuminate\Queue\Middleware\WithoutOverlapping;

    /**
     * Get the middleware the job should pass through.
     *
     * @return array
     */
    public function middleware()
    {
        return [new WithoutOverlapping($this->order->id)];
    }

Будь-які перекриваються завдання будуть повернуті до черги. Ви також можете вказати кількість секунд, яка повинна пройти перед повторною спробою завдання:

    /**
     * Get the middleware the job should pass through.
     *
     * @return array
     */
    public function middleware()
    {
        return [(new WithoutOverlapping($this->order->id))->releaseAfter(60)];
    }

Якщо ви хочете негайно видалити будь-які перекриваються завдання, ви можете використовувати`dontRelease`метод:

    /**
     * Get the middleware the job should pass through.
     *
     * @return array
     */
    public function middleware()
    {
        return [(new WithoutOverlapping($this->order->id))->dontRelease()];
    }

> {примітка}`WithoutOverlapping`Middlware вимагає драйвера кешу, який підтримує[замки](/docs/{{version}}/cache#atomic-locks).

<a name="dispatching-jobs"></a>

## Диспетчерські роботи

Після того, як ви написали свій робочий клас, ви можете відправити його за допомогою`dispatch`метод на самій роботі. Аргументи, передані`dispatch`метод буде переданий конструктору завдання:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            ProcessPodcast::dispatch($podcast);
        }
    }

Якщо ви хочете умовно відправити роботу, ви можете використовувати`dispatchIf`і`dispatchUnless`методи:

    ProcessPodcast::dispatchIf($accountActive === true, $podcast);

    ProcessPodcast::dispatchUnless($accountSuspended === false, $podcast);

<a name="delayed-dispatching"></a>

### Затримка відправки

Якщо ви хочете відкласти виконання завдання в черзі, ви можете використовувати`delay`метод при відправленні роботи. Наприклад, вкажемо, що завдання має бути доступним для обробки лише через 10 хвилин після його відправлення:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            ProcessPodcast::dispatch($podcast)
                    ->delay(now()->addMinutes(10));
        }
    }

> {note} Послуга черги Amazon SQS має максимальний час затримки 15 хвилин.

<a name="dispatching-after-the-response-is-sent-to-browser"></a>

#### Диспетчеризація після того, як відповідь надіслано браузеру

Крім того,`dispatchAfterResponse`метод затримує відправлення завдання, поки відповідь не буде надіслана в браузер користувача. Це все одно дозволить користувачеві почати користуватися програмою, навіть якщо завдання в черзі все ще виконується. Зазвичай це слід використовувати лише для завдань, які займають близько секунди, наприклад, надсилання електронного листа:

    use App\Jobs\SendNotification;

    SendNotification::dispatchAfterResponse();

Ви можете`dispatch`a Закриття та ланцюжок`afterResponse`методу на помічник для виконання Закриття після того, як відповідь буде надіслана браузеру:

    use App\Mail\WelcomeMessage;
    use Illuminate\Support\Facades\Mail;

    dispatch(function () {
        Mail::to('taylor@laravel.com')->send(new WelcomeMessage);
    })->afterResponse();

<a name="synchronous-dispatching"></a>

### Синхронна диспетчеризація

Якщо ви хочете негайно відправити роботу (синхронно), ви можете використовувати`dispatchSync`метод. При використанні цього методу завдання не буде в черзі і буде запущено негайно в рамках поточного процесу:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            ProcessPodcast::dispatchSync($podcast);
        }
    }

<a name="job-chaining"></a>

### Підключення робочих місць

Ланцюжок завдань дозволяє вказати список завдань, що стоять у черзі, які слід виконувати послідовно після успішного виконання основного завдання. Якщо одне завдання в послідовності не вдається, решта завдань не запускатимуться. Щоб виконати ланцюжок завдань, що стоять у черзі, ви можете використовувати`chain`метод, передбачений`Bus`фасад:

    use Illuminate\Support\Facades\Bus;

    Bus::chain([
        new ProcessPodcast,
        new OptimizePodcast,
        new ReleasePodcast,
    ])->dispatch();

Окрім ланцюжка екземплярів робочих класів, ви також можете створити ланцюжок Закриття:

    Bus::chain([
        new ProcessPodcast,
        new OptimizePodcast,
        function () {
            Podcast::update(...);
        },
    ])->dispatch();

> {note} Видалення завдань за допомогою`$this->delete()`метод не завадить обробляти ланцюгові завдання. Ланцюг зупинить виконання лише в тому випадку, якщо завдання в ланцюзі не вдається.

<a name="chain-connection-queue"></a>

#### Мережеве підключення та черга

Якщо ви хочете вказати підключення та чергу, які слід використовувати для ланцюгових завдань, ви можете використовувати`onConnection`і`onQueue`методи. Ці методи визначають підключення черги та ім'я черги, які слід використовувати, якщо завданням, що знаходяться в черзі, явно не призначено інше підключення / чергу:

    Bus::chain([
        new ProcessPodcast,
        new OptimizePodcast,
        new ReleasePodcast,
    ])->onConnection('redis')->onQueue('podcasts')->dispatch();

<a name="chain-failures"></a>

#### Несправності ланцюга

При ланцюжку завдань ви можете використовувати`catch`метод, щоб вказати Закриття, яке слід викликати, якщо завдання в ланцюжку не вдається. Даний зворотний виклик отримає екземпляр винятку, який спричинив збій завдання:

    use Illuminate\Support\Facades\Bus;
    use Throwable;

    Bus::chain([
        new ProcessPodcast,
        new OptimizePodcast,
        new ReleasePodcast,
    ])->catch(function (Throwable $e) {
        // A job within the chain has failed...
    })->dispatch();

<a name="customizing-the-queue-and-connection"></a>

### Налаштування черги та підключення

<a name="dispatching-to-a-particular-queue"></a>

#### Відправка до певної черги

Переміщуючи завдання в різні черги, ви можете "класифікувати" свої завдання в черзі і навіть визначити пріоритет, скільки працівників ви призначите в різні черги. Майте на увазі, це не спрямовує завдання на різні "підключення" черги, як це визначено у вашому файлі конфігурації черги, а лише на конкретні черги в межах одного з'єднання. Щоб вказати чергу, використовуйте`onQueue`метод при відправленні завдання:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            ProcessPodcast::dispatch($podcast)->onQueue('processing');
        }
    }

<a name="dispatching-to-a-particular-connection"></a>

#### Відправка до конкретного сполучення

Якщо ви працюєте з кількома підключеннями до черги, ви можете вказати, до якого підключення потрібно надіслати завдання. Щоб вказати з'єднання, використовуйте`onConnection`метод при відправленні завдання:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            ProcessPodcast::dispatch($podcast)->onConnection('sqs');
        }
    }

Ви можете ланцюжок`onConnection`і`onQueue`методи для визначення з'єднання та черги на роботу:

    ProcessPodcast::dispatch($podcast)
                  ->onConnection('sqs')
                  ->onQueue('processing');

<a name="max-job-attempts-and-timeout"></a>

### Вказівка ​​максимальних спроб роботи / значень часу очікування

<a name="max-attempts"></a>

#### Макс. Спроб

Один із підходів до визначення максимальної кількості спроб роботи може бути здійснений через`--tries`увімкніть командний рядок Artisan:

    php artisan queue:work --tries=3

Однак ви можете застосувати більш детальний підхід, визначивши максимальну кількість спроб самого класу завдань. Якщо для завдання вказано максимальну кількість спроб, воно матиме перевагу над значенням, вказаним у командному рядку:

    <?php

    namespace App\Jobs;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * The number of times the job may be attempted.
         *
         * @var int
         */
        public $tries = 5;
    }

<a name="time-based-attempts"></a>

#### Спроби, засновані на часі

Як альтернативу визначенню того, скільки разів робота може бути виконана до того, як вона провалиться, ви можете визначити час, коли робота повинна робити час очікування. Це дозволяє виконати роботу будь-яку кількість разів протягом заданого періоду часу. Щоб визначити час, коли час роботи повинен закінчитися, додайте a`retryUntil`метод до вашого робочого класу:

    /**
     * Determine the time at which the job should timeout.
     *
     * @return \DateTime
     */
    public function retryUntil()
    {
        return now()->addSeconds(5);
    }

> {tip} Ви також можете визначити a`retryUntil`методу для ваших прослуховувачів подій, що стоять у черзі.

<a name="max-exceptions"></a>

#### Макс. Винятки

Іноді вам може знадобитися вказати, що завдання може бути спробувано багато разів, але воно повинно провалитися, якщо повторні спроби ініціюються заданою кількістю винятків. Для цього ви можете визначити a`maxExceptions`майно на вашому робочому класі:

    <?php

    namespace App\Jobs;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * The number of times the job may be attempted.
         *
         * @var int
         */
        public $tries = 25;

        /**
         * The maximum number of exceptions to allow before failing.
         *
         * @var int
         */
        public $maxExceptions = 3;

        /**
         * Execute the job.
         *
         * @return void
         */
        public function handle()
        {
            Redis::throttle('key')->allow(10)->every(60)->then(function () {
                // Lock obtained, process the podcast...
            }, function () {
                // Unable to obtain lock...
                return $this->release(10);
            });
        }
    }

У цьому прикладі завдання звільняється на десять секунд, якщо програма не може отримати блокування Redis і буде продовжувати повторюватися до 25 разів. Однак завдання не вдасться, якщо завдання призведе до трьох не оброблених винятків.

<a name="timeout"></a>

#### Timeout

> {примітка}`pcntl`Потрібно встановити розширення PHP, щоб вказати час очікування.

Аналогічно, максимальна кількість секунд, яку можуть виконувати завдання, може бути вказана за допомогою`--timeout`увімкніть командний рядок Artisan:

    php artisan queue:work --timeout=30

Тим не менш, ви можете також визначити максимальну кількість секунд, коли завдання має бути дозволене для запуску в самому класі завдань. Якщо час очікування вказаний у завданні, він матиме перевагу перед будь-яким часом очікування, вказаним у командному рядку:

    <?php

    namespace App\Jobs;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * The number of seconds the job can run before timing out.
         *
         * @var int
         */
        public $timeout = 120;
    }

Іноді процеси блокування вводу-виводу, такі як сокети або вихідні з'єднання HTTP, можуть не відповідати вказаному часу очікування. Тому, використовуючи ці функції, ви завжди повинні намагатися вказати час очікування, використовуючи також їх API. Наприклад, використовуючи Guzzle, ви завжди повинні вказувати значення підключення та запитувати час очікування.

<a name="error-handling"></a>

### Обробка помилок

Якщо під час обробки завдання буде видано виняток, завдання буде автоматично випущено назад у чергу, тому його можна буде спробувати знову. Завдання продовжуватиме випускатися до тих пір, поки не буде здійснено спробу максимальну кількість разів, дозволених вашим додатком. Максимальна кількість спроб визначається`--tries`перемикач, що використовується на`queue:work`artisan командування. Крім того, максимальна кількість спроб може бути визначена для самого класу завдань. Докладніше про запуск робочої черги[можна знайти нижче](#running-the-queue-worker).

<a name="job-batching"></a>

## Розміщення робочих місць

Функція пакетного керування завданнями Laravel дозволяє вам легко виконати партію завдань, а потім виконати певні дії, коли партія завдань завершиться. Перш ніж розпочати, слід створити міграцію бази даних, щоб створити таблицю, яка міститиме метаінформацію про ваш пакет роботи. Ця міграція може бути створена за допомогою`queue:batches-table`Команда ремісників:

    php artisan queue:batches-table

    php artisan migrate

<a name="defining-batchable-jobs"></a>

### Визначення сумісних робочих місць

Ви повинні створити групову роботу[створити завдання, що стоїть у черзі](#creating-jobs)як нормально; однак вам слід додати`Illuminate\Bus\Batchable`риса до робочого класу. Ця риса забезпечує доступ до`batch`метод, який може бути використаний для отримання поточної партії, в якій виконується завдання:

    <?php

    namespace App\Jobs;

    use App\Models\Podcast;
    use App\Services\AudioProcessor;
    use Illuminate\Bus\Batchable;
    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Queue\SerializesModels;

    class ProcessPodcast implements ShouldQueue
    {
        use Batchable, Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        /**
         * Execute the job.
         *
         * @return void
         */
        public function handle()
        {
            if ($this->batch()->cancelled()) {
                // Detected cancelled batch...

                return;
            }

            // Batched job executing...
        }
    }

<a name="dispatching-batches"></a>

### Диспетчерські партії

Щоб відправити партію завдань, слід скористатися`batch`метод`Bus`фасад. Звичайно, пакетне обслуговування в першу чергу корисно в поєднанні із зворотними викликами завершення. Отже, ви можете використовувати`then`,`catch`, і`finally`методи визначення зворотних викликів завершення для пакета. Кожен із цих зворотних дзвінків отримає`Illuminate\Bus\Batch`екземпляр, коли вони викликаються:

    use App\Jobs\ProcessPodcast;
    use App\Podcast;
    use Illuminate\Bus\Batch;
    use Illuminate\Support\Facades\Bus;
    use Throwable;

    $batch = Bus::batch([
        new ProcessPodcast(Podcast::find(1)),
        new ProcessPodcast(Podcast::find(2)),
        new ProcessPodcast(Podcast::find(3)),
        new ProcessPodcast(Podcast::find(4)),
        new ProcessPodcast(Podcast::find(5)),
    ])->then(function (Batch $batch) {
        // All jobs completed successfully...
    })->catch(function (Batch $batch, Throwable $e) {
        // First batch job failure detected...
    })->finally(function (Batch $batch) {
        // The batch has finished executing...
    })->dispatch();

    return $batch->id;

<a name="naming-batches"></a>

#### Іменування партій

Деякі інструменти, такі як Laravel Horizon та Laravel Telescope, можуть надати більш зручну інформацію про налагодження пакетів, якщо партії названі. Щоб призначити партії довільне ім'я, ви можете зателефонувати до`name`метод при визначенні партії:

    $batch = Bus::batch([
        // ...
    ])->then(function (Batch $batch) {
        // All jobs completed successfully...
    })->name('Process Podcasts')->dispatch();

<a name="batch-connection-queue"></a>

#### Пакетне підключення та черга

Якщо ви хочете вказати підключення та чергу, які слід використовувати для пакетних завдань, ви можете використовувати`onConnection`і`onQueue`методи:

    $batch = Bus::batch([
        // ...
    ])->then(function (Batch $batch) {
        // All jobs completed successfully...
    })->onConnection('redis')->onQueue('podcasts')->dispatch();

<a name="chains-within-batches"></a>

#### Ланцюги в межах партій

Ви можете додати набір[прикуті роботи](#job-chaining)всередині партії, розмістивши ланцюгові завдання всередині масиву. Наприклад, ми можемо виконати два ланцюжки завдань паралельно. Оскільки обидві ланцюжки партіюються, ми зможемо перевірити прогрес завершення партії в цілому:

    Bus::batch([
        [
            new ReleasePodcast(1),
            new SendPodcastReleaseNotification(1),
        ],
        [
            new ReleasePodcast(2),
            new SendPodcastReleaseNotification(2),
        ],
    ])->dispatch();

<a name="adding-jobs-to-batches"></a>

### Додавання робочих місць до пакетів

Іноді може бути корисним додавання додаткових завдань до партії зсередини пакетного завдання. Цей шаблон може бути корисним, коли вам потрібно скласти тисячі завдань, які можуть надто довго надходити під час веб-запиту. Отже, замість цього, можливо, ви захочете відправити початкову партію завдань "навантажувача", які зволожують партію новими завданнями:

    $batch = Bus::batch([
        new LoadImportBatch,
        new LoadImportBatch,
        new LoadImportBatch,
    ])->then(function (Batch $batch) {
        // All jobs completed successfully...
    })->name('Import Contacts')->dispatch();

У цьому прикладі ми будемо використовувати`LoadImportBatch`завдання гідратувати партію додатковими робочими місцями. Для цього ми можемо використовувати`add`метод на екземплярі пакета, до якого можна отримати доступ в рамках завдання:

    use App\Jobs\ImportContacts;
    use Illuminate\Support\Collection;

    /**
     * Execute the job.
     *
     * @return void
     */
    public function handle()
    {
        if ($this->batch()->cancelled()) {
            return;
        }

        $this->batch()->add(Collection::times(1000, function () {
            return new ImportContacts;
        }));
    }

> {note} Додавати завдання до партії можна лише із задання, яке належить тій самій партії.

<a name="inspecting-batches"></a>

### Огляд партій

`Illuminate\Bus\Batch`Метод, який надається для зворотного виклику пакетного завершення, має різноманітні властивості та методи, що допомагають вам взаємодіяти та перевіряти дану партію завдань.

    // The UUID of the batch...
    $batch->id;

    // The name of the batch (if applicable)...
    $batch->name;

    // The number of jobs assigned to the batch...
    $batch->totalJobs;

    // The number of jobs that have not been processed by the queue...
    $batch->pendingJobs;

    // The number of jobs that have failed...
    $batch->failedJobs;

    // The number of jobs that have been processed thus far...
    $batch->processedJobs();

    // The completion percentage of the batch (0-100)...
    $batch->progress();

    // Indicates if the batch has finished executing...
    $batch->finished();

    // Cancel the execution of the batch...
    $batch->cancel();

    // Indicates if the batch has been cancelled...
    $batch->cancelled();

<a name="returning-batches-from-routes"></a>

#### Повернення пакетів з маршрутів

Всі`Illuminate\Bus\Batch`екземпляри можна JSON серіалізувати, тобто ви можете повернути їх безпосередньо з одного із маршрутів вашої програми, щоб отримати корисне навантаження JSON, що містить інформацію про пакет, включаючи хід його завершення. Щоб отримати партію за її ідентифікатором, ви можете використовувати`Bus`фасадні`findBatch`метод:

    use Illuminate\Support\Facades\Bus;
    use Illuminate\Support\Facades\Route;

    Route::get('/batch/{batchId}', function (string $batchId) {
        return Bus::findBatch($batchId);
    });

<a name="cancelling-batches"></a>

### Скасування пакетів

Іноді може знадобитися скасувати виконання даної партії. Це можна зробити, зателефонувавши до`cancel`метод на`Illuminate\Bus\Batch`примірник:

    /**
     * Execute the job.
     *
     * @return void
     */
    public function handle()
    {
        if ($this->user->exceedsImportLimit()) {
            return $this->batch()->cancel();
        }

        if ($this->batch()->cancelled()) {
            return;
        }
    }

<a name="batch-failures"></a>

### Пакетні помилки

Коли пакетне завдання не вдається,`catch`буде викликаний зворотний виклик (якщо призначений). Цей зворотний виклик викликається лише для завдання, яке не вдається виконати в пакетному режимі.

<a name="allowing-failures"></a>

#### Допущення збоїв

Коли завдання в партії не вдається, Laravel автоматично позначить партію як "скасовану". За бажанням ви можете вимкнути цю поведінку, щоб відмова в роботі автоматично не позначив пакет як скасований. Це може бути здійснено за допомогою виклику`allowFailures`метод під час відправлення партії:

    $batch = Bus::batch([
        // ...
    ])->then(function (Batch $batch) {
        // All jobs completed successfully...
    })->allowFailures()->dispatch();

<a name="retrying-failed-batch-jobs"></a>

#### Повторна спроба невдалих пакетних завдань

Для зручності Laravel пропонує`queue:retry-batch`Команда Artisan, яка дозволяє легко повторити всі невдалі завдання для даної партії.`queue:retry-batch`команда приймає UUID пакета, чиї невдалі завдання слід повторити:

    php artisan queue:retry-batch 32dbc76c-4f82-4749-b610-a639fe0099b5

<a name="queueing-closures"></a>

## Закриття в черзі

Замість відправлення класу завдань до черги, ви також можете надіслати Закриття. Це чудово підходить для швидких, простих завдань, які потрібно виконувати поза поточним циклом запиту. При відправленні Закриття до черги вміст коду Закриття підписується криптографічно, тому його не можна змінювати при передачі:

    $podcast = App\Podcast::find(1);

    dispatch(function () use ($podcast) {
        $podcast->publish();
    });

Використання`catch`методом, ви можете надати Закриття, яке слід виконати, якщо Закриття в черзі не вдається успішно завершити після вичерпання всіх налаштованих спроб спроби повторної спроби у вашій черзі:

    use Throwable;

    dispatch(function () use ($podcast) {
        $podcast->publish();
    })->catch(function (Throwable $e) {
        // This job has failed...
    });

<a name="running-the-queue-worker"></a>

## Запуск програми черги

Laravel включає працівника черги, який буде обробляти нові завдання, коли їх надсилатимуть у чергу. Ви можете запустити працівника за допомогою`queue:work`artisan командування. Зверніть увагу, що один раз`queue:work`команда запущена, вона буде продовжувати працювати, доки не буде зупинена вручну або ви не закриєте термінал:

    php artisan queue:work

> {tip} Зберігати`queue:work`процес, який постійно працює у фоновому режимі, слід використовувати монітор процесу, такий як[Керівник](#supervisor-configuration)щоб переконатися, що працівник черги не припиняє працювати.

Пам'ятайте, що працівники черги - це довгоживучі процеси, які зберігають завантажений стан програми в пам'яті. Як результат, вони не помітять змін у вашій базі коду після їх запуску. Отже, під час процесу розгортання обов’язково[перезапустіть працівників черги](#queue-workers-and-deployment). Крім того, пам’ятайте, що будь-який статичний стан, створений або змінений вашою програмою, не буде автоматично скидатися між завданнями.

Крім того, ви можете запустити`queue:listen`команди. При використанні`queue:listen`команда, вам не потрібно вручну перезапускати працівника, коли ви хочете перезавантажити ваш оновлений код або скинути стан програми; однак ця команда не така ефективна, як`queue:work`:

    php artisan queue:listen

<a name="specifying-the-connection-queue"></a>

#### Вказівка ​​підключення та черги

Ви також можете вказати, яке підключення черги повинен використовувати працівник. Ім'я підключення передано до`work`команда повинна відповідати одному із з'єднань, визначених у вашому`config/queue.php`файл конфігурації:

    php artisan queue:work redis

Ви можете ще більше налаштувати свою чергу, лише обробляючи певні черги для даного з'єднання. Наприклад, якщо всі ваші електронні листи обробляються в`emails`черга на ваш`redis`підключення черги, ви можете виконати таку команду для запуску працівника, який обробляє лише ту чергу:

    php artisan queue:work redis --queue=emails

<a name="processing-a-specified-number-of-jobs"></a>

#### Обробка заданої кількості робочих місць

`--once`Опція може бути використана для інструктування працівника обробляти лише одне завдання з черги:

    php artisan queue:work --once

`--max-jobs`варіант може бути використаний, щоб доручити працівникові обробити задану кількість завдань, а потім вийти. Цей параметр може бути корисним у поєднанні з[Керівник](#supervisor-configuration)так що ваші працівники будуть автоматично перезапущені після обробки заданої кількості завдань:

    php artisan queue:work --max-jobs=1000

<a name="processing-all-queued-jobs-then-exiting"></a>

#### Обробка всіх завдань, що стоять у черзі, а потім вихід

`--stop-when-empty`Опція може бути використана, щоб доручити працівникові обробити всі завдання, а потім вийти витончено. Цей параметр може бути корисним при роботі з чергами Laravel у контейнері Docker, якщо ви хочете вимкнути контейнер після порожньої черги:

    php artisan queue:work --stop-when-empty

<a name="processing-jobs-for-a-given-number-of-seconds"></a>

#### Обробка завдань протягом заданої кількості секунд

`--max-time`Опція може бути використана, щоб доручити працівникові обробити завдання протягом заданої кількості секунд, а потім вийти. Цей параметр може бути корисним у поєднанні з[Керівник](#supervisor-configuration)щоб ваші працівники автоматично перезапускались після обробки завдань протягом певного періоду часу:

    // Process jobs for one hour and then exit...
    php artisan queue:work --max-time=3600

<a name="resource-considerations"></a>

#### Міркування щодо ресурсів

Працівники черги демонів не «перезавантажують» фреймворк перед обробкою кожного завдання. Тому ви повинні звільнити важкі ресурси після виконання кожної роботи. Наприклад, якщо ви виконуєте обробку зображень за допомогою бібліотеки GD, вам слід звільнити пам'ять за допомогою`imagedestroy`коли закінчите.

<a name="queue-priorities"></a>

### Пріоритети черги

Іноді вам може знадобитися визначити пріоритет, як обробляються ваші черги. Наприклад, у вашому`config/queue.php`Ви можете встановити за замовчуванням`queue`для вашого`redis`підключення до`low`. Однак іноді, можливо, ви захочете підштовхнути роботу до`high`пріоритетна черга приблизно так:

    dispatch((new Job)->onQueue('high'));

Щоб запустити працівника, який перевіряє, що всі`high`завдання черги обробляються перед тим, як перейти до будь-яких завдань на`low`черзі, передайте список назв черг, розділених комами, в`work`команда:

    php artisan queue:work --queue=high,low

<a name="queue-workers-and-deployment"></a>

### Робітники черги та розгортання

Оскільки працівники черги є довготривалими процесами, вони не будуть приймати зміни у вашому коді без перезапуску. Отже, найпростіший спосіб розгортання програми за допомогою черги - це перезапустити працівників під час процесу розгортання. Ви можете витончено перезапустити всіх працівників, видавши`queue:restart`команда:

    php artisan queue:restart

Ця команда дасть вказівку всім працівникам черги витончено "померти" після того, як вони закінчать обробку свого поточного завдання, щоб жодна існуюча робота не була втрачена. Так як працівники черги загинуть, коли`queue:restart`команда виконується, ви повинні запускати менеджер процесів, такий як[Керівник](#supervisor-configuration)для автоматичного перезапуску працівників черги.

> {tip} Черга використовує[кеш](/docs/{{version}}/cache)для збереження сигналів перезапуску, тому перед використанням цієї функції слід перевірити, чи правильно налаштований драйвер кешу для вашого додатка.

<a name="job-expirations-and-timeouts"></a>

### Термін дії та час очікування

<a name="job-expiration"></a>

#### Закінчення роботи

У вашому`config/queue.php`файл конфігурації, кожне підключення черги визначає a`retry_after`варіант. Цей параметр визначає, скільки секунд слід чекати підключення до черги перед повторною спробою завдання, яке обробляється. Наприклад, якщо значення`retry_after`встановлено на`90`, завдання буде випущено назад у чергу, якщо воно оброблялося протягом 90 секунд без видалення. Як правило, слід встановити`retry_after`значення до максимальної кількості секунд, які ваші робочі місця повинні прийняти розумно, щоб завершити обробку.

> {note} Єдине підключення до черги, яке не містить`retry_after`значення - SQS Amazon. SQS повторить спробу завдання на основі[Типовий час очікування видимості](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html)який керується в консолі AWS.

<a name="worker-timeouts"></a>

#### Час очікування працівника

`queue:work`Командування ремісників викриває a`--timeout`варіант.`--timeout`Параметр вказує, скільки часу буде чекати головний процес черги Laravel, перш ніж вбивати дочірнього працівника черги, який обробляє завдання. Іноді процес дочірньої черги може з різних причин «заморозитися».`--timeout`Параметр видаляє заморожені процеси, які перевищили вказаний часовий ліміт:

    php artisan queue:work --timeout=60

`retry_after`параметр конфігурації та`--timeout`Параметри CLI різні, але працюйте разом, щоб гарантувати, що робочі місця не втрачаються і що завдання успішно обробляються лише один раз.

> {примітка}`--timeout`значення завжди має бути принаймні на кілька секунд коротше вашого`retry_after`значення конфігурації. Це забезпечить, щоб працівник, який обробляє дане завдання, завжди був убитий перед тим, як його повторити. Якщо ти`--timeout`варіант довший за ваш`retry_after`значення конфігурації, ваші завдання можуть бути оброблені двічі.

<a name="worker-sleep-duration"></a>

#### Тривалість сну працівника

Коли завдання доступні в черзі, працівник продовжить обробку завдань без затримки між ними. Однак`sleep`Опція визначає, як довго (у секундах) працівник буде "спати", якщо нових робочих місць немає. Під час сну працівник не буде обробляти жодної нової роботи - робота буде оброблена після того, як робітник знову прокинеться.

    php artisan queue:work --sleep=3

<a name="supervisor-configuration"></a>

## Конфігурація супервізора

<a name="installing-supervisor"></a>

#### Встановлення Supervisor

Supervisor - це монітор процесів для операційної системи Linux, який автоматично перезапустить ваш`queue:work`обробляти, якщо він не вдається. Щоб встановити Supervisor на Ubuntu, ви можете використати таку команду:

    sudo apt-get install supervisor

> {tip} Якщо налаштування супервізора самостійно здається вражаючим, рекомендуємо використовувати[Кузня Laravel](https://forge.laravel.com), який автоматично встановить і налаштує Supervisor для ваших проектів Laravel.

<a name="configuring-supervisor"></a>

#### Налаштування супервізора

Файли конфігурації супервізора зазвичай зберігаються в`/etc/supervisor/conf.d`каталог. У цьому каталозі ви можете створити будь-яку кількість файлів конфігурації, які вказують керівнику, як слід контролювати ваші процеси. Наприклад, давайте створимо файл`laravel-worker.conf`файл, який запускається та контролює a`queue:work`процес:

    [program:laravel-worker]
    process_name=%(program_name)s_%(process_num)02d
    command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3 --max-time=3600
    autostart=true
    autorestart=true
    user=forge
    numprocs=8
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/worker.log
    stopwaitsecs=3600

У цьому прикладі`numprocs`директива доручить супервізору запустити 8`queue:work`обробляє та контролює всі з них, автоматично перезапускаючи їх, якщо вони не вдаються. Вам слід змінити`queue:work sqs`частина`command`директиви, щоб відобразити бажане з'єднання з чергою

> {note} Ви повинні переконатися, що значення`stopwaitsecs`більше, ніж кількість секунд, витрачених вашим найдовшим завданням. В іншому випадку Супервайзер може вбити завдання до його обробки.

<a name="starting-supervisor"></a>

#### Початковий керівник

Після створення файлу конфігурації ви можете оновити конфігурацію супервізора та запустити процеси, використовуючи такі команди:

    sudo supervisorctl reread

    sudo supervisorctl update

    sudo supervisorctl start laravel-worker:*

Щоб отримати додаткову інформацію про наглядача, зверніться до[Документація керівника](http://supervisord.org/index.html).

<a name="dealing-with-failed-jobs"></a>

## Робота з невдалими робочими місцями

Іноді ваші роботи в черзі не вдаються. Не хвилюйся, не завжди все відбувається за планом! Laravel включає зручний спосіб вказати максимальну кількість спроб роботи. Після того, як завдання перевищить цю кількість спроб, воно буде вставлено в`failed_jobs`таблиця бази даних. Створити міграцію для`failed_jobs`таблиці, ви можете використовувати`queue:failed-table`команда:

    php artisan queue:failed-table

    php artisan migrate

Потім, при запуску вашого[працівник черги](#running-the-queue-worker), Ви можете вказати максимальну кількість спроб роботи за допомогою`--tries`увімкніть`queue:work`команди. Якщо ви не вказали значення для`--tries`варіант, робота буде виконана лише один раз:

    php artisan queue:work redis --tries=3

Крім того, ви можете вказати, скільки секунд слід чекати Laravel перед повторною спробою завдання, яке не вдалося використати`--backoff`варіант. За замовчуванням завдання негайно повторюється:

    php artisan queue:work redis --tries=3 --backoff=3

Якщо ви хочете налаштувати затримку повторної спроби невдалого завдання для кожного завдання, ви можете зробити це, визначивши a`backoff`майно у вашому класі роботи в черзі:

    /**
     * The number of seconds to wait before retrying the job.
     *
     * @var int
     */
    public $backoff = 3;

Якщо вам потрібна більш складна логіка для визначення затримки повторної спроби, ви можете визначити a`backoff`у вашому робочому класі в черзі:

    /**
    * Calculate the number of seconds to wait before retrying the job.
    *
    * @return int
    */
    public function backoff()
    {
        return 3;
    }

Ви можете легко налаштувати "експоненціальні" резервні копії, повернувши масив змінних значень із`backoff`метод. У цьому прикладі затримка повторної спроби становитиме 1 секунду для першої спроби, 5 секунд для другої спроби та 10 секунд для третьої спроби:

    /**
    * Calculate the number of seconds to wait before retrying the job.
    *
    * @return array
    */
    public function backoff()
    {
        return [1, 5, 10];
    }

<a name="cleaning-up-after-failed-jobs"></a>

### Прибирання після невдалих робіт

Ви можете визначити a`failed`метод безпосередньо на вашому робочому класі, що дозволяє виконувати конкретне очищення роботи, коли виникає несправність. Це ідеальне місце, щоб надіслати Notification своїм користувачам або скасувати будь-які дії, виконані завданням.`Throwable`виняток, який спричинив помилку завдання, буде передано до`failed`метод:

    <?php

    namespace App\Jobs;

    use App\Models\Podcast;
    use App\Services\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Queue\SerializesModels;
    use Throwable;

    class ProcessPodcast implements ShouldQueue
    {
        use InteractsWithQueue, Queueable, SerializesModels;

        protected $podcast;

        /**
         * Create a new job instance.
         *
         * @param  \App\Models\Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }

        /**
         * Execute the job.
         *
         * @param  \App\Services\AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // Process uploaded podcast...
        }

        /**
         * Handle a job failure.
         *
         * @param  \Throwable  $exception
         * @return void
         */
        public function failed(Throwable $exception)
        {
            // Send user notification of failure, etc...
        }
    }

<a name="failed-job-events"></a>

### Помилка роботи

Якщо ви хочете зареєструвати подію, яка буде викликана, коли завдання не вдасться, ви можете використовувати`Queue::failing`метод. Ця подія - чудова можливість повідомити свою команду електронною поштою або[Ослаблення](https://www.slack.com). Наприклад, ми можемо приєднати зворотний дзвінок до цієї події з`AppServiceProvider`що входить до складу Laravel:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Queue\Events\JobFailed;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Queue::failing(function (JobFailed $event) {
                // $event->connectionName
                // $event->job
                // $event->exception
            });
        }
    }

<a name="retrying-failed-jobs"></a>

### Повторна спроба невдалих завдань

Щоб переглянути всі ваші невдалі завдання, які були вставлені у ваш`failed_jobs`таблиці бази даних, ви можете використовувати`queue:failed`Команда ремісників:

    php artisan queue:failed

`queue:failed`команда перелічить ідентифікатор завдання, підключення, чергу, час відмов та іншу інформацію про роботу. Ідентифікатор завдання може бути використаний для повторної спроби невдалого завдання. Наприклад, повторити спробу невдалого завдання з ідентифікатором`5`, виконайте таку команду:

    php artisan queue:retry 5

За необхідності ви можете передати декілька ідентифікаторів або діапазон ідентифікаторів (при використанні числових ідентифікаторів) команді:

    php artisan queue:retry 5 6 7 8 9 10

    php artisan queue:retry --range=5-10

Щоб повторити всі невдалі завдання, виконайте`queue:retry`командувати та передавати`all`як ідентифікатор:

    php artisan queue:retry all

Якщо ви хочете видалити невдале завдання, ви можете використовувати`queue:forget`команда:

    php artisan queue:forget 5

> {tip} Під час використання[Горизонт](/docs/{{version}}/horizon), вам слід використовувати`horizon:forget`команда видалити невдале завдання замість`queue:forget`команди.

Щоб видалити всі свої невдалі завдання, ви можете використовувати`queue:flush`команда:

    php artisan queue:flush

<a name="ignoring-missing-models"></a>

### Ігнорування відсутніх моделей

При введенні красномовної моделі в завдання вона автоматично серіалізується перед тим, як розміщуватись у черзі, і відновлюється при обробці завдання. Однак якщо модель була видалена, поки завдання чекало на обробку працівником, ваше завдання може провалитись із`ModelNotFoundException`.

Для зручності ви можете вибрати автоматичне видалення завдань із відсутніми моделями, встановивши завдання`deleteWhenMissingModels`майно до`true`:

    /**
     * Delete the job if its models no longer exist.
     *
     * @var bool
     */
    public $deleteWhenMissingModels = true;

<a name="clearing-jobs-from-queues"></a>

## Очищення робочих місць із черг

> {tip} Під час використання[Горизонт](/docs/{{version}}/horizon), вам слід використовувати`horizon:clear`команда для очищення завдань з черги замість`queue:clear`команди.

Якщо ви хочете видалити всі завдання із черги за замовчуванням підключення за замовчуванням, ви можете зробити це за допомогою`queue:clear`Команда ремісників:

    php artisan queue:clear

Ви також можете надати`connection`аргумент і`queue`можливість видалення завдань із певного підключення та черги:

    php artisan queue:clear redis --queue=emails

> {note} Видалення завдань із черг доступне лише для драйверів черги SQS, Redis та бази даних. Крім того, процес видалення повідомлення SQS займає до 60 секунд, тому завдання, надіслані до черги SQS до 60 секунд після очищення черги, також можуть бути видалені.

<a name="job-events"></a>

## Події роботи

Використання`before`і`after`методи на`Queue`[фасад](/docs/{{version}}/facades), ви можете вказати зворотні виклики, які будуть виконуватися до або після обробки завдання в черзі. Ці зворотні виклики є чудовою можливістю виконати додаткову реєстрацію або збільшення статистики для інформаційної панелі. Як правило, вам слід викликати ці методи з[постачальник послуг](/docs/{{version}}/providers). Наприклад, ми можемо використовувати`AppServiceProvider`що входить до складу Laravel:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Queue\Events\JobProcessed;
    use Illuminate\Queue\Events\JobProcessing;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Queue::before(function (JobProcessing $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
            });

            Queue::after(function (JobProcessed $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
            });
        }
    }

Використання`looping`метод на`Queue`[фасад](/docs/{{version}}/facades), ви можете вказати зворотні виклики, які виконуються до того, як працівник намагається отримати завдання з черги. Наприклад, ви можете зареєструвати закриття для відкату будь-яких транзакцій, які залишились відкритими раніше невдалим завданням:

    Queue::looping(function () {
        while (DB::transactionLevel() > 0) {
            DB::rollBack();
        }
    });
