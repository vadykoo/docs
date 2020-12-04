# Помічники

-   [Вступ](#introduction)
-   [Доступні методи](#available-methods)

<a name="introduction"></a>

## Вступ

Laravel включає різноманітні глобальні «допоміжні» функції PHP. Багато з цих функцій використовуються самою структурою; однак ви можете вільно використовувати їх у власних програмах, якщо вважаєте, що це зручно.

<a name="available-methods"></a>

## Доступні методи

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<a name="arrays-and-objects-method-list"></a>

### Масиви та об'єкти

<div class="collection-method-list" markdown="1">

[Arr :: доступний](#method-array-accessible)[Arr :: додати](#method-array-add)[Arr :: крах](#method-array-collapse)[Arr :: crossJoin](#method-array-crossjoin)[Arr :: розділити](#method-array-divide)[Arr :: точка](#method-array-dot)[Arr :: за винятком](#method-array-except)[Arr :: існує](#method-array-exists)[Arr :: перший](#method-array-first)[Arr :: сплющити](#method-array-flatten)[Arr :: забути](#method-array-forget)[Arr :: get](#method-array-get)[Arr :: has](#method-array-has)[Arr :: hasAny](#method-array-hasany)[Arr :: isAssoc](#method-array-isassoc)[Arr :: останній](#method-array-last)[Arr :: only](#method-array-only)[Arr :: зривати](#method-array-pluck)[Arr :: prepend](#method-array-prepend)[Arr :: pull](#method-array-pull)[Arr :: запит](#method-array-query)[Arr :: випадковий](#method-array-random)[Arr :: set](#method-array-set)[Arr :: перетасувати](#method-array-shuffle)[Arr :: sort](#method-array-sort)[Arr :: sortRecursive](#method-array-sort-recursive)[Arr :: де](#method-array-where)[Arr :: wrap](#method-array-wrap)[заповнення\_даних](#method-data-fill)[data_get](#method-data-get)[набір даних](#method-data-set)[голова](#method-head)[останній](#method-last)

</div>

<a name="paths-method-list"></a>

### Шляхи

<div class="collection-method-list" markdown="1">

[app_path](#method-app-path)[base_path](#method-base-path)[конфігураційний шлях](#method-config-path)[шлях до бази даних](#method-database-path)[Mix](#method-mix)[public_path](#method-public-path)[шлях до ресурсу](#method-resource-path)[шлях\_зберігання](#method-storage-path)

</div>

<a name="strings-method-list"></a>

### Струни

<div class="collection-method-list" markdown="1">

[\_\_](#method-__)[ім'я\_бази класу](#method-class-basename)[є](#method-e)[preg_replace масив](#method-preg-replace-array)[Str :: after](#method-str-after)[Str :: afterLast](#method-str-after-last)[Вулиця :: ascii](#method-str-ascii)[Str :: до](#method-str-before)[Str :: beforeLast](#method-str-before-last)[Str :: між](#method-str-between)[Str :: верблюд](#method-camel-case)[Str :: містить](#method-str-contains)[Str :: containsAll](#method-str-contains-all)[Str :: ENDWith](#method-ends-with)[Str :: закінчити](#method-str-finish)[Str :: is](#method-str-is)[Вулиця :: isAscii](#method-str-is-ascii)[Вулиця :: isUuid](#method-str-is-uuid)[Str :: шашлик](#method-kebab-case)[Str :: length](#method-str-length)[Вулиця :: ліміт](#method-str-limit)[Str :: нижній](#method-str-lower)[Str :: orderUuid](#method-str-ordered-uuid)[Str :: padBoth](#method-str-padboth)[Вулиця :: padLeft](#method-str-padleft)[Str :: padRight](#method-str-padright)[Str :: множина](#method-str-plural)[Str :: random](#method-str-random)[Str :: replaceArray](#method-str-replace-array)[Str :: replaceFirst](#method-str-replace-first)[Рядок :: replaceLast](#method-str-replace-last)[Вулиця :: однина](#method-str-singular)[Str :: slug](#method-str-slug)[Str :: змія](#method-snake-case)[Str :: start](#method-str-start)[Str :: startWith](#method-starts-with)[Str :: studly](#method-studly-case)[Str :: substr](#method-str-substr)[Str :: title](#method-title-case)[Вулиця :: ucfirst](#method-str-ucfirst)[Str :: верхній](#method-str-upper)[Str :: uuid](#method-str-uuid)[Str :: слова](#method-str-words)[переклад](#method-trans)[trans_choice](#method-trans-choice)

</div>

<a name="fluent-strings-method-list"></a>

### Свободні струнні

<div class="collection-method-list" markdown="1">

[після](#method-fluent-str-after)[після](#method-fluent-str-after-last)[додати](#method-fluent-str-append)[ascii](#method-fluent-str-ascii)[базове ім'я](#method-fluent-str-basename)[раніше](#method-fluent-str-before)[beforeLast](#method-fluent-str-before-last)[верблюд](#method-fluent-str-camel)[містить](#method-fluent-str-contains)[містить усі](#method-fluent-str-contains-all)[прізвище](#method-fluent-str-dirname)[закінчуєтьсяз](#method-fluent-str-ends-with)[точно](#method-fluent-str-exactly)[вибухнути](#method-fluent-str-explode)[закінчити](#method-fluent-str-finish)[є](#method-fluent-str-is)[isAscii](#method-fluent-str-is-ascii)[пусто](#method-fluent-str-is-empty)[isNotEmpty](#method-fluent-str-is-not-empty)[шашлик](#method-fluent-str-kebab)[довжина](#method-fluent-str-length)[обмеження](#method-fluent-str-limit)[нижній](#method-fluent-str-lower)[LTrim](#method-fluent-str-ltrim)[матч](#method-fluent-str-match)[matchAll](#method-fluent-str-match-all)[padBoth](#method-fluent-str-padboth)[padLeft](#method-fluent-str-padleft)[padRight](#method-fluent-str-padright)[множини](#method-fluent-str-plural)[попередньо](#method-fluent-str-prepend)[замінити](#method-fluent-str-replace)[replaceArray](#method-fluent-str-replace-array)[replaceFirst](#method-fluent-str-replace-first)[replaceLast](#method-fluent-str-replace-last)[replaceMatches](#method-fluent-str-replace-matches)[rtrim](#method-fluent-str-rtrim)[однина](#method-fluent-str-singular)[слизень](#method-fluent-str-slug)[змія](#method-fluent-str-snake)[розколоти](#method-fluent-str-split)[почати](#method-fluent-str-start)[починається з](#method-fluent-str-starts-with)[студійно](#method-fluent-str-studly)[підстр](#method-fluent-str-substr)[заголовок](#method-fluent-str-title)[обрізати](#method-fluent-str-trim)[ucfirst](#method-fluent-str-ucfirst)[верхній](#method-fluent-str-upper)[коли](#method-fluent-str-when)[whenEmpty](#method-fluent-str-when-empty)[слова](#method-fluent-str-words)

</div>

<a name="urls-method-list"></a>

### URL-адреси

<div class="collection-method-list" markdown="1">

[дії](#method-action)[активу](#method-asset)[маршруту](#method-route)[secure_asset](#method-secure-asset)[secure_url](#method-secure-url)[URL-адреса](#method-url)

</div>

<a name="miscellaneous-method-list"></a>

### Різне

<div class="collection-method-list" markdown="1">

[аборт](#method-abort)[abort_if](#method-abort-if)[abort_unless](#method-abort-unless)[додаток](#method-app)[авт](#method-auth)[назад](#method-back)[bcrypt](#method-bcrypt)[порожній](#method-blank)[мовлення](#method-broadcast)[кеш](#method-cache)[class_uses_recursive](#method-class-uses-recursive)[збирати](#method-collect)[конфігурація](#method-config)[Cookies](#method-cookie)[csrf_field](#method-csrf-field)[csrf_token](#method-csrf-token)[dd](#method-dd)[відправлення](#method-dispatch)[dispatch_now](#method-dispatch-now)[смітник](#method-dump)[приблизно](#method-env)[подія](#method-event)[заповнені](#method-filled)[інформація](#method-info)[журнали](#method-logger)[метод\_поле](#method-method-field)[зараз](#method-now)[старий](#method-old)[за бажанням](#method-optional)[політика](#method-policy)[переспрямування](#method-redirect)[доповідь](#method-report)[запит](#method-request)[порятунку](#method-rescue)[вирішити](#method-resolve)[відповідь](#method-response)[повторити спробу](#method-retry)[сесія](#method-session)[натисніть](#method-tap)[throw_if](#method-throw-if)[кинути\_нещо](#method-throw-unless)[сьогодні](#method-today)[trait_uses_recursive](#method-trait-uses-recursive)[перетворювати](#method-transform)[валідатор](#method-validator)[значення](#method-value)[вид](#method-view)[з](#method-with)

</div>

<a name="method-listing"></a>

## Перелік методів

<style>
    #collection-method code {
        font-size: 14px;
    }

    #collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="arrays"></a>

## Масиви та об'єкти

<a name="method-array-accessible"></a>

#### `Arr::accessible()`{# collection-method .first-collection-method}

`Arr::accessible`метод перевіряє, чи є вказане значення доступним для масиву:

    use Illuminate\Support\Arr;
    use Illuminate\Support\Collection;

    $isAccessible = Arr::accessible(['a' => 1, 'b' => 2]);

    // true

    $isAccessible = Arr::accessible(new Collection);

    // true

    $isAccessible = Arr::accessible('abc');

    // false

    $isAccessible = Arr::accessible(new stdClass);

    // false

<a name="method-array-add"></a>

#### `Arr::add()`{# collection-method}

`Arr::add`метод додає дану пару ключ / значення до масиву, якщо даний ключ ще не існує в масиві або для нього встановлено значення`null`:

    use Illuminate\Support\Arr;

    $array = Arr::add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

    $array = Arr::add(['name' => 'Desk', 'price' => null], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-collapse"></a>

#### `Arr::collapse()`{# collection-method}

`Arr::collapse`метод згортає масив масивів в один масив:

    use Illuminate\Support\Arr;

    $array = Arr::collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-array-crossjoin"></a>

#### `Arr::crossJoin()`{# collection-method}

`Arr::crossJoin`метод cross приєднує дані масиви, повертаючи декартовий добуток з усіма можливими перестановками:

    use Illuminate\Support\Arr;

    $matrix = Arr::crossJoin([1, 2], ['a', 'b']);

    /*
        [
            [1, 'a'],
            [1, 'b'],
            [2, 'a'],
            [2, 'b'],
        ]
    */

    $matrix = Arr::crossJoin([1, 2], ['a', 'b'], ['I', 'II']);

    /*
        [
            [1, 'a', 'I'],
            [1, 'a', 'II'],
            [1, 'b', 'I'],
            [1, 'b', 'II'],
            [2, 'a', 'I'],
            [2, 'a', 'II'],
            [2, 'b', 'I'],
            [2, 'b', 'II'],
        ]
    */

<a name="method-array-divide"></a>

#### `Arr::divide()`{# collection-method}

`Arr::divide`метод повертає два масиви, один з яких містить ключі, а інший містить значення даного масиву:

    use Illuminate\Support\Arr;

    [$keys, $values] = Arr::divide(['name' => 'Desk']);

    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>

#### `Arr::dot()`{# collection-method}

`Arr::dot`метод вирівнює багатовимірний масив в однорівневий масив, який використовує Tagging "точка" для Tagging глибини:

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    $flattened = Arr::dot($array);

    // ['products.desk.price' => 100]

<a name="method-array-except"></a>

#### `Arr::except()`{# collection-method}

`Arr::except`метод видаляє дані маси ключ / значення з масиву:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100];

    $filtered = Arr::except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-exists"></a>

#### `Arr::exists()`{# collection-method}

`Arr::exists`метод перевіряє наявність вказаного ключа у наданому масиві:

    use Illuminate\Support\Arr;

    $array = ['name' => 'John Doe', 'age' => 17];

    $exists = Arr::exists($array, 'name');

    // true

    $exists = Arr::exists($array, 'salary');

    // false

<a name="method-array-first"></a>

#### `Arr::first()`{# collection-method}

`Arr::first`метод повертає перший елемент масиву, що проходить заданий тест на істинність:

    use Illuminate\Support\Arr;

    $array = [100, 200, 300];

    $first = Arr::first($array, function ($value, $key) {
        return $value >= 150;
    });

    // 200

Значення за замовчуванням також може бути передане методу як третій параметр. Це значення буде повернуто, якщо жодне значення не пройде перевірку істинності:

    use Illuminate\Support\Arr;

    $first = Arr::first($array, $callback, $default);

<a name="method-array-flatten"></a>

#### `Arr::flatten()`{# collection-method}

`Arr::flatten`метод вирівнює багатовимірний масив в однорівневий масив:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $flattened = Arr::flatten($array);

    // ['Joe', 'PHP', 'Ruby']

<a name="method-array-forget"></a>

#### `Arr::forget()`{# collection-method}

`Arr::forget`метод видаляє дану пару ключ / значення з глибоко вкладеного масиву, використовуючи Tagging "точка":

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    Arr::forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>

#### `Arr::get()`{# collection-method}

`Arr::get`метод отримує значення з глибоко вкладеного масиву, використовуючи Tagging "точка":

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    $price = Arr::get($array, 'products.desk.price');

    // 100

`Arr::get`метод також приймає значення за замовчуванням, яке буде повернуто, якщо конкретний ключ не знайдено:

    use Illuminate\Support\Arr;

    $discount = Arr::get($array, 'products.desk.discount', 0);

    // 0

<a name="method-array-has"></a>

#### `Arr::has()`{# collection-method}

`Arr::has`метод перевіряє, чи існує певний елемент або елементи в масиві, використовуючи Tagging "крапка":

    use Illuminate\Support\Arr;

    $array = ['product' => ['name' => 'Desk', 'price' => 100]];

    $contains = Arr::has($array, 'product.name');

    // true

    $contains = Arr::has($array, ['product.price', 'product.discount']);

    // false

<a name="method-array-hasany"></a>

#### `Arr::hasAny()`{# collection-method}

`Arr::hasAny`метод перевіряє, чи існує будь-який елемент у даному наборі в масиві, використовуючи Tagging "крапка":

    use Illuminate\Support\Arr;

    $array = ['product' => ['name' => 'Desk', 'price' => 100]];

    $contains = Arr::hasAny($array, 'product.name');

    // true

    $contains = Arr::hasAny($array, ['product.name', 'product.discount']);

    // true

    $contains = Arr::hasAny($array, ['category', 'product.discount']);

    // false

<a name="method-array-isassoc"></a>

#### `Arr::isAssoc()`{# collection-method}

`Arr::isAssoc`повертається`true`якщо даний масив є асоціативним масивом. Масив вважається "асоціативним", якщо він не має послідовних числових клавіш, що починаються з нуля:

    use Illuminate\Support\Arr;

    $isAssoc = Arr::isAssoc(['product' => ['name' => 'Desk', 'price' => 100]]);

    // true

    $isAssoc = Arr::isAssoc([1, 2, 3]);

    // false

<a name="method-array-last"></a>

#### `Arr::last()`{# collection-method}

`Arr::last`метод повертає останній елемент масиву, що проходить заданий тест на істинність:

    use Illuminate\Support\Arr;

    $array = [100, 200, 300, 110];

    $last = Arr::last($array, function ($value, $key) {
        return $value >= 150;
    });

    // 300

Значення за замовчуванням може бути передане як третій аргумент методу. Це значення буде повернуто, якщо жодне значення не пройде перевірку істинності:

    use Illuminate\Support\Arr;

    $last = Arr::last($array, $callback, $default);

<a name="method-array-only"></a>

#### `Arr::only()`{# collection-method}

`Arr::only`метод повертає із зазначеного масиву лише вказані пари ключ / значення:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $slice = Arr::only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>

#### `Arr::pluck()`{# collection-method}

`Arr::pluck`метод отримує всі значення для даного ключа з масиву:

    use Illuminate\Support\Arr;

    $array = [
        ['developer' => ['id' => 1, 'name' => 'Taylor']],
        ['developer' => ['id' => 2, 'name' => 'Abigail']],
    ];

    $names = Arr::pluck($array, 'developer.name');

    // ['Taylor', 'Abigail']

Ви також можете вказати, як ви хочете ввести отриманий список:

    use Illuminate\Support\Arr;

    $names = Arr::pluck($array, 'developer.name', 'developer.id');

    // [1 => 'Taylor', 2 => 'Abigail']

<a name="method-array-prepend"></a>

#### `Arr::prepend()`{# collection-method}

`Arr::prepend`метод висуне елемент на початок масиву:

    use Illuminate\Support\Arr;

    $array = ['one', 'two', 'three', 'four'];

    $array = Arr::prepend($array, 'zero');

    // ['zero', 'one', 'two', 'three', 'four']

Якщо потрібно, ви можете вказати ключ, який слід використовувати для значення:

    use Illuminate\Support\Arr;

    $array = ['price' => 100];

    $array = Arr::prepend($array, 'Desk', 'name');

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pull"></a>

#### `Arr::pull()`{# collection-method}

`Arr::pull`метод повертає і видаляє пару масив / значення з масиву:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100];

    $name = Arr::pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

Значення за замовчуванням може бути передане як третій аргумент методу. Це значення буде повернуто, якщо ключ не існує:

    use Illuminate\Support\Arr;

    $value = Arr::pull($array, $key, $default);

<a name="method-array-query"></a>

#### `Arr::query()`{# collection-method}

`Arr::query`метод перетворює масив у рядок запиту:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Taylor', 'order' => ['column' => 'created_at', 'direction' => 'desc']];

    Arr::query($array);

    // name=Taylor&order[column]=created_at&order[direction]=desc

<a name="method-array-random"></a>

#### `Arr::random()`{# collection-method}

`Arr::random`метод повертає випадкове значення з масиву:

    use Illuminate\Support\Arr;

    $array = [1, 2, 3, 4, 5];

    $random = Arr::random($array);

    // 4 - (retrieved randomly)

Ви також можете вказати кількість елементів для повернення як необов’язковий другий аргумент. Зверніть увагу, що надання цього аргументу поверне масив, навіть якщо бажаний лише один елемент:

    use Illuminate\Support\Arr;

    $items = Arr::random($array, 2);

    // [2, 5] - (retrieved randomly)

<a name="method-array-set"></a>

#### `Arr::set()`{# collection-method}

`Arr::set`метод встановлює значення в глибоко вкладеному масиві, використовуючи Tagging "точка":

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    Arr::set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-shuffle"></a>

#### `Arr::shuffle()`{# collection-method}

`Arr::shuffle`метод випадковим чином перемішує елементи в масиві:

    use Illuminate\Support\Arr;

    $array = Arr::shuffle([1, 2, 3, 4, 5]);

    // [3, 2, 5, 1, 4] - (generated randomly)

<a name="method-array-sort"></a>

#### `Arr::sort()`{# collection-method}

`Arr::sort`метод сортує масив за його значеннями:

    use Illuminate\Support\Arr;

    $array = ['Desk', 'Table', 'Chair'];

    $sorted = Arr::sort($array);

    // ['Chair', 'Desk', 'Table']

Ви також можете сортувати масив за результатами даного Закриття:

    use Illuminate\Support\Arr;

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Table'],
        ['name' => 'Chair'],
    ];

    $sorted = array_values(Arr::sort($array, function ($value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Chair'],
            ['name' => 'Desk'],
            ['name' => 'Table'],
        ]
    */

<a name="method-array-sort-recursive"></a>

#### `Arr::sortRecursive()`{# collection-method}

`Arr::sortRecursive`метод рекурсивно сортує масив за допомогою`sort`функція для числових під = масивів та`ksort`для асоціативних підмасивів:

    use Illuminate\Support\Arr;

    $array = [
        ['Roman', 'Taylor', 'Li'],
        ['PHP', 'Ruby', 'JavaScript'],
        ['one' => 1, 'two' => 2, 'three' => 3],
    ];

    $sorted = Arr::sortRecursive($array);

    /*
        [
            ['JavaScript', 'PHP', 'Ruby'],
            ['one' => 1, 'three' => 3, 'two' => 2],
            ['Li', 'Roman', 'Taylor'],
        ]
    */

<a name="method-array-where"></a>

#### `Arr::where()`{# collection-method}

`Arr::where`метод фільтрує масив, використовуючи вказане Закриття:

    use Illuminate\Support\Arr;

    $array = [100, '200', 300, '400', 500];

    $filtered = Arr::where($array, function ($value, $key) {
        return is_string($value);
    });

    // [1 => '200', 3 => '400']

<a name="method-array-wrap"></a>

#### `Arr::wrap()`{# collection-method}

`Arr::wrap`метод обертає задане значення в масив. Якщо вказане значення вже є масивом, воно не буде змінено:

    use Illuminate\Support\Arr;

    $string = 'Laravel';

    $array = Arr::wrap($string);

    // ['Laravel']

Якщо вказане значення є нульовим, буде повернено порожній масив:

    use Illuminate\Support\Arr;

    $nothing = null;

    $array = Arr::wrap($nothing);

    // []

<a name="method-data-fill"></a>

#### `data_fill()`{# collection-method}

`data_fill`функція встановлює відсутні значення у вкладеному масиві або об'єкті, використовуючи Tagging "точка":

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_fill($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 100]]]

    data_fill($data, 'products.desk.discount', 10);

    // ['products' => ['desk' => ['price' => 100, 'discount' => 10]]]

Ця функція також приймає зірочки як символи підстановки і відповідно заповнює ціль:

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2'],
        ],
    ];

    data_fill($data, 'products.*.price', 200);

    /*
        [
            'products' => [
                ['name' => 'Desk 1', 'price' => 100],
                ['name' => 'Desk 2', 'price' => 200],
            ],
        ]
    */

<a name="method-data-get"></a>

#### `data_get()`{# collection-method}

`data_get`функція отримує значення з вкладеного масиву або об'єкта за допомогою Tagging "точка":

    $data = ['products' => ['desk' => ['price' => 100]]];

    $price = data_get($data, 'products.desk.price');

    // 100

`data_get`функція також приймає значення за замовчуванням, яке буде повернуто, якщо вказаний ключ не знайдено:

    $discount = data_get($data, 'products.desk.discount', 0);

    // 0

Функція також приймає символи підстановки, використовуючи зірочки, які можуть націлюватися на будь-який ключ масиву або об’єкта:

    $data = [
        'product-one' => ['name' => 'Desk 1', 'price' => 100],
        'product-two' => ['name' => 'Desk 2', 'price' => 150],
    ];

    data_get($data, '*.name');

    // ['Desk 1', 'Desk 2'];

<a name="method-data-set"></a>

#### `data_set()`{# collection-method}

`data_set`функція встановлює значення у вкладеному масиві або об'єкті, використовуючи Tagging "точка":

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

Ця функція також приймає символи підстановки і відповідно встановлюватиме значення для цілі:

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2', 'price' => 150],
        ],
    ];

    data_set($data, 'products.*.price', 200);

    /*
        [
            'products' => [
                ['name' => 'Desk 1', 'price' => 200],
                ['name' => 'Desk 2', 'price' => 200],
            ],
        ]
    */

За замовчуванням усі існуючі значення перезаписуються. Якщо ви хочете встановити значення лише тоді, коли воно не існує, ви можете передати`false`як четвертий аргумент:

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200, false);

    // ['products' => ['desk' => ['price' => 100]]]

