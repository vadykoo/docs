# База даних: Конструктор запитів

-   [Вступ](#introduction)
-   [Отримання результатів](#retrieving-results)
    -   [Результати Chunking](#chunking-results)
    -   [Агрегати](#aggregates)
-   [Selects](#selects)
-   [Сирі(Raw) вирази](#raw-expressions)
-   [Joins](#joins)
-   [Unions](#unions)
-   [ Where Clauses](#where-clauses)
    -   [Групування параметрів](#parameter-grouping)
    -   [ Where існує Clauses](#where-exists-clauses)
    -   [Підзапит де Clauses](#subquery-where-clauses)
    -   [JSON  Where Clauses](#json-where-clauses)
-   [Впорядкування, групування, обмеження та зміщення](#ordering-grouping-limit-and-offset)
-   [Умовні Clauses](#conditional-clauses)
-   [Вставки](#inserts)
-   [Оновлення](#updates)
    -   [Оновлення стовпців JSON](#updating-json-columns)
    -   [Increment та Decrement](#increment-and-decrement)
-   [Deletes](#deletes)
-   [Песимістичний Locking](#pessimistic-locking)
-   [Debugging](#debugging)

<a name="introduction"></a>

## Вступ

Конструктор запитів до бази даних Laravel забезпечує зручний, вільний інтерфейс для створення та запуску запитів до бази даних. Він може бути використаний для виконання більшості операцій з базами даних у вашому додатку та працює на всіх підтримуваних системах баз даних.

Конструктор запитів Laravel використовує прив’язку параметрів PDO для захисту вашого додатка від атак введення SQL. Немає необхідності очищати рядки, що передаються як прив'язки.

> {note} PDO не підтримує прив’язку назв стовпців. Таким чином, ви ніколи не повинні дозволяти введенню користувачем диктувати імена стовпців, на які посилаються ваші запити, включаючи стовпці "упорядкування" тощо. Якщо ви повинні дозволити користувачеві вибрати певні стовпці, для яких потрібно здійснити запит, завжди перевіряйте імена стовпців у білих список дозволених стовпців.

<a name="retrieving-results"></a>

## Отримання результатів

<a name="retrieving-all-rows-from-a-table"></a>

#### Отримання всіх рядків із таблиці

Ви можете використовувати`table`метод на`DB`фасад, щоб розпочати запит.`table`Метод повертає екземпляр конструктора вільних запитів для даної таблиці, дозволяючи вам прив'язати більше обмежень до запиту, а потім нарешті отримати результати за допомогою`get`метод:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\DB;

    class UserController extends Controller
    {
        /**
         * Show a list of all of the application's users.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->get();

            return view('user.index', ['users' => $users]);
        }
    }

`get`метод повертає`Illuminate\Support\Collection`містять результати, де кожен результат є екземпляром PHP`stdClass`об'єкт. Ви можете отримати доступ до значення кожного стовпця, отримавши доступ до стовпця як властивості об’єкта:

    foreach ($users as $user) {
        echo $user->name;
    }

<a name="retrieving-a-single-row-column-from-a-table"></a>

#### Отримання одного рядка / стовпця зі таблиці

Якщо вам просто потрібно отримати один рядок з таблиці бази даних, ви можете використовувати`first`метод. Цей метод поверне сингл`stdClass`об'єкт:

    $user = DB::table('users')->where('name', 'John')->first();

    echo $user->name;

Якщо вам навіть не потрібен цілий рядок, ви можете виділити одне значення із запису за допомогою`value`метод. Цей метод поверне значення стовпця безпосередньо:

    $email = DB::table('users')->where('name', 'John')->value('email');

Щоб отримати один рядок за його`id`значення стовпця, використовуйте`find`метод:

    $user = DB::table('users')->find(3);

<a name="retrieving-a-list-of-column-values"></a>

#### Отримання списку значень стовпців

Якщо ви хочете отримати Колекцію, що містить значення одного стовпця, Ви можете використовувати`pluck`метод. У цьому прикладі ми отримаємо Колекцію назв ролей:

    $titles = DB::table('roles')->pluck('title');

    foreach ($titles as $title) {
        echo $title;
    }

Ви також можете вказати спеціальний стовпець ключа для поверненої колекції:

    $roles = DB::table('roles')->pluck('title', 'name');

    foreach ($roles as $name => $title) {
        echo $title;
    }

<a name="chunking-results"></a>

### Chunking Results

Якщо вам потрібно працювати з тисячами записів бази даних, розгляньте можливість використання`chunk`метод. Цей метод отримує невеликий шматок результатів за раз і подає кожен шматок у файл`Closure`для переробки. Цей метод дуже корисний для письма[Ремісничі команди](/docs/{{version}}/artisan)які обробляють тисячі записів. Наприклад, давайте працюватимемо з цілим`users`таблиця шматками по 100 записів одночасно:

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        foreach ($users as $user) {
            //
        }
    });

Ви можете зупинити обробку подальших фрагментів, повернувшись`false`від`Closure`:

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        // Process the records...

        return false;
    });

Якщо ви оновлюєте записи бази даних під час обробки результатів, ваші результати можуть змінитися несподіваними способами. Отже, при оновленні записів під час Chunking завжди найкраще використовувати`chunkById`замість цього. Цей метод автоматично переносить результати на сторінки з використанням первинного ключа запису:

    DB::table('users')->where('active', false)
        ->chunkById(100, function ($users) {
            foreach ($users as $user) {
                DB::table('users')
                    ->where('id', $user->id)
                    ->update(['active' => true]);
            }
        });

> {note} Під час оновлення або видалення записів усередині зворотного виклику послідовності будь-які зміни первинного ключа або зовнішніх ключів можуть вплинути на запит послідовності. Це може призвести до того, що записи не будуть включені до фрагментованих результатів.

<a name="aggregates"></a>

### Агрегати

Конструктор запитів також надає безліч сукупних методів, таких як`count`,`max`,`min`,`avg`, і`sum`. Ви можете викликати будь-який із цих методів після побудови запиту:

    $users = DB::table('users')->count();

    $price = DB::table('orders')->max('price');

Ви можете поєднувати ці методи з іншими Clausesми:

    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');

<a name="determining-if-records-exist"></a>

#### Визначення, чи існують записи

Замість використання`count`метод, щоб визначити, чи існують записи, які відповідають обмеженням вашого запиту, ви можете використовувати`exists`і`doesntExist`методи:

    return DB::table('orders')->where('finalized', 1)->exists();

    return DB::table('orders')->where('finalized', 1)->doesntExist();

<a name="selects"></a>

## Selects

<a name="specifying-a-select-clause"></a>

#### Вказівка ​​пункту вибору

Можливо, ви не завжди хочете вибрати всі стовпці з таблиці бази даних. Використання`select`метод, ви можете вказати спеціальний`select`пункт для запиту:

    $users = DB::table('users')->select('name', 'email as user_email')->get();

`distinct`метод дозволяє примусити запит повертати різні результати:

    $users = DB::table('users')->distinct()->get();

Якщо у вас вже є екземпляр конструктора запитів, і ви хочете додати стовпець до існуючого Clauses вибору, ви можете використовувати`addSelect`метод:

    $query = DB::table('users')->select('name');

    $users = $query->addSelect('age')->get();

<a name="raw-expressions"></a>

## Сирі вирази

Іноді вам може знадобитися використовувати необроблений вираз у запиті. Щоб створити необроблений вираз, ви можете використовувати`DB::raw`метод:

    $users = DB::table('users')
                         ->select(DB::raw('count(*) as user_count, status'))
                         ->where('status', '<>', 1)
                         ->groupBy('status')
                         ->get();

> {note} Сировинні оператори вводяться у запит як рядки, тому ви повинні бути надзвичайно обережними, щоб не створювати вразливості введення SQL.

<a name="raw-methods"></a>

### Сирі методи

Замість використання`DB::raw`, Ви можете також використовувати наступні методи, щоб вставити необроблений вираз у різні частини вашого запиту.

<a name="selectraw"></a>

#### `selectRaw`

`selectRaw`метод може бути використаний замість`addSelect(DB::raw(...))`. Цей метод приймає необов’язковий масив Bindings як другий аргумент:

    $orders = DB::table('orders')
                    ->selectRaw('price * ? as price_with_tax', [1.0825])
                    ->get();

<a name="whereraw-orwhereraw"></a>

#### `whereRaw / orWhereRaw`

`whereRaw`і`orWhereRaw`методи можуть бути використані для введення сировини`where`у вашому запиті. Ці методи приймають необов’язковий масив Bindings як другий аргумент:

    $orders = DB::table('orders')
                    ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
                    ->get();

<a name="havingraw-orhavingraw"></a>

#### `havingRaw / orHavingRaw`

`havingRaw`і`orHavingRaw`Методи можуть бути використані для встановлення необробленого рядка як значення`having`застереження. Ці методи приймають необов’язковий масив Bindings як другий аргумент:

    $orders = DB::table('orders')
                    ->select('department', DB::raw('SUM(price) as total_sales'))
                    ->groupBy('department')
                    ->havingRaw('SUM(price) > ?', [2500])
                    ->get();

<a name="orderbyraw"></a>

#### `orderByRaw`

`orderByRaw`метод може бути використаний для встановлення необробленого рядка як значення`order by`Clauses:

    $orders = DB::table('orders')
                    ->orderByRaw('updated_at - created_at DESC')
                    ->get();

<a name="groupbyraw"></a>

### `groupByRaw`

`groupByRaw`метод може бути використаний для встановлення необробленого рядка як значення`group by`Clauses:

    $orders = DB::table('orders')
                    ->select('city', 'state')
                    ->groupByRaw('city, state')
                    ->get();

<a name="joins"></a>

## Joins

<a name="inner-join-clause"></a>

#### Застереження про внутрішнє приєднання

Конструктор запитів також може використовуватися для написання операторів об’єднання. Для виконання базового "внутрішнього з'єднання" ви можете використовувати`join`метод на екземплярі конструктора запитів. Перший аргумент, переданий в`join`method - це ім'я таблиці, до якої потрібно приєднатися, тоді як решта аргументів визначають обмеження стовпців для об'єднання. Ви навіть можете приєднатися до декількох таблиць в одному запиті:

    $users = DB::table('users')
                ->join('contacts', 'users.id', '=', 'contacts.user_id')
                ->join('orders', 'users.id', '=', 'orders.user_id')
                ->select('users.*', 'contacts.phone', 'orders.price')
                ->get();

<a name="left-join-right-join-clause"></a>

#### Застереження щодо лівого приєднання / правого приєднання

Якщо ви хочете виконати "ліве з'єднання" або "праве з'єднання" замість "внутрішнє з'єднання", використовуйте`leftJoin`або`rightJoin`методи. Ці методи мають той самий підпис, що і`join`метод:

    $users = DB::table('users')
                ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

    $users = DB::table('users')
                ->rightJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

<a name="cross-join-clause"></a>

#### Застереження про перехресне приєднання

Для виконання "перехресного з'єднання" використовуйте`crossJoin`метод з назвою таблиці, до якої ви хочете приєднатися. Поперечні з’єднання генерують декартовий добуток між першою таблицею та об’єднаною таблицею:

    $sizes = DB::table('sizes')
                ->crossJoin('colors')
                ->get();

<a name="advanced-join-clauses"></a>

#### Розширені пункти приєднання

Ви також можете вказати більш розширені Clauses приєднання. Для початку пройдіть a`Closure`як другий аргумент`join`метод.`Closure`отримає a`JoinClause`об'єкт, який дозволяє вказати обмеження на`join`Clauses:

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
            })
            ->get();

Якщо ви хочете використовувати пункт стилю "де" для своїх об'єднань, ви можете використовувати`where`і`orWhere`методи приєднання. Замість порівняння двох стовпців, ці методи порівняють стовпець зі значенням:

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')
                     ->where('contacts.user_id', '>', 5);
            })
            ->get();

<a name="subquery-joins"></a>

#### Підзапит Joins

Ви можете використовувати`joinSub`,`leftJoinSub`, і`rightJoinSub`методи приєднання запиту до підзапиту. Кожен із цих методів отримує три аргументи: підзапит, його псевдонім таблиці та Закриття, яке визначає відповідні стовпці:

    $latestPosts = DB::table('posts')
                       ->select('user_id', DB::raw('MAX(created_at) as last_post_created_at'))
                       ->where('is_published', true)
                       ->groupBy('user_id');

    $users = DB::table('users')
            ->joinSub($latestPosts, 'latest_posts', function ($join) {
                $join->on('users.id', '=', 'latest_posts.user_id');
            })->get();

<a name="unions"></a>

## Unions

Конструктор запитів також забезпечує швидкий спосіб об’єднати два або більше запитів разом. Наприклад, ви можете створити початковий запит і використовувати`union`метод об'єднати його з більшою кількістю запитів:

    $first = DB::table('users')
                ->whereNull('first_name');

    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();

> {tip} The`unionAll`метод також доступний і має той самий підпис методу, що і`union`.

<a name="where-clauses"></a>

##  Where Clauses

<a name="simple-where-clauses"></a>

#### Прості Clauses

Ви можете використовувати`where`метод на екземплярі конструктора запитів для додавання`where`пункти запиту. Найпростіший заклик до`where`вимагає трьох аргументів. Перший аргумент - це назва стовпця. Другим аргументом є оператор, яким може бути будь-який з підтримуваних операторів бази даних. Нарешті, третім аргументом є значення, яке потрібно оцінити щодо стовпця.

Наприклад, ось запит, який підтверджує, що значення стовпця "голоси" дорівнює 100:

    $users = DB::table('users')->where('votes', '=', 100)->get();

Для зручності, якщо ви хочете перевірити, що стовпець дорівнює заданому значенню, ви можете передати значення безпосередньо як другий аргумент до`where`метод:

    $users = DB::table('users')->where('votes', 100)->get();

Ви можете використовувати різні інші оператори під час написання`where`Clauses:

    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();

Ви також можете передати масив умов в`where`функція:

    $users = DB::table('users')->where([
        ['status', '=', '1'],
        ['subscribed', '<>', '1'],
    ])->get();

<a name="or-statements"></a>

#### Або Заяви

Ви можете зв’язувати де обмеження разом, а також додавати`or`пункти запиту.`orWhere`метод приймає ті самі аргументи, що і`where`метод:

    $users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();

Якщо вам потрібно згрупувати умову "або" в дужках, ви можете передати Закриття як перший аргумент для`orWhere`метод:

    $users = DB::table('users')
                ->where('votes', '>', 100)
                ->orWhere(function($query) {
                    $query->where('name', 'Abigail')
                          ->where('votes', '>', 50);
                })
                ->get();

    // SQL: select * from users where votes > 100 or (name = 'Abigail' and votes > 50)

<a name="additional-where-clauses"></a>

#### Додаткові Clauses Where

**whereBetween / orWhereBetween**

`whereBetween`метод перевіряє, що значення стовпця знаходиться між двома значеннями:

    $users = DB::table('users')
               ->whereBetween('votes', [1, 100])
               ->get();

**whereNotBetween / orWhereNotBetween**

`whereNotBetween`метод перевіряє, що значення стовпця знаходиться поза двома значеннями:

    $users = DB::table('users')
                        ->whereNotBetween('votes', [1, 100])
                        ->get();

**whereIn / whereNotIn / orWhereIn / orWhereNotIn**

`whereIn`метод перевіряє, що значення заданого стовпця міститься в даному масиві:

    $users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();

`whereNotIn`метод перевіряє, що значення даного стовпця становить**ні**міститься в даному масиві:

    $users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();

> {note} Якщо ви додаєте до свого запиту величезний масив цілочисельних прив'язок, файл`whereIntegerInRaw`або`whereIntegerNotInRaw`методи можуть бути використані для значного зменшення використання пам'яті.

**whereNull / whereNotNull / orWhereNull / orWhereNotNull**

`whereNull`метод перевіряє, що значення даного стовпця дорівнює`NULL`:

    $users = DB::table('users')
                        ->whereNull('updated_at')
                        ->get();

`whereNotNull`метод перевіряє, що значення стовпця не є`NULL`:

    $users = DB::table('users')
                        ->whereNotNull('updated_at')
                        ->get();

**whereDate / whereMonth / whereDay / whereYear / whereTime**

`whereDate`метод може бути використаний для порівняння значення стовпця з датою:

    $users = DB::table('users')
                    ->whereDate('created_at', '2016-12-31')
                    ->get();

`whereMonth`метод може бути використаний для порівняння значення стовпця з конкретним місяцем року:

    $users = DB::table('users')
                    ->whereMonth('created_at', '12')
                    ->get();

`whereDay`метод може бути використаний для порівняння значення стовпця з конкретним днем ​​місяця:

    $users = DB::table('users')
                    ->whereDay('created_at', '31')
                    ->get();

`whereYear`метод може бути використаний для порівняння значення стовпця з конкретним роком:

    $users = DB::table('users')
                    ->whereYear('created_at', '2016')
                    ->get();

`whereTime`метод може бути використаний для порівняння значення стовпця з певним часом:

    $users = DB::table('users')
                    ->whereTime('created_at', '=', '11:20:45')
                    ->get();

**whereColumn / orWhereColumn**

`whereColumn`метод може бути використаний для перевірки того, що два стовпці рівні:

    $users = DB::table('users')
                    ->whereColumn('first_name', 'last_name')
                    ->get();

Ви також можете передати оператору порівняння метод:

    $users = DB::table('users')
                    ->whereColumn('updated_at', '>', 'created_at')
                    ->get();

`whereColumn`Метод може також передавати масив з декількох умов. Ці умови будуть об'єднані за допомогою`and`оператор:

    $users = DB::table('users')
                    ->whereColumn([
                        ['first_name', '=', 'last_name'],
                        ['updated_at', '>', 'created_at'],
                    ])->get();

<a name="parameter-grouping"></a>

### Групування параметрів

Іноді вам може знадобитися створити більш просунуті Clauses, такі як Clauses "де існує" або вкладені групування параметрів. Конструктор запитів Laravel може впоратися і з ними. Для початку давайте розглянемо приклад групування обмежень у дужках:

    $users = DB::table('users')
               ->where('name', '=', 'John')
               ->where(function ($query) {
                   $query->where('votes', '>', 100)
                         ->orWhere('title', '=', 'Admin');
               })
               ->get();

Як бачите, проходження а`Closure`в`where`метод вказує конструктору запитів розпочати групу обмежень.`Closure`отримає екземпляр конструктора запитів, який ви можете використовувати для встановлення обмежень, які повинні міститися в групі дужок. У наведеному вище прикладі буде створено такий SQL:

    select * from users where name = 'John' and (votes > 100 or title = 'Admin')

> {tip} Вам завжди слід групуватися`orWhere`дзвінки, щоб уникнути несподіваної поведінки, коли застосовуються глобальні Scopes.

<a name="where-exists-clauses"></a>

###  Where існує Clauses

`whereExists`метод дозволяє писати`where exists`Clauses SQL.`whereExists`метод приймає a`Closure`аргумент, який отримає екземпляр конструктора запитів, що дозволяє визначити запит, який повинен бути розміщений всередині Clauses "існує":

    $users = DB::table('users')
               ->whereExists(function ($query) {
                   $query->select(DB::raw(1))
                         ->from('orders')
                         ->whereRaw('orders.user_id = users.id');
               })
               ->get();

Запит вище дасть такий SQL:

    select * from users
    where exists (
        select 1 from orders where orders.user_id = users.id
    )

<a name="subquery-where-clauses"></a>

### Підзапит де Clauses

Іноді вам може знадобитися побудувати Clauses where, яке порівнює результати підзапиту із заданим значенням. Ви можете досягти цього, передавши Закриття та значення в`where`метод. Наприклад, наступний запит отримає всіх користувачів, які нещодавно отримали "членство" певного типу;

    use App\Models\User;

    $users = User::where(function ($query) {
        $query->select('type')
            ->from('membership')
            ->whereColumn('user_id', 'users.id')
            ->orderByDesc('start_date')
            ->limit(1);
    }, 'Pro')->get();

<a name="json-where-clauses"></a>

### JSON  Where Clauses

Laravel також підтримує запити типів стовпців JSON у базах даних, що забезпечують підтримку типів стовпців JSON. В даний час це включає MySQL 5.7, PostgreSQL, SQL Server 2016 та SQLite 3.9.0 (з[Розширення JSON1](https://www.sqlite.org/json1.html)). Для запиту стовпця JSON використовуйте`->`оператор:

    $users = DB::table('users')
                    ->where('options->language', 'en')
                    ->get();

    $users = DB::table('users')
                    ->where('preferences->dining->meal', 'salad')
                    ->get();

Ви можете використовувати`whereJsonContains`для запиту масивів JSON (не підтримується на SQLite):

    $users = DB::table('users')
                    ->whereJsonContains('options->languages', 'en')
                    ->get();

Підтримка MySQL та PostgreSQL`whereJsonContains`з кількома значеннями:

    $users = DB::table('users')
                    ->whereJsonContains('options->languages', ['en', 'de'])
                    ->get();

Ви можете використовувати`whereJsonLength`для запиту масивів JSON за їх довжиною:

    $users = DB::table('users')
                    ->whereJsonLength('options->languages', 0)
                    ->get();

    $users = DB::table('users')
                    ->whereJsonLength('options->languages', '>', 1)
                    ->get();

<a name="ordering-grouping-limit-and-offset"></a>

## Впорядкування, групування, обмеження та зміщення

<a name="orderby"></a>

#### Сортувати за

`orderBy`метод дозволяє сортувати результат запиту за заданим стовпцем. Перший аргумент`orderBy`метод повинен бути стовпцем, за яким ви хочете сортувати, тоді як другий аргумент керує напрямком сортування і може бути будь-яким`asc`або`desc`:

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();

Якщо вам потрібно відсортувати за кількома стовпцями, ви можете викликати`orderBy`стільки разів, скільки потрібно:

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->orderBy('email', 'asc')
                    ->get();

<a name="latest-oldest"></a>

#### останні / найстаріші

`latest`і`oldest`методи дозволяють легко впорядковувати результати за датою. За замовчуванням результат буде впорядковано`created_at`стовпець. Або ви можете передати назву стовпця, за яким ви хочете відсортувати:

    $user = DB::table('users')
                    ->latest()
                    ->first();

<a name="inrandomorder"></a>

#### Інландо р де р

`inRandomOrder`метод може бути використаний для випадкового сортування результатів запиту. Наприклад, ви можете використовувати цей метод для отримання випадкового користувача:

    $randomUser = DB::table('users')
                    ->inRandomOrder()
                    ->first();

<a name="reorder"></a>

#### змінити порядок

`reorder`метод дозволяє видалити всі існуючі замовлення та додатково застосувати нове замовлення. Наприклад, ви можете видалити всі існуючі замовлення:

    $query = DB::table('users')->orderBy('name');

    $unorderedUsers = $query->reorder()->get();

Щоб видалити всі існуючі замовлення та застосувати нове замовлення, вкажіть стовпець та напрямок як аргументи методу:

    $query = DB::table('users')->orderBy('name');

    $usersOrderedByEmail = $query->reorder('email', 'desc')->get();

<a name="groupby-having"></a>

#### група За / маючи

`groupBy`і`having`Методи можуть бути використані для групування результатів запиту.`having`підпис методу подібний до підпису`where`метод:

    $users = DB::table('users')
                    ->groupBy('account_id')
                    ->having('account_id', '>', 100)
                    ->get();

Ви можете передавати кілька аргументів до`groupBy`метод групування за кількома стовпцями:

    $users = DB::table('users')
                    ->groupBy('first_name', 'status')
                    ->having('account_id', '>', 100)
                    ->get();

Для більш просунутих`having`заяви, див[`havingRaw`](#raw-methods)метод.

<a name="skip-take"></a>

#### пропустити / взяти

Щоб обмежити кількість результатів, повернутих із запиту, або пропустити задану кількість результатів у запиті, ви можете використовувати`skip`і`take`методи:

    $users = DB::table('users')->skip(10)->take(5)->get();

Ви також можете використовувати`limit`і`offset`методи:

    $users = DB::table('users')
                    ->offset(10)
                    ->limit(5)
                    ->get();

<a name="conditional-clauses"></a>

## Умовні Clauses

Іноді вам може знадобитися, щоб Clauses застосовувались до запиту лише тоді, коли щось інше відповідає дійсності. Наприклад, ви можете застосувати лише файл`where`оператор, якщо задане вхідне значення присутнє на вхідному запиті. Ви можете досягти цього за допомогою`when`метод:

    $role = $request->input('role');

    $users = DB::table('users')
                    ->when($role, function ($query, $role) {
                        return $query->where('role_id', $role);
                    })
                    ->get();

`when`метод виконує дане Закриття лише тоді, коли є першим параметром`true`. Якщо перший параметр -`false`, Закриття не буде виконано.

Ви можете передати інше Закриття як третій параметр в`when`метод. Це Закриття буде виконано, якщо перший параметр оцінюється як`false`. Щоб проілюструвати, як можна використовувати цю функцію, ми будемо використовувати її для налаштування сортування запиту за замовчуванням:

    $sortBy = null;

    $users = DB::table('users')
                    ->when($sortBy, function ($query, $sortBy) {
                        return $query->orderBy($sortBy);
                    }, function ($query) {
                        return $query->orderBy('name');
                    })
                    ->get();

<a name="inserts"></a>

## Вставки

Конструктор запитів також надає файл`insert`метод вставки записів у таблицю бази даних.`insert`метод приймає масив імен та значень стовпців:

    DB::table('users')->insert(
        ['email' => 'john@example.com', 'votes' => 0]
    );

Ви навіть можете вставити кілька записів у таблицю за один виклик`insert`передаючи масив масивів. Кожен масив представляє рядок, який потрібно вставити в таблицю:

    DB::table('users')->insert([
        ['email' => 'taylor@example.com', 'votes' => 0],
        ['email' => 'dayle@example.com', 'votes' => 0],
    ]);

`insertOrIgnore`метод ігноруватиме повторювані помилки записів під час вставки записів до бази даних:

    DB::table('users')->insertOrIgnore([
        ['id' => 1, 'email' => 'taylor@example.com'],
        ['id' => 2, 'email' => 'dayle@example.com'],
    ]);

`upsert`метод вставить неіснуючі рядки та оновить вже наявні рядки новими значеннями. Перший аргумент методу складається зі значень, які потрібно вставити або оновити, тоді як другий аргумент містить перелік стовпців (колонок), які однозначно ідентифікують записи в асоційованій таблиці. Третім і остаточним аргументом методу є масив стовпців, який слід оновити, якщо відповідний запис уже існує в базі даних:

    DB::table('flights')->upsert([
        ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
        ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
    ], ['departure', 'destination'], ['price']);

> {note} Для всіх баз даних, крім SQL Server, потрібні стовпці у другому аргументі`upsert`метод, щоб мати "первинний" або "унікальний" індекс.

<a name="auto-incrementing-ids"></a>

#### Автоматичне збільшення ідентифікаторів

Якщо в таблиці є ідентифікатор, що автоматично збільшується, використовуйте`insertGetId`метод, щоб вставити запис, а потім отримати ідентифікатор:

    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );

> {note} При використанні PostgreSQL файл`insertGetId`метод очікує, що буде вказано стовпець із автоматичним збільшенням`id`. Якщо ви хочете отримати ідентифікатор з іншої "послідовності", ви можете передати ім'я стовпця як другий параметр`insertGetId`метод.

<a name="updates"></a>

## Оновлення

На додаток до вставки записів у базу даних, конструктор запитів може також оновити існуючі записи за допомогою`update`метод.`update`метод, як`insert`метод, приймає масив пар стовпців і значень, що містять стовпці, які слід оновити. Ви можете обмежити`update`запит за допомогою`where`Clauses:

    $affected = DB::table('users')
                  ->where('id', 1)
                  ->update(['votes' => 1]);

<a name="update-or-insert"></a>

#### Оновити або вставити

Іноді вам може знадобитися оновити існуючий запис у базі даних або створити його, якщо відповідний запис не існує. У цьому випадку сценарій`updateOrInsert`може бути використаний метод.`updateOrInsert`Метод приймає два аргументи: масив умов, за якими можна знайти запис, і масив пар стовпців та значень, що містять стовпці, які слід оновити.

`updateOrInsert`Спочатку спробує знайти відповідний запис бази даних, використовуючи пари стовпця та значення першого аргументу. Якщо запис існує, він буде оновлений зі значеннями у другому аргументі. Якщо запис знайти не вдається, буде вставлений новий запис із об’єднаними атрибутами обох аргументів:

    DB::table('users')
        ->updateOrInsert(
            ['email' => 'john@example.com', 'name' => 'John'],
            ['votes' => '2']
        );

<a name="updating-json-columns"></a>

### Оновлення стовпців JSON

Під час оновлення стовпця JSON вам слід використовувати`->`синтаксис для доступу до відповідного ключа в об’єкті JSON. Ця операція підтримується на MySQL 5.7+ та PostgreSQL 9.5+:

    $affected = DB::table('users')
                  ->where('id', 1)
                  ->update(['options->enabled' => true]);

<a name="increment-and-decrement"></a>

### Збільшення та зменшення

Конструктор запитів також надає зручні методи збільшення або зменшення значення даного стовпця. Це ярлик, що забезпечує більш виразний та стислий інтерфейс порівняно з ручним написанням`update`заява.

Обидва ці методи приймають принаймні один аргумент: стовпець для модифікації. За бажанням може бути переданий другий аргумент для контролю суми, на яку слід збільшувати або зменшувати стовпець:

    DB::table('users')->increment('votes');

    DB::table('users')->increment('votes', 5);

    DB::table('users')->decrement('votes');

    DB::table('users')->decrement('votes', 5);

Ви також можете вказати додаткові стовпці для оновлення під час операції:

    DB::table('users')->increment('votes', 1, ['name' => 'John']);

> {note} Події моделі не запускаються під час використання`increment`і`decrement`методи.

<a name="deletes"></a>

## Deletes

Конструктор запитів також може використовуватися для видалення записів з таблиці через`delete`метод. Ви можете обмежити`delete`твердження шляхом додавання`where`перед викликом`delete`метод:

    DB::table('users')->delete();

    DB::table('users')->where('votes', '>', 100)->delete();

Якщо ви хочете скоротити всю таблицю, яка видалить всі рядки та скине ідентифікатор автоматичного збільшення до нуля, ви можете`truncate`метод:

    DB::table('users')->truncate();

<a name="table-truncation-and-postgresql"></a>

#### Зрізання таблиці та PostgreSQL

При скороченні бази даних PostgreSQL файл`CASCADE`поведінку буде застосовано. Це означає, що всі записи, пов'язані із зовнішніми ключами, також будуть видалені в інших таблицях.

<a name="pessimistic-locking"></a>

## Песимістичний Locking

Конструктор запитів також включає кілька функцій, які допоможуть вам зробити "песимістичне блокування" на вашому`select`заяви. Щоб запустити оператор із "спільним блоForge", ви можете використовувати`sharedLock`метод за запитом. Спільний Locking не дозволяє модифікувати вибрані рядки, доки ваша транзакція не буде здійснена:

    DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

Ви також можете використовувати`lockForUpdate`метод. Блокування "для оновлення" запобігає зміні рядків або їх вибору за допомогою іншого спільного блокування:

    DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();

<a name="debugging"></a>

## Debug

Ви можете використовувати`dd`або`dump`методів під час побудови запиту для скидання прив'язок запитів та SQL.`dd`метод відобразить інформацію про Debug, а потім припинить виконання запиту.`dump`буде відображати інформацію про Debug, але дозволить запиту продовжувати виконуватись:

    DB::table('users')->where('votes', '>', 100)->dd();

    DB::table('users')->where('votes', '>', 100)->dump();
