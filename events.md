# Події

-   [Вступ](#introduction)
-   [Реєстрація подій та Listeners](#registering-events-and-listeners)
    -   [Генерування подій та Listeners](#generating-events-and-listeners)
    -   [Реєстрація подій вручну](#manually-registering-events)
    -   [Відкриття подій](#event-discovery)
-   [Визначення подій](#defining-events)
-   [Визначення Listeners](#defining-listeners)
-   [Listeners подій у черзі](#queued-event-listeners)
    -   [Доступ до черги вручну](#manually-accessing-the-queue)
    -   [Обробка невдалих завдань](#handling-failed-jobs)
-   [Диспетчеризація подій](#dispatching-events)
-   [Підписники подій](#event-subscribers)
    -   [Написання Підписників подій](#writing-event-subscribers)
    -   [Реєстрація Підписників подій](#registering-event-subscribers)

<a name="introduction"></a>

## Вступ

Події Laravel забезпечують просту реалізацію Observers(Спостерігачі)в, що дозволяє підписатися та слухати різні події, що відбуваються у вашому додатку. Класи подій зазвичай зберігаються в`app/Events`в той час як їхні Listeners зберігаються в`app/Listeners`. Не хвилюйтеся, якщо ви не бачите цих каталогів у своїй програмі, оскільки вони будуть створені для вас, коли ви створюєте події та Listeners за допомогою команд консолі Artisan.

Події служать чудовим способом розділити різні аспекти вашої програми, оскільки в одній події може бути кілька Listeners, які не залежать один від одного. Наприклад, ви можете надіслати сповіщення Slack своєму користувачеві кожного разу, коли замовлення відправляється. Замість того, щоб поєднувати код обробки замовлень із кодом сповіщення Slack, ви можете підняти`OrderShipped`подія, яку Listener може отримати та перетворити на сповіщення Slack.

<a name="registering-events-and-listeners"></a>

## Реєстрація подій та Listeners

`EventServiceProvider`у комплекті з вашою програмою Laravel є зручне місце для реєстрації всіх Listeners подій вашої програми.`listen`властивість містить масив усіх подій (ключів) та їх Listeners (значень). Ви можете додати до цього масиву стільки подій, скільки вимагає ваша програма. Наприклад, додамо`OrderShipped`подія:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'App\Events\OrderShipped' => [
            'App\Listeners\SendShipmentNotification',
        ],
    ];

<a name="generating-events-and-listeners"></a>

### Генерування подій та Listeners

Звичайно, ручне створення файлів для кожної події та Listenerа є громіздким. Натомість додайте Listeners та події до свого`EventServiceProvider`і використовувати`event:generate`команди. Ця команда генерує будь-які події або Listeners, перелічені у вашому`EventServiceProvider`. Події та Listeners, які вже існують, залишаться незайманими:

    php artisan event:generate

<a name="manually-registering-events"></a>

### Реєстрація подій вручну

Як правило, події слід реєструвати через`EventServiceProvider``$listen`масив; однак ви також можете реєструвати події на основі закриття вручну в`boot`метод вашого`EventServiceProvider`:

    use App\Events\PodcastProcessed;

    /**
     * Register any other events for your application.
     *
     * @return void
     */
    public function boot()
    {
        Event::listen(function (PodcastProcessed $event) {
            //
        });
    }

<a name="queuable-anonymous-event-listeners"></a>

#### Listeners анонімних подій, які можна чекати в чергу

При реєстрації прослуховувачів подій вручну, ви можете обернути закриття прослуховувача всередині`Illuminate\Events\queueable`функція, щоб доручити Laravel виконати Listenerа за допомогою[черга](/docs/{{version}}/queues):

    use App\Events\PodcastProcessed;
    use function Illuminate\Events\queueable;
    use Illuminate\Support\Facades\Event;

    /**
     * Register any other events for your application.
     *
     * @return void
     */
    public function boot()
    {
        Event::listen(queueable(function (PodcastProcessed $event) {
            //
        }));
    }

Як і завдання в черзі, ви можете використовувати`onConnection`,`onQueue`, і`delay`методи налаштування виконання прослуховувача в черзі:

    Event::listen(queueable(function (PodcastProcessed $event) {
        //
    })->onConnection('redis')->onQueue('podcasts')->delay(now()->addSeconds(10)));

Якщо ви хочете усунути анонімні помилки Listeners, що стоять у черзі, ви можете надати закриття для`catch`метод при визначенні`queueable`Listener:

    use App\Events\PodcastProcessed;
    use function Illuminate\Events\queueable;
    use Illuminate\Support\Facades\Event;
    use Throwable;

    Event::listen(queueable(function (PodcastProcessed $event) {
        //
    })->catch(function (PodcastProcessed $event, Throwable $e) {
        // The queued listener failed...
    }));

<a name="wildcard-event-listeners"></a>

#### Listeners подій підстановки

Ви навіть можете зареєструвати Listeners за допомогою`*`як підстановочний параметр, що дозволяє ловити кілька подій на одному Listeners. Listeners підстановки отримують назву події як перший аргумент, а весь масив даних про події - як другий аргумент:

    Event::listen('event.*', function ($eventName, array $data) {
        //
    });

<a name="event-discovery"></a>

### Відкриття подій

Замість того, щоб реєструвати події та Listeners вручну у`$listen`масив`EventServiceProvider`, ви можете ввімкнути автоматичне виявлення подій. Коли увімкнено виявлення подій, Laravel автоматично буде знаходити та реєструвати ваші події та Listeners, скануючи вашу програму`Listeners`каталог. Крім того, будь-які чітко визначені події, перелічені в`EventServiceProvider`все одно буде зареєстровано.

Laravel знаходить Listeners подій, скануючи класи Listeners за допомогою рефлексії. Коли Laravel знаходить будь-який метод класу Listenerа, який починається з`handle`, Laravel зареєструє ці методи як прослуховувачі подій для події, про яку вказується тип у підписі методу:

    use App\Events\PodcastProcessed;

    class SendPodcastProcessedNotification
    {
        /**
         * Handle the given event.
         *
         * @param  \App\Events\PodcastProcessed
         * @return void
         */
        public function handle(PodcastProcessed $event)
        {
            //
        }
    }

Виявлення подій за замовчуванням вимкнено, але ви можете ввімкнути його, замінивши`shouldDiscoverEvents`метод вашої програми`EventServiceProvider`:

    /**
     * Determine if events and listeners should be automatically discovered.
     *
     * @return bool
     */
    public function shouldDiscoverEvents()
    {
        return true;
    }

За замовчуванням будуть проскановані всі Listeners в каталозі Listeners вашої програми. Якщо ви хочете визначити додаткові каталоги для сканування, ви можете замінити`discoverEventsWithin`метод у вашому`EventServiceProvider`:

    /**
     * Get the listener directories that should be used to discover events.
     *
     * @return array
     */
    protected function discoverEventsWithin()
    {
        return [
            $this->app->path('Listeners'),
        ];
    }

У виробництві ви, швидше за все, не хочете, щоб фреймворк сканував усіх ваших Listeners на кожен запит. Тому під час процесу розгортання вам слід запустити`event:cache`Команда Artisan: кешувати маніфест усіх подій вашого додатка та Listeners. Цей маніфест буде використовуватися фреймворком для пришвидшення процесу реєстрації події.`event:clear`команда може бути використана для знищення кешу.

> {tip} The`event:list`команда може використовуватися для відображення списку всіх подій та Listeners, зареєстрованих вашою програмою.

<a name="defining-events"></a>

## Визначення подій

Клас події - це контейнер даних, який містить інформацію, що стосується події. Наприклад, припустимо, що наш генерований`OrderShipped`подія отримує[Eloquent ОРМ](/docs/{{version}}/eloquent)об'єкт:

    <?php

    namespace App\Events;

    use App\Models\Order;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Foundation\Events\Dispatchable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped
    {
        use Dispatchable, InteractsWithSockets, SerializesModels;

        public $order;

        /**
         * Create a new event instance.
         *
         * @param  \App\Models\Order  $order
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }
    }

Як бачите, цей клас подій не містить логіки. Це контейнер для`Order`екземпляр, який був придбаний.`SerializesModels`риса, яка використовується подією, витончено серіалізує будь-які Eloquent моделі, якщо об’єкт події серіалізується за допомогою PHP`serialize`функція.

<a name="defining-listeners"></a>

## Визначення Listeners

Далі, давайте поглянемо на Listenerа для нашого прикладу події. Listeners подій отримують екземпляр події у своєму`handle`метод.`event:generate`Команда автоматично імпортує належний клас події та вкаже на підказку про подію в`handle`метод. У межах`handle`методом, ви можете виконувати будь-які дії, необхідні для реагування на подію:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;

    class SendShipmentNotification
    {
        /**
         * Create the event listener.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * Handle the event.
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            // Access the order using $event->order...
        }
    }

> {tip} Ваші прослуховувачі подій можуть також натякати на будь-які залежності, які їм потрібні, від своїх конструкторів. Усі Listeners подій вирішуються через Laravel[службовий контейнер](/docs/{{version}}/container), тому залежності будуть вводитись автоматично.

<a name="stopping-the-propagation-of-an-event"></a>

#### Припинення поширення події

Іноді, можливо, ви захочете припинити розповсюдження події іншим Listenerам. Ви можете зробити це, повернувшись`false`від вашого Listenerа`handle`метод.

<a name="queued-event-listeners"></a>

## Listeners подій у черзі

Listeners в черзі можуть бути корисними, якщо ваш Listener буде виконувати повільні завдання, наприклад, відправляти електронне повідомлення або робити HTTP-запит. Перш ніж розпочати роботу зі Listenerами, які перебувають у черзі, обов’язково[налаштувати свою чергу](/docs/{{version}}/queues)і запустіть прослуховувач черги на вашому сервері або в локальному середовищі розробки.

Щоб вказати, що Listener повинен бути в черзі, додайте`ShouldQueue`інтерфейс до класу Listenerа. Listeners, створені`event:generate`Команда Artisan вже має цей інтерфейс, імпортований у поточний простір імен, тому ви можете використовувати його відразу:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        //
    }

Це воно! Тепер, коли цей Listener буде викликаний до події, він автоматично буде поставлений у чергу в диспетчері подій за допомогою Laravel's[система черг](/docs/{{version}}/queues). Якщо під час виконання прослуховувача чергою не буде вилучено жодних винятків, завдання в черзі автоматично видаляється після завершення обробки.

<a name="customizing-the-queue-connection-queue-name"></a>

#### Налаштування підключення черги та назви черги

Якщо ви хочете налаштувати підключення черги, назву черги або час затримки черги прослуховувача події, ви можете визначити`$connection`,`$queue`, або`$delay`властивості вашого класу Listenerа:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        /**
         * The name of the connection the job should be sent to.
         *
         * @var string|null
         */
        public $connection = 'sqs';

        /**
         * The name of the queue the job should be sent to.
         *
         * @var string|null
         */
        public $queue = 'listeners';

        /**
         * The time (seconds) before the job should be processed.
         *
         * @var int
         */
        public $delay = 60;
    }

Якщо ви хочете визначити чергу Listenerа під час виконання, ви можете визначити файл`viaQueue`метод на Listeners:

    /**
     * Get the name of the listener's queue.
     *
     * @return string
     */
    public function viaQueue()
    {
        return 'listeners';
    }

<a name="conditionally-queueing-listeners"></a>

#### Listeners умовно в черзі

Іноді вам може знадобитися визначити, чи слід ставити Listenerа в чергу, виходячи з деяких даних, доступних лише під час виконання. Для цього:`shouldQueue`метод може бути доданий до Listenerа, щоб визначити, чи слід Listenerа поставити в чергу. Якщо`shouldQueue`метод повертає`false`, Listener не буде виконаний:

    <?php

    namespace App\Listeners;

    use App\Events\OrderPlaced;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class RewardGiftCard implements ShouldQueue
    {
        /**
         * Reward a gift card to the customer.
         *
         * @param  \App\Events\OrderPlaced  $event
         * @return void
         */
        public function handle(OrderPlaced $event)
        {
            //
        }

        /**
         * Determine whether the listener should be queued.
         *
         * @param  \App\Events\OrderPlaced  $event
         * @return bool
         */
        public function shouldQueue(OrderPlaced $event)
        {
            return $event->order->subtotal >= 5000;
        }
    }

<a name="manually-accessing-the-queue"></a>

### Доступ до черги вручну

Якщо вам потрібно вручну отримати доступ до основних завдань черги Listenerа`delete`і`release`методами, ви можете зробити це за допомогою`Illuminate\Queue\InteractsWithQueue`риса. Ця ознака імпортується за замовчуванням для згенерованих Listeners і забезпечує доступ до таких методів:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Queue\InteractsWithQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * Handle the event.
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            if (true) {
                $this->release(30);
            }
        }
    }

<a name="handling-failed-jobs"></a>

### Обробка невдалих завдань

Іноді ваші прослуховувачі подій у черзі можуть вийти з ладу. Якщо Listener в черзі перевищує максимальну кількість спроб, визначену вашим працівником черги, файл`failed`метод буде викликаний для вашого Listenerа.`failed`метод отримує екземпляр події та виняток, що спричинив помилку:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Queue\InteractsWithQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * Handle the event.
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            //
        }

        /**
         * Handle a job failure.
         *
         * @param  \App\Events\OrderShipped  $event
         * @param  \Throwable  $exception
         * @return void
         */
        public function failed(OrderShipped $event, $exception)
        {
            //
        }
    }

<a name="dispatching-events"></a>

## Диспетчеризація подій

Щоб надіслати подію, ви можете передати примірник події в`event`помічник. Помічник надішле подію всім зареєстрованим Listenerам. Так як`event`helper доступний у всьому світі, ви можете зателефонувати йому з будь-якої точки вашої програми:

    <?php

    namespace App\Http\Controllers;

    use App\Events\OrderShipped;
    use App\Http\Controllers\Controller;
    use App\Models\Order;

    class OrderController extends Controller
    {
        /**
         * Ship the given order.
         *
         * @param  int  $orderId
         * @return Response
         */
        public function ship($orderId)
        {
            $order = Order::findOrFail($orderId);

            // Order shipment logic...

            event(new OrderShipped($order));
        }
    }

Крім того, якщо ваша подія використовує`Illuminate\Foundation\Events\Dispatchable`риси, ви можете назвати статичним`dispatch`метод події. Будь-які аргументи, передані в`dispatch`метод буде передано конструктору події:

    OrderShipped::dispatch($order);

> {tip} Під час тестування може бути корисним стверджувати, що певні події були надіслані, фактично не викликаючи їх Listeners. Laravel[вбудовані помічники для тестування](/docs/{{version}}/mocking#event-fake)робить це чинчем.

<a name="event-subscribers"></a>

## Підписники подій

<a name="writing-event-subscribers"></a>

### Написання Підписників подій

Абоненти подій - це класи, які можуть підпискити кілька подій із самого класу, що дозволяє визначити кілька обробників подій в межах одного класу. Абоненти повинні визначити a`subscribe`метод, який буде переданий екземпляру диспетчера подій. Ви можете зателефонувати до`listen`для даного диспетчера для реєстрації Listeners подій:

    <?php

    namespace App\Listeners;

    class UserEventSubscriber
    {
        /**
         * Handle user login events.
         */
        public function handleUserLogin($event) {}

        /**
         * Handle user logout events.
         */
        public function handleUserLogout($event) {}

        /**
         * Register the listeners for the subscriber.
         *
         * @param  \Illuminate\Events\Dispatcher  $events
         * @return void
         */
        public function subscribe($events)
        {
            $events->listen(
                'Illuminate\Auth\Events\Login',
                [UserEventSubscriber::class, 'handleUserLogin']
            );

            $events->listen(
                'Illuminate\Auth\Events\Logout',
                [UserEventSubscriber::class, 'handleUserLogout']
            );
        }
    }

Крім того, ваш абонент`subscribe`метод може повернути масив подій до відображень обробників. У цьому випадку відображення прослуховувача подій будуть автоматично зареєстровані для вас:

    use Illuminate\Auth\Events\Login;
    use Illuminate\Auth\Events\Logout;

    /**
     * Register the listeners for the subscriber.
     *
     * @return array
     */
    public function subscribe()
    {
        return [
            Login::class => [
                [UserEventSubscriber::class, 'handleUserLogin']
            ],

            Logout::class => [
                [UserEventSubscriber::class, 'handleUserLogout']
            ],
        ];
    }

<a name="registering-event-subscribers"></a>

### Реєстрація Підписників подій

Після написання Підписника ви готові зареєструвати його у диспетчері подій. Ви можете зареєструвати абонентів за допомогою`$subscribe`майно на`EventServiceProvider`. Наприклад, додамо`UserEventSubscriber`до списку:

    <?php

    namespace App\Providers;

    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        /**
         * The event listener mappings for the application.
         *
         * @var array
         */
        protected $listen = [
            //
        ];

        /**
         * The subscriber classes to register.
         *
         * @var array
         */
        protected $subscribe = [
            'App\Listeners\UserEventSubscriber',
        ];
    }