<a name="method-head"></a>

#### `head()`{# collection-method}

`head`функція повертає перший елемент у заданому масиві:

    $array = [100, 200, 300];

    $first = head($array);

    // 100

<a name="method-last"></a>

#### `last()`{# collection-method}

`last`функція повертає останній елемент у даному масиві:

    $array = [100, 200, 300];

    $last = last($array);

    // 300

<a name="paths"></a>

## Шляхи

<a name="method-app-path"></a>

#### `app_path()`{# collection-method}

`app_path`повертає повністю кваліфікований шлях до`app`каталог. Ви також можете використовувати`app_path`функція для створення повністю кваліфікованого шляху до файлу щодо каталогу додатків:

    $path = app_path();

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>

#### `base_path()`{# collection-method}

`base_path`повертає повний шлях до кореня проекту. Ви також можете використовувати`base_path`функція для створення повноцінного шляху до заданого файлу щодо кореневого каталогу проекту:

    $path = base_path();

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>

#### `config_path()`{# collection-method}

`config_path`повертає повністю кваліфікований шлях до`config`каталог. Ви також можете використовувати`config_path`функція для створення повноцінного шляху до заданого файлу в каталозі конфігурації програми:

    $path = config_path();

    $path = config_path('app.php');

<a name="method-database-path"></a>

