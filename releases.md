# Примітки до випуску

-   [Схема версій](#versioning-scheme)
-   [Політика підтримки](#support-policy)
-   [Laravel 8](#laravel-8)

<a name="versioning-scheme"></a>

## Схема версій

Далі йде Laravel та інші власні пакети[Семантична версія](https://semver.org). Основні фреймворкові випуски випускаються кожні шість місяців (~ березень та ~ вересень), тоді як незначні та виправлені випуски можуть виходити так часто, як щотижня. Незначні випуски та випуски патчів повинні**ніколи**містять зміни, що порушують.

Посилаючись на фреймворк Laravel або його компоненти із вашої програми чи пакету, ви завжди повинні використовувати обмеження версії, такі як`^8.0`, оскільки основні випуски Laravel дійсно включають важкі зміни. Однак ми прагнемо завжди забезпечувати можливість оновлення до нового основного випуску за один день або менше.

<a name="support-policy"></a>

## Support Policy

Для випусків LTS, таких як Laravel 6, виправлення помилок надається на 2 роки, а виправлення безпеки - на 3 роки. Ці випуски забезпечують найдовше вікно підтримки та обслуговування. У загальних версіях виправлення помилок надаються протягом 7 місяців, а виправлення безпеки - протягом 1 року. Для всіх додаткових бібліотек, включаючи Lumen, лише останній випуск отримує виправлення помилок. Крім того, перегляньте версії бази даних[підтримується Laravel](/docs/{{version}}/database#introduction).

| Версія  | Звільнення                    | Виправлено помилки до | Виправлення безпеки до        |
| ------- | ----------------------------- | --------------------- | ----------------------------- |
| 6 (LTS) | Вересневе шоу, 2019           | 5 жовтня 2021 року    | September 3rd, 2022           |
| 7       | Відображення бренду, 2020 рік | 6 жовтня 2020 р       | Відображення бренду, 2021 рік |
| 8       | 8 вересня 2020 р              | 6 квітня 2021 року    | 8 вересня 2021 року           |

<a name="laravel-8"></a>

## Laravel 8

Laravel 8 продовжує вдосконалення, внесені в Laravel 7.x, представляючи Laravel Jetstream, класи заводських моделей, складання міграцій, пакетну роботу, вдосконалене обмеження швидкості, вдосконалення черги, динамічні компоненти Blade, пагінаційні View Tailwind, допоміжні засоби тестування часу, вдосконалення`artisan serve`, вдосконалення прослуховувача подій та безліч інших виправлень помилок та покращення зручності використання.

<a name="laravel-jetstream"></a>

### Laravel Jetstream

_Laravel Jetstream був написаний[Тейлор Отуелл](https://github.com/taylorotwell)_.

[Laravel Jetstream](https://jetstream.laravel.com)- це красиво розроблені будівельні риштування для Laravel. Jetstream забезпечує ідеальну початкову точку для вашого наступного проекту і включає вхід, реєстрацію, перевірку електронної пошти, двофакторну автентифікацію, управління сеансами, підтримку API через Laravel Sanctum та необов’язкове управління командою. Laravel Jetstream замінює та вдосконалює застарілі інтерфейси автентифікації, доступні для попередніх версій Laravel.

Jetstream розроблений з використанням[CSS вітру хвоста](https://tailwindcss.com)і пропонує на ваш вибір[Дріт під напругою](https://laravel-livewire.com)або[Інерція](https://inertiajs.com)риштування.

<a name="models-directory"></a>

### Каталог моделей

Завдяки переважному попиту громади, за замовчуванням скелет програми Laravel тепер містить файл`app/Models`каталог. Ми сподіваємось, вам сподобається цей новий будинок для ваших Eloquent моделей! Всі відповідні команди генератора були оновлені, щоб припустити, що моделі існують у`app/Models`каталог, якщо він існує. Якщо каталог не існує, фреймворк вважатиме, що ваші моделі слід розміщувати в`app`каталог.

<a name="model-factory-classes"></a>

### Заводські класи моделей

_Зразкові фабричні класи сприяли[Тейлор Отуелл](https://github.com/taylorotwell)_.

Красномовний[модельні заводи](/docs/{{version}}/database-testing#creating-factories) have been entirely re-written as class based factories and improved to have first-class relationship support. For example, the `UserFactory`включений до Laravel пишеться так:

    <?php

    namespace Database\Factories;

    use App\Models\User;
    use Illuminate\Database\Eloquent\Factories\Factory;
    use Illuminate\Support\Str;

    class UserFactory extends Factory
    {
        /**
         * The name of the factory's corresponding model.
         *
         * @var string
         */
        protected $model = User::class;

        /**
         * Define the model's default state.
         *
         * @return array
         */
        public function definition()
        {
            return [
                'name' => $this->faker->name,
                'email' => $this->faker->unique()->safeEmail,
                'email_verified_at' => now(),
                'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
                'remember_token' => Str::random(10),
            ];
        }
    }

Завдяки новому`HasFactory`ознака, доступна на згенерованих моделях, фабрика моделей може використовуватися так:

    use App\Models\User;

    User::factory()->count(50)->create();

Since model factories are now simple PHP classes, state transformations may be written as class methods. In addition, you may add any other helper classes to your Eloquent model factory as needed.

Наприклад, ваш`User`модель може мати`suspended`стан, який змінює одне зі значень атрибута за замовчуванням. Ви можете визначити свої перетворення стану, використовуючи базові заводи`state`метод. Ви можете назвати свій метод стану як завгодно. Зрештою, це лише типовий метод PHP:

    /**
     * Indicate that the user is suspended.
     *
     * @return \Illuminate\Database\Eloquent\Factories\Factory
     */
    public function suspended()
    {
        return $this->state([
            'account_status' => 'suspended',
        ]);
    }

Після визначення методу перетворення стану ми можемо використовувати його так:

    use App\Models\User;

    User::factory()->count(5)->suspended()->create();

Як уже згадувалося, фабрики моделей Laravel 8 містять першокласну підтримку стосунків. Отже, припускаючи наш`User`модель має`posts`Метод відносин, ми можемо просто запустити такий код, щоб створити користувача з трьома повідомленнями:

    $users = User::factory()
                ->hasPosts(3, [
                    'published' => false,
                ])
                ->create();

Для полегшення процесу оновлення,[laravel/legacy-factory](https://github.com/laravel/legacy-factories)пакет був випущений для забезпечення підтримки попередньої ітерації модельних фабрик в Laravel 8.x.

Переписані фабрики Laravel містять набагато більше можливостей, які, на нашу думку, вам сподобаються. Щоб дізнатись більше про фабричні моделі, будь ласка, проконсультуйтесь із[документація щодо тестування баз даних](/docs/{{version}}/database-testing#creating-factories).

<a name="migration-squashing"></a>

### Зменшення міграції

_Міграційне стиснення сприяло[Тейлор Отуелл](https://github.com/taylorotwell)_.

Створюючи додаток, з часом ви можете накопичувати все більше і більше міграцій. Це може призвести до того, що ваш каталог міграції роздується потенційно сотнями міграцій. Якщо ви використовуєте MySQL або PostgreSQL, тепер ви можете "перекласти" свої міграції в один файл SQL. Для початку виконайте`schema:dump`команда:

    php artisan schema:dump

    // Dump the current database schema and prune all existing migrations...
    php artisan schema:dump --prune

Коли ви виконуєте цю команду, Laravel напише файл "schema" на ваш`database/schema`каталог. Тепер, коли ви намагаєтесь перенести вашу базу даних і жодної іншої міграції не було виконано, Laravel спочатку виконає SQL-файл файлу схеми. Після виконання команд файлу схеми Laravel виконає всі залишені міграції, які не були частиною дампа схеми.

<a name="job-batching"></a>

### Розміщення робочих місць

_Розміщення робочих місць сприяло[Тейлор Отуелл](https://github.com/taylorotwell)&[Мохаммад Саєд](https://github.com/themsaid)_.

Функція пакетного керування завданнями Laravel дозволяє вам легко виконати партію завдань, а потім виконати певні дії, коли партія завдань завершиться.

Новий`batch`метод`Bus`Facadeможе бути використаний для відправки партії робочих місць. Звичайно, пакетне обслуговування в першу чергу корисно в поєднанні із зворотними викликами завершення. Отже, ви можете використовувати`then`,`catch`, і`finally`методи визначення зворотних викликів завершення для пакета. Кожен із цих зворотних дзвінків отримає`Illuminate\Bus\Batch`екземпляр, коли вони викликаються:

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

Щоб дізнатись більше про пакет роботи, проконсультуйтесь із[документація черги](/docs/{{version}}/queues#job-batching).

<a name="improved-rate-limiting"></a>

### Покращене обмеження тарифу

_Поліпшення обмеження ставок зробили[Тейлор Отуелл](https://github.com/taylorotwell)_.

Функція обмеження швидкості запиту Laravel була розширена з більшою гнучкістю та потужністю, зберігаючи при цьому зворотну сумісність із попередніми випусками`throttle`API проміжного програмного забезпечення.

Обмежувачі ставок визначаються за допомогою`RateLimiter`фасадні`for`метод.`for`метод приймає ім'я обмежувача швидкості та Закриття, яке повертає конфігурацію обмеження, яка повинна застосовуватися до маршрутів, яким призначений цей обмежувач швидкості:

    use Illuminate\Cache\RateLimiting\Limit;
    use Illuminate\Support\Facades\RateLimiter;

    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000);
    });

Оскільки зворотні виклики обмежувача швидкості отримують вхідний екземпляр запиту HTTP, ви можете динамічно створювати відповідне обмеження швидкості на основі вхідного запиту або автентифікованого користувача:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()->vipCustomer()
                    ? Limit::none()
                    : Limit::perMinute(100);
    });

Іноді вам може знадобитися сегментувати обмеження тарифів на якесь довільне значення. Наприклад, ви можете дозволити користувачам отримати доступ до даного маршруту 100 разів на хвилину за IP-адресою. Для цього ви можете використовувати`by`метод при побудові обмеження тарифу:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()->vipCustomer()
                    ? Limit::none()
                    : Limit::perMinute(100)->by($request->ip());
    });

Обмежувачі ставок можуть бути прикріплені до маршрутів або груп маршрутів за допомогою`throttle`[Middlware](/docs/{{version}}/middleware). Middleware дросельної заслінки приймає назву обмежувача швидкості, який ви хочете призначити маршруту:

    Route::middleware(['throttle:uploads'])->group(function () {
        Route::post('/audio', function () {
            //
        });

        Route::post('/video', function () {
            //
        });
    });

Щоб дізнатись більше про обмеження тарифу, зверніться до[документація про маршрутизацію](/docs/{{version}}/routing#rate-limiting).

<a name="improved-maintenance-mode"></a>

### Покращений режим обслуговування

_Покращенню режиму обслуговування сприяв[Тейлор Отуелл](https://github.com/taylorotwell)з натхненням від[Космос](https://spatie.be)_.

У попередніх випусках Laravel,`php artisan down`функцію режиму обслуговування можна обійти, використовуючи "список дозволів" IP-адрес, яким було дозволено отримати доступ до програми. Ця функція була вилучена на користь більш простого рішення "секрет" / маркер.

Перебуваючи в режимі обслуговування, ви можете використовувати`secret`можливість вказати маркер обходу режиму обслуговування:

    php artisan down --secret="1630542a-246b-4b66-afa1-dd72a4c43515"

Після розміщення програми в режимі обслуговування ви можете перейти до URL-адреси програми, що відповідає цьому маркеру, і Laravel видасть браузеру обхідний файл cookie режиму обслуговування:

    https://example.com/1630542a-246b-4b66-afa1-dd72a4c43515

Коли ви отримуєте доступ до цього прихованого маршруту, ви будете перенаправлені на`/`маршрут програми. Після видачі файлу cookie у ваш браузер ви зможете нормально переглядати програму, ніби вона не перебуває в режимі обслуговування.

<a name="pre-rendering-the-maintenance-mode-view"></a>

#### Попереднє відтворення View режиму обслуговування

Якщо ви використовуєте`php artisan down`під час розгортання, користувачі все одно можуть іноді стикатися з помилками, якщо отримують доступ до програми під час оновлення залежностей Composer або інших компонентів інфраструктури. Це відбувається тому, що значна частина фреймворку Laravel повинна завантажитися для того, щоб визначити, чи ваш додаток перебуває в режимі технічного обслуговування та відтворити View режиму обслуговування за допомогою механізму шаблонування.

З цієї причини Laravel тепер дозволяє попередньо відтворити View режиму обслуговування, яке буде повернуто на самому початку циклу запиту. Цей View відображається до завантаження залежностей вашої програми. Ви можете попередньо відтворити обраний вами шаблон за допомогою`down`команди`render`варіант:

    php artisan down --render="errors::503"

<a name="closure-dispatch-chain-catch"></a>

### Відправлення / ланцюг закриття`catch`

_Поліпшенню вилову сприяв[Мохаммад Саєд](https://github.com/themsaid)_.

Використання нового`catch`методом, тепер ви можете надати Закриття, яке слід виконати, якщо Закриття в черзі не вдається успішно завершити після вичерпання всіх спроблених спроб спроби повторної спроби у вашій черзі:

    use Throwable;

    dispatch(function () use ($podcast) {
        $podcast->publish();
    })->catch(function (Throwable $e) {
        // This job has failed...
    });

<a name="dynamic-blade-components"></a>

### Динамічні компоненти Blade

_Динамічні компоненти Blade внесли вклад[Тейлор Отуелл](https://github.com/taylorotwell)_.

Іноді вам може знадобитися відобразити компонент, але не знати, який компонент повинен бути відтворений до часу виконання. У цій ситуації тепер ви можете використовувати вбудований Laravel`dynamic-component`компонент для візуалізації компонента на основі значення середовища або змінної:

    <x-dynamic-component :component="$componentName" class="mt-4" />

Щоб дізнатися більше про компоненти Blade, зверніться до[Документація Blade](/docs/{{version}}/blade#components).

<a name="event-listener-improvements"></a>

### Удосконалення прослуховувача подій

_Покращення слухачів подій зробив[Тейлор Отуелл](https://github.com/taylorotwell)_.

Слухачі подій на основі закриття тепер можуть бути зареєстровані, лише передавши закриття до`Event::listen`метод. Laravel перевірить Закриття, щоб визначити, який тип події слухач обробляє:

    use App\Events\PodcastProcessed;
    use Illuminate\Support\Facades\Event;

    Event::listen(function (PodcastProcessed $event) {
        //
    });

Крім того, прослуховувачі подій на основі закриття тепер можуть бути позначені як такі, що стоять у черзі за допомогою`Illuminate\Events\queueable`функція:

    use App\Events\PodcastProcessed;
    use function Illuminate\Events\queueable;
    use Illuminate\Support\Facades\Event;

    Event::listen(queueable(function (PodcastProcessed $event) {
        //
    }));

Як і завдання в черзі, ви можете використовувати`onConnection`,`onQueue`, і`delay`методи налаштування виконання прослуховувача в черзі:

    Event::listen(queueable(function (PodcastProcessed $event) {
        //
    })->onConnection('redis')->onQueue('podcasts')->delay(now()->addSeconds(10)));

Якщо ви хочете усунути анонімні помилки слухачів, що стоять у черзі, ви можете надати закриття для`catch`метод при визначенні`queueable`слухач:

    use App\Events\PodcastProcessed;
    use function Illuminate\Events\queueable;
    use Illuminate\Support\Facades\Event;
    use Throwable;

    Event::listen(queueable(function (PodcastProcessed $event) {
        //
    })->catch(function (PodcastProcessed $event, Throwable $e) {
        // The queued listener failed...
    }));

<a name="time-testing-helpers"></a>

### Тестування часу помічників

_Помічники тестування часу внесли[Тейлор Отуелл](https://github.com/taylorotwell)з натхненням від Ruby on Rails_.

Під час тестування вам іноді може знадобитися змінити час, повернутий помічниками, такими як`now`або`Illuminate\Support\Carbon::now()`. Базовий клас тестування функцій Laravel тепер включає помічники, які дозволяють маніпулювати поточним часом:

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

<a name="artisan-serve-improvements"></a>

### Ремісник`serve`Покращення

_Ремісник`serve`вдосконаленням сприяв[Тейлор Отуелл](https://github.com/taylorotwell)_.

Ремісник`serve`команда була покращена за допомогою автоматичного перезавантаження, коли зміни змінної середовища виявляються в межах вашого локального`.env`файл. Раніше команду потрібно було зупинити та перезапустити вручну.

<a name="tailwind-pagination-views"></a>

### Перегляди пагінації хвостового вітру

Пагінатор Laravel оновлений для використання[CSS вітру хвоста](https://tailwindcss.com)за замовчуванням. Tailwind CSS - це високо настроюваний, низькорівневий фреймворк CSS, який надає вам всі будівельні блоки, необхідні для створення індивідуальних проектів, без будь-яких настирливих, самовпевнених стилів, з якими вам доводиться боротися, щоб їх замінити. Звичайно, Bootstrap 3 і 4 також залишаються доступними.

<a name="routing-namespace-updates"></a>

### Routing оновлень простору імен

У попередніх випусках Laravel,`RouteServiceProvider`містили a`$namespace`майно. Значення цієї властивості буде автоматично додано до префіксу до визначень маршруту контролера та викликів до`action`помічник /`URL::action`метод. У Laravel 8.x це властивість`null`за замовчуванням. Це означає, що Laravel не робить автоматичного префікса простору імен. Отже, у нових програмах Laravel 8.x визначення маршрутів контролера слід визначати, використовуючи стандартний синтаксис, що викликається PHP:

    use App\Http\Controllers\UserController;

    Route::get('/users', [UserController::class, 'index']);

Дзвінки до`action`пов'язані методи повинні використовувати однаковий синтаксис, що викликається:

    action([UserController::class, 'index']);

    return Redirect::action([UserController::class, 'index']);

Якщо ви віддаєте перевагу префіксу маршруту контролера стилю Laravel 7.x, ви можете просто додати`$namespace`власності у вашій програмі`RouteServiceProvider`.

> {note} Ця зміна стосується лише нових програм Laravel 8.x. Програми, що оновлюються з Laravel 7.x, все ще матимуть`$namespace`майно в їх`RouteServiceProvider`.
