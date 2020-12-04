# Eloquent: зв'язки

-   [Вступ](#introduction)
-   [Визначення зв'язків](#defining-relationships)
    -   [Один до одного](#one-to-one)
    -   [Один до багатьох](#one-to-many)
    -   [Один до багатьох (зворотний)](#one-to-many-inverse)
    -   [Багато до Багатьох](#many-to-many)
    -   [Визначення власних моделей проміжних таблиць](#defining-custom-intermediate-table-models)
    -   [Has One Through](#has-one-through)
    -   [Has Many Through](#has-many-through)
-   [Поліморфні зв'язки](#polymorphic-relationships)
    -   [Один до одного](#one-to-one-polymorphic-relations)
    -   [Один до багатьох](#one-to-many-polymorphic-relations)
    -   [Багато до Багатьох](#many-to-many-polymorphic-relations)
    -   [Спеціальні поліморфні типи](#custom-polymorphic-types)
-   [Динамічні зв'язки](#dynamic-relationships)
-   [Запит зв'язків](#querying-relations)
    -   [Методи зв'язків vs. Динамічні властивості](#relationship-methods-vs-dynamic-properties)
    -   [Запит про існування зв'язків](#querying-relationship-existence)
    -   [Запитання про відсутність зв'язків](#querying-relationship-absence)
    -   [Запит поліморфних зв’язків](#querying-polymorphic-relationships)
    -   [Агрегування суміжних моделей](#aggregating-related-models)
    -   [Підрахунок суміжних моделей на поліморфних зв'язках](#counting-related-models-on-polymorphic-relationships)
-   [Eager завантаження](#eager-loading)
    -   [Стримуючі Eager навантаження](#constraining-eager-loads)
    -   [Lazy eager завантаження](#lazy-eager-loading)
-   [Вставка та оновлення суміжних моделей](#inserting-and-updating-related-models)
    -   [`save`Метод](#the-save-method)
    -   [`create`Метод](#the-create-method)
    -   [Belongs To Relationships](#updating-belongs-to-relationships)
    -   [Багато до багатьох зв'язків](#updating-many-to-many-relationships)
-   [Touching Parent Timestamps](#touching-parent-timestamps)

<a name="introduction"></a>

## Вступ

Таблиці баз даних часто пов'язані між собою. Наприклад, публікація в блозі може мати багато коментарів, або замовлення може бути пов’язане з користувачем, який його розмістив. Eloquent полегшує керування цими зв'язківами та роботу з ними, а також підтримує кілька різних типів зв'язків:

<div class="content-list" markdown="1">
- [One To One](#one-to-one)
- [One To Many](#one-to-many)
- [Many To Many](#many-to-many)
- [Has One Through](#has-one-through)
- [Has Many Through](#has-many-through)
- [One To One (Polymorphic)](#one-to-one-polymorphic-relations)
- [One To Many (Polymorphic)](#one-to-many-polymorphic-relations)
- [Many To Many (Polymorphic)](#many-to-many-polymorphic-relations)
</div>

<a name="defining-relationships"></a>

## Визначення зв'язків

Eloquent зв'язки визначаються як методи на ваших класах Eloquent моделей. Оскільки, як і самі Eloquent моделі, зв'язки також слугують потужними[конструктори запитів](/docs/{{version}}/queries), визначення зв'язків як методів забезпечує потужні можливості ланцюжка методів та запитів. Наприклад, ми можемо запровадити додаткові обмеження щодо цього`posts`зв'язки:

    $user->posts()->where('active', 1)->get();

Але перед тим, як заглибитися занадто глибоко у використання зв'язків, давайте навчимось визначати кожен тип.

> {note} Імена зв'язків не можуть зіткнутися з іменами атрибутів, оскільки це може призвести до того, що ваша модель не зможе знати, яке з них вирішити.

<a name="one-to-one"></a>

### Один до одного

зв'язки один до одного - це дуже базове відношення. Наприклад, a`User`модель може бути пов'язана з однією`Phone`. Щоб визначити ці зв'язки, ми ставимо a`phone`метод на`User`модель.`phone`метод повинен викликати`hasOne`і повертає його результат:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get the phone record associated with the user.
         */
        public function phone()
        {
            return $this->hasOne('App\Models\Phone');
        }
    }

Перший аргумент, переданий в`hasOne`метод - це назва відповідної моделі. Після визначення зв’язку ми можемо отримати відповідний запис, використовуючи динамічні властивості Eloquent. Динамічні властивості дозволяють отримувати доступ до методів зв'язків так, ніби вони були властивостями, визначеними в моделі:

    $phone = User::find(1)->phone;

Eloquent визначає зовнішній ключ зв'язків на основі назви моделі. У цьому випадку`Phone`модель автоматично приймається як`user_id`зовнішній ключ. Якщо ви хочете замінити цю конвенцію, ви можете передати другий аргумент до`hasOne`метод:

    return $this->hasOne('App\Models\Phone', 'foreign_key');

Крім того, Eloquent припускає, що зовнішній ключ повинен мати значення, яке відповідає`id`(або звичай`$primaryKey`) стовпець батьківського. Іншими словами, Eloquent шукатиме цінність для користувача`id`у стовпці`user_id`стовпець`Phone`запис. Якщо ви хочете, щоб зв'язки використовували значення, відмінне від`id`, ви можете передати третій аргумент до`hasOne`метод, що вказує власний ключ:

    return $this->hasOne('App\Models\Phone', 'foreign_key', 'local_key');

<a name="one-to-one-defining-the-inverse-of-the-relationship"></a>

#### Визначення зворотного відношення

Отже, ми можемо отримати доступ до`Phone`модель від нашого`User`. Тепер давайте визначимо зв'язки на`Phone`модель, яка дозволить нам отримати доступ до`User`якому належить телефон. Ми можемо визначити обернену до a`hasOne`зв'язки за допомогою`belongsTo`метод:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Phone extends Model
    {
        /**
         * Get the user that owns the phone.
         */
        public function user()
        {
            return $this->belongsTo('App\Models\User');
        }
    }

У наведеному вище прикладі Eloquent спробує зіставити`user_id`від`Phone`модель до`id`на`User`модель. Eloquent визначає ім’я зовнішнього ключа за замовчуванням, перевіряючи ім’я методу зв’язку та додаючи ім’я методу за допомогою`_id`. Однак якщо зовнішній ключ на`Phone`модель не є`user_id`, Ви можете передати ім'я власного ключа як другий аргумент для`belongsTo`метод:

    /**
     * Get the user that owns the phone.
     */
    public function user()
    {
        return $this->belongsTo('App\Models\User', 'foreign_key');
    }

Якщо ваша батьківська модель не використовує`id`як його основний ключ, або ви хочете приєднати дочірню модель до іншого стовпця, ви можете передати третій аргумент до`belongsTo`метод, що визначає спеціальний ключ вашої батьківської таблиці:

    /**
     * Get the user that owns the phone.
     */
    public function user()
    {
        return $this->belongsTo('App\Models\User', 'foreign_key', 'owner_key');
    }

<a name="one-to-many"></a>

### Один до багатьох

Відношення "один до багатьох" використовується для визначення зв'язків, коли одна модель володіє будь-якою кількістю інших моделей. Наприклад, допис у блозі може мати нескінченну кількість коментарів. Як і всі інші Eloquent зв'язки, зв'язки один до багатьох визначаються шляхом розміщення функції у вашій Eloquentй моделі:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * Get the comments for the blog post.
         */
        public function comments()
        {
            return $this->hasMany('App\Models\Comment');
        }
    }

Пам'ятайте, Eloquent автоматично визначатиме правильний стовпець зовнішнього ключа на`Comment`модель. За домовленістю, Eloquent візьме назву "справи змії" моделі, що володіє, і суфіксує її з`_id`. Отже, для цього прикладу Eloquent прийме зовнішній ключ на`Comment`модель є`post_id`.

Після того, як зв'язки будуть визначені, ми зможемо отримати доступ до колекції коментарів за допомогою доступу до`comments`майно. Пам'ятайте, оскільки Eloquent надає "динамічні властивості", ми можемо отримати доступ до методів взаємозв'язку так, ніби вони були визначені як властивості на моделі:

    $comments = App\Models\Post::find(1)->comments;

    foreach ($comments as $comment) {
        //
    }

Оскільки всі взаємозв'язки також виконують функції побудови запитів, ви можете додати додаткові обмеження, до яких отримувати коментарі, викликаючи`comments`і продовжуючи прив'язувати умови до запиту:

    $comment = App\Models\Post::find(1)->comments()->where('title', 'foo')->first();

Подобається`hasOne`методу, ви також можете замінити зовнішній та локальний ключі, передавши додаткові аргументи до`hasMany`метод:

    return $this->hasMany('App\Models\Comment', 'foreign_key');

    return $this->hasMany('App\Models\Comment', 'foreign_key', 'local_key');

<a name="one-to-many-inverse"></a>

### Один до багатьох (зворотний)

Тепер, коли ми можемо отримати доступ до всіх коментарів до публікації, давайте визначимо зв'язки, щоб дозволити коментарю отримати доступ до батьківського повідомлення. Визначити обернену до a`hasMany`зв'язки, визначте функцію взаємозв'язку на дочірній моделі, яка викликає`belongsTo`метод:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * Get the post that owns the comment.
         */
        public function post()
        {
            return $this->belongsTo('App\Models\Post');
        }
    }

Після того, як зв'язки будуть визначені, ми можемо отримати файл`Post`модель для`Comment`шляхом доступу до`post`"динамічна властивість":

    $comment = App\Models\Comment::find(1);

    echo $comment->post->title;

У наведеному вище прикладі Eloquent спробує зіставити`post_id`від`Comment`модель до`id`на`Post`модель. Eloquent визначає ім’я зовнішнього ключа за замовчуванням, перевіряючи ім’я методу зв’язку та додаючи ім’я методу за допомогою a`_`а потім ім'я стовпця первинного ключа. Однак якщо зовнішній ключ на`Comment`модель не є`post_id`, Ви можете передати ім'я власного ключа як другий аргумент для`belongsTo`метод:

    /**
     * Get the post that owns the comment.
     */
    public function post()
    {
        return $this->belongsTo('App\Models\Post', 'foreign_key');
    }

Якщо ваша батьківська модель не використовує`id`як його основний ключ, або ви хочете приєднати дочірню модель до іншого стовпця, ви можете передати третій аргумент до`belongsTo`метод, що визначає спеціальний ключ вашої батьківської таблиці:

    /**
     * Get the post that owns the comment.
     */
    public function post()
    {
        return $this->belongsTo('App\Models\Post', 'foreign_key', 'owner_key');
    }

<a name="many-to-many"></a>

### Багато до Багатьох

зв'язки "багато-до-багатьох" дещо складніше, ніж`hasOne`і`hasMany`зв'язки. Прикладом таких зв'язків є користувач із багатьма ролями, де ролі також спільно використовуються іншими користувачами. Наприклад, багато користувачів можуть виконувати роль "Адміністратора".

<a name="many-to-many-table-structure"></a>

#### Структура таблиці

Для визначення цього взаємозв'язку потрібні три таблиці бази даних:`users`,`roles`, і`role_user`.`role_user`таблиця виведена з алфавітного порядку відповідних назв моделей і містить`user_id`і`role_id`стовпці:

    users
        id - integer
        name - string

    roles
        id - integer
        name - string

    role_user
        user_id - integer
        role_id - integer

<a name="many-to-many-model-structure"></a>

#### Структура моделі

зв'язки багато-до-багатьох визначаються написанням методу, який повертає результат`belongsToMany`метод. Наприклад, визначимо`roles`метод на нашому`User`модель:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The roles that belong to the user.
         */
        public function roles()
        {
            return $this->belongsToMany('App\Models\Role');
        }
    }

Після того, як зв'язки визначені, ви можете отримати доступ до ролей користувача за допомогою`roles`динамічна властивість:

    $user = App\Models\User::find(1);

    foreach ($user->roles as $role) {
        //
    }

Як і всі інші типи зв'язків, ви можете зателефонувати на`roles`метод продовжувати прив'язувати обмеження запитів до зв'язків:

    $roles = App\Models\User::find(1)->roles()->orderBy('name')->get();

Як згадувалося раніше, для визначення назви таблиці таблиці приєднання зв'язків Eloquent об'єднає дві пов'язані назви моделей в алфавітному порядку. Однак ви можете відмінити цю конвенцію. Ви можете зробити це, передавши другий аргумент до`belongsToMany`метод:

    return $this->belongsToMany('App\Models\Role', 'role_user');

Окрім налаштування імені таблиці, що Joins, ви також можете налаштувати імена стовпців ключів таблиці, передавши додаткові аргументи до`belongsToMany`метод. Третій аргумент - ім'я зовнішнього ключа моделі, за якою ви визначаєте зв'язок, тоді як четвертий аргумент - ім'я зовнішнього ключа моделі, до якої ви приєднуєтесь:

    return $this->belongsToMany('App\Models\Role', 'role_user', 'user_id', 'role_id');

<a name="many-to-many-defining-the-inverse-of-the-relationship"></a>

#### Визначення зворотного відношення

Щоб визначити зворотне відношення багато-до-багатьох, ви робите інший дзвінок`belongsToMany`на вашій пов’язаній моделі. Щоб продовжити наш приклад користувацьких ролей, давайте визначимо`users`метод на`Role`модель:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * The users that belong to the role.
         */
        public function users()
        {
            return $this->belongsToMany('App\Models\User');
        }
    }

Як бачите, зв'язки визначаються точно так само, як і їхні`User`аналог, за винятком посилання на`App\Models\User`модель. Оскільки ми повторно використовуємо`belongsToMany`методу, усі звичайні параметри налаштування таблиці та ключів доступні при визначенні зворотних зв'язків багато-до-багатьох.

<a name="retrieving-intermediate-table-columns"></a>

#### Отримання проміжних стовпців таблиці

Як ви вже дізналися, робота з зв'язківами багато-до-багатьох вимагає наявності проміжної таблиці. Eloquent пропонує кілька дуже корисних способів взаємодії з цією таблицею. Наприклад, припустимо наш`User`об'єкт має багато`Role`об'єкти, з якими воно пов'язане. Після доступу до цього відношення ми можемо отримати доступ до проміжної таблиці за допомогою`pivot`атрибут на моделях:

    $user = App\Models\User::find(1);

    foreach ($user->roles as $role) {
        echo $role->pivot->created_at;
    }

Зверніть увагу, що кожен`Role`модель, яку ми отримуємо, автоматично призначається a`pivot`атрибут. Цей атрибут містить модель, що представляє проміжну таблицю, і може використовуватися як будь-яка інша Eloquent модель.

За замовчуванням на. Буде присутній лише ключ моделі`pivot`об'єкт. Якщо ваша зведена таблиця містить додаткові атрибути, ви повинні вказати їх при визначенні взаємозв'язку:

    return $this->belongsToMany('App\Models\Role')->withPivot('column1', 'column2');

Якщо ви хочете, щоб ваша зведена таблиця підтримувалася автоматично`created_at`і`updated_at`позначки часу, використовуйте`withTimestamps`метод визначення зв'язків:

    return $this->belongsToMany('App\Models\Role')->withTimestamps();

> {note} При використанні міток часу на зведених таблицях таблиця повинна мати обидва`created_at`і`updated_at`стовпці мітки часу.

<a name="customizing-the-pivot-attribute-name"></a>

#### Налаштування`pivot`Назва атрибута

Як зазначалося раніше, доступ до атрибутів з проміжної таблиці можна отримати на моделях із використанням`pivot`атрибут. Однак ви можете налаштувати назву цього атрибута, щоб краще відображати його мету у вашій програмі.

Наприклад, якщо у вашій програмі є користувачі, які можуть підпискити подкасти, у вас, ймовірно, є взаємозв'язок "багато-до-багатьох" між користувачами та подкастами. У цьому випадку, можливо, ви захочете перейменувати ваш проміжний пристрій доступу до таблиці на`subscription`замість`pivot`. Це можна зробити за допомогою`as`метод при визначенні взаємозв'язку:

    return $this->belongsToMany('App\Models\Podcast')
                    ->as('subscription')
                    ->withTimestamps();

Як тільки це буде зроблено, ви зможете отримати доступ до даних проміжної таблиці, використовуючи спеціальне ім'я:

    $users = User::with('podcasts')->get();

    foreach ($users->flatMap->podcasts as $podcast) {
        echo $podcast->subscription->created_at;
    }

<a name="filtering-relationships-via-intermediate-table-columns"></a>

#### Фільтрування взаємозв’язків через стовпці проміжних таблиць

Ви також можете відфільтрувати результати, які повертає`belongsToMany`за допомогою`wherePivot`,`wherePivotIn`, і`wherePivotNotIn`методи при визначенні взаємозв'язку:

    return $this->belongsToMany('App\Models\Role')->wherePivot('approved', 1);

    return $this->belongsToMany('App\Models\Role')->wherePivotIn('priority', [1, 2]);

    return $this->belongsToMany('App\Models\Role')->wherePivotNotIn('priority', [1, 2]);

<a name="defining-custom-intermediate-table-models"></a>

### Визначення власних моделей проміжних таблиць

Якщо ви хочете визначити власну модель для представлення проміжної таблиці ваших зв'язків, ви можете зателефонувати до`using`метод при визначенні зв'язків. Спеціальні моделі поворотів багато-до-багатьох повинні розширювати`Illuminate\Database\Eloquent\Relations\Pivot`класу, тоді як власні поліморфні багато-до-багатьох опорні моделі повинні розширити`Illuminate\Database\Eloquent\Relations\MorphPivot`клас. Наприклад, ми можемо визначити a`Role`який використовує звичай`RoleUser`опорна модель:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * The users that belong to the role.
         */
        public function users()
        {
            return $this->belongsToMany('App\Models\User')->using('App\Models\RoleUser');
        }
    }

При визначенні`RoleUser`модель, ми продовжимо`Pivot`клас:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Relations\Pivot;

    class RoleUser extends Pivot
    {
        //
    }

Можна комбінувати`using`і`withPivot`для того, щоб отримати стовпці з проміжної таблиці. Наприклад, ви можете отримати файл`created_by`і`updated_by`стовпці з`RoleUser`зведену таблицю, передавши імена стовпців до`withPivot`метод:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * The users that belong to the role.
         */
        public function users()
        {
            return $this->belongsToMany('App\Models\User')
                            ->using('App\Models\RoleUser')
                            ->withPivot([
                                'created_by',
                                'updated_by',
                            ]);
        }
    }

> **Примітка:**Моделі Pivot можуть не використовувати`SoftDeletes`риса. Якщо вам потрібно м’яко видалити зведені записи, розгляньте можливість перетворення вашої зведеної моделі на фактичну модель Eloquent.

<a name="custom-pivot-models-and-incrementing-ids"></a>

#### Спеціальні моделі зведення та збільшення ідентифікаторів

Якщо ви визначили взаємозв'язок "багато-до-багатьох", який використовує власну модель повороту, і ця модель повороту має первинний ключ із автоматичним збільшенням, слід переконатися, що ваш власний клас поворотної моделі визначає`incrementing`властивість, для якої встановлено`true`.

    /**
     * Indicates if the IDs are auto-incrementing.
     *
     * @var bool
     */
    public $incrementing = true;

<a name="has-one-through"></a>

### Has One Through

Моделі зв'язків "має один наскрізний" через єдине проміжне відношення.

Наприклад, у додатку автосервісу кожен`Mechanic`може мати один`Car`, і кожен`Car`може мати один`Owner`. Тоді як`Mechanic`та`Owner`не мають прямого зв'язку,`Mechanic`може отримати доступ до`Owner`_через_`Car`себе. Давайте розглянемо таблиці, необхідні для визначення цього співвідношення:

    mechanics
        id - integer
        name - string

    cars
        id - integer
        model - string
        mechanic_id - integer

    owners
        id - integer
        name - string
        car_id - integer

Тепер, коли ми розглянули структуру таблиці для взаємозв'язку, давайте визначимо зв'язок на`Mechanic`модель:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Mechanic extends Model
    {
        /**
         * Get the car's owner.
         */
        public function carOwner()
        {
            return $this->hasOneThrough('App\Models\Owner', 'App\Models\Car');
        }
    }

Перший аргумент, переданий в`hasOneThrough`method - це назва остаточної моделі, до якої ми хочемо отримати доступ, тоді як другий аргумент - це назва проміжної моделі.

Типові Eloquent домовленості про зовнішній ключ будуть використовуватися при виконанні запитів зв'язків. Якщо ви хочете налаштувати ключі зв'язків, ви можете передати їх як третій та четвертий аргументи аргументу`hasOneThrough`метод. Третім аргументом є назва зовнішнього ключа на проміжній моделі. Четвертим аргументом є назва зовнішнього ключа на кінцевій моделі. П'ятий аргумент - це локальний ключ, тоді як шостий аргумент - локальний ключ проміжної моделі:

    class Mechanic extends Model
    {
        /**
         * Get the car's owner.
         */
        public function carOwner()
        {
            return $this->hasOneThrough(
                'App\Models\Owner',
                'App\Models\Car',
                'mechanic_id', // Foreign key on cars table...
                'car_id', // Foreign key on owners table...
                'id', // Local key on mechanics table...
                'id' // Local key on cars table...
            );
        }
    }

<a name="has-many-through"></a>

### Has Many Through

Відношення "Has Many Through" забезпечує зручний ярлик для доступу до віддалених зв'язків через проміжне відношення. Наприклад, a`Country`модель може мати багато`Post`моделі через проміжний`User`модель. У цьому прикладі ви можете легко зібрати всі публікації в блозі для певної країни. Давайте розглянемо таблиці, необхідні для визначення цього співвідношення:

    countries
        id - integer
        name - string

    users
        id - integer
        country_id - integer
        name - string

    posts
        id - integer
        user_id - integer
        title - string

Хоча`posts`не містить a`country_id`стовпець,`hasManyThrough`Relations забезпечує доступ до постів країни через`$country->posts`. Для виконання цього запиту Eloquent перевіряє файл`country_id`на проміжний`users`таблиця. Після пошуку відповідних ідентифікаторів користувачів вони використовуються для запиту`posts`таблиця.

Тепер, коли ми вивчили структуру таблиці для взаємозв'язку, давайте визначимо її на`Country`модель:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Country extends Model
    {
        /**
         * Get all of the posts for the country.
         */
        public function posts()
        {
            return $this->hasManyThrough('App\Models\Post', 'App\Models\User');
        }
    }

Перший аргумент, переданий в`hasManyThrough`method - це назва остаточної моделі, до якої ми хочемо отримати доступ, тоді як другий аргумент - це назва проміжної моделі.

Типові Eloquent домовленості про зовнішній ключ будуть використовуватися при виконанні запитів зв'язків. Якщо ви хочете налаштувати ключі зв'язків, ви можете передати їх як третій та четвертий аргументи аргументу`hasManyThrough`метод. Третім аргументом є назва зовнішнього ключа на проміжній моделі. Четвертим аргументом є назва зовнішнього ключа на кінцевій моделі. П'ятий аргумент - це локальний ключ, тоді як шостий аргумент - локальний ключ проміжної моделі:

    class Country extends Model
    {
        public function posts()
        {
            return $this->hasManyThrough(
                'App\Models\Post',
                'App\Models\User',
                'country_id', // Foreign key on users table...
                'user_id', // Foreign key on posts table...
                'id', // Local key on countries table...
                'id' // Local key on users table...
            );
        }
    }

<a name="polymorphic-relationships"></a>

## Поліморфні зв'язки

Поліморфне відношення дозволяє цільовій моделі належати до більш ніж одного типу моделі, використовуючи одну асоціацію.

<a name="one-to-one-polymorphic-relations"></a>

### Один до одного (поліморфний)

<a name="one-to-one-polymorphic-table-structure"></a>

#### Структура таблиці

Поліморфне відношення "один до одного" подібне до простого відношення "один до одного"; однак цільова модель може належати до більш ніж одного типу моделі на одній асоціації. Наприклад, блог`Post`і a`User`може поділяти поліморфне відношення до`Image`модель. Використання поліморфного відношення один до одного дозволяє створити єдиний список унікальних зображень, які використовуються як для публікацій у блозі, так і для облікових записів користувачів. Спочатку розглянемо структуру таблиці:

    posts
        id - integer
        name - string

    users
        id - integer
        name - string

    images
        id - integer
        url - string
        imageable_id - integer
        imageable_type - string

Зверніть увагу на`imageable_id`і`imageable_type`стовпці на`images`таблиця.`imageable_id`стовпець міститиме значення ідентифікатора повідомлення або користувача, тоді як`imageable_type`стовпець міститиме ім'я класу батьківської моделі.`imageable_type`Eloquent використовує стовпець, щоб визначити, який "тип" батьківської моделі повернути при доступі до`imageable`відношення.

<a name="one-to-one-polymorphic-model-structure"></a>

#### Структура моделі

Далі розглянемо визначення моделі, необхідні для побудови цього взаємозв'язку:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Image extends Model
    {
        /**
         * Get the owning imageable model.
         */
        public function imageable()
        {
            return $this->morphTo();
        }
    }

    class Post extends Model
    {
        /**
         * Get the post's image.
         */
        public function image()
        {
            return $this->morphOne('App\Models\Image', 'imageable');
        }
    }

    class User extends Model
    {
        /**
         * Get the user's image.
         */
        public function image()
        {
            return $this->morphOne('App\Models\Image', 'imageable');
        }
    }

<a name="one-to-one-polymorphic-retrieving-the-relationship"></a>

#### Отримання зв'язків

Після визначення таблиці бази даних та моделей ви можете отримати доступ до взаємозв’язків через свої моделі. Наприклад, щоб отримати зображення для публікації, ми можемо використовувати`image`динамічна властивість:

    $post = App\Models\Post::find(1);

    $image = $post->image;

Ви також можете отримати батьківського з поліморфної моделі, отримавши доступ до імені методу, який виконує виклик`morphTo`. У нашому випадку це`imageable`метод на`Image`модель. Отже, ми отримаємо доступ до цього методу як динамічної властивості:

    $image = App\Models\Image::find(1);

    $imageable = $image->imageable;

`imageable`відношення на`Image`модель поверне або a`Post`або`User`наприклад, залежно від того, якому типу моделі належить зображення. Якщо вам потрібно вказати спеціальний`type`і`id`стовпці для`morphTo`відношення, завжди переконайтеся, що ви передаєте ім'я зв'язку (яке повинно точно відповідати імені методу) як перший параметр:

    /**
     * Get the model that the image belongs to.
     */
    public function imageable()
    {
        return $this->morphTo(__FUNCTION__, 'imageable_type', 'imageable_id');
    }

<a name="one-to-many-polymorphic-relations"></a>

### Один до багатьох (поліморфний)

<a name="one-to-many-polymorphic-table-structure"></a>

#### Структура таблиці

Поліморфне відношення один до багатьох подібне до простого відношення один до багатьох; однак цільова модель може належати до більш ніж одного типу моделі на одній асоціації. Наприклад, уявіть, що користувачі вашої програми можуть "коментувати" як публікації, так і відео. Використовуючи поліморфні зв'язки, ви можете використовувати одинарний`comments`таблицю для обох цих сценаріїв. Спочатку розглянемо структуру таблиці, необхідну для побудови цього взаємозв'язку:

    posts
        id - integer
        title - string
        body - text

    videos
        id - integer
        title - string
        url - string

    comments
        id - integer
        body - text
        commentable_id - integer
        commentable_type - string

<a name="one-to-many-polymorphic-model-structure"></a>

#### Структура моделі

Далі розглянемо визначення моделі, необхідні для побудови цього взаємозв'язку:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * Get the owning commentable model.
         */
        public function commentable()
        {
            return $this->morphTo();
        }
    }

    class Post extends Model
    {
        /**
         * Get all of the post's comments.
         */
        public function comments()
        {
            return $this->morphMany('App\Models\Comment', 'commentable');
        }
    }

    class Video extends Model
    {
        /**
         * Get all of the video's comments.
         */
        public function comments()
        {
            return $this->morphMany('App\Models\Comment', 'commentable');
        }
    }

<a name="one-to-many-polymorphic-retrieving-the-relationship"></a>

#### Отримання зв'язків

Після визначення таблиці бази даних та моделей ви можете отримати доступ до взаємозв’язків через свої моделі. Наприклад, щоб отримати доступ до всіх коментарів до допису, ми можемо використовувати`comments`динамічна властивість:

    $post = App\Models\Post::find(1);

    foreach ($post->comments as $comment) {
        //
    }

Ви також можете отримати власника поліморфного відношення з поліморфної моделі, отримавши доступ до імені методу, який виконує виклик до`morphTo`. У нашому випадку це`commentable`метод на`Comment`модель. Отже, ми отримаємо доступ до цього методу як динамічної властивості:

    $comment = App\Models\Comment::find(1);

    $commentable = $comment->commentable;

`commentable`відношення на`Comment`модель поверне або a`Post`або`Video`наприклад, залежно від того, якому типу моделі належить коментар.

<a name="many-to-many-polymorphic-relations"></a>

### Багато до багатьох (поліморфні)

<a name="many-to-many-polymorphic-table-structure"></a>

#### Структура таблиці

Поліморфні зв'язки "багато-до-багатьох" дещо складніші, ніж`morphOne`і`morphMany`зв'язки. Наприклад, блог`Post`і`Video`модель може поділяти поліморфне відношення до a`Tag`модель. Використання поліморфного відношення багато-до-багатьох дозволяє створити єдиний перелік унікальних тегів, якими поділяються публікації в блозі та відео. Спочатку розглянемо структуру таблиці:

    posts
        id - integer
        name - string

    videos
        id - integer
        name - string

    tags
        id - integer
        name - string

    taggables
        tag_id - integer
        taggable_id - integer
        taggable_type - string

<a name="many-to-many-polymorphic-model-structure"></a>

#### Структура моделі

Далі ми готові визначити взаємозв'язки на моделі.`Post`і`Video`обидві моделі матимуть a`tags`метод, який викликає`morphToMany`метод на базовому класі Eloquent:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * Get all of the tags for the post.
         */
        public function tags()
        {
            return $this->morphToMany('App\Models\Tag', 'taggable');
        }
    }

<a name="many-to-many-polymorphic-defining-the-inverse-of-the-relationship"></a>

#### Визначення зворотного відношення

Далі, на`Tag`моделі, вам слід визначити метод для кожної з пов'язаних з нею моделей. Отже, для цього прикладу ми визначимо a`posts`метод і a`videos`метод:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Tag extends Model
    {
        /**
         * Get all of the posts that are assigned this tag.
         */
        public function posts()
        {
            return $this->morphedByMany('App\Models\Post', 'taggable');
        }

        /**
         * Get all of the videos that are assigned this tag.
         */
        public function videos()
        {
            return $this->morphedByMany('App\Models\Video', 'taggable');
        }
    }

<a name="many-to-many-polymorphic-retrieving-the-relationship"></a>

#### Отримання зв'язків

Після визначення таблиці бази даних та моделей ви можете отримати доступ до взаємозв’язків через свої моделі. Наприклад, щоб отримати доступ до всіх тегів для публікації, ви можете використовувати`tags`динамічна властивість:

    $post = App\Models\Post::find(1);

    foreach ($post->tags as $tag) {
        //
    }

Ви також можете отримати власника поліморфного відношення з поліморфної моделі, отримавши доступ до імені методу, який виконує виклик до`morphedByMany`. У нашому випадку це`posts`або`videos`методи на`Tag`модель. Отже, ви отримаєте доступ до цих методів як динамічні властивості:

    $tag = App\Models\Tag::find(1);

    foreach ($tag->videos as $video) {
        //
    }

<a name="custom-polymorphic-types"></a>

### Спеціальні поліморфні типи

За замовчуванням Laravel використовуватиме повністю кваліфіковане ім'я класу для зберігання типу відповідної моделі. Наприклад, з огляду на приклад "один до багатьох", де a`Comment`може належати а`Post`або a`Video`, за замовчуванням`commentable_type`буде або`App\Models\Post`або`App\Models\Video`відповідно. Однак, можливо, ви захочете відокремити базу даних від внутрішньої структури вашої програми. У цьому випадку ви можете визначити "карту перетворення", щоб доручити Eloquent використовувати власне ім'я для кожної моделі замість імені класу:

    use Illuminate\Database\Eloquent\Relations\Relation;

    Relation::morphMap([
        'posts' => 'App\Models\Post',
        'videos' => 'App\Models\Video',
    ]);

Ви можете зареєструвати`morphMap`в`boot`функція вашого`AppServiceProvider`або створіть окремого постачальника послуг, якщо хочете.

> {note} Під час додавання "морф-карти" до вашого існуючого додатка, кожна змінна`*_type`Значення стовпця у вашій базі даних, яке все ще містить повністю кваліфікований клас, потрібно буде перетворити на його назву "map".

Ви можете визначити морф-псевдонім даної моделі під час виконання, використовуючи`getMorphClass`метод. І навпаки, ви можете визначити повну назву класу, пов’язану з морфічним псевдонімом, використовуючи`Relation::getMorphedModel`метод:

    use Illuminate\Database\Eloquent\Relations\Relation;

    $alias = $post->getMorphClass();

    $class = Relation::getMorphedModel($alias);

<a name="dynamic-relationships"></a>

### Динамічні зв'язки

Ви можете використовувати`resolveRelationUsing`метод визначення зв'язків між Eloquent моделями під час виконання. Хоча це зазвичай не рекомендується для нормальної розробки додатків, це іноді може бути корисним при розробці пакетів Laravel:

    use App\Models\Order;
    use App\Models\Customer;

    Order::resolveRelationUsing('customer', function ($orderModel) {
        return $orderModel->belongsTo(Customer::class, 'customer_id');
    });

> {note} Під час визначення динамічних зв’язків завжди вказуйте явні аргументи імені ключа для методів Eloquent Relationship.

<a name="querying-relations"></a>

## Запит зв'язків

Оскільки всі типи Eloquent зв'язків визначаються за допомогою методів, ви можете викликати ці методи, щоб отримати екземпляр зв'язків, фактично не виконуючи запитів щодо зв'язків. Крім того, всі типи Eloquent зв'язків також слугують[конструктори запитів](/docs/{{version}}/queries), що дозволяє продовжувати прив'язувати обмеження до запиту взаємозв'язку перед остаточним виконанням SQL щодо вашої бази даних.

Наприклад, уявіть систему блогу, в якій a`User`модель має багато пов'язаних`Post`моделі:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get all of the posts for the user.
         */
        public function posts()
        {
            return $this->hasMany('App\Models\Post');
        }
    }

Ви можете запитати`posts`зв'язків і додати додаткові обмеження у зв'язки так:

    $user = App\Models\User::find(1);

    $user->posts()->where('active', 1)->get();

Ви можете використовувати будь-який із[конструктор запитів](/docs/{{version}}/queries)методів на взаємозв'язку, тому обов'язково вивчіть документацію конструктора запитів, щоб дізнатись про всі доступні вам методи.

<a name="chaining-orwhere-clauses-after-relationships"></a>

#### Мережа`orWhere`Статті після зв'язків

Як продемонстровано у наведеному вище прикладі, ви можете вільно додавати додаткові обмеження до зв'язків під час їх запиту. Однак будьте обережні під час ланцюга`orWhere`положення про зв'язки, як`orWhere`пропозиції будуть логічно згруповані на тому самому рівні, що і обмеження зв'язків:

    $user->posts()
            ->where('active', 1)
            ->orWhere('votes', '>=', 100)
            ->get();

    // select * from posts
    // where user_id = ? and active = 1 or votes >= 100

У більшості ситуацій ви, мабуть, маєте намір використовувати[групи обмежень](/docs/{{version}}/queries#parameter-grouping)щоб логічно згрупувати умовні перевірки між дужками:

    use Illuminate\Database\Eloquent\Builder;

    $user->posts()
            ->where(function (Builder $query) {
                return $query->where('active', 1)
                             ->orWhere('votes', '>=', 100);
            })
            ->get();

    // select * from posts
    // where user_id = ? and (active = 1 or votes >= 100)

<a name="relationship-methods-vs-dynamic-properties"></a>

### Методи зв'язків проти. Динамічні властивості

Якщо вам не потрібно додавати додаткові обмеження до запиту зв'язків Eloquent, ви можете отримати доступ до зв'язків так, ніби це властивість. Наприклад, продовжуючи використовувати наш`User`і`Post`приклади моделей, ми можемо отримати доступ до всіх публікацій користувача таким чином:

    $user = App\Models\User::find(1);

    foreach ($user->posts as $post) {
        //
    }

Динамічні властивості - це "ліниве завантаження", тобто вони завантажуватимуть дані своїх зв'язків лише тоді, коли ви фактично отримаєте до них доступ. Через це розробники часто використовують[охоче завантаження](#eager-loading)для попереднього завантаження зв'язків, які вони знають, будуть доступні після завантаження моделі. Ретельне завантаження забезпечує значне зменшення запитів SQL, які повинні виконуватися для завантаження зв'язків моделі.

<a name="querying-relationship-existence"></a>

### Запит про існування зв'язків

Отримуючи доступ до записів моделі, ви можете обмежити свої результати на основі існування зв'язків. Наприклад, уявіть, що ви хочете отримати всі публікації в блозі, які містять принаймні один коментар. Для цього ви можете передати ім’я зв'язків`has`і`orHas`методи:

    // Retrieve all posts that have at least one comment...
    $posts = App\Models\Post::has('comments')->get();

Ви також можете вказати оператор і підрахувати для подальшої настройки запиту:

    // Retrieve all posts that have three or more comments...
    $posts = App\Models\Post::has('comments', '>=', 3)->get();

Вкладені`has`твердження також можуть бути побудовані з використанням Tagging "крапка". Наприклад, ви можете отримати всі публікації, які мають принаймні один коментар, і проголосувати:

    // Retrieve posts that have at least one comment with votes...
    $posts = App\Models\Post::has('comments.votes')->get();

Якщо вам потрібно ще більше енергії, ви можете використовувати`whereHas`і`orWhereHas`методи, щоб поставити умови "де" на вашому`has`запитів. Ці методи дозволяють додавати власні обмеження до обмеження зв'язків, наприклад перевірку вмісту коментаря:

    use Illuminate\Database\Eloquent\Builder;

    // Retrieve posts with at least one comment containing words like foo%...
    $posts = App\Models\Post::whereHas('comments', function (Builder $query) {
        $query->where('content', 'like', 'foo%');
    })->get();

    // Retrieve posts with at least ten comments containing words like foo%...
    $posts = App\Models\Post::whereHas('comments', function (Builder $query) {
        $query->where('content', 'like', 'foo%');
    }, '>=', 10)->get();

<a name="querying-relationship-absence"></a>

### Запитання про відсутність зв'язків

Отримуючи доступ до записів для моделі, ви можете обмежити свої результати на основі відсутності зв'язків. Наприклад, уявіть, що ви хочете отримати всі повідомлення в блозі**не робіть**є коментарі. Для цього ви можете передати ім’я зв'язків`doesntHave`і`orDoesntHave`методи:

    $posts = App\Models\Post::doesntHave('comments')->get();

Якщо вам потрібно ще більше енергії, ви можете використовувати`whereDoesntHave`і`orWhereDoesntHave`методи, щоб поставити умови "де" на вашому`doesntHave`запитів. Ці методи дозволяють додавати власні обмеження до обмеження зв'язків, наприклад перевірку вмісту коментаря:

    use Illuminate\Database\Eloquent\Builder;

    $posts = App\Models\Post::whereDoesntHave('comments', function (Builder $query) {
        $query->where('content', 'like', 'foo%');
    })->get();

Ви можете використовувати Tagging "крапка" для виконання запиту щодо вкладеного відношення. Наприклад, наступний запит отримає всі дописи, які не мають коментарів, та дописи з коментарями авторів, які не заборонені:

    use Illuminate\Database\Eloquent\Builder;

    $posts = App\Models\Post::whereDoesntHave('comments.author', function (Builder $query) {
        $query->where('banned', 0);
    })->get();

<a name="querying-polymorphic-relationships"></a>

### Запит поліморфних зв’язків

Запитати про існування`MorphTo`Ви можете використовувати`whereHasMorph`метод та відповідні йому методи:

    use Illuminate\Database\Eloquent\Builder;

    // Retrieve comments associated to posts or videos with a title like foo%...
    $comments = App\Models\Comment::whereHasMorph(
        'commentable',
        ['App\Models\Post', 'App\Models\Video'],
        function (Builder $query) {
            $query->where('title', 'like', 'foo%');
        }
    )->get();

    // Retrieve comments associated to posts with a title not like foo%...
    $comments = App\Models\Comment::whereDoesntHaveMorph(
        'commentable',
        'App\Models\Post',
        function (Builder $query) {
            $query->where('title', 'like', 'foo%');
        }
    )->get();

Ви можете використовувати`$type`параметр для додавання різних обмежень залежно від відповідної моделі:

    use Illuminate\Database\Eloquent\Builder;

    $comments = App\Models\Comment::whereHasMorph(
        'commentable',
        ['App\Models\Post', 'App\Models\Video'],
        function (Builder $query, $type) {
            $query->where('title', 'like', 'foo%');

            if ($type === 'App\Models\Post') {
                $query->orWhere('content', 'like', 'foo%');
            }
        }
    )->get();

Замість передачі масиву можливих поліморфних моделей ви можете надати`*`як підстановочний знак, і нехай Laravel отримує всі можливі поліморфні типи з бази даних. Laravel виконає додатковий запит, щоб виконати цю операцію:

    use Illuminate\Database\Eloquent\Builder;

    $comments = App\Models\Comment::whereHasMorph('commentable', '*', function (Builder $query) {
        $query->where('title', 'like', 'foo%');
    })->get();

<a name="aggregating-related-models"></a>

### Агрегування суміжних моделей

<a name="counting-related-models"></a>

#### Підрахунок суміжних моделей

Якщо ви хочете підрахувати кількість результатів зв'язків, не завантажуючи їх, ви можете скористатися`withCount`метод, який розмістить a`{relation}_count`на отриманих моделях. Наприклад:

    $posts = App\Models\Post::withCount('comments')->get();

    foreach ($posts as $post) {
        echo $post->comments_count;
    }

Ви можете додати "кількість" для кількох зв'язків, а також додати обмеження до запитів:

    use Illuminate\Database\Eloquent\Builder;

    $posts = App\Models\Post::withCount(['votes', 'comments' => function (Builder $query) {
        $query->where('content', 'like', 'foo%');
    }])->get();

    echo $posts[0]->votes_count;
    echo $posts[0]->comments_count;

Ви також можете встановити псевдонім результату підрахунку зв'язків, дозволяючи кілька відліків одного і того ж відношення:

    use Illuminate\Database\Eloquent\Builder;

    $posts = App\Models\Post::withCount([
        'comments',
        'comments as pending_comments_count' => function (Builder $query) {
            $query->where('approved', false);
        },
    ])->get();

    echo $posts[0]->comments_count;

    echo $posts[0]->pending_comments_count;

Якщо ви комбінуєте`withCount`з`select`заяву, переконайтеся, що ви телефонуєте`withCount`після`select`метод:

    $posts = App\Models\Post::select(['title', 'body'])->withCount('comments')->get();

    echo $posts[0]->title;
    echo $posts[0]->body;
    echo $posts[0]->comments_count;

Крім того, за допомогою`loadCount`методом, ви можете завантажити кількість зв'язків після того, як батьківська модель уже отримана:

    $book = App\Models\Book::first();

    $book->loadCount('genres');

Якщо вам потрібно встановити додаткові обмеження запиту на запит енергійного завантаження, ви можете передати масив, закріплений зв'язківами, які ви хочете завантажити. Значення масиву мають бути`Closure`екземпляри, які отримують екземпляр конструктора запитів:

    $book->loadCount(['reviews' => function ($query) {
        $query->where('rating', 5);
    }])

#### Інші сукупні функції

На додаток до`withCount`метод, наводить Eloquent`withMin`,`withMax`,`withAvg`, і`withSum`. Ці методи розмістять a`{relation}_{function}_{column}`на отриманих моделях. Наприклад:

    $posts = App\Models\Post::withSum('comments', 'votes')->get();

    foreach ($posts as $post) {
        echo $post->comments_sum_votes;
    }

Ці додаткові сукупні операції можуть також виконуватися на Eloquent моделях, які вже були отримані:

    $post = App\Models\Post::first();

    $post->loadSum('comments', 'votes');

<a name="counting-related-models-on-polymorphic-relationships"></a>

### Підрахунок суміжних моделей на поліморфних зв'язках

Якщо ви хочете охоче завантажити a`morphTo`зв'язки, а також вкладені зв'язки розраховує на різні сутності, які можуть бути повернені цими зв'язківами, ви можете використовувати`with`метод у поєднанні з`morphTo`зв'язки`morphWithCount`метод.

У цьому прикладі припустимо`Photo`і`Post`моделі можуть створювати`ActivityFeed`моделі. Крім того, припустимо, що`Photo`моделі пов'язані з`Tag`моделі та`Post`моделі пов'язані з`Comment`моделі.

Використовуючи ці визначення моделі та взаємозв'язки, ми можемо отримати`ActivityFeed`модельні екземпляри та охоче завантажують усі`parentable`моделі та їх кількість вкладених зв'язків:

    use Illuminate\Database\Eloquent\Relations\MorphTo;

    $activities = ActivityFeed::query()
        ->with(['parentable' => function (MorphTo $morphTo) {
            $morphTo->morphWithCount([
                Photo::class => ['tags'],
                Post::class => ['comments'],
            ]);
        }])->get();

Крім того, ви можете використовувати`loadMorphCount`Метод охочого завантаження всіх вкладених зв'язків розраховується на різні сутності поліморфного відношення, якщо`ActivityFeed`моделі вже отримано:

    $activities = ActivityFeed::with('parentable')
        ->get()
        ->loadMorphCount('parentable', [
            Photo::class => ['tags'],
            Post::class => ['comments'],
        ]);

<a name="eager-loading"></a>

## eager завантаження

При доступі до Eloquent зв'язків як властивостей, дані зв'язків "завантажуються ліниво". Це означає, що дані зв'язків фактично не завантажуються до першого доступу до властивості. Однак Eloquent може «охоче завантажувати» зв'язки під час запиту батьківської моделі. Бажаюче завантаження полегшує проблему запиту N + 1. Щоб проілюструвати проблему запиту N + 1, розглянемо a`Book`модель, яка пов'язана з`Author`:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Book extends Model
    {
        /**
         * Get the author that wrote the book.
         */
        public function author()
        {
            return $this->belongsTo('App\Models\Author');
        }
    }

Тепер давайте отримаємо всі книги та їх авторів:

    $books = App\Models\Book::all();

    foreach ($books as $book) {
        echo $book->author->name;
    }

Цей цикл виконає 1 запит для отримання всіх книг у таблиці, а потім ще один запит для кожної книги для отримання автора. Отже, якщо у нас є 25 книг, у наведеному вище коді буде виконано 26 запитів: 1 для оригінальної книги та 25 додаткових запитів для пошуку автора кожної книги.

На щастя, ми можемо використовувати завзяте завантаження, щоб зменшити цю операцію лише до 2 запитів. Під час запиту ви можете вказати, які взаємозв'язки слід завантажувати, використовуючи`with`метод:

    $books = App\Models\Book::with('author')->get();

    foreach ($books as $book) {
        echo $book->author->name;
    }

Для цієї операції буде виконано лише два запити:

    select * from books

    select * from authors where id in (1, 2, 3, 4, 5, ...)

<a name="eager-loading-multiple-relationships"></a>

#### Бажання завантажувати кілька зв'язків

Іноді вам може знадобитися охоче завантажити кілька різних зв'язків за одну операцію. Для цього просто передайте додаткові аргументи в`with`метод:

    $books = App\Models\Book::with(['author', 'publisher'])->get();

<a name="nested-eager-loading"></a>

#### Вкладене нетерпляче завантаження

Щоб охоче завантажувати вкладені зв'язки, ви можете використовувати синтаксис "крапка". Наприклад, давайте охоче завантажимо всіх авторів книги та всі особисті контакти автора в одне промовисте висловлювання:

    $books = App\Models\Book::with('author.contacts')->get();

<a name="nested-eager-loading-morphto-relationships"></a>

#### Вкладене нетерпляче завантаження`morphTo`зв'язки

Якщо ви хочете охоче завантажити a`morphTo`зв'язки, а також вкладені зв'язки на різні сутності, які можуть бути повернені цими зв'язківами, ви можете використовувати`with`метод у поєднанні з`morphTo`зв'язки`morphWith`метод. Щоб проілюструвати цей метод, давайте розглянемо таку модель:

    <?php

    use Illuminate\Database\Eloquent\Model;

    class ActivityFeed extends Model
    {
        /**
         * Get the parent of the activity feed record.
         */
        public function parentable()
        {
            return $this->morphTo();
        }
    }

У цьому прикладі припустимо`Event`,`Photo`, і`Post`моделі можуть створювати`ActivityFeed`моделі. Крім того, припустимо, що`Event`моделі належать до`Calendar`модель,`Photo`моделі пов'язані з`Tag`моделі та`Post`моделі належать до`Author`модель.

Використовуючи ці визначення моделі та взаємозв'язки, ми можемо отримати`ActivityFeed`модельні екземпляри та охоче завантажують усі`parentable`моделі та їх відповідні вкладені зв'язки:

    use Illuminate\Database\Eloquent\Relations\MorphTo;

    $activities = ActivityFeed::query()
        ->with(['parentable' => function (MorphTo $morphTo) {
            $morphTo->morphWith([
                Event::class => ['calendar'],
                Photo::class => ['tags'],
                Post::class => ['author'],
            ]);
        }])->get();

<a name="eager-loading-specific-columns"></a>

#### Бажаюче завантаження конкретних стовпців

Можливо, вам не завжди потрібні кожен стовпець із зв'язків, які ви отримуєте. З цієї причини Eloquent дозволяє вказати, які стовпці відношення ви хочете отримати:

    $books = App\Models\Book::with('author:id,name')->get();

> {note} Під час використання цієї функції ви завжди повинні включати`id`і будь-які відповідні стовпці зовнішнього ключа у списку стовпців, які ви хочете отримати.

<a name="eager-loading-by-default"></a>

#### Бажаюче завантаження за замовчуванням

Іноді вам може знадобитися завжди завантажувати деякі зв'язки під час отримання моделі. Для цього ви можете визначити a`$with`властивість на моделі:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Book extends Model
    {
        /**
         * The relationships that should always be loaded.
         *
         * @var array
         */
        protected $with = ['author'];

        /**
         * Get the author that wrote the book.
         */
        public function author()
        {
            return $this->belongsTo('App\Models\Author');
        }
    }

Якщо ви хочете видалити елемент із`$with`властивість для одного запиту, ви можете використовувати`without`метод:

    $books = App\Models\Book::without('author')->get();

<a name="constraining-eager-loads"></a>

### Стримуючі Eager навантаження

Іноді ви можете захотіти завантажити зв'язки, але також вказати додаткові умови запиту для запиту охочого завантаження. Ось приклад:

    $users = App\Models\User::with(['posts' => function ($query) {
        $query->where('title', 'like', '%first%');
    }])->get();

У цьому прикладі Eloquent буде прагнути завантажувати лише пости там, де їх розміщено`title`стовпець містить слово`first`. Ви можете зателефонувати іншому[конструктор запитів](/docs/{{version}}/queries)методи для подальшої настройки налаштування операції завантаження:

    $users = App\Models\User::with(['posts' => function ($query) {
        $query->orderBy('created_at', 'desc');
    }])->get();

> {примітка}`limit`і`take`методи побудови запитів не можуть використовуватися при обмеженні великих навантажень.

<a name="constraining-eager-loading-of-morph-to-relationships"></a>

#### Стримуюче eager завантаження`MorphTo`зв'язки

Якщо ви охоче завантажуєте a`MorphTo`Eloquent буде запускати кілька запитів для отримання кожного різного типу пов'язаної моделі. Ви можете націлити умови на кожен із цих запитів, використовуючи`MorphTo`зв'язки`constrain`метод:

    use Illuminate\Database\Eloquent\Builder;
    use Illuminate\Database\Eloquent\Relations\MorphTo;

    $comments = Comment::with(['commentable' => function (MorphTo $morphTo) {
        $morphTo->constrain([
            Post::class => function (Builder $query) {
                $query->whereNull('hidden_at');
            },
            Video::class => function (Builder $query) {
                $query->where('type', 'educational');
            },
        ]);
    }])->get();

У цьому прикладі Eloquent буде прагнути лише завантажувати повідомлення, які не були приховані, а відео мають`type`значення "освітній".

<a name="lazy-eager-loading"></a>

### Lazy eager завантаження

Іноді вам може знадобитися охоче завантажувати зв'язки після того, як батьківська модель уже отримана. Наприклад, це може бути корисно, якщо вам потрібно динамічно вирішити, чи завантажувати відповідні моделі:

    $books = App\Models\Book::all();

    if ($someCondition) {
        $books->load('author', 'publisher');
    }

Якщо вам потрібно встановити додаткові обмеження запиту на запит енергійного завантаження, ви можете передати масив, закріплений зв'язківами, які ви хочете завантажити. Значення масиву мають бути`Closure`екземпляри, які отримують екземпляр запиту:

    $author->load(['books' => function ($query) {
        $query->orderBy('published_date', 'asc');
    }]);

Щоб завантажити зв'язки лише тоді, коли вони ще не завантажені, використовуйте`loadMissing`метод:

    public function format(Book $book)
    {
        $book->loadMissing('author');

        return [
            'name' => $book->name,
            'author' => $book->author->name,
        ];
    }

<a name="nested-lazy-eager-loading-morphto"></a>

#### Вкладений Lazy eagerння завантаження &`morphTo`

Якщо ви хочете охоче завантажити a`morphTo`зв'язки, а також вкладені зв'язки на різні сутності, які можуть бути повернені цими зв'язківами, ви можете використовувати`loadMorph`метод.

Цей метод приймає назву`morphTo`відношення як перший аргумент, а масив пар модель / відношення як другий аргумент. Щоб проілюструвати цей метод, давайте розглянемо таку модель:

    <?php

    use Illuminate\Database\Eloquent\Model;

    class ActivityFeed extends Model
    {
        /**
         * Get the parent of the activity feed record.
         */
        public function parentable()
        {
            return $this->morphTo();
        }
    }

У цьому прикладі припустимо`Event`,`Photo`, і`Post`моделі можуть створювати`ActivityFeed`моделі. Крім того, припустимо, що`Event`моделі належать до`Calendar`модель,`Photo`моделі пов'язані з`Tag`моделі та`Post`моделі належать до`Author`модель.

Використовуючи ці визначення моделі та взаємозв'язки, ми можемо отримати`ActivityFeed`модельні екземпляри та охоче завантажують усі`parentable`моделі та їх відповідні вкладені зв'язки:

    $activities = ActivityFeed::with('parentable')
        ->get()
        ->loadMorph('parentable', [
            Event::class => ['calendar'],
            Photo::class => ['tags'],
            Post::class => ['author'],
        ]);

<a name="inserting-and-updating-related-models"></a>

## Вставка та оновлення суміжних моделей

<a name="the-save-method"></a>

### Метод збереження

Eloquent пропонує зручні методи додавання нових моделей до зв'язків. Наприклад, можливо, вам потрібно вставити новий`Comment`для`Post`модель. Замість ручного встановлення`post_id`атрибут на`Comment`, ви можете вставити`Comment`безпосередньо з зв'язків`save`метод:

    $comment = new App\Models\Comment(['message' => 'A new comment.']);

    $post = App\Models\Post::find(1);

    $post->comments()->save($comment);

Зверніть увагу, що ми не мали доступу до`comments` relationship as a dynamic property. Instead, we called the `comments`метод отримання екземпляра зв'язків.`save`метод автоматично додасть відповідний`post_id`значення для нового`Comment`модель.

Якщо вам потрібно зберегти кілька споріднених моделей, ви можете використовувати`saveMany`метод:

    $post = App\Models\Post::find(1);

    $post->comments()->saveMany([
        new App\Models\Comment(['message' => 'A new comment.']),
        new App\Models\Comment(['message' => 'Another comment.']),
    ]);

`save`і`saveMany`Методи не додаватимуть нові моделі до будь-яких зв'язків в пам'яті, які вже завантажені до батьківської моделі. Якщо ви плануєте отримати доступ до зв'язків після використання`save`або`saveMany`методами, можливо, ви захочете використовувати`refresh`метод для перезавантаження моделі та її взаємозв’язків:

    $post->comments()->save($comment);

    $post->refresh();

    // All comments, including the newly saved comment...
    $post->comments;

<a name="the-push-method"></a>

#### Рекурсивно збереження моделей та зв'язків

Якщо ви хотіли б`save`Вашу модель та всі пов'язані з нею зв'язки, Ви можете використовувати`push`метод:

    $post = App\Models\Post::find(1);

    $post->comments[0]->message = 'Message';
    $post->comments[0]->author->name = 'Author Name';

    $post->push();

<a name="the-create-method"></a>

### Метод створення

На додаток до`save`і`saveMany`методами, ви також можете використовувати`create`метод, який приймає масив атрибутів, створює модель і вставляє її в базу даних. Знову ж таки, різниця між`save`і`create`чи це`save`приймає повний екземпляр Eloquentї моделі, поки`create`приймає звичайний PHP`array`:

    $post = App\Models\Post::find(1);

    $comment = $post->comments()->create([
        'message' => 'A new comment.',
    ]);

> {tip} Перед використанням`create`методу, обов’язково перегляньте документацію щодо атрибута[масове доручення](/docs/{{version}}/eloquent#mass-assignment).

Ви можете використовувати`createMany`метод створення декількох пов'язаних моделей:

    $post = App\Models\Post::find(1);

    $post->comments()->createMany([
        [
            'message' => 'A new comment.',
        ],
        [
            'message' => 'Another new comment.',
        ],
    ]);

Ви також можете використовувати`findOrNew`,`firstOrNew`,`firstOrCreate`і`updateOrCreate`методи до[створювати та оновлювати моделі зв'язків](https://laravel.com/docs/{{version}}/eloquent#other-creation-methods).

<a name="updating-belongs-to-relationships"></a>

### Belongs To Relationships

Під час оновлення a`belongsTo`Ви можете використовувати`associate`метод. Цей метод встановить зовнішній ключ для дочірньої моделі:

    $account = App\Models\Account::find(10);

    $user->account()->associate($account);

    $user->save();

При видаленні a`belongsTo`Ви можете використовувати`dissociate`метод. Цей метод встановлює зовнішній ключ зв'язків`null`:

    $user->account()->dissociate();

    $user->save();

<a name="default-models"></a>

#### Моделі за замовчуванням

`belongsTo`,`hasOne`,`hasOneThrough`, і`morphOne`зв'язки дозволяють визначити модель за замовчуванням, яка буде повернута, якщо заданий зв'язок є`null`. Цю закономірність часто називають[Нульовий шаблон об'єкта](https://en.wikipedia.org/wiki/Null_Object_pattern)і може допомогти видалити умовні перевірки у коді. У наступному прикладі файл`user`відношення поверне порожнє`App\Models\User`модель, якщо ні`user`додається до поста:

    /**
     * Get the author of the post.
     */
    public function user()
    {
        return $this->belongsTo('App\Models\User')->withDefault();
    }

Щоб заповнити атрибутами модель за замовчуванням, ви можете передати масив або Закриття в`withDefault`метод:

    /**
     * Get the author of the post.
     */
    public function user()
    {
        return $this->belongsTo('App\Models\User')->withDefault([
            'name' => 'Guest Author',
        ]);
    }

    /**
     * Get the author of the post.
     */
    public function user()
    {
        return $this->belongsTo('App\Models\User')->withDefault(function ($user, $post) {
            $user->name = 'Guest Author';
        });
    }

<a name="updating-many-to-many-relationships"></a>

### Багато до багатьох зв'язків

<a name="attaching-detaching"></a>

#### Кріплення / від'єднання

Eloquent також надає кілька додаткових допоміжних методів, щоб зробити роботу із суміжними моделями більш зручною. Наприклад, уявімо, що користувач може мати багато ролей, а роль може мати багато користувачів. Щоб прикріпити роль до користувача, вставивши запис у проміжну таблицю, яка приєднує моделі, використовуйте`attach`метод:

    $user = App\Models\User::find(1);

    $user->roles()->attach($roleId);

Прикріплюючи відношення до моделі, ви також можете передати масив додаткових даних, які потрібно вставити в проміжну таблицю:

    $user->roles()->attach($roleId, ['expires' => $expires]);

Іноді може знадобитися видалити роль від користувача. Щоб видалити запис зв'язків багато-до-багатьох, використовуйте`detach`метод.`detach`метод видалить відповідний запис із проміжної таблиці; проте обидві моделі залишаться в базі даних:

    // Detach a single role from the user...
    $user->roles()->detach($roleId);

    // Detach all roles from the user...
    $user->roles()->detach();

Для зручності,`attach`і`detach`також приймати масиви ідентифікаторів як вхідні дані:

    $user = App\Models\User::find(1);

    $user->roles()->detach([1, 2, 3]);

    $user->roles()->attach([
        1 => ['expires' => $expires],
        2 => ['expires' => $expires],
    ]);

<a name="syncing-associations"></a>

#### Асоціації, що синхронізують

Ви також можете використовувати`sync`метод побудови асоціацій багато-до-багатьох.`sync`метод приймає масив ідентифікаторів для розміщення в проміжній таблиці. Будь-які ідентифікатори, яких немає в даному масиві, будуть видалені з проміжної таблиці. Отже, після завершення цієї операції в проміжній таблиці будуть існувати лише ідентифікатори в даному масиві:

    $user->roles()->sync([1, 2, 3]);

Ви також можете передавати додаткові проміжні значення таблиці з ідентифікаторами:

    $user->roles()->sync([1 => ['expires' => true], 2, 3]);

Якщо ви не хочете від'єднувати існуючі ідентифікатори, ви можете використовувати`syncWithoutDetaching`метод:

    $user->roles()->syncWithoutDetaching([1, 2, 3]);

<a name="toggling-associations"></a>

#### Перемикання асоціацій

зв'язки "багато-до-багатьох" також надають "`toggle`метод, який "перемикає" статус вкладення даних ідентифікаторів. Якщо даний ідентифікатор наразі додається, він буде від’єднаний. Аналогічним чином, якщо він наразі від'єднаний, він буде доданий:

    $user->roles()->toggle([1, 2, 3]);

<a name="saving-additional-data-on-a-pivot-table"></a>

#### Збереження додаткових даних у зведеній таблиці

При роботі з зв'язківами багато-до-багатьох,`save`метод приймає масив додаткових атрибутів проміжної таблиці як другий аргумент:

    App\Models\User::find(1)->roles()->save($role, ['expires' => $expires]);

<a name="updating-a-record-on-a-pivot-table"></a>

#### Оновлення запису на зведеній таблиці

Якщо вам потрібно оновити існуючий рядок у зведеній таблиці, ви можете використовувати`updateExistingPivot`метод. Цей метод приймає зовнішній ключ зведеного запису та масив атрибутів для оновлення:

    $user = App\Models\User::find(1);

    $user->roles()->updateExistingPivot($roleId, $attributes);

<a name="touching-parent-timestamps"></a>

## Touching Parent Timestamps

When a model `belongsTo`або`belongsToMany`інша модель, така як`Comment`який належить а`Post`, іноді корисно оновити позначку часу батьків під час оновлення дочірньої моделі. Наприклад, коли a`Comment`модель оновлена, можливо, ви захочете автоматично "торкнутися"`updated_at`позначка часу володіння`Post`. Красномовство робить це просто. Просто додайте`touches`властивість, що містить назви відношень до дочірньої моделі:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * All of the relationships to be touched.
         *
         * @var array
         */
        protected $touches = ['post'];

        /**
         * Get the post that the comment belongs to.
         */
        public function post()
        {
            return $this->belongsTo('App\Models\Post');
        }
    }

Тепер, коли ви оновлюєте`Comment`, що володіє`Post`матиме своє`updated_at`Оновлено також стовпець, що робить зручнішим знати, коли анулювати кеш файлу`Post`модель:

    $comment = App\Models\Comment::find(1);

    $comment->text = 'Edit to this comment!';

    $comment->save();