#### `database_path()`{# collection-method}

`database_path`повертає повністю кваліфікований шлях до`database`каталог. Ви також можете використовувати`database_path`функція для створення повноцінного шляху до заданого файлу в каталозі бази даних:

    $path = database_path();

    $path = database_path('factories/UserFactory.php');

<a name="method-mix"></a>

#### `mix()`{# collection-method}

`mix`повертає шлях до a[файл версії Mix](/docs/{{version}}/mix):

    $path = mix('css/app.css');

<a name="method-public-path"></a>

#### `public_path()`{# collection-method}

`public_path`повертає повністю кваліфікований шлях до`public`каталог. Ви також можете використовувати`public_path`функція для створення повноцінного шляху до заданого файлу в загальнодоступному каталозі:

    $path = public_path();

    $path = public_path('css/app.css');

<a name="method-resource-path"></a>

#### `resource_path()`{# collection-method}

`resource_path`повертає повністю кваліфікований шлях до`resources`каталог. Ви також можете використовувати`resource_path`функція для створення повноцінного шляху до заданого файлу в каталозі ресурсів:

    $path = resource_path();

    $path = resource_path('sass/app.scss');

<a name="method-storage-path"></a>

#### `storage_path()`{# collection-method}

`storage_path`повертає повністю кваліфікований шлях до`storage`каталог. Ви також можете використовувати`storage_path`функція для створення повноцінного шляху до заданого файлу в каталозі зберігання:

    $path = storage_path();

    $path = storage_path('app/file.txt');

<a name="strings"></a>

## Струни

<a name="method-__"></a>

#### `__()`{# collection-method}

`__`функція переводить заданий рядок перекладу або ключ перекладу за допомогою вашого[файли локалізації](/docs/{{version}}/localization):

    echo __('Welcome to our application');

    echo __('messages.welcome');

Якщо вказаний рядок або ключ перекладу не існує, файл`__`функція поверне задане значення. Отже, використовуючи приклад вище, файл`__`функція повернеться`messages.welcome`якщо цей ключ перекладу не існує.

<a name="method-class-basename"></a>

#### `class_basename()`{# collection-method}

`class_basename`функція повертає ім'я класу даного класу із видаленим простором імен класу:

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>

#### `e()`{# collection-method}

`e`функція запускає PHP`htmlspecialchars`функція за допомогою`double_encode`параметр встановлено на`true`за замовчуванням:

    echo e('<html>foo</html>');

    // &lt;html&gt;foo&lt;/html&gt;

<a name="method-preg-replace-array"></a>

#### `preg_replace_array()`{# collection-method}

`preg_replace_array`Функція послідовно замінює заданий шаблон у рядку за допомогою масиву:

    $string = 'The event will take place between :start and :end';

    $replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-str-after"></a>

#### `Str::after()`{# collection-method}

`Str::after`метод повертає все після заданого значення у рядку. Повернеться весь рядок, якщо значення не існує в рядку:

    use Illuminate\Support\Str;

    $slice = Str::after('This is my name', 'This is');

    // ' my name'

<a name="method-str-after-last"></a>

#### `Str::afterLast()`{# collection-method}

`Str::afterLast`метод повертає все після останнього входження даного значення в рядок. Повернеться весь рядок, якщо значення не існує в рядку:

    use Illuminate\Support\Str;

    $slice = Str::afterLast('App\Http\Controllers\Controller', '\\');

    // 'Controller'

<a name="method-str-ascii"></a>

#### `Str::ascii()`{# collection-method}

`Str::ascii`метод спробує транслітерувати рядок у значення ASCII:

    use Illuminate\Support\Str;

    $slice = Str::ascii('û');

    // 'u'

<a name="method-str-before"></a>

#### `Str::before()`{# collection-method}

`Str::before`метод повертає все перед заданим значенням у рядку:

    use Illuminate\Support\Str;

    $slice = Str::before('This is my name', 'my name');

    // 'This is '

<a name="method-str-before-last"></a>

#### `Str::beforeLast()`{# collection-method}

`Str::beforeLast`метод повертає все до останнього входження даного значення в рядок:

    use Illuminate\Support\Str;

    $slice = Str::beforeLast('This is my name', 'is');

    // 'This '

<a name="method-str-between"></a>

#### `Str::between()`{# collection-method}

`Str::between`метод повертає частину рядка між двома значеннями:

    use Illuminate\Support\Str;

    $slice = Str::between('This is my name', 'This', 'name');

    // ' is my '

<a name="method-camel-case"></a>

#### `Str::camel()`{# collection-method}

`Str::camel`метод перетворює заданий рядок у`camelCase`:

    use Illuminate\Support\Str;

    $converted = Str::camel('foo_bar');

    // fooBar

<a name="method-str-contains"></a>

#### `Str::contains()`{# collection-method}

