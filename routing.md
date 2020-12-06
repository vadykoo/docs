# Маршрутизація

-   [Основна маршрутизація](#basic-routing)
    -   [Перенаправлення маршрутів](#redirect-routes)
    -   [Переглянути маршрути](#view-routes)
-   [Параметри маршруту](#route-parameters)
    -   [Обов’язкові параметри](#required-parameters)
    -   [Необов’язкові параметри](#parameters-optional-parameters)
    -   [Обмеження регулярних виразів](#parameters-regular-expression-constraints)
-   [Іменовані маршрути](#named-routes)
-   [Route Groups](#route-groups)
    -   [Проміжне програмне забезпечення](#route-group-middleware)
    -   [Маршрутизація субдоменів](#route-group-subdomain-routing)
    -   [Префікси маршрутів](#route-group-prefixes)
    -   [Префікси імен маршрутів](#route-group-name-prefixes)
-   [Прив'язка моделі маршруту](#route-model-binding)
    -   [Неявне прив'язування](#implicit-binding)
    -   [Явне прив'язування](#explicit-binding)
-   [Запасні маршрути](#fallback-routes)
-   [Обмеження ставки](#rate-limiting)
    -   [Визначення обмежувачів ставок](#defining-rate-limiters)
    -   [Прикріплення обмежувачів ставок до маршрутів](#attaching-rate-limiters-to-routes)
-   [Метод підміни форми](#form-method-spoofing)
-   [Доступ до поточного маршруту](#accessing-the-current-route)
-   [Спільне використання ресурсів (CORS)](#cors)

<a name="basic-routing"></a>

## Основна маршрутизація

Найпростіші маршрути Laravel приймають URI та a`Closure`, забезпечуючи дуже простий і виразний метод визначення маршрутів:

    Route::get('foo', function () {
        return 'Hello World';
    });

<a name="the-default-route-files"></a>

#### Файли маршрутів за замовчуванням

Всі маршрути Laravel визначені у файлах ваших маршрутів, які знаходяться в`routes`каталог. Ці файли автоматично завантажуються фреймворком.`routes/web.php`файл визначає маршрути, призначені для вашого веб-інтерфейсу. Цим маршрутам присвоєно`web`група проміжного програмного забезпечення, яка надає такі функції, як стан сеансу та захист CSRF. Маршрути в`routes/api.php`є без громадянства і їм присвоєно`api`група проміжного програмного забезпечення.

Для більшості програм ви почнете з визначення маршрутів у вашому`routes/web.php`файл. Маршрути, визначені в`routes/web.php`можна отримати доступ, ввівши URL-адресу визначеного маршруту у своєму браузері. Наприклад, ви можете отримати доступ до наступного маршруту, перейшовши до`http://your-app.test/user`у вашому браузері:

    use App\Http\Controllers\UserController;

    Route::get('/user', [UserController::class, 'index']);

Маршрути, визначені в`routes/api.php`файл вкладений у групу маршрутів`RouteServiceProvider`. У межах цієї групи`/api`Префікс URI застосовується автоматично, тому вам не потрібно вручну застосовувати його до кожного маршруту у файлі. Ви можете змінити префікс та інші параметри групи маршрутів, змінивши ваш`RouteServiceProvider`клас.

<a name="available-router-methods"></a>

#### Доступні методи маршрутизатора

Маршрутизатор дозволяє реєструвати маршрути, які відповідають на будь-яке дієслово HTTP&#x3A;

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

Іноді вам може знадобитися зареєструвати маршрут, який відповідає на декілька дієслів HTTP. Ви можете зробити це за допомогою`match`метод. Або ви можете навіть зареєструвати маршрут, який відповідає на всі дієслова HTTP, використовуючи`any`метод:

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('/', function () {
        //
    });

<a name="csrf-protection"></a>

#### Захист CSRF

Будь-які HTML-форми, що вказують на`POST`,`PUT`,`PATCH`, або`DELETE`маршрути, визначені в`web`Файл маршрутів повинен містити поле маркера CSRF. В іншому випадку запит буде відхилено. Детальніше про захист CSRF ви можете прочитати в[Документація CSRF](/docs/{{version}}/csrf):

    <form method="POST" action="/profile">
        @csrf
        ...
    </form>

<a name="redirect-routes"></a>

### Перенаправлення маршрутів

Якщо ви визначаєте маршрут, який переспрямовує на інший URI, ви можете використовувати`Route::redirect`метод. Цей метод забезпечує зручний ярлик, так що вам не потрібно визначати повний маршрут або контролер для виконання простого перенаправлення:

    Route::redirect('/here', '/there');

За замовчуванням,`Route::redirect`повертає a`302`код стану. Ви можете налаштувати код стану, використовуючи необов’язковий третій параметр:

    Route::redirect('/here', '/there', 301);

Ви можете використовувати`Route::permanentRedirect`метод повернення a`301`код стану:

    Route::permanentRedirect('/here', '/there');

> {note} При використанні параметрів маршруту в маршрутах перенаправлення, Laravel зарезервує такі параметри, і їх не можна використовувати:`destination`і`status`.

<a name="view-routes"></a>

### Переглянути маршрути

Якщо для вашого маршруту потрібно лише повернути вигляд, ви можете використовувати`Route::view`метод. Подобається`redirect`метод, цей метод забезпечує простий ярлик, так що вам не потрібно визначати повний маршрут або контролер.`view`метод приймає URI як перший аргумент, а ім'я подання як другий аргумент. Крім того, ви можете надати масив даних для передачі у подання як необов’язковий третій аргумент:

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

> {note} При використанні параметрів маршруту у перегляді маршрутів Laravel зарезервує такі параметри і не може їх використовувати:`view`,`data`,`status`, і`headers`.

<a name="route-parameters"></a>

## Параметри маршруту

<a name="required-parameters"></a>

### Обов’язкові параметри

Іноді вам потрібно буде захопити сегменти URI у межах вашого маршруту. Наприклад, вам може знадобитися захопити ідентифікатор користувача з URL-адреси. Ви можете зробити це, визначивши параметри маршруту:

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

Ви можете визначити стільки параметрів маршруту, скільки вимагає ваш маршрут:

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

Параметри маршруту завжди укладені в межах`{}`фігурні дужки і повинні складатися з алфавітних символів і не можуть містити символу`-`характер. Замість використання`-`символу, використовуйте підкреслення (`_`). Параметри маршруту вводяться у зворотні виклики / контролери маршруту на основі їхнього порядку - імена аргументів зворотного виклику / контролера не мають значення.

<a name="parameters-optional-parameters"></a>

### Необов’язкові параметри

Іноді вам може знадобитися вказати параметр маршруту, але необов’язково вкажіть наявність цього параметра маршруту. Ви можете зробити це, поставивши`?`позначка після імені параметра. Обов’язково надайте відповідній змінній маршруту значення за замовчуванням:

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>

### Обмеження регулярних виразів

Ви можете обмежити формат параметрів маршруту, використовуючи`where`метод на екземплярі маршруту.`where`метод приймає ім'я параметра та регулярний вираз, що визначає, як слід обмежувати параметр:

    Route::get('user/{name}', function ($name) {
        //
    })->where('name', '[A-Za-z]+');

    Route::get('user/{id}', function ($id) {
        //
    })->where('id', '[0-9]+');

    Route::get('user/{id}/{name}', function ($id, $name) {
        //
    })->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

Для зручності деякі загальновживані шаблони регулярних виразів мають допоміжні методи, які дозволяють швидко додавати обмеження до шаблонів до ваших маршрутів:

    Route::get('user/{id}/{name}', function ($id, $name) {
        //
    })->whereNumber('id')->whereAlpha('name');

    Route::get('user/{name}', function ($name) {
        //
    })->whereAlphaNumeric('name');

    Route::get('user/{id}', function ($id) {
        //
    })->whereUuid('id');

<a name="parameters-global-constraints"></a>

#### Глобальні обмеження

Якщо ви хочете, щоб параметр маршруту завжди був обмежений даним регулярним виразом, ви можете використовувати`pattern`метод. Ви повинні визначити ці закономірності в`boot`метод вашого`RouteServiceProvider`:

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @return void
     */
    public function boot()
    {
        Route::pattern('id', '[0-9]+');
    }

Після того, як шаблон визначено, він автоматично застосовується до всіх маршрутів з використанням цього імені параметра:

    Route::get('user/{id}', function ($id) {
        // Only executed if {id} is numeric...
    });

<a name="parameters-encoded-forward-slashes"></a>

#### Зашифровані похилі риски

Компонент маршрутизації Laravel дозволяє всі символи, крім`/`. Ви повинні явно дозволити`/`бути частиною вашого заповнювача за допомогою`where`умовний регулярний вираз:

    Route::get('search/{search}', function ($search) {
        return $search;
    })->where('search', '.*');

> {note} Кодовані скісні риски вперед підтримуються лише в останньому сегменті маршруту.

<a name="named-routes"></a>

## Іменовані маршрути

Іменовані маршрути дозволяють зручно створювати URL-адреси або перенаправляти для конкретних маршрутів. Ви можете вказати назву маршруту, встановивши ланцюжок`name`методу у визначенні маршруту:

    Route::get('user/profile', function () {
        //
    })->name('profile');

Ви також можете вказати назви маршрутів для дій контролера:

    Route::get('user/profile', [UserProfileController::class, 'show'])->name('profile');

> {note} Назви маршрутів завжди повинні бути унікальними.

<a name="generating-urls-to-named-routes"></a>

#### Створення URL-адрес для іменованих маршрутів

Після того, як ви присвоїли ім'я даному маршруту, ви можете використовувати його назву під час створення URL-адрес або переспрямувань через глобальний`route`функція:

    // Generating URLs...
    $url = route('profile');

    // Generating Redirects...
    return redirect()->route('profile');

Якщо названий маршрут визначає параметри, ви можете передати параметри як другий аргумент до`route`функція. Наведені параметри будуть автоматично вставлені в URL-адресу в їх правильних позиціях:

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

Якщо ви передасте в масив додаткові параметри, ці пари ключ / значення автоматично будуть додані до рядка запиту згенерованої URL-адреси:

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1, 'photos' => 'yes']);

    // /user/1/profile?photos=yes

> {tip} Іноді, можливо, ви захочете вказати значення параметрів URL-адрес за замовчуванням за замовчуванням, наприклад, поточну локаль. Для цього ви можете використовувати[`URL::defaults`метод](/docs/{{version}}/urls#default-values).

<a name="inspecting-the-current-route"></a>

#### Огляд поточного маршруту

Якщо ви хочете визначити, чи поточний запит був перенаправлений на заданий іменований маршрут, ви можете використовувати`named`метод на екземплярі Route. Наприклад, ви можете перевірити поточну назву маршруту з проміжного програмного забезпечення маршруту:

    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->route()->named('profile')) {
            //
        }

        return $next($request);
    }

<a name="route-groups"></a>

## Групи маршрутів

Групи маршрутів дозволяють спільно використовувати атрибути маршруту, наприклад Middlware, для великої кількості маршрутів без необхідності визначати ці атрибути для кожного окремого маршруту.

Вкладені групи намагаються розумно "об'єднати" атрибути зі своєю батьківською групою. Проміжне програмне забезпечення та`where`умови об’єднуються, а імена та префікси додаються. Розділювачі простору імен та похилі риски в префіксах URI автоматично додаються, де це доречно.

<a name="route-group-middleware"></a>

### Проміжне програмне забезпечення

Щоб призначити Middlware для всіх маршрутів у групі, ви можете використовувати`middleware`метод перед визначенням групи. Проміжне програмне забезпечення виконується в порядку, вказаному в масиві:

    Route::middleware(['first', 'second'])->group(function () {
        Route::get('/', function () {
            // Uses first & second middleware...
        });

        Route::get('user/profile', function () {
            // Uses first & second middleware...
        });
    });

<a name="route-group-subdomain-routing"></a>

### Маршрутизація субдоменів

Групи маршрутів також можуть використовуватися для обробки маршрутизації субдоменів. Субдоменам можуть бути призначені параметри маршруту, як і URI маршруту, що дозволяє захопити частину субдомену для використання у вашому маршруті або контролері. Субдомен можна вказати, зателефонувавши до`domain`метод перед визначенням групи:

    Route::domain('{account}.myapp.com')->group(function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

> {note} Для того, щоб забезпечити доступність ваших маршрутів субдомену, вам слід зареєструвати маршрути субдомену перед реєстрацією маршрутів кореневого домену. Це не дозволить маршрутам кореневого домену перезаписувати маршрути субдоменів, які мають однаковий шлях URI.

<a name="route-group-prefixes"></a>

### Префікси маршрутів

`prefix`метод може використовуватися для префікса кожного маршруту в групі із заданим URI. Наприклад, вам може знадобитися додати до всіх URI маршрутів у групі префікс`admin`:

    Route::prefix('admin')->group(function () {
        Route::get('users', function () {
            // Matches The "/admin/users" URL
        });
    });

<a name="route-group-name-prefixes"></a>

### Префікси імен маршрутів

`name`метод може використовуватися для префікса кожного імені маршруту в групі заданим рядком. Наприклад, вам може знадобитися префікс всіх назв згрупованих маршрутів`admin`. Даний рядок має префікс до назви маршруту точно так, як це вказано, тому ми обов’язково надамо кінцевий результат`.`символ у префіксі:

    Route::name('admin.')->group(function () {
        Route::get('users', function () {
            // Route assigned name "admin.users"...
        })->name('users');
    });

<a name="route-model-binding"></a>

## Прив'язка моделі маршруту

Вводячи ідентифікатор моделі в маршрут або дію контролера, ви часто будете запитувати, щоб отримати модель, яка відповідає цьому ідентифікатору. Прив'язка моделі маршруту Laravel забезпечує зручний спосіб автоматичного введення екземплярів моделі безпосередньо у ваші маршрути. Наприклад, замість того, щоб вводити ідентифікатор користувача, ви можете ввести весь`User`екземпляр моделі, який відповідає заданому ідентифікатору.

<a name="implicit-binding"></a>

### Неявне прив'язування

Laravel автоматично вирішує красномовні моделі, визначені в маршрутах або діях контролера, імена змінних яких наведених на тип відповідають імені сегмента маршруту. Наприклад:

    Route::get('api/users/{user}', function (App\Models\User $user) {
        return $user->email;
    });

Так як`$user`змінна наводиться на тип як`App\Models\User`Красномовна модель та назва змінної відповідає`{user}`Сегмент URI, Laravel автоматично вводить екземпляр моделі, який має ідентифікатор, що відповідає відповідному значенню з URI запиту. Якщо відповідний екземпляр моделі не знайдено в базі даних, відповідь 404 HTTP буде автоматично сформовано.

Звичайно, неявне прив'язування також можливе при використанні методів контролера. Знову зверніть увагу на`{user}`Сегмент URI відповідає`$user`змінна в контролері, яка містить`App\Models\User`тип-підказка:

    use App\Http\Controllers\UserController;
    use App\Models\User;

    Route::get('users/{user}', [UserController::class, 'show']);

    public function show(User $user)
    {
        return view('user.profile', ['user' => $user]);
    }

<a name="customizing-the-key"></a>

#### Налаштування ключа

Іноді вам може знадобитися вирішити красномовні моделі, використовуючи стовпець, відмінний від`id`. Для цього ви можете вказати стовпець у визначенні параметрів маршруту:

    Route::get('api/posts/{post:slug}', function (App\Models\Post $post) {
        return $post;
    });

<a name="implicit-model-binding-scoping"></a>

#### Спеціальні ключі та сфера дії

Іноді, при неявному прив’язуванні декількох Eloquent моделей в одному визначенні маршруту, можливо, ви захочете застосувати другу модель красномовства таким чином, що вона повинна бути дочірньою до першої красномовної моделі. Наприклад, розглянемо цю ситуацію, яка отримує допис у блозі від slug для певного користувача:

    use App\Models\Post;
    use App\Models\User;

    Route::get('api/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
        return $post;
    });

При використанні користувацького неявного прив’язування як вкладеного параметра маршруту, Laravel автоматично застосує запит для отримання вкладеної моделі своїм батьком, використовуючи конвенції, щоб вгадати ім’я зв’язку на батьківському. У цьому випадку передбачається, що`User`модель має відносини з іменем`posts`(множина імені параметра маршруту), який можна використовувати для отримання`Post`модель.

<a name="customizing-the-default-key-name"></a>

#### Налаштування імені ключа за замовчуванням

Якщо ви хочете, щоб прив'язка моделі використовувала стовпець бази даних за замовчуванням, відмінний від`id`при отриманні заданого класу моделі ви можете замінити`getRouteKeyName`на красномовній моделі:

    /**
     * Get the route key for the model.
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

<a name="explicit-binding"></a>

### Явне прив'язування

Щоб зареєструвати явне прив'язування, використовуйте маршрутизатор`model`метод, щоб вказати клас для даного параметра. Ви повинні визначити свої явні прив'язки моделей на початку`boot`метод вашого`RouteServiceProvider`клас:

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @return void
     */
    public function boot()
    {
        Route::model('user', \App\Models\User::class);

        // ...
    }

Далі визначте маршрут, який містить`{user}`параметр:

    Route::get('profile/{user}', function (App\Models\User $user) {
        //
    });

Оскільки ми зв'язали всіх`{user}`параметрів до`App\Models\User`модель, a`User`екземпляр буде введений у маршрут. Так, наприклад, запит до`profile/1`буде вводити`User`з бази даних, що має ідентифікатор`1`.

Якщо відповідний екземпляр моделі не знайдено в базі даних, відповідь 404 HTTP буде автоматично сформовано.

<a name="customizing-the-resolution-logic"></a>

#### Налаштування логіки роздільної здатності

Якщо ви хочете використовувати свою власну логіку роздільної здатності, ви можете використовувати`Route::bind`метод.`Closure`ви переходите до`bind`метод отримає значення сегменту URI і повинен повернути екземпляр класу, який слід ввести в маршрут:

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @return void
     */
    public function boot()
    {
        Route::bind('user', function ($value) {
            return App\Models\User::where('name', $value)->firstOrFail();
        });

        // ...
    }

Крім того, ви можете замінити`resolveRouteBinding`метод на вашій красномовній моделі. Цей метод отримає значення сегменту URI і повинен повернути екземпляр класу, який слід ввести в маршрут:

    /**
     * Retrieve the model for a bound value.
     *
     * @param  mixed  $value
     * @param  string|null  $field
     * @return \Illuminate\Database\Eloquent\Model|null
     */
    public function resolveRouteBinding($value, $field = null)
    {
        return $this->where('name', $value)->firstOrFail();
    }

Якщо маршрут використовується[неявне прив'язування](#implicit-model-binding-scoping),`resolveChildRouteBinding`метод буде використаний для вирішення дочірнього прив'язки батьківської моделі:

    /**
     * Retrieve the child model for a bound value.
     *
     * @param  string  $childType
     * @param  mixed  $value
     * @param  string|null  $field
     * @return \Illuminate\Database\Eloquent\Model|null
     */
    public function resolveChildRouteBinding($childType, $value, $field)
    {
        return parent::resolveChildRouteBinding($childType, $value, $field);
    }

<a name="fallback-routes"></a>

## Запасні маршрути

Використання`Route::fallback`методом, ви можете визначити маршрут, який буде виконаний, коли жоден інший маршрут не відповідає вхідному запиту. Як правило, необроблені запити автоматично відображають сторінку "404" через обробник винятків вашої програми. Однак, оскільки ви можете визначити`fallback`маршрут у межах вашого`routes/web.php`файл, все Middlware в`web`до маршруту застосовуватиметься група проміжного програмного забезпечення. Ви можете додавати додаткове Middlware до цього маршруту за потреби:

    Route::fallback(function () {
        //
    });

> {note} Запасний маршрут завжди повинен бути останнім маршрутом, зареєстрованим вашим додатком.

<a name="rate-limiting"></a>

## Обмеження ставки

<a name="defining-rate-limiters"></a>

### Визначення обмежувачів ставок

Laravel включає потужні та настроювані послуги обмеження швидкості, які ви можете використовувати для обмеження обсягу трафіку для певного маршруту або групи маршрутів. Для початку слід визначити конфігурації обмежувача швидкості, які відповідають потребам вашої програми. Як правило, це можна зробити у вашій програмі`RouteServiceProvider`.

Обмежувачі ставок визначаються за допомогою`RateLimiter`фасадні`for`метод.`for`метод приймає ім'я обмежувача швидкості та Закриття, яке повертає конфігурацію обмеження, яка повинна застосовуватися до маршрутів, яким призначений цей обмежувач швидкості:

    use Illuminate\Cache\RateLimiting\Limit;
    use Illuminate\Support\Facades\RateLimiter;

    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000);
    });

Якщо вхідний запит перевищує вказаний ліміт швидкості, відповідь із кодом стану 429 HTTP автоматично повернеться Laravel. Якщо ви хочете визначити власну відповідь, яку слід повернути з обмеженням швидкості, ви можете скористатися`response`метод:

    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000)->response(function () {
            return response('Custom response...', 429);
        });
    });

Оскільки зворотні виклики обмежувача швидкості отримують вхідний екземпляр запиту HTTP, ви можете динамічно створювати відповідне обмеження швидкості на основі вхідного запиту або автентифікованого користувача:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()->vipCustomer()
                    ? Limit::none()
                    : Limit::perMinute(100);
    });

<a name="segmenting-rate-limits"></a>

#### Сегментування обмежень ставок

Іноді вам може знадобитися сегментувати обмеження тарифів на якесь довільне значення. Наприклад, ви можете дозволити користувачам отримати доступ до даного маршруту 100 разів на хвилину за IP-адресою. Для цього ви можете використовувати`by`метод при побудові обмеження тарифу:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()->vipCustomer()
                    ? Limit::none()
                    : Limit::perMinute(100)->by($request->ip());
    });

<a name="multiple-rate-limits"></a>

#### Кілька обмежень тарифу

Якщо потрібно, ви можете повернути масив обмежень швидкості для заданої конфігурації обмежувача швидкості. Кожне обмеження тарифу буде оцінено для маршруту на основі порядку, розміщеного в масиві:

    RateLimiter::for('login', function (Request $request) {
        return [
            Limit::perMinute(500),
            Limit::perMinute(3)->by($request->input('email')),
        ];
    });

<a name="attaching-rate-limiters-to-routes"></a>

### Прикріплення обмежувачів ставок до маршрутів

Обмежувачі ставок можуть бути прикріплені до маршрутів або груп маршрутів за допомогою`throttle`[Middlware](/docs/{{version}}/middleware). Проміжне програмне забезпечення дросельної заслінки приймає назву обмежувача швидкості, який ви хочете призначити маршруту:

    Route::middleware(['throttle:uploads'])->group(function () {
        Route::post('/audio', function () {
            //
        });

        Route::post('/video', function () {
            //
        });
    });

<a name="throttling-with-redis"></a>

#### Дротування з Редісом

Як правило,`throttle`Middlware відображається на`Illuminate\Routing\Middleware\ThrottleRequests`клас. Це відображення визначено у ядрі HTTP вашої програми. Однак, якщо ви використовуєте Redis як драйвер кешу вашої програми, можливо, ви захочете змінити це відображення, щоб використовувати`Illuminate\Routing\Middleware\ThrottleRequestsWithRedis`клас. Цей клас ефективніше управляє обмеженням швидкості за допомогою Redis:

    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,

<a name="form-method-spoofing"></a>

## Метод підміни форми

Форми HTML не підтримують`PUT`,`PATCH`або`DELETE`дії. Отже, при визначенні`PUT`,`PATCH`або`DELETE`маршрути, які викликаються з HTML-форми, вам потрібно буде додати прихований`_method`поле до форми. Значення, надіслане з`_method`поле буде використано як метод запиту HTTP&#x3A;

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

Ви можете використовувати`@method`Директива Blade для створення`_method`вхід:

    <form action="/foo/bar" method="POST">
        @method('PUT')
        @csrf
    </form>

<a name="accessing-the-current-route"></a>

## Доступ до поточного маршруту

Ви можете використовувати`current`,`currentRouteName`, and `currentRouteAction`методи на`Route`фасад для доступу до інформації про маршрут, який обробляє вхідний запит:

    $route = Route::current();

    $name = Route::currentRouteName();

    $action = Route::currentRouteAction();

Зверніться до документації API для обох[основний клас фасаду маршруту](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)і[Екземпляр маршруту](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html)переглянути всі доступні методи.

<a name="cors"></a>

## Спільне використання ресурсів (CORS)

Laravel може автоматично відповідати на запити CORS OPTIONS зі значеннями, які ви налаштовуєте. Усі налаштування CORS можуть бути налаштовані у вашому`cors`файл конфігурації та запити OPTIONS автоматично оброблятимуться`HandleCors`Middlware, яке за замовчуванням включено у ваш глобальний стек проміжного програмного забезпечення.

> {tip} Щоб отримати додаткову інформацію про CORS та заголовки CORS, зверніться до[Веб-документація MDN щодо CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#The_HTTP_response_headers).
