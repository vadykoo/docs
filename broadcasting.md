# Мовлення

-   [Вступ](#introduction)
    -   [Конфігурація](#configuration)
    -   [Передумови драйвера](#driver-prerequisites)
-   [Огляд концепції](#concept-overview)
    -   [Використання прикладу програми](#using-example-application)
-   [Визначення трансляційних подій](#defining-broadcast-events)
    -   [Назва трансляції](#broadcast-name)
    -   [B Roa d або st](#broadcast-data)
    -   [Черга трансляції](#broadcast-queue)
    -   [Умови мовлення](#broadcast-conditions)
-   [Авторизація каналів](#authorizing-channels)
    -   [Визначення шляхів авторизації](#defining-authorization-routes)
    -   [Визначення зворотних викликів авторизації](#defining-authorization-callbacks)
    -   [Визначення класів каналів](#defining-channel-classes)
-   [Трансляція подій](#broadcasting-events)
    -   [Тільки іншим](#only-to-others)
-   [Отримання трансляцій](#receiving-broadcasts)
    -   [Встановлення Laravel Echo](#installing-laravel-echo)
    -   [Прослуховування подій](#listening-for-events)
    -   [Вихід із каналу](#leaving-a-channel)
    -   [Простори імен](#namespaces)
-   [Канали присутності](#presence-channels)
    -   [Авторизація каналів присутності](#authorizing-presence-channels)
    -   [Joining Presence Channels](#joining-presence-channels)
    -   [Трансляція на канали присутності](#broadcasting-to-presence-channels)
-   [Події клієнта](#client-events)
-   [Notifications](#notifications)

<a name="introduction"></a>

## Вступ

У багатьох сучасних веб-додатках WebSockets використовуються для реалізації в режимі реального часу оновлюваних користувацьких інтерфейсів. Коли деякі дані оновлюються на сервері, повідомлення, як правило, надсилається через з'єднання WebSocket для обробки клієнтом. Це забезпечує більш надійну та ефективну альтернативу постійному опитуванню вашої програми для внесення змін.

Щоб допомогти вам у створенні таких типів програм, Laravel полегшує "трансляцію" ваших[події](/docs/{{version}}/events)через з'єднання WebSocket. Трансляція подій Laravel дозволяє обмінюватися однаковими іменами подій між кодом на стороні сервера та вашим додатком JavaScript на стороні клієнта.

> {tip} Перш ніж заглибитися в трансляцію подій, переконайтесь, що ви прочитали всю документацію щодо Laravel[події та слухачі](/docs/{{version}}/events).

<a name="configuration"></a>

### Конфігурація

Усі конфігурації трансляції подій вашої програми зберігаються в`config/broadcasting.php`файл конфігурації. Laravel підтримує декілька драйверів мовлення:[Канали штовхача](https://pusher.com/channels),[Редіс](/docs/{{version}}/redis), і a`log`драйвер для місцевого розвитку та налагодження. Крім того, a`null`включений драйвер, що дозволяє повністю відключити мовлення. Приклад конфігурації наведено для кожного з цих драйверів у`config/broadcasting.php`файл конфігурації.

<a name="broadcast-service-provider"></a>

#### Постачальник послуг трансляції

Перш ніж транслювати будь-які події, спочатку потрібно зареєструвати`App\Providers\BroadcastServiceProvider`. У нових програмах Laravel вам потрібно лише прокоментувати цього постачальника в`providers`масив вашого`config/app.php`файл конфігурації. Цей постачальник дозволить вам реєструвати маршрути авторизації мовлення та зворотні дзвінки.

<a name="csrf-token"></a>

#### Токен CSRF

[Laravel Echo](#installing-laravel-echo)потребуватиме доступу до маркера CSRF поточної сесії. Ви повинні перевірити, що ваша програма`head`Елемент HTML визначає a`meta`тег, що містить маркер CSRF:

    <meta name="csrf-token" content="{{ csrf_token() }}">

<a name="driver-prerequisites"></a>

### Передумови драйвера

<a name="pusher-channels"></a>

#### Канали штовхача

Якщо ви транслюєте свої події[Канали штовхача](https://pusher.com/channels), вам слід встановити PHP SDK Pusher Channels за допомогою менеджера пакетів Composer:

    composer require pusher/pusher-php-server "~4.0"

Далі слід налаштувати облікові дані каналів у`config/broadcasting.php`файл конфігурації. Приклад конфігурації каналів уже включений у цей файл, що дозволяє швидко вказати ключ, секрет та ідентифікатор програми.`config/broadcasting.php`файлів`pusher`конфігурація також дозволяє вказати додаткові`options`які підтримуються Каналами, такими як кластер:

    'options' => [
        'cluster' => 'eu',
        'useTLS' => true
    ],

При використанні каналів та[Laravel Echo](#installing-laravel-echo), слід вказати`pusher`як бажаний мовник, коли створює екземпляр Echo у вашому`resources/js/bootstrap.js`файл:

    import Echo from "laravel-echo";

    window.Pusher = require('pusher-js');

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key'
    });

Нарешті, вам потрібно буде змінити драйвер мовлення на`pusher`у вашому`.env`файл:

    BROADCAST_DRIVER=pusher

<a name="pusher-compatible-laravel-websockets"></a>

#### Сумісні веб-розетки Laravel

[laravel-websockets](https://github.com/beyondcode/laravel-websockets)- це чистий пакет PHP, сумісний з Pusher, для Laravel. Цей пакет дозволяє використовувати всю потужність трансляції Laravel без зовнішнього постачальника веб-сокетів або Node. Для отримання додаткової інформації щодо встановлення та використання цього пакету, будь ласка, зверніться до його[official documentation](https://beyondco.de/docs/laravel-websockets).

<a name="redis"></a>

#### Редіс

Якщо ви використовуєте мовник Redis, вам слід або встановити розширення phpredis PHP через PECL, або встановити бібліотеку Predis через Composer:

    composer require predis/predis

Далі слід оновити драйвер мовлення до`redis`у вашому`.env`файл:

    BROADCAST_DRIVER=redis

Телеканал Redis буде транслювати повідомлення за допомогою функції pub / sub Redis; однак вам потрібно буде з'єднати це з сервером WebSocket, який може отримувати повідомлення від Redis і транслювати їх на ваші канали WebSocket.

Коли мовник Redis опублікує подію, вона буде опублікована на вказаних назвах каналів події, а корисним навантаженням буде кодований JSON рядок, що містить назву події,`data`корисне навантаження та користувача, який згенерував ідентифікатор сокета події (якщо застосовується).

<a name="socketio"></a>

#### Socket.IO

Якщо ви збираєтеся з'єднати мовника Redis із сервером Socket.IO, вам потрібно буде включити клієнтську бібліотеку Socket.IO у свою програму. Ви можете встановити його через менеджер пакунків NPM:

    npm install --save-dev socket.io-client@2

Далі вам потрібно буде створити екземпляр Echo за допомогою`socket.io`роз'єм та a`host`.

    import Echo from "laravel-echo"

    window.io = require('socket.io-client');

    window.Echo = new Echo({
        broadcaster: 'socket.io',
        host: window.location.hostname + ':6001'
    });

Нарешті, вам потрібно буде запустити сумісний сервер Socket.IO. Laravel не включає реалізацію сервера Socket.IO; проте сервер Socket.IO, керований спільнотою, наразі підтримується на[tlaverdure / laravel-echo-server](https://github.com/tlaverdure/laravel-echo-server)Репозиторій GitHub.

<a name="queue-prerequisites"></a>

#### Передумови черги

Перед трансляцією подій вам також потрібно буде налаштувати та запустити a[прослуховувач черги](/docs/{{version}}/queues). Усі трансляції подій здійснюються через завдання в черзі, так що час відгуку вашої програми серйозно не позначається.

<a name="concept-overview"></a>

## Огляд концепції

Трансляція подій Laravel дозволяє транслювати події Laravel на сервері до вашого додатку JavaScript на стороні клієнта, використовуючи підхід на основі драйверів до WebSockets. В даний час Laravel поставляється з[Канали штовхача](https://pusher.com/channels)та драйвери Redis. Події можуть бути легко використані на стороні клієнта за допомогою[Laravel Echo](#installing-laravel-echo)Пакет Javascript.

Події транслюються за "каналами", які можуть бути визначені як загальнодоступні чи приватні. Будь-який відвідувач вашої програми може підписатися на загальнодоступний канал без будь-якої автентифікації або дозволу; однак, щоб підписатися на приватний канал, користувач повинен пройти аутентифікацію та авторизуватися для прослуховування на цьому каналі.

> {tip} Якщо ви хочете скористатися відкритим кодом, PHP-керованою альтернативою Pusher, перегляньте[laravel-websockets](https://github.com/beyondcode/laravel-websockets)пакет.

<a name="using-example-application"></a>

### Використання прикладу програми

Перш ніж заглибитися в кожен компонент трансляції подій, давайте візьмемо огляд високого рівня, використовуючи приклад магазину електронної комерції. Ми не будемо обговорювати деталі налаштування[Канали штовхача](https://pusher.com/channels)або[Laravel Echo](#installing-laravel-echo)оскільки це буде детально обговорено в інших розділах цієї документації.

В нашому додатку, припустимо, у нас є сторінка, яка дозволяє користувачам переглядати статус доставки своїх замовлень. Припустимо також, що a`ShippingStatusUpdated`подія запускається, коли додаток обробляє оновлення статусу доставки:

    event(new ShippingStatusUpdated($update));

<a name="the-shouldbroadcast-interface"></a>

#### `ShouldBroadcast`Інтерфейс

Коли користувач переглядає одне із своїх замовлень, ми не хочемо, щоб йому потрібно було оновлювати сторінку, щоб переглянути оновлення стану. Натомість ми хочемо транслювати оновлення програми під час їх створення. Отже, нам потрібно позначити`ShippingStatusUpdated`подія з`ShouldBroadcast`інтерфейс. Це дасть вказівку Laravel транслювати подію під час її запуску:

    <?php

    namespace App\Events;

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
    use Illuminate\Queue\SerializesModels;

    class ShippingStatusUpdated implements ShouldBroadcast
    {
        /**
         * Information about the shipping status update.
         *
         * @var string
         */
        public $update;
    }

`ShouldBroadcast`інтерфейс вимагає, щоб наша подія визначила a`broadcastOn`метод. Цей метод відповідає за повернення каналів, на яких має транслюватись подія. Порожній заглушка цього методу вже визначена для сформованих класів подій, тому нам потрібно лише заповнити його деталі. Ми хочемо, щоб творець замовлення мав можливість переглядати оновлення стану, тому ми транслюватимемо подію на приватному каналі, пов’язаному із замовленням:

    /**
     * Get the channels the event should broadcast on.
     *
     * @return \Illuminate\Broadcasting\PrivateChannel
     */
    public function broadcastOn()
    {
        return new PrivateChannel('order.'.$this->update->order_id);
    }

<a name="example-application-authorizing-channels"></a>

#### Авторизація каналів

Пам’ятайте, користувачі повинні мати дозвіл на прослуховування на приватних каналах. Ми можемо визначити наші правила авторизації каналу в`routes/channels.php`файл. У цьому прикладі нам потрібно підтвердити, що будь-який користувач, який намагається прослуховувати приватний звук`order.1`канал насправді є творцем замовлення:

    Broadcast::channel('order.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

`channel`Метод приймає два аргументи: назву каналу та зворотний виклик, який повертається`true`або`false`вказує, чи дозволено користувачеві прослуховувати канал.

Усі зворотні виклики авторизації отримують поточно автентифікованого користувача як перший аргумент, а будь-які додаткові параметри підстановки - як наступні аргументи. У цьому прикладі ми використовуємо`{orderId}`заповнювач, щоб вказати, що частина «Ідентифікатор» назви каналу є символом підстановки.

<a name="listening-for-event-broadcasts"></a>

#### Прослуховування трансляцій подій

Далі, залишається лише прослухати подію в нашому додатку JavaScript. Ми можемо зробити це за допомогою Laravel Echo. Спочатку ми використаємо`private`спосіб підписатися на приватний канал. Тоді ми можемо використовувати`listen`метод прослуховування`ShippingStatusUpdated`подія. За замовчуванням усі загальнодоступні властивості події будуть включені до трансляції події:

    Echo.private(`order.${orderId}`)
        .listen('ShippingStatusUpdated', (e) => {
            console.log(e.update);
        });

<a name="defining-broadcast-events"></a>

## Визначення трансляційних подій

Щоб повідомити Laravel про те, що дана подія має транслюватися, застосуйте`Illuminate\Contracts\Broadcasting\ShouldBroadcast`інтерфейс класу подій. Цей інтерфейс вже імпортований до всіх класів подій, створених фреймворком, тому ви можете легко додати його до будь-якої з ваших подій.

`ShouldBroadcast`інтерфейс вимагає реалізації одного методу:`broadcastOn`.`broadcastOn`метод повинен повертати канал або масив каналів, на яких подія повинна транслювати. Канали мають бути екземплярами`Channel`,`PrivateChannel`, або`PresenceChannel`. Екземпляри`Channel`представляти загальнодоступні канали, на які може підписатися будь-який користувач`PrivateChannels`і`PresenceChannels`представляють приватні канали, які вимагають[авторизація каналу](#authorizing-channels):

    <?php

    namespace App\Events;

    use App\Models\User;
    use Illuminate\Broadcasting\Channel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
    use Illuminate\Queue\SerializesModels;

    class ServerCreated implements ShouldBroadcast
    {
        use SerializesModels;

        public $user;

        /**
         * Create a new event instance.
         *
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }

        /**
         * Get the channels the event should broadcast on.
         *
         * @return Channel|array
         */
        public function broadcastOn()
        {
            return new PrivateChannel('user.'.$this->user->id);
        }
    }

Тоді вам потрібно лише[звільнити подію](/docs/{{version}}/events)як зазвичай. Як тільки подію було звільнено, a[робота в черзі](/docs/{{version}}/queues)автоматично транслюватиме подію за вказаним драйвером мовлення.

<a name="broadcast-name"></a>

### Назва трансляції

За замовчуванням Laravel транслюватиме подію, використовуючи назву класу події. Однак ви можете налаштувати назву трансляції, вказавши a`broadcastAs`метод події:

    /**
     * The event's broadcast name.
     *
     * @return string
     */
    public function broadcastAs()
    {
        return 'server.created';
    }

Якщо ви налаштовуєте назву трансляції за допомогою`broadcastAs`методу, ви повинні переконатись, що зареєстрували свого слухача у ведучому`.`характер. Це дозволить Echo не додавати простір імен програми до події:

    .listen('.server.created', function (e) {
        ....
    });

<a name="broadcast-data"></a>

### B Roa d або st

Коли трансляція події, усі її`public`властивості автоматично серіалізуються і транслюються як корисне навантаження події, що дозволяє отримати доступ до будь-яких її загальнодоступних даних із програми JavaScript. Так, наприклад, якщо ваша подія має єдину публіку`$user`властивість, яка містить красномовну модель, корисне навантаження події буде:

    {
        "user": {
            "id": 1,
            "name": "Patrick Stewart"
            ...
        }
    }

Однак, якщо ви хочете мати більш чіткий контроль над вашим корисним навантаженням, ви можете додати`broadcastWith`метод до вашої події. Цей метод повинен повертати масив даних, які ви хочете транслювати як корисне навантаження події:

    /**
     * Get the data to broadcast.
     *
     * @return array
     */
    public function broadcastWith()
    {
        return ['id' => $this->user->id];
    }

<a name="broadcast-queue"></a>

### Черга трансляції

За замовчуванням кожна трансляція події розміщується в черзі за замовчуванням для з'єднання черги за замовчуванням, зазначеного у вашому`queue.php`файл конфігурації. Ви можете налаштувати чергу, що використовується мовником, визначивши a`broadcastQueue`властивість вашого класу подій. У цій властивості повинно бути вказано назву черги, яку ви хочете використовувати під час трансляції:

    /**
     * The name of the queue on which to place the event.
     *
     * @var string
     */
    public $broadcastQueue = 'your-queue-name';

Якщо ви хочете транслювати свою подію за допомогою`sync`чергу замість драйвера черги за замовчуванням, ви можете реалізувати`ShouldBroadcastNow`інтерфейс замість`ShouldBroadcast`:

    <?php

    use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

    class ShippingStatusUpdated implements ShouldBroadcastNow
    {
        //
    }

<a name="broadcast-conditions"></a>

### Умови мовлення

Іноді ви хочете транслювати свою подію, лише якщо задана умова відповідає дійсності. Ви можете визначити ці умови, додавши a`broadcastWhen`метод до вашого класу подій:

    /**
     * Determine if this event should broadcast.
     *
     * @return bool
     */
    public function broadcastWhen()
    {
        return $this->value > 100;
    }

<a name="authorizing-channels"></a>

## Авторизація каналів

Приватні канали вимагають від вас дозволу на те, що поточно зареєстрований користувач може насправді слухати канал. Це досягається шляхом надсилання HTTP-запиту до вашої програми Laravel з назвою каналу та надання можливості додатку визначити, чи може користувач слухати на цьому каналі. При використанні[Laravel Echo](#installing-laravel-echo), запит HTTP про авторизацію підписки на приватні канали буде зроблений автоматично; однак вам потрібно визначити правильні маршрути, щоб відповісти на ці запити.

<a name="defining-authorization-routes"></a>

### Визначення шляхів авторизації

На щастя, Laravel дозволяє легко визначити маршрути для відповіді на запити авторизації каналу. В`BroadcastServiceProvider`включений до вашої програми Laravel, ви побачите дзвінок до`Broadcast::routes`метод. Цей метод реєструє`/broadcasting/auth`маршрут для обробки запитів на авторизацію:

    Broadcast::routes();

`Broadcast::routes`метод автоматично розмістить свої маршрути в межах`web`група проміжного програмного забезпечення; однак ви можете передати методу масив атрибутів маршруту, якщо хочете налаштувати призначені атрибути:

    Broadcast::routes($attributes);

<a name="customizing-the-authorization-endpoint"></a>

#### Налаштування кінцевої точки авторизації

За замовчуванням Echo використовуватиме`/broadcasting/auth`кінцева точка для авторизації доступу до каналу. Однак ви можете вказати власну кінцеву точку авторизації, передавши`authEndpoint`параметр конфігурації для вашого екземпляра Echo:

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key',
        authEndpoint: '/custom/endpoint/auth'
    });

<a name="defining-authorization-callbacks"></a>

### Визначення зворотних викликів авторизації

Далі нам потрібно визначити логіку, яка насправді буде виконувати авторизацію каналу. Це робиться в`routes/channels.php`файл, який входить до вашої програми. У цьому файлі ви можете використовувати файл`Broadcast::channel`спосіб реєстрації зворотних викликів авторизації каналу:

    Broadcast::channel('order.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

`channel`Метод приймає два аргументи: назву каналу та зворотний виклик, який повертається`true` or `false`вказує, чи дозволено користувачеві прослуховувати канал.

Усі зворотні виклики авторизації отримують поточно автентифікованого користувача як перший аргумент, а будь-які додаткові параметри підстановки - як наступні аргументи. У цьому прикладі ми використовуємо`{orderId}`заповнювач, щоб вказати, що частина «Ідентифікатор» назви каналу є символом підстановки.

<a name="authorization-callback-model-binding"></a>

#### Прив'язка моделі авторизації зворотного виклику

Подібно до HTTP-маршрутів, маршрути каналів також можуть використовувати неявні та явні[route model binding](/docs/{{version}}/routing#route-model-binding). Наприклад, замість отримання рядка або ідентифікатора числового замовлення ви можете попросити фактичний`Order`примірник моделі:

    use App\Models\Order;

    Broadcast::channel('order.{order}', function ($user, Order $order) {
        return $user->id === $order->user_id;
    });

<a name="authorization-callback-authentication"></a>

#### Авторизація Аутентифікація зворотного дзвінка

Приватні канали та канали мовної присутності автентифікують поточного користувача через захист автентифікації вашого додатка за замовчуванням. Якщо користувач не аутентифікований, авторизація каналу автоматично відмовляється, а зворотний виклик авторизації ніколи не виконується. Однак ви можете призначити декілька спеціальних охоронців, які за необхідності мають автентифікувати вхідний запит:

    Broadcast::channel('channel', function () {
        // ...
    }, ['guards' => ['web', 'admin']]);

<a name="defining-channel-classes"></a>

### Визначення класів каналів

Якщо ваша програма використовує багато різних каналів, ваш`routes/channels.php`файл може стати громіздким. Отже, замість використання Closures для авторизації каналів, ви можете використовувати класи каналів. Щоб створити клас каналу, використовуйте`make:channel`Ремісниче командування. Ця команда помістить новий клас каналу в`App/Broadcasting`каталог.

    php artisan make:channel OrderChannel

Далі зареєструйте свій канал у своєму`routes/channels.php`файл:

    use App\Broadcasting\OrderChannel;

    Broadcast::channel('order.{order}', OrderChannel::class);

Нарешті, ви можете помістити логіку авторизації для свого каналу в клас каналу '`join`метод. Це`join`метод буде містити ту саму логіку, яку ви зазвичай розміщували б у своєму закритті авторизації каналу. Ви також можете скористатися перевагами прив'язки моделі каналу:

    <?php

    namespace App\Broadcasting;

    use App\Models\Order;
    use App\Models\User;

    class OrderChannel
    {
        /**
         * Create a new channel instance.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * Authenticate the user's access to the channel.
         *
         * @param  \App\Models\User  $user
         * @param  \App\Models\Order  $order
         * @return array|bool
         */
        public function join(User $user, Order $order)
        {
            return $user->id === $order->user_id;
        }
    }

> {tip} Як і багато інших класів у Laravel, класи каналів автоматично визначаються[службовий контейнер](/docs/{{version}}/container). Отже, ви можете ввести в конструктор будь-які залежності, потрібні вашому каналу.

<a name="broadcasting-events"></a>

## Трансляція подій

Визначивши подію та позначивши її значком`ShouldBroadcast`інтерфейсу, вам потрібно активувати подію лише за допомогою`event`функція. Диспетчер подій помітить, що подія позначена значком`ShouldBroadcast`інтерфейс і поставить у чергу подію для трансляції:

    event(new ShippingStatusUpdated($update));

<a name="only-to-others"></a>

### Тільки іншим

Створюючи додаток, що використовує трансляцію подій, ви можете замінити`event`функція за допомогою`broadcast`функція. Подобається`event`функція,`broadcast`функція передає подію вашим серверним слухачам:

    broadcast(new ShippingStatusUpdated($update));

Однак,`broadcast`функція також виставляє`toOthers`метод, який дозволяє виключити поточного користувача з одержувачів трансляції:

    broadcast(new ShippingStatusUpdated($update))->toOthers();

Щоб краще зрозуміти, коли вам може знадобитися використовувати`toOthers`, уявімо собі програму списку завдань, де користувач може створити нове завдання, ввівши ім’я завдання. Щоб створити завдання, ваша програма може зробити запит до`/task`кінцева точка, яка транслює створення завдання та повертає представлення JSON нового завдання. Коли ваша програма JavaScript отримує відповідь від кінцевої точки, вона може безпосередньо вставити нове завдання у свій список завдань так:

    axios.post('/task', task)
        .then((response) => {
            this.tasks.push(response.data);
        });

Однак пам’ятайте, що ми також транслювали створення завдання. Якщо ваша програма JavaScript прослуховує цю подію, щоб додати завдання до списку завдань, у вашому списку будуть повторювані завдання: одне з кінцевої точки та одне з трансляції. Ви можете вирішити це за допомогою`toOthers`метод вказівки мовнику не транслювати подію поточному користувачеві.

> {note} Ваша подія повинна використовувати`Illuminate\Broadcasting\InteractsWithSockets`риса для того, щоб викликати`toOthers`метод.

<a name="only-to-others-configuration"></a>

#### Конфігурація

Коли ви ініціалізуєте екземпляр Laravel Echo, з'єднанню присвоюється ідентифікатор сокета. Якщо ви використовуєте[Переглянути](https://vuejs.org)і[Аксіос](https://github.com/mzabriskie/axios), ідентифікатор сокета буде автоматично приєднаний до кожного вихідного запиту як`X-Socket-ID`заголовок. Потім, коли ви телефонуєте на`toOthers`методу, Laravel витягне ідентифікатор сокета із заголовка та дасть вказівку мовнику не транслювати жодні з'єднання з цим ідентифікатором сокета.

Якщо ви не використовуєте Vue та Axios, вам доведеться вручну налаштувати свою програму JavaScript для надсилання`X-Socket-ID`заголовок. Ви можете отримати ідентифікатор сокета за допомогою`Echo.socketId`метод:

    var socketId = Echo.socketId();

<a name="receiving-broadcasts"></a>

## Отримання трансляцій

<a name="installing-laravel-echo"></a>

### Встановлення Laravel Echo

Laravel Echo - це бібліотека JavaScript, яка робить безболісним підписку на канали та прослуховування подій, які транслює Laravel. Ви можете встановити Echo через менеджер пакунків NPM. У цьому прикладі ми також встановимо`pusher-js`пакет, оскільки ми будемо використовувати мовник Pusher Channels:

    npm install --save-dev laravel-echo pusher-js

Після встановлення Echo ви готові створити новий екземпляр Echo у JavaScript вашого додатка. Чудове місце для цього - внизу`resources/js/bootstrap.js`файл, що входить до фреймворку Laravel:

    import Echo from "laravel-echo"

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key'
    });

При створенні екземпляра Echo, який використовує файл`pusher`Ви також можете вказати a`cluster`а також про те, чи потрібно з'єднання здійснювати через TLS (за замовчуванням, коли`forceTLS` is `false`, буде підключено не TLS-з'єднання, якщо сторінка була завантажена через HTTP, або як резервний варіант, якщо TLS-з'єднання не вдається):

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key',
        cluster: 'eu',
        forceTLS: true
    });

<a name="using-an-existing-client-instance"></a>

#### Використання існуючого екземпляра клієнта

Якщо у вас вже є екземпляр клієнта Pusher Channels або Socket.io, який ви хотіли б використовувати Echo, ви можете передати його Echo через`client`варіант конфігурації:

    const client = require('pusher-js');

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key',
        client: client
    });

<a name="listening-for-events"></a>

### Прослуховування подій

Після встановлення та створення екземпляра Echo ви готові розпочати прослуховування трансляцій подій. По-перше, використовуйте`channel`метод отримати екземпляр каналу, а потім викликати`listen`метод прослуховування вказаної події:

    Echo.channel('orders')
        .listen('OrderShipped', (e) => {
            console.log(e.order.name);
        });

Якщо ви хочете слухати події на приватному каналі, використовуйте`private`замість цього. Ви можете продовжувати ланцюгові дзвінки до`listen`метод прослуховування декількох подій на одному каналі:

    Echo.private('orders')
        .listen(...)
        .listen(...)
        .listen(...);

<a name="leaving-a-channel"></a>

### Вихід із каналу

Щоб залишити канал, ви можете зателефонувати до`leaveChannel`у вашому екземплярі Echo:

    Echo.leaveChannel('orders');

Якщо ви хочете залишити канал, а також пов'язані з ним приватні канали та канали присутності, ви можете зателефонувати за номером`leave`метод:

    Echo.leave('orders');

<a name="namespaces"></a>

### Простори імен

Можливо, ви помітили у прикладах вище, що ми не вказали повний простір імен для класів подій. Це тому, що Echo автоматично припустить, що події знаходяться в`App\Events`простір імен. Однак ви можете налаштувати кореневий простір імен під час створення екземпляра Echo, передавши файл`namespace`варіант конфігурації:

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key',
        namespace: 'App.Other.Namespace'
    });

Крім того, ви можете поставити префікс класів подій перед символом`.`підписуючись на них за допомогою Echo. Це дозволить завжди вказувати повну назву класу:

    Echo.channel('orders')
        .listen('.Namespace\\Event\\Class', (e) => {
            //
        });

<a name="presence-channels"></a>

## Канали присутності

Канали присутності базуються на безпеці приватних каналів, одночасно виявляючи додаткову функцію обізнаності про те, хто підписаний на канал. Це спрощує створення потужних спільних функцій додатків, таких як сповіщення користувачів, коли інший користувач переглядає ту саму сторінку.

<a name="authorizing-presence-channels"></a>

### Авторизація каналів присутності

Усі канали присутності також є приватними; тому користувачі повинні бути[уповноважений на доступ до них](#authorizing-channels). Однак при визначенні зворотних викликів для каналів присутності ви не повернетесь`true`якщо користувач уповноважений приєднуватися до каналу. Натомість вам слід повернути масив даних про користувача.

Дані, повернені авторизаційним зворотним викликом, будуть доступні слухачам подій каналу присутності у вашому додатку JavaScript. Якщо користувач не має права приєднуватися до каналу присутності, вам слід повернутися`false`або`null`:

    Broadcast::channel('chat.{roomId}', function ($user, $roomId) {
        if ($user->canJoinRoom($roomId)) {
            return ['id' => $user->id, 'name' => $user->name];
        }
    });

<a name="joining-presence-channels"></a>

### Приєднання до каналів присутності

Щоб приєднатися до каналу присутності, ви можете використовувати Echo's`join`метод.`join`метод поверне a`PresenceChannel`реалізації, яка поряд з викриттям`listen`метод, дозволяє підписатися на`here`,`joining`, і`leaving`події.

    Echo.join(`chat.${roomId}`)
        .here((users) => {
            //
        })
        .joining((user) => {
            console.log(user.name);
        })
        .leaving((user) => {
            console.log(user.name);
        });

`here`зворотний дзвінок буде виконано негайно, як тільки канал буде успішно приєднаний, і отримає масив, що містить інформацію про користувачів для всіх інших користувачів, які зараз підписані на канал.`joining`буде виконано, коли новий користувач приєднається до каналу, тоді як`leaving`метод буде виконаний, коли користувач покине канал.

<a name="broadcasting-to-presence-channels"></a>

### Трансляція на канали присутності

Канали присутності можуть приймати події подібно до державних або приватних каналів. На прикладі чату ми можемо захотіти транслювати`NewMessage`події на канал присутності кімнати. Для цього ми повернемо екземпляр`PresenceChannel`від події`broadcastOn`метод:

    /**
     * Get the channels the event should broadcast on.
     *
     * @return Channel|array
     */
    public function broadcastOn()
    {
        return new PresenceChannel('room.'.$this->message->room_id);
    }

Як і публічні чи приватні події, події каналу присутності можуть транслюватися за допомогою`broadcast`функція. Як і у випадку інших подій, ви можете використовувати`toOthers`спосіб виключити поточного користувача з прийому трансляції:

    broadcast(new NewMessage($message));

    broadcast(new NewMessage($message))->toOthers();

Ви можете прослухати подію приєднання через Echo's`listen`метод:

    Echo.join(`chat.${roomId}`)
        .here(...)
        .joining(...)
        .leaving(...)
        .listen('NewMessage', (e) => {
            //
        });

<a name="client-events"></a>

## Події клієнта

> {tip} Під час використання[Канали штовхача](https://pusher.com/channels), ви повинні ввімкнути опцію "Клієнтські події" у розділі "Налаштування програми" на вашому[інформаційна панель програми](https://dashboard.pusher.com/)для того, щоб надсилати події клієнта.

Іноді вам може знадобитися транслювати подію для інших підключених клієнтів, взагалі не натискаючи вашу програму Laravel. Це може бути особливо корисно для таких речей, як сповіщення про "набір тексту", коли ви хочете попередити користувачів своєї програми, що інший користувач вводить повідомлення на даному екрані.

Для трансляції клієнтських подій ви можете використовувати Echo's`whisper`метод:

    Echo.private('chat')
        .whisper('typing', {
            name: this.user.name
        });

Для прослуховування подій клієнта ви можете використовувати`listenForWhisper`метод:

    Echo.private('chat')
        .listenForWhisper('typing', (e) => {
            console.log(e.name);
        });

<a name="notifications"></a>

## Повідомлення

З’єднавши трансляцію події з[повідомлення](/docs/{{version}}/notifications), ваш додаток JavaScript може отримувати нові сповіщення в міру їх появи, не потребуючи оновлення сторінки. По-перше, обов’язково прочитайте документацію щодо використання[канал сповіщення про трансляцію](/docs/{{version}}/notifications#broadcast-notifications).

Після того, як ви налаштували сповіщення для використання трансляційного каналу, ви можете слухати трансляційні події за допомогою Echo's`notification`метод. Пам’ятайте, назва каналу повинна відповідати назві класу сутності, яка отримує сповіщення:

    Echo.private(`App.Models.User.${userId}`)
        .notification((notification) => {
            console.log(notification.type);
        });

У цьому прикладі всі сповіщення надіслані на`App\Models\User`екземпляри через`broadcast`канал отримає зворотний дзвінок. Зворотний виклик авторизації каналу для`App.Models.User.{id}`канал включений за замовчуванням`BroadcastServiceProvider`що поставляється з рамкою Laravel.
