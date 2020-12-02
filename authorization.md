# Авторизація

-   [Вступ](#introduction)
-   [Ворота](#gates)
    -   [Написання Gates](#writing-gates)
    -   [Авторизаційні Actions](#authorizing-actions-via-gates)
    -   [Відповіді на Gate](#gate-responses)
    -   [Перехоплення Gate Перевірок](#intercepting-gate-checks)
-   [Створення Policies](#creating-policies)
    -   [Генерація Policies](#generating-policies)
    -   [Реєстрація Policies](#registering-policies)
-   [Написання Policies](#writing-policies)
    -   [Методи Policies](#policy-methods)
    -   [Відповіді Policies](#policy-responses)
    -   [Методи без моделей](#methods-without-models)
    -   [Гостьові користувачів](#guest-users)
    -   [Policy фільтри](#policy-filters)
-   [Авторизація дій із використанням політик](#authorizing-actions-using-policies)
    -   [За допомогою моделі користувача](#via-the-user-model)
    -   [За допомогою Middleware](#via-middleware)
    -   [За допомогою Controller Helpers](#via-controller-helpers)
    -   [За допомогою шаблонів Blade](#via-blade-templates)
    -   [Надання додаткового контексту](#supplying-additional-context)

<a name="introduction"></a>

## Вступ

Крім забезпечення[автентифікації](/docs/{{version}}/authentication)нестандартних служб, Laravel також пропонує простий спосіб санкціонувати дії користувачів щодо даного ресурсу. Як і автентифікація, підхід Laravel до авторизації простий, і є два основних способи санкціонування дій: ворота та політики.

Подумайте про ворота та правила, як маршрути та контролери. Gates забезпечують простий підхід до авторизації на основі закриття, тоді як політики, такі як контролери, групують свою логіку навколо певної моделі чи ресурсу. Спочатку ми дослідимо ворота, а потім вивчимо політику.

Вам не потрібно вибирати між виключно використанням воріт або виключно використанням політик під час створення додатка. Більшість програм, швидше за все, міститимуть суміш шлюзів і правил, і це цілком нормально! Шлюзи найбільш застосовні до дій, які не пов’язані з жодною моделлю чи ресурсом, наприклад, перегляд інформаційної панелі адміністратора. На відміну від цього, політики слід використовувати, коли ви хочете дозволити дію для певної моделі чи ресурсу.

<a name="gates"></a>

## Gates

<a name="writing-gates"></a>

### Написання Gates

Gates - це закриття, які визначають, чи має користувач дозвіл виконувати певну дію, і, як правило, визначаються в`App\Providers\AuthServiceProvider`клас за допомогою`Gate`фасад. Гейтс завжди отримує екземпляр користувача як перший аргумент, а за бажанням може отримувати додаткові аргументи, такі як відповідна модель Eloquent:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('edit-settings', function ($user) {
            return $user->isAdmin;
        });

        Gate::define('update-post', function ($user, $post) {
            return $user->id === $post->user_id;
        });
    }

Gates також можуть бути визначені за допомогою масиву зворотного виклику класу, як контролери:

    use App\Policies\PostPolicy;

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', [PostPolicy::class, 'update']);
    }

<a name="authorizing-actions-via-gates"></a>

### Авторизаційні дії

Щоб дозволити дію за допомогою воріт, вам слід використовувати`allows`або`denies`методи. Зверніть увагу, що вам не потрібно передавати поточно перевіреного користувача цим методам. Laravel автоматично подбає про передачу користувача в Закриття воріт:

    if (Gate::allows('edit-settings')) {
        // The current user can edit settings
    }

    if (Gate::allows('update-post', $post)) {
        // The current user can update the post...
    }

    if (Gate::denies('update-post', $post)) {
        // The current user can't update the post...
    }

Якщо ви хочете визначити, чи конкретний користувач уповноважений виконувати дію, ви можете використовувати`forUser`метод на`Gate`фасад:

    if (Gate::forUser($user)->allows('update-post', $post)) {
        // The user can update the post...
    }

    if (Gate::forUser($user)->denies('update-post', $post)) {
        // The user can't update the post...
    }

Ви можете санкціонувати кілька дій одночасно за допомогою`any`або`none`методи:

    if (Gate::any(['update-post', 'delete-post'], $post)) {
        // The user can update or delete the post
    }

    if (Gate::none(['update-post', 'delete-post'], $post)) {
        // The user cannot update or delete the post
    }

<a name="authorizing-or-throwing-exceptions"></a>

#### Авторизація або викидання винятків

Якщо ви хочете спробувати дозволити дію та автоматично кинути`Illuminate\Auth\Access\AuthorizationException`якщо користувачеві не дозволено виконати дану дію, ви можете використовувати`Gate::authorize`метод. Екземпляри`AuthorizationException`автоматично перетворюються на`403`Відповідь HTTP&#x3A;

    Gate::authorize('update-post', $post);

    // The action is authorized...

<a name="gates-supplying-additional-context"></a>

#### Надання додаткового контексту

Методи шлюзу для авторизації здібностей (`allows`,`denies`,`check`,`any`,`none`,`authorize`,`can`,`cannot`) та дозвіл[Директиви щодо леза](#via-blade-templates)(`@can`,`@cannot`,`@canany`) може отримати масив як другий аргумент. Ці елементи масиву передаються в якості параметрів на шлюз і можуть бути використані для додаткового контексту при прийнятті рішень щодо авторизації:

    Gate::define('create-post', function ($user, $category, $extraFlag) {
        return $category->group > 3 && $extraFlag === true;
    });

    if (Gate::check('create-post', [$category, $extraFlag])) {
        // The user can create the post...
    }

<a name="gate-responses"></a>

### Відповіді на ворота

Наразі ми розглянули лише ворота, які повертають прості булеві значення. Однак іноді вам може знадобитися повернути більш детальну відповідь, включаючи повідомлення про помилку. Для цього ви можете повернути`Illuminate\Auth\Access\Response`від ваших воріт:

    use Illuminate\Auth\Access\Response;
    use Illuminate\Support\Facades\Gate;

    Gate::define('edit-settings', function ($user) {
        return $user->isAdmin
                    ? Response::allow()
                    : Response::deny('You must be a super administrator.');
    });

Коли ви повертаєте відповідь авторизації з вашого входу,`Gate::allows`метод все одно поверне просте логічне значення; однак ви можете використовувати`Gate::inspect`спосіб отримати повну відповідь авторизації, що повертається воротами:

    $response = Gate::inspect('edit-settings', $post);

    if ($response->allowed()) {
        // The action is authorized...
    } else {
        echo $response->message();
    }

Звичайно, при використанні`Gate::authorize`метод кинути`AuthorizationException`якщо дія не авторизована, повідомлення про помилку, надане відповіддю авторизації, буде передано до відповіді HTTP&#x3A;

    Gate::authorize('edit-settings', $post);

    // The action is authorized...

<a name="intercepting-gate-checks"></a>

### Перехоплення перевірок воріт

Іноді ви можете надати всі можливості певному користувачеві. Ви можете використовувати`before`метод для визначення зворотного виклику, який запускається перед усіма іншими перевірками авторизації:

    Gate::before(function ($user, $ability) {
        if ($user->isSuperAdmin()) {
            return true;
        }
    });

Якщо`before`callback повертає ненульовий результат, який вважатиметься результатом перевірки.

Ви можете використовувати`after`метод визначення зворотного виклику, який буде виконаний після всіх інших перевірок авторизації:

    Gate::after(function ($user, $ability, $result, $arguments) {
        if ($user->isSuperAdmin()) {
            return true;
        }
    });

Подібно до`before`перевірити, якщо`after`callback повертає ненульовий результат, який вважатиметься результатом перевірки.

<a name="creating-policies"></a>

## Створення політики

<a name="generating-policies"></a>

### Створення політики

Політики - це класи, що організовують логіку авторизації навколо певної моделі чи ресурсу. Наприклад, якщо ваша програма є блогом, ви можете мати`Post`модель та відповідну`PostPolicy`дозволити дії користувачів, такі як створення або оновлення публікацій.

Ви можете створити політику, використовуючи`make:policy`[ремісниче командування](/docs/{{version}}/artisan). Створена політика буде розміщена в`app/Policies`каталог. Якщо цього каталогу немає у вашому додатку, Laravel створить його для вас:

    php artisan make:policy PostPolicy

`make:policy`команда генерує порожній клас політики. Якщо ви хочете створити клас з основними методами політики "CRUD", які вже включені в клас, ви можете вказати a`--model`при виконанні команди:

    php artisan make:policy PostPolicy --model=Post

> {tip} Усі правила вирішуються через Laravel[службовий контейнер](/docs/{{version}}/container), що дозволяє вам натякати на необхідні залежності в конструкторі політики, щоб їх автоматично вводити.

<a name="registering-policies"></a>

### Реєстрація політики

Коли політика існує, її потрібно зареєструвати.`AuthServiceProvider`, що входить до складу нових програм Laravel, містить`policies`властивість, яке відображає ваші красномовні моделі відповідно до їхніх політик. Реєстрація політики вкаже Laravel, яку політику використовувати під час санкціонування дій проти даної моделі:

    <?php

    namespace App\Providers;

    use App\Models\Post;
    use App\Policies\PostPolicy;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    use Illuminate\Support\Facades\Gate;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * The policy mappings for the application.
         *
         * @var array
         */
        protected $policies = [
            Post::class => PostPolicy::class,
        ];

        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            //
        }
    }

<a name="policy-auto-discovery"></a>

#### Політика автоматичного виявлення

Замість того, щоб реєструвати політику моделі вручну, Laravel може автоматично виявляти політики, якщо модель і політика відповідають стандартним правилам іменування Laravel. Зокрема, політика повинна відповідати a`Policies`каталог у каталозі, що містить ваші моделі, або вище. Так, наприклад, моделі можуть бути розміщені в`app/Models`в той час як політики можуть бути розміщені в`app/Policies`каталог. У цій ситуації Laravel перевірить наявність політик у`app/Models/Policies`тоді`app/Policies`. Крім того, назва політики повинна відповідати назві моделі та мати символ`Policy`суфікс. Отже, a`User`модель відповідала б a`UserPolicy`клас.

Якщо ви хочете надати власну логіку виявлення політики, ви можете зареєструвати власний зворотний виклик за допомогою`Gate::guessPolicyNamesUsing`метод. Як правило, цей метод слід викликати з`boot`метод вашої програми`AuthServiceProvider`:

    use Illuminate\Support\Facades\Gate;

    Gate::guessPolicyNamesUsing(function ($modelClass) {
        // return policy class name...
    });

> {note} Будь-яка політика, яка прямо вказана у вашому`AuthServiceProvider`матиме перевагу над будь-якими потенційно відкритими політиками.

<a name="writing-policies"></a>

## Написання політики

<a name="policy-methods"></a>

### Методи політики

Після того, як політика буде зареєстрована, ви можете додавати методи для кожної дії, яку вона санкціонує. Наприклад, визначимо`update`метод на нашому`PostPolicy`який визначає, чи дана`User`може оновити заданий`Post`екземпляр.

`update`метод отримає a`User`і a`Post`екземпляр як аргументи, і повинен повернутись`true`або`false`вказуючи, чи уповноважений користувач оновлювати дані`Post`. Отже, для цього прикладу, давайте перевіримо, чи користувач`id`відповідає`user_id`на посаді:

    <?php

    namespace App\Policies;

    use App\Models\Post;
    use App\Models\User;

    class PostPolicy
    {
        /**
         * Determine if the given post can be updated by the user.
         *
         * @param  \App\Models\User  $user
         * @param  \App\Models\Post  $post
         * @return bool
         */
        public function update(User $user, Post $post)
        {
            return $user->id === $post->user_id;
        }
    }

Ви можете продовжувати визначати додаткові методи в політиці, як це потрібно для різних дій, які вона санкціонує. Наприклад, ви можете визначити`view`або`delete`методи санкціонування різних`Post`дії, але пам’ятайте, що ви можете надавати своїм методам політики будь-яке ім’я, яке вам подобається.

> {tip} Якщо ви використовували`--model`при генерації вашої політики через консоль Artisan, вона вже міститиме методи для`viewAny`,`view`,`create`,`update`,`delete`,`restore`, і`forceDelete`дії.

<a name="policy-responses"></a>

### Відповіді на політику

Наразі ми розглядали лише методи політики, які повертають прості логічні значення. Однак іноді вам може знадобитися повернути більш детальну відповідь, включаючи повідомлення про помилку. Для цього ви можете повернути`Illuminate\Auth\Access\Response`з вашого методу політики:

    use Illuminate\Auth\Access\Response;

    /**
     * Determine if the given post can be updated by the user.
     *
     * @param  \App\Models\User  $user
     * @param  \App\Models\Post  $post
     * @return \Illuminate\Auth\Access\Response
     */
    public function update(User $user, Post $post)
    {
        return $user->id === $post->user_id
                    ? Response::allow()
                    : Response::deny('You do not own this post.');
    }

Повертаючи відповідь авторизації з вашої політики,`Gate::allows`метод все одно поверне просте логічне значення; однак ви можете використовувати`Gate::inspect`спосіб отримати повну відповідь авторизації, що повертається воротами:

    $response = Gate::inspect('update', $post);

    if ($response->allowed()) {
        // The action is authorized...
    } else {
        echo $response->message();
    }

Звичайно, при використанні`Gate::authorize`метод кинути`AuthorizationException`якщо дія не авторизована, повідомлення про помилку, надане відповіддю авторизації, буде передано до відповіді HTTP&#x3A;

    Gate::authorize('update', $post);

    // The action is authorized...

<a name="methods-without-models"></a>

### Методи без моделей

Деякі методи політики отримують лише поточно автентифікованого користувача, а не екземпляр авторизованої ними моделі. Така ситуація найчастіше зустрічається при наданні дозволу`create`дії. Наприклад, якщо ви створюєте щоденник, можливо, ви захочете перевірити, чи користувач взагалі уповноважений створювати будь-які дописи.

При визначенні методів політики, які не отримуватимуть екземпляр моделі, наприклад`create`метод, він не отримає екземпляр моделі. Натомість вам слід визначити метод як такий, що очікує лише автентифікованого користувача:

    /**
     * Determine if the given user can create posts.
     *
     * @param  \App\Models\User  $user
     * @return bool
     */
    public function create(User $user)
    {
        //
    }

<a name="guest-users"></a>

### Гість користувачів

За замовчуванням усі ворота та правила автоматично повертаються`false`якщо вхідний запит HTTP не був ініційований аутентифікованим користувачем. Тим не менш, ви можете дозволити цим перевіркам авторизації пройти до ваших воріт і політик, оголосивши "необов'язковий" підказку типу або надавши`null`значення за замовчуванням для визначення аргументу користувача:

    <?php

    namespace App\Policies;

    use App\Models\Post;
    use App\Models\User;

    class PostPolicy
    {
        /**
         * Determine if the given post can be updated by the user.
         *
         * @param  \App\Models\User  $user
         * @param  \App\Models\Post  $post
         * @return bool
         */
        public function update(?User $user, Post $post)
        {
            return optional($user)->id === $post->user_id;
        }
    }

<a name="policy-filters"></a>

### Політичні фільтри

Для певних користувачів ви можете дозволити всі дії в межах певної політики. Для цього визначте a`before`метод на полісі.`before`метод буде виконаний перед будь-якими іншими методами у політиці, що дає вам можливість санкціонувати дію до того, як запланований метод політики буде фактично викликаний. Ця функція найчастіше використовується для уповноваження адміністраторів додатків виконувати будь-які дії:

    public function before($user, $ability)
    {
        if ($user->isSuperAdmin()) {
            return true;
        }
    }

Якщо ви хочете заборонити всі дозволи для користувача, вам слід повернутися`false`від`before`метод. Якщо`null`повернеться, авторизація перейде до методу політики.

> {примітка}`before`метод класу політики не буде викликаний, якщо клас не містить методу з іменем, що відповідає імені перевіряється здатності.

<a name="authorizing-actions-using-policies"></a>

## Авторизація дій із використанням політик

<a name="via-the-user-model"></a>

### Через модель користувача

`User`модель, яка входить до програми Laravel, включає два корисні методи для санкціонування дій:`can`і`cant`.`can`метод отримує дію, яку ви хочете дозволити, та відповідну модель. Наприклад, давайте визначимо, чи уповноважений користувач оновлювати певний`Post`модель:

    if ($user->can('update', $post)) {
        //
    }

Якщо[політика зареєстрована](#registering-policies)для даної моделі,`can`метод автоматично викликає відповідну політику і повертає логічний результат. Якщо для моделі не зареєстровано жодної політики,`can`метод спробує викликати шлюз на основі закриття, що відповідає вказаній назві дії.

<a name="user-model-actions-that-dont-require-models"></a>

#### Дії, які не потребують моделей

Пам'ятайте, деякі дії на зразок`create`може не вимагати екземпляра моделі. У цих ситуаціях ви можете передати назву класу на`can`метод. Ім'я класу буде використано для визначення політики, яку слід використовувати при авторизації дії:

    use App\Models\Post;

    if ($user->can('create', Post::class)) {
        // Executes the "create" method on the relevant policy...
    }

<a name="via-middleware"></a>

### Через Middleware

Laravel включає проміжне програмне забезпечення, яке може санкціонувати дії, перш ніж вхідний запит потрапить навіть до ваших маршрутів або контролерів. За замовчуванням`Illuminate\Auth\Middleware\Authorize`проміжне програмне забезпечення присвоєно`can`ключ у вашому`App\Http\Kernel`клас. Давайте розберемо приклад використання`can`проміжне програмне забезпечення, щоб дозволити користувачеві оновлювати допис у блозі:

    use App\Models\Post;

    Route::put('/post/{post}', function (Post $post) {
        // The current user may update the post...
    })->middleware('can:update,post');

У цьому прикладі ми передаємо`can`два аргументи проміжного програмного забезпечення. Перший - це назва дії, яку ми хочемо дозволити, а друга - параметр маршруту, який ми хочемо передати в метод політики. У цьому випадку, оскільки ми використовуємо[неявна прив'язка моделі](/docs/{{version}}/routing#implicit-binding), a`Post`модель буде передана методу політики. Якщо користувач не має права виконувати дану дію, відповідь HTTP із`403`код стану генерується проміжним програмним забезпеченням.

<a name="middleware-actions-that-dont-require-models"></a>

#### Дії, які не потребують моделей

Знову ж таки, такі дії, як`create`може не вимагати екземпляра моделі. У цих ситуаціях ви можете передати назву класу проміжному програмному забезпеченню. Ім'я класу буде використано для визначення політики, яку слід використовувати при авторизації дії:

    Route::post('/post', function () {
        // The current user may create posts...
    })->middleware('can:create,App\Models\Post');

<a name="via-controller-helpers"></a>

### Через помічники контролера

На додаток до корисних методів, наданих`User`модель, Laravel надає корисну інформацію`authorize`метод до будь-якого з ваших контролерів, які розширюють`App\Http\Controllers\Controller`базовий клас. Подобається`can`метод, цей метод приймає назву дії, яку ви хочете дозволити, та відповідну модель. Якщо дія не дозволена,`authorize`метод кине`Illuminate\Auth\Access\AuthorizationException`, який за замовчуванням обробник винятків Laravel перетворить у відповідь HTTP за допомогою`403`код стану:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\Post;
    use Illuminate\Http\Request;

    class PostController extends Controller
    {
        /**
         * Update the given blog post.
         *
         * @param  Request  $request
         * @param  Post  $post
         * @return Response
         * @throws \Illuminate\Auth\Access\AuthorizationException
         */
        public function update(Request $request, Post $post)
        {
            $this->authorize('update', $post);

            // The current user can update the blog post...
        }
    }

<a name="controller-actions-that-dont-require-models"></a>

#### Дії, які не потребують моделей

Як вже обговорювалося, деякі дії на зразок`create`може не вимагати екземпляра моделі. У цих ситуаціях вам слід передати назву класу в`authorize`метод. Ім'я класу буде використано для визначення політики, яку слід використовувати при авторизації дії:

    /**
     * Create a new blog post.
     *
     * @param  Request  $request
     * @return Response
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function create(Request $request)
    {
        $this->authorize('create', Post::class);

        // The current user can create blog posts...
    }

<a name="authorizing-resource-controllers"></a>

#### Авторизація контролерів ресурсів

Якщо ви використовуєте[контролери ресурсів](/docs/{{version}}/controllers#resource-controllers), ви можете скористатися`authorizeResource`метод у конструкторі контролера. Цей метод додасть відповідний`can`визначення проміжного програмного забезпечення до методів контролера ресурсів.

`authorizeResource`метод приймає ім'я класу моделі як перший аргумент, а ім'я параметра маршруту / запиту, який буде містити ідентифікатор моделі як другий аргумент. Ви повинні забезпечити свій[контролер ресурсів](/docs/{{version}}/controllers#resource-controllers)створюється за допомогою`--model`прапор, щоб мати необхідні підписи методу та підказки типу:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\Post;
    use Illuminate\Http\Request;

    class PostController extends Controller
    {
        public function __construct()
        {
            $this->authorizeResource(Post::class, 'post');
        }
    }

Наступні методи контролера будуть зіставлені з відповідним методом політики:

| Метод контролера | Метод політики |
| ---------------- | -------------- |
| індекс           | viewAny        |
| шоу              | вид            |
| створити         | створити       |
| магазин          | створити       |
| редагувати       | оновлення      |
| оновлення        | оновлення      |
| знищити          | видалити       |

> {tip} Ви можете використовувати`make:policy`команда за допомогою`--model`можливість швидкого створення класу політики для даної моделі:`php artisan make:policy PostPolicy --model=Post`.

<a name="via-blade-templates"></a>

### Через шаблони Blade

Під час написання шаблонів Blade, можливо, ви захочете відобразити частину сторінки, лише якщо користувач уповноважений виконувати певну дію. Наприклад, можливо, ви захочете показати форму оновлення для публікації в блозі, лише якщо користувач зможе насправді оновити публікацію. У цій ситуації ви можете використовувати`@can`і`@cannot`сімейство директив:

    @can('update', $post)
        <!-- The Current User Can Update The Post -->
    @elsecan('create', App\Models\Post::class)
        <!-- The Current User Can Create New Post -->
    @endcan

    @cannot('update', $post)
        <!-- The Current User Cannot Update The Post -->
    @elsecannot('create', App\Models\Post::class)
        <!-- The Current User Cannot Create A New Post -->
    @endcannot

Ці директиви є зручними комбінаціями клавіш для написання`@if`і`@unless`заяви.`@can`і`@cannot`наведені вище твердження відповідно перекладаються на такі твердження:

    @if (Auth::user()->can('update', $post))
        <!-- The Current User Can Update The Post -->
    @endif

    @unless (Auth::user()->can('update', $post))
        <!-- The Current User Cannot Update The Post -->
    @endunless

Ви також можете визначити, чи має користувач якісь можливості авторизації з певного переліку можливостей. Для цього скористайтеся`@canany`директива:

    @canany(['update', 'view', 'delete'], $post)
        // The current user can update, view, or delete the post
    @elsecanany(['create'], \App\Models\Post::class)
        // The current user can create a post
    @endcanany

<a name="blade-actions-that-dont-require-models"></a>

#### Дії, які не потребують моделей

Як і більшість інших методів авторизації, ви можете передати назву класу в`@can`і`@cannot`директиви, якщо для дії не потрібен екземпляр моделі:

    @can('create', App\Models\Post::class)
        <!-- The Current User Can Create Posts -->
    @endcan

    @cannot('create', App\Models\Post::class)
        <!-- The Current User Can't Create Posts -->
    @endcannot

<a name="supplying-additional-context"></a>

### Надання додаткового контексту

Під час авторизації дій із використанням політик ви можете передавати масив як другий аргумент різним функціям авторизації та помічникам. Перший елемент у масиві буде використаний для визначення, яку політику слід викликати, тоді як решта елементів масиву передаються як параметри методу політики і можуть бути використані для додаткового контексту при прийнятті рішень щодо авторизації. Наприклад, розглянемо наступне`PostPolicy`визначення методу, яке містить додатковий`$category`параметр:

    /**
     * Determine if the given post can be updated by the user.
     *
     * @param  \App\Models\User  $user
     * @param  \App\Models\  $post
     * @param  int  $category
     * @return bool
     */
    public function update(User $user, Post $post, int $category)
    {
        return $user->id === $post->user_id &&
               $category > 3;
    }

Під час спроби визначити, чи може автентифікований користувач оновити дану публікацію, ми можемо застосувати цей метод політики так:

    /**
     * Update the given blog post.
     *
     * @param  Request  $request
     * @param  Post  $post
     * @return Response
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function update(Request $request, Post $post)
    {
        $this->authorize('update', [$post, $request->input('category')]);

        // The current user can update the blog post...
    }
