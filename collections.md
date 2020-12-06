# Колекції

-   [Вступ](#introduction)
    -   [Створення колекцій](#creating-collections)
    -   [Розширення колекцій](#extending-collections)
-   [Доступні методи](#available-methods)
-   [Повідомлення вищого порядку](#higher-order-messages)
-   [Ледачі колекції](#lazy-collections)
    -   [Вступ](#lazy-collection-introduction)
    -   [Створення ледачих колекцій](#creating-lazy-collections)
    -   [Контракт, що перелічується](#the-enumerable-contract)
    -   [Методи лінивого збору](#lazy-collection-methods)

<a name="introduction"></a>

## Вступ

`Illuminate\Support\Collection`class забезпечує вільну, зручну обгортку для роботи з масивами даних. Наприклад, перевірте наступний код. Ми будемо використовувати`collect`помічник для створення нового екземпляра колекції з масиву, запустіть`strtoupper`функцію для кожного елемента, а потім видаліть усі порожні елементи:

    $collection = collect(['taylor', 'abigail', null])->map(function ($name) {
        return strtoupper($name);
    })
    ->reject(function ($name) {
        return empty($name);
    });

Як бачите,`Collection`клас дозволяє зв’язати його методи для виконання плавного зіставлення та зменшення базового масиву. Загалом, колекції є незмінними, тобто кожен`Collection`метод повертає абсолютно новий`Collection`інстанції.

<a name="creating-collections"></a>

### Створення колекцій

Як зазначалося вище,`collect`Помічник повертає новий`Illuminate\Support\Collection`екземпляр для даного масиву. Отже, створити колекцію просто:

    $collection = collect([1, 2, 3]);

> {tip} Результати[Красномовний](/docs/{{version}}/eloquent)запити завжди повертаються як`Collection`екземпляри.

<a name="extending-collections"></a>

### Розширення колекцій

Колекції є "макроздатними", що дозволяє додавати додаткові методи до`Collection`клас під час виконання. Наприклад, наступний код додає a`toUpper`метод до`Collection`клас:

    use Illuminate\Support\Collection;
    use Illuminate\Support\Str;

    Collection::macro('toUpper', function () {
        return $this->map(function ($value) {
            return Str::upper($value);
        });
    });

    $collection = collect(['first', 'second']);

    $upper = $collection->toUpper();

    // ['FIRST', 'SECOND']

Як правило, ви повинні оголосити макроси колекції в[постачальник послуг](/docs/{{version}}/providers).

<a name="available-methods"></a>

## Доступні методи

Для решти цієї документації ми обговоримо кожен метод, доступний на`Collection`клас. Пам’ятайте, що всі ці методи можуть бути зв’язані ланцюгами для плавного маніпулювання базовим масивом. Крім того, майже кожен метод повертає новий`Collection`екземпляр, що дозволяє зберегти оригінальну копію колекції, коли це необхідно:

<style>
    #collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    #collection-method-list a {
        display: block;
    }
</style>

<div id="collection-method-list" markdown="1">

[всі](#method-all)[середній](#method-average)[сер](#method-avg)[шматок](#method-chunk)[шматок](#method-chunkwhile)[крах](#method-collapse)[збирати](#method-collect)[комбінувати](#method-combine)[конкат](#method-concat)[містить](#method-contains)[міститьStrict](#method-containsstrict)[рахувати](#method-count)[countBy](#method-countBy)[crossJoin](#method-crossjoin)[dd](#method-dd)[різниця](#method-diff)[diffAssoc](#method-diffassoc)[diffKeys](#method-diffkeys)[смітник](#method-dump)[дублікати](#method-duplicates)[дублікати Строгий](#method-duplicatesstrict)[кожен](#method-each)[коженРозширення](#method-eachspread)[кожен](#method-every)[крім](#method-except)[фільтр](#method-filter)[спочатку](#method-first)[першийДе](#method-first-where)[flatMap](#method-flatmap)[сплющити](#method-flatten)[перевернути](#method-flip)[забути](#method-forget)[forPage](#method-forpage)[отримати](#method-get)[groupBy](#method-groupby)[має](#method-has)[вибухнути](#method-implode)[перетинаються](#method-intersect)[intersectByKeys](#method-intersectbykeys)[пусто](#method-isempty)[isNotEmpty](#method-isnotempty)[приєднуватися](#method-join)[keyBy](#method-keyby)[клавіші](#method-keys)[останній](#method-last)[макрос](#method-macro)[зробити](#method-make)[карта](#method-map)[вступ](#method-mapinto)[mapSpread](#method-mapspread)[mapToGroups](#method-maptogroups)[mapWithKeys](#method-mapwithkeys)[макс](#method-max)[медіана](#method-median)[піти](#method-merge)[mergeRecursive](#method-mergerecursive)[хв](#method-min)[режимі](#method-mode)[n-й](#method-nth)[лише](#method-only)[подушечка](#method-pad)[розділ](#method-partition)[труба](#method-pipe)[πιπεΊντο](#method-pipeinto)[зривати](#method-pluck)[поп](#method-pop)[попередньо](#method-prepend)[тягнути](#method-pull)[штовхати](#method-push)[поставити](#method-put)[випадкові](#method-random)[зменшити](#method-reduce)[відкинути](#method-reject)[замінити](#method-replace)[replaceRecursive](#method-replacerecursive)[зворотний](#method-reverse)[пошук](#method-search)[зміна](#method-shift)[перетасувати](#method-shuffle)[пропустити](#method-skip)[shipUtil](#method-skipuntil)[skipWhile](#method-skipwhile)[скибочка](#method-slice)[дещо](#method-some)[сортувати](#method-sort)[сортувати за](#method-sortby)[sortByDesc](#method-sortbydesc)[sortDesc](#method-sortdesc)[sortKeys](#method-sortkeys)[sortKeysDesc](#method-sortkeysdesc)[зрощення](#method-splice)[розколоти](#method-split)[Спліт](#method-splitin)[сума](#method-sum)[приймати](#method-take)[takeUntil](#method-takeuntil)[takeWhile](#method-takewhile)[натисніть](#method-tap)[разів](#method-times)[toArray](#method-toarray)[до Джсона](#method-tojson)[перетворювати](#method-transform)[союз](#method-union)[унікальний](#method-unique)[uniqueStrict](#method-uniquestrict)[хіба що](#method-unless)[elseEmpty](#method-unlessempty)[ifNotEmpty](#method-unlessnotempty)[розгорнути](#method-unwrap)[значення](#method-values)[коли](#method-when)[whenEmpty](#method-whenempty)[whenNotEmpty](#method-whennotempty)[де](#method-where)[деStrict](#method-wherestrict)[де Між](#method-wherebetween)[де](#method-wherein)[whereInStrict](#method-whereinstrict)[whereInstanceOf](#method-whereinstanceof)[де не між](#method-wherenotbetween)[деНеВ](#method-wherenotin)[деNotInStrict](#method-wherenotinstrict)[де неНуль](#method-wherenotnull)[деНуль](#method-wherenull)[обернути](#method-wrap)[застібку-блискавку](#method-zip)

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

<a name="method-all"></a>

#### `all()`{# collection-method .first-collection-method}

`all`метод повертає базовий масив, представлений колекцією:

    collect([1, 2, 3])->all();

    // [1, 2, 3]

<a name="method-average"></a>

#### `average()`{# collection-method}

Псевдонім для[`avg`](#method-avg)метод.

<a name="method-avg"></a>

#### `avg()`{# collection-method}

`avg`метод повертає[середнє значення](https://en.wikipedia.org/wiki/Average)даного ключа:

    $average = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->avg('foo');

    // 20

    $average = collect([1, 1, 2, 4])->avg();

    // 2

<a name="method-chunk"></a>

#### `chunk()`{# collection-method}

`chunk`метод розбиває колекцію на кілька менших колекцій заданого розміру:

    $collection = collect([1, 2, 3, 4, 5, 6, 7]);

    $chunks = $collection->chunk(4);

    $chunks->all();

    // [[1, 2, 3, 4], [5, 6, 7]]

Цей метод особливо корисний у[погляди](/docs/{{version}}/views)при роботі з сітковою системою, такою як[Bootstrap](https://getbootstrap.com/docs/4.1/layout/grid/). Уявіть, у вас є колекція[Красномовний](/docs/{{version}}/eloquent)моделі, які потрібно відобразити в сітці:

    @foreach ($products->chunk(3) as $chunk)
        <div class="row">
            @foreach ($chunk as $product)
                <div class="col-xs-4">{{ $product->name }}</div>
            @endforeach
        </div>
    @endforeach

<a name="method-chunkwhile"></a>

#### `chunkWhile()`{# collection-method}

`chunkWhile`метод розбиває колекцію на кілька менших колекцій на основі оцінки даного зворотного виклику:

    $collection = collect(str_split('AABBCCCD'));

    $chunks = $collection->chunkWhile(function ($current, $key, $chunk) {
        return $current === $chunk->last();
    });

    $chunks->all();

    // [['A', 'A'], ['B', 'B'], ['C', 'C', 'C'], ['D']]

<a name="method-collapse"></a>

#### `collapse()`{# collection-method}

`collapse`метод згортає колекцію масивів в єдину плоску колекцію:

    $collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    $collapsed = $collection->collapse();

    $collapsed->all();

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-combine"></a>

#### `combine()`{# collection-method}

`combine`метод поєднує значення колекції, як ключі, зі значеннями іншого масиву або колекції:

    $collection = collect(['name', 'age']);

    $combined = $collection->combine(['George', 29]);

    $combined->all();

    // ['name' => 'George', 'age' => 29]

<a name="method-collect"></a>

#### `collect()`{# collection-method}

`collect`метод повертає новий`Collection`екземпляр з предметами, що перебувають на даний момент у колекції:

    $collectionA = collect([1, 2, 3]);

    $collectionB = $collectionA->collect();

    $collectionB->all();

    // [1, 2, 3]

`collect`метод в першу чергу корисний для перетворення[ледачі колекції](#lazy-collections)в стандартний`Collection`екземпляри:

    $lazyCollection = LazyCollection::make(function () {
        yield 1;
        yield 2;
        yield 3;
    });

    $collection = $lazyCollection->collect();

    get_class($collection);

    // 'Illuminate\Support\Collection'

    $collection->all();

    // [1, 2, 3]

> {tip} The`collect`метод особливо корисний, коли у вас є екземпляр`Enumerable`і потрібен не ледачий екземпляр колекції. Оскільки`collect()`є частиною`Enumerable`контракту, ви можете сміливо використовувати його для отримання`Collection`інстанції.

<a name="method-concat"></a>

#### `concat()`{# collection-method}

`concat`метод додає поданий`array`або значення колекції в кінці колекції:

    $collection = collect(['John Doe']);

    $concatenated = $collection->concat(['Jane Doe'])->concat(['name' => 'Johnny Doe']);

    $concatenated->all();

    // ['John Doe', 'Jane Doe', 'Johnny Doe']

<a name="method-contains"></a>

#### `contains()`{# collection-method}

`contains`Метод визначає, чи містить колекція даний предмет:

    $collection = collect(['name' => 'Desk', 'price' => 100]);

    $collection->contains('Desk');

    // true

    $collection->contains('New York');

    // false

Ви також можете передати пару ключ / значення в`contains`метод, який визначатиме, чи існує дана пара в колекції:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);

    $collection->contains('product', 'Bookcase');

    // false

Нарешті, ви також можете передати зворотний дзвінок на`contains`спосіб провести власний тест на істину:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->contains(function ($value, $key) {
        return $value > 5;
    });

    // false

`contains`метод використовує "вільні" порівняння при перевірці значень елементів, тобто рядок із цілим значенням вважатиметься рівним цілому числу того самого значення. Використовувати[`containsStrict`](#method-containsstrict)метод фільтрації за допомогою "суворих" порівнянь.

<a name="method-containsstrict"></a>

#### `containsStrict()`{# collection-method}

Цей метод має той самий підпис, що і[`contains`](#method-contains)метод; однак усі значення порівнюються за допомогою "суворих" порівнянь.

> {tip} Поведінка цього методу змінюється під час використання[Красномовні колекції](/docs/{{version}}/eloquent-collections#method-contains).

<a name="method-count"></a>

#### `count()`{# collection-method}

`count`метод повертає загальну кількість предметів у колекції:

    $collection = collect([1, 2, 3, 4]);

    $collection->count();

    // 4

<a name="method-countBy"></a>

#### `countBy()`{# collection-method}

`countBy`метод підраховує випадки значень у колекції. За замовчуванням метод враховує випадки кожного елемента:

    $collection = collect([1, 2, 2, 2, 3]);

    $counted = $collection->countBy();

    $counted->all();

    // [1 => 1, 2 => 3, 3 => 1]

Однак ви передаєте зворотний дзвінок на`countBy`метод для підрахунку всіх елементів за власним значенням:

    $collection = collect(['alice@gmail.com', 'bob@yahoo.com', 'carlos@gmail.com']);

    $counted = $collection->countBy(function ($email) {
        return substr(strrchr($email, "@"), 1);
    });

    $counted->all();

    // ['gmail.com' => 2, 'yahoo.com' => 1]

<a name="method-crossjoin"></a>

#### `crossJoin()`{# collection-method}

`crossJoin`метод cross поєднує значення колекції серед даних масивів або колекцій, повертаючи декартовий твір з усіма можливими перестановками:

    $collection = collect([1, 2]);

    $matrix = $collection->crossJoin(['a', 'b']);

    $matrix->all();

    /*
        [
            [1, 'a'],
            [1, 'b'],
            [2, 'a'],
            [2, 'b'],
        ]
    */

    $collection = collect([1, 2]);

    $matrix = $collection->crossJoin(['a', 'b'], ['I', 'II']);

    $matrix->all();

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

<a name="method-dd"></a>

#### `dd()`{# collection-method}

`dd`метод скидає елементи колекції і закінчує виконання сценарію:

    $collection = collect(['John Doe', 'Jane Doe']);

    $collection->dd();

    /*
        Collection {
            #items: array:2 [
                0 => "John Doe"
                1 => "Jane Doe"
            ]
        }
    */

Якщо ви не хочете припиняти виконання сценарію, використовуйте[`dump`](#method-dump)замість цього.

<a name="method-diff"></a>

#### `diff()`{# collection-method}

`diff`метод порівнює колекцію з іншою колекцією або звичайним PHP`array`виходячи з його цінностей. Цей метод поверне значення у вихідній колекції, яких немає у даній колекції:

    $collection = collect([1, 2, 3, 4, 5]);

    $diff = $collection->diff([2, 4, 6, 8]);

    $diff->all();

    // [1, 3, 5]

> {tip} Поведінка цього методу змінюється під час використання[Красномовні колекції](/docs/{{version}}/eloquent-collections#method-diff).

<a name="method-diffassoc"></a>

#### `diffAssoc()`{# collection-method}

`diffAssoc`метод порівнює колекцію з іншою колекцією або звичайним PHP`array`на основі його ключів та значень. Цей метод повертає пари ключ / значення в вихідній колекції, яких немає у даній колекції:

    $collection = collect([
        'color' => 'orange',
        'type' => 'fruit',
        'remain' => 6,
    ]);

    $diff = $collection->diffAssoc([
        'color' => 'yellow',
        'type' => 'fruit',
        'remain' => 3,
        'used' => 6,
    ]);

    $diff->all();

    // ['color' => 'orange', 'remain' => 6]

<a name="method-diffkeys"></a>

#### `diffKeys()`{# collection-method}

`diffKeys`метод порівнює колекцію з іншою колекцією або звичайним PHP`array`на основі його ключів. Цей метод повертає пари ключ / значення в вихідній колекції, яких немає у даній колекції:

    $collection = collect([
        'one' => 10,
        'two' => 20,
        'three' => 30,
        'four' => 40,
        'five' => 50,
    ]);

    $diff = $collection->diffKeys([
        'two' => 2,
        'four' => 4,
        'six' => 6,
        'eight' => 8,
    ]);

    $diff->all();

    // ['one' => 10, 'three' => 30, 'five' => 50]

<a name="method-dump"></a>

#### `dump()`{# collection-method}

`dump`метод скидає елементи колекції:

    $collection = collect(['John Doe', 'Jane Doe']);

    $collection->dump();

    /*
        Collection {
            #items: array:2 [
                0 => "John Doe"
                1 => "Jane Doe"
            ]
        }
    */

Якщо ви хочете припинити виконання сценарію після скидання колекції, використовуйте[`dd`](#method-dd)замість цього.

<a name="method-duplicates"></a>

#### `duplicates()`{# collection-method}

`duplicates`метод отримує та повертає повторювані значення з колекції:

    $collection = collect(['a', 'b', 'a', 'c', 'b']);

    $collection->duplicates();

    // [2 => 'a', 4 => 'b']

Якщо колекція містить масиви або об'єкти, ви можете передати ключ атрибутів, які потрібно перевірити на наявність повторюваних значень:

    $employees = collect([
        ['email' => 'abigail@example.com', 'position' => 'Developer'],
        ['email' => 'james@example.com', 'position' => 'Designer'],
        ['email' => 'victoria@example.com', 'position' => 'Developer'],
    ])

    $employees->duplicates('position');

    // [2 => 'Developer']

<a name="method-duplicatesstrict"></a>

#### `duplicatesStrict()`{# collection-method}

Цей метод має той самий підпис, що і[`duplicates`](#method-duplicates)метод; однак усі значення порівнюються за допомогою "суворих" порівнянь.

<a name="method-each"></a>

#### `each()`{# collection-method}

`each`метод перебирає елементи в колекції і передає кожен елемент до зворотного виклику:

    $collection->each(function ($item, $key) {
        //
    });

Якщо ви хочете припинити перегляд елементів, ви можете повернутися`false`з вашого зворотного дзвінка:

    $collection->each(function ($item, $key) {
        if (/* some condition */) {
            return false;
        }
    });

<a name="method-eachspread"></a>

#### `eachSpread()`{# collection-method}

`eachSpread`метод перебирає елементи колекції, передаючи кожне вкладене значення елемента в заданий зворотний виклик:

    $collection = collect([['John Doe', 35], ['Jane Doe', 33]]);

    $collection->eachSpread(function ($name, $age) {
        //
    });

Ви можете припинити перегляд елементів, повернувшись`false`із зворотного дзвінка:

    $collection->eachSpread(function ($name, $age) {
        return false;
    });

<a name="method-every"></a>

#### `every()`{# collection-method}

`every`метод може бути використаний для перевірки того, що всі елементи колекції проходять заданий тест на істинність:

    collect([1, 2, 3, 4])->every(function ($value, $key) {
        return $value > 2;
    });

    // false

Якщо колекція порожня,`every`поверне true:

    $collection = collect([]);

    $collection->every(function ($value, $key) {
        return $value > 2;
    });

    // true

<a name="method-except"></a>

#### `except()`{# collection-method}

`except`метод повертає всі елементи колекції, крім тих, що мають вказані ключі:

    $collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);

    $filtered = $collection->except(['price', 'discount']);

    $filtered->all();

    // ['product_id' => 1]

Для оберненого до`except`, див[лише](#method-only)метод.

> {tip} Поведінка цього методу змінюється під час використання[Красномовні колекції](/docs/{{version}}/eloquent-collections#method-except).

<a name="method-filter"></a>

#### `filter()`{# collection-method}

`filter`метод фільтрує колекцію за допомогою заданого зворотного виклику, зберігаючи лише ті елементи, які проходять заданий тест на істинність:

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->filter(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [3, 4]

Якщо зворотного виклику не надано, усі записи колекції, які еквівалентні`false`буде видалено:

    $collection = collect([1, 2, 3, null, false, '', 0, []]);

    $collection->filter()->all();

    // [1, 2, 3]

Для оберненого до`filter`, див[відкинути](#method-reject)метод.

<a name="method-first"></a>

#### `first()`{# collection-method}

`first`метод повертає перший елемент у колекції, який проходить заданий тест на істинність:

    collect([1, 2, 3, 4])->first(function ($value, $key) {
        return $value > 2;
    });

    // 3

Ви також можете зателефонувати до`first`метод без аргументів для отримання першого елемента в колекції. Якщо колекція порожня,`null`повертається:

    collect([1, 2, 3, 4])->first();

    // 1

<a name="method-first-where"></a>

#### `firstWhere()`{# collection-method}

`firstWhere`метод повертає перший елемент у колекції із заданою парою ключ / значення:

    $collection = collect([
        ['name' => 'Regena', 'age' => null],
        ['name' => 'Linda', 'age' => 14],
        ['name' => 'Diego', 'age' => 23],
        ['name' => 'Linda', 'age' => 84],
    ]);

    $collection->firstWhere('name', 'Linda');

    // ['name' => 'Linda', 'age' => 14]

Ви також можете зателефонувати до`firstWhere`метод з оператором:

    $collection->firstWhere('age', '>=', 18);

    // ['name' => 'Diego', 'age' => 23]

Подобається[де](#method-where)метод, ви можете передати один аргумент до`firstWhere`метод. У цьому випадку сценарій`firstWhere`метод поверне перший елемент, де значення даного ключа елемента "істинно":

    $collection->firstWhere('age');

    // ['name' => 'Linda', 'age' => 14]

<a name="method-flatmap"></a>

#### `flatMap()`{# collection-method}

`flatMap`метод перебирає колекцію та передає кожне значення до заданого зворотного виклику. Зворотний дзвінок може змінювати товар та повертати його, таким чином формуючи нову колекцію модифікованих елементів. Потім масив згладжується рівнем:

    $collection = collect([
        ['name' => 'Sally'],
        ['school' => 'Arkansas'],
        ['age' => 28]
    ]);

    $flattened = $collection->flatMap(function ($values) {
        return array_map('strtoupper', $values);
    });

    $flattened->all();

    // ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];

<a name="method-flatten"></a>

#### `flatten()`{# collection-method}

`flatten`метод вирівнює багатовимірну колекцію в один вимір:

    $collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

    $flattened = $collection->flatten();

    $flattened->all();

    // ['taylor', 'php', 'javascript'];

За бажанням ви можете передати функції аргумент "глибина":

    $collection = collect([
        'Apple' => [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
        ],
        'Samsung' => [
            ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
        ],
    ]);

    $products = $collection->flatten(1);

    $products->values()->all();

    /*
        [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
            ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
        ]
    */

У цьому прикладі виклик`flatten`без надання глибини також згладив би вкладені масиви, в результаті чого`['iPhone 6S', 'Apple', 'Galaxy S7', 'Samsung']`. Надання глибини дозволяє обмежити рівні вкладених масивів, які будуть згладжені.

<a name="method-flip"></a>

#### `flip()`{# collection-method}

`flip`метод міняє місцями ключі колекції з відповідними значеннями:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $flipped = $collection->flip();

    $flipped->all();

    // ['taylor' => 'name', 'laravel' => 'framework']

<a name="method-forget"></a>

#### `forget()`{# collection-method}

`forget`метод видаляє елемент із колекції за його ключем:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $collection->forget('name');

    $collection->all();

    // ['framework' => 'laravel']

> {note} На відміну від більшості інших методів збору,`forget`не повертає нову змінену колекцію; він змінює колекцію, до якої його викликають.

<a name="method-forpage"></a>

#### `forPage()`{# collection-method}

`forPage`метод повертає нову колекцію, що містить елементи, які будуть присутні на заданому номері сторінки. Метод приймає номер сторінки як перший аргумент, а кількість елементів, що відображаються на сторінці, як другий аргумент:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunk = $collection->forPage(2, 3);

    $chunk->all();

    // [4, 5, 6]

<a name="method-get"></a>

#### `get()`{# collection-method}

`get`метод повертає елемент за заданим ключем. Якщо ключ не існує,`null`повертається:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('name');

    // taylor

Ви можете додатково передати значення за замовчуванням як другий аргумент:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('foo', 'default-value');

    // default-value

Ви навіть можете передати зворотний дзвінок як значення за замовчуванням. Результат зворотного виклику буде повернено, якщо вказаний ключ не існує:

    $collection->get('email', function () {
        return 'default-value';
    });

    // default-value

<a name="method-groupby"></a>

#### `groupBy()`{# collection-method}

`groupBy`метод групує елементи колекції за заданим ключем:

    $collection = collect([
        ['account_id' => 'account-x10', 'product' => 'Chair'],
        ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ['account_id' => 'account-x11', 'product' => 'Desk'],
    ]);

    $grouped = $collection->groupBy('account_id');

    $grouped->all();

    /*
        [
            'account-x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'account-x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

Замість передачі рядка`key`, Ви можете передати зворотний дзвінок. Зворотний дзвінок повинен повернути значення, яке ви хочете ввести в групу:

    $grouped = $collection->groupBy(function ($item, $key) {
        return substr($item['account_id'], -3);
    });

    $grouped->all();

    /*
        [
            'x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

Кілька критеріїв групування можуть передаватися як масив. Кожен елемент масиву буде застосовано до відповідного рівня в багатовимірному масиві:

    $data = new Collection([
        10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
        20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
        30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
        40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
    ]);

    $result = $data->groupBy([
        'skill',
        function ($item) {
            return $item['roles'];
        },
    ], $preserveKeys = true);

    /*
    [
        1 => [
            'Role_1' => [
                10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
                20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
            ],
            'Role_2' => [
                20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
            ],
            'Role_3' => [
                10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
            ],
        ],
        2 => [
            'Role_1' => [
                30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
            ],
            'Role_2' => [
                40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
            ],
        ],
    ];
    */

<a name="method-has"></a>

#### `has()`{# collection-method}

`has`Метод визначає, чи існує даний колекція в колекції:

    $collection = collect(['account_id' => 1, 'product' => 'Desk', 'amount' => 5]);

    $collection->has('product');

    // true

    $collection->has(['product', 'amount']);

    // true

    $collection->has(['amount', 'price']);

    // false

<a name="method-implode"></a>

#### `implode()`{# collection-method}

`implode`метод приєднує елементи до колекції. Його аргументи залежать від типу предметів у колекції. Якщо колекція містить масиви або об'єкти, ви повинні передати ключ атрибутів, до яких ви хочете приєднатися, і рядок "склеювання", який ви хочете розмістити між значеннями:

    $collection = collect([
        ['account_id' => 1, 'product' => 'Desk'],
        ['account_id' => 2, 'product' => 'Chair'],
    ]);

    $collection->implode('product', ', ');

    // Desk, Chair

Якщо колекція містить прості рядки або числові значення, передайте "клей" як єдиний аргумент методу:

    collect([1, 2, 3, 4, 5])->implode('-');

    // '1-2-3-4-5'

<a name="method-intersect"></a>

#### `intersect()`{# collection-method}

`intersect`метод видаляє будь-які значення з вихідної колекції, які відсутні в даному`array`або колекція. Отримана колекція збереже ключі оригінальної колекції:

    $collection = collect(['Desk', 'Sofa', 'Chair']);

    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

    $intersect->all();

    // [0 => 'Desk', 2 => 'Chair']

> {tip} Поведінка цього методу змінюється під час використання[Красномовні колекції](/docs/{{version}}/eloquent-collections#method-intersect).

<a name="method-intersectbykeys"></a>

#### `intersectByKeys()`{# collection-method}

`intersectByKeys`метод видаляє будь-які ключі з оригінальної колекції, яких немає у даній`array`або колекція:

    $collection = collect([
        'serial' => 'UX301', 'type' => 'screen', 'year' => 2009,
    ]);

    $intersect = $collection->intersectByKeys([
        'reference' => 'UX404', 'type' => 'tab', 'year' => 2011,
    ]);

    $intersect->all();

    // ['type' => 'screen', 'year' => 2009]

<a name="method-isempty"></a>

#### `isEmpty()`{# collection-method}

`isEmpty`метод повертає`true`якщо колекція порожня; інакше,`false`повертається:

    collect([])->isEmpty();

    // true

<a name="method-isnotempty"></a>

#### `isNotEmpty()`{# collection-method}

`isNotEmpty`метод повертає`true`якщо колекція не порожня; інакше,`false`повертається:

    collect([])->isNotEmpty();

    // false

<a name="method-join"></a>

#### `join()`{# collection-method}

`join`метод об'єднує значення колекції за допомогою рядка:

    collect(['a', 'b', 'c'])->join(', '); // 'a, b, c'
    collect(['a', 'b', 'c'])->join(', ', ', and '); // 'a, b, and c'
    collect(['a', 'b'])->join(', ', ' and '); // 'a and b'
    collect(['a'])->join(', ', ' and '); // 'a'
    collect([])->join(', ', ' and '); // ''

<a name="method-keyby"></a>

#### `keyBy()`{# collection-method}

`keyBy`метод ключі колекція за даним ключем. Якщо кілька елементів мають однаковий ключ, у новій колекції з’явиться лише останній:

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keyed = $collection->keyBy('product_id');

    $keyed->all();

    /*
        [
            'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

Ви також можете передати метод зворотного виклику. Зворотний виклик повинен повернути значення для ключа колекції за допомогою:

    $keyed = $collection->keyBy(function ($item) {
        return strtoupper($item['product_id']);
    });

    $keyed->all();

    /*
        [
            'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

<a name="method-keys"></a>

#### `keys()`{# collection-method}

`keys`метод повертає всі ключі колекції:

    $collection = collect([
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keys = $collection->keys();

    $keys->all();

    // ['prod-100', 'prod-200']

<a name="method-last"></a>

#### `last()`{# collection-method}

`last`метод повертає останній елемент у колекції, який проходить заданий тест на істинність:

    collect([1, 2, 3, 4])->last(function ($value, $key) {
        return $value < 3;
    });

    // 2

Ви також можете зателефонувати до`last`метод без аргументів для отримання останнього елемента в колекції. Якщо колекція порожня,`null`повертається:

    collect([1, 2, 3, 4])->last();

    // 4

<a name="method-macro"></a>

#### `macro()`{# collection-method}

Статичний`macro`метод дозволяє додавати методи до`Collection`клас під час виконання. Зверніться до документації на[розширення колекцій](#extending-collections)для отримання додаткової інформації.

<a name="method-make"></a>

#### `make()`{# collection-method}

Статичний`make`метод створює новий екземпляр колекції. Див[Створення колекцій](#creating-collections)розділ.

<a name="method-map"></a>

#### `map()`{# collection-method}

`map`метод перебирає колекцію та передає кожне значення до заданого зворотного виклику. Зворотний дзвінок може змінювати товар та повертати його, таким чином формуючи нову колекцію модифікованих елементів:

    $collection = collect([1, 2, 3, 4, 5]);

    $multiplied = $collection->map(function ($item, $key) {
        return $item * 2;
    });

    $multiplied->all();

    // [2, 4, 6, 8, 10]

> {note} Як і більшість інших методів збору,`map`повертає новий екземпляр колекції; він не змінює колекцію, до якої його викликають. Якщо ви хочете перетворити оригінальну колекцію, використовуйте[`transform`](#method-transform)метод.

<a name="method-mapinto"></a>

#### `mapInto()`{# collection-method}

`mapInto()`метод перебирає колекцію, створюючи новий екземпляр даного класу, передаючи значення в конструктор:

    class Currency
    {
        /**
         * Create a new currency instance.
         *
         * @param  string  $code
         * @return void
         */
        function __construct(string $code)
        {
            $this->code = $code;
        }
    }

    $collection = collect(['USD', 'EUR', 'GBP']);

    $currencies = $collection->mapInto(Currency::class);

    $currencies->all();

    // [Currency('USD'), Currency('EUR'), Currency('GBP')]

<a name="method-mapspread"></a>

#### `mapSpread()`{# collection-method}

`mapSpread`метод перебирає елементи колекції, передаючи кожне вкладене значення елемента в заданий зворотний виклик. Зворотний дзвінок може змінювати товар та повертати його, таким чином формуючи нову колекцію модифікованих елементів:

    $collection = collect([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunks = $collection->chunk(2);

    $sequence = $chunks->mapSpread(function ($even, $odd) {
        return $even + $odd;
    });

    $sequence->all();

    // [1, 5, 9, 13, 17]

<a name="method-maptogroups"></a>

#### `mapToGroups()`{# collection-method}

`mapToGroups`метод групує елементи колекції за даним зворотним викликом. Зворотний виклик повинен повернути асоціативний масив, що містить одну пару ключ / значення, формуючи таким чином нову колекцію згрупованих значень:

    $collection = collect([
        [
            'name' => 'John Doe',
            'department' => 'Sales',
        ],
        [
            'name' => 'Jane Doe',
            'department' => 'Sales',
        ],
        [
            'name' => 'Johnny Doe',
            'department' => 'Marketing',
        ]
    ]);

    $grouped = $collection->mapToGroups(function ($item, $key) {
        return [$item['department'] => $item['name']];
    });

    $grouped->all();

    /*
        [
            'Sales' => ['John Doe', 'Jane Doe'],
            'Marketing' => ['Johnny Doe'],
        ]
    */

    $grouped->get('Sales')->all();

    // ['John Doe', 'Jane Doe']

<a name="method-mapwithkeys"></a>

#### `mapWithKeys()`{# collection-method}

`mapWithKeys`метод перебирає колекцію та передає кожне значення до заданого зворотного виклику. Зворотний виклик повинен повернути асоціативний масив, що містить одну пару ключ / значення:

    $collection = collect([
        [
            'name' => 'John',
            'department' => 'Sales',
            'email' => 'john@example.com',
        ],
        [
            'name' => 'Jane',
            'department' => 'Marketing',
            'email' => 'jane@example.com',
        ]
    ]);

    $keyed = $collection->mapWithKeys(function ($item) {
        return [$item['email'] => $item['name']];
    });

    $keyed->all();

    /*
        [
            'john@example.com' => 'John',
            'jane@example.com' => 'Jane',
        ]
    */

<a name="method-max"></a>

#### `max()`{# collection-method}

`max`метод повертає максимальне значення заданого ключа:

    $max = collect([['foo' => 10], ['foo' => 20]])->max('foo');

    // 20

    $max = collect([1, 2, 3, 4, 5])->max();

    // 5

<a name="method-median"></a>

#### `median()`{# collection-method}

`median`метод повертає[серединне значення](https://en.wikipedia.org/wiki/Median)даного ключа:

    $median = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->median('foo');

    // 15

    $median = collect([1, 1, 2, 4])->median();

    // 1.5

<a name="method-merge"></a>

#### `merge()`{# collection-method}

`merge`метод об'єднує даний масив або колекцію з вихідною колекцією. Якщо рядовий ключ у даних елементах відповідає строковому ключу в оригінальній колекції, значення даних елементів замінить значення в оригінальній колекції:

    $collection = collect(['product_id' => 1, 'price' => 100]);

    $merged = $collection->merge(['price' => 200, 'discount' => false]);

    $merged->all();

    // ['product_id' => 1, 'price' => 200, 'discount' => false]

Якщо ключі даних елементів є числовими, значення будуть додані до кінця колекції:

    $collection = collect(['Desk', 'Chair']);

    $merged = $collection->merge(['Bookcase', 'Door']);

    $merged->all();

    // ['Desk', 'Chair', 'Bookcase', 'Door']

<a name="method-mergerecursive"></a>

#### `mergeRecursive()`{# collection-method}

`mergeRecursive`метод рекурсивно зливає даний масив або колекцію з вихідною колекцією. Якщо рядовий ключ у даних елементах відповідає рядковому ключу в оригінальній колекції, тоді значення цих ключів об’єднуються разом у масив, і це робиться рекурсивно:

    $collection = collect(['product_id' => 1, 'price' => 100]);

    $merged = $collection->mergeRecursive(['product_id' => 2, 'price' => 200, 'discount' => false]);

    $merged->all();

    // ['product_id' => [1, 2], 'price' => [100, 200], 'discount' => false]

<a name="method-min"></a>

#### `min()`{# collection-method}

`min`метод повертає мінімальне значення даного ключа:

    $min = collect([['foo' => 10], ['foo' => 20]])->min('foo');

    // 10

    $min = collect([1, 2, 3, 4, 5])->min();

    // 1

<a name="method-mode"></a>

#### `mode()`{# collection-method}

`mode`метод повертає[значення режиму](https://en.wikipedia.org/wiki/Mode_(statistics))даного ключа:

    $mode = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->mode('foo');

    // [10]

    $mode = collect([1, 1, 2, 4])->mode();

    // [1]

<a name="method-nth"></a>

#### `nth()`{# collection-method}

`nth`метод створює нову колекцію, що складається з кожного n-го елемента:

    $collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

    $collection->nth(4);

    // ['a', 'e']

Ви можете додатково передати зміщення як другий аргумент:

    $collection->nth(4, 1);

    // ['b', 'f']

<a name="method-only"></a>

#### `only()`{# collection-method}

`only`метод повертає елементи в колекції із зазначеними ключами:

    $collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

    $filtered = $collection->only(['product_id', 'name']);

    $filtered->all();

    // ['product_id' => 1, 'name' => 'Desk']

Для оберненого до`only`, див[крім](#method-except)метод.

> {tip} Поведінка цього методу змінюється під час використання[Красномовні колекції](/docs/{{version}}/eloquent-collections#method-only).

<a name="method-pad"></a>

#### `pad()`{# collection-method}

`pad`метод буде заповнювати масив заданим значенням, поки масив не досягне заданого розміру. Цей метод поводиться як[array_pad](https://secure.php.net/manual/en/function.array-pad.php)Функція PHP.

Щоб додати ліворуч, слід вказати від’ємний розмір. Заповнення не відбудеться, якщо абсолютне значення заданого розміру менше або дорівнює довжині масиву:

    $collection = collect(['A', 'B', 'C']);

    $filtered = $collection->pad(5, 0);

    $filtered->all();

    // ['A', 'B', 'C', 0, 0]

    $filtered = $collection->pad(-5, 0);

    $filtered->all();

    // [0, 0, 'A', 'B', 'C']

<a name="method-partition"></a>

#### `partition()`{# collection-method}

`partition`метод може поєднуватися з`list`Функція PHP для відокремлення елементів, які проходять заданий тест на істину, від тих, які цього не роблять:

    $collection = collect([1, 2, 3, 4, 5, 6]);

    list($underThree, $equalOrAboveThree) = $collection->partition(function ($i) {
        return $i < 3;
    });

    $underThree->all();

    // [1, 2]

    $equalOrAboveThree->all();

    // [3, 4, 5, 6]

<a name="method-pipe"></a>

#### `pipe()`{# collection-method}

`pipe`метод передає колекцію даному зворотному виклику і повертає результат:

    $collection = collect([1, 2, 3]);

    $piped = $collection->pipe(function ($collection) {
        return $collection->sum();
    });

    // 6

<a name="method-pipeinto"></a>

#### `pipeInto()`{# collection-method}

`pipeInto`метод створює новий екземпляр даного класу і передає колекцію в конструктор:

    class ResourceCollection
    {
        /**
         * The Collection instance.
         */
        public $collection;

        /**
         * Create a new ResourceCollection instance.
         *
         * @param  Collection  $resource
         * @return void
         */
        public function __construct(Collection $collection)
        {
            $this->collection = $collection;
        }
    }

    $collection = collect([1, 2, 3]);

    $resource = $collection->pipeInto(ResourceCollection::class);

    $resource->collection->all();

    // [1, 2, 3]

<a name="method-pluck"></a>

#### `pluck()`{# collection-method}

`pluck`метод отримує всі значення для даного ключа:

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $plucked = $collection->pluck('name');

    $plucked->all();

    // ['Desk', 'Chair']

Ви також можете вказати, як ви хочете вводити отриману колекцію:

    $plucked = $collection->pluck('name', 'product_id');

    $plucked->all();

    // ['prod-100' => 'Desk', 'prod-200' => 'Chair']

`pluck`метод також підтримує отримання вкладених значень за допомогою позначення "точка":

    $collection = collect([
        [
            'speakers' => [
                'first_day' => ['Rosa', 'Judith'],
                'second_day' => ['Angela', 'Kathleen'],
            ],
        ],
    ]);

    $plucked = $collection->pluck('speakers.first_day');

    $plucked->all();

    // ['Rosa', 'Judith']

Якщо існують повторювані ключі, останній відповідний елемент буде вставлений до зібраної колекції:

    $collection = collect([
        ['brand' => 'Tesla',  'color' => 'red'],
        ['brand' => 'Pagani', 'color' => 'white'],
        ['brand' => 'Tesla',  'color' => 'black'],
        ['brand' => 'Pagani', 'color' => 'orange'],
    ]);

    $plucked = $collection->pluck('color', 'brand');

    $plucked->all();

    // ['Tesla' => 'black', 'Pagani' => 'orange']

<a name="method-pop"></a>

#### `pop()`{# collection-method}

`pop`метод видаляє та повертає останній елемент із колекції:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->pop();

    // 5

    $collection->all();

    // [1, 2, 3, 4]

<a name="method-prepend"></a>

#### `prepend()`{# collection-method}

`prepend`метод додає елемент на початок колекції:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->prepend(0);

    $collection->all();

    // [0, 1, 2, 3, 4, 5]

Ви також можете передати другий аргумент для встановлення ключа попередньо доданого елемента:

    $collection = collect(['one' => 1, 'two' => 2]);

    $collection->prepend(0, 'zero');

    $collection->all();

    // ['zero' => 0, 'one' => 1, 'two' => 2]

<a name="method-pull"></a>

#### `pull()`{# collection-method}

`pull`метод видаляє та повертає елемент із колекції за його ключем:

    $collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

    $collection->pull('name');

    // 'Desk'

    $collection->all();

    // ['product_id' => 'prod-100']

<a name="method-push"></a>

#### `push()`{# collection-method}

`push`метод додає елемент до кінця колекції:

    $collection = collect([1, 2, 3, 4]);

    $collection->push(5);

    $collection->all();

    // [1, 2, 3, 4, 5]

<a name="method-put"></a>

#### `put()`{# collection-method}

`put`метод встановлює заданий ключ і значення в колекції:

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $collection->put('price', 100);

    $collection->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100]

<a name="method-random"></a>

#### `random()`{# collection-method}

`random`метод повертає випадковий елемент із колекції:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->random();

    // 4 - (retrieved randomly)

За бажанням ви можете передати ціле число`random`щоб вказати, скільки елементів ви хотіли б отримати випадковим чином. Колекція предметів завжди повертається, коли явно передається кількість елементів, які ви хочете отримати:

    $random = $collection->random(3);

    $random->all();

    // [2, 4, 5] - (retrieved randomly)

Якщо у колекції менше предметів, ніж запитувалося, метод видасть`InvalidArgumentException`.

<a name="method-reduce"></a>

#### `reduce()`{# collection-method}

`reduce`метод зменшує колекцію до одного значення, передаючи результат кожної ітерації в наступну ітерацію:

    $collection = collect([1, 2, 3]);

    $total = $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    });

    // 6

Значення для`$carry`на першій ітерації`null`; однак ви можете вказати його початкове значення, передавши другий аргумент`reduce`:

    $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    }, 4);

    // 10

<a name="method-reject"></a>

#### `reject()`{# collection-method}

`reject`метод фільтрує колекцію за допомогою заданого зворотного виклику. Зворотний дзвінок повинен повернутися`true`якщо предмет слід вилучити з отриманої колекції:

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->reject(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [1, 2]

Для оберненого до`reject`метод, див[`filter`](#method-filter)метод.

<a name="method-replace"></a>

#### `replace()`{# collection-method}

`replace`метод поводиться подібно до`merge`; однак, на додаток до перезапису відповідних елементів за допомогою рядкових ключів,`replace`метод також перезапише елементи в колекції, які мають відповідні числові клавіші:

    $collection = collect(['Taylor', 'Abigail', 'James']);

    $replaced = $collection->replace([1 => 'Victoria', 3 => 'Finn']);

    $replaced->all();

    // ['Taylor', 'Victoria', 'James', 'Finn']

<a name="method-replacerecursive"></a>

#### `replaceRecursive()`{# collection-method}

Цей метод працює як`replace`, але він перетвориться на масиви і застосує той самий процес заміни до внутрішніх значень:

    $collection = collect(['Taylor', 'Abigail', ['James', 'Victoria', 'Finn']]);

    $replaced = $collection->replaceRecursive(['Charlie', 2 => [1 => 'King']]);

    $replaced->all();

    // ['Charlie', 'Abigail', ['James', 'King', 'Finn']]

<a name="method-reverse"></a>

#### `reverse()`{# collection-method}

`reverse`метод змінює порядок предметів колекції, зберігаючи оригінальні ключі:

    $collection = collect(['a', 'b', 'c', 'd', 'e']);

    $reversed = $collection->reverse();

    $reversed->all();

    /*
        [
            4 => 'e',
            3 => 'd',
            2 => 'c',
            1 => 'b',
            0 => 'a',
        ]
    */

<a name="method-search"></a>

#### `search()`{# collection-method}

`search`метод шукає в колекції задане значення і повертає його ключ, якщо його знайдено. Якщо предмет не знайдено,`false`повертається.

    $collection = collect([2, 4, 6, 8]);

    $collection->search(4);

    // 1

Пошук здійснюється за допомогою "вільного" порівняння, тобто рядок із цілим значенням вважатиметься рівним цілому числу того самого значення. Щоб скористатися "суворим" порівнянням, здайте`true`як другий аргумент методу:

    $collection->search('4', true);

    // false

Крім того, ви можете передати власний зворотний дзвінок для пошуку першого елемента, який проходить тест на істину:

    $collection->search(function ($item, $key) {
        return $item > 5;
    });

    // 2

<a name="method-shift"></a>

#### `shift()`{# collection-method}

`shift`метод видаляє та повертає перший елемент із колекції:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->shift();

    // 1

    $collection->all();

    // [2, 3, 4, 5]

<a name="method-shuffle"></a>

#### `shuffle()`{# collection-method}

`shuffle`метод випадковим чином перетасовує елементи колекції:

    $collection = collect([1, 2, 3, 4, 5]);

    $shuffled = $collection->shuffle();

    $shuffled->all();

    // [3, 2, 5, 1, 4] - (generated randomly)

<a name="method-skip"></a>

#### `skip()`{# collection-method}

`skip`метод повертає нову колекцію без першої заданої кількості елементів:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $collection = $collection->skip(4);

    $collection->all();

    // [5, 6, 7, 8, 9, 10]

<a name="method-skipuntil"></a>

#### `skipUntil()`{# collection-method}

`skipUntil`метод пропускає елементи, поки не повернеться даний зворотний виклик`true`а потім повертає решту елементів у колекції:

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->skipUntil(function ($item) {
        return $item >= 3;
    });

    $subset->all();

    // [3, 4]

Ви також можете передати просте значення в`skipUntil`метод, щоб пропустити всі елементи, поки не буде знайдено задане значення:

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->skipUntil(3);

    $subset->all();

    // [3, 4]

> {note} Якщо заданого значення не знайдено або зворотний виклик ніколи не повернеться`true`,`skipUntil`метод поверне порожню колекцію.

<a name="method-skipwhile"></a>

#### `skipWhile()`{# collection-method}

`skipWhile`метод пропускає елементи, поки наданий зворотний виклик повертається`true`а потім повертає решту елементів у колекції:

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->skipWhile(function ($item) {
        return $item <= 3;
    });

    $subset->all();

    // [4]

> {note} Якщо зворотний дзвінок ніколи не повернеться`true`,`skipWhile`метод поверне порожню колекцію.

<a name="method-slice"></a>

#### `slice()`{# collection-method}

`slice`метод повертає фрагмент колекції, починаючи з заданого індексу:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $slice = $collection->slice(4);

    $slice->all();

    // [5, 6, 7, 8, 9, 10]

Якщо ви хочете обмежити розмір повернутого фрагмента, передайте бажаний розмір як другий аргумент методу:

    $slice = $collection->slice(4, 2);

    $slice->all();

    // [5, 6]

Повернутий фрагмент збереже ключі за замовчуванням. Якщо ви не хочете зберігати оригінальні ключі, ви можете використовувати[`values`](#method-values)метод їх переіндексації.

<a name="method-some"></a>

#### `some()`{# collection-method}

Псевдонім для[`contains`](#method-contains)метод.

<a name="method-sort"></a>

#### `sort()`{# collection-method}

`sort`метод сортує колекцію. Відсортована колекція зберігає оригінальні ключі масиву, тому в цьому прикладі ми будемо використовувати[`values`](#method-values)метод скидання ключів до послідовно пронумерованих індексів:

    $collection = collect([5, 3, 1, 2, 4]);

    $sorted = $collection->sort();

    $sorted->values()->all();

    // [1, 2, 3, 4, 5]

Якщо ваші потреби у сортуванні є більш досконалими, ви можете передати зворотній дзвінок`sort`за власним алгоритмом. Зверніться до документації PHP на[`uasort`](https://secure.php.net/manual/en/function.uasort.php#refsect1-function.uasort-parameters), що є те, що колекція`sort`виклику методу під капотом.

> {tip} Якщо вам потрібно відсортувати колекцію вкладених масивів або об'єктів, див[`sortBy`](#method-sortby)і[`sortByDesc`](#method-sortbydesc)методи.

<a name="method-sortby"></a>

#### `sortBy()`{# collection-method}

`sortBy`метод сортує колекцію за заданим ключем. Відсортована колекція зберігає оригінальні ключі масиву, тому в цьому прикладі ми будемо використовувати[`values`](#method-values)метод скидання ключів до послідовно пронумерованих індексів:

    $collection = collect([
        ['name' => 'Desk', 'price' => 200],
        ['name' => 'Chair', 'price' => 100],
        ['name' => 'Bookcase', 'price' => 150],
    ]);

    $sorted = $collection->sortBy('price');

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'price' => 100],
            ['name' => 'Bookcase', 'price' => 150],
            ['name' => 'Desk', 'price' => 200],
        ]
    */

Цей метод приймає[сортувати прапори](https://www.php.net/manual/en/function.sort.php)як другий аргумент:

    $collection = collect([
        ['title' => 'Item 1'],
        ['title' => 'Item 12'],
        ['title' => 'Item 3'],
    ]);

    $sorted = $collection->sortBy('title', SORT_NATURAL);

    $sorted->values()->all();

    /*
        [
            ['title' => 'Item 1'],
            ['title' => 'Item 3'],
            ['title' => 'Item 12'],
        ]
    */

Крім того, ви можете передати власний зворотний виклик, щоб визначити, як сортувати значення колекції:

    $collection = collect([
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $sorted = $collection->sortBy(function ($product, $key) {
        return count($product['colors']);
    });

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'colors' => ['Black']],
            ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
            ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
        ]
    */

<a name="method-sortbydesc"></a>

#### `sortByDesc()`{# collection-method}

Цей метод має той самий підпис, що і[`sortBy`](#method-sortby)метод, але сортуватиме колекцію в зворотному порядку.

<a name="method-sortdesc"></a>

#### `sortDesc()`{# collection-method}

Цей метод сортує колекцію в протилежному порядку як[`sort`](#method-sort)метод:

    $collection = collect([5, 3, 1, 2, 4]);

    $sorted = $collection->sortDesc();

    $sorted->values()->all();

    // [5, 4, 3, 2, 1]

На відміну від`sort`, Ви не можете передати зворотній дзвінок`sortDesc`. Якщо ви хочете скористатися зворотним дзвінком, вам слід скористатися[`sort`](#method-sort)та оберніть своє порівняння.

<a name="method-sortkeys"></a>

#### `sortKeys()`{# collection-method}

`sortKeys`метод сортує колекцію за ключами базового асоціативного масиву:

    $collection = collect([
        'id' => 22345,
        'first' => 'John',
        'last' => 'Doe',
    ]);

    $sorted = $collection->sortKeys();

    $sorted->all();

    /*
        [
            'first' => 'John',
            'id' => 22345,
            'last' => 'Doe',
        ]
    */

<a name="method-sortkeysdesc"></a>

#### `sortKeysDesc()`{# collection-method}

Цей метод має той самий підпис, що і[`sortKeys`](#method-sortkeys)метод, але сортуватиме колекцію в зворотному порядку.

<a name="method-splice"></a>

#### `splice()`{# collection-method}

`splice`метод видаляє та повертає фрагмент елементів, починаючи з вказаного індексу:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2);

    $chunk->all();

    // [3, 4, 5]

    $collection->all();

    // [1, 2]

Ви можете передати другий аргумент, щоб обмежити розмір отриманого фрагмента:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 4, 5]

Крім того, ви можете передати третій аргумент, що містить нові елементи, щоб замінити вилучені з колекції елементи:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1, [10, 11]);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 10, 11, 4, 5]

<a name="method-split"></a>

#### `split()`{# collection-method}

`split`метод розбиває колекцію на задану кількість груп:

    $collection = collect([1, 2, 3, 4, 5]);

    $groups = $collection->split(3);

    $groups->all();

    // [[1, 2], [3, 4], [5]]

<a name="method-splitin"></a>

#### `splitIn()`{# collection-method}

`splitIn`метод розбиває колекцію на задану кількість груп, повністю заповнюючи нетермінальні групи, перш ніж розподілити залишок до кінцевої групи:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $groups = $collection->splitIn(3);

    $groups->all();

    // [[1, 2, 3, 4], [5, 6, 7, 8], [9, 10]]

<a name="method-sum"></a>

#### `sum()`{# collection-method}

`sum`метод повертає суму всіх предметів у колекції:

    collect([1, 2, 3, 4, 5])->sum();

    // 15

Якщо колекція містить вкладені масиви або об'єкти, вам слід передати ключ, який буде використовуватися для визначення значень, які потрібно підсумувати:

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->sum('pages');

    // 1272

Крім того, ви можете передати власний зворотний виклик, щоб визначити, які значення колекції підсумовувати:

    $collection = collect([
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $collection->sum(function ($product) {
        return count($product['colors']);
    });

    // 6

<a name="method-take"></a>

#### `take()`{# collection-method}

`take`метод повертає нову колекцію із зазначеною кількістю елементів:

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(3);

    $chunk->all();

    // [0, 1, 2]

Ви також можете передати від’ємне ціле число, щоб взяти вказану кількість предметів з кінця колекції:

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(-2);

    $chunk->all();

    // [4, 5]

<a name="method-takeuntil"></a>

#### `takeUntil()`{# collection-method}

`takeUntil`метод повертає елементи в колекції, поки не повернеться даний зворотний виклик`true`:

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->takeUntil(function ($item) {
        return $item >= 3;
    });

    $subset->all();

    // [1, 2]

Ви також можете передати просте значення в`takeUntil`спосіб отримати елементи, поки не буде знайдено задане значення:

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->takeUntil(3);

    $subset->all();

    // [1, 2]

> {note} Якщо заданого значення не знайдено або зворотний виклик ніколи не повернеться`true`,`takeUntil`метод поверне всі елементи колекції.

<a name="method-takewhile"></a>

#### `takeWhile()`{# collection-method}

`takeWhile`метод повертає елементи в колекції, поки не повернеться даний зворотний виклик`false`:

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->takeWhile(function ($item) {
        return $item < 3;
    });

    $subset->all();

    // [1, 2]

> {note} Якщо зворотний дзвінок ніколи не повернеться`false`,`takeWhile`метод поверне всі елементи колекції.

<a name="method-tap"></a>

#### `tap()`{# collection-method}

`tap`метод передає колекцію даному зворотному виклику, дозволяючи "натискати" на колекцію в певний момент і робити щось із елементами, не впливаючи на саму колекцію:

    collect([2, 4, 3, 1, 5])
        ->sort()
        ->tap(function ($collection) {
            Log::debug('Values after sorting', $collection->values()->all());
        })
        ->shift();

    // 1

<a name="method-times"></a>

#### `times()`{# collection-method}

Статичний`times`метод створює нову колекцію, викликаючи зворотний виклик задану кількість разів:

    $collection = Collection::times(10, function ($number) {
        return $number * 9;
    });

    $collection->all();

    // [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]

Цей метод може бути корисним у поєднанні з фабриками для створення[Красномовний](/docs/{{version}}/eloquent)моделі:

    $categories = Collection::times(3, function ($number) {
        return Category::factory()->create(['name' => "Category No. $number"]);
    });

    $categories->all();

    /*
        [
            ['id' => 1, 'name' => 'Category No. 1'],
            ['id' => 2, 'name' => 'Category No. 2'],
            ['id' => 3, 'name' => 'Category No. 3'],
        ]
    */

<a name="method-toarray"></a>

#### `toArray()`{# collection-method}

`toArray`метод перетворює колекцію на звичайний PHP`array`. Якщо значення колекції[Красномовний](/docs/{{version}}/eloquent)моделі, моделі також будуть перетворені в масиви:

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toArray();

    /*
        [
            ['name' => 'Desk', 'price' => 200],
        ]
    */

> {Примітка}`toArray`також перетворює всі вкладені об'єкти колекції, які є екземпляром`Arrayable`до масиву. Якщо ви хочете отримати необроблений базовий масив, використовуйте[`all`](#method-all)замість цього.

<a name="method-tojson"></a>

#### `toJson()`{# collection-method}

`toJson`метод перетворює колекцію в серіалізований рядок JSON:

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toJson();

    // '{"name":"Desk", "price":200}'

<a name="method-transform"></a>

#### `transform()`{# collection-method}

`transform`метод перебирає колекцію та викликає заданий зворотний виклик з кожним елементом у колекції. Елементи колекції будуть замінені значеннями, повернутими зворотним викликом:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->transform(function ($item, $key) {
        return $item * 2;
    });

    $collection->all();

    // [2, 4, 6, 8, 10]

> {note} На відміну від більшості інших методів збору,`transform`модифікує саму колекцію. Якщо замість цього ви хочете створити нову колекцію, використовуйте[`map`](#method-map)метод.

<a name="method-union"></a>

#### `union()`{# collection-method}

`union`метод додає даний масив до колекції. Якщо даний масив містить ключі, які вже є в оригінальній колекції, переважними будуть значення оригінальної колекції:

    $collection = collect([1 => ['a'], 2 => ['b']]);

    $union = $collection->union([3 => ['c'], 1 => ['b']]);

    $union->all();

    // [1 => ['a'], 2 => ['b'], 3 => ['c']]

<a name="method-unique"></a>

#### `unique()`{# collection-method}

`unique`метод повертає всі унікальні предмети в колекції. Повернена колекція зберігає оригінальні ключі масиву, тому в цьому прикладі ми будемо використовувати[`values`](#method-values)метод скидання ключів до послідовно пронумерованих індексів:

    $collection = collect([1, 1, 2, 2, 3, 4, 2]);

    $unique = $collection->unique();

    $unique->values()->all();

    // [1, 2, 3, 4]

Маючи справу з вкладеними масивами або об’єктами, ви можете вказати ключ, що використовується для визначення унікальності:

    $collection = collect([
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'iPhone 5', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
    ]);

    $unique = $collection->unique('brand');

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ]
    */

Ви також можете передати власний зворотний дзвінок, щоб визначити унікальність товару:

    $unique = $collection->unique(function ($item) {
        return $item['brand'].$item['type'];
    });

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
            ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
        ]
    */

`unique`метод використовує "вільні" порівняння при перевірці значень елементів, тобто рядок із цілим значенням вважатиметься рівним цілому числу того самого значення. Використовувати[`uniqueStrict`](#method-uniquestrict)метод фільтрації за допомогою "суворих" порівнянь.

> {tip} Поведінка цього методу змінюється під час використання[Красномовні колекції](/docs/{{version}}/eloquent-collections#method-unique).

<a name="method-uniquestrict"></a>

#### `uniqueStrict()`{# collection-method}

Цей метод має той самий підпис, що і[`unique`](#method-unique)метод; однак усі значення порівнюються за допомогою "суворих" порівнянь.

<a name="method-unless"></a>

#### `unless()`{# collection-method}

`unless`method виконає даний зворотний виклик, якщо перший аргумент, наданий методу, не має значення`true`:

    $collection = collect([1, 2, 3]);

    $collection->unless(true, function ($collection) {
        return $collection->push(4);
    });

    $collection->unless(false, function ($collection) {
        return $collection->push(5);
    });

    $collection->all();

    // [1, 2, 3, 5]

Для оберненого до`unless`, див[`when`](#method-when)метод.

<a name="method-unlessempty"></a>

#### `unlessEmpty()`{# collection-method}

Псевдонім для[`whenNotEmpty`](#method-whennotempty)метод.

<a name="method-unlessnotempty"></a>

#### `unlessNotEmpty()`{# collection-method}

Псевдонім для[`whenEmpty`](#method-whenempty)метод.

<a name="method-unwrap"></a>

#### `unwrap()`{# collection-method}

Статичний`unwrap`метод повертає основні елементи колекції із заданого значення, коли це застосовно:

    Collection::unwrap(collect('John Doe'));

    // ['John Doe']

    Collection::unwrap(['John Doe']);

    // ['John Doe']

    Collection::unwrap('John Doe');

    // 'John Doe'

<a name="method-values"></a>

#### `values()`{# collection-method}

`values`метод повертає нову колекцію із скиданням ключів до послідовних цілих чисел:

    $collection = collect([
        10 => ['product' => 'Desk', 'price' => 200],
        11 => ['product' => 'Desk', 'price' => 200],
    ]);

    $values = $collection->values();

    $values->all();

    /*
        [
            0 => ['product' => 'Desk', 'price' => 200],
            1 => ['product' => 'Desk', 'price' => 200],
        ]
    */

<a name="method-when"></a>

#### `when()`{# collection-method}

`when`method виконає даний зворотний виклик, коли перший аргумент, наданий методу, оцінюється як`true`:

    $collection = collect([1, 2, 3]);

    $collection->when(true, function ($collection) {
        return $collection->push(4);
    });

    $collection->when(false, function ($collection) {
        return $collection->push(5);
    });

    $collection->all();

    // [1, 2, 3, 4]

Для оберненого до`when`, див[`unless`](#method-unless)метод.

<a name="method-whenempty"></a>

#### `whenEmpty()`{# collection-method}

`whenEmpty`метод виконає даний зворотний виклик, коли колекція порожня:

    $collection = collect(['michael', 'tom']);

    $collection->whenEmpty(function ($collection) {
        return $collection->push('adam');
    });

    $collection->all();

    // ['michael', 'tom']


    $collection = collect();

    $collection->whenEmpty(function ($collection) {
        return $collection->push('adam');
    });

    $collection->all();

    // ['adam']


    $collection = collect(['michael', 'tom']);

    $collection->whenEmpty(function ($collection) {
        return $collection->push('adam');
    }, function ($collection) {
        return $collection->push('taylor');
    });

    $collection->all();

    // ['michael', 'tom', 'taylor']

Для оберненого до`whenEmpty`, див[`whenNotEmpty`](#method-whennotempty)метод.

<a name="method-whennotempty"></a>

#### `whenNotEmpty()`{# collection-method}

`whenNotEmpty`метод виконає даний зворотний виклик, коли колекція не порожня:

    $collection = collect(['michael', 'tom']);

    $collection->whenNotEmpty(function ($collection) {
        return $collection->push('adam');
    });

    $collection->all();

    // ['michael', 'tom', 'adam']


    $collection = collect();

    $collection->whenNotEmpty(function ($collection) {
        return $collection->push('adam');
    });

    $collection->all();

    // []


    $collection = collect();

    $collection->whenNotEmpty(function ($collection) {
        return $collection->push('adam');
    }, function ($collection) {
        return $collection->push('taylor');
    });

    $collection->all();

    // ['taylor']

Для оберненого до`whenNotEmpty`, див[`whenEmpty`](#method-whenempty)метод.

<a name="method-where"></a>

#### `where()`{# collection-method}

`where`метод фільтрує колекцію за заданою парою ключ / значення:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->where('price', 100);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */

`where`метод використовує "вільні" порівняння при перевірці значень елементів, тобто рядок із цілим значенням вважатиметься рівним цілому числу того самого значення. Використовувати[`whereStrict`](#method-wherestrict)метод фільтрації за допомогою "суворих" порівнянь.

За бажанням ви можете передати оператор порівняння як другий параметр.

    $collection = collect([
        ['name' => 'Jim', 'deleted_at' => '2019-01-01 00:00:00'],
        ['name' => 'Sally', 'deleted_at' => '2019-01-02 00:00:00'],
        ['name' => 'Sue', 'deleted_at' => null],
    ]);

    $filtered = $collection->where('deleted_at', '!=', null);

    $filtered->all();

    /*
        [
            ['name' => 'Jim', 'deleted_at' => '2019-01-01 00:00:00'],
            ['name' => 'Sally', 'deleted_at' => '2019-01-02 00:00:00'],
        ]
    */

<a name="method-wherestrict"></a>

#### `whereStrict()`{# collection-method}

Цей метод має той самий підпис, що і[`where`](#method-where)метод; однак усі значення порівнюються за допомогою "суворих" порівнянь.

<a name="method-wherebetween"></a>

#### `whereBetween()`{# collection-method}

`whereBetween`метод фільтрує колекцію в заданому діапазоні:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 80],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Pencil', 'price' => 30],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereBetween('price', [100, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Desk', 'price' => 200],
            ['product' => 'Bookcase', 'price' => 150],
            ['product' => 'Door', 'price' => 100],
        ]
    */

<a name="method-wherein"></a>

#### `whereIn()`{# collection-method}

`whereIn`метод фільтрує колекцію за заданим ключем / значенням, що містяться в даному масиві:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereIn('price', [150, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Desk', 'price' => 200],
            ['product' => 'Bookcase', 'price' => 150],
        ]
    */

`whereIn`метод використовує "вільні" порівняння при перевірці значень елементів, тобто рядок із цілим значенням вважатиметься рівним цілому числу того самого значення. Використовувати[`whereInStrict`](#method-whereinstrict)метод фільтрації за допомогою "суворих" порівнянь.

<a name="method-whereinstrict"></a>

#### `whereInStrict()`{# collection-method}

Цей метод має той самий підпис, що і[`whereIn`](#method-wherein)метод; однак усі значення порівнюються за допомогою "суворих" порівнянь.

<a name="method-whereinstanceof"></a>

#### `whereInstanceOf()`{# collection-method}

`whereInstanceOf`метод фільтрує колекцію за заданим типом класу:

    use App\Models\User;
    use App\Models\Post;

    $collection = collect([
        new User,
        new User,
        new Post,
    ]);

    $filtered = $collection->whereInstanceOf(User::class);

    $filtered->all();

    // [App\Models\User, App\Models\User]

<a name="method-wherenotbetween"></a>

#### `whereNotBetween()`{# collection-method}

`whereNotBetween`метод фільтрує колекцію в заданому діапазоні:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 80],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Pencil', 'price' => 30],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereNotBetween('price', [100, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 80],
            ['product' => 'Pencil', 'price' => 30],
        ]
    */

<a name="method-wherenotin"></a>

#### `whereNotIn()`{# collection-method}

`whereNotIn`метод фільтрує колекцію за заданим ключем / значенням, що не містяться в даному масиві:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereNotIn('price', [150, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */

`whereNotIn`метод використовує "вільні" порівняння при перевірці значень елементів, тобто рядок із цілим значенням вважатиметься рівним цілому числу того самого значення. Використовувати[`whereNotInStrict`](#method-wherenotinstrict)метод фільтрації за допомогою "суворих" порівнянь.

<a name="method-wherenotinstrict"></a>

#### `whereNotInStrict()`{# collection-method}

Цей метод має той самий підпис, що і[`whereNotIn`](#method-wherenotin)метод; однак усі значення порівнюються за допомогою "суворих" порівнянь.

<a name="method-wherenotnull"></a>

#### `whereNotNull()`{# collection-method}

`whereNotNull`метод фільтрує елементи, де даний ключ не має значення null:

    $collection = collect([
        ['name' => 'Desk'],
        ['name' => null],
        ['name' => 'Bookcase'],
    ]);

    $filtered = $collection->whereNotNull('name');

    $filtered->all();

    /*
        [
            ['name' => 'Desk'],
            ['name' => 'Bookcase'],
        ]
    */

<a name="method-wherenull"></a>

#### `whereNull()`{# collection-method}

`whereNull`метод фільтрує елементи, де даний ключ має значення null:

    $collection = collect([
        ['name' => 'Desk'],
        ['name' => null],
        ['name' => 'Bookcase'],
    ]);

    $filtered = $collection->whereNull('name');

    $filtered->all();

    /*
        [
            ['name' => null],
        ]
    */

<a name="method-wrap"></a>

#### `wrap()`{# collection-method}

Статичний`wrap`метод обертає задане значення у колекції, коли це застосовно:

    $collection = Collection::wrap('John Doe');

    $collection->all();

    // ['John Doe']

    $collection = Collection::wrap(['John Doe']);

    $collection->all();

    // ['John Doe']

    $collection = Collection::wrap(collect('John Doe'));

    $collection->all();

    // ['John Doe']

<a name="method-zip"></a>

#### `zip()`{# collection-method}

`zip`метод об'єднує значення даного масиву зі значеннями вихідної колекції за відповідним індексом:

    $collection = collect(['Chair', 'Desk']);

    $zipped = $collection->zip([100, 200]);

    $zipped->all();

    // [['Chair', 100], ['Desk', 200]]

<a name="higher-order-messages"></a>

## Повідомлення вищого порядку

Колекції також забезпечують підтримку "повідомлень вищого порядку", які є комбінаціями клавіш для виконання загальних дій над колекціями. Методами збору, які надають повідомлення вищого порядку, є:[`average`](#method-average),[`avg`](#method-avg),[`contains`](#method-contains),[`each`](#method-each),[`every`](#method-every),[`filter`](#method-filter),[`first`](#method-first),[`flatMap`](#method-flatmap),[`groupBy`](#method-groupby), [`keyBy`](#method-keyby),[`map`](#method-map),[`max`](#method-max),[`min`](#method-min),[`partition`](#method-partition),[`reject`](#method-reject),[`skipUntil`](#method-skipuntil),[`skipWhile`](#method-skipwhile),[`some`](#method-some),[`sortBy`](#method-sortby),[`sortByDesc`](#method-sortbydesc),[`sum`](#method-sum),[`takeUntil`](#method-takeuntil),[`takeWhile`](#method-takewhile)і[`unique`](#method-unique).

Кожне повідомлення вищого порядку можна отримати як динамічну властивість на екземплярі колекції. Наприклад, давайте використаємо`each`повідомлення вищого порядку для виклику методу для кожного об’єкта в колекції:

    $users = User::where('votes', '>', 500)->get();

    $users->each->markAsVip();

Так само ми можемо використовувати`sum`повідомлення вищого порядку, щоб зібрати загальну кількість "голосів" для набору користувачів:

    $users = User::where('group', 'Development')->get();

    return $users->sum->votes;

<a name="lazy-collections"></a>

## Ледачі колекції

<a name="lazy-collection-introduction"></a>

### Вступ

> {note} Перш ніж дізнатись більше про ледачі колекції Laravel, знайдіть трохи часу, щоб ознайомитись із ними[PHP-генератори](https://www.php.net/manual/en/language.generators.overview.php).

Доповнити і без того потужний`Collection`клас,`LazyCollection`клас використовує PHP[генератори](https://www.php.net/manual/en/language.generators.overview.php)щоб дозволити вам працювати з дуже великими наборами даних, зберігаючи при цьому низьке використання пам'яті.

Наприклад, уявіть, що вашій програмі потрібно обробити файл журналу на декілька гігабайт, користуючись методами збору Laravel для аналізу журналів. Замість зчитування всього файлу в пам’ять одночасно, ліниві колекції можуть використовуватися для збереження лише невеликої частини файлу в пам’яті на даний момент:

    use App\Models\LogEntry;
    use Illuminate\Support\LazyCollection;

    LazyCollection::make(function () {
        $handle = fopen('log.txt', 'r');

        while (($line = fgets($handle)) !== false) {
            yield $line;
        }
    })->chunk(4)->map(function ($lines) {
        return LogEntry::fromLines($lines);
    })->each(function (LogEntry $logEntry) {
        // Process the log entry...
    });

Або уявіть, що вам потрібно переглядати 10 000 Eloquent моделей. При використанні традиційних колекцій Laravel усі 10000 Eloquent моделей повинні завантажуватися в пам’ять одночасно:

    $users = App\Models\User::all()->filter(function ($user) {
        return $user->id > 500;
    });

Однак запит розробника`cursor`метод повертає a`LazyCollection`інстанції. Це дозволяє виконувати лише один запит до бази даних, але одночасно зберігати в пам'яті лише одну модель Eloquent. У цьому прикладі`filter`зворотний виклик не виконується, поки ми фактично не повторимо кожного користувача окремо, що дозволить різко зменшити використання пам'яті:

    $users = App\Models\User::cursor()->filter(function ($user) {
        return $user->id > 500;
    });

    foreach ($users as $user) {
        echo $user->id;
    }

<a name="creating-lazy-collections"></a>

### Створення ледачих колекцій

Щоб створити ледачий екземпляр колекції, вам слід передати функцію генератора PHP колекції`make`метод:

    use Illuminate\Support\LazyCollection;

    LazyCollection::make(function () {
        $handle = fopen('log.txt', 'r');

        while (($line = fgets($handle)) !== false) {
            yield $line;
        }
    });

<a name="the-enumerable-contract"></a>

### Контракт, що перелічується

Майже всі методи, доступні на`Collection`клас також доступні на`LazyCollection`клас. Обидва ці класи реалізують`Illuminate\Support\Enumerable`контракт, який визначає такі методи:

<div id="collection-method-list" markdown="1">

[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[chunk](#method-chunk)
[chunkWhile](#method-chunkwhile)
[collapse](#method-collapse)
[collect](#method-collect)
[combine](#method-combine)
[concat](#method-concat)
[contains](#method-contains)
[containsStrict](#method-containsstrict)
[count](#method-count)
[countBy](#method-countBy)
[crossJoin](#method-crossjoin)
[dd](#method-dd)
[diff](#method-diff)
[diffAssoc](#method-diffassoc)
[diffKeys](#method-diffkeys)
[dump](#method-dump)
[duplicates](#method-duplicates)
[duplicatesStrict](#method-duplicatesstrict)
[each](#method-each)
[eachSpread](#method-eachspread)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[firstWhere](#method-first-where)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[implode](#method-implode)
[intersect](#method-intersect)
[intersectByKeys](#method-intersectbykeys)
[isEmpty](#method-isempty)
[isNotEmpty](#method-isnotempty)
[join](#method-join)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[macro](#method-macro)
[make](#method-make)
[map](#method-map)
[mapInto](#method-mapinto)
[mapSpread](#method-mapspread)
[mapToGroups](#method-maptogroups)
[mapWithKeys](#method-mapwithkeys)
[max](#method-max)
[median](#method-median)
[merge](#method-merge)
[mergeRecursive](#method-mergerecursive)
[min](#method-min)
[mode](#method-mode)
[nth](#method-nth)
[only](#method-only)
[pad](#method-pad)
[partition](#method-partition)
[pipe](#method-pipe)
[pluck](#method-pluck)
[random](#method-random)
[reduce](#method-reduce)
[reject](#method-reject)
[replace](#method-replace)
[replaceRecursive](#method-replacerecursive)
[reverse](#method-reverse)
[search](#method-search)
[shuffle](#method-shuffle)
[skip](#method-skip)
[slice](#method-slice)
[some](#method-some)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[sortKeys](#method-sortkeys)
[sortKeysDesc](#method-sortkeysdesc)
[split](#method-split)
[sum](#method-sum)
[take](#method-take)
[tap](#method-tap)
[times](#method-times)
[toArray](#method-toarray)
[toJson](#method-tojson)
[union](#method-union)
[unique](#method-unique)
[uniqueStrict](#method-uniquestrict)
[unless](#method-unless)
[unlessEmpty](#method-unlessempty)
[unlessNotEmpty](#method-unlessnotempty)
[unwrap](#method-unwrap)
[values](#method-values)
[when](#method-when)
[whenEmpty](#method-whenempty)
[whenNotEmpty](#method-whennotempty)
[where](#method-where)
[whereStrict](#method-wherestrict)
[whereBetween](#method-wherebetween)
[whereIn](#method-wherein)
[whereInStrict](#method-whereinstrict)
[whereInstanceOf](#method-whereinstanceof)
[whereNotBetween](#method-wherenotbetween)
[whereNotIn](#method-wherenotin)
[whereNotInStrict](#method-wherenotinstrict)
[wrap](#method-wrap)
[zip](#method-zip)

</div>

> {note} Методи, що мутують колекцію (наприклад`shift`,`pop`,`prepend`тощо)_ні_доступні на`LazyCollection`клас.

<a name="lazy-collection-methods"></a>

### Методи лінивого збору

На додаток до методів, визначених у`Enumerable`контракт,`LazyCollection`клас містить такі методи:

<a name="method-tapEach"></a>

#### `tapEach()`{# collection-method}

Поки`each`метод викликає даний зворотний виклик для кожного елемента в колекції відразу,`tapEach`метод викликає лише даний зворотний виклик, оскільки елементи витягуються зі списку по одному:

    $lazyCollection = LazyCollection::times(INF)->tapEach(function ($value) {
        dump($value);
    });

    // Nothing has been dumped so far...

    $array = $lazyCollection->take(3)->all();

    // 1
    // 2
    // 3

<a name="method-remember"></a>

#### `remember()`{# collection-method}

`remember`Метод повертає нову ліниву колекцію, яка запам'ятає всі перераховані значення і не отримає їх знову, коли колекція буде перерахована знову:

    $users = User::cursor()->remember();

    // No query has been executed yet...

    $users->take(5)->all();

    // The query has been executed and the first 5 users have been hydrated from the database...

    $users->take(20)->all();

    // First 5 users come from the collection's cache... The rest are hydrated from the database...
