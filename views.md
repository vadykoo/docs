# Перегляди

-   [Створення Views](#creating-views)
-   [Передача даних у View](#passing-data-to-views)
    -   [Обмін даними з усіма Views](#sharing-data-with-all-views)
-   [Переглянути Composers](#view-composers)
-   [Оптимізація переглядів](#optimizing-views)

<a name="creating-views"></a>

## Створення Views

> {tip} Шукаєте додаткову інформацію про те, як писати шаблони Blade? Перевірте повне[Документація леза](/docs/{{version}}/blade)щоб розпочати.

Представлення містять HTML, який обслуговує ваша програма, та відокремлюють логіку контролера / програми від логіки презентації. Перегляди зберігаються в`resources/views`каталог. Простий вигляд може виглядати приблизно так:

    <!-- View stored in resources/views/greeting.blade.php -->

    <html>
        <body>
            <h1>Hello, {{ $name }}</h1>
        </body>
    </html>

Оскільки цей вигляд зберігається в`resources/views/greeting.blade.php`, ми можемо повернути його, використовуючи глобальний`view`помічник ось так:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

Як бачите, перший аргумент, переданий в`view`helper відповідає імені файлу представлення у`resources/views`каталог. Другий аргумент - це масив даних, який повинен бути доступним для View. У цьому випадку ми передаємо`name`змінна, яка відображається у поданні за допомогою[Леза синтаксису](/docs/{{version}}/blade).

Представлення також можуть бути вкладені в підкаталоги`resources/views`каталог. Позначення "крапка" може використовуватися для посилання на вкладені View. Наприклад, якщо ваш вигляд зберігається в`resources/views/admin/profile.blade.php`, ви можете посилатися на нього так:

    return view('admin.profile', $data);

> {note} Переглянути імена каталогів не повинні містити`.`характер.

<a name="determining-if-a-view-exists"></a>

#### Визначення, чи існує View

Якщо вам потрібно визначити, чи існує View, ви можете використовувати`View`фасад.`exists`метод повернеться`true`якщо View існує:

    use Illuminate\Support\Facades\View;

    if (View::exists('emails.customer')) {
        //
    }

<a name="creating-the-first-available-view"></a>

#### Створення першого доступного View

Використання`first`методом, ви можете створити перший вигляд, який існує у даному масиві переглядів. Це корисно, якщо ваша програма чи пакет дозволяє налаштовувати або перезаписувати View:

    return view()->first(['custom.admin', 'admin'], $data);

Ви також можете викликати цей метод через`View`[фасад](/docs/{{version}}/facades):

    use Illuminate\Support\Facades\View;

    return View::first(['custom.admin', 'admin'], $data);

<a name="passing-data-to-views"></a>

## Передача даних у View

Як ви бачили в попередніх прикладах, ви можете передавати масив даних у View:

    return view('greetings', ['name' => 'Victoria']);

При передачі інформації таким чином дані повинні бути масивом з парами ключ / значення. Усередині вашого View ви можете отримати доступ до кожного значення за допомогою відповідного ключа, наприклад`<?php echo $key; ?>`. Як альтернатива передачі повного масиву даних до`view`допоміжну функцію, ви можете використовувати`with`метод додавання окремих фрагментів даних до View:

    return view('greeting')->with('name', 'Victoria');

<a name="sharing-data-with-all-views"></a>

#### Обмін даними з усіма переглядами

Іноді вам може знадобитися поділитися частиною даних із усіма Viewми, які відображаються вашою програмою. Ви можете зробити це за допомогою фасадів вигляду`share`метод. Як правило, вам слід телефонувати`share`у постачальника послуг`boot`метод. Ви можете вільно додавати їх до`AppServiceProvider`або створити окремого постачальника послуг для розміщення їх:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;

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
            View::share('key', 'value');
        }
    }

<a name="view-composers"></a>

## Переглянути композиторів

Композитори View - це зворотні виклики або методи класу, які викликаються при отриманні View. Якщо у вас є дані, які ви хочете прив’язати до View кожного разу, коли цей вигляд відображається, композитор View може допомогти вам організувати цю логіку в одне місце.

У цьому прикладі давайте зареєструємо композиторів View в межах[постачальник послуг](/docs/{{version}}/providers). Ми будемо використовувати`View`Facadeдля доступу до основи`Illuminate\Contracts\View\Factory`виконання контракту. Пам'ятайте, Laravel не включає каталог за замовчуванням для композиторів View. Ви можете організувати їх, як завгодно. Наприклад, ви можете створити файл`app/Http/View/Composers`каталог:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;
    use Illuminate\Support\ServiceProvider;

    class ViewServiceProvider extends ServiceProvider
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
            // Using class based composers...
            View::composer(
                'profile', 'App\Http\View\Composers\ProfileComposer'
            );

            // Using Closure based composers...
            View::composer('dashboard', function ($view) {
                //
            });
        }
    }

> {note} Пам’ятайте, якщо ви створюєте нового постачальника послуг, який міститиме ваші реєстрації композитора View, вам потрібно буде додати постачальника послуг до`providers`масив у`config/app.php`файл конфігурації.

Тепер, коли ми зареєстрували композитора,`ProfileComposer@compose`метод буде виконуватися кожного разу, коли файл`profile`подається вид. Отже, давайте визначимо клас композитора:

    <?php

    namespace App\Http\View\Composers;

    use App\Repositories\UserRepository;
    use Illuminate\View\View;

    class ProfileComposer
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * Create a new profile composer.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            // Dependencies automatically resolved by service container...
            $this->users = $users;
        }

        /**
         * Bind data to the view.
         *
         * @param  View  $view
         * @return void
         */
        public function compose(View $view)
        {
            $view->with('count', $this->users->count());
        }
    }

