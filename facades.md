# Фасади

-   [Вступ](#introduction)
-   [Коли використовувати фасади](#when-to-use-facades)
    -   [Фасади Vs. Ін’єкція залежності](#facades-vs-dependency-injection)
    -   [Фасади Vs. Допоміжні функції](#facades-vs-helper-functions)
-   [Як працюють фасади](#how-facades-work)
-   [Фасади в режимі реального часу](#real-time-facades)
-   [Class Reference фасадів](#facade-class-reference)

<a name="introduction"></a>

## Вступ

Фасади забезпечують "статичний" інтерфейс для класів, доступних у програмі[службовий контейнер](/docs/{{version}}/container). Судна Laravel мають безліч фасадів, що забезпечують доступ майже до всіх функцій Laravel. Фасади Laravel служать "статичними проксі" основних класів у службовому контейнері, забезпечуючи перевагу стислого, виразного синтаксису, зберігаючи при цьому більшу перевірочність і гнучкість, ніж традиційні статичні методи.

Усі фасади Laravel визначені в`Illuminate\Support\Facades`простір імен. Отже, ми можемо легко отримати доступ до фасаду так:

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

У документації Laravel багато прикладів використовуватимуть фасади, щоб продемонструвати різні особливості фреймворку.

<a name="when-to-use-facades"></a>

## Коли використовувати фасади

Фасади мають багато переваг. Вони забезпечують стислий запам’ятовуваний синтаксис, що дозволяє використовувати функції Laravel, не пам’ятаючи довгих назв класів, які потрібно вводити або конфігурувати вручну. Крім того, завдяки унікальному використанню динамічних методів PHP, їх легко перевірити.

Однак слід дотримуватися певної обережності при використанні фасадів. Основною небезпекою фасадів є повзучість класу. Оскільки фасади настільки прості у використанні і не потребують впорскування, можна легко дозволити вашим класам продовжувати рости та використовувати багато фасадів в одному класі. Використовуючи ін’єкцію залежностей, цей потенціал пом’якшується за допомогою візуального зворотного зв’язку, який великий конструктор дає вам, що ваш клас стає занадто великим. Тож, використовуючи фасади, звертайте особливу увагу на розмір вашого класу, щоб сфера його відповідальності залишалася вузькою.

> {tip} При створенні стороннього пакету, який взаємодіє з Laravel, краще вводити[Контракти Laravel](/docs/{{version}}/contracts)замість використання фасадів. Оскільки пакети створюються за межами самої Laravel, ви не матимете доступу до помічників тестування фасадів Laravel.

<a name="facades-vs-dependency-injection"></a>

### Фасади проти Ін’єкція залежності

Однією з основних переваг введення залежностей є можливість обміну реалізаціями введеного класу. Це корисно під час тестування, оскільки ви можете ввести макет або заглушку та стверджувати, що на заглушці були викликані різні методи.

Як правило, неможливо знущатись чи заглушати дійсно статичний метод класу. Однак, оскільки фасади використовують динамічні методи для виклику методів проксі до об'єктів, вирішених із контейнера служби, ми фактично можемо тестувати фасади так само, як і тестований ін'єкційний клас. Наприклад, враховуючи такий маршрут:

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

Ми можемо написати наступний тест, щоб переконатися, що`Cache::get`метод був викликаний з аргументом, який ми очікували:

    use Illuminate\Support\Facades\Cache;

    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $response = $this->get('/cache');

        $response->assertSee('value');
    }

<a name="facades-vs-helper-functions"></a>

### Фасади проти Допоміжні функції

На додаток до фасадів, Laravel включає в себе безліч "допоміжних" функцій, які можуть виконувати загальні завдання, такі як генерація поглядів, активація подій, відправлення завдань або надсилання відповідей HTTP. Багато з цих допоміжних функцій виконують ту саму функцію, що і відповідний фасад. Наприклад, цей фасадний виклик та допоміжний виклик еквівалентні:

    return View::make('profile');

    return view('profile');

Практичної різниці між фасадами та допоміжними функціями немає абсолютно. Використовуючи допоміжні функції, ви все одно можете протестувати їх точно так само, як і відповідний фасад. Наприклад, враховуючи такий маршрут:

    Route::get('/cache', function () {
        return cache('key');
    });

Під капотом`cache`помічник збирається зателефонувати`get`метод для класу, що лежить в основі`Cache`фасад. Отже, незважаючи на те, що ми використовуємо допоміжну функцію, ми можемо написати наступний тест, щоб переконатися, що метод був викликаний з аргументом, який ми очікували:

    use Illuminate\Support\Facades\Cache;

    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $response = $this->get('/cache');

        $response->assertSee('value');
    }

<a name="how-facades-work"></a>

## Як працюють фасади

У програмі Laravel Facade- це клас, який забезпечує доступ до об’єкта з контейнера. Машина, яка робить цю роботу, знаходиться в`Facade`клас. Фасади Laravel та будь-які спеціальні фасади, які ви створюєте, розширять основу`Illuminate\Support\Facades\Facade`клас.

`Facade`базовий клас використовує`__callStatic()`magic-метод для відкладання викликів з вашого фасаду до об'єкта, вирішеного з контейнера. У наведеному нижче прикладі здійснюється виклик кеш-системи Laravel. Поглянувши на цей код, можна припустити, що статичний метод`get`викликається на`Cache`клас:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Cache;

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
            $user = Cache::get('user:'.$id);

            return view('profile', ['user' => $user]);
        }
    }

Зверніть увагу, що вгорі файлу ми "імпортуємо" файл`Cache`фасад. Цей Facadeслужить проксі для доступу до базової реалізації`Illuminate\Contracts\Cache\Factory`інтерфейс. Будь-які дзвінки, які ми робимо за допомогою фасаду, будуть передані базовому екземпляру кеш-служби Laravel.

Якщо ми подивимось на це`Illuminate\Support\Facades\Cache`клас, ви побачите, що не існує статичного методу`get`:

    class Cache extends Facade
    {
        /**
         * Get the registered name of the component.
         *
         * @return string
         */
        protected static function getFacadeAccessor() { return 'cache'; }
    }

Натомість,`Cache`Facadeпродовжує основу`Facade`клас і визначає метод`getFacadeAccessor()`. Завдання цього методу - повернути ім’я Binding контейнера служби. Коли користувач посилається на будь-який статичний метод на`Cache`фасаду, Laravel вирішує`cache`прив'язка з[службовий контейнер](/docs/{{version}}/container)і запускає запитаний метод (у цьому випадку`get`) проти цього об’єкта.

<a name="real-time-facades"></a>

## Фасади в режимі реального часу

Використовуючи фасади в режимі реального часу, ви можете поводитися з будь-яким класом у вашій програмі так, ніби це фасад. Щоб проілюструвати, як це можна використовувати, давайте розглянемо альтернативу. Наприклад, припустимо наш`Podcast`модель має`publish`метод. Однак, щоб опублікувати подкаст, нам потрібно ввести a`Publisher`примірник:

    <?php

    namespace App\Models;

    use App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;

    class Podcast extends Model
    {
        /**
         * Publish the podcast.
         *
         * @param  Publisher  $publisher
         * @return void
         */
        public function publish(Publisher $publisher)
        {
            $this->update(['publishing' => now()]);

            $publisher->publish($this);
        }
    }

Введення реалізації видавця в метод дозволяє нам легко протестувати метод ізольовано, оскільки ми можемо знущатись над введеним видавцем. Однак це вимагає від нас, щоб ми завжди передавали екземпляр видавця кожного разу, коли ми телефонуємо до`publish`метод. Використовуючи фасади в режимі реального часу, ми можемо підтримувати однакову перевірочність, не вимагаючи явного проходження а`Publisher`екземпляр. Щоб сформувати Facadeу реальному часі, додайте до простору імен імпортованого класу префікс`Facades`:

    <?php

    namespace App\Models;

    use Facades\App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;

    class Podcast extends Model
    {
        /**
         * Publish the podcast.
         *
         * @return void
         */
        public function publish()
        {
            $this->update(['publishing' => now()]);

            Publisher::publish($this);
        }
    }

Коли використовується Facadeу реальному часі, реалізація видавця буде вирішена з контейнера служби за допомогою частини інтерфейсу або імені класу, що з'являється після`Facades`префікс. Під час тестування ми можемо використовувати вбудовані помічники фасадного тестування Laravel для глузування над цим викликом методу:

    <?php

    namespace Tests\Feature;

    use App\Models\Podcast;
    use Facades\App\Contracts\Publisher;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Tests\TestCase;

    class PodcastTest extends TestCase
    {
        use RefreshDatabase;

        /**
         * A test example.
         *
         * @return void
         */
        public function test_podcast_can_be_published()
        {
            $podcast = Podcast::factory()->create();

            Publisher::shouldReceive('publish')->once()->with($podcast);

            $podcast->publish();
        }
    }

<a name="facade-class-reference"></a>

## Class Reference фасадів

Нижче ви знайдете кожен Facadeта його основний клас. Це корисний інструмент для швидкого вивчення документації API для даного кореня фасаду.[прив'язка службового контейнера](/docs/{{version}}/container)ключ також включений, де це можливо.

| Фасад                   | Клас                                                                                                                                                 | Прив'язка контейнера послуг |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------- |
| Додаток                 | [Illuminate \\ Foundation \\ Application](https://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html)                                | `app`                       |
| Ремісник                | [Висвітлити \\ Contracts \\ Console \\ Kernel](https://laravel.com/api/{{version}}/Illuminate/Contracts/Console/Kernel.html)                         | `artisan`                   |
| Авт                     | [Висвітлити \\ Auth \\ AuthManager](https://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html)                                            | `auth`                      |
| Авторизація (екземпляр) | [Висвітлити \\ Контракти \\ Авторизація \\ Захист](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Guard.html)                         | `auth.driver`               |
| Blade                    | [Висвітлити \\ Переглянути \\ Компілятори \\ BladeCompiler](https://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html)        | `blade.compiler`            |
| Broadcast              | [Висвітлити \\ Контракти \\ Broadcast \\ Фабрика](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Factory.html)               |                             |
| Broadcast (екземпляр)  | [Висвітлити \\ Contracts \\ Broadcasting \\ Broadcaster](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Broadcaster.html)     |                             |
| Автобус                 | [Висвітлити \\ Contracts \\ Bus \\ Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html)                         |                             |
| Кеш                     | [Висвітлити \\ Cache \\ CacheManager](https://laravel.com/api/{{version}}/Illuminate/Cache/CacheManager.html)                                        | `cache`                     |
| Кеш (екземпляр)         | [Illuminate \\ Cache \\ Repository](https://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html)                                            | `cache.store`               |
| Налаштувати             | [Illuminate \\ Config \\ Repository](https://laravel.com/api/{{version}}/Illuminate/Config/Repository.html)                                          | `config`                    |
| Cookies                  | [Висвітлює \\ Cookie \\ CookieJar](https://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html)                                             | `cookie`                    |
| Склеп                   | [Висвітлити \\ Шифрування \\ Шифрування](https://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html)                                   | `encrypter`                 |
| БД                      | [Висвітлити \\ Database \\ DatabaseManager](https://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html)                            | `db`                        |
| DB (Екземпляр)          | [Висвітлити \\ База даних \\ З'єднання](https://laravel.com/api/{{version}}/Illuminate/Database/Connection.html)                                     | `db.connection`             |
| Подія                   | [Висвітлити \\ Події \\ Диспетчер](https://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html)                                            | `events`                    |
| Файл                    | [Висвітлити \\ Файлова система \\ Файлова система](https://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html)                        | `files`                     |
| Ворота                  | [Висвітлити \\ Contracts \\ Auth \\ Access \\ Gate](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Access/Gate.html)                  |                             |
| Хеш                     | [Освітлити \\ Контракти \\ Хешування \\ Хеш](https://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html)                           | `hash`                      |
| Http                    | [Висвітлити \\ Http \\ Client \\ Factory](https://laravel.com/api/{{version}}/Illuminate/Http/Client/Factory.html)                                   |                             |
| Язик                    | [Висвітлити \\ Переклад \\ Перекладач](https://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html)                                   | `translator`                |
| Журнал                  | [Висвітлити \\ Log \\ LogManager](https://laravel.com/api/{{version}}/Illuminate/Log/LogManager.html)                                                | `log`                       |
| Пошта                   | [Висвітлити \\ Mail \\ Mailer](https://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html)                                                      | `mailer`                    |
| Повідомлення            | [Висвітлити \\ Notification \\ ChannelManager](https://laravel.com/api/{{version}}/Illuminate/Notifications/ChannelManager.html)                       |                             |
| Пароль                  | [Висвітлити \\ Auth \\ Passwords \\ PasswordBrokerManager](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBrokerManager.html) | `auth.password`             |
| Пароль (екземпляр)      | [Висвітлити \\ Auth \\ Passwords \\ PasswordBroker](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html)               | `auth.password.broker`      |
| Черга                   | [Висвітлити \\ Queue \\ QueueManager](https://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html)                                        | `queue`                     |
| Черга (екземпляр)       | [Висвітлити \\ Contracts \\ Queue \\ Queue](https://laravel.com/api/{{version}}/Illuminate/Contracts/Queue/Queue.html)                               | `queue.connection`          |
| Черга (базовий клас)    | [Висвітлити \\ Черга \\ Черга](https://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html)                                                      |                             |
| Переспрямування         | [Висвітлити \\ Routing \\ Перенаправлення](https://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html)                             | `redirect`                  |
| Редіс                   | [Висвітлити \\ Redis \\ RedisManager](https://laravel.com/api/{{version}}/Illuminate/Redis/RedisManager.html)                                        | `redis`                     |
| Redis (екземпляр)       | [Підсвічує \\ Redis \\ Connections \\ Connection](https://laravel.com/api/{{version}}/Illuminate/Redis/Connections/Connection.html)                  | `redis.connection`          |
| Запит                   | [Висвітлити \\ Http \\ Запит](https://laravel.com/api/{{version}}/Illuminate/Http/Request.html)                                                      | `request`                   |
| Відповідь               | [Висвітлити \\ Contracts \\ Routing \\ ResponseFactory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html)       |                             |
| Відповідь (екземпляр)   | [Висвітлити \\ Http \\ Response](https://laravel.com/api/{{version}}/Illuminate/Http/Response.html)                                                  |                             |
| Маршрут                 | [Підсвічує \\ Routing \\ Маршрутизатор](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)                                    | `router`                    |
| Схема                   | [Висвітлити \\ База даних \\ Схема \\ Конструктор](https://laravel.com/api/{{version}}/Illuminate/Database/Schema/Builder.html)                      |                             |
| Сесія                   | [Висвітлити \\ Session \\ SessionManager](https://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html)                                | `session`                   |
| Session (Instance)      | [Освітлити \\ Session \\ Store](https://laravel.com/api/{{version}}/Illuminate/Session/Store.html)                                                   | `session.store`             |
| Зберігання              | [Висвітлити \\ Filesystem \\ FilesystemManager](https://laravel.com/api/{{version}}/Illuminate/Filesystem/FilesystemManager.html)                    | `filesystem`                |
| Зберігання (екземпляр)  | [Висвітлити \\ Contracts \\ Filesystem \\ Filesystem](https://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Filesystem.html)           | `filesystem.disk`           |
| URL                     | [Висвітлити \\ Routing \\ UrlGenerator](https://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html)                              | `url`                       |
| Валідатор               | [Висвітлити \\ Перевірка \\ Factory](https://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html)                                           | `validator`                 |
| Валідатор (екземпляр)   | [Висвітлити \\ перевірку \\ перевірку](https://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html)                                     |                             |
| Переглянути             | [Висвітлити \\ Переглянути \\ Factory](https://laravel.com/api/{{version}}/Illuminate/View/Factory.html)                                               | `view`                      |
| Вид (Екземпляр)         | [Висвітлити \\ Переглянути \\ Переглянути](https://laravel.com/api/{{version}}/Illuminate/View/View.html)                                            |                             |
