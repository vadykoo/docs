# Красномовний: Мутатори

-   [Вступ](#introduction)
-   [Аксесуари та мутатори](#accessors-and-mutators)
    -   [Визначення доступу](#defining-an-accessor)
    -   [Defining A Mutator](#defining-a-mutator)
-   [Мутатори дати](#date-mutators)
-   [Кастинг атрибутів](#attribute-casting)
    -   [Спеціальні актори](#custom-casts)
    -   [Масив та JSON-кастинг](#array-and-json-casting)
    -   [Дата Кастинг](#date-casting)
    -   [Час запиту](#query-time-casting)

<a name="introduction"></a>

## Вступ

Аксесуари та мутатори дозволяють форматувати значення атрибутів Eloquent під час отримання або встановлення їх на екземплярах моделі. Наприклад, ви можете використовувати[Шифрувач Laravel](/docs/{{version}}/encryption)щоб зашифрувати значення, коли воно зберігається в базі даних, а потім автоматично розшифрувати атрибут при доступі до нього за моделлю Eloquent.

На додаток до спеціальних аксесуарів та мутаторів, Eloquent може також автоматично передавати поля дати[Вуглець](https://github.com/briannesbitt/Carbon)екземпляри або навіть[перекинути текстові поля в JSON](#attribute-casting).

<a name="accessors-and-mutators"></a>

## Аксесуари та мутатори

<a name="defining-an-accessor"></a>

### Визначення доступу

Щоб визначити аксесуар, створіть файл`getFooAttribute`метод на вашій моделі де`Foo`- це "стовбурчаста" назва стовпця, до якого ви хочете отримати доступ. У цьому прикладі ми визначимо аксесуар для`first_name`атрибут. Eloquent автоматично намагається викликати аксесуар при спробі отримати значення`first_name`атрибут:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get the user's first name.
         *
         * @param  string  $value
         * @return string
         */
        public function getFirstNameAttribute($value)
        {
            return ucfirst($value);
        }
    }

Як бачите, вихідне значення стовпця передається користувачеві, що дозволяє маніпулювати та повертати значення. Щоб отримати доступ до значення доступу, ви можете отримати доступ до`first_name`атрибут на екземплярі моделі:

    $user = App\Models\User::find(1);

    $firstName = $user->first_name;

Ви також можете використовувати засоби доступу для повернення нових, обчислених значень із існуючих атрибутів:

    /**
     * Get the user's full name.
     *
     * @return string
     */
    public function getFullNameAttribute()
    {
        return "{$this->first_name} {$this->last_name}";
    }

> {tip} Якщо ви хочете, щоб ці обчислювані значення були додані до подання масиву / JSON вашої моделі,[вам потрібно буде їх додати](https://laravel.com/docs/{{version}}/eloquent-serialization#appending-values-to-json).

<a name="defining-a-mutator"></a>

### Визначення мутатора

Щоб визначити мутатора, визначте a`setFooAttribute`метод на вашій моделі де`Foo`- це "стовбурчаста" назва стовпця, до якого ви хочете отримати доступ. Отже, знову давайте визначимо мутатора для`first_name`атрибут. Цей мутатор буде автоматично викликаний при спробі встановити значення`first_name`атрибут на моделі:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Set the user's first name.
         *
         * @param  string  $value
         * @return void
         */
        public function setFirstNameAttribute($value)
        {
            $this->attributes['first_name'] = strtolower($value);
        }
    }

Мутатор отримає значення, яке встановлюється в атрибуті, що дозволяє вам маніпулювати значенням і встановлювати маніпульоване значення на внутрішній внутрішній моделі Eloquent`$attributes`майно. Так, наприклад, якщо ми намагаємося встановити`first_name`приписувати`Sally`:

    $user = App\Models\User::find(1);

    $user->first_name = 'Sally';

У цьому прикладі`setFirstNameAttribute`функція буде викликана зі значенням`Sally`. Потім мутатор застосує`strtolower`функцію до імені та встановіть її результуюче значення у внутрішній`$attributes`масив.

<a name="date-mutators"></a>

## Мутатори дати

За замовчуванням Eloquent конвертує файл`created_at`і`updated_at` columns to instances of [Вуглець](https://github.com/briannesbitt/Carbon), що розширює PHP`DateTime`класу та надає асортимент корисних методів. Ви можете додати додаткові атрибути дати, встановивши`$dates`властивість вашої моделі:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be mutated to dates.
         *
         * @var array
         */
        protected $dates = [
            'seen_at',
        ];
    }

> {tip} Ви можете вимкнути за замовчуванням`created_at`і`updated_at`позначки часу, встановлюючи загальнодоступні`$timestamps`властивість вашої моделі до`false`.

Коли стовпець вважається датою, ви можете встановити його значення як позначку часу UNIX, рядок дати (`Y-m-d`), рядок дати-часу або a`DateTime`/`Carbon`інстанції. Значення дати буде правильно перетворено та збережено у вашій базі даних:

    $user = App\Models\User::find(1);

    $user->deleted_at = now();

    $user->save();

Як зазначалося вище, при отриманні атрибутів, перелічених у вашому`$dates`власності, вони будуть автоматично передані в[Вуглець](https://github.com/briannesbitt/Carbon)екземплярів, що дозволяє використовувати будь-який із методів Carbon для своїх атрибутів:

    $user = App\Models\User::find(1);

    return $user->deleted_at->getTimestamp();

<a name="date-formats"></a>

#### Формати дат

За замовчуванням мітки часу мають формат`'Y-m-d H:i:s'`. Якщо вам потрібно налаштувати формат позначки часу, встановіть`$dateFormat`властивість на вашій моделі. Ця властивість визначає спосіб зберігання атрибутів дати в базі даних:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The storage format of the model's date columns.
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

<a name="attribute-casting"></a>

## Кастинг атрибутів

`$casts`властивість на вашій моделі забезпечує зручний метод перетворення атрибутів у загальні типи даних.`$casts`властивість має бути масивом, де ключ - це ім'я атрибута, що відтворюється, а значення - тип, до якого ви бажаєте привести стовпець. Підтримувані типи складання:`integer`,`real`,`float`,`double`,`decimal:<digits>`,`string`,`boolean`,`object`,`array`,`collection`,`date`,`datetime`,`timestamp`,`encrypted`,`encrypted:object`,`encrypted:array`, і`encrypted:collection`. При відливі до`decimal`, ви повинні визначити кількість цифр (`decimal:2`).

Щоб продемонструвати кастинг атрибутів, давайте додамо`is_admin`атрибут, який зберігається в нашій базі даних як ціле число (`0`або`1`) до логічного значення:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be cast.
         *
         * @var array
         */
        protected $casts = [
            'is_admin' => 'boolean',
        ];
    }

Тепер`is_admin`атрибут завжди буде передано до логічного значення, коли ви отримаєте до нього доступ, навіть якщо базове значення зберігається в базі даних як ціле число:

    $user = App\Models\User::find(1);

    if ($user->is_admin) {
        //
    }

> {note} Атрибути, які є`null`не буде кинутий. Крім того, ви ніколи не повинні визначати склад (або атрибут), який має те саме ім'я, що і зв'язок.

<a name="custom-casts"></a>

### Спеціальні актори

Laravel має безліч вбудованих, корисних типів актори; однак, іноді вам може знадобитися визначити власні типи акторського складу. Ви можете досягти цього, визначивши клас, який реалізує`CastsAttributes`інтерфейс.

Класи, що реалізують цей інтерфейс, повинні визначати a`get`і`set`метод.`get`Метод відповідає за перетворення вихідного значення з бази даних у додане значення, тоді як`set`метод повинен перетворити значення відлиття у вихідне значення, яке можна зберегти в базі даних. Як приклад, ми повторно реалізуємо вбудований`json`тип лиття як власний тип актори:

    <?php

    namespace App\Casts;

    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

    class Json implements CastsAttributes
    {
        /**
         * Cast the given value.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  mixed  $value
         * @param  array  $attributes
         * @return array
         */
        public function get($model, $key, $value, $attributes)
        {
            return json_decode($value, true);
        }

        /**
         * Prepare the given value for storage.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  array  $value
         * @param  array  $attributes
         * @return string
         */
        public function set($model, $key, $value, $attributes)
        {
            return json_encode($value);
        }
    }

Після того, як ви визначили власний тип лиття, ви можете додати його до атрибута моделі, використовуючи його назву класу:

    <?php

    namespace App\Models;

    use App\Casts\Json;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be cast.
         *
         * @var array
         */
        protected $casts = [
            'options' => Json::class,
        ];
    }

<a name="value-object-casting"></a>

#### Кастинг об’єкта значення

Ви не обмежуєтесь передачею значень до примітивних типів. Ви також можете передавати значення об'єктам. Визначення користувацьких приводів, які передають значення об'єктам, дуже схоже на приведення в примітивні типи; однак,`set`Метод повинен повертати масив пар ключ / значення, який буде використовуватися для встановлення необроблених значень, що зберігаються в моделі.

Як приклад, ми визначимо власний клас лиття, який відтворює кілька значень моделі в один`Address`об'єкт значення. Ми припустимо`Address`value має дві загальнодоступні властивості:`lineOne`і`lineTwo`:

    <?php

    namespace App\Casts;

    use App\Models\Address as AddressModel;
    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;
    use InvalidArgumentException;

    class Address implements CastsAttributes
    {
        /**
         * Cast the given value.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  mixed  $value
         * @param  array  $attributes
         * @return \App\Models\Address
         */
        public function get($model, $key, $value, $attributes)
        {
            return new AddressModel(
                $attributes['address_line_one'],
                $attributes['address_line_two']
            );
        }

        /**
         * Prepare the given value for storage.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  \App\Models\Address  $value
         * @param  array  $attributes
         * @return array
         */
        public function set($model, $key, $value, $attributes)
        {
            if (! $value instanceof AddressModel) {
                throw new InvalidArgumentException('The given value is not an Address instance.');
            }

            return [
                'address_line_one' => $value->lineOne,
                'address_line_two' => $value->lineTwo,
            ];
        }
    }

Під час трансляції до об'єктів значення всі зміни, внесені до об'єкта значення, будуть автоматично синхронізовані з моделлю перед збереженням моделі:

    $user = App\Models\User::find(1);

    $user->address->lineOne = 'Updated Address Value';

    $user->save();

> {tip} Якщо ви плануєте серіалізувати ваші красномовні моделі, що містять об'єкти значень, у форматі JSON або масиви, вам слід застосувати`Illuminate\Contracts\Support\Arrayable`і`JsonSerializable`інтерфейси на об'єкті значення.

<a name="array-json-serialization"></a>

#### Серіалізація масиву / JSON

Коли модель Eloquent перетворюється на масив або JSON за допомогою`toArray`Метод, ваші користувацькі об'єкти значення додавання, як правило, будуть серіалізовані, доки вони реалізують`Illuminate\Contracts\Support\Arrayable`і`JsonSerializable`інтерфейси. Однак при використанні об'єктів значення, наданих сторонніми бібліотеками, можливо, у вас не буде можливості додавати ці інтерфейси до об'єкта.

Таким чином, ви можете вказати, що ваш власний клас приведення буде відповідати за серіалізацію об'єкта значення. Для цього у вашому користувацькому складі класу має бути реалізовано`Illuminate\Contracts\Database\Eloquent\SerializesCastableAttributes`інтерфейс. Цей інтерфейс стверджує, що ваш клас повинен містити файл`serialize`метод, який повинен повернути серіалізовану форму вашого об’єкта значення:

    /**
     * Get the serialized representation of the value.
     *
     * @param  \Illuminate\Database\Eloquent\Model  $model
     * @param  string  $key
     * @param  mixed  $value
     * @param  array  $attributes
     * @return mixed
     */
    public function serialize($model, string $key, $value, array $attributes)
    {
        return (string) $value;
    }

<a name="inbound-casting"></a>

#### Вхідний кастинг

Іноді вам може знадобитися написати власний привід, який перетворює лише значення, встановлені в моделі, і не виконує жодних операцій, коли атрибути отримуються з моделі. Класичний приклад лише вхідного складу - це хешування. Тільки вхідні спеціальні зливки повинні реалізовувати`CastsInboundAttributes`інтерфейс, який вимагає лише`set`метод, який слід визначити.

    <?php

    namespace App\Casts;

    use Illuminate\Contracts\Database\Eloquent\CastsInboundAttributes;

    class Hash implements CastsInboundAttributes
    {
        /**
         * The hashing algorithm.
         *
         * @var string
         */
        protected $algorithm;

        /**
         * Create a new cast class instance.
         *
         * @param  string|null  $algorithm
         * @return void
         */
        public function __construct($algorithm = null)
        {
            $this->algorithm = $algorithm;
        }

        /**
         * Prepare the given value for storage.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  array  $value
         * @param  array  $attributes
         * @return string
         */
        public function set($model, $key, $value, $attributes)
        {
            return is_null($this->algorithm)
                        ? bcrypt($value)
                        : hash($this->algorithm, $value);
        }
    }

<a name="cast-parameters"></a>

#### Параметри лиття

Під час приєднання користувацького приведення до моделі параметри приведення можуть бути вказані, відокремлюючи їх від імені класу, використовуючи a`:`символом та кількома параметрами, що розмежовують комами. Параметри передаються конструктору класу актори:

    /**
     * The attributes that should be cast.
     *
     * @var array
     */
    protected $casts = [
        'secret' => Hash::class.':sha256',
    ];

<a name="castables"></a>

#### Кастеблі

Замість того, щоб приєднувати спеціальний привід до вашої моделі, ви можете альтернативно приєднати клас, який реалізує`Illuminate\Contracts\Database\Eloquent\Castable`інтерфейс:

    protected $casts = [
        'address' => \App\Models\Address::class,
    ];

Об'єкти, що реалізують`Castable`інтерфейс повинен визначати a`castUsing`метод, який повертає ім'я класу спеціального класу заклинателя, який відповідає за трансляцію до та з`Castable`клас:

    <?php

    namespace App\Models;

    use Illuminate\Contracts\Database\Eloquent\Castable;
    use App\Casts\Address as AddressCast;

    class Address implements Castable
    {
        /**
         * Get the name of the caster class to use when casting from / to this cast target.
         *
         * @param  array  $arguments
         * @return string
         */
        public static function castUsing(array $arguments)
        {
            return AddressCast::class;
        }
    }

При використанні`Castable`класів, ви все ще можете наводити аргументи в`$casts`визначення. Аргументи будуть передані безпосередньо до класу заклинача:

    protected $casts = [
        'address' => \App\Models\Address::class.':argument',
    ];

<a name="array-and-json-casting"></a>

### Масив та JSON-кастинг

`array`тип приведення особливо корисний при роботі зі стовпцями, які зберігаються як серіалізовані JSON. Наприклад, якщо у вашій базі даних є файл`JSON`або`TEXT`тип поля, що містить серіалізований JSON, додаючи`array`прив'язка до цього атрибуту автоматично десеріалізує атрибут до масиву PHP, коли ви отримуєте доступ до нього на вашій красномовній моделі:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be cast.
         *
         * @var array
         */
        protected $casts = [
            'options' => 'array',
        ];
    }

Після визначення акторського складу ви можете отримати доступ до`options`атрибут, і він буде автоматично десеріалізований з JSON у масив PHP. Коли ви встановлюєте значення`options`атрибут, даний масив буде автоматично серіалізований назад у JSON для зберігання:

    $user = App\Models\User::find(1);

    $options = $user->options;

    $options['key'] = 'value';

    $user->options = $options;

    $user->save();

Щоб оновити одне поле атрибута JSON з більш стислим синтаксисом, ви можете використовувати файл`->`оператор:

    $user = App\Models\User::find(1);

    $user->update(['options->key' => 'value']);

<a name="date-casting"></a>

### Дата Кастинг

При використанні`date` or `datetime`тип касту, ви можете вказати формат дати. Цей формат буде використовуватися, коли[модель серіалізується до масиву або JSON](/docs/{{version}}/eloquent-serialization):

    /**
     * The attributes that should be cast.
     *
     * @var array
     */
    protected $casts = [
        'created_at' => 'datetime:Y-m-d',
    ];

<a name="query-time-casting"></a>

### Час запиту

Іноді вам може знадобитися застосувати закиди під час виконання запиту, наприклад, при виборі вихідного значення з таблиці. Наприклад, розглянемо такий запит:

    use App\Models\Post;
    use App\Models\User;

    $users = User::select([
        'users.*',
        'last_posted_at' => Post::selectRaw('MAX(created_at)')
                ->whereColumn('user_id', 'users.id')
    ])->get();

`last_posted_at` attribute on the results of this query will be a raw string. It would be convenient if we could apply a `date`прив'язка до цього атрибута під час виконання запиту. Для цього ми можемо використовувати`withCasts`метод:

    $users = User::select([
        'users.*',
        'last_posted_at' => Post::selectRaw('MAX(created_at)')
                ->whereColumn('user_id', 'users.id')
    ])->withCasts([
        'last_posted_at' => 'datetime'
    ])->get();
