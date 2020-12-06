# Редіс

-   [Вступ](#introduction)
    -   [Конфігурація](#configuration)
    -   [Преддіс](#predis)
    -   [PhpRedis](#phpredis)
-   [Взаємодія з Redis](#interacting-with-redis)
    -   [Конвеєрні команди](#pipelining-commands)
-   [Паб / Під](#pubsub)

<a name="introduction"></a>

## Вступ

[Редіс](https://redis.io)є розширеним сховищем ключ-значення з відкритим кодом. Його часто називають сервером структури даних, оскільки ключі можуть містити[струни](https://redis.io/topics/data-types#strings),[хеші](https://redis.io/topics/data-types#hashes),[списки](https://redis.io/topics/data-types#lists),[набори](https://redis.io/topics/data-types#sets), і[відсортовані набори](https://redis.io/topics/data-types#sorted-sets).

Перш ніж використовувати Redis з Laravel, радимо встановити та використовувати[PhpRedis](https://github.com/phpredis/phpredis)Розширення PHP через PECL. Розширення є більш складним для встановлення, але може забезпечити кращу продуктивність для програм, які інтенсивно використовують Redis.

Крім того, ви можете встановити`predis/predis`пакет через Composer:

    composer require predis/predis

<a name="configuration"></a>

### Конфігурація

Конфігурація Redis для вашої програми знаходиться в`config/database.php`файл конфігурації. У цьому файлі ви побачите файл`redis`масив, що містить сервери Redis, що використовуються вашим додатком:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'default' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', 6379),
            'database' => env('REDIS_DB', 0),
        ],

        'cache' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', 6379),
            'database' => env('REDIS_CACHE_DB', 1),
        ],

    ],

Конфігурації сервера за замовчуванням має бути достатньо для розробки. Однак ви можете змінювати цей масив, виходячи з вашого оточення. Кожен сервер Redis, визначений у вашому конфігураційному файлі, повинен мати ім'я, хост та порт, якщо ви не визначите одну URL-адресу, яка представляє з'єднання Redis:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'default' => [
            'url' => 'tcp://127.0.0.1:6379?database=0',
        ],

        'cache' => [
            'url' => 'tls://user:password@127.0.0.1:6380?database=1',
        ],

    ],

<a name="configuring-the-connection-scheme"></a>

#### Налаштування схеми підключення

За замовчуванням клієнти Redis будуть використовувати`tcp`схема при підключенні до серверів Redis; однак ви можете використовувати шифрування TLS / SSL, вказавши a`scheme`параметр конфігурації у вашій конфігурації сервера Redis:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'default' => [
            'scheme' => 'tls',
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', 6379),
            'database' => env('REDIS_DB', 0),
        ],

    ],

<a name="configuring-clusters"></a>

#### Налаштування кластерів

Якщо у вашій програмі використовується кластер серверів Redis, вам слід визначити ці кластери в межах`clusters`ключ вашої конфігурації Redis:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'clusters' => [
            'default' => [
                [
                    'host' => env('REDIS_HOST', 'localhost'),
                    'password' => env('REDIS_PASSWORD', null),
                    'port' => env('REDIS_PORT', 6379),
                    'database' => 0,
                ],
            ],
        ],

    ],

За замовчуванням кластери виконуватимуть шардінг на стороні клієнта між вашими вузлами, що дозволить вам об'єднати вузли та створити велику кількість доступної оперативної пам'яті. Однак зверніть увагу, що на стороні клієнта шардінг не справляється з відмовою; тому він в першу чергу підходить для кешованих даних, доступних з іншого основного сховища даних. Якщо ви хочете використовувати рідну кластеризацію Redis, ви повинні вказати це в`options`ключ вашої конфігурації Redis:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
        ],

        'clusters' => [
            // ...
        ],

    ],

<a name="predis"></a>

### Преддіс

Щоб використовувати розширення Predis, вам слід змінити`REDIS_CLIENT`змінна середовища від`phpredis`до`predis`:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'predis'),

        // Rest of Redis configuration...
    ],

На додаток до типового`host`,`port`,`database`, і`password`параметри конфігурації сервера, Predis підтримує додаткові[параметри з'єднання](https://github.com/nrk/predis/wiki/Connection-Parameters)які можуть бути визначені для кожного з ваших серверів Redis. Щоб використовувати ці додаткові параметри конфігурації, додайте їх до конфігурації сервера Redis у`config/database.php`файл конфігурації:

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_write_timeout' => 60,
    ],

<a name="phpredis"></a>

### PhpRedis

Розширення PhpRedis налаштовано за замовчуванням на`REDIS_CLIENT`env та у вашому`config/database.php`:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        // Rest of Redis configuration...
    ],

Якщо ви плануєте використовувати розширення PhpRedis разом із`Redis`Фасадний псевдонім, вам слід перейменувати його на щось інше, наприклад`RedisManager`, to avoid a collision with the Redis class. You can do that in the aliases section of your `app.php`конфігураційний файл.

    'RedisManager' => Illuminate\Support\Facades\Redis::class,

