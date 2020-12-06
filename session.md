# Сесія HTTP

-   [Вступ](#introduction)
    -   [Конфігурація](#configuration)
    -   [Передумови драйвера](#driver-prerequisites)
-   [Використання сесії](#using-the-session)
    -   [Отримання даних](#retrieving-data)
    -   [Зберігання даних](#storing-data)
    -   [Flash-дані](#flash-data)
    -   [Видалення даних](#deleting-data)
    -   [Відновлення ідентифікатора сесії](#regenerating-the-session-id)
-   [Блокування сеансу](#session-blocking)
-   [Додавання користувацьких драйверів сеансів](#adding-custom-session-drivers)
    -   [Впровадження драйвера](#implementing-the-driver)
    -   [Реєстрація драйвера](#registering-the-driver)

<a name="introduction"></a>

## Вступ

Оскільки програми, керовані HTTP, не мають стану, сеанси забезпечують спосіб зберігання інформації про користувача в декількох запитах. Laravel постачається з різноманітними серверними сервісами, доступ до яких здійснюється за допомогою виразного уніфікованого API. Підтримка таких популярних серверних систем, як[Memcached](https://memcached.org),[Редіс](https://redis.io), а бази даних включені нестандартно.

<a name="configuration"></a>

### Конфігурація

Файл конфігурації сеансу зберігається за адресою`config/session.php`. Не забудьте переглянути варіанти, доступні вам у цьому файлі. За замовчуванням Laravel налаштовано на використання`file`драйвер сеансу, який буде добре працювати для багатьох додатків.

Сесія`driver`Параметр конфігурації визначає, де будуть зберігатися дані сеансу для кожного запиту. Laravel поставляється з декількома чудовими водіями, що вийшли з коробки:

<div class="content-list" markdown="1">
- `file` - sessions are stored in `storage/framework/sessions`.
- `cookie` - sessions are stored in secure, encrypted cookies.
- `database` - sessions are stored in a relational database.
- `memcached` / `redis` - sessions are stored in one of these fast, cache based stores.
- `array` - sessions are stored in a PHP array and will not be persisted.
</div>

> {tip} Драйвер масиву використовується під час[тестування](/docs/{{version}}/testing)і запобігає збереженню даних, що зберігаються в сеансі.

<a name="driver-prerequisites"></a>

### Передумови драйвера

<a name="database"></a>

#### База даних

При використанні`database`драйвера сеансу, вам потрібно буде створити таблицю, яка міститиме елементи сеансу. Нижче наведено приклад`Schema`декларація для таблиці:

    Schema::create('sessions', function ($table) {
        $table->string('id')->unique();
        $table->foreignId('user_id')->nullable();
        $table->string('ip_address', 45)->nullable();
        $table->text('user_agent')->nullable();
        $table->text('payload');
        $table->integer('last_activity');
    });

Ви можете використовувати`session:table`Команда Artisan для генерації цієї міграції:

    php artisan session:table

    php artisan migrate

<a name="redis"></a>

#### Редіс

Перш ніж використовувати сеанси Redis з Laravel, вам доведеться або встановити розширення PHpRedis PHP через PECL, або встановити`predis/predis`пакет (~ 1.0) через Composer. Щоб отримати додаткову інформацію про налаштування Redis, перегляньте його[Сторінка документації Laravel](/docs/{{version}}/redis#configuration).

> {tip} У`session`файл конфігурації, файл`connection`Параметр може бути використаний, щоб вказати, яке з'єднання Redis використовується сеансом.

<a name="using-the-session"></a>

## Використання сесії

<a name="retrieving-data"></a>

### Отримання даних

У Laravel існує два основних способи роботи з даними сеансів: глобальний`session`помічник і через a`Request`інстанції. Спочатку давайте розглянемо доступ до сеансу через`Request`екземпляр, який можна підказати типу методу контролера. Пам'ятайте, залежності методів контролера автоматично вводяться через Laravel[службовий контейнер](/docs/{{version}}/container):

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function show(Request $request, $id)
        {
            $value = $request->session()->get('key');

            //
        }
    }

Коли ви отримуєте елемент із сеансу, ви можете також передати значення за замовчуванням як другий аргумент до`get`метод. Це значення за замовчуванням буде повернуто, якщо вказаний ключ не існує у сеансі. Якщо ви здаєте`Closure`як значення за замовчуванням для`get`і запитуваний ключ не існує,`Closure`буде виконано, а його результат повернено:

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function () {
        return 'default';
    });

<a name="the-global-session-helper"></a>

#### Помічник з глобальної сесії

Ви також можете використовувати глобальний`session`Функція PHP для отримання та збереження даних у сеансі. Коли`session`helper викликається одним аргументом рядка, він поверне значення цього ключа сеансу. Коли помічник викликається з масивом пар ключ / значення, ці значення зберігатимуться у сеансі:

    Route::get('home', function () {
        // Retrieve a piece of data from the session...
        $value = session('key');

        // Specifying a default value...
        $value = session('key', 'default');

        // Store a piece of data in the session...
        session(['key' => 'value']);
    });

> {tip} Існує невелика практична різниця між використанням сеансу через екземпляр запиту HTTP та використанням глобального`session`помічник. Обидва методи є[перевіряється](/docs/{{version}}/testing)через`assertSessionHas`метод, який доступний у всіх ваших тестових випадках.

<a name="retrieving-all-session-data"></a>

#### Отримання всіх даних сеансу

Якщо ви хочете отримати всі дані сеансу, ви можете використовувати`all`метод:

    $data = $request->session()->all();

<a name="determining-if-an-item-exists-in-the-session"></a>

#### Визначення, чи існує елемент на сесії

Щоб визначити, чи присутній елемент у сесії, ви можете використовувати`has`метод.`has`метод повертає`true`якщо предмет присутній і його немає`null`:

    if ($request->session()->has('users')) {
        //
    }

Щоб визначити, чи присутній елемент у сеансі, навіть якщо його значення становить`null`, ви можете використовувати`exists`метод.`exists`метод повертає`true`якщо товар присутній:

    if ($request->session()->exists('users')) {
        //
    }

<a name="storing-data"></a>

### Зберігання даних

Для зберігання даних у сеансі ви зазвичай використовуєте`put`метод або`session`помічник:

    // Via a request instance...
    $request->session()->put('key', 'value');

    // Via the global helper...
    session(['key' => 'value']);

<a name="pushing-to-array-session-values"></a>

#### Підштовхування до масиву значень сеансу

`push`метод може бути використаний для надсилання нового значення на значення сеансу, яке є масивом. Наприклад, якщо`user.teams`key містить масив імен команд, ви можете натиснути нове значення на масив приблизно так:

    $request->session()->push('user.teams', 'developers');

<a name="retrieving-deleting-an-item"></a>

#### Отримання та видалення елемента

`pull`метод отримає та видалить елемент із сеансу в одному висловлюванні:

    $value = $request->session()->pull('key', 'default');

<a name="flash-data"></a>

### Flash-дані

Іноді вам може знадобитися зберігати елементи в сесії лише для наступного запиту. Ви можете зробити це за допомогою`flash`метод. Дані, що зберігаються в сеансі за допомогою цього методу, будуть доступні негайно та під час наступного запиту HTTP. Після наступного HTTP-запиту миготливі дані будуть видалені. Дані Flash в першу чергу корисні для короткочасних повідомлень про стан:

    $request->session()->flash('status', 'Task was successful!');

Якщо вам потрібно зберегти ваші флеш-дані для кількох запитів, ви можете використовувати`reflash`метод, який зберігатиме всі дані флеш-пам'яті для додаткового запиту. Якщо вам потрібно зберегти лише певні флеш-дані, ви можете використовувати`keep`метод:

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

<a name="deleting-data"></a>

### Видалення даних

`forget`метод видалить фрагмент даних із сеансу. Якщо ви хочете видалити всі дані із сеансу, ви можете використовувати`flush`метод:

    // Forget a single key...
    $request->session()->forget('key');

    // Forget multiple keys...
    $request->session()->forget(['key1', 'key2']);

    $request->session()->flush();

<a name="regenerating-the-session-id"></a>

### Відновлення ідентифікатора сесії

Регенерація ідентифікатора сеансу часто робиться з метою запобігання зловмисним користувачам використовувати[фіксація сеансу](https://owasp.org/www-community/attacks/Session_fixation)атака на вашу програму.

Laravel автоматично відновлює ідентифікатор сеансу під час автентифікації, якщо ви використовуєте[Laravel Jetstream](https://jetstream.laravel.com); однак, якщо вам потрібно вручну відтворити ідентифікатор сеансу, ви можете використовувати файл`regenerate`метод.

    $request->session()->regenerate();

<a name="session-blocking"></a>

## Блокування сеансу

> {note} Щоб використовувати блокування сеансів, ваша програма повинна використовувати драйвер кешу, який підтримує[атомні замки](/docs/{{version}}/cache#atomic-locks). В даний час ці драйвери кешу включають`memcached`,`dynamodb`,`redis`, і`database`водіїв. Крім того, ви не можете використовувати`cookie`драйвер сеансу.

За замовчуванням Laravel дозволяє одночасно виконувати запити, що використовують один і той же сеанс. Так, наприклад, якщо ви використовуєте бібліотеку JavaScript HTTP, щоб зробити два HTTP-запити до вашої програми, вони обидва будуть виконуватися одночасно. Для багатьох додатків це не проблема; однак втрата даних сеансу може статися в невеликому підмножині додатків, які роблять одночасні запити до двох різних кінцевих точок програми, які обидва записують дані в сеанс.

Щоб пом'якшити це, Laravel надає функціональність, яка дозволяє обмежувати одночасні запити для даного сеансу. Для початку, ви можете просто ланцюжок`block`метод у визначенні маршруту. У цьому прикладі вхідний запит до`/profile`кінцева точка отримає блокування сеансу. Поки цей замок утримується, будь-які вхідні запити до`/profile`або`/order`кінцеві точки, що мають однаковий ідентифікатор сеансу, будуть чекати завершення першого запиту, перш ніж продовжувати їх виконання:

    Route::post('/profile', function () {
        //
    })->block($lockSeconds = 10, $waitSeconds = 10)

    Route::post('/order', function () {
        //
    })->block($lockSeconds = 10, $waitSeconds = 10)

`block`метод приймає два необов'язкові аргументи. Перший аргумент, прийнятий`block`метод - це максимальна кількість секунд, протягом якої слід утримувати блокування сеансу, перш ніж вона буде звільнена. Звичайно, якщо запит завершиться до цього часу, блокування буде звільнено раніше.

Другий аргумент, прийнятий`block`метод - це кількість секунд, протягом яких запит повинен чекати під час спроби отримати блокування сеансу. Ан`Illuminate\Contracts\Cache\LockTimeoutException`буде видано, якщо запит не може отримати блокування сеансу протягом заданої кількості секунд.

Якщо не передано жоден з цих аргументів, блокування буде отримано максимум 10 секунд, а запити будуть чекати максимум 10 секунд під час спроби отримати блокування:

    Route::post('/profile', function () {
        //
    })->block()

<a name="adding-custom-session-drivers"></a>

## Додавання користувацьких драйверів сеансів

<a name="implementing-the-driver"></a>

#### Впровадження драйвера

Ваш користувальницький драйвер сеансу повинен реалізувати`SessionHandlerInterface`. Цей інтерфейс містить лише кілька простих методів, які нам потрібно реалізувати. Стібна реалізація MongoDB виглядає приблизно так:

    <?php

    namespace App\Extensions;

    class MongoSessionHandler implements \SessionHandlerInterface
    {
        public function open($savePath, $sessionName) {}
        public function close() {}
        public function read($sessionId) {}
        public function write($sessionId, $data) {}
        public function destroy($sessionId) {}
        public function gc($lifetime) {}
    }

> {tip} Laravel не постачається з каталогом, що містить ваші розширення. Ви можете розмістити їх де завгодно. У цьому прикладі ми створили файл`Extensions`каталог для розміщення`MongoSessionHandler`.

Оскільки мета цих методів не зрозуміла, давайте швидко розглянемо, що робить кожен із методів:

<div class="content-list" markdown="1">
- The `open` method would typically be used in file based session store systems. Since Laravel ships with a `file` session driver, you will almost never need to put anything in this method. You can leave it as an empty stub. It is a fact of poor interface design (which we'll discuss later) that PHP requires us to implement this method.
- The `close` method, like the `open` method, can also usually be disregarded. For most drivers, it is not needed.
- The `read` method should return the string version of the session data associated with the given `$sessionId`. There is no need to do any serialization or other encoding when retrieving or storing session data in your driver, as Laravel will perform the serialization for you.
- The `write` method should write the given `$data` string associated with the `$sessionId` to some persistent storage system, such as MongoDB, Dynamo, etc.  Again, you should not perform any serialization - Laravel will have already handled that for you.
- The `destroy` method should remove the data associated with the `$sessionId` from persistent storage.
- The `gc` method should destroy all session data that is older than the given `$lifetime`, which is a UNIX timestamp. For self-expiring systems like Memcached and Redis, this method may be left empty.
</div>

<a name="registering-the-driver"></a>

#### Реєстрація драйвера

Після реалізації вашого драйвера ви готові зареєструвати його у фреймворку. Щоб додати додаткові драйвери до бекенду сеансу Laravel, ви можете використовувати`extend`метод на`Session`[фасад](/docs/{{version}}/facades). Вам слід зателефонувати до`extend`метод з`boot`метод a[постачальник послуг](/docs/{{version}}/providers). Ви можете зробити це з існуючих`AppServiceProvider`або створити абсолютно нового постачальника:

    <?php

    namespace App\Providers;

    use App\Extensions\MongoSessionHandler;
    use Illuminate\Support\Facades\Session;
    use Illuminate\Support\ServiceProvider;

    class SessionServiceProvider extends ServiceProvider
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
            Session::extend('mongo', function ($app) {
                // Return implementation of SessionHandlerInterface...
                return new MongoSessionHandler;
            });
        }
    }

Після реєстрації драйвера сеансу ви можете використовувати`mongo`драйвер у вашому`config/session.php`файл конфігурації.
