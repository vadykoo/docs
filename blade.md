# Лезо шаблони

-   [Вступ](#introduction)
-   [Спадщина шаблону](#template-inheritance)
    -   [Визначення макета](#defining-a-layout)
    -   [Розширення макета](#extending-a-layout)
-   [Відображення даних](#displaying-data)
    -   [Blade & JavaScript Frameworks](#blade-and-javascript-frameworks)
-   [Структури управління](#control-structures)
    -   [Якщо Заяви](#if-statements)
    -   [Заяви про перемикання](#switch-statements)
    -   [Петлі](#loops)
    -   [Змінна петлі](#the-loop-variable)
    -   [Коментарі](#comments)
    -   [PHP](#php)
    -   [`@once`Директива](#the-once-directive)
-   [Форми](#forms)
    -   [Поле CSRF](#csrf-field)
    -   [Поле методу](#method-field)
    -   [Помилки перевірки](#validation-errors)
-   [Компоненти](#components)
    -   [Відображення компонентів](#displaying-components)
    -   [Передача даних компонентам](#passing-data-to-components)
    -   [Управління атрибутами](#managing-attributes)
    -   [Слоти](#slots)
    -   [Вбудовані перегляди компонентів](#inline-component-views)
    -   [Анонімні компоненти](#anonymous-components)
    -   [Динамічні компоненти](#dynamic-components)
-   [Включаючи підпрограми](#including-subviews)
    -   [Візуалізація подань для колекцій](#rendering-views-for-collections)
-   [Стеки](#stacks)
-   [Сервісна ін’єкція](#service-injection)
-   [Extending Blade](#extending-blade)
    -   [Спеціальні виписки If](#custom-if-statements)

<a name="introduction"></a>

## Вступ

Blade - це простий, але потужний двигун для створення шаблонів, що постачається разом із Laravel. На відміну від інших популярних PHP-шаблоністів, Blade не обмежує використання звичайного PHP-коду у ваших поданнях. Насправді всі подання Blade компілюються у звичайний PHP-код і кешуються, доки їх не модифікують, тобто Blade додає по суті нульові накладні витрати у вашу програму. Файли перегляду леза використовують`.blade.php`файлу і зазвичай зберігаються в`resources/views`каталог.

<a name="template-inheritance"></a>

## Спадщина шаблону

<a name="defining-a-layout"></a>

### Визначення макета

Дві основні переваги використання Blade - це_успадкування шаблону_і_розділи_. Для початку давайте розглянемо простий приклад. Спочатку ми розглянемо макет сторінки. Оскільки більшість веб-додатків підтримують однаковий загальний макет на різних сторінках, зручно визначити цей макет як єдиний вигляд Blade:

    <!-- Stored in resources/views/layouts/app.blade.php -->

    <html>
        <head>
            <title>App Name - @yield('title')</title>
        </head>
        <body>
            @section('sidebar')
                This is the master sidebar.
            @show

            <div class="container">
                @yield('content')
            </div>
        </body>
    </html>

Як бачите, цей файл містить типову розмітку HTML. Однак зверніть увагу на`@section`і`@yield`директиви.`@section`Директива, як випливає з назви, визначає розділ змісту, тоді як`@yield`Директива використовується для відображення вмісту даного розділу.

Тепер, коли ми визначили макет для нашої програми, давайте визначимо дочірню сторінку, яка успадковує макет.

<a name="extending-a-layout"></a>

### Розширення макета

Визначаючи дитячий погляд, використовуйте лезо`@extends`директива, щоб вказати, який макет дочірній вигляд повинен "успадкувати". Представлення, які розширюють макет Blade, можуть вливати вміст у розділи макета за допомогою`@section`директиви. Пам’ятайте, як видно з прикладу вище, вміст цих розділів буде відображатися в макеті за допомогою`@yield`:

    <!-- Stored in resources/views/child.blade.php -->

    @extends('layouts.app')

    @section('title', 'Page Title')

    @section('sidebar')
        @@parent

        <p>This is appended to the master sidebar.</p>
    @endsection

    @section('content')
        <p>This is my body content.</p>
    @endsection

У цьому прикладі`sidebar`розділ використовує`@@parent`директива додавати (а не перезаписувати) вміст на бічну панель макета.`@@parent`Директива буде замінена вмістом макета, коли подається подання.

> {tip} На відміну від попереднього прикладу, це`sidebar`розділ закінчується на`@endsection`замість`@show`.`@endsection`Директива визначатиме лише розділ while`@show`визначить і**негайно врожай**розділу.

`@yield`Директива також приймає значення за замовчуванням як другий параметр. Це значення буде відображено, якщо вказаний розділ не визначений:

    @yield('content', View::make('view.name'))

Види леза можуть повертатися з маршрутів за допомогою глобального`view`помічник:

    Route::get('blade', function () {
        return view('child');
    });

<a name="displaying-data"></a>

## Відображення даних

Ви можете відображати дані, передані у ваші подання Blade, обертаючи змінну фігурними дужками. Наприклад, враховуючи такий маршрут:

    Route::get('greeting', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

Ви можете відобразити вміст`name`така змінна:

    Hello, {{ $name }}.

> {tip} Лезо`{{ }}`виписки автоматично надсилаються через PHP`htmlspecialchars`функція для запобігання атакам XSS.

Ви не обмежуєтесь відображенням вмісту змінних, переданих у подання. Ви також можете повторити результати будь-якої функції PHP. Насправді ви можете помістити будь-який PHP-код, який хочете, всередину інструкції Blade echo:

    The current UNIX timestamp is {{ time() }}.

<a name="displaying-unescaped-data"></a>

#### Відображення незахищених даних

За замовчуванням Blade`{{ }}`виписки автоматично надсилаються через PHP`htmlspecialchars` function to prevent XSS attacks. If you do not want your data to be escaped, you may use the following syntax:

    Hello, {!! $name !!}.

> {note} Будьте дуже обережні, повторюючи контент, який надають користувачі вашої програми. Завжди використовуйте захищений синтаксис подвійних фігурних дужок, щоб запобігти атакам XSS під час відображення даних, наданих користувачем.

<a name="rendering-json"></a>

#### Візуалізація JSON

Іноді ви можете передати масив своєму поданню з наміром зробити його як JSON, щоб ініціалізувати змінну JavaScript. Наприклад:

    <script>
        var app = <?php echo json_encode($array); ?>;
    </script>

Однак замість виклику вручну`json_encode`, ви можете використовувати`@json`Директива леза.`@json`Директива приймає ті самі аргументи, що і PHP`json_encode`функція. За замовчуванням`@json`директива викликає`json_encode`функція за допомогою`JSON_HEX_TAG`,`JSON_HEX_APOS`,`JSON_HEX_AMP`, і`JSON_HEX_QUOT`прапори:

    <script>
        var app = @json($array);

        var app = @json($array, JSON_PRETTY_PRINT);
    </script>

> {note} Вам слід використовувати лише`@json`директива відображати існуючі змінні як JSON. Шаблон Blade базується на регулярних виразах, і спроби передати складний вираз директиві можуть спричинити несподівані помилки.

<a name="html-entity-encoding"></a>

#### Кодування сутності HTML

За замовчуванням Blade (і Laravel`e`helper) буде подвійно кодувати HTML-сутності. Якщо ви хочете відключити подвійне кодування, зателефонуйте на`Blade::withoutDoubleEncoding`метод з`boot`метод вашого`AppServiceProvider`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Blade::withoutDoubleEncoding();
        }
    }

<a name="blade-and-javascript-frameworks"></a>

### Blade & JavaScript Frameworks

Оскільки багато фреймворків JavaScript також використовують "фігурні" фігурні дужки, щоб вказати, що даний вираз повинен відображатися у браузері, ви можете використовувати`@`символ, який повідомляє механізм рендеринга Blade про вираз, повинен залишатися незайманим. Наприклад:

    <h1>Laravel</h1>

    Hello, @{{ name }}.

У цьому прикладі`@`символ буде видалений Blade; однак,`{{ name }}`вираз залишиться незайманим механізмом Blade, що дозволить його натомість рендерити вашим фреймворком JavaScript.

`@`символ також може використовуватися для виходу з директив Blade:

    {{-- Blade --}}
    @@json()

    <!-- HTML output -->
    @json()

<a name="the-at-verbatim-directive"></a>

#### `@verbatim`Директива

Якщо ви відображаєте змінні JavaScript у великій частині свого шаблону, ви можете обернути HTML у`@verbatim`директиви, так що вам не потрібно префіксувати кожен оператор Blade echo префіксом`@`символ:

    @verbatim
        <div class="container">
            Hello, {{ name }}.
        </div>
    @endverbatim

<a name="control-structures"></a>

## Структури управління

На додаток до успадкування шаблону та відображення даних, Blade також пропонує зручні ярлики для загальних структур управління PHP, таких як умовні оператори та цикли. Ці комбінації клавіш забезпечують дуже чіткий, стислий спосіб роботи зі структурами управління PHP, залишаючись при цьому знайомими для своїх колег PHP.

<a name="if-statements"></a>

### Якщо Заяви

Ви можете побудувати`if`заяви з використанням`@if`,`@elseif`,`@else`, і`@endif`директиви. Ці директиви функціонують ідентично своїм аналогам PHP:

    @if (count($records) === 1)
        I have one record!
    @elseif (count($records) > 1)
        I have multiple records!
    @else
        I don't have any records!
    @endif

Для зручності Blade також пропонує`@unless`директива:

    @unless (Auth::check())
        You are not signed in.
    @endunless

На додаток до вже обговорених умовних директив,`@isset`і`@empty`директиви можуть використовуватися як зручні ярлики для відповідних функцій PHP:

    @isset($records)
        // $records is defined and is not null...
    @endisset

    @empty($records)
        // $records is "empty"...
    @endempty

<a name="authentication-directives"></a>

#### Директиви щодо автентифікації

`@auth`і`@guest`директиви можуть бути використані для швидкого визначення того, чи поточний користувач аутентифікований чи є гостем:

    @auth
        // The user is authenticated...
    @endauth

    @guest
        // The user is not authenticated...
    @endguest

Якщо потрібно, ви можете вказати[охорона автентифікації](/docs/{{version}}/authentication)це слід перевірити при використанні`@auth`і`@guest`директиви:

    @auth('admin')
        // The user is authenticated...
    @endauth

    @guest('admin')
        // The user is not authenticated...
    @endguest

<a name="section-directives"></a>

#### Розділові директиви

Ви можете перевірити, чи містить розділ вміст, використовуючи`@hasSection`директива:

    @hasSection('navigation')
        <div class="pull-right">
            @yield('navigation')
        </div>

        <div class="clearfix"></div>
    @endif

Ви можете використовувати`sectionMissing`директива, щоб визначити, чи не має розділ змісту:

    @sectionMissing('navigation')
        <div class="pull-right">
            @include('default-navigation')
        </div>
    @endif

<a name="environment-directives"></a>

#### Директиви про довкілля

Ви можете перевірити, чи працює програма у виробничому середовищі, використовуючи`@production`директива:

    @production
        // Production specific content...
    @endproduction

Або ви можете визначити, чи працює програма в певному середовищі, використовуючи`@env`директива:

    @env('staging')
        // The application is running in "staging"...
    @endenv

    @env(['staging', 'production'])
        // The application is running in "staging" or "production"...
    @endenv

<a name="switch-statements"></a>

### Заяви про перемикання

Інструкції комутатора можна побудувати за допомогою`@switch`,`@case`,`@break`,`@default`і`@endswitch`директиви:

    @switch($i)
        @case(1)
            First case...
            @break

        @case(2)
            Second case...
            @break

        @default
            Default case...
    @endswitch

<a name="loops"></a>

### Петлі

На додаток до умовних операторів, Blade надає прості вказівки для роботи з цикловими структурами PHP. Знову ж таки, кожна з цих директив функціонує ідентично своїм аналогам PHP:

    @for ($i = 0; $i < 10; $i++)
        The current value is {{ $i }}
    @endfor

    @foreach ($users as $user)
        <p>This is user {{ $user->id }}</p>
    @endforeach

    @forelse ($users as $user)
        <li>{{ $user->name }}</li>
    @empty
        <p>No users</p>
    @endforelse

    @while (true)
        <p>I'm looping forever.</p>
    @endwhile

> {tip} Під час циклу ви можете використовувати[змінна циклу](#the-loop-variable)отримати цінну інформацію про цикл, наприклад, чи перебуваєте ви на першій чи останній ітерації через цикл.

При використанні циклів ви також можете закінчити цикл або пропустити поточну ітерацію:

    @foreach ($users as $user)
        @if ($user->type == 1)
            @continue
        @endif

        <li>{{ $user->name }}</li>

        @if ($user->number == 5)
            @break
        @endif
    @endforeach

Ви також можете включити умову з декларацією директиви в один рядок:

    @foreach ($users as $user)
        @continue($user->type == 1)

        <li>{{ $user->name }}</li>

        @break($user->number == 5)
    @endforeach

<a name="the-loop-variable"></a>

### Змінна петлі

Під час циклу a`$loop`змінна буде доступна всередині вашого циклу. Ця змінна надає доступ до деяких корисних бітів інформації, таких як індекс поточного циклу, а також, є це перша або остання ітерація через цикл:

    @foreach ($users as $user)
        @if ($loop->first)
            This is the first iteration.
        @endif

        @if ($loop->last)
            This is the last iteration.
        @endif

        <p>This is user {{ $user->id }}</p>
    @endforeach

Якщо ви перебуваєте у вкладеному циклі, ви можете отримати доступ до батьківського циклу`$loop`змінна через`parent`властивість:

    @foreach ($users as $user)
        @foreach ($user->posts as $post)
            @if ($loop->parent->first)
                This is first iteration of the parent loop.
            @endif
        @endforeach
    @endforeach

`$loop`змінна також містить безліч інших корисних властивостей:

| Власність          | Опис                                                      |
| ------------------ | --------------------------------------------------------- |
| `$loop->index`     | Індекс ітерації поточного циклу (починається з 0).        |
| `$loop->iteration` | Ітерація поточного циклу (починається з 1).               |
| `$loop->remaining` | Ітерації, що залишаються в циклі.                         |
| `$loop->count`     | Загальна кількість елементів у масиві, який повторюється. |
| `$loop->first`     | Whether this is the first iteration through the loop.     |
| `$loop->last`      | Чи є це останньою ітерацією циклу.                        |
| `$loop->even`      | Чи це рівна ітерація через цикл.                          |
| `$loop->odd`       | Чи є це непарною ітерацією через цикл.                    |
| `$loop->depth`     | Рівень вкладеності поточного циклу.                       |
| `$loop->parent`    | Перебуваючи у вкладеному циклі, батьківська змінна циклу. |

<a name="comments"></a>

### Коментарі

Blade також дозволяє визначати коментарі у своїх поглядах. Однак, на відміну від коментарів HTML, коментарі Blade не включаються в HTML, який повертає ваша програма:

    {{-- This comment will not be present in the rendered HTML --}}

<a name="php"></a>

### PHP

У деяких ситуаціях корисно вбудовувати PHP-код у свої подання. Ви можете використовувати Лезо`@php`для виконання блоку простого PHP у вашому шаблоні:

    @php
        //
    @endphp

> {рада} Хоча Blade надає цю функцію, її часто використання може свідчити про те, що у вашому шаблоні закладено занадто багато логіки.

<a name="the-once-directive"></a>

### `@once`Директива

`@once`Директива дозволяє визначити частину шаблону, яка буде оцінюватися лише один раз за цикл рендерингу. Це може бути корисно для введення заданого фрагмента JavaScript у заголовок сторінки за допомогою[стеки](#stacks). Наприклад, якщо ви рендеріть заданий[компонент](#components)в межах циклу, можливо, ви захочете натиснути JavaScript у заголовок лише під час першого відображення компонента:

    @once
        @push('scripts')
            <script>
                // Your custom JavaScript...
            </script>
        @endpush
    @endonce

<a name="forms"></a>

## Форми

<a name="csrf-field"></a>

### Поле CSRF

Кожного разу, коли ви визначаєте форму HTML у своїй програмі, ви повинні включити приховане поле маркера CSRF у форму, щоб це[захист CSRF](https://laravel.com/docs/{{version}}/csrf)проміжне програмне забезпечення може перевірити запит. Ви можете використовувати`@csrf`Директива Blade для створення поля маркера:

    <form method="POST" action="/profile">
        @csrf

        ...
    </form>

<a name="method-field"></a>

### Поле методу

Оскільки форми HTML неможливо зробити`PUT`,`PATCH`, або`DELETE`запити, вам потрібно буде додати прихований`_method`поле для підробки цих HTTP-дієслів.`@method`Директива Blade може створити для вас це поле:

    <form action="/foo/bar" method="POST">
        @method('PUT')

        ...
    </form>

<a name="validation-errors"></a>

### Помилки перевірки

`@error`директива може бути використана для швидкої перевірки, якщо[повідомлення про помилку перевірки](/docs/{{version}}/validation#quick-displaying-the-validation-errors)існують для даного атрибута. В межах`@error`директиви, ви можете повторити`$message`змінна для відображення повідомлення про помилку:

    <!-- /resources/views/post/create.blade.php -->

    <label for="title">Post Title</label>

    <input id="title" type="text" class="@error('title') is-invalid @enderror">

    @error('title')
        <div class="alert alert-danger">{{ $message }}</div>
    @enderror

Ви можете пройти[назва конкретного пакета помилок](/docs/{{version}}/validation#named-error-bags)як другий параметр до`@error`директива для отримання повідомлень про помилки перевірки на сторінках, що містять кілька форм:

    <!-- /resources/views/auth.blade.php -->

    <label for="email">Email address</label>

    <input id="email" type="email" class="@error('email', 'login') is-invalid @enderror">

    @error('email', 'login')
        <div class="alert alert-danger">{{ $message }}</div>
    @enderror

<a name="components"></a>

## Компоненти

Компоненти та слоти забезпечують подібні переваги для розділів та макетів; проте, деяким може бути ментальна модель компонентів та слотів легшою для розуміння. Існує два підходи до написання компонентів: компоненти на основі класу та анонімні компоненти.

Щоб створити компонент на основі класу, ви можете використовувати`make:component`Ремісниче командування. Щоб проілюструвати, як використовувати компоненти, ми створимо просту`Alert`компонент.`make:component`команда розмістить компонент у`App\View\Components`каталог:

    php artisan make:component Alert

`make:component`команда також створить шаблон подання для компонента. Вигляд буде розміщено в`resources/views/components`каталог.

Ви також можете створювати компоненти в підкаталогах:

    php artisan make:component Forms/Input

Команда вище створить файл`Input`компонент в`App\View\Components\Forms`і подання буде розміщено в`resources/views/components/forms`каталог.

<a name="manually-registering-package-components"></a>

#### Реєстрація компонентів пакунку вручну

Під час написання компонентів для власного додатка компоненти автоматично виявляються в`app/View/Components`каталог і`resources/views/components`каталог.

Однак, якщо ви створюєте пакет, який використовує компоненти Blade, вам потрібно буде вручну зареєструвати клас свого компонента та його псевдонім тегу HTML. Зазвичай слід реєструвати свої компоненти в`boot`метод постачальника послуг вашого пакета:

    use Illuminate\Support\Facades\Blade;

    /**
     * Bootstrap your package's services.
     */
    public function boot()
    {
        Blade::component('package-alert', AlertComponent::class);
    }

Після того, як ваш компонент буде зареєстровано, він може бути відтворений за допомогою його псевдоніма:

    <x-package-alert/>

Крім того, ви можете використовувати`componentNamespace`метод для автоматичного завантаження класів компонентів за домовленістю. Наприклад, a`Nightshade`пакет може мати`Calendar`і`ColorPicker`компоненти, що знаходяться в межах`Package\Views\Components`простір імен:

    use Illuminate\Support\Facades\Blade;

    /**
     * Bootstrap your package's services.
     */
    public function boot()
    {
        Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
    }

Це дозволить використовувати компоненти пакету їхнім простором імен постачальника за допомогою`package-name::`синтаксис:

    <x-nightshade::calendar />
    <x-nightshade::color-picker />

Блейд автоматично виявляє клас, який пов'язаний з цим компонентом, за допомогою паскаля-регістру імені компонента. Підкаталоги також підтримуються за допомогою позначення "крапка".

<a name="displaying-components"></a>

### Відображення компонентів

Для відображення компонента ви можете використовувати тег компонента Blade в одному зі своїх шаблонів Blade. Теги компонентів леза починаються з рядка`x-`а потім ім’я справи шашлику класу компонента:

    <x-alert/>

    <x-user-profile/>

Якщо клас компонента вкладений глибше в`App\View\Components`каталог, ви можете використовувати`.`символ для позначення вкладеності в каталог. Наприклад, якщо ми припустимо, що компонент знаходиться в`App\View\Components\Inputs\Button.php`, ми можемо зробити це так:

    <x-inputs.button/>

<a name="passing-data-to-components"></a>

### Передача даних компонентам

Ви можете передавати дані компонентам Blade за допомогою атрибутів HTML. Твердо закодовані примітивні значення можуть передаватися компоненту за допомогою простих атрибутів HTML. PHP-вирази та змінні повинні передаватися компоненту через атрибути, що використовують`:`символ як префікс:

    <x-alert type="error" :message="$message"/>

Ви повинні визначити необхідні дані компонента в його конструкторі класів. Усі загальнодоступні властивості компонента автоматично будуть доступні для перегляду компонента. Не потрібно передавати дані до подання з компонента`render`метод:

    <?php

    namespace App\View\Components;

    use Illuminate\View\Component;

    class Alert extends Component
    {
        /**
         * The alert type.
         *
         * @var string
         */
        public $type;

        /**
         * The alert message.
         *
         * @var string
         */
        public $message;

        /**
         * Create the component instance.
         *
         * @param  string  $type
         * @param  string  $message
         * @return void
         */
        public function __construct($type, $message)
        {
            $this->type = $type;
            $this->message = $message;
        }

        /**
         * Get the view / contents that represent the component.
         *
         * @return \Illuminate\View\View|\Closure|string
         */
        public function render()
        {
            return view('components.alert');
        }
    }

Коли ваш компонент відображається, ви можете відображати вміст загальнодоступних змінних вашого компонента, повторюючи змінні за іменем:

    <div class="alert alert-{{ $type }}">
        {{ $message }}
    </div>

<a name="casing"></a>

#### Кожух

Аргументи конструктора компонентів слід вказувати за допомогою`camelCase`, поки`kebab-case`слід використовувати при посиланні на імена аргументів у ваших атрибутах HTML. Наприклад, враховуючи такий конструктор компонентів:

    /**
     * Create the component instance.
     *
     * @param  string  $alertType
     * @return void
     */
    public function __construct($alertType)
    {
        $this->alertType = $alertType;
    }

`$alertType`аргумент може бути поданий так:

    <x-alert alert-type="danger" />

<a name="component-methods"></a>

#### Компонентні методи

Окрім загальнодоступних змінних, доступних для вашого шаблону компонента, також можуть бути виконані будь-які відкриті методи на компоненті. Наприклад, уявіть компонент, який має`isSelected`метод:

    /**
     * Determine if the given option is the current selected option.
     *
     * @param  string  $option
     * @return bool
     */
    public function isSelected($option)
    {
        return $option === $this->selected;
    }

Ви можете виконати цей метод із шаблону компонента, викликаючи змінну, що відповідає імені методу:

    <option {{ $isSelected($value) ? 'selected="selected"' : '' }} value="{{ $value }}">
        {{ $label }}
    </option>

<a name="using-attributes-slots-inside-the-class"></a>

#### Використання атрибутів та слотів всередині класу

Компоненти леза також дозволяють отримати доступ до імені компонента, атрибутів та слота всередині методу візуалізації класу. Однак для того, щоб отримати доступ до цих даних, вам слід повернути Закриття від компонента`render`метод. Закриття отримає a`$data`масив як єдиний аргумент:

    /**
     * Get the view / contents that represent the component.
     *
     * @return \Illuminate\View\View|\Closure|string
     */
    public function render()
    {
        return function (array $data) {
            // $data['componentName'];
            // $data['attributes'];
            // $data['slot'];

            return '<div>Components content</div>';
        };
    }

`componentName`дорівнює імені, що використовується в тегу HTML після`x-`префікс. Тому`<x-alert />`'s`componentName`буде`alert`.`attributes`Елемент буде містити всі атрибути, які були присутні в тезі HTML.`slot`елементом є`Illuminate\Support\HtmlString`екземпляр із вмістом слота від компонента.

Закриття має повернути рядок. Якщо повернутий рядок відповідає існуючому поданню, цей вигляд буде відтворений; в іншому випадку повернутий рядок буде оцінено як вбудований вигляд леза.

<a name="additional-dependencies"></a>

#### Додаткові залежності

Якщо ваш компонент вимагає залежностей від Laravel's[службовий контейнер](/docs/{{version}}/container), ви можете перерахувати їх перед будь-яким з атрибутів даних компонента, і вони автоматично вводяться контейнером:

    use App\Services\AlertCreator

    /**
     * Create the component instance.
     *
     * @param  \App\Services\AlertCreator  $creator
     * @param  string  $type
     * @param  string  $message
     * @return void
     */
    public function __construct(AlertCreator $creator, $type, $message)
    {
        $this->creator = $creator;
        $this->type = $type;
        $this->message = $message;
    }

<a name="managing-attributes"></a>

### Управління атрибутами

Ми вже розглядали, як передавати атрибути даних компоненту; однак іноді може знадобитися вказати додаткові атрибути HTML, наприклад`class`, які не є частиною даних, необхідних для функціонування компонента. Зазвичай ці додаткові атрибути потрібно передавати кореневому елементу шаблону компонента. Наприклад, уявіть, що ми хочемо зробити файл`alert`компонент так:

    <x-alert type="error" :message="$message" class="mt-4"/>

Всі атрибути, які не є частиною конструктора компонента, будуть автоматично додані до "мішка атрибутів" компонента. Цей пакет атрибутів автоматично стає доступним компоненту через`$attributes`змінна. Усі атрибути можуть бути відтворені в компоненті, повторюючи цю змінну:

    <div {{ $attributes }}>
        <!-- Component Content -->
    </div>

> {note} Використовуючи директиви, такі як`@env`безпосередньо на компонент зараз не підтримується.

<a name="default-merged-attributes"></a>

#### Атрибути за замовчуванням / об’єднані

Іноді вам може знадобитися вказати значення за замовчуванням для атрибутів або об'єднати додаткові значення в деякі атрибути компонента. Для цього ви можете використовувати атрибут bag's`merge`метод:

    <div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
        {{ $message }}
    </div>

Якщо ми припустимо, що цей компонент використовується так:

    <x-alert type="error" :message="$message" class="mb-4"/>

Остаточний, відображений HTML компонента буде виглядати так:

    <div class="alert alert-error mb-4">
        <!-- Contents of the $message variable -->
    </div>

<a name="non-class-attribute-merging"></a>

#### Некласове об’єднання атрибутів

При об'єднанні атрибутів, які не є`class`атрибути, значення, надані в`merge`Метод вважатиметься значеннями атрибуту за замовчуванням, які можуть бути замінені споживачем компонента. На відміну від`class`атрибути, некласові атрибути не додаються один до одного. Наприклад, a`button`компонент може виглядати наступним чином:

    <button {{ $attributes->merge(['type' => 'button']) }}>
        {{ $slot }}
    </button>

Візуалізувати компонент кнопки за допомогою спеціального`type`, це може бути вказано при споживанні компонента. Якщо тип не вказаний, то`button`буде використовуватися тип:

    <x-button type="submit">
        Submit
    </x-button>

Візуалізований HTML`button`компонентом у цьому прикладі буде:

    <button type="submit">
        Submit
    </button>

Якщо вам потрібен атрибут, відмінний від`class`щоб його значення складалися разом, ви можете використовувати`prepends`метод:

    <div {{ $attributes->merge(['data-controller' => $attributes->prepends('profile-controller')]) }}>
        {{ $slot }}
    </div>

<a name="filtering-attributes"></a>

#### Атрибути фільтрації

Ви можете фільтрувати атрибути за допомогою`filter`метод. Цей метод приймає Закриття, яке повинно повернутися`true`якщо ви хочете зберегти атрибут у мішці атрибутів:

    {{ $attributes->filter(fn ($value, $key) => $key == 'foo') }}

Для зручності ви можете використовувати`whereStartsWith`метод отримання всіх атрибутів, ключі яких починаються з заданого рядка:

    {{ $attributes->whereStartsWith('wire:model') }}

Використання`first`методом, ви можете відтворити перший атрибут у даному пакеті атрибутів:

    {{ $attributes->whereStartsWith('wire:model')->first() }}

<a name="slots"></a>

### Слоти

Часто вам потрібно буде передати додатковий вміст своєму компоненту через "слоти". Давайте уявимо, що an`alert`компонент, який ми створили, має таку розмітку:

    <!-- /resources/views/components/alert.blade.php -->

    <div class="alert alert-danger">
        {{ $slot }}
    </div>

Ми можемо передавати вміст до`slot`шляхом введення вмісту в компонент:

    <x-alert>
        <strong>Whoops!</strong> Something went wrong!
    </x-alert>

Іноді компоненту може знадобитися відобразити кілька різних слотів у різних місцях усередині компонента. Давайте змінимо наш компонент оповіщення, щоб дозволити введення "заголовка":

    <!-- /resources/views/components/alert.blade.php -->

    <span class="alert-title">{{ $title }}</span>

    <div class="alert alert-danger">
        {{ $slot }}
    </div>

Ви можете визначити вміст названого слота за допомогою`x-slot`тег. Будь-який вміст, який не знаходиться в`x-slot`тег буде переданий компоненту в`$slot`змінна:

    <x-alert>
        <x-slot name="title">
            Server Error
        </x-slot>

        <strong>Whoops!</strong> Something went wrong!
    </x-alert>

<a name="scoped-slots"></a>

#### Слоти

Якщо ви використовували фреймворк JavaScript, такий як Vue, можливо, ви знайомі з "масштабованими слотами", які дозволяють отримувати доступ до даних або методів з компонента у вашому слоті. Ви можете досягти подібної поведінки в Laravel, визначивши загальнодоступні методи або властивості для свого компонента та отримавши доступ до компонента у вашому слоті через`$component`змінна:

    <x-alert>
        <x-slot name="title">
            {{ $component->formatAlert('Server Error') }}
        </x-slot>

        <strong>Whoops!</strong> Something went wrong!
    </x-alert>

<a name="inline-component-views"></a>

### Вбудовані перегляди компонентів

Для дуже маленьких компонентів може здатися громіздким управління як класом компонентів, так і шаблоном подання компонента. З цієї причини ви можете повернути розмітку компонента безпосередньо з`render`метод:

    /**
     * Get the view / contents that represent the component.
     *
     * @return \Illuminate\View\View|\Closure|string
     */
    public function render()
    {
        return <<<'blade'
            <div class="alert alert-danger">
                {{ $slot }}
            </div>
        blade;
    }

<a name="generating-inline-view-components"></a>

#### Генерування компонентів вбудованого перегляду

Щоб створити компонент, який відображає вбудований вигляд, ви можете використовувати`inline`параметр при виконанні`make:component`команда:

    php artisan make:component Alert --inline

<a name="anonymous-components"></a>

### Анонімні компоненти

Подібно до вбудованих компонентів, анонімні компоненти забезпечують механізм управління компонентом через один файл. Однак анонімні компоненти використовують один файл перегляду і не мають пов'язаного класу. Щоб визначити анонімний компонент, вам потрібно лише розмістити шаблон Blade у вашому`resources/views/components`каталог. Наприклад, припускаючи, що ви визначили компонент на`resources/views/components/alert.blade.php`:

    <x-alert/>

Ви можете використовувати`.`символ, щоб вказати, чи компонент вкладений глибше всередину`components`каталог. Наприклад, припускаючи, що компонент визначений в`resources/views/components/inputs/button.blade.php`, ви можете зробити це так:

    <x-inputs.button/>

<a name="data-properties-attributes"></a>

#### Властивості даних / атрибути

Оскільки анонімні компоненти не мають жодного асоційованого класу, ви можете задатися питанням, як можна диференціювати, які дані повинні передаватися компоненту як змінні, а які атрибути слід розміщувати в[атрибут сумка](#managing-attributes).

Ви можете вказати, які атрибути слід вважати змінними даних, використовуючи`@props`вгорі шаблону Blade вашого компонента. Усі інші атрибути компонента будуть доступні через пакет атрибутів компонента. Якщо ви хочете надати змінній даних значення за замовчуванням, ви можете вказати ім'я змінної як ключ масиву, а значення за замовчуванням як значення масиву:

    <!-- /resources/views/components/alert.blade.php -->

    @props(['type' => 'info', 'message'])

    <div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
        {{ $message }}
    </div>

<a name="dynamic-components"></a>

### Динамічні компоненти

Іноді вам може знадобитися відобразити компонент, але не знати, який компонент повинен бути відтворений до часу виконання. У цій ситуації ви можете використовувати вбудований Laravel`dynamic-component`компонент для візуалізації компонента на основі значення середовища або змінної:

    <x-dynamic-component :component="$componentName" class="mt-4" />

<a name="including-subviews"></a>

## Включаючи підпрограми

Лезо`@include`Директива дозволяє включити Blade подання з іншого подання. Усі змінні, доступні для батьківського подання, будуть доступні для включеного подання:

    <div>
        @include('shared.errors')

        <form>
            <!-- Form Contents -->
        </form>
    </div>

Незважаючи на те, що включений подання успадкує всі дані, доступні в батьківському поданні, ви також можете передати масив додаткових даних включеному поданню:

    @include('view.name', ['some' => 'data'])

Якщо ви спробуєте`@include`подання, яке не існує, Laravel видасть помилку. Якщо ви хочете включити подання, яке може бути або не бути, вам слід скористатися`@includeIf`директива:

    @includeIf('view.name', ['some' => 'data'])

Якщо ви хотіли б`@include`подання, якщо заданий булевий вираз має значення`true`, ви можете використовувати`@includeWhen`директива:

    @includeWhen($boolean, 'view.name', ['some' => 'data'])

Якщо ви хотіли б`@include`подання, якщо заданий булевий вираз має значення`false`, ви можете використовувати`@includeUnless`директива:

    @includeUnless($boolean, 'view.name', ['some' => 'data'])

Для включення першого подання, яке існує із заданого масиву переглядів, ви можете використовувати`includeFirst`директива:

    @includeFirst(['custom.admin', 'admin'], ['some' => 'data'])

> {note} Вам слід уникати використання`__DIR__`і`__FILE__`константи у ваших поданнях Blade, оскільки вони будуть посилатися на розташування кешованого, скомпільованого подання.

<a name="aliasing-includes"></a>

#### Псевдонім включає

Якщо ваш Blade включений зберігається у підкаталозі, можливо, ви захочете псевдонім для полегшення доступу. Наприклад, уявіть, що входить Blade, що зберігається в`resources/views/includes/input.blade.php`з таким змістом:

    <input type="{{ $type ?? 'text' }}">

Ви можете використовувати`include`метод для псевдоніма включення з`includes.input`до`input`. Як правило, це слід робити в`boot`метод вашого`AppServiceProvider`:

    use Illuminate\Support\Facades\Blade;

    Blade::include('includes.input', 'input');

Коли псевдонім буде включено, ви можете зробити його, використовуючи псевдонім як директиву Blade:

    @input(['type' => 'email'])

<a name="rendering-views-for-collections"></a>

### Візуалізація подань для колекцій

Ви можете поєднувати петлі та включення в одну лінію з лезами`@each`директива:

    @each('view.name', $jobs, 'job')

Перший аргумент - це частковий вигляд, який відображається для кожного елемента масиву чи колекції. Другий аргумент - це масив або колекція, для яких ви хочете виконати ітерацію, тоді як третій аргумент - це назва змінної, яка буде присвоєна поточній ітерації у поданні. Так, наприклад, якщо ви виконуєте ітерацію по масиву`jobs`, як правило, ви хочете отримати доступ до кожного завдання як`job`змінна у вашому поданні часткова. Ключ поточної ітерації буде доступний як`key`змінна у вашому поданні часткова.

Ви також можете передати четвертий аргумент`@each`директива. Цей аргумент визначає подання, яке буде відображатися, якщо даний масив порожній.

    @each('view.name', $jobs, 'job', 'view.empty')

> {note} Перегляди, надані через`@each`не успадковувати змінні з батьківського подання. Якщо дочірній перегляд вимагає цих змінних, вам слід скористатися`@foreach`і`@include`замість цього.

<a name="stacks"></a>

## Стеки

Blade дозволяє перейти до іменованих стеків, які можуть бути відтворені деінде в іншому поданні або макеті. Це може бути особливо корисно для вказівки будь-яких бібліотек JavaScript, необхідних для перегляду вашої дитини:

    @push('scripts')
        <script src="/example.js"></script>
    @endpush

Ви можете натиснути на стос стільки разів, скільки потрібно. Щоб відобразити повний вміст стека, передайте ім'я стеку в`@stack`директива:

    <head>
        <!-- Head Contents -->

        @stack('scripts')
    </head>

Якщо ви хочете додати вміст на початок стека, вам слід використовувати`@prepend`директива:

    @push('scripts')
        This will be second...
    @endpush

    // Later...

    @prepend('scripts')
        This will be first...
    @endprepend

<a name="service-injection"></a>

## Сервісна ін’єкція

`@inject`Директива може бути використана для отримання послуги з Laravel[службовий контейнер](/docs/{{version}}/container). Перший аргумент передано`@inject`це ім'я змінної, до якої буде розміщена служба, тоді як другий аргумент - це назва класу або інтерфейсу служби, яку ви хочете вирішити:

    @inject('metrics', 'App\Services\MetricsService')

    <div>
        Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
    </div>

<a name="extending-blade"></a>

## Подовжувальний клинок

Blade дозволяє вам визначити власні власні директиви за допомогою`directive`метод. Коли компілятор Blade зустріне користувацьку директиву, він викличе наданий зворотний виклик із виразом, який містить директива.

Наступний приклад створює a`@datetime($var)`директива, яка форматує даний`$var`, який повинен бути екземпляром`DateTime`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Blade::directive('datetime', function ($expression) {
                return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
            });
        }
    }

Як бачите, ми будемо ланцюжок`format`метод на будь-який вираз, переданий в директиву. Отже, у цьому прикладі остаточний PHP, згенерований цією директивою, буде таким:

    <?php echo ($var)->format('m/d/Y H:i'); ?>

> {note} Після оновлення логіки директиви Blade вам потрібно буде видалити всі кешовані подання Blade. Кешовані види Blade можна видалити за допомогою`view:clear`Ремісниче командування.

<a name="custom-if-statements"></a>

### Спеціальні виписки If

Програмування користувацької директиви іноді є складнішим, ніж це необхідно при визначенні простих, користувацьких умовних операторів. З цієї причини Blade надає a`Blade::if`метод, який дозволяє швидко визначати власні умовні директиви за допомогою Closures. Наприклад, давайте визначимо спеціальний умовний, який перевіряє поточного постачальника хмарних додатків. Ми можемо зробити це в`boot`метод нашого`AppServiceProvider`:

    use Illuminate\Support\Facades\Blade;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('cloud', function ($provider) {
            return config('filesystems.default') === $provider;
        });
    }

Після того, як спеціальний умовний буде визначений, ми можемо легко використовувати його на наших шаблонах:

    @cloud('digitalocean')
        // The application is using the digitalocean cloud provider...
    @elsecloud('aws')
        // The application is using the aws provider...
    @else
        // The application is not using the digitalocean or aws environment...
    @endcloud

    @unlesscloud('aws')
        // The application is not using the aws environment...
    @endcloud
