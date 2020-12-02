# Красномовний: Колекції

-   [Вступ](#introduction)
-   [Доступні методи](#available-methods)
-   [Спеціальні колекції](#custom-collections)

<a name="introduction"></a>

## Вступ

Всі множини результатів, повернені Eloquent, є екземплярами`Illuminate\Database\Eloquent\Collection`об'єкт, включаючи результати, отримані через`get`метод або доступ до яких здійснюється через стосунки. Об'єкт колекції Eloquent розширює Laravel[базова колекція](/docs/{{version}}/collections), тому він, природно, успадковує десятки методів, що використовуються для вільної роботи з базовим масивом красномовних моделей.

Усі колекції також слугують ітераторами, дозволяючи вам циклічно перебирати їх, ніби це прості масиви PHP:

    $users = App\Models\User::where('active', 1)->get();

    foreach ($users as $user) {
        echo $user->name;
    }

Однак колекції набагато потужніші, ніж масиви, і надають різноманітні операції картографування / зменшення, які можуть бути пов'язані за допомогою інтуїтивно зрозумілого інтерфейсу. Наприклад, видалимо всі неактивні моделі та зберемо ім’я для кожного користувача, що залишився:

    $users = App\Models\User::all();

    $names = $users->reject(function ($user) {
        return $user->active === false;
    })
    ->map(function ($user) {
        return $user->name;
    });

> {note} Хоча більшість красномовних методів збору повертають новий екземпляр красномовної колекції, файл`pluck`,`keys`,`zip`,`collapse`,`flatten`і`flip`повернення методів a[base collection](/docs/{{version}}/collections)інстанції. Так само, якщо a`map`операція повертає колекцію, яка не містить жодної красномовної моделі, вона буде автоматично передана до базової колекції.

<a name="available-methods"></a>

## Доступні методи

Всі колекції Eloquent розширюють базу[Колекція Laravel](/docs/{{version}}/collections#available-methods)об'єкт; отже, вони успадковують усі потужні методи, що надаються базовим класом колекцій.

Крім того,`Illuminate\Database\Eloquent\Collection`class надає набір методів, що допомагають керувати колекціями моделей. Більшість методів повертаються`Illuminate\Database\Eloquent\Collection`екземпляри; проте деякі методи повертають базу`Illuminate\Support\Collection`інстанції.

<style>
    #collection-method-list > p {
        column-count: 1; -moz-column-count: 1; -webkit-column-count: 1;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    #collection-method-list a {
        display: block;
    }
</style>

<div id="collection-method-list" markdown="1">

[містить](#method-contains)[різниця](#method-diff)[крім](#method-except)[знайти](#method-find)[свіжий](#method-fresh)[перетинаються](#method-intersect)[навантаження](#method-load)[loadMissing](#method-loadMissing)[модель Ключі](#method-modelKeys)[makeVisible](#method-makeVisible)[makeHidden](#method-makeHidden)[тільки](#method-only)[toQuery](#method-toquery)[унікальний](#method-unique)

</div>

<a name="method-contains"></a>

#### `contains($key, $operator = null, $value = null)`

`contains`метод може бути використаний, щоб визначити, чи міститься даний екземпляр моделі в колекції. Цей метод приймає первинний ключ або екземпляр моделі:

    $users->contains(1);

    $users->contains(User::find(1));

<a name="method-diff"></a>

#### `diff($items)`

`diff`метод повертає всі моделі, яких немає в даній колекції:

    use App\Models\User;

    $users = $users->diff(User::whereIn('id', [1, 2, 3])->get());

<a name="method-except"></a>

#### `except($keys)`

`except`метод повертає всі моделі, які не мають вказаних первинних ключів:

    $users = $users->except([1, 2, 3]);

<a name="method-find"></a>

#### `find($key)`{# collection-method .first-collection-method}

`find`метод знаходить модель, що має заданий первинний ключ. Якщо`$key`є екземпляром моделі,`find`спробує повернути модель, що відповідає первинному ключу. Якщо`$key`це масив ключів,`find`поверне всі моделі, які відповідають`$keys` using `whereIn()`:

    $users = User::all();

    $user = $users->find(1);

<a name="method-fresh"></a>

#### `fresh($with = [])`

`fresh`метод отримує з бази даних новий екземпляр кожної моделі в колекції. Крім того, будь-які вказані відносини будуть готові завантажувати:

    $users = $users->fresh();

    $users = $users->fresh('comments');

<a name="method-intersect"></a>

#### `intersect($items)`

`intersect`метод повертає всі моделі, які також присутні в даній колекції:

    use App\Models\User;

    $users = $users->intersect(User::whereIn('id', [1, 2, 3])->get());

<a name="method-load"></a>

#### `load($relations)`

`load`нетерплячий метод завантажує дані співвідношення для всіх моделей колекції:

    $users->load('comments', 'posts');

    $users->load('comments.author');

<a name="method-loadMissing"></a>

#### `loadMissing($relations)`

`loadMissing`метод прагне завантажувати дані відносини для всіх моделей у колекції, якщо відносини ще не завантажені:

    $users->loadMissing('comments', 'posts');

    $users->loadMissing('comments.author');

<a name="method-modelKeys"></a>

#### `modelKeys()`

`modelKeys`метод повертає первинні ключі для всіх моделей колекції:

    $users->modelKeys();

    // [1, 2, 3, 4, 5]

<a name="method-makeVisible"></a>

#### `makeVisible($attributes)`

`makeVisible`метод робить видимими атрибути, які зазвичай "приховані" для кожної моделі в колекції:

    $users = $users->makeVisible(['address', 'phone_number']);

<a name="method-makeHidden"></a>

#### `makeHidden($attributes)`

`makeHidden`метод приховує атрибути, які зазвичай "видно" для кожної моделі в колекції:

    $users = $users->makeHidden(['address', 'phone_number']);

<a name="method-only"></a>

#### `only($keys)`

`only`метод повертає всі моделі, що мають задані первинні ключі:

    $users = $users->only([1, 2, 3]);

<a name="method-toquery"></a>

#### `toQuery()`

`toQuery`метод повертає екземпляр Eloquent query builder, що містить`whereIn`обмеження на первинні ключі моделі колекції:

    $users = App\Models\User::where('status', 'VIP')->get();

    $users->toQuery()->update([
        'status' => 'Administrator',
    ]);

<a name="method-unique"></a>

#### `unique($key = null, $strict = false)`

`unique`метод повертає всі унікальні моделі в колекції. Будь-які моделі того ж типу з тим самим первинним ключем, що й інша модель у колекції, видаляються.

    $users = $users->unique();

<a name="custom-collections"></a>

## Спеціальні колекції

Якщо вам потрібно використовувати спеціальний`Collection`об'єкт із власними методами розширення, ви можете замінити`newCollection`метод на вашій моделі:

    <?php

    namespace App\Models;

    use App\Support\CustomCollection;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Create a new Eloquent Collection instance.
         *
         * @param  array  $models
         * @return \Illuminate\Database\Eloquent\Collection
         */
        public function newCollection(array $models = [])
        {
            return new CustomCollection($models);
        }
    }

Після того, як ви визначили a`newCollection`метод, ви отримаєте примірник власної колекції щоразу, коли Eloquent повертає a`Collection`екземпляр цієї моделі. Якщо ви хочете використовувати власну колекцію для кожної моделі у вашій програмі, вам слід замінити`newCollection`метод на базовому класі моделі, який розширений усіма вашими моделями.