`Str::contains`метод визначає, чи містить вказаний рядок вказане значення (чутливий до регістру):

    use Illuminate\Support\Str;

    $contains = Str::contains('This is my name', 'my');

    // true

Ви також можете передати масив значень, щоб визначити, чи містить даний рядок якесь із значень:

    use Illuminate\Support\Str;

    $contains = Str::contains('This is my name', ['my', 'foo']);

    // true

<a name="method-str-contains-all"></a>

#### `Str::containsAll()`{# collection-method}

`Str::containsAll`метод визначає, чи містить даний рядок усі значення масиву:

    use Illuminate\Support\Str;

    $containsAll = Str::containsAll('This is my name', ['my', 'name']);

    // true

<a name="method-ends-with"></a>

#### `Str::endsWith()`{# collection-method}

`Str::endsWith`Метод визначає, чи закінчується вказаний рядок заданим значенням:

    use Illuminate\Support\Str;

    $result = Str::endsWith('This is my name', 'name');

    // true

Ви також можете передати масив значень, щоб визначити, чи закінчується даний рядок будь-яким із заданих значень:

    use Illuminate\Support\Str;

    $result = Str::endsWith('This is my name', ['name', 'foo']);

    // true

    $result = Str::endsWith('This is my name', ['this', 'foo']);

    // false

<a name="method-str-finish"></a>

#### `Str::finish()`{# collection-method}

`Str::finish`метод додає один рядок даного значення до рядка, якщо він ще не закінчується значенням:

    use Illuminate\Support\Str;

    $adjusted = Str::finish('this/string', '/');

    // this/string/

    $adjusted = Str::finish('this/string/', '/');

    // this/string/

<a name="method-str-is"></a>

#### `Str::is()`{# collection-method}

`Str::is`Метод визначає, чи відповідає даний рядок даному шаблону. Зірочками можна використовувати для Tagging підстановочних знаків:

    use Illuminate\Support\Str;

    $matches = Str::is('foo*', 'foobar');

    // true

    $matches = Str::is('baz*', 'foobar');

    // false

<a name="method-str-is-ascii"></a>

#### `Str::isAscii()`{# collection-method}

`Str::isAscii`метод визначає, чи є даний рядок 7-бітовим ASCII:

    use Illuminate\Support\Str;

    $isAscii = Str::isAscii('Taylor');

    // true

    $isAscii = Str::isAscii('ü');

    // false

<a name="method-str-is-uuid"></a>

#### `Str::isUuid()`{# collection-method}

`Str::isUuid`метод визначає, чи вказаний рядок є дійсним UUID:

    use Illuminate\Support\Str;

    $isUuid = Str::isUuid('a0a2a2d2-0b87-4a18-83f2-2529882be2de');

    // true

    $isUuid = Str::isUuid('laravel');

    // false

<a name="method-kebab-case"></a>

#### `Str::kebab()`{# collection-method}

`Str::kebab`метод перетворює заданий рядок у`kebab-case`:

    use Illuminate\Support\Str;

    $converted = Str::kebab('fooBar');

    // foo-bar

<a name="method-str-length"></a>

#### `Str::length()`{# collection-method}

`Str::length`метод повертає довжину заданого рядка:

    use Illuminate\Support\Str;

    $length = Str::length('Laravel');

    // 7

<a name="method-str-limit"></a>

#### `Str::limit()`{# collection-method}

`Str::limit`метод усікає заданий рядок із заданою довжиною:

    use Illuminate\Support\Str;

    $truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20);

    // The quick brown fox...

Ви також можете передати третій аргумент, щоб змінити рядок, який буде доданий до кінця:

    use Illuminate\Support\Str;

    $truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20, ' (...)');

    // The quick brown fox (...)

<a name="method-str-lower"></a>

#### `Str::lower()`{# collection-method}

`Str::lower`метод перетворює заданий рядок у нижній регістр:

    use Illuminate\Support\Str;

    $converted = Str::lower('LARAVEL');

    // laravel

<a name="method-str-ordered-uuid"></a>

#### `Str::orderedUuid()`{# collection-method}

`Str::orderedUuid`метод генерує UUID "мітка часу", який може ефективно зберігатися в індексованому стовпці бази даних:

    use Illuminate\Support\Str;

    return (string) Str::orderedUuid();

<a name="method-str-padboth"></a>

#### `Str::padBoth()`{# collection-method}

`Str::padBoth`метод обгортає PHP`str_pad`функція, доповнюючи обидві сторони рядка іншою:

    use Illuminate\Support\Str;

    $padded = Str::padBoth('James', 10, '_');

    // '__James___'

    $padded = Str::padBoth('James', 10);

    // '  James   '

<a name="method-str-padleft"></a>

#### `Str::padLeft()`{# collection-method}

`Str::padLeft`метод обгортає PHP`str_pad`функція, доповнюючи ліву частину рядка іншою:

    use Illuminate\Support\Str;

    $padded = Str::padLeft('James', 10, '-=');

    // '-=-=-James'

    $padded = Str::padLeft('James', 10);

    // '     James'

<a name="method-str-padright"></a>

#### `Str::padRight()`{# collection-method}

`Str::padRight`метод обгортає PHP`str_pad`функція, доповнюючи праву частину рядка іншою:

    use Illuminate\Support\Str;

    $padded = Str::padRight('James', 10, '-');

    // 'James-----'

    $padded = Str::padRight('James', 10);

    // 'James     '

<a name="method-str-plural"></a>

#### `Str::plural()`{# collection-method}

`Str::plural`метод перетворює рядок одного слова у форму множини. Наразі ця функція підтримує лише англійську мову:

    use Illuminate\Support\Str;

    $plural = Str::plural('car');

    // cars

    $plural = Str::plural('child');

    // children

Ви можете вказати ціле число як другий аргумент функції для отримання форми однини чи множини рядка:

    use Illuminate\Support\Str;

    $plural = Str::plural('child', 2);

    // children

    $plural = Str::plural('child', 1);

    // child

<a name="method-str-random"></a>

#### `Str::random()`{# collection-method}

`Str::random`метод генерує випадковий рядок вказаної довжини. Ця функція використовує PHP`random_bytes`функція:

    use Illuminate\Support\Str;

    $random = Str::random(40);

<a name="method-str-replace-array"></a>

#### `Str::replaceArray()`{# collection-method}

