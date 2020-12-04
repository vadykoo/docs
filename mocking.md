# Mocking

-   [Вступ](#introduction)
-   [Mocking Objects](#mocking-objects)
-   [Автобус Підробка](#bus-fake)
-   [Подія Фейк](#event-fake)
    -   [Підроблені фальшиві події](#scoped-event-fakes)
-   [Підробка HTTP](#http-fake)
-   [Mail підробка](#mail-fake)
-   [Повідомлення Фейк](#notification-fake)
-   [Черга Fake](#queue-fake)
-   [Зберігання Підробка](#storage-fake)
-   [Взаємодія з часом](#interacting-with-time)
-   [Фасади](#mocking-facades)

<a name="introduction"></a>

## Вступ

Під час тестування додатків Laravel, можливо, ви захочете "знущатись" над деякими аспектами вашого додатка, щоб вони фактично не виконувались під час даного тесту. Наприклад, під час тестування контролера, який відправляє подію, можливо, ви захочете знущатись над Listenerами подій, щоб вони насправді не виконувались під час тесту. Це дозволяє протестувати лише HTTP-відповідь контролера, не турбуючись про виконання прослуховувачів подій, оскільки прослуховувачі подій можна перевірити у власному тестовому випадку.

Laravel пропонує помічників для знущань над подіями, робочими місцями та фасадами. Ці помічники в основному забезпечують зручний шар над Mockery, тому вам не доведеться вручну робити складні виклики методу Mockery. Ви також можете використовувати[Mocking](http://docs.mockery.io/en/latest/)або PHPUnit для створення власних макетів або шпигунів.

<a name="mocking-objects"></a>

## Mocking Objects

При знущанні над об’єктом, який буде введено у вашу програму через службовий контейнер Laravel, вам потрібно буде прив’язати свій знущаний екземпляр до контейнера як`instance`прив'язка. Це дозволить контейнеру використовувати ваш знущаний екземпляр об’єкта замість того, щоб будувати сам об’єкт:

    use App\Service;
    use Mockery;

    $this->instance(Service::class, Mockery::mock(Service::class, function ($mock) {
        $mock->shouldReceive('process')->once();
    }));

Для того, щоб зробити це більш зручним, ви можете використовувати`mock`метод, який забезпечується базовим класом тестового випадку Laravel:

    use App\Service;

    $this->mock(Service::class, function ($mock) {
        $mock->shouldReceive('process')->once();
    });

Ви можете використовувати`partialMock`метод, коли потрібно знущатись лише над кількома методами об’єкта. Методи, які не знущаються, будуть виконуватися нормально при виклику:

    use App\Service;

    $this->partialMock(Service::class, function ($mock) {
        $mock->shouldReceive('process')->once();
    });

Подібним чином, якщо ви хочете шпигувати за об’єктом, базовий клас тестового випадку Laravel пропонує a`spy`метод як зручна обгортка навколо`Mockery::spy`метод:

    use App\Service;

    $this->spy(Service::class, function ($mock) {
        $mock->shouldHaveReceived('process');
    });

<a name="bus-fake"></a>

## Автобус Підробка

Як альтернативу знущанню ви можете використовувати`Bus`фасадні`fake`метод запобігання відправленню Jobs. При використанні підробок Assertions робляться після виконання тестованого коду:

    <?php

    namespace Tests\Feature;

    use App\Jobs\ShipOrder;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Support\Facades\Bus;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Bus::fake();

            // Perform order shipping...

            // Assert a specific type of job was dispatched meeting the given truth test...
            Bus::assertDispatched(function (ShipOrder $job) use ($order) {
                return $job->order->id === $order->id;
            });

            // Assert a job was not dispatched...
            Bus::assertNotDispatched(AnotherJob::class);
        }
    }

<a name="event-fake"></a>

## Подія Фейк

Як альтернативу знущанню ви можете використовувати`Event`фасадні`fake`метод, щоб запобігти виконанню всіх прослуховувачів подій. Потім ви можете стверджувати, що події були відправлені, і навіть перевіряти отримані дані. При використанні підробок Assertions робляться після виконання тестованого коду:

    <?php

    namespace Tests\Feature;

    use App\Events\OrderFailedToShip;
    use App\Events\OrderShipped;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Support\Facades\Event;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Test order shipping.
         */
        public function testOrderShipping()
        {
            Event::fake();

            // Perform order shipping...

            // Assert a specific type of event was dispatched meeting the given truth test...
            Event::assertDispatched(function (OrderShipped $event) use ($order) {
                return $event->order->id === $order->id;
            });

            // Assert an event was dispatched twice...
            Event::assertDispatched(OrderShipped::class, 2);

            // Assert an event was not dispatched...
            Event::assertNotDispatched(OrderFailedToShip::class);
        }
    }

> {note} Після дзвінка`Event::fake()`, жоден Listener подій не буде виконаний. Отже, якщо у ваших тестах використовуються фабрики моделей, які покладаються на події, наприклад, створення UUID під час моделі`creating`події, вам слід зателефонувати`Event::fake()`**після**використовуючи ваші Factory.

<a name="faking-a-subset-of-events"></a>

#### Підробка підмножини подій

Якщо ви хочете підробити Listeners подій лише для певного набору подій, ви можете передати їх до`fake`або`fakeFor`метод:

    /**
     * Test order process.
     */
    public function testOrderProcess()
    {
        Event::fake([
            OrderCreated::class,
        ]);

        $order = Order::factory()->create();

        Event::assertDispatched(OrderCreated::class);

        // Other events are dispatched as normal...
        $order->update([...]);
    }

<a name="scoped-event-fakes"></a>

### Підроблені фальшиві події

Якщо ви хочете підробити Listeners подій лише для частини вашого тесту, ви можете використовувати файл`fakeFor`метод:

    <?php

    namespace Tests\Feature;

    use App\Events\OrderCreated;
    use App\Models\Order;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Support\Facades\Event;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Test order process.
         */
        public function testOrderProcess()
        {
            $order = Event::fakeFor(function () {
                $order = Order::factory()->create();

                Event::assertDispatched(OrderCreated::class);

                return $order;
            });

            // Events are dispatched as normal and observers will run ...
            $order->update([...]);
        }
    }

<a name="http-fake"></a>

## Підробка HTTP

`Http`фасадні`fake`Метод дозволяє вказувати клієнтові HTTP повертати тактові / фіктивні відповіді під час надсилання запитів. Для отримання додаткової інформації про підробку вихідних HTTP-запитів, будь ласка, зверніться до[Документація щодо тестування клієнта HTTP](/docs/{{version}}/http-client#testing).

<a name="mail-fake"></a>

## Пошта підробка

Ви можете використовувати`Mail`фасадні`fake`спосіб запобігти надсиланню пошти. Тоді ви можете стверджувати це[доступні](/docs/{{version}}/mail)були надіслані користувачам і навіть перевіряли отримані ними дані. При використанні підробок Assertions робляться після виконання тестованого коду:

    <?php

    namespace Tests\Feature;

    use App\Mail\OrderShipped;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Support\Facades\Mail;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Mail::fake();

            // Assert that no mailables were sent...
            Mail::assertNothingSent();

            // Perform order shipping...

            // Assert a specific type of mailable was dispatched meeting the given truth test...
            Mail::assertSent(function (OrderShipped $mail) use ($order) {
                return $mail->order->id === $order->id;
            });

            // Assert a message was sent to the given users...
            Mail::assertSent(OrderShipped::class, function ($mail) use ($user) {
                return $mail->hasTo($user->email) &&
                       $mail->hasCc('...') &&
                       $mail->hasBcc('...');
            });

            // Assert a mailable was sent twice...
            Mail::assertSent(OrderShipped::class, 2);

            // Assert a mailable was not sent...
            Mail::assertNotSent(AnotherMailable::class);
        }
    }

Якщо ви ставите в чергу доступні для доставки у фоновому режимі, вам слід скористатися`assertQueued`метод замість`assertSent`:

    Mail::assertQueued(...);
    Mail::assertNotQueued(...);

<a name="notification-fake"></a>

## Повідомлення Фейк

Ви можете використовувати`Notification`фасадні`fake`метод запобігання надсиланню сповіщень. Тоді ви можете стверджувати це[повідомлення](/docs/{{version}}/notifications)були надіслані користувачам і навіть перевіряли отримані ними дані. При використанні підробок Assertions робляться після виконання тестованого коду:

    <?php

    namespace Tests\Feature;

    use App\Notifications\OrderShipped;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Notifications\AnonymousNotifiable;
    use Illuminate\Support\Facades\Notification;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Notification::fake();

            // Assert that no notifications were sent...
            Notification::assertNothingSent();

            // Perform order shipping...

            // Assert a specific type of notification was sent meeting the given truth test...
            Notification::assertSentTo(
                $user,
                function (OrderShipped $notification, $channels) use ($order) {
                    return $notification->order->id === $order->id;
                }
            );

            // Assert a notification was sent to the given users...
            Notification::assertSentTo(
                [$user], OrderShipped::class
            );

            // Assert a notification was not sent...
            Notification::assertNotSentTo(
                [$user], AnotherNotification::class
            );

            // Assert a notification was sent via Notification::route() method...
            Notification::assertSentTo(
                new AnonymousNotifiable, OrderShipped::class
            );

            // Assert Notification::route() method sent notification to the correct user...
            Notification::assertSentTo(
                new AnonymousNotifiable,
                OrderShipped::class,
                function ($notification, $channels, $notifiable) use ($user) {
                    return $notifiable->routes['mail'] === $user->email;
                }
            );
        }
    }

<a name="queue-fake"></a>

## Черга Fake

Як альтернативу знущанню ви можете використовувати`Queue`фасадні`fake`метод, щоб запобігти черзі Jobs. Потім ви можете стверджувати, що завдання були перенесені в чергу і навіть перевіряти отримані дані. При використанні підробок Assertions робляться після виконання тестованого коду:

    <?php

    namespace Tests\Feature;

    use App\Jobs\AnotherJob;
    use App\Jobs\FinalJob;
    use App\Jobs\ShipOrder;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Support\Facades\Queue;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Queue::fake();

            // Assert that no jobs were pushed...
            Queue::assertNothingPushed();

            // Perform order shipping...

            // Assert a specific type of job was pushed meeting the given truth test...
            Queue::assertPushed(function (ShipOrder $job) use ($order) {
                return $job->order->id === $order->id;
            });

            // Assert a job was pushed to a given queue...
            Queue::assertPushedOn('queue-name', ShipOrder::class);

            // Assert a job was pushed twice...
            Queue::assertPushed(ShipOrder::class, 2);

            // Assert a job was not pushed...
            Queue::assertNotPushed(AnotherJob::class);

            // Assert a job was pushed with a given chain of jobs, matching by class...
            Queue::assertPushedWithChain(ShipOrder::class, [
                AnotherJob::class,
                FinalJob::class
            ]);

            // Assert a job was pushed with a given chain of jobs, matching by both class and properties...
            Queue::assertPushedWithChain(ShipOrder::class, [
                new AnotherJob('foo'),
                new FinalJob('bar'),
            ]);

            // Assert a job was pushed without a chain of jobs...
            Queue::assertPushedWithoutChain(ShipOrder::class);
        }
    }

<a name="storage-fake"></a>

## Зберігання Підробка

`Storage`фасадні`fake`Метод дозволяє легко створити підроблений диск, який у поєднанні з утилітами генерації файлів`UploadedFile`клас, значно спрощує перевірку завантаження файлів. Наприклад:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function testAlbumUpload()
        {
            Storage::fake('photos');

            $response = $this->json('POST', '/photos', [
                UploadedFile::fake()->image('photo1.jpg'),
                UploadedFile::fake()->image('photo2.jpg')
            ]);

            // Assert one or more files were stored...
            Storage::disk('photos')->assertExists('photo1.jpg');
            Storage::disk('photos')->assertExists(['photo1.jpg', 'photo2.jpg']);

            // Assert one or more files were not stored...
            Storage::disk('photos')->assertMissing('missing.jpg');
            Storage::disk('photos')->assertMissing(['missing.jpg', 'non-existing.jpg']);
        }
    }

> {tip} За замовчуванням`fake`метод видалить усі файли з тимчасового каталогу. Якщо ви хочете зберегти ці файли, ви можете замість цього застосувати метод "persistentFake".

<a name="interacting-with-time"></a>

## Взаємодія з часом

Під час тестування вам іноді може знадобитися змінити час, повернутий помічниками, такими як`now`або`Illuminate\Support\Carbon::now()`. На щастя, базовий клас тестування функцій Laravel включає помічники, які дозволяють вам керувати поточним часом:

    public function testTimeCanBeManipulated()
    {
        // Travel into the future...
        $this->travel(5)->milliseconds();
        $this->travel(5)->seconds();
        $this->travel(5)->minutes();
        $this->travel(5)->hours();
        $this->travel(5)->days();
        $this->travel(5)->weeks();
        $this->travel(5)->years();

        // Travel into the past...
        $this->travel(-5)->hours();

        // Travel to an explicit time...
        $this->travelTo(now()->subHours(6));

        // Return back to the present time...
        $this->travelBack();
    }

<a name="mocking-facades"></a>

## Фасади

На відміну від традиційних викликів статичних методів,[фасади](/docs/{{version}}/facades)може знущатися. Це забезпечує велику перевагу перед традиційними статичними методами і надає вам ту саму перевірочність, яку ви мали б, якби використовували ін’єкцію залежностей. Під час тестування ви можете часто хотіти глузувати над дзвінком на фасад Laravel в одному з ваших контролерів. Наприклад, розглянемо таку дію контролера:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Show a list of all users of the application.
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

Ми можемо знущатися із дзвінка до`Cache`фасаду за допомогою`shouldReceive`метод, який поверне екземпляр[Mocking](https://github.com/padraic/mockery)глузувати. Оскільки фасади фактично вирішуються та управляються Laravel[службовий контейнер](/docs/{{version}}/container), вони мають набагато більшу перевірочність, ніж типовий статичний клас. Наприклад, давайте глузуватимемо над нашим дзвінком до`Cache`фасадні`get`метод:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Support\Facades\Cache;
    use Tests\TestCase;

    class UserControllerTest extends TestCase
    {
        public function testGetIndex()
        {
            Cache::shouldReceive('get')
                        ->once()
                        ->with('key')
                        ->andReturn('value');

            $response = $this->get('/users');

            // ...
        }
    }

> {note} Ви не повинні глузувати з`Request`фасад. Натомість передайте потрібні дані в допоміжні методи HTTP, такі як`get`і`post`під час запуску тесту. Так само, замість глузування над`Config`фасад, зателефонуйте на`Config::set`метод у ваших тестах.
