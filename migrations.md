# База даних: Міграції

-   [Вступ](#introduction)
-   [Generating Migrations](#generating-migrations)
    -   [Міграція, що стискає](#squashing-migrations)
-   [Структура міграції](#migration-structure)
-   [Запуск міграцій](#running-migrations)
    -   [Відкочування міграцій](#rolling-back-migrations)
-   [Столи](#tables)
    -   [Створення таблиць](#creating-tables)
    -   [Перейменування / скидання таблиць](#renaming-and-dropping-tables)
-   [Стовпці](#columns)
    -   [Створення стовпців](#creating-columns)
    -   [Модифікатори стовпців](#column-modifiers)
    -   [Змінення стовпців](#modifying-columns)
    -   [Скидання стовпців](#dropping-columns)
-   [Покажчики](#indexes)
    -   [Створення індексів](#creating-indexes)
    -   [Перейменування покажчиків](#renaming-indexes)
    -   [Падіння індексів](#dropping-indexes)
    -   [Зовнішні ключові обмеження](#foreign-key-constraints)

<a name="introduction"></a>

## Вступ

Міграції - це як контроль версій для вашої бази даних, що дозволяє вашій команді змінювати та надавати спільний доступ до схеми бази даних програми. Міграції, як правило, поєднуються з конструктором схем Laravel для побудови схеми бази даних вашої програми. Якщо вам коли-небудь доводилося казати товаришу по команді вручну додати стовпець до їх локальної схеми бази даних, ви зіткнулися з проблемою, яку вирішує міграція бази даних.

Ларавель`Schema`[фасад](/docs/{{version}}/facades)надає агностичну підтримку баз даних для створення та обробки таблиць у всіх підтримуваних системах баз даних Laravel.

<a name="generating-migrations"></a>

## Генерування міграцій

Щоб створити міграцію, використовуйте`make:migration`[Artisan командування](/docs/{{version}}/artisan):

    php artisan make:migration create_users_table

Нова міграція буде розміщена у вашому`database/migrations`каталог. Кожне ім'я файлу міграції містить мітку часу, що дозволяє Laravel визначати порядок міграцій.

> {tip} Заглушки міграції можна налаштувати за допомогою[заглушка видавництва](/docs/{{version}}/artisan#stub-customization)

`--table`і`--create`Параметри також можуть використовуватися для вказівки назви таблиці та того, чи буде міграція створювати нову таблицю. Ці параметри попередньо заповнюють згенерований файл заглушки вказаною таблицею:

    php artisan make:migration create_users_table --create=users

    php artisan make:migration add_votes_to_users_table --table=users

Якщо ви хочете вказати власний вихідний шлях для згенерованої міграції, ви можете використовувати`--path`параметр при виконанні`make:migration`команди. Вказаний шлях повинен бути відносно базового шляху вашої програми.

<a name="squashing-migrations"></a>

### Міграція, що стискає

Створюючи додаток, з часом ви можете накопичувати все більше і більше міграцій. Це може призвести до того, що ваш каталог міграції роздується потенційно сотнями міграцій. Якщо ви хочете, ви можете "зім'яти" свої міграції в один файл SQL. Для початку виконайте`schema:dump`команда:

    php artisan schema:dump

    // Dump the current database schema and prune all existing migrations...
    php artisan schema:dump --prune

Коли ви виконуєте цю команду, Laravel напише файл "schema" на ваш`database/schema`каталог. Тепер, коли ви намагаєтесь перенести вашу базу даних і жодної іншої міграції не було виконано, Laravel спочатку виконає SQL-файл файлу схеми. Після виконання команд файлу схеми Laravel виконає всі залишені міграції, які не були частиною дампа схеми.

Вам слід закріпити файл схеми бази даних для керування джерелом, щоб інші нові розробники у вашій команді могли швидко створити початкову структуру бази даних вашого додатка.

> {note} Зменшення міграції доступне лише для баз даних MySQL, PostgreSQL та SQLite. Однак дампи бази даних не можуть бути відновлені до баз даних SQLite в пам'яті.

<a name="migration-structure"></a>

## Структура міграції

Клас міграції містить два методи:`up`і`down`.`up`метод використовується для додавання нових таблиць, стовпців або індексів до вашої бази даних, тоді як`down`метод повинен змінити операції, що виконуються`up`метод.

В обох цих методах ви можете використовувати конструктор схем Laravel для виразного створення та модифікації таблиць. Щоб дізнатись про всі методи, доступні на`Schema`будівельник,[ознайомтесь з його документацією](#creating-tables). Наприклад, наступна міграція створює файл`flights`стіл:

    <?php

    use Illuminate\Database\Migrations\Migration;
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    class CreateFlightsTable extends Migration
    {
        /**
         * Run the migrations.
         *
         * @return void
         */
        public function up()
        {
            Schema::create('flights', function (Blueprint $table) {
                $table->id();
                $table->string('name');
                $table->string('airline');
                $table->timestamps();
            });
        }

        /**
         * Reverse the migrations.
         *
         * @return void
         */
        public function down()
        {
            Schema::drop('flights');
        }
    }

<a name="running-migrations"></a>

## Запуск міграцій

Щоб запустити всі ваші видатні міграції, запустіть`migrate`Команда ремісників:

    php artisan migrate

> {note} Якщо ви використовуєте[Віртуальна машина Homestead](/docs/{{version}}/homestead), вам слід запустити цю команду з віртуальної машини.

<a name="forcing-migrations-to-run-in-production"></a>

#### Примушування міграцій до виробництва

Деякі операції міграції є руйнівними, а це означає, що вони можуть призвести до втрати даних. Щоб захистити вас від запуску цих команд проти вашої виробничої бази даних, перед виконанням команд вам буде запропоновано підтвердити. Щоб змусити команди запускатись без підказки, використовуйте`--force`прапор:

    php artisan migrate --force

<a name="rolling-back-migrations"></a>

### Відкочування міграцій

Щоб повернути останню операцію міграції, ви можете використовувати`rollback`команди. Ця команда відкочує останню "партію" міграцій, яка може включати кілька файлів міграції:

    php artisan migrate:rollback

Ви можете відкотити обмежену кількість міграцій, надавши`step`варіант для`rollback`команди. Наприклад, наступна команда відкотить останні п’ять міграцій:

    php artisan migrate:rollback --step=5

`migrate:reset`команда відкотить усі міграції програми:

    php artisan migrate:reset

<a name="roll-back-migrate-using-a-single-command"></a>

#### Відкат і перехід за допомогою однієї команди

`migrate:refresh`команда відкотить усі ваші міграції, а потім виконає`migrate`команди. Ця команда ефективно відтворює всю вашу базу даних:

    php artisan migrate:refresh

    // Refresh the database and run all database seeds...
    php artisan migrate:refresh --seed

Ви можете відкотитись і повторно перенести обмежену кількість міграцій, надавши`step`варіант для`refresh`команди. Наприклад, наступна команда відкотить і повторно мігрує останні п’ять міграцій:

    php artisan migrate:refresh --step=5

<a name="drop-all-tables-migrate"></a>

#### Відкиньте всі таблиці та перенесіть

`migrate:fresh`команда видалить усі таблиці з бази даних, а потім виконає`migrate`команда:

    php artisan migrate:fresh

    php artisan migrate:fresh --seed

> {примітка}`migrate:fresh`команда скине всі таблиці бази даних незалежно від їх префікса. Цю команду слід застосовувати з обережністю при розробці бази даних, спільно використовуваної з іншими програмами.

<a name="tables"></a>

## Столи

<a name="creating-tables"></a>

### Створення таблиць

Щоб створити нову таблицю бази даних, використовуйте`create`метод на`Schema`фасад.`create`метод приймає два аргументи: перший - це назва таблиці, а другий - a`Closure`який отримує a`Blueprint`об'єкт, який може бути використаний для визначення нової таблиці:

    Schema::create('users', function (Blueprint $table) {
        $table->id();
    });

Створюючи таблицю, ви можете використовувати будь-яку з конструкторів схем[колонкові методи](#creating-columns)для визначення стовпців таблиці.

<a name="checking-for-table-column-existence"></a>

#### Перевірка наявності таблиці / стовпця

Ви можете перевірити наявність таблиці або стовпця за допомогою`hasTable`і`hasColumn`методи:

    if (Schema::hasTable('users')) {
        //
    }

    if (Schema::hasColumn('users', 'email')) {
        //
    }

<a name="database-connection-table-options"></a>

#### Параметри підключення до бази даних та таблиці

Якщо ви хочете виконати операцію схеми з підключенням до бази даних, яке не є вашим з'єднанням за замовчуванням, використовуйте`connection`метод:

    Schema::connection('foo')->create('users', function (Blueprint $table) {
        $table->id();
    });

Ви можете використовувати такі команди у конструкторі схем для визначення параметрів таблиці:

| Команда                                     | Опис                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| `$table->engine = 'InnoDB';`                | Вкажіть механізм зберігання таблиць (MySQL).                 |
| `$table->charset = 'utf8mb4';`              | Вкажіть набір символів за замовчуванням для таблиці (MySQL). |
| `$table->collation = 'utf8mb4_unicode_ci';` | Вкажіть порівняння за замовчуванням для таблиці (MySQL).     |
| `$table->temporary();`                      | Створіть тимчасову таблицю (крім SQL Server).                |

<a name="renaming-and-dropping-tables"></a>

### Перейменування / скидання таблиць

Щоб перейменувати існуючу таблицю бази даних, використовуйте`rename`метод:

    Schema::rename($from, $to);

Щоб скинути існуючу таблицю, ви можете використовувати`drop`або`dropIfExists`методи:

    Schema::drop('users');

    Schema::dropIfExists('users');

<a name="renaming-tables-with-foreign-keys"></a>

#### Перейменування таблиць за допомогою іноземних ключів

Перш ніж перейменовувати таблицю, слід переконатися, що будь-які обмеження зовнішнього ключа в таблиці мають явне ім'я у файлах міграції, замість того, щоб дозволяти Laravel присвоювати ім'я на основі конвенції. В іншому випадку ім’я обмеження зовнішнього ключа буде посилатися на старе ім’я таблиці.

<a name="columns"></a>

## Стовпці

<a name="creating-columns"></a>

### Створення стовпців

`table`метод на`Schema`фасад може використовуватися для оновлення існуючих таблиць. Подобається`create`метод,`table`метод приймає два аргументи: ім'я таблиці та a`Closure`що отримує a`Blueprint`екземпляр, який ви можете використовувати для додавання стовпців до таблиці:

    Schema::table('users', function (Blueprint $table) {
        $table->string('email');
    });

<a name="available-column-types"></a>

#### Доступні типи стовпців

Command  |  Description
-------  |  -----------
`$table->id();`  |  Alias of `$table->bigIncrements('id')`.
`$table->foreignId('user_id');`  |  Alias of `$table->unsignedBigInteger('user_id')`.
`$table->bigIncrements('id');`  |  Auto-incrementing UNSIGNED BIGINT (primary key) equivalent column.
`$table->bigInteger('votes');`  |  BIGINT equivalent column.
`$table->binary('data');`  |  BLOB equivalent column.
`$table->boolean('confirmed');`  |  BOOLEAN equivalent column.
`$table->char('name', 100);`  |  CHAR equivalent column with a length.
`$table->date('created_at');`  |  DATE equivalent column.
`$table->dateTime('created_at', 0);`  |  DATETIME equivalent column with precision (total digits).
`$table->dateTimeTz('created_at', 0);`  |  DATETIME (with timezone) equivalent column with precision (total digits).
`$table->decimal('amount', 8, 2);`  |  DECIMAL equivalent column with precision (total digits) and scale (decimal digits).
`$table->double('amount', 8, 2);`  |  DOUBLE equivalent column with precision (total digits) and scale (decimal digits).
`$table->enum('level', ['easy', 'hard']);`  |  ENUM equivalent column.
`$table->float('amount', 8, 2);`  |  FLOAT equivalent column with a precision (total digits) and scale (decimal digits).
`$table->geometry('positions');`  |  GEOMETRY equivalent column.
`$table->geometryCollection('positions');`  |  GEOMETRYCOLLECTION equivalent column.
`$table->increments('id');`  |  Auto-incrementing UNSIGNED INTEGER (primary key) equivalent column.
`$table->integer('votes');`  |  INTEGER equivalent column.
`$table->ipAddress('visitor');`  |  IP address equivalent column.
`$table->json('options');`  |  JSON equivalent column.
`$table->jsonb('options');`  |  JSONB equivalent column.
`$table->lineString('positions');`  |  LINESTRING equivalent column.
`$table->longText('description');`  |  LONGTEXT equivalent column.
`$table->macAddress('device');`  |  MAC address equivalent column.
`$table->mediumIncrements('id');`  |  Auto-incrementing UNSIGNED MEDIUMINT (primary key) equivalent column.
`$table->mediumInteger('votes');`  |  MEDIUMINT equivalent column.
`$table->mediumText('description');`  |  MEDIUMTEXT equivalent column.
`$table->morphs('taggable');`  |  Adds `taggable_id` UNSIGNED BIGINT and `taggable_type` VARCHAR equivalent columns.
`$table->uuidMorphs('taggable');`  |  Adds `taggable_id` CHAR(36) and `taggable_type` VARCHAR(255) UUID equivalent columns.
`$table->multiLineString('positions');`  |  MULTILINESTRING equivalent column.
`$table->multiPoint('positions');`  |  MULTIPOINT equivalent column.
`$table->multiPolygon('positions');`  |  MULTIPOLYGON equivalent column.
`$table->nullableMorphs('taggable');`  |  Adds nullable versions of `morphs()` columns.
`$table->nullableUuidMorphs('taggable');`  |  Adds nullable versions of `uuidMorphs()` columns.
`$table->nullableTimestamps(0);`  |  Alias of `timestamps()` method.
`$table->point('position');`  |  POINT equivalent column.
`$table->polygon('positions');`  |  POLYGON equivalent column.
`$table->rememberToken();`  |  Adds a nullable `remember_token` VARCHAR(100) equivalent column.
`$table->set('flavors', ['strawberry', 'vanilla']);`  |  SET equivalent column.
`$table->smallIncrements('id');`  |  Auto-incrementing UNSIGNED SMALLINT (primary key) equivalent column.
`$table->smallInteger('votes');`  |  SMALLINT equivalent column.
`$table->softDeletes('deleted_at', 0);`  |  Adds a nullable `deleted_at` TIMESTAMP equivalent column for soft deletes with precision (total digits).
`$table->softDeletesTz('deleted_at', 0);`  |  Adds a nullable `deleted_at` TIMESTAMP (with timezone) equivalent column for soft deletes with precision (total digits).
`$table->string('name', 100);`  |  VARCHAR equivalent column with a length.
`$table->text('description');`  |  TEXT equivalent column.
`$table->time('sunrise', 0);`  |  TIME equivalent column with precision (total digits).
`$table->timeTz('sunrise', 0);`  |  TIME (with timezone) equivalent column with precision (total digits).
`$table->timestamp('added_on', 0);`  |  TIMESTAMP equivalent column with precision (total digits).
`$table->timestampTz('added_on', 0);`  |  TIMESTAMP (with timezone) equivalent column with precision (total digits).
`$table->timestamps(0);`  |  Adds nullable `created_at` and `updated_at` TIMESTAMP equivalent columns with precision (total digits).
`$table->timestampsTz(0);`  |  Adds nullable `created_at` and `updated_at` TIMESTAMP (with timezone) equivalent columns with precision (total digits).
`$table->tinyIncrements('id');`  |  Auto-incrementing UNSIGNED TINYINT (primary key) equivalent column.
`$table->tinyInteger('votes');`  |  TINYINT equivalent column.
`$table->unsignedBigInteger('votes');`  |  UNSIGNED BIGINT equivalent column.
`$table->unsignedDecimal('amount', 8, 2);`  |  UNSIGNED DECIMAL equivalent column with a precision (total digits) and scale (decimal digits).
`$table->unsignedInteger('votes');`  |  UNSIGNED INTEGER equivalent column.
`$table->unsignedMediumInteger('votes');`  |  UNSIGNED MEDIUMINT equivalent column.
`$table->unsignedSmallInteger('votes');`  |  UNSIGNED SMALLINT equivalent column.
`$table->unsignedTinyInteger('votes');`  |  UNSIGNED TINYINT equivalent column.
`$table->uuid('id');`  |  UUID equivalent column.
`$table->year('birth_year');`  |  YEAR equivalent column.

<a name="column-modifiers"></a>

### Модифікатори стовпців

На додаток до перерахованих вище типів стовпців, є кілька "модифікаторів" стовпців, які ви можете використовувати під час додавання стовпця до таблиці бази даних. Наприклад, щоб зробити стовпець "нульовим", ви можете використовувати`nullable`метод:

    Schema::table('users', function (Blueprint $table) {
        $table->string('email')->nullable();
    });

Наступний список містить усі доступні модифікатори стовпців. Цей список не включає[модифікатори індексу](#creating-indexes):

Modifier  |  Description
--------  |  -----------
`->after('column')`  |  Place the column "after" another column (MySQL)
`->autoIncrement()`  |  Set INTEGER columns as auto-increment (primary key)
`->charset('utf8mb4')`  |  Specify a character set for the column (MySQL)
`->collation('utf8mb4_unicode_ci')`  |  Specify a collation for the column (MySQL/PostgreSQL/SQL Server)
`->comment('my comment')`  |  Add a comment to a column (MySQL/PostgreSQL)
`->default($value)`  |  Specify a "default" value for the column
`->first()`  |  Place the column "first" in the table (MySQL)
`->from($integer)`  |  Set the starting value of an auto-incrementing field (MySQL / PostgreSQL)
`->nullable($value = true)`  |  Allows (by default) NULL values to be inserted into the column
`->storedAs($expression)`  |  Create a stored generated column (MySQL)
`->unsigned()`  |  Set INTEGER columns as UNSIGNED (MySQL)
`->useCurrent()`  |  Set TIMESTAMP columns to use CURRENT_TIMESTAMP as default value
`->useCurrentOnUpdate()`  |  Set TIMESTAMP columns to use CURRENT_TIMESTAMP when a record is updated
`->virtualAs($expression)`  |  Create a virtual generated column (MySQL)
`->generatedAs($expression)`  |  Create an identity column with specified sequence options (PostgreSQL)
`->always()`  |  Defines the precedence of sequence values over input for an identity column (PostgreSQL)

<a name="default-expressions"></a>

#### Вирази за замовчуванням

`default`модифікатор приймає значення або`\Illuminate\Database\Query\Expression`інстанції. Використання`Expression`екземпляр запобіжить загортанню значення у лапки та дозволить використовувати спеціальні функції бази даних. Одна ситуація, коли це особливо корисно, це коли вам потрібно призначити значення за замовчуванням стовпцям JSON:

    <?php

    use Illuminate\Support\Facades\Schema;
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Query\Expression;
    use Illuminate\Database\Migrations\Migration;

    class CreateFlightsTable extends Migration
    {
        /**
         * Run the migrations.
         *
         * @return void
         */
        public function up()
        {
            Schema::create('flights', function (Blueprint $table) {
                $table->id();
                $table->json('movies')->default(new Expression('(JSON_ARRAY())'));
                $table->timestamps();
            });
        }
    }

> {note} Підтримка виразів за замовчуванням залежить від драйвера бази даних, версії бази даних та типу поля. Будь ласка, зверніться до відповідної документації щодо сумісності. Також зауважте, що використання певних функцій бази даних може щільно поєднати вас із певним драйвером.

<a name="modifying-columns"></a>

### Змінення стовпців

<a name="prerequisites"></a>

#### Передумови

Перш ніж змінювати стовпець, обов’язково додайте`doctrine/dbal`залежність від вашого`composer.json`файл. Бібліотека DBAL Doctrine використовується для визначення поточного стану стовпця та створення запитів SQL, необхідних для внесення необхідних коригувань:

    composer require doctrine/dbal

<a name="updating-column-attributes"></a>

#### Оновлення атрибутів стовпців

`change`метод дозволяє змінювати тип та атрибути існуючих стовпців. Наприклад, ви можете збільшити розмір a`string`стовпець. Щоб побачити`change`в дії, давайте збільшимо розмір`name` column from 25 to 50:

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->change();
    });

Ми також можемо змінити стовпець, щоб його можна було опустити:

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->nullable()->change();
    });

> {note} Можна змінити лише такі типи стовпців: bigInteger, двійкові, логічні, дата, dateTime, dateTimeTz, десяткове, ціле число, json, longText, mediumText, smallInteger, рядок, текст, час, unsignedBigInteger, unsignedInteger, unsignedSmallInteger і uuid.

<a name="renaming-columns"></a>

#### Перейменування стовпців

Щоб перейменувати стовпець, ви можете використовувати`renameColumn`у конструкторі схем. Перш ніж перейменовувати стовпець, обов’язково додайте`doctrine/dbal`залежність від вашого`composer.json`файл:

    Schema::table('users', function (Blueprint $table) {
        $table->renameColumn('from', 'to');
    });

> {note} Перейменування`enum`На даний момент стовпець не підтримується.

<a name="dropping-columns"></a>

### Скидання стовпців

Щоб скинути стовпець, використовуйте`dropColumn`у конструкторі схем. Перш ніж скидати стовпці з бази даних SQLite, вам потрібно додати файл`doctrine/dbal`залежність від вашого`composer.json`файл і запустіть`composer update`команда у вашому терміналі для встановлення бібліотеки:

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('votes');
    });

Ви можете скинути кілька таблиць із таблиці, передавши масив імен стовпців до`dropColumn`метод:

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn(['votes', 'avatar', 'location']);
    });

> {note} Видалення або зміна кількох стовпців у межах однієї міграції під час використання бази даних SQLite не підтримується.

<a name="available-command-aliases"></a>

#### Доступні псевдоніми команд

Command  |  Description
-------  |  -----------
`$table->dropMorphs('morphable');`  |  Drop the `morphable_id` and `morphable_type` columns.
`$table->dropRememberToken();`  |  Drop the `remember_token` column.
`$table->dropSoftDeletes();`  |  Drop the `deleted_at` column.
`$table->dropSoftDeletesTz();`  |  Alias of `dropSoftDeletes()` method.
`$table->dropTimestamps();`  |  Drop the `created_at` and `updated_at` columns.
`$table->dropTimestampsTz();` |  Alias of `dropTimestamps()` method.

<a name="indexes"></a>

## Indexes

<a name="creating-indexes"></a>

### Створення індексів

Конструктор схем Laravel підтримує кілька типів індексів. Наступний приклад створює новий`email`і вказує, що його значення повинні бути унікальними. Для створення індексу ми можемо встановити ланцюжок`unique`метод до визначення стовпця:

    $table->string('email')->unique();

Крім того, ви можете створити індекс після визначення стовпця. Наприклад:

    $table->unique('email');

Ви навіть можете передати масив стовпців методу індексу для створення складеного (або складеного) індексу:

    $table->index(['account_id', 'created_at']);

Laravel автоматично генерує ім'я індексу на основі таблиці, імен стовпців та типу індексу, але ви можете передати другий аргумент методу, щоб вказати ім'я індексу самостійно:

    $table->unique('email', 'unique_email');

<a name="available-index-types"></a>

#### Доступні типи індексів

Кожен метод індексу приймає необов’язковий другий аргумент, щоб вказати ім’я індексу. Якщо його не вказати, ім’я буде отримано з назв таблиці та стовпців, що використовуються для індексу, а також типу індексу.

| Команда                                 | Опис                                    |
| --------------------------------------- | --------------------------------------- |
| `$table->primary('id');`                | Додає первинний ключ.                   |
| `$table->primary(['id', 'parent_id']);` | Додає складені клавіші.                 |
| `$table->unique('email');`              | Додає унікальний індекс.                |
| `$table->index('state');`               | Додає звичайний індекс.                 |
| `$table->spatialIndex('location');`     | Додає просторовий індекс. (крім SQLite) |

<a name="index-lengths-mysql-mariadb"></a>

#### Довжини індексу & MySQL / MariaDB

Laravel використовує`utf8mb4`набір символів за замовчуванням, що включає підтримку зберігання "смайликів" у базі даних. Якщо у вас запущена версія MySQL, старша за версію 5.7.7 або MariaDB, старша за версію 10.2.2, можливо, вам доведеться вручну налаштувати довжину рядка за замовчуванням, що генерується міграціями, щоб MySQL створював для них індекси. Ви можете налаштувати це, зателефонувавши до`Schema::defaultStringLength`метод у вашому`AppServiceProvider`:

    use Illuminate\Support\Facades\Schema;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Schema::defaultStringLength(191);
    }

Ви також можете ввімкнути`innodb_large_prefix`варіант для вашої бази даних. Зверніться до документації вашої бази даних, щоб отримати інструкції щодо того, як правильно увімкнути цю опцію.

<a name="renaming-indexes"></a>

### Перейменування покажчиків

Щоб перейменувати індекс, ви можете використовувати`renameIndex`метод. Цей метод приймає поточне ім'я індексу як перший аргумент, а бажане нове ім'я як другий аргумент:

    $table->renameIndex('from', 'to')

<a name="dropping-indexes"></a>

### Падіння індексів

Щоб скинути індекс, потрібно вказати його назву. За замовчуванням Laravel автоматично призначає ім'я індексу на основі імені таблиці, імені індексованого стовпця та типу індексу. Ось кілька прикладів:

| Команда                                                  | Опис                                                        |
| -------------------------------------------------------- | ----------------------------------------------------------- |
| `$table->dropPrimary('users_id_primary');`               | Видаліть первинний ключ із таблиці "користувачі".           |
| `$table->dropUnique('users_email_unique');`              | Видаліть унікальний індекс із таблиці "користувачі".        |
| `$table->dropIndex('geo_state_index');`                  | Видаліть основний індекс із таблиці "geo".                  |
| `$table->dropSpatialIndex('geo_location_spatialindex');` | Видаліть просторовий індекс із таблиці "geo" (крім SQLite). |

Якщо ви передаєте масив стовпців у метод, який скидає індекси, звичайне ім'я індексу буде сформовано на основі імені таблиці, стовпців і типу ключа:

    Schema::table('geo', function (Blueprint $table) {
        $table->dropIndex(['state']); // Drops index 'geo_state_index'
    });

<a name="foreign-key-constraints"></a>

### Зовнішні ключові обмеження

Laravel також надає підтримку для створення обмежень зовнішнього ключа, які використовуються для примусу посилальної цілісності на рівні бази даних. Наприклад, визначимо a`user_id`на стовпці`posts`таблиця, яка посилається на`id`стовпець на a`users`стіл:

    Schema::table('posts', function (Blueprint $table) {
        $table->unsignedBigInteger('user_id');

        $table->foreign('user_id')->references('id')->on('users');
    });

Оскільки цей синтаксис досить багатослівний, Laravel пропонує додаткові, строгіші методи, які використовують конвенцію, щоб забезпечити кращий досвід для розробників. Приклад вище можна написати так:

    Schema::table('posts', function (Blueprint $table) {
        $table->foreignId('user_id')->constrained();
    });

`foreignId`метод є псевдонімом для`unsignedBigInteger`в той час як`constrained`метод буде використовувати конвенцію для визначення імені таблиці та стовпця, на яку посилаються. Якщо назва вашої таблиці не відповідає умовам, ви можете вказати назву таблиці, передавши її як аргумент до`constrained`метод:

    Schema::table('posts', function (Blueprint $table) {
        $table->foreignId('user_id')->constrained('users');
    });

Ви також можете вказати бажану дію для властивостей обмеження "при видаленні" та "при оновленні":

    $table->foreignId('user_id')
          ->constrained()
          ->onDelete('cascade');

Будь-які додаткові[модифікатори стовпців](#column-modifiers)потрібно дзвонити раніше`constrained`:

    $table->foreignId('user_id')
          ->nullable()
          ->constrained();

Щоб випустити зовнішній ключ, ви можете використовувати`dropForeign`метод, передаючи обмеження зовнішнього ключа, яке буде видалено як аргумент. Обмеження зовнішнього ключа використовують те саме правило іменування, що і індекси, на основі імені таблиці та стовпців у обмеженні, після чого "\_іноземний "суфікс:

    $table->dropForeign('posts_user_id_foreign');

Крім того, ви можете передати масив, що містить ім'я стовпця, що містить зовнішній ключ до`dropForeign`метод. Масив буде автоматично перетворений, використовуючи дозвіл імені обмеження, використовуваний конструктором схем Laravel:

    $table->dropForeign(['user_id']);

Ви можете ввімкнути або вимкнути обмеження зовнішнього ключа в рамках своїх міграцій, використовуючи такі методи:

    Schema::enableForeignKeyConstraints();

    Schema::disableForeignKeyConstraints();

> {note} SQLite за замовчуванням вимикає обмеження зовнішнього ключа. Використовуючи SQLite, обов’язково[увімкнути підтримку зовнішнього ключа](/docs/{{version}}/database#configuration)у конфігурації бази даних перед спробою створити їх під час міграції. Крім того, SQLite підтримує зовнішні ключі лише при створенні таблиці та[не при зміні таблиць](https://www.sqlite.org/omitted.html).
