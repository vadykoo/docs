# Eloquent: серіалізація

-   [Вступ](#introduction)
-   [Серіалізація моделей та колекцій](#serializing-models-and-collections)
    -   [Серіалізація до масивів](#serializing-to-arrays)
    -   [Серіалізація до JSON](#serializing-to-json)
-   [Приховування атрибутів з JSON](#hiding-attributes-from-json)
-   [Додавання значень до JSON](#appending-values-to-json)
-   [Серіалізація дати](#date-serialization)

<a name="introduction"></a>

## Вступ

Створюючи API JSON, вам часто потрібно буде перетворити свої моделі та зв'язки в масиви або JSON. Eloquent включає зручні методи для здійснення цих перетворень, а також контроль того, які атрибути входять до ваших серіалізацій.

<a name="serializing-models-and-collections"></a>

## Серіалізація моделей та колекцій

<a name="serializing-to-arrays"></a>

### Серіалізація до масивів

Для перетворення моделі та її завантаження[зв'язки](/docs/{{version}}/eloquent-relationships)для масиву, ви повинні використовувати`toArray`метод. Цей метод є рекурсивним, тому всі атрибути та всі зв'язки (включаючи зв'язки зв'язків) будуть перетворені в масиви:

    $user = App\Models\User::with('roles')->first();

    return $user->toArray();

Щоб перетворити в масив лише атрибути моделі, використовуйте`attributesToArray`метод:

    $user = App\Models\User::first();

    return $user->attributesToArray();

Ви також можете конвертувати цілі[колекцій](/docs/{{version}}/eloquent-collections)моделей до масивів:

    $users = App\Models\User::all();

    return $users->toArray();

<a name="serializing-to-json"></a>

### Серіалізація до JSON

Щоб перетворити модель у JSON, вам слід використовувати`toJson`метод. Люблю`toArray`,`toJson`метод є рекурсивним, тому всі атрибути та відношення будуть перетворені в JSON. Ви також можете вказати параметри кодування JSON[підтримується PHP](https://secure.php.net/manual/en/function.json-encode.php):

    $user = App\Models\User::find(1);

    return $user->toJson();

    return $user->toJson(JSON_PRETTY_PRINT);

Крім того, ви можете відлити модель або колекцію до рядка, який автоматично викликатиме`toJson`метод на моделі або колекції:

    $user = App\Models\User::find(1);

    return (string) $user;

Оскільки моделі та колекції перетворюються на JSON при передачі в рядок, ви можете повертати об'єкти Eloquent безпосередньо з маршрутів або контролерів програми:

    Route::get('users', function () {
        return App\Models\User::all();
    });

<a name="relationships"></a>

#### зв'язки

Коли модель Eloquent перетворюється на JSON, її завантажені зв'язки автоматично включатимуться як атрибути об'єкта JSON. Крім того, хоча Eloquent методи взаємозв'язку визначаються за допомогою "справи верблюда", атрибут JSON зв'язки буде "випадком змії".

<a name="hiding-attributes-from-json"></a>

## Приховування атрибутів з JSON

Іноді вам може знадобитися обмежити такі атрибути, як паролі, які входять до масиву вашої моделі або представлення JSON. Для цього додайте a`$hidden`властивість до вашої моделі:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be hidden for arrays.
         *
         * @var array
         */
        protected $hidden = ['password'];
    }

> {note} Під час приховування зв’язків використовуйте ім’я методу зв’язку.

Ви також можете використовувати`visible`властивість визначати білий список атрибутів, які повинні бути включені в масив вашої моделі та представлення JSON. Усі інші атрибути будуть приховані при перетворенні моделі на масив або JSON:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be visible in arrays.
         *
         * @var array
         */
        protected $visible = ['first_name', 'last_name'];
    }

<a name="temporarily-modifying-attribute-visibility"></a>

#### Тимчасове змінення видимості атрибутів

Якщо ви хочете зробити деякі типово приховані атрибути видимими на даному екземплярі моделі, ви можете використовувати`makeVisible`метод.`makeVisible`метод повертає екземпляр моделі для зручного ланцюжка методів:

    return $user->makeVisible('attribute')->toArray();

Аналогічним чином, якщо ви хочете зробити деякі типово видимі атрибути прихованими на даному екземплярі моделі, ви можете використовувати`makeHidden`метод.

    return $user->makeHidden('attribute')->toArray();

<a name="appending-values-to-json"></a>

## Додавання значень до JSON

Іноді, при передачі моделей до масиву або JSON, можливо, ви захочете додати атрибути, які не мають відповідного стовпця у вашій базі даних. Для цього спочатку визначте[аксесуар](/docs/{{version}}/eloquent-mutators)для значення:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get the administrator flag for the user.
         *
         * @return bool
         */
        public function getIsAdminAttribute()
        {
            return $this->attributes['admin'] === 'yes';
        }
    }

Після створення програми доступу додайте ім'я атрибута до`appends`властивість на моделі. Зверніть увагу, що на імена атрибутів зазвичай посилаються у "випадку змії", навіть незважаючи на те, що аксесуар визначається за допомогою "випадку верблюда":

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The accessors to append to the model's array form.
         *
         * @var array
         */
        protected $appends = ['is_admin'];
    }

Після додавання атрибута до`appends`списку, він буде включений як у масив моделі, так і у подання JSON. Атрибути в`appends`масив також буде поважати`visible`і`hidden`налаштування, налаштовані на моделі.

<a name="appending-at-run-time"></a>

#### Додавання під час роботи

Ви можете доручити одному екземпляру моделі додавати атрибути за допомогою`append`метод. Або ви можете використовувати`setAppends`метод, щоб замінити весь масив доданих властивостей для даного екземпляра моделі:

    return $user->append('is_admin')->toArray();

    return $user->setAppends(['is_admin'])->toArray();

<a name="date-serialization"></a>

## Серіалізація дати

<a name="customizing-the-default-date-format"></a>

#### Налаштування формату дати за замовчуванням

Ви можете налаштувати формат серіалізації за замовчуванням, замінивши`serializeDate`метод:

    /**
     * Prepare a date for array / JSON serialization.
     *
     * @param  \DateTimeInterface  $date
     * @return string
     */
    protected function serializeDate(DateTimeInterface $date)
    {
        return $date->format('Y-m-d');
    }

<a name="customizing-the-date-format-per-attribute"></a>

#### Налаштування формату дати на атрибут

Ви можете налаштувати формат серіалізації окремих Eloquent атрибутів дати, вказавши формат дати в[акторська декларація](/docs/{{version}}/eloquent-mutators#attribute-casting):

    protected $casts = [
        'birthday' => 'date:Y-m-d',
        'joined_at' => 'datetime:Y-m-d H:00',
    ];
