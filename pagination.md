# База даних: пагінація

-   [Вступ](#introduction)
-   [Базове використання](#basic-usage)
    -   [Paginating Query Builder Results](#paginating-query-builder-results)
    -   [Paginating Eloquent результатів](#paginating-eloquent-results)
    -   [Створення вручну Paginator](#manually-creating-a-paginator)
-   [Відображення результатів пагінації](#displaying-pagination-results)
    -   [Перетворення результатів у JSON](#converting-results-to-json)
-   [Налаштування views пагінації](#customizing-the-pagination-view)
    -   [Використання Bootstrap](#using-bootstrap)
-   [Методи екземпляра Paginator](#paginator-instance-methods)

<a name="introduction"></a>

## Вступ

В інших рамках пагінація може бути дуже болючою. Пагінатор Laravel інтегрований з[конструктор запитів](/docs/{{version}}/queries)і[Красномовний ОРМ](/docs/{{version}}/eloquent)і забезпечує зручну, просту у використанні пагінацію результатів баз даних нестандартно. За замовчуванням HTML, створений пагінатором, сумісний з[Фреймворк CSS Tailwind](https://tailwindcss.com/); однак доступні також views Bootstrap.

<a name="basic-usage"></a>

## Базове використання

<a name="paginating-query-builder-results"></a>

### Paginating Query Builder Results

Є кілька способів перенесення сторінок на сторінки. Найпростішим є використання`paginate`метод на[конструктор запитів](/docs/{{version}}/queries)або an[Красномовний запит](/docs/{{version}}/eloquent).`paginate`метод автоматично дбає про встановлення належного ліміту та зміщення на основі поточної сторінки, яку переглядає користувач. За замовчуванням поточна сторінка визначається за значенням`page`аргумент рядка запиту на запит HTTP. Це значення автоматично визначається Laravel, а також автоматично вставляється в посилання, створені пагінатором.

У цьому прикладі єдиний аргумент, переданий в`paginate`метод - це кількість елементів, які ви хотіли б відображати "на сторінку". У цьому випадку вкажемо, що ми хотіли б відобразити`15`елементів на сторінці:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\DB;

    class UserController extends Controller
    {
        /**
         * Show all of the users for the application.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->paginate(15);

            return view('user.index', ['users' => $users]);
        }
    }

> {note} В даний час операції з нумерацією сторінок, які використовують a`groupBy`оператор не може бути ефективно виконаний Laravel. Якщо вам потрібно використовувати a`groupBy`з набором сторінок із сторінками рекомендується здійснити запит до бази даних і створити пагінатор вручну.

<a name="simple-pagination"></a>

#### "Проста пагінація"

Якщо вам потрібно лише відобразити прості посилання "Далі" та "Попередній" у поданні пагінації, ви можете використовувати`simplePaginate`метод для виконання більш ефективного запиту. Це дуже корисно для великих наборів даних, коли вам не потрібно відображати посилання для кожного номера сторінки під час рендерингу вашого views:

    $users = DB::table('users')->simplePaginate(15);

<a name="paginating-eloquent-results"></a>

### Paginating Eloquent результатів

Ви також можете зробити сторінку[Красномовний](/docs/{{version}}/eloquent)запитів. У цьому прикладі ми передамо сторінку`User`модель з`15`елементів на сторінці. Як бачите, синтаксис майже ідентичний результатам побудови пошукових запитів:

    $users = App\Models\User::paginate(15);

Ви можете зателефонувати`paginate`після встановлення інших обмежень для запиту, таких як`where`речення:

    $users = User::where('votes', '>', 100)->paginate(15);

Ви також можете використовувати`simplePaginate`метод підбиття сторінок Eloquent моделей:

    $users = User::where('votes', '>', 100)->simplePaginate(15);

<a name="manually-creating-a-paginator"></a>

### Створення Paginator вручну

Іноді вам може знадобитися створити екземпляр пагінації вручну, передавши йому масив елементів. Ви можете зробити це, створивши або`Illuminate\Pagination\Paginator`або`Illuminate\Pagination\LengthAwarePaginator`наприклад, залежно від ваших потреб.

`Paginator`класу не потрібно знати загальну кількість елементів у наборі результатів; однак через це клас не має методів для отримання індексу останньої сторінки.`LengthAwarePaginator`приймає майже ті самі аргументи, що і`Paginator`; однак для цього потрібен підрахунок загальної кількості елементів у наборі результатів.

Іншими словами,`Paginator`відповідає`simplePaginate`метод на конструкторі запитів та Eloquent, тоді як`LengthAwarePaginator`відповідає`paginate`метод.

> {note} Під час створення вручну екземпляра пагінатора слід вручну "нарізати" масив результатів, які ви передаєте пагінатору. Якщо ви не впевнені, як це зробити, перегляньте[array_slice](https://secure.php.net/manual/en/function.array-slice.php)Функція PHP.

<a name="displaying-pagination-results"></a>

## Відображення результатів пагінації

При дзвінку в`paginate`метод, ви отримаєте екземпляр`Illuminate\Pagination\LengthAwarePaginator`. При дзвінку в`simplePaginate`метод, ви отримаєте екземпляр`Illuminate\Pagination\Paginator`. Ці об'єкти забезпечують декілька методів, що описують набір результатів. На додаток до цих допоміжних методів, екземпляри пагінатора є ітераторами і можуть бути циклічними як циклічний масив. Отже, після того, як ви отримали результати, ви можете відобразити результати та відтворити посилання на сторінки за допомогою[Blade](/docs/{{version}}/blade):

    <div class="container">
        @foreach ($users as $user)
            {{ $user->name }}
        @endforeach
    </div>

    {{ $users->links() }}

`links`метод відобразить посилання на решту сторінок у наборі результатів. Кожне з цих посилань уже міститиме належне`page`змінна рядка запиту. Пам'ятайте, HTML, сформований`links`метод сумісний з[Фреймворк CSS Tailwind](https://tailwindcss.com).

<a name="customizing-the-paginator-uri"></a>

#### Налаштування URI Paginator

`withPath`Метод дозволяє налаштувати URI, який використовується пагінатором при генерації посилань. Наприклад, якщо ви хочете, щоб пагінатор генерував такі посилання`http://example.com/custom/url?page=N`, вам слід пройти`custom/url`до`withPath`метод:

    Route::get('users', function () {
        $users = App\Models\User::paginate(15);

        $users->withPath('custom/url');

        //
    });

<a name="appending-to-pagination-links"></a>

#### Додавання до посилань на пагінацію

Ви можете додати до рядка запиту посилання на пагінацію за допомогою`appends`метод. Наприклад, додати`sort=votes`до кожного посилання на пагінацію, вам слід зробити наступний дзвінок`appends`:

    {{ $users->appends(['sort' => 'votes'])->links() }}

Якщо ви хочете додати всі поточні значення рядка запиту до посилань на пагінацію, ви можете використовувати`withQueryString`метод:

    {{ $users->withQueryString()->links() }}

Якщо ви хочете додати "хеш-фрагмент" до URL-адрес пагінатора, ви можете використовувати`fragment`метод. Наприклад, додати`#foo`до кінця кожного посилання на пагінацію зробіть наступний дзвінок на`fragment`метод:

    {{ $users->fragment('foo')->links() }}

<a name="adjusting-the-pagination-link-window"></a>

#### Налаштування вікна посилання на пагінацію

Ви можете контролювати, скільки додаткових посилань відображається з кожного боку "вікна" URL-адреси пагінатора. За замовчуванням три посилання відображаються на кожній стороні основних посилань сторінок. Однак ви можете керувати цим номером за допомогою`onEachSide`метод:

    {{ $users->onEachSide(5)->links() }}

<a name="converting-results-to-json"></a>

### Перетворення результатів у JSON

Класи результатів пагінатора Laravel реалізують`Illuminate\Contracts\Support\Jsonable`Інтерфейс контракту і викрити`toJson`, тому дуже легко перетворити результати пагінації в JSON. Ви також можете перетворити екземпляр пагінатора в JSON, повернувши його з дії маршруту або контролера:

    Route::get('users', function () {
        return App\Models\User::paginate();
    });

JSON від пагінатора включатиме таку інформацію, як`total`,`current_page`,`last_page`, і більше. Фактичні об'єкти результатів будуть доступні через`data`ключ у масиві JSON. Ось приклад JSON, створеного поверненням екземпляра пагінатора з маршруту:

    {
       "total": 50,
       "per_page": 15,
       "current_page": 1,
       "last_page": 4,
       "first_page_url": "http://laravel.app?page=1",
       "last_page_url": "http://laravel.app?page=4",
       "next_page_url": "http://laravel.app?page=2",
       "prev_page_url": null,
       "path": "http://laravel.app",
       "from": 1,
       "to": 15,
       "data":[
            {
                // Result Object
            },
            {
                // Result Object
            }
       ]
    }

<a name="customizing-the-pagination-view"></a>

## Налаштування views пагінації

За замовчуванням views, що відображаються для відображення посилань на пагінацію, сумісні з фреймворком Tailwind CSS. Однак, якщо ви не використовуєте Tailwind, ви можете самостійно визначати власні погляди для відтворення цих посилань. При дзвінку в`links`методу на екземплярі пагінатора, передайте ім'я views як перший аргумент методу:

    {{ $paginator->links('view.name') }}

    // Passing data to the view...
    {{ $paginator->links('view.name', ['foo' => 'bar']) }}

Однак найпростіший спосіб налаштувати views сторінок з розміщенням сторінок - експортувати їх до вашого`resources/views/vendor`каталог за допомогою`vendor:publish`команда:

    php artisan vendor:publish --tag=laravel-pagination

Ця команда розмістить views у`resources/views/vendor/pagination`каталог.`tailwind.blade.php`файл у цьому каталозі відповідає поданню сторінок за замовчуванням. Ви можете редагувати цей файл, щоб змінити HTML пагінації.

Якщо ви хочете призначити інший файл як views сторінок за замовчуванням, ви можете скористатися файлом пагінатора`defaultView`і`defaultSimpleView`методів у вашому`AppServiceProvider`:

    use Illuminate\Pagination\Paginator;

    public function boot()
    {
        Paginator::defaultView('view-name');

        Paginator::defaultSimpleView('view-name');
    }

<a name="using-bootstrap"></a>

### Використання Bootstrap

Laravel включає пагінаційні views, побудовані за допомогою[Bootstrap CSS](https://getbootstrap.com/). Щоб використовувати ці views замість стандартних Views Tailwind, ви можете зателефонувати до пагінатора`useBootstrap`метод у вашому`AppServiceProvider`:

    use Illuminate\Pagination\Paginator;

    public function boot()
    {
        Paginator::useBootstrap();
    }

<a name="paginator-instance-methods"></a>

## Методи екземпляра Paginator

Кожен екземпляр пагінатора надає додаткову інформацію про пагінацію за допомогою таких методів:

| Метод                                   | Опис                                                                                                                  |
| --------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `$paginator->count()`                   | Отримайте кількість елементів для поточної сторінки.                                                                  |
| `$paginator->currentPage()`             | Отримайте номер поточної сторінки.                                                                                    |
| `$paginator->firstItem()`               | Отримайте номер результату першого елемента в результатах.                                                            |
| `$paginator->getOptions()`              | Отримайте параметри пагінатора.                                                                                       |
| `$paginator->getUrlRange($start, $end)` | Створіть діапазон URL-адрес з пагінацією.                                                                             |
| `$paginator->hasPages()`                | Визначте, чи достатньо елементів для розділення на кілька сторінок.                                                   |
| `$paginator->hasMorePages()`            | Визначте, чи більше елементів у сховищі даних.                                                                        |
| `$paginator->items()`                   | Отримайте елементи для поточної сторінки.                                                                             |
| `$paginator->lastItem()`                | Отримайте номер результату останнього елемента в результатах.                                                         |
| `$paginator->lastPage()`                | Отримайте номер останньої доступної сторінки. (Недоступно під час використання`simplePaginate`).                      |
| `$paginator->nextPageUrl()`             | Отримайте URL-адресу для наступної сторінки.                                                                          |
| `$paginator->onFirstPage()`             | Визначте, чи є сторінка першокласником.                                                                               |
| `$paginator->perPage()`                 | Кількість елементів, що відображаються на сторінці.                                                                   |
| `$paginator->previousPageUrl()`         | Отримайте URL-адресу попередньої сторінки.                                                                            |
| `$paginator->total()`                   | Визначте загальну кількість відповідних елементів у сховищі даних. (Недоступно під час використання`simplePaginate`). |
| `$paginator->url($page)`                | Отримайте URL-адресу для вказаного номера сторінки.                                                                   |
| `$paginator->getPageName()`             | Отримайте змінну рядка запиту, яка використовується для зберігання сторінки.                                          |
| `$paginator->setPageName($name)`        | Встановіть змінну рядка запиту, яка використовується для зберігання сторінки.                                         |