`Str::replaceArray`метод послідовно замінює задане значення в рядку за допомогою масиву:

    use Illuminate\Support\Str;

    $string = 'The event will take place between ? and ?';

    $replaced = Str::replaceArray('?', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-str-replace-first"></a>

#### `Str::replaceFirst()`{# collection-method}

`Str::replaceFirst`метод замінює перше входження заданого значення в рядок:

    use Illuminate\Support\Str;

    $replaced = Str::replaceFirst('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // a quick brown fox jumps over the lazy dog

<a name="method-str-replace-last"></a>

#### `Str::replaceLast()`{# collection-method}

`Str::replaceLast`метод замінює останнє входження даного значення в рядок:

    use Illuminate\Support\Str;

    $replaced = Str::replaceLast('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // the quick brown fox jumps over a lazy dog

<a name="method-str-singular"></a>

#### `Str::singular()`{# collection-method}

`Str::singular`метод перетворює рядок у форму однини. Наразі ця функція підтримує лише англійську мову:

    use Illuminate\Support\Str;

    $singular = Str::singular('cars');

    // car

    $singular = Str::singular('children');

    // child

<a name="method-str-slug"></a>

#### `Str::slug()`{# collection-method}

`Str::slug`метод генерує URL-адресу, що відповідає URL-адресам, із заданого рядка:

    use Illuminate\Support\Str;

    $slug = Str::slug('Laravel 5 Framework', '-');

    // laravel-5-framework

<a name="method-snake-case"></a>

#### `Str::snake()`{# collection-method}

`Str::snake`метод перетворює заданий рядок у`snake_case`:

    use Illuminate\Support\Str;

    $converted = Str::snake('fooBar');

    // foo_bar

<a name="method-str-start"></a>

#### `Str::start()`{# collection-method}

`Str::start`метод додає один рядок даного значення до рядка, якщо він ще не починається зі значення:

    use Illuminate\Support\Str;

    $adjusted = Str::start('this/string', '/');

    // /this/string

    $adjusted = Str::start('/this/string', '/');

    // /this/string

<a name="method-starts-with"></a>

#### `Str::startsWith()`{# collection-method}

`Str::startsWith`метод визначає, чи заданий рядок починається з заданого значення:

    use Illuminate\Support\Str;

    $result = Str::startsWith('This is my name', 'This');

    // true

<a name="method-studly-case"></a>

#### `Str::studly()`{# collection-method}

`Str::studly`метод перетворює заданий рядок у`StudlyCase`:

    use Illuminate\Support\Str;

    $converted = Str::studly('foo_bar');

    // FooBar

<a name="method-str-substr"></a>

#### `Str::substr()`{# collection-method}

`Str::substr`метод повертає частину рядка, задану параметрами початку і довжини:

    use Illuminate\Support\Str;

    $converted = Str::substr('The Laravel Framework', 4, 7);

    // Laravel

<a name="method-title-case"></a>

#### `Str::title()`{# collection-method}

`Str::title`метод перетворює заданий рядок у`Title Case`:

    use Illuminate\Support\Str;

    $converted = Str::title('a nice title uses the correct case');

    // A Nice Title Uses The Correct Case

<a name="method-str-ucfirst"></a>

#### `Str::ucfirst()`{# collection-method}

`Str::ucfirst`метод повертає заданий рядок з першим символом з великої літери:

    use Illuminate\Support\Str;

    $string = Str::ucfirst('foo bar');

    // Foo bar

<a name="method-str-upper"></a>

#### `Str::upper()`{# collection-method}

`Str::upper`метод перетворює заданий рядок у верхній регістр:

    use Illuminate\Support\Str;

    $string = Str::upper('laravel');

    // LARAVEL

<a name="method-str-uuid"></a>

#### `Str::uuid()`{# collection-method}

`Str::uuid`метод генерує UUID (версія 4):

    use Illuminate\Support\Str;

    return (string) Str::uuid();

<a name="method-str-words"></a>

#### `Str::words()`{# collection-method}

`Str::words`метод обмежує кількість слів у рядку:

    use Illuminate\Support\Str;

    return Str::words('Perfectly balanced, as all things should be.', 3, ' >>>');

    // Perfectly balanced, as >>>

<a name="method-trans"></a>

#### `trans()`{# collection-method}

`trans`функція переводить даний ключ перекладу за допомогою вашого[файли локалізації](/docs/{{version}}/localization):

    echo trans('messages.welcome');

Якщо вказаний ключ перекладу не існує, файл`trans`функція поверне заданий ключ. Отже, використовуючи приклад вище, файл`trans`функція повернеться`messages.welcome`якщо ключ перекладу не існує.

<a name="method-trans-choice"></a>

#### `trans_choice()`{# collection-method}

`trans_choice`функція переводить даний ключ перекладу з флексією:

    echo trans_choice('messages.notifications', $unreadCount);

Якщо вказаний ключ перекладу не існує, файл`trans_choice`функція поверне заданий ключ. Отже, використовуючи приклад вище, файл`trans_choice`функція повернеться`messages.notifications`якщо ключ перекладу не існує.

<a name="fluent-strings"></a>

## Свободні струнні

Свободні рядки забезпечують більш плавний, об'єктно-орієнтований інтерфейс для роботи зі значеннями рядків, що дозволяє зв'язати кілька операцій рядка разом, використовуючи більш читабельний синтаксис у порівнянні з традиційними рядковими операціями.

<a name="method-fluent-str-after"></a>

#### `after`{# collection-method}

`after`метод повертає все після заданого значення у рядку. Повернеться весь рядок, якщо значення не існує в рядку:

    use Illuminate\Support\Str;

    $slice = Str::of('This is my name')->after('This is');

    // ' my name'

<a name="method-fluent-str-after-last"></a>

#### `afterLast`{# collection-method}

`afterLast`метод повертає все після останнього входження даного значення в рядок. Повернеться весь рядок, якщо значення не існує в рядку:

    use Illuminate\Support\Str;

    $slice = Str::of('App\Http\Controllers\Controller')->afterLast('\\');

    // 'Controller'

<a name="method-fluent-str-append"></a>

#### `append`{# collection-method}

`append`метод додає дані значення до рядка:

    use Illuminate\Support\Str;

    $string = Str::of('Taylor')->append(' Otwell');

    // 'Taylor Otwell'

<a name="method-fluent-str-ascii"></a>

#### `ascii`{# collection-method}

`ascii`метод спробує транслітерувати рядок у значення ASCII:

    use Illuminate\Support\Str;

    $string = Str::of('ü')->ascii();

    // 'u'

<a name="method-fluent-str-basename"></a>

#### `basename`{# collection-method}

`basename`метод повертає кінцевий компонент імені даного рядка:

    use Illuminate\Support\Str;

    $string = Str::of('/foo/bar/baz')->basename();

    // 'baz'

Якщо потрібно, ви можете надати "розширення", яке буде видалено з кінцевого компонента:

    use Illuminate\Support\Str;

    $string = Str::of('/foo/bar/baz.jpg')->basename('.jpg');

    // 'baz'

<a name="method-fluent-str-before"></a>

#### `before`{# collection-method}

`before`метод повертає все перед заданим значенням у рядку:

    use Illuminate\Support\Str;

    $slice = Str::of('This is my name')->before('my name');

    // 'This is '

<a name="method-fluent-str-before-last"></a>

#### `beforeLast`{# collection-method}

`beforeLast`метод повертає все до останнього входження даного значення в рядок:

    use Illuminate\Support\Str;

    $slice = Str::of('This is my name')->beforeLast('is');

    // 'This '

<a name="method-fluent-str-camel"></a>

#### `camel`{# collection-method}

`camel`метод перетворює заданий рядок у`camelCase`:

    use Illuminate\Support\Str;

    $converted = Str::of('foo_bar')->camel();

    // fooBar

<a name="method-fluent-str-contains"></a>

#### `contains`{# collection-method}

`contains`метод визначає, чи містить вказаний рядок вказане значення (чутливий до регістру):

    use Illuminate\Support\Str;

    $contains = Str::of('This is my name')->contains('my');

    // true

Ви також можете передати масив значень, щоб визначити, чи містить даний рядок якесь із значень:

    use Illuminate\Support\Str;

    $contains = Str::of('This is my name')->contains(['my', 'foo']);

    // true

<a name="method-fluent-str-contains-all"></a>

#### `containsAll`{# collection-method}

`containsAll`метод визначає, чи містить даний рядок усі значення масиву:

    use Illuminate\Support\Str;

    $containsAll = Str::of('This is my name')->containsAll(['my', 'name']);

    // true

<a name="method-fluent-str-dirname"></a>

#### `dirname`{# collection-method}

`dirname`метод повертає частину батьківського каталогу даного рядка:

    use Illuminate\Support\Str;

    $string = Str::of('/foo/bar/baz')->dirname();

    // '/foo/bar'

За бажанням Ви можете вказати, скільки рівнів каталогів Ви бажаєте обрізати із рядка:

    use Illuminate\Support\Str;

    $string = Str::of('/foo/bar/baz')->dirname(2);

    // '/foo'

<a name="method-fluent-str-ends-with"></a>

#### `endsWith`{# collection-method}

`endsWith`Метод визначає, чи закінчується вказаний рядок заданим значенням:

    use Illuminate\Support\Str;

    $result = Str::of('This is my name')->endsWith('name');

    // true

Ви також можете передати масив значень, щоб визначити, чи закінчується даний рядок будь-яким із заданих значень:

    use Illuminate\Support\Str;

    $result = Str::of('This is my name')->endsWith(['name', 'foo']);

    // true

    $result = Str::of('This is my name')->endsWith(['this', 'foo']);

    // false

<a name="method-fluent-str-exactly"></a>

#### `exactly`{# collection-method}

`exactly`метод визначає, чи точно відповідає даний рядок іншому рядку:

    use Illuminate\Support\Str;

    $result = Str::of('Laravel')->exactly('Laravel');

    // true

<a name="method-fluent-str-explode"></a>

#### `explode`{# collection-method}

`explode`метод розбиває рядок на заданий роздільник і повертає колекцію, що містить кожен розділ розділеного рядка:

    use Illuminate\Support\Str;

    $collection = Str::of('foo bar baz')->explode(' ');

    // collect(['foo', 'bar', 'baz'])

<a name="method-fluent-str-finish"></a>

#### `finish`{# collection-method}

`finish`метод додає один рядок даного значення до рядка, якщо він ще не закінчується значенням:

    use Illuminate\Support\Str;

    $adjusted = Str::of('this/string')->finish('/');

    // this/string/

    $adjusted = Str::of('this/string/')->finish('/');

    // this/string/

<a name="method-fluent-str-is"></a>

#### `is`{# collection-method}

`is`Метод визначає, чи відповідає даний рядок даному шаблону. Зірочками можна використовувати для Tagging підстановочних знаків:

    use Illuminate\Support\Str;

    $matches = Str::of('foobar')->is('foo*');

    // true

    $matches = Str::of('foobar')->is('baz*');

    // false

<a name="method-fluent-str-is-ascii"></a>

#### `isAscii`{# collection-method}

`isAscii`метод визначає, чи вказаний рядок є рядком ASCII:

    use Illuminate\Support\Str;

    $result = Str::of('Taylor')->isAscii();

    // true

    $result = Str::of('ü')->isAscii();

    // false

<a name="method-fluent-str-is-empty"></a>

#### `isEmpty`{# collection-method}

`isEmpty`метод визначає, чи вказаний рядок порожній:

    use Illuminate\Support\Str;

    $result = Str::of('  ')->trim()->isEmpty();

    // true

    $result = Str::of('Laravel')->trim()->isEmpty();

    // false

<a name="method-fluent-str-is-not-empty"></a>

#### `isNotEmpty`{# collection-method}

`isNotEmpty`метод визначає, чи не вказаний рядок порожнім:

    use Illuminate\Support\Str;

    $result = Str::of('  ')->trim()->isNotEmpty();

    // false

    $result = Str::of('Laravel')->trim()->isNotEmpty();

    // true

<a name="method-fluent-str-kebab"></a>

#### `kebab`{# collection-method}

`kebab`метод перетворює заданий рядок у`kebab-case`:

    use Illuminate\Support\Str;

    $converted = Str::of('fooBar')->kebab();

    // foo-bar

<a name="method-fluent-str-length"></a>

#### `length`{# collection-method}

`length`метод повертає довжину заданого рядка:

    use Illuminate\Support\Str;

    $length = Str::of('Laravel')->length();

    // 7

<a name="method-fluent-str-limit"></a>

#### `limit`{# collection-method}

`limit`метод усікає заданий рядок із заданою довжиною:

    use Illuminate\Support\Str;

    $truncated = Str::of('The quick brown fox jumps over the lazy dog')->limit(20);

    // The quick brown fox...

Ви також можете передати другий аргумент, щоб змінити рядок, який буде доданий до кінця:

    use Illuminate\Support\Str;

    $truncated = Str::of('The quick brown fox jumps over the lazy dog')->limit(20, ' (...)');

    // The quick brown fox (...)

<a name="method-fluent-str-lower"></a>

#### `lower`{# collection-method}

`lower`метод перетворює заданий рядок у нижній регістр:

    use Illuminate\Support\Str;

    $result = Str::of('LARAVEL')->lower();

    // 'laravel'

<a name="method-fluent-str-ltrim"></a>

#### `ltrim`{# collection-method}

`ltrim`метод залишив обрізання заданого рядка:

    use Illuminate\Support\Str;

    $string = Str::of('  Laravel  ')->ltrim();

    // 'Laravel  '

    $string = Str::of('/Laravel/')->ltrim('/');

    // 'Laravel/'

<a name="method-fluent-str-match"></a>

#### `match`{# collection-method}

`match`метод поверне частину рядка, яка відповідає заданому шаблону регулярного виразу:

    use Illuminate\Support\Str;

    $result = Str::of('foo bar')->match('/bar/');

    // 'bar'

    $result = Str::of('foo bar')->match('/foo (.*)/');

    // 'bar'

<a name="method-fluent-str-match-all"></a>

#### `matchAll`{# collection-method}

`matchAll`метод поверне колекцію, що містить частини рядка, що відповідають заданому шаблону регулярних виразів:

    use Illuminate\Support\Str;

    $result = Str::of('bar foo bar')->matchAll('/bar/');

    // collect(['bar', 'bar'])

Якщо ви вказали відповідну групу у виразі, Laravel поверне колекцію збігів цієї групи:

    use Illuminate\Support\Str;

    $result = Str::of('bar fun bar fly')->matchAll('/f(\w*)/');

    // collect(['un', 'ly']);

Якщо збігів не знайдено, буде повернуто порожню колекцію.

<a name="method-fluent-str-padboth"></a>

#### `padBoth`{# collection-method}

`padBoth`метод обгортає PHP`str_pad`функція, доповнюючи обидві сторони рядка іншою:

    use Illuminate\Support\Str;

    $padded = Str::of('James')->padBoth(10, '_');

    // '__James___'

    $padded = Str::of('James')->padBoth(10);

    // '  James   '

<a name="method-fluent-str-padleft"></a>

#### `padLeft`{# collection-method}

`padLeft`метод обгортає PHP`str_pad`функція, доповнюючи ліву частину рядка іншою:

    use Illuminate\Support\Str;

    $padded = Str::of('James')->padLeft(10, '-=');

    // '-=-=-James'

    $padded = Str::of('James')->padLeft(10);

    // '     James'

<a name="method-fluent-str-padright"></a>

#### `padRight`{# collection-method}

`padRight`метод обгортає PHP`str_pad`функція, доповнюючи праву частину рядка іншою:

    use Illuminate\Support\Str;

    $padded = Str::of('James')->padRight(10, '-');

    // 'James-----'

    $padded = Str::of('James')->padRight(10);

    // 'James     '

<a name="method-fluent-str-plural"></a>

#### `plural`{# collection-method}

`plural`метод перетворює рядок одного слова у форму множини. Наразі ця функція підтримує лише англійську мову:

    use Illuminate\Support\Str;

    $plural = Str::of('car')->plural();

    // cars

    $plural = Str::of('child')->plural();

    // children

Ви можете вказати ціле число як другий аргумент функції для отримання форми однини чи множини рядка:

    use Illuminate\Support\Str;

    $plural = Str::of('child')->plural(2);

    // children

    $plural = Str::of('child')->plural(1);

    // child

<a name="method-fluent-str-prepend"></a>

#### `prepend`{# collection-method}

`prepend`метод додає дані значення до рядка:

    use Illuminate\Support\Str;

    $string = Str::of('Framework')->prepend('Laravel ');

    // Laravel Framework

<a name="method-fluent-str-replace"></a>

#### `replace`{# collection-method}

`replace`метод замінює заданий рядок у рядку:

    use Illuminate\Support\Str;

    $replaced = Str::of('Laravel 6.x')->replace('6.x', '7.x');

    // Laravel 7.x

<a name="method-fluent-str-replace-array"></a>

#### `replaceArray`{# collection-method}

`replaceArray`метод послідовно замінює задане значення в рядку за допомогою масиву:

    use Illuminate\Support\Str;

    $string = 'The event will take place between ? and ?';

    $replaced = Str::of($string)->replaceArray('?', ['8:30', '9:00']);

    // The event will take place between 8:30 and 9:00

<a name="method-fluent-str-replace-first"></a>

#### `replaceFirst`{# collection-method}

`replaceFirst`метод замінює перше входження заданого значення в рядок:

    use Illuminate\Support\Str;

    $replaced = Str::of('the quick brown fox jumps over the lazy dog')->replaceFirst('the', 'a');

    // a quick brown fox jumps over the lazy dog

<a name="method-fluent-str-replace-last"></a>

#### `replaceLast`{# collection-method}

`replaceLast`метод замінює останнє входження даного значення в рядок:

    use Illuminate\Support\Str;

    $replaced = Str::of('the quick brown fox jumps over the lazy dog')->replaceLast('the', 'a');

    // the quick brown fox jumps over a lazy dog

<a name="method-fluent-str-replace-matches"></a>

#### `replaceMatches`{# collection-method}

`replaceMatches`метод замінює всі частини рядка, що відповідають заданому шаблону, даним рядком заміни:

    use Illuminate\Support\Str;

    $replaced = Str::of('(+1) 501-555-1000')->replaceMatches('/[^A-Za-z0-9]++/', '')

    // '15015551000'

`replaceMatches`метод також приймає Закриття, яке буде викликано з кожною частиною рядка, що відповідає даній стороні, що дозволяє виконати логіку заміни в межах Закриття та повернути замінене значення:

    use Illuminate\Support\Str;

    $replaced = Str::of('123')->replaceMatches('/\d/', function ($match) {
        return '['.$match[0].']';
    });

    // '[1][2][3]'

<a name="method-fluent-str-rtrim"></a>

#### `rtrim`{# collection-method}

`rtrim`метод праворуч обрізає заданий рядок:

    use Illuminate\Support\Str;

    $string = Str::of('  Laravel  ')->rtrim();

    // '  Laravel'

    $string = Str::of('/Laravel/')->rtrim('/');

    // '/Laravel'

<a name="method-fluent-str-singular"></a>

#### `singular`{# collection-method}

`singular`метод перетворює рядок у форму однини. Наразі ця функція підтримує лише англійську мову:

    use Illuminate\Support\Str;

    $singular = Str::of('cars')->singular();

    // car

    $singular = Str::of('children')->singular();

    // child

<a name="method-fluent-str-slug"></a>

#### `slug`{# collection-method}

`slug`метод генерує URL-адресу, що відповідає URL-адресам, із заданого рядка:

    use Illuminate\Support\Str;

    $slug = Str::of('Laravel Framework')->slug('-');

    // laravel-framework

<a name="method-fluent-str-snake"></a>

#### `snake`{# collection-method}

`snake`метод перетворює заданий рядок у`snake_case`:

    use Illuminate\Support\Str;

    $converted = Str::of('fooBar')->snake();

    // foo_bar

<a name="method-fluent-str-split"></a>

#### `split`{# collection-method}

`split`метод розділяє рядок на колекцію за допомогою регулярного виразу:

    use Illuminate\Support\Str;

    $segments = Str::of('one, two, three')->split('/[\s,]+/');

    // collect(["one", "two", "three"])

<a name="method-fluent-str-start"></a>

#### `start`{# collection-method}

`start`метод додає один рядок даного значення до рядка, якщо він ще не починається зі значення:

    use Illuminate\Support\Str;

    $adjusted = Str::of('this/string')->start('/');

    // /this/string

    $adjusted = Str::of('/this/string')->start('/');

    // /this/string

<a name="method-fluent-str-starts-with"></a>

#### `startsWith`{# collection-method}

`startsWith`метод визначає, чи заданий рядок починається з заданого значення:

    use Illuminate\Support\Str;

    $result = Str::of('This is my name')->startsWith('This');

    // true

<a name="method-fluent-str-studly"></a>

#### `studly`{# collection-method}

`studly`метод перетворює заданий рядок у`StudlyCase`:

    use Illuminate\Support\Str;

    $converted = Str::of('foo_bar')->studly();

    // FooBar

<a name="method-fluent-str-substr"></a>

#### `substr`{# collection-method}

`substr`метод повертає частину рядка, задану заданими параметрами початку та довжини:

    use Illuminate\Support\Str;

    $string = Str::of('Laravel Framework')->substr(8);

    // Framework

    $string = Str::of('Laravel Framework')->substr(8, 5);

    // Frame

<a name="method-fluent-str-title"></a>

#### `title`{# collection-method}

`title`метод перетворює заданий рядок у`Title Case`:

    use Illuminate\Support\Str;

    $converted = Str::of('a nice title uses the correct case')->title();

    // A Nice Title Uses The Correct Case

<a name="method-fluent-str-trim"></a>

#### `trim`{# collection-method}

`trim`метод обрізає заданий рядок:

    use Illuminate\Support\Str;

    $string = Str::of('  Laravel  ')->trim();

    // 'Laravel'

    $string = Str::of('/Laravel/')->trim('/');

    // 'Laravel'

<a name="method-fluent-str-ucfirst"></a>

#### `ucfirst`{# collection-method}

`ucfirst`метод повертає заданий рядок з першим символом з великої літери:

    use Illuminate\Support\Str;

    $string = Str::of('foo bar')->ucfirst();

    // Foo bar

<a name="method-fluent-str-upper"></a>

#### `upper`{# collection-method}

`upper`метод перетворює заданий рядок у верхній регістр:

    use Illuminate\Support\Str;

    $adjusted = Str::of('laravel')->upper();

    // LARAVEL

<a name="method-fluent-str-when"></a>

#### `when`{# collection-method}

`when`метод викликає дане Закриття, якщо задана умова відповідає дійсності. Закриття отримає екземпляр вільного рядка:

    use Illuminate\Support\Str;

    $string = Str::of('Taylor')
                    ->when(true, function ($string) {
                        return $string->append(' Otwell');
                    });

    // 'Taylor Otwell'

Якщо потрібно, ви можете передати інше Закриття як третій параметр до`when`метод. Це Закриття буде виконано, якщо параметр умови має значення`false`.

<a name="method-fluent-str-when-empty"></a>

#### `whenEmpty`{# collection-method}

`whenEmpty`метод викликає дане Закриття, якщо рядок порожній. Якщо Закриття повертає значення, це значення також повернеться`whenEmpty`метод. Якщо закриття не повертає значення, буде повернуто екземпляр плавного рядка:

    use Illuminate\Support\Str;

    $string = Str::of('  ')->whenEmpty(function ($string) {
        return $string->trim()->prepend('Laravel');
    });

    // 'Laravel'

<a name="method-fluent-str-words"></a>

#### `words`{# collection-method}

`words`метод обмежує кількість слів у рядку:

    use Illuminate\Support\Str;

    $string = Str::of('Perfectly balanced, as all things should be.')->words(3, ' >>>');

    // Perfectly balanced, as >>>

<a name="urls"></a>

## URL-адреси

<a name="method-action"></a>

#### `action()`{# collection-method}

`action`функція генерує URL-адресу для даної дії контролера:

    $url = action([HomeController::class, 'index']);

Якщо метод приймає параметри маршруту, ви можете передати їх як другий аргумент методу:

    $url = action([UserController::class, 'profile'], ['id' => 1]);

<a name="method-asset"></a>

#### `asset()`{# collection-method}

`asset`функція генерує URL-адресу для активу, використовуючи поточну схему запиту (HTTP або HTTPS):

    $url = asset('img/photo.jpg');

Ви можете налаштувати хост URL-адреси ресурсу, встановивши`ASSET_URL`змінна у вашому`.env`файл. Це може бути корисно, якщо ви розміщуєте свої Assets на зовнішній службі, як Amazon S3:

    // ASSET_URL=http://example.com/assets

    $url = asset('img/photo.jpg'); // http://example.com/assets/img/photo.jpg

<a name="method-route"></a>

#### `route()`{# collection-method}

`route`функція генерує URL-адресу для вказаного іменованого маршруту:

    $url = route('routeName');

Якщо маршрут приймає параметри, ви можете передати їх як другий аргумент методу:

    $url = route('routeName', ['id' => 1]);

За замовчуванням`route`функція генерує абсолютну URL-адресу. Якщо ви хочете сформувати відносну URL-адресу, ви можете пройти`false`як третій аргумент:

    $url = route('routeName', ['id' => 1], false);

<a name="method-secure-asset"></a>

#### `secure_asset()`{# collection-method}

`secure_asset`функція генерує URL-адресу для активу за допомогою HTTPS&#x3A;

    $url = secure_asset('img/photo.jpg');

<a name="method-secure-url"></a>

#### `secure_url()`{# collection-method}

`secure_url`функція генерує повністю кваліфіковану URL-адресу HTTPS на вказаний шлях:

    $url = secure_url('user/profile');

    $url = secure_url('user/profile', [1]);

<a name="method-url"></a>

#### `url()`{# collection-method}

`url`функція генерує повноцінну URL-адресу на вказаний шлях:

    $url = url('user/profile');

    $url = url('user/profile', [1]);

Якщо шлях не вказаний, файл`Illuminate\Routing\UrlGenerator`екземпляр повертається:

    $current = url()->current();

    $full = url()->full();

    $previous = url()->previous();

<a name="miscellaneous"></a>

## Різне

<a name="method-abort"></a>

#### `abort()`{# collection-method}

`abort`функція кидає[виняток HTTP](/docs/{{version}}/errors#http-exceptions)який буде надано[обробник винятків](/docs/{{version}}/errors#the-exception-handler):

    abort(403);

Ви також можете надати повідомлення про виняток та спеціальні заголовки відповідей, які слід надіслати браузеру:

    abort(403, 'Unauthorized.', $headers);

<a name="method-abort-if"></a>

#### `abort_if()`{# collection-method}

`abort_if`Функція видає виняток HTTP, якщо заданий булевий вираз має значення`true`:

    abort_if(! Auth::user()->isAdmin(), 403);

Подобається`abort`методу, ви також можете надати текст відповіді на виняток як третій аргумент і масив власних заголовків відповідей як четвертий аргумент.

<a name="method-abort-unless"></a>

#### `abort_unless()`{# collection-method}

`abort_unless`Функція видає виняток HTTP, якщо заданий булевий вираз має значення`false`:

    abort_unless(Auth::user()->isAdmin(), 403);

Подобається`abort`методу, ви також можете надати текст відповіді на виняток як третій аргумент і масив власних заголовків відповідей як четвертий аргумент.

<a name="method-app"></a>

#### `app()`{# collection-method}

`app`повертає функцію[службовий контейнер](/docs/{{version}}/container)примірник:

    $container = app();

Ви можете передати ім’я класу або інтерфейсу, щоб вирішити це з контейнера:

    $api = app('HelpSpot\API');

<a name="method-auth"></a>

#### `auth()`{# collection-method}

`auth`повертає функцію[автентифікатор](/docs/{{version}}/authentication)екземпляр. Ви можете використовувати його замість`Auth`фасад для зручності:

    $user = auth()->user();

За потреби ви можете вказати, до якого екземпляра охорони ви хочете отримати доступ:

    $user = auth('admin')->user();

<a name="method-back"></a>

#### `back()`{# collection-method}

`back`функція генерує a[перенаправити відповідь HTTP](/docs/{{version}}/responses#redirects)до попереднього місцезнаходження користувача:

    return back($status = 302, $headers = [], $fallback = false);

    return back();

<a name="method-bcrypt"></a>

#### `bcrypt()`{# collection-method}

`bcrypt`функція[хеші](/docs/{{version}}/hashing)задане значення за допомогою Bcrypt. Ви можете використовувати його як альтернативу`Hash`фасад:

    $password = bcrypt('my-secret-password');

<a name="method-blank"></a>

#### `blank()`{# collection-method}

`blank`функція повертає, чи вказане значення "порожнім":

    blank('');
    blank('   ');
    blank(null);
    blank(collect());

    // true

    blank(0);
    blank(true);
    blank(false);

    // false

Для оберненого до`blank`, див[`filled`](#method-filled)метод.

<a name="method-broadcast"></a>

#### `broadcast()`{# collection-method}

`broadcast`функція[Broadcast](/docs/{{version}}/broadcasting)даного[подія](/docs/{{version}}/events)своїм Listenerам:

    broadcast(new UserRegistered($user));

<a name="method-cache"></a>

#### `cache()`{# collection-method}

`cache`функція може бути використана для отримання значень із[кеш](/docs/{{version}}/cache). Якщо даний ключ не існує в кеші, буде повернуто необов’язкове значення за замовчуванням:

    $value = cache('key');

    $value = cache('key', 'default');

Ви можете додавати елементи до кешу, передаючи функції функції масив пар ключ / значення. Ви також повинні вказати кількість секунд або тривалість кешованого значення слід вважати дійсним:

    cache(['key' => 'value'], 300);

    cache(['key' => 'value'], now()->addSeconds(10));

<a name="method-class-uses-recursive"></a>

#### `class_uses_recursive()`{# collection-method}

`class_uses_recursive`функція повертає всі ознаки, використовувані класом, включаючи ознаки, що використовуються усіма його батьківськими класами:

    $traits = class_uses_recursive(App\Models\User::class);

<a name="method-collect"></a>

#### `collect()`{# collection-method}

`collect`функція створює a[колекція](/docs/{{version}}/collections)екземпляр із заданого значення:

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>

#### `config()`{# collection-method}

`config`функція отримує значення a[конфігурації](/docs/{{version}}/configuration)змінна. Значення конфігурації можна отримати за допомогою синтаксису "крапка", який включає ім'я файлу та опцію, до якої ви хочете отримати доступ. Може бути вказано значення за замовчуванням, яке повертається, якщо параметр конфігурації не існує:

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

Ви можете встановити змінні конфігурації під час виконання, передавши масив пар ключ / значення:

    config(['app.debug' => true]);

<a name="method-cookie"></a>

#### `cookie()`{# collection-method}

`cookie`функція створює нову[Cookies](/docs/{{version}}/requests#cookies)примірник:

    $cookie = cookie('name', 'value', $minutes);

<a name="method-csrf-field"></a>

#### `csrf_field()`{# collection-method}

`csrf_field`функція генерує HTML`hidden`поле введення, що містить значення маркера CSRF. Наприклад, використовуючи[Леза синтаксису](/docs/{{version}}/blade):

    {{ csrf_field() }}

<a name="method-csrf-token"></a>

#### `csrf_token()`{# collection-method}

`csrf_token`функція отримує значення поточного маркера CSRF:

    $token = csrf_token();

<a name="method-dd"></a>

#### `dd()`{# collection-method}

`dd`функція скидає дані змінні і закінчує виконання сценарію:

    dd($value);

    dd($value1, $value2, $value3, ...);

Якщо ви не хочете зупиняти виконання вашого сценарію, використовуйте[`dump`](#method-dump)замість цього.

<a name="method-dispatch"></a>

#### `dispatch()`{# collection-method}

`dispatch`функція штовхає задану[робота](/docs/{{version}}/queues#creating-jobs)на Laravelь[черга на роботу](/docs/{{version}}/queues):

    dispatch(new App\Jobs\SendEmails);

<a name="method-dispatch-now"></a>

#### `dispatch_now()`{# collection-method}

`dispatch_now`функція запускає задану[робота](/docs/{{version}}/queues#creating-jobs)негайно і повертає значення з його`handle`метод:

    $result = dispatch_now(new App\Jobs\SendEmails);

<a name="method-dump"></a>

#### `dump()`{# collection-method}

`dump`функція скидає дані змінні:

    dump($value);

    dump($value1, $value2, $value3, ...);

Якщо ви хочете припинити виконання сценарію після скидання змінних, використовуйте[`dd`](#method-dd)замість цього.

<a name="method-env"></a>

#### `env()`{# collection-method}

`env`функція отримує значення[змінна середовища](/docs/{{version}}/configuration#environment-configuration)або повертає значення за замовчуванням:

    $env = env('APP_ENV');

    // Returns 'production' if APP_ENV is not set...
    $env = env('APP_ENV', 'production');

> {note} Якщо ви виконаєте файл`config:cache`команди під час процесу розгортання, ви повинні бути впевнені, що телефонуєте лише на`env`функція з файлів конфігурації. Після кешування конфігурації файл`.env`файл не буде завантажений, і всі дзвінки до`env`функція повернеться`null`.

<a name="method-event"></a>

#### `event()`{# collection-method}

`event`функція відправляє задану[подія](/docs/{{version}}/events)своїм Listenerам:

    event(new UserRegistered($user));

<a name="method-filled"></a>

#### `filled()`{# collection-method}

`filled`функція повертає, чи не вказане значення "порожнім":

    filled(0);
    filled(true);
    filled(false);

    // true

    filled('');
    filled('   ');
    filled(null);
    filled(collect());

    // false

Для оберненого до`filled`, див[`blank`](#method-blank)метод.

<a name="method-info"></a>

#### `info()`{# collection-method}

`info`функція запише інформацію в[журнал](/docs/{{version}}/logging):

    info('Some helpful information!');

Масив контекстних даних також може бути переданий функції:

    info('User login attempt failed.', ['id' => $user->id]);

<a name="method-logger"></a>

#### `logger()`{# collection-method}

`logger`функція може бути використана для написання`debug`повідомлення рівня до[журнал](/docs/{{version}}/logging):

    logger('Debug message');

Масив контекстних даних також може бути переданий функції:

    logger('User has logged in.', ['id' => $user->id]);

A[журнали](/docs/{{version}}/errors#logging)екземпляр буде повернено, якщо функції не передано значення:

    logger()->error('You are not allowed here.');

<a name="method-method-field"></a>

#### `method_field()`{# collection-method}

`method_field`функція генерує HTML`hidden`поле введення, що містить підроблене значення HTTP-дієслова форми. Наприклад, використовуючи[Леза синтаксису](/docs/{{version}}/blade):

    <form method="POST">
        {{ method_field('DELETE') }}
    </form>

<a name="method-now"></a>

#### `now()`{# collection-method}

`now`функція створює нову`Illuminate\Support\Carbon`приклад для поточного часу:

    $now = now();

<a name="method-old"></a>

#### `old()`{# collection-method}

`old`функція[отримує](/docs/{{version}}/requests#retrieving-input)an[старий вхід](/docs/{{version}}/requests#old-input)значення блимало в сеансі:

    $value = old('value');

    $value = old('value', 'default');

<a name="method-optional"></a>

#### `optional()`{# collection-method}

`optional`Функція приймає будь-який аргумент і дозволяє отримати доступ до властивостей або методів виклику для цього об'єкта. Якщо даний об'єкт є`null`, властивості та методи повернуться`null`замість того, щоб викликати помилку:

    return optional($user->address)->street;

    {!! old('name', optional($user)->name) !!}

`optional`функція також приймає Закриття як другий аргумент. Закриття буде викликано, якщо значення, вказане як перший аргумент, не має значення null:

    return optional(User::find($id), function ($user) {
        return $user->name;
    });

<a name="method-policy"></a>

#### `policy()`{# collection-method}

`policy`метод отримує a[політика](/docs/{{version}}/authorization#creating-policies)екземпляр для даного класу:

    $policy = policy(App\Models\User::class);

<a name="method-redirect"></a>

#### `redirect()`{# collection-method}

`redirect`функція повертає a[перенаправити відповідь HTTP](/docs/{{version}}/responses#redirects), або повертає екземпляр редиректора, якщо викликається без аргументів:

    return redirect($to = null, $status = 302, $headers = [], $secure = null);

    return redirect('/home');

    return redirect()->route('route.name');

<a name="method-report"></a>

#### `report()`{# collection-method}

`report`функція повідомить про виняток, використовуючи ваш[обробник винятків](/docs/{{version}}/errors#the-exception-handler):

    report($e);

<a name="method-request"></a>

#### `request()`{# collection-method}

`request`функція повертає струм[запит](/docs/{{version}}/requests)екземпляр або отримує елемент введення:

    $request = request();

    $value = request('key', $default);

<a name="method-rescue"></a>

#### `rescue()`{# collection-method}

`rescue`функція виконує дане Закриття та вловлює будь-які винятки, що виникають під час його виконання. Усі винятки, які будуть виявлені, будуть надіслані вам[обробник винятків](/docs/{{version}}/errors#the-exception-handler); однак запит продовжить обробку:

    return rescue(function () {
        return $this->method();
    });

Ви також можете передати другий аргумент`rescue`функція. Цей аргумент буде значенням "за замовчуванням", яке слід повернути, якщо під час виконання Закриття виникає виняток:

    return rescue(function () {
        return $this->method();
    }, false);

    return rescue(function () {
        return $this->method();
    }, function () {
        return $this->failure();
    });

<a name="method-resolve"></a>

#### `resolve()`{# collection-method}

`resolve`функція вирішує заданий клас або ім'я інтерфейсу в його екземплярі за допомогою[службовий контейнер](/docs/{{version}}/container):

    $api = resolve('HelpSpot\API');

<a name="method-response"></a>

#### `response()`{# collection-method}

`response`функція створює a[відповідь](/docs/{{version}}/responses)екземпляр або отримує екземпляр фабрики відповідей:

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-retry"></a>

#### `retry()`{# collection-method}

`retry`функція намагається виконати заданий зворотний виклик до досягнення заданого максимального порогу спроби. Якщо зворотний виклик не видає винятку, його повернене значення буде повернуто. Якщо зворотний дзвінок видає виняток, він буде автоматично повторений. Якщо максимальна кількість спроб перевищена, буде вилучено виняток:

    return retry(5, function () {
        // Attempt 5 times while resting 100ms in between attempts...
    }, 100);

<a name="method-session"></a>

#### `session()`{# collection-method}

`session`функція може бути використана для отримання або встановлення[сесія](/docs/{{version}}/session)значення:

    $value = session('key');

Ви можете встановити значення, передавши функції функції масив пар ключ / значення:

    session(['chairs' => 7, 'instruments' => 3]);

Зберігання сеансів буде повернуто, якщо функції не передано значення:

    $value = session()->get('key');

    session()->put('key', $value);

<a name="method-tap"></a>

#### `tap()`{# collection-method}

`tap`Функція приймає два аргументи: довільний`$value`та закриття.`$value`буде передано до закриття, а потім повернене`tap`функція. Повернене значення Закриття не має значення:

    $user = tap(User::first(), function ($user) {
        $user->name = 'taylor';

        $user->save();
    });

Якщо закриття не передається до`tap`функцію, ви можете викликати будь-який метод із заданого`$value`. Повернене значення методу, який ви викликаєте, буде завжди`$value`, незалежно від того, що насправді метод повертає у своєму визначенні. Наприклад, Eloquent`update`метод, як правило, повертає ціле число. Однак ми можемо змусити метод повернути саму модель, ланцюжком`update`виклик методу через`tap`функція:

    $user = tap($user)->update([
        'name' => $name,
        'email' => $email,
    ]);

Щоб додати a`tap`метод до класу, ви можете додати файл`Illuminate\Support\Traits\Tappable`риса до класу.`tap`метод цієї ознаки приймає Закриття як єдиний аргумент. Сам екземпляр об'єкта буде переданий до Закриття, а потім повернутий`tap`метод:

    return $user->tap(function ($user) {
        //
    });

<a name="method-throw-if"></a>

#### `throw_if()`{# collection-method}

`throw_if`Функція викидає даний виняток, якщо заданий логічний вираз має значення`true`:

    throw_if(! Auth::user()->isAdmin(), AuthorizationException::class);

    throw_if(
        ! Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page'
    );

<a name="method-throw-unless"></a>

#### `throw_unless()`{# collection-method}

`throw_unless`Функція викидає даний виняток, якщо заданий логічний вираз має значення`false`:

    throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);

    throw_unless(
        Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page'
    );

<a name="method-today"></a>

#### `today()`{# collection-method}

`today`функція створює нову`Illuminate\Support\Carbon`приклад для поточної дати:

    $today = today();

<a name="method-trait-uses-recursive"></a>

#### `trait_uses_recursive()`{# collection-method}

`trait_uses_recursive`функція повертає всі ознаки, використовувані ознакою:

    $traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);

<a name="method-transform"></a>

#### `transform()`{# collection-method}

`transform`функція виконує a`Closure`для заданого значення, якщо значення не є[порожній](#method-blank)і повертає результат`Closure`:

    $callback = function ($value) {
        return $value * 2;
    };

    $result = transform(5, $callback);

    // 10

Значення за замовчуванням або`Closure`також може бути переданий методу як третій параметр. Це значення буде повернуто, якщо вказане значення порожнє:

    $result = transform(null, $callback, 'The value is blank');

    // The value is blank

<a name="method-validator"></a>

#### `validator()`{# collection-method}

`validator`функція створює нову[валідатор](/docs/{{version}}/validation)з наведеними аргументами. Ви можете використовувати його замість`Validator`фасад для зручності:

    $validator = validator($data, $rules, $messages);

<a name="method-value"></a>

#### `value()`{# collection-method}

`value`функція повертає вказане значення. Однак, якщо ви пройдете`Closure`до функції,`Closure`буде виконано, тоді буде повернено його результат:

    $result = value(true);

    // true

    $result = value(function () {
        return false;
    });

    // false

<a name="method-view"></a>

#### `view()`{# collection-method}

`view`функція отримує a[вид](/docs/{{version}}/views)примірник:

    return view('auth.login');

<a name="method-with"></a>

#### `with()`{# collection-method}

`with`функція повертає вказане значення. Якщо`Closure`передається як другий аргумент функції,`Closure`буде виконано, а його результат буде повернено:

    $callback = function ($value) {
        return (is_numeric($value)) ? $value * 2 : 0;
    };

    $result = with(5, $callback);

    // 10

    $result = with(null, $callback);

    // 0

    $result = with(5, null);

    // 5
