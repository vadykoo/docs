# Контролери

-   [Вступ](#introduction)
-   [Основні контролери](#basic-controllers)
    -   [Визначення контролерів](#defining-controllers)
    -   [Контролери однієї дії](#single-action-controllers)
-   [Проміжне програмне забезпечення контролера](#controller-middleware)
-   [Контролери ресурсів](#resource-controllers)
    -   [Часткові маршрути ресурсів](#restful-partial-resource-routes)
    -   [Вкладені ресурси](#restful-nested-resources)
    -   [Іменування маршрутів ресурсів](#restful-naming-resource-routes)
    -   [Присвоєння параметрів маршруту ресурсу](#restful-naming-resource-route-parameters)
    -   [Обсяг маршрутів використання ресурсів](#restful-scoping-resource-routes)
    -   [Локалізація ресурсів URI](#restful-localizing-resource-uris)
    -   [Доповнення контролерів ресурсів](#restful-supplementing-resource-controllers)
-   [Інжекція залежності та контролери](#dependency-injection-and-controllers)
-   [Кешування маршрутів](#route-caching)

<a name="introduction"></a>

## Вступ

Замість того, щоб визначати всю логіку обробки запитів як Закриття у файлах маршрутів, можливо, ви захочете організувати цю поведінку за допомогою класів контролера. Контролери можуть групувати пов'язану логіку обробки запитів в один клас. Контролери зберігаються в`app/Http/Controllers`каталог.

<a name="basic-controllers"></a>

## Основні контролери

<a name="defining-controllers"></a>

### Визначення контролерів

Нижче наведено приклад базового класу контролера. Зауважте, що контролер розширює базовий клас контролера, що входить до складу Laravel. Базовий клас забезпечує кілька зручних методів, таких як`middleware`метод, який може бути використаний для приєднання проміжного програмного забезпечення до дій контролера:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\User;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return \Illuminate\View\View
         */
        public function show($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

Ви можете визначити маршрут до цієї дії контролера так:

    use App\Http\Controllers\UserController;

    Route::get('user/{id}', [UserController::class, 'show']);

Тепер, коли запит відповідає вказаному URI маршруту, файл`show`метод на`UserController`клас буде виконаний. Параметри маршруту також будуть передані методу.

> {tip} Контролери - ні**вимагається**розширити базовий клас. Однак ви не матимете доступу до таких зручних функцій, як`middleware`,`validate`, і`dispatch`методи.

<a name="single-action-controllers"></a>

### Контролери однієї дії

Якщо ви хочете визначити контролер, який обробляє лише одну дію, ви можете розмістити одну`__invoke`метод на контролері:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\User;

    class ShowProfile extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return \Illuminate\View\View
         */
        public function __invoke($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

При реєстрації маршрутів для контролерів однієї дії не потрібно вказувати метод:

    use App\Http\Controllers\ShowProfile;

    Route::get('user/{id}', ShowProfile::class);

Ви можете створити контролер, що викликається, за допомогою`--invokable`варіант`make:controller`Команда ремісників:

    php artisan make:controller ShowProfile --invokable

> {tip} Заглушки контролера можна налаштувати за допомогою[заглушка видавництва](/docs/{{version}}/artisan#stub-customization)

<a name="controller-middleware"></a>

## Проміжне програмне забезпечення контролера

[Проміжне програмне забезпечення](/docs/{{version}}/middleware)можуть бути призначені маршрутам контролера у ваших файлах маршрутів:

    Route::get('profile', [UserController::class, 'show'])->middleware('auth');

Однак зручніше вказувати проміжне програмне забезпечення в конструкторі вашого контролера. Використання`middleware`методу з конструктора вашого контролера, ви можете легко призначити проміжне програмне забезпечення дії контролера. Ви навіть можете обмежити проміжне програмне забезпечення лише певними методами класу контролера:

    class UserController extends Controller
    {
        /**
         * Instantiate a new controller instance.
         *
         * @return void
         */
        public function __construct()
        {
            $this->middleware('auth');

            $this->middleware('log')->only('index');

            $this->middleware('subscribed')->except('store');
        }
    }

Контролери також дозволяють реєструвати проміжне програмне забезпечення за допомогою Закриття. Це забезпечує зручний спосіб визначити проміжне програмне забезпечення для одного контролера без визначення цілого класу проміжного програмного забезпечення:

    $this->middleware(function ($request, $next) {
        // ...

        return $next($request);
    });

> {tip} Ви можете призначити проміжне програмне забезпечення підмножині дій контролера; однак це може означати, що ваш контролер стає занадто великим. Натомість розгляньте можливість розбиття контролера на кілька менших контролерів.

<a name="resource-controllers"></a>

## Контролери ресурсів

Маршрутизація ресурсів Laravel призначає типові маршрути "CRUD" контролеру з одним рядком коду. Наприклад, ви можете створити контролер, який обробляє всі HTTP-запити на "фотографії", що зберігаються у вашому додатку. Використання`make:controller`Реміснича команда, ми можемо швидко створити такий контролер:

    php artisan make:controller PhotoController --resource

Ця команда генерує контролер за адресою`app/Http/Controllers/PhotoController.php`. Контролер буде містити метод для кожної з доступних операцій з ресурсами.

Далі ви можете зареєструвати винахідливий маршрут до контролера:

    Route::resource('photos', PhotoController::class);

Це одне оголошення маршруту створює кілька маршрутів для обробки різноманітних дій на ресурсі. Сформований контролер вже матиме методи, стербінг для кожної з цих дій, включаючи примітки, що повідомляють вас про дієслова HTTP та URI, якими вони обробляють.

Ви можете зареєструвати багато контролерів ресурсів одночасно, передавши масив до`resources`метод:

    Route::resources([
        'photos' => PhotoController::class,
        'posts' => PostController::class,
    ]);

<a name="actions-handled-by-resource-controller"></a>

#### Дії, які обробляє контролер ресурсів

| Дієслово               | Ненависть              | Дія        | Назва маршруту |
| ---------------------- | ---------------------- | ---------- | -------------- |
| ОТРИМАТИ               | `/photos`              | індекс     | photos.index   |
| ОТРИМАТИ               | `/photos/create`       | створити   | photos.create  |
| ПОСТ                   | `/photos`              | магазин    | photos.store   |
| ОТРИМАТИ               | `/photos/{photo}`      | шоу        | photos.show    |
| ОТРИМАТИ               | `/photos/{photo}/edit` | редагувати | photos.edit    |
| ВСТАНОВИТИ / ВІДМИТИТИ | `/photos/{photo}`      | оновлення  | photos.update  |
| ВИДАЛИТИ               | `/photos/{photo}`      | знищити    | photos.destroy |

<a name="specifying-the-resource-model"></a>

#### Вказівка ​​моделі ресурсу

Якщо ви використовуєте прив'язку моделі маршруту і хочете, щоб методи контролера ресурсів натякали на примірник моделі, ви можете використовувати`--model`опція при генерації контролера:

    php artisan make:controller PhotoController --resource --model=Photo

<a name="restful-partial-resource-routes"></a>

### Часткові маршрути ресурсів

При оголошенні маршруту ресурсу ви можете вказати підмножину дій, які повинен виконувати контролер замість повного набору дій за замовчуванням:

    Route::resource('photos', PhotoController::class)->only([
        'index', 'show'
    ]);

    Route::resource('photos', PhotoController::class)->except([
        'create', 'store', 'update', 'destroy'
    ]);

<a name="api-resource-routes"></a>

#### Маршрути ресурсів API

Оголошуючи маршрути ресурсів, які будуть споживатися API, ви зазвичай хочете виключити маршрути, що містять HTML-шаблони, такі як`create`і`edit`. Для зручності ви можете використовувати`apiResource`метод автоматичного виключення цих двох маршрутів:

    Route::apiResource('photos', PhotoController::class);

Ви можете зареєструвати багато контролерів ресурсів API одночасно, передавши масив до`apiResources` method:

    Route::apiResources([
        'photos' => PhotoController::class,
        'posts' => PostController::class,
    ]);

Швидко сформувати контролер ресурсів API, який не включає`create`або`edit`методи, використовуйте`--api`перемикач при виконанні`make:controller`команда:

    php artisan make:controller API/PhotoController --api

<a name="restful-nested-resources"></a>

### Вкладені ресурси

Іноді вам може знадобитися визначити маршрути до вкладеного ресурсу. Наприклад, фоторесурс може мати кілька коментарів, які можуть бути прикріплені до фотографії. Щоб вкласти контролери ресурсів, використовуйте позначення «крапка» у декларації маршруту:

    Route::resource('photos.comments', PhotoCommentController::class);

Цей маршрут реєструє вкладений ресурс, до якого можна отримати доступ за допомогою URI, як показано нижче:

    /photos/{photo}/comments/{comment}

<a name="scoping-nested-resources"></a>

#### Обсяг вкладених ресурсів

Ларавеля[implicit model binding](/docs/{{version}}/routing#implicit-model-binding-scoping)функція може автоматично застосовувати вкладені прив'язки таким чином, що дозволена дочірня модель підтверджується, що належить батьківській моделі. За допомогою`scoped`методу при визначенні вкладеного ресурсу, ви можете ввімкнути автоматичне масштабування, а також проінструктувати Laravel, яке поле має отримати дочірній ресурс:

    Route::resource('photos.comments', PhotoCommentController::class)->scoped([
        'comment' => 'slug',
    ]);

Цей маршрут буде реєструвати вкладений ресурс, до якого можна отримати доступ за допомогою URI, як показано нижче:

    /photos/{photo}/comments/{comment:slug}

<a name="shallow-nesting"></a>

#### Неглибоке гніздування

Часто в URI не обов’язково мати і батьківський, і дочірній ідентифікатори, оскільки дочірній ідентифікатор вже є унікальним ідентифікатором. Використовуючи унікальні ідентифікатори, такі як автоматичне збільшення первинних ключів для ідентифікації ваших моделей у сегментах URI, ви можете вибрати використання "неглибокого вкладання":

    Route::resource('photos.comments', CommentController::class)->shallow();

Визначення маршруту вище визначатиме такі маршрути:

| Дієслово               | Ненависть                         | Дія        | Назва маршруту         |
| ---------------------- | --------------------------------- | ---------- | ---------------------- |
| ОТРИМАТИ               | `/photos/{photo}/comments`        | індекс     | photos.comments.index  |
| ОТРИМАТИ               | `/photos/{photo}/comments/create` | створити   | photos.comments.create |
| ПОСТ                   | `/photos/{photo}/comments`        | магазин    | photos.comments.store  |
| ОТРИМАТИ               | `/comments/{comment}`             | шоу        | comments.show          |
| ОТРИМАТИ               | `/comments/{comment}/edit`        | редагувати | коментарі. ред         |
| ВСТАНОВИТИ / ВІДМИТИТИ | `/comments/{comment}`             | оновлення  | коментарі. оновити     |
| ВИДАЛИТИ               | `/comments/{comment}`             | знищити    | коментарі.руйнувати    |

<a name="restful-naming-resource-routes"></a>

### Іменування маршрутів ресурсів

За замовчуванням усі дії контролера ресурсів мають назву маршруту; однак ви можете замінити ці імена, передавши a`names`масив з вашими опціями:

    Route::resource('photos', PhotoController::class)->names([
        'create' => 'photos.build'
    ]);

<a name="restful-naming-resource-route-parameters"></a>

### Присвоєння параметрів маршруту ресурсу

За замовчуванням,`Route::resource`створить параметри маршрутів для ваших маршрутів ресурсів на основі "одиничної" версії імені ресурсу. Ви можете легко замінити це для кожного ресурсу, використовуючи`parameters`метод. Масив перейшов у`parameters`метод повинен бути асоціативним масивом імен ресурсів та імен параметрів:

    Route::resource('users', AdminUserController::class)->parameters([
        'users' => 'admin_user'
    ]);

Наведений вище приклад генерує такі URI для ресурсів`show`маршрут:

    /users/{admin_user}

<a name="restful-scoping-resource-routes"></a>

### Обсяг маршрутів використання ресурсів

Іноді, при неявному прив’язуванні декількох красномовних моделей у визначеннях маршрутів ресурсів, можливо, ви захочете застосувати другу модель красномовства таким чином, що вона повинна бути дочірньою до першої красномовної моделі. Наприклад, розглянемо цю ситуацію, яка отримує допис у блозі від slug для певного користувача:

    use App\Http\Controllers\PostsController;

    Route::resource('users.posts', PostsController::class)->scoped();

Ви можете замінити типові ключі моделі за замовчуванням, передавши масив до`scoped`метод:

    use App\Http\Controllers\PostsController;

    Route::resource('users.posts', PostsController::class)->scoped([
        'post' => 'slug',
    ]);

При використанні користувацького неявного прив’язування як вкладеного параметра маршруту, Laravel автоматично застосує запит для отримання вкладеної моделі своїм батьком, використовуючи конвенції, щоб вгадати ім’я зв’язку на батьківському. У цьому випадку передбачається, що`User`модель має відносини з іменем`posts`(множина імені параметра параметра маршруту), який можна використовувати для отримання`Post`модель.

<a name="restful-localizing-resource-uris"></a>

### Локалізація ресурсів URI

За замовчуванням,`Route::resource`створить ресурсні URI з використанням англійських дієслів. Якщо вам потрібно локалізувати`create`і`edit`дієслова дії, ви можете використовувати`Route::resourceVerbs`метод. Це можна зробити в`boot`метод вашого`AppServiceProvider`:

    use Illuminate\Support\Facades\Route;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Route::resourceVerbs([
            'create' => 'crear',
            'edit' => 'editar',
        ]);
    }

Після налаштування дієслів, реєстрація маршруту ресурсу, наприклад`Route::resource('fotos', 'PhotoController')`видасть такі URI:

    /fotos/crear

    /fotos/{foto}/editar

<a name="restful-supplementing-resource-controllers"></a>

### Доповнення контролерів ресурсів

Якщо вам потрібно додати додаткові маршрути до контролера ресурсів, що перевищують набір маршрутів ресурсів за замовчуванням, вам слід визначити ці маршрути перед вашим викликом`Route::resource`; в іншому випадку маршрути, визначені`resource`метод ненавмисно може мати перевагу над вашими додатковими маршрутами:

    Route::get('photos/popular', [PhotoController::class, 'popular']);

    Route::resource('photos', PhotoController::class);

> {tip} Пам’ятайте, щоб ваші контролери були зосередженими. Якщо ви вважаєте, що вам регулярно потрібні методи поза типовим набором дій із ресурсами, розгляньте можливість розділити свій контролер на два менших контролери.

<a name="dependency-injection-and-controllers"></a>

## Інжекція залежності та контролери

<a name="constructor-injection"></a>

#### Інжектор конструктора

Ларавель[службовий контейнер](/docs/{{version}}/container)використовується для вирішення всіх контролерів Laravel. Як результат, ви можете вводити натяк на будь-які залежності, які можуть знадобитися вашому контролеру, у своєму конструкторі. Заявлені залежності будуть автоматично вирішені та введені в екземпляр контролера:

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * The user repository instance.
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }
    }

Ви також можете ввести будь-який натяк[Контракт Laravel](/docs/{{version}}/contracts). Якщо контейнер може це вирішити, ви можете ввести-підказати. Залежно від вашої програми, введення ваших залежностей у ваш контролер може забезпечити кращу перевірочність.

<a name="method-injection"></a>

#### Метод ін’єкції

На додаток до введення конструктора, ви також можете натякнути на залежність від методів вашого контролера. Поширеним варіантом використання ін'єкції методу є ін'єкція`Illuminate\Http\Request`екземпляр у ваші методи контролера:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Store a new user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $name = $request->name;

            //
        }
    }

Якщо ваш метод контролера також очікує введення від параметра маршруту, перелічіть аргументи маршруту після інших залежностей. Наприклад, якщо ваш маршрут визначено так:

    Route::put('user/{id}', [UserController::class, 'update']);

Ви все ще можете навести натяк на`Illuminate\Http\Request`і отримати доступ до вашого`id`параметром, визначивши метод контролера наступним чином:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Update the given user.
         *
         * @param  Request  $request
         * @param  string  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }

<a name="route-caching"></a>

## Кешування маршрутів

Якщо у вашій програмі використовуються виключно маршрути, засновані на контролерах, вам слід скористатися кешем маршрутів Laravel. Використання кешу маршрутів різко зменшить кількість часу, необхідного для реєстрації всіх маршрутів вашої програми. У деяких випадках реєстрація маршруту може бути навіть в 100 разів швидшою. Щоб створити кеш маршруту, просто запустіть`route:cache`Команда ремісників:

    php artisan route:cache

Після запуску цієї команди ваш кешований файл маршрутів буде завантажений при кожному запиті. Пам'ятайте, що якщо ви додаєте нові маршрути, вам потрібно буде створити новий кеш маршрутів. Через це вам слід лише запускати`route:cache`команда під час розгортання вашого проекту.

Ви можете використовувати`route:clear`команда для очищення кешу маршруту:

    php artisan route:clear