Безпосередньо перед Viewм View композитора`compose`метод викликається за допомогою`Illuminate\View\View`екземпляр. Ви можете використовувати`with`метод прив’язки даних до View.

> {tip} Усі композитори View вирішуються через[службовий контейнер](/docs/{{version}}/container), тому ви можете натякнути на будь-які залежності, які вам потрібні в конструкторі композитора.

<a name="attaching-a-composer-to-multiple-views"></a>

#### Приєднання композитора до декількох Views

Ви можете приєднати композитор View до декількох Views одночасно, передавши масив Views як перший аргумент для`composer`метод:

    View::composer(
        ['profile', 'dashboard'],
        'App\Http\View\Composers\MyViewComposer'
    );

`composer`метод також приймає`*`символ як підстановочний знак, що дозволяє приєднати композитора до всіх видів:

    View::composer('*', function ($view) {
        //
    });

<a name="view-creators"></a>

#### Переглянути творців

Переглянути**творці**дуже схожі на композиторів вигляду; однак вони виконуються відразу після створення екземпляра View, а не чекають, поки View збирається відтворити. Щоб зареєструвати творця View, використовуйте`creator`метод:

    View::creator('profile', 'App\Http\View\Creators\ProfileCreator');

<a name="optimizing-views"></a>

## Оптимізація переглядів

За замовчуванням View складаються на вимогу. Коли виконується запит, який відображає View, Laravel визначить, чи існує скомпільована версія представлення. Якщо файл існує, Laravel тоді визначить, чи був скомпільований вигляд змінений недавно, ніж скомпільований. Якщо складене представлення або не існує, або нескладене представлення було змінено, Laravel перекомпілює представлення.

Компіляція поглядів під час запиту негативно впливає на ефективність, тому Laravel надає`view:cache`Команда Artisan для попередньої компіляції всіх Views, використовуваних вашим додатком. Для підвищення продуктивності ви можете запустити цю команду як частину процесу розгортання:

    php artisan view:cache

Ви можете використовувати`view:clear`команда для очищення кешу View:

    php artisan view:clear
