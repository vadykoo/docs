# Контракти

-   [Вступ](#introduction)
    -   [Контракти проти Фасади](#contracts-vs-facades)
-   [Коли використовувати контракти](#when-to-use-contracts)
    -   [Слабке зчеплення](#loose-coupling)
    -   [Простота](#simplicity)
-   [Як користуватися контрактами](#how-to-use-contracts)
-   [Посилання на контракт](#contract-reference)

<a name="introduction"></a>

## Вступ

Контракти Laravel - це набір інтерфейсів, що визначають основні послуги, що надаються в рамках. Наприклад,`Illuminate\Contracts\Queue\Queue`контракт визначає методи, необхідні для черги робочих місць, тоді як`Illuminate\Contracts\Mail\Mailer`Договір визначає методи, необхідні для надсилання електронної пошти.

Кожен контракт має відповідне виконання, передбачене рамками. Наприклад, Laravel забезпечує реалізацію черги з різноманітними драйверами та реалізацію поштової пошти, яка працює від[SwiftMailer](https://swiftmailer.symfony.com/).

Усі контракти Laravel живуть в[власне сховище GitHub](https://github.com/illuminate/contracts). Це забезпечує швидкий орієнтир для всіх доступних контрактів, а також єдиний розв’язаний пакет, який може бути використаний розробниками пакетів.

<a name="contracts-vs-facades"></a>

### Контракти проти Фасади

Ларавеля[фасади](/docs/{{version}}/facades)і допоміжні функції забезпечують простий спосіб використання послуг Laravel без необхідності вводити підказки та вирішувати контракти поза контейнером служби. У більшості випадків кожен фасад має еквівалентний контракт.

На відміну від фасадів, які не вимагають, щоб ви вимагали їх у конструкторі вашого класу, контракти дозволяють вам визначати явні залежності для ваших класів. Деякі розробники вважають за краще чітко визначати свої залежності таким чином і тому воліють використовувати контракти, тоді як інші розробники користуються зручністю фасадів.

> {tip} Більшість додатків будуть чудовими, незалежно від того, віддаєте перевагу фасадам чи контрактам. Однак, якщо ви створюєте пакет, вам настійно слід подумати про використання контрактів, оскільки їх буде простіше перевірити в контексті пакета.

<a name="when-to-use-contracts"></a>

## Коли використовувати контракти

Як обговорювалося в інших місцях, більша частина рішення про використання контрактів або фасадів буде зводитися до особистого смаку та смаків вашої команди розробників. Як контракти, так і фасади можуть бути використані для створення надійних, добре перевірених додатків Laravel. Поки ви зосереджуєте відповідальність свого класу, ви помітите дуже мало практичних відмінностей між використанням контрактів та фасадів.

Однак у вас все ще може бути кілька запитань щодо контрактів. Наприклад, навіщо взагалі використовувати інтерфейси? Чи не складніше користуватися інтерфейсами? Давайте розберемо причини використання інтерфейсів до таких заголовків: вільне зчеплення та простота.

<a name="loose-coupling"></a>

### Слабке зчеплення

Спочатку давайте розглянемо деякий код, який тісно пов’язаний із реалізацією кешу. Розглянемо наступне:

    <?php

    namespace App\Orders;

    class Repository
    {
        /**
         * The cache instance.
         */
        protected $cache;

        /**
         * Create a new repository instance.
         *
         * @param  \SomePackage\Cache\Memcached  $cache
         * @return void
         */
        public function __construct(\SomePackage\Cache\Memcached $cache)
        {
            $this->cache = $cache;
        }

        /**
         * Retrieve an Order by ID.
         *
         * @param  int  $id
         * @return Order
         */
        public function find($id)
        {
            if ($this->cache->has($id)) {
                //
            }
        }
    }

У цьому класі код тісно пов'язаний із заданою реалізацією кешу. Це тісно пов’язано, оскільки ми залежамо від конкретного класу кешу від постачальника пакетів. Якщо API цього пакета зміниться, повинен змінитися і наш код.

Аналогічним чином, якщо ми хочемо замінити нашу базову технологію кешування (Memcached) іншою технологією (Redis), нам знову доведеться змінити наш репозиторій. Наш репозитарій не повинен мати стільки знань щодо того, хто надає їм дані або як вони надають їх.

**Замість цього підходу ми можемо вдосконалити наш код, залежачи від простого агностичного інтерфейсу постачальника:**

    <?php

    namespace App\Orders;

    use Illuminate\Contracts\Cache\Repository as Cache;

    class Repository
    {
        /**
         * The cache instance.
         */
        protected $cache;

        /**
         * Create a new repository instance.
         *
         * @param  Cache  $cache
         * @return void
         */
        public function __construct(Cache $cache)
        {
            $this->cache = $cache;
        }
    }

Тепер код не пов'язаний ні з яким конкретним постачальником, ані навіть з Laravel. Оскільки пакет контрактів не містить реалізації та залежностей, ви можете легко написати альтернативну реалізацію будь-якого даного контракту, що дозволяє замінити реалізацію кешу, не змінюючи жодного коду, що споживає кеш.

<a name="simplicity"></a>

### Простота

Коли всі послуги Laravel акуратно визначені в простих інтерфейсах, дуже легко визначити функціональність, пропоновану даною службою.**Контракти служать стислою документацією до особливостей фреймворку.**

Крім того, коли ви залежате від простих інтерфейсів, ваш код легше розуміти та підтримувати. Замість того, щоб відстежувати, які методи доступні для вас у великому складному класі, ви можете звернутися до простого, чистого інтерфейсу.

<a name="how-to-use-contracts"></a>

## Як користуватися контрактами

Отже, як отримати виконання контракту? Насправді це досить просто.

Багато типів занять у Laravel вирішуються через[службовий контейнер](/docs/{{version}}/container), включаючи контролери, прослуховувачі подій, проміжне програмне забезпечення, завдання в черзі та навіть закриття маршрутів. Отже, щоб отримати реалізацію контракту, ви можете просто "натякнути" на інтерфейс у конструкторі класу, що вирішується.

Наприклад, погляньте на слухача цієї події:

    <?php

    namespace App\Listeners;

    use App\Events\OrderWasPlaced;
    use App\Models\User;
    use Illuminate\Contracts\Redis\Factory;

    class CacheOrderInformation
    {
        /**
         * The Redis factory implementation.
         */
        protected $redis;

        /**
         * Create a new event handler instance.
         *
         * @param  Factory  $redis
         * @return void
         */
        public function __construct(Factory $redis)
        {
            $this->redis = $redis;
        }

        /**
         * Handle the event.
         *
         * @param  OrderWasPlaced  $event
         * @return void
         */
        public function handle(OrderWasPlaced $event)
        {
            //
        }
    }

Коли прослуховувач подій вирішено, контейнер служби прочитає підказки типу в конструкторі класу та введе відповідне значення. Щоб дізнатися більше про реєстрацію речей у службовому контейнері, ознайомтесь[його документація](/docs/{{version}}/container).

<a name="contract-reference"></a>

## Посилання на контракт

У цій таблиці наведено короткі посилання на всі контракти Laravel та їх еквівалентні фасади:

| Договір                                                                                                                                                         | Список літератури Фасад   |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------- |
| [Illuminate \\ Contracts \\ Auth \\ Access \\ Authorizable](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Access/Authorizable.php)              |                           |
| [Висвітлити \\ Contracts \\ Auth \\ Access \\ Gate](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Access/Gate.php)                              | `Gate`                    |
| [Illuminate \\ Contracts \\ Auth \\ Authenticatable](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Authenticatable.php)                         |                           |
| [Висвітлити \\ Contracts \\ Auth \\ CanResetPassword](https://github.com/illuminate/contracts/blob/{{version}}/Auth/CanResetPassword.php)                       |                           |
| [Illuminate \\ Contracts \\ Auth \\ Factory](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Factory.php)                                         | `Auth`                    |
| [Висвітлити \\ Контракти \\ Авторизація \\ Захист](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Guard.php)                                     | `Auth::guard()`           |
| [Висвітлити \\ Contracts \\ Auth \\ PasswordBroker](https://github.com/illuminate/contracts/blob/{{version}}/Auth/PasswordBroker.php)                           | `Password::broker()`      |
| [Висвітлити \\ Contracts \\ Auth \\ PasswordBrokerFactory](https://github.com/illuminate/contracts/blob/{{version}}/Auth/PasswordBrokerFactory.php)             | `Password`                |
| [Illuminate \\ Contracts \\ Auth \\ StatefulGuard](https://github.com/illuminate/contracts/blob/{{version}}/Auth/StatefulGuard.php)                             |                           |
| [Висвітлити \\ Contracts \\ Auth \\ SupportsBasicAuth](https://github.com/illuminate/contracts/blob/{{version}}/Auth/SupportsBasicAuth.php)                     |                           |
| [Висвітлити \\ Contracts \\ Auth \\ UserProvider](https://github.com/illuminate/contracts/blob/{{version}}/Auth/UserProvider.php)                               |                           |
| [Висвітлити \\ Contracts \\ Bus \\ Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Bus/Dispatcher.php)                                     | `Bus`                     |
| [Висвітлити \\ Contracts \\ Bus \\ QueueingDispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Bus/QueueingDispatcher.php)                     | `Bus::dispatchToQueue()`  |
| [Висвітлити \\ Контракти \\ Трансляція \\ Фабрика](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/Factory.php)                           | `Broadcast`               |
| [Висвітлити \\ Contracts \\ Broadcasting \\ Broadcaster](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/Broadcaster.php)                 | `Broadcast::connection()` |
| [Висвітлити \\ Contracts \\ Broadcasting \\ ShouldBroadcast](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/ShouldBroadcast.php)         |                           |
| [Висвітлити \\ Contracts \\ Broadcasting \\ ShouldBroadcastNow](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/ShouldBroadcastNow.php)   |                           |
| [Висвітлити \\ Contracts \\ Cache \\ Factory](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Factory.php)                                       | `Cache`                   |
| [Висвітлити \\ Contracts \\ Cache \\ Lock](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Lock.php)                                             |                           |
| [Illuminate \\ Contracts \\ Cache \\ LockProvider](https://github.com/illuminate/contracts/blob/{{version}}/Cache/LockProvider.php)                             |                           |
| [Illuminate \\ Contracts \\ Cache \\ Repository](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Repository.php)                                 | `Cache::driver()`         |
| [Висвітлити \\ Contracts \\ Cache \\ Store](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Store.php)                                           |                           |
| [Illuminate \\ Contracts \\ Config \\ Repository](https://github.com/illuminate/contracts/blob/{{version}}/Config/Repository.php)                               | `Config`                  |
| [Illuminate \\ Contracts \\ Console \\ Application](https://github.com/illuminate/contracts/blob/{{version}}/Console/Application.php)                           |                           |
| [Висвітлити \\ Contracts \\ Console \\ Kernel](https://github.com/illuminate/contracts/blob/{{version}}/Console/Kernel.php)                                     | `Artisan`                 |
| [Висвітлити \\ Contracts \\ Container \\ Container](https://github.com/illuminate/contracts/blob/{{version}}/Container/Container.php)                           | `App`                     |
| [Освітлити \\ Contracts \\ Cookie \\ Factory](https://github.com/illuminate/contracts/blob/{{version}}/Cookie/Factory.php)                                      | `Cookie`                  |
| [Illuminate \\ Contracts \\ Cookie \\ QueueingFactory](https://github.com/illuminate/contracts/blob/{{version}}/Cookie/QueueingFactory.php)                     | `Cookie::queue()`         |
| [Висвітлити \\ Contracts \\ Database \\ ModelIdentifier](https://github.com/illuminate/contracts/blob/{{version}}/Database/ModelIdentifier.php)                 |                           |
| [Illuminate \\ Contracts \\ Debug \\ ExceptionHandler](https://github.com/illuminate/contracts/blob/{{version}}/Debug/ExceptionHandler.php)                     |                           |
| [Висвітлити \\ Contracts \\ Encryption \\ Encrypter](https://github.com/illuminate/contracts/blob/{{version}}/Encryption/Encrypter.php)                         | `Crypt`                   |
| [Висвітлити \\ Контракти \\ Події \\ Диспетчер](https://github.com/illuminate/contracts/blob/{{version}}/Events/Dispatcher.php)                                 | `Event`                   |
| [Illuminate \\ Contracts \\ Filesystem \\ Cloud](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Cloud.php)                                 | `Storage::cloud()`        |
| [Illuminate \\ Contracts \\ Filesystem \\ Factory](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Factory.php)                             | `Storage`                 |
| [Висвітлити \\ Contracts \\ Filesystem \\ Filesystem](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Filesystem.php)                       | `Storage::disk()`         |
| [Illuminate \\ Contracts \\ Foundation \\ Application](https://github.com/illuminate/contracts/blob/{{version}}/Foundation/Application.php)                     | `App`                     |
| [Освітлити \\ Контракти \\ Хешування \\ Хеш](https://github.com/illuminate/contracts/blob/{{version}}/Hashing/Hasher.php)                                       | `Hash`                    |
| [Висвітлити \\ Contracts \\ Http \\ Kernel](https://github.com/illuminate/contracts/blob/{{version}}/Http/Kernel.php)                                           |                           |
| [Висвітлити \\ Contracts \\ Mail \\ MailQueue](https://github.com/illuminate/contracts/blob/{{version}}/Mail/MailQueue.php)                                     | `Mail::queue()`           |
| [Висвітлює \\ Контракти \\ Пошта \\ Доступні](https://github.com/illuminate/contracts/blob/{{version}}/Mail/Mailable.php)                                       |                           |
| [Висвітлити \\ Contracts \\ Mail \\ Mailer](https://github.com/illuminate/contracts/blob/{{version}}/Mail/Mailer.php)                                           | `Mail`                    |
| [Висвітлити \\ Контракти \\ Повідомлення \\ Диспетчер](https://github.com/illuminate/contracts/blob/{{version}}/Notifications/Dispatcher.php)                   | `Notification`            |
| [Освітлити \\ Contracts \\ Notifications \\ Factory](https://github.com/illuminate/contracts/blob/{{version}}/Notifications/Factory.php)                        | `Notification`            |
| [Illuminate \\ Contracts \\ Pagination \\ LengthAwarePaginator](https://github.com/illuminate/contracts/blob/{{version}}/Pagination/LengthAwarePaginator.php)   |                           |
| [Висвітлити \\ Contracts \\ Pagination \\ Paginator](https://github.com/illuminate/contracts/blob/{{version}}/Pagination/Paginator.php)                         |                           |
| [Освітлити \\ Contracts \\ Pipeline \\ Hub](https://github.com/illuminate/contracts/blob/{{version}}/Pipeline/Hub.php)                                          |                           |
| [Освітлити \\ Контракти \\ Трубопровід \\ Трубопровід](https://github.com/illuminate/contracts/blob/{{version}}/Pipeline/Pipeline.php)                          |                           |
| [Illuminate \\ Contracts \\ Queue \\ EntityResolver](https://github.com/illuminate/contracts/blob/{{version}}/Queue/EntityResolver.php)                         |                           |
| [Висвітлити \\ Contracts \\ Queue \\ Factory](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Factory.php)                                       | `Queue`                   |
| [Illuminate \\ Contracts \\ Queue \\ Job](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Job.php)                                               |                           |
| [Висвітлити \\ Contracts \\ Queue \\ Monitor](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Monitor.php)                                       | `Queue`                   |
| [Висвітлити \\ Contracts \\ Queue \\ Queue](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Queue.php)                                           | `Queue::connection()`     |
| [Висвітлити \\ Contracts \\ Queue \\ QueueableCollection](https://github.com/illuminate/contracts/blob/{{version}}/Queue/QueueableCollection.php)               |                           |
| [Illuminate \\ Contracts \\ Queue \\ QueueableEntity](https://github.com/illuminate/contracts/blob/{{version}}/Queue/QueueableEntity.php)                       |                           |
| [Висвітлити \\ Contracts \\ Queue \\ ShouldQueue](https://github.com/illuminate/contracts/blob/{{version}}/Queue/ShouldQueue.php)                               |                           |
| [Illuminate \\ Contracts \\ Redis \\ Factory](https://github.com/illuminate/contracts/blob/{{version}}/Redis/Factory.php)                                       | `Redis`                   |
| [Illuminate \\ Contracts \\ Routing \\ BindingRegistrar](https://github.com/illuminate/contracts/blob/{{version}}/Routing/BindingRegistrar.php)                 | `Route`                   |
| [Illuminate \\ Contracts \\ Routing \\ Registrar](https://github.com/illuminate/contracts/blob/{{version}}/Routing/Registrar.php)                               | `Route`                   |
| [Висвітлити \\ Contracts \\ Routing \\ ResponseFactory](https://github.com/illuminate/contracts/blob/{{version}}/Routing/ResponseFactory.php)                   | `Response`                |
| [Висвітлити \\ Contracts \\ Routing \\ UrlGenerator](https://github.com/illuminate/contracts/blob/{{version}}/Routing/UrlGenerator.php)                         | `URL`                     |
| [Висвітлити \\ Контракти \\ Маршрутизація \\ UrlRoutable](https://github.com/illuminate/contracts/blob/{{version}}/Routing/UrlRoutable.php)                     |                           |
| [Висвітлити \\ Контракти \\ Сесія \\ Сесія](https://github.com/illuminate/contracts/blob/{{version}}/Session/Session.php)                                       | `Session::driver()`       |
| [Illuminate \\ Contracts \\ Support \\ Arrayable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Arrayable.php)                               |                           |
| [Висвітлити \\ Contracts \\ Support \\ Htmlable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Htmlable.php)                                 |                           |
| [Illuminate \\ Contracts \\ Support \\ Jsonable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Jsonable.php)                                 |                           |
| [Illuminate \\ Contracts \\ Support \\ MessageBag](https://github.com/illuminate/contracts/blob/{{version}}/Support/MessageBag.php)                             |                           |
| [Illuminate \\ Contracts \\ Support \\ MessageProvider](https://github.com/illuminate/contracts/blob/{{version}}/Support/MessageProvider.php)                   |                           |
| [Illuminate \\ Contracts \\ Support \\ Renderable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Renderable.php)                             |                           |
| [Illuminate \\ Contracts \\ Support \\ Responsible](https://github.com/illuminate/contracts/blob/{{version}}/Support/Responsable.php)                           |                           |
| [Висвітлити \\ Contracts \\ Translation \\ Loader](https://github.com/illuminate/contracts/blob/{{version}}/Translation/Loader.php)                             |                           |
| [Illuminate \\ Contracts \\ Translation \\ Translator](https://github.com/illuminate/contracts/blob/{{version}}/Translation/Translator.php)                     | `Lang`                    |
| [Illuminate \\ Contracts \\ Validation \\ Factory](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Factory.php)                             | `Validator`               |
| [Висвітлити \\ Contracts \\ Validation \\ ImplicitRule](https://github.com/illuminate/contracts/blob/{{version}}/Validation/ImplicitRule.php)                   |                           |
| [Висвітлити \\ Контракти \\ Перевірка \\ Правило](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Rule.php)                                 |                           |
| [Illuminate \\ Contracts \\ Validation \\ ValidatesWhenResolved](https://github.com/illuminate/contracts/blob/{{version}}/Validation/ValidatesWhenResolved.php) |                           |
| [Illuminate \\ Contracts \\ Validation \\ Validator](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Validator.php)                         | `Validator::make()`       |
| [Висвітлити \\ Contracts \\ View \\ Engine](https://github.com/illuminate/contracts/blob/{{version}}/View/Engine.php)                                           |                           |
| [Висвітлити \\ Contracts \\ View \\ Factory](https://github.com/illuminate/contracts/blob/{{version}}/View/Factory.php)                                         | `View`                    |
| [Висвітлити \\ Contracts \\ View \\ View](https://github.com/illuminate/contracts/blob/{{version}}/View/View.php)                                               | `View::make()`            |