На додаток до типового`host`,`port`,`database`, і`password`Параметри конфігурації сервера, PhpRedis підтримує такі додаткові параметри підключення:`persistent`,`prefix`,`read_timeout`,`retry_interval`,`timeout`, і`context`. Ви можете додати будь-який із цих параметрів до конфігурації сервера Redis у`config/database.php`файл конфігурації:

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_timeout' => 60,
        'context' => [
            // 'auth' => ['username', 'secret'],
            // 'stream' => ['verify_peer' => false],
        ],
    ],

<a name="the-redis-facade"></a>

#### Фасад Redis

Щоб уникнути зіткнень імен класів із самим розширенням Redis PHP, вам потрібно буде видалити або перейменувати`Illuminate\Support\Facades\Redis`фасадний псевдонім від вашого`app`файли конфігурації`aliases`масив. Як правило, ви повинні повністю видалити цей псевдонім і посилатися на фасад лише за його повною назвою класу під час використання розширення Redis PHP.

<a name="interacting-with-redis"></a>

## Взаємодія з Redis

Ви можете взаємодіяти з Redis, викликаючи різні методи на`Redis`[фасад](/docs/{{version}}/facades).`Redis`фасад підтримує динамічні методи, тобто ви можете зателефонувати будь-яким[Команда Redis](https://redis.io/commands)на фасаді і команда буде передана безпосередньо Редісу. У цьому прикладі ми будемо називати Redis`GET`команда за допомогою виклику`get`метод на`Redis`фасад:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Redis;

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
            $user = Redis::get('user:profile:'.$id);

            return view('user.profile', ['user' => $user]);
        }
    }

Як зазначалося вище, ви можете викликати будь-яку з команд Redis на`Redis`фасад. Laravel використовує магічні методи для передачі команд серверу Redis, тому передайте аргументи, яких очікує команда Redis:

    Redis::set('name', 'Taylor');

    $values = Redis::lrange('names', 5, 10);

Крім того, ви також можете передавати команди серверу за допомогою`command`метод, який приймає ім'я команди як перший аргумент, а масив значень як другий аргумент:

    $values = Redis::command('lrange', ['name', 5, 10]);

<a name="using-multiple-redis-connections"></a>

#### Використання декількох з'єднань Redis

Ви можете отримати екземпляр Redis, зателефонувавши до`Redis::connection`метод:

    $redis = Redis::connection();

Це дасть вам екземпляр сервера Redis за замовчуванням. Ви також можете передати ім'я з'єднання або кластера в`connection`спосіб отримати певний сервер або кластер, як визначено у вашій конфігурації Redis:

    $redis = Redis::connection('my-connection');

<a name="pipelining-commands"></a>

### Конвеєрні команди

Конвеєризацію слід використовувати, коли вам потрібно надіслати багато команд на сервер.`pipeline`метод приймає один аргумент: a`Closure`що отримує екземпляр Redis. Ви можете видати всі свої команди цьому екземпляру Redis, і всі вони будуть передаватися на сервер, таким чином забезпечуючи кращу продуктивність:

    Redis::pipeline(function ($pipe) {
        for ($i = 0; $i < 1000; $i++) {
            $pipe->set("key:$i", $i);
        }
    });

<a name="pubsub"></a>

## Паб / Під

Laravel забезпечує зручний інтерфейс для Redis`publish`і`subscribe`команди. Ці команди Redis дозволяють прослуховувати повідомлення на даному "каналі". Ви можете публікувати повідомлення на каналі з іншої програми або навіть з використанням іншої мови програмування, що дозволяє легко спілкуватися між програмами та процесами.

Спочатку налаштуємо слухач каналу за допомогою`subscribe`метод. Ми розмістимо цей виклик методу в межах[Artisan командування](/docs/{{version}}/artisan)з моменту виклику`subscribe`метод починає тривалий процес:

    <?php

    namespace App\Console\Commands;

    use Illuminate\Console\Command;
    use Illuminate\Support\Facades\Redis;

    class RedisSubscribe extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'redis:subscribe';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Subscribe to a Redis channel';

        /**
         * Execute the console command.
         *
         * @return mixed
         */
        public function handle()
        {
            Redis::subscribe(['test-channel'], function ($message) {
                echo $message;
            });
        }
    }

Тепер ми можемо публікувати повідомлення на каналі за допомогою`publish`метод:

    Route::get('publish', function () {
        // Route logic...

        Redis::publish('test-channel', json_encode(['foo' => 'bar']));
    });

<a name="wildcard-subscriptions"></a>

#### Підписка на підстановку

Використання`psubscribe`Ви можете підписатись на підстановочний канал, що може бути корисним для перегляду всіх повідомлень на всіх каналах.`$channel`name буде передано як другий аргумент наданому зворотному виклику`Closure`:

    Redis::psubscribe(['*'], function ($message, $channel) {
        echo $message;
    });

    Redis::psubscribe(['users.*'], function ($message, $channel) {
        echo $message;
    });
