# Перевірка

-   [Вступ](#introduction)
-   [Швидкий старт перевірки](#validation-quickstart)
    -   [Визначення маршрутів](#quick-defining-the-routes)
    -   [Створення контролера](#quick-creating-the-controller)
    -   [Написання логіки перевірки](#quick-writing-the-validation-logic)
    -   [Відображення помилок перевірки](#quick-displaying-the-validation-errors)
    -   [Примітка щодо необов’язкових полів](#a-note-on-optional-fields)
-   [Перевірка запиту форми](#form-request-validation)
    -   [Створення запитів на форми](#creating-form-requests)
    -   [Запити на авторизацію форми](#authorizing-form-requests)
    -   [Налаштування повідомлень про помилки](#customizing-the-error-messages)
    -   [Налаштування атрибутів перевірки](#customizing-the-validation-attributes)
    -   [Підготуйте дані для перевірки](#prepare-input-for-validation)
-   [Створення валідаторів вручну](#manually-creating-validators)
    -   [Автоматичне перенаправлення](#automatic-redirection)
    -   [Іменовані пакети помилок](#named-error-bags)
    -   [Після перевірки гачок](#after-validation-hook)
-   [Робота з повідомленнями про помилки](#working-with-error-messages)
    -   [Спеціальні повідомлення про помилки](#custom-error-messages)
-   [Доступні правила перевірки](#available-validation-rules)
-   [Умовно додавання правил](#conditionally-adding-rules)
-   [Перевірка масивів](#validating-arrays)
-   [Спеціальні правила перевірки](#custom-validation-rules)
    -   [Використання об’єктів правила](#using-rule-objects)
    -   [Використання замикань](#using-closures)
    -   [Використання розширень](#using-extensions)
    -   [Неявне розширення](#implicit-extensions)

<a name="introduction"></a>

## Вступ

Laravel пропонує декілька різних підходів для перевірки вхідних даних вашої програми. За замовчуванням базовий клас контролера Laravel використовує a`ValidatesRequests`ознака, яка забезпечує зручний метод перевірки вхідних запитів HTTP за допомогою різноманітних потужних правил перевірки.

<a name="validation-quickstart"></a>

## Швидкий старт перевірки

Щоб дізнатись про потужні функції перевірки Laravel, давайте розглянемо повний приклад перевірки форми та відображення повідомлень про помилки назад для користувача.

<a name="quick-defining-the-routes"></a>

### Визначення маршрутів

По-перше, припустимо, що в нашому визначені такі маршрути`routes/web.php`файл:

    use App\Http\Controllers\PostController;

    Route::get('post/create', [PostController::class, 'create']);

    Route::post('post', [PostController::class, 'store']);

`GET`route буде відображати форму для користувача, щоб створити нову публікацію в блозі, тоді як`POST`route збереже новий запис у блозі в базі даних.

<a name="quick-creating-the-controller"></a>

### Створення контролера

Далі, давайте поглянемо на простий контролер, який обробляє ці маршрути. Ми залишимо`store`на даний момент метод порожній:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class PostController extends Controller
    {
        /**
         * Show the form to create a new blog post.
         *
         * @return Response
         */
        public function create()
        {
            return view('post.create');
        }

        /**
         * Store a new blog post.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Validate and store the blog post...
        }
    }

<a name="quick-writing-the-validation-logic"></a>

### Написання логіки перевірки

Тепер ми готові заповнити наш`store`метод із логікою перевірки нового повідомлення в блозі. Для цього ми будемо використовувати`validate`метод, передбачений`Illuminate\Http\Request`об'єкт. Якщо правила перевірки пройдуть, ваш код буде продовжувати працювати нормально; однак, якщо перевірка не вдається, буде видано виняток і відповідна відповідь на помилку буде автоматично надіслана користувачеві. У випадку традиційного HTTP-запиту буде сформовано відповідь на переадресацію, тоді як відповідь JSON буде надіслана на запити AJAX.

Для кращого розуміння`validate`метод, давайте перейдемо назад до`store`метод:

    /**
     * Store a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $validatedData = $request->validate([
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        // The blog post is valid...
    }

Як бачите, ми передаємо бажані правила перевірки в`validate`метод. Знову ж таки, якщо перевірка не вдається, відповідна відповідь буде автоматично сформовано. Якщо перевірка пройде, наш контролер продовжить нормальне виконання.

Як варіант, правила перевірки можуть бути вказані як масиви правил замість одного`|`розділений рядок:

    $validatedData = $request->validate([
        'title' => ['required', 'unique:posts', 'max:255'],
        'body' => ['required'],
    ]);

Ви можете використовувати`validateWithBag`метод для перевірки запиту та збереження повідомлень про помилки в[іменований пакет помилок](#named-error-bags):

    $validatedData = $request->validateWithBag('post', [
        'title' => ['required', 'unique:posts', 'max:255'],
        'body' => ['required'],
    ]);

<a name="stopping-on-first-validation-failure"></a>

#### Зупинка на першій помилці перевірки

Іноді, можливо, вам доведеться припинити запуск правил перевірки атрибута після першої помилки перевірки. Для цього призначте`bail`правило для атрибута:

    $request->validate([
        'title' => 'bail|required|unique:posts|max:255',
        'body' => 'required',
    ]);

У цьому прикладі, якщо`unique`правило про`title`атрибут не вдається,`max`правило не перевірятиметься. Правила перевірятимуться у тому порядку, в якому вони були призначені.

<a name="a-note-on-nested-attributes"></a>

#### Примітка про вкладені атрибути

Якщо ваш запит HTTP містить "вкладені" параметри, ви можете вказати їх у своїх правилах перевірки, використовуючи синтаксис "крапка":

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'author.name' => 'required',
        'author.description' => 'required',
    ]);

З іншого боку, якщо ваше ім'я поля містить буквальний крапка, ви можете явно заборонити інтерпретувати це як синтаксис "крапки", уникаючи крапки з зворотною рискою:

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'v1\.0' => 'required',
    ]);

<a name="quick-displaying-the-validation-errors"></a>

### Відображення помилок перевірки

Отже, що, якщо параметри вхідного запиту не передають задані правила перевірки? Як вже згадувалося раніше, Laravel автоматично перенаправить користувача назад у попереднє місце. Крім того, усі помилки перевірки будуть автоматично[блиснув до сесії](/docs/{{version}}/session#flash-data).

Знову зауважте, що нам не потрібно було явно прив’язувати повідомлення про помилки до подання в нашому`GET`маршрут. Це пов’язано з тим, що Laravel перевірить наявність помилок у даних сеансу та автоматично прив’яже їх до подання, якщо вони доступні.`$errors`змінна буде екземпляром`Illuminate\Support\MessageBag`. Для отримання додаткової інформації про роботу з цим об’єктом,[ознайомтесь з його документацією](#working-with-error-messages).

> {tip} The`$errors`змінна пов'язана з поданням`Illuminate\View\Middleware\ShareErrorsFromSession`Middlware, яке надає`web`група проміжного програмного забезпечення.**Коли застосовується це Middlware`$errors`змінна завжди буде доступна у ваших поданнях**, що дозволяє зручно припустити`$errors`змінна завжди визначається і може безпечно використовуватися.

Отже, у нашому прикладі користувач буде перенаправлений на наш контролер`create`метод, коли перевірка не вдається, що дозволяє нам відображати повідомлення про помилки у поданні:

    <!-- /resources/views/post/create.blade.php -->

    <h1>Create Post</h1>

    @if ($errors->any())
        <div class="alert alert-danger">
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    <!-- Create Post Form -->

<a name="the-at-error-directive"></a>

#### `@error`Директива

Ви також можете використовувати`@error`[Лезо](/docs/{{version}}/blade)директива, щоб швидко перевірити, чи існують повідомлення про помилки перевірки для даного атрибута. В межах`@error`директиви, ви можете повторити`$message`змінна для відображення повідомлення про помилку:

    <!-- /resources/views/post/create.blade.php -->

    <label for="title">Post Title</label>

    <input id="title" type="text" class="@error('title') is-invalid @enderror">

    @error('title')
        <div class="alert alert-danger">{{ $message }}</div>
    @enderror

<a name="a-note-on-optional-fields"></a>

### Примітка щодо необов’язкових полів

За замовчуванням Laravel включає`TrimStrings`і`ConvertEmptyStringsToNull`Middlware у глобальному стеку проміжного програмного забезпечення вашої програми. Це Middlware перелічено в стеку`App\Http\Kernel`клас. Через це вам часто доведеться позначати свої "необов'язкові" поля запиту як`nullable`якщо ви не хочете, щоб валідатор розглядав`null`значення як недійсні. Наприклад:

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

У цьому прикладі ми вказуємо, що`publish_at`поле може бути будь-яким`null`або подання дійсної дати. Якщо`nullable`модифікатор не додається до визначення правила, вважав би валідатор`null`недійсна дата.

<a name="quick-ajax-requests-and-validation"></a>

#### Запити та перевірка AJAX

У цьому прикладі ми використовували традиційну форму для надсилання даних до програми. Однак багато програм використовують запити AJAX. При використанні`validate`під час запиту AJAX, Laravel не генерує переадресацію відповіді. Натомість Laravel генерує відповідь JSON, що містить усі помилки перевірки. Ця відповідь JSON буде надіслана із 422 кодом стану HTTP.

<a name="form-request-validation"></a>

## Перевірка запиту форми

<a name="creating-form-requests"></a>

### Створення запитів на форми

Для більш складних сценаріїв перевірки ви можете створити "запит форми". Запити форми - це власні класи запитів, які містять логіку перевірки. Щоб створити клас запиту форми, використовуйте`make:request`Команда Artisan CLI:

    php artisan make:request StoreBlogPost

Створений клас буде розміщений у`app/Http/Requests`каталог. Якщо цей каталог не існує, він буде створений під час запуску`make:request`команди. Давайте додамо кілька правил перевірки до`rules`метод:

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }

> {tip} Ви можете ввести натяк на будь-які залежності, які вам потрібні в межах`rules`підпис методу. Вони будуть автоматично вирішені через Laravel[службовий контейнер](/docs/{{version}}/container).

Отже, як оцінюються правила перевірки? Все, що вам потрібно зробити, це ввести натяк на запит щодо методу вашого контролера. Запит на вхідну форму перевіряється до виклику методу контролера, тобто вам не потрібно захаращувати свій контролер жодною логікою перевірки:

    /**
     * Store the incoming blog post.
     *
     * @param  StoreBlogPost  $request
     * @return Response
     */
    public function store(StoreBlogPost $request)
    {
        // The incoming request is valid...

        // Retrieve the validated input data...
        $validated = $request->validated();
    }

Якщо перевірка не вдається, буде сформовано відповідь на переадресацію, щоб відправити користувача назад у попереднє місце. Помилки також будуть передані в сеанс, щоб вони були доступними для відображення. Якщо запит був запитом AJAX, користувачеві буде повернуто відповідь HTTP із кодом стану 422, включаючи представлення помилок перевірки у форматі JSON.

<a name="adding-after-hooks-to-form-requests"></a>

#### Додавання після гачків для формування запитів

Якщо ви хочете додати гачок "після" до запиту форми, ви можете використовувати`withValidator`метод. Цей метод отримує повністю побудований валідатор, що дозволяє вам викликати будь-який з його методів до того, як фактично перевіряються правила перевірки:

    /**
     * Configure the validator instance.
     *
     * @param  \Illuminate\Validation\Validator  $validator
     * @return void
     */
    public function withValidator($validator)
    {
        $validator->after(function ($validator) {
            if ($this->somethingElseIsInvalid()) {
                $validator->errors()->add('field', 'Something is wrong with this field!');
            }
        });
    }

<a name="authorizing-form-requests"></a>

### Запити на авторизацію форми

Клас запиту форми також містить`authorize`метод. За допомогою цього методу ви можете перевірити, чи справді аутентифікований користувач має повноваження оновлювати даний ресурс. Наприклад, ви можете визначити, чи є користувач насправді власником коментаря до блогу, який він намагається оновити:

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        $comment = Comment::find($this->route('comment'));

        return $comment && $this->user()->can('update', $comment);
    }

Оскільки всі запити форми розширюють базовий клас запитів Laravel, ми можемо використовувати`user`для доступу до поточно автентифікованого користувача. Також зверніть увагу на дзвінок до`route`у наведеному вище прикладі. Цей метод надає вам доступ до параметрів URI, визначених на маршруті, що викликається, наприклад`{comment}`параметр у прикладі нижче:

    Route::post('comment/{comment}');

Якщо`authorize`метод повертає`false`, відповідь HTTP із кодом стану 403 буде автоматично повернуто, і ваш метод контролера не буде виконаний.

Якщо ви плануєте мати логіку авторизації в іншій частині вашої програми, поверніться`true`від`authorize`метод:

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

> {tip} Ви можете ввести натяк на будь-які залежності, які вам потрібні в межах`authorize`підпис методу. Вони будуть автоматично вирішені через Laravel[службовий контейнер](/docs/{{version}}/container).

<a name="customizing-the-error-messages"></a>

### Налаштування повідомлень про помилки

Ви можете налаштувати повідомлення про помилки, що використовуються запитом форми, замінивши`messages`метод. Цей метод повинен повертати масив пар атрибут / правило та відповідні повідомлення про помилки:

    /**
     * Get the error messages for the defined validation rules.
     *
     * @return array
     */
    public function messages()
    {
        return [
            'title.required' => 'A title is required',
            'body.required' => 'A message is required',
        ];
    }

<a name="customizing-the-validation-attributes"></a>

### Налаштування атрибутів перевірки

Якщо ви хочете`:attribute`Частина вашого повідомлення про перевірку, яку потрібно замінити на ім'я власного атрибута, ви можете вказати власні імена, замінивши`attributes`метод. Цей метод повинен повертати масив пар атрибутів / імен:

    /**
     * Get custom attributes for validator errors.
     *
     * @return array
     */
    public function attributes()
    {
        return [
            'email' => 'email address',
        ];
    }

<a name="prepare-input-for-validation"></a>

### Підготуйте дані для перевірки

Якщо вам потрібно продезінфікувати будь-які дані із запиту, перш ніж застосовувати свої правила перевірки, ви можете скористатися`prepareForValidation`метод:

    use Illuminate\Support\Str;

    /**
     * Prepare the data for validation.
     *
     * @return void
     */
    protected function prepareForValidation()
    {
        $this->merge([
            'slug' => Str::slug($this->slug),
        ]);
    }

<a name="manually-creating-validators"></a>

## Створення валідаторів вручну

Якщо ви не хочете використовувати`validate`методу на запит, ви можете створити екземпляр валідатора вручну, використовуючи`Validator`[фасад](/docs/{{version}}/facades).`make`на фасаді генерує новий екземпляр валідатора:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Validator;

    class PostController extends Controller
    {
        /**
         * Store a new blog post.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $validator = Validator::make($request->all(), [
                'title' => 'required|unique:posts|max:255',
                'body' => 'required',
            ]);

            if ($validator->fails()) {
                return redirect('post/create')
                            ->withErrors($validator)
                            ->withInput();
            }

            // Store the blog post...
        }
    }

Перший аргумент, переданий в`make`метод - це дані, що перевіряються. Другий аргумент - це масив правил перевірки, які слід застосовувати до даних.

Перевіривши, чи не вдалося перевірити запит, ви можете використовувати`withErrors`метод передавати повідомлення про помилки до сеансу. При використанні цього методу,`$errors`Змінна автоматично буде передана вашим переглядам після перенаправлення, що дозволить вам легко відображати їх назад користувачеві.`withErrors`метод приймає валідатор, a`MessageBag`, or a PHP `array`.

<a name="automatic-redirection"></a>

### Автоматичне перенаправлення

Якщо ви хочете створити екземпляр валідатора вручну, але все ж скористайтеся перевагами автоматичного перенаправлення, запропонованого запитом`validate`метод, ви можете зателефонувати до`validate`метод на існуючому екземплярі валідатора. Якщо перевірка не вдається, користувач буде автоматично перенаправлений або, у разі запиту AJAX, буде повернуто відповідь JSON:

    Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ])->validate();

Ви можете використовувати`validateWithBag`метод зберігання повідомлень про помилки в[іменований пакет помилок](#named-error-bags)якщо перевірка не вдається:

    Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ])->validateWithBag('post');

<a name="named-error-bags"></a>

### Іменовані пакети помилок

Якщо у вас є кілька форм на одній сторінці, ви можете назвати`MessageBag`помилок, дозволяючи отримувати повідомлення про помилки для певної форми. Передайте ім'я як другий аргумент`withErrors`:

    return redirect('register')
                ->withErrors($validator, 'login');

Потім ви можете отримати доступ до названого`MessageBag`екземпляр з`$errors`змінна:

    {{ $errors->login->first('email') }}

<a name="after-validation-hook"></a>

### Після перевірки гачок

Валідатор також дозволяє приєднати зворотні виклики, які будуть запущені після завершення перевірки. Це дозволяє легко виконувати подальшу перевірку та навіть додавати більше повідомлень про помилки до колекції повідомлень. Для початку використовуйте`after`метод на екземплярі валідатора:

    $validator = Validator::make(...);

    $validator->after(function ($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });

    if ($validator->fails()) {
        //
    }

<a name="working-with-error-messages"></a>

## Робота з повідомленнями про помилки

Після виклику`errors`метод на a`Validator`наприклад, ви отримаєте`Illuminate\Support\MessageBag`екземпляр, який має безліч зручних методів роботи з повідомленнями про помилки.`$errors`Змінна, яка автоматично стає доступною для всіх подань, також є екземпляром`MessageBag`клас.

<a name="retrieving-the-first-error-message-for-a-field"></a>

#### Отримання першого повідомлення про помилку для поля

Щоб отримати перше повідомлення про помилку для даного поля, використовуйте`first`метод:

    $errors = $validator->errors();

    echo $errors->first('email');

<a name="retrieving-all-error-messages-for-a-field"></a>

#### Отримання всіх повідомлень про помилки для поля

Якщо вам потрібно отримати масив усіх повідомлень для даного поля, використовуйте`get`метод:

    foreach ($errors->get('email') as $message) {
        //
    }

Якщо ви перевіряєте поле форми масиву, ви можете отримати всі повідомлення для кожного з елементів масиву, використовуючи`*`характер:

    foreach ($errors->get('attachments.*') as $message) {
        //
    }

<a name="retrieving-all-error-messages-for-all-fields"></a>

#### Отримання всіх повідомлень про помилки для всіх полів

Щоб отримати масив усіх повідомлень для всіх полів, використовуйте`all`метод:

    foreach ($errors->all() as $message) {
        //
    }

<a name="determining-if-messages-exist-for-a-field"></a>

#### Визначення, чи існують повідомлення для поля

`has`метод може бути використаний, щоб визначити, чи існують повідомлення про помилку для даного поля:

    if ($errors->has('email')) {
        //
    }

<a name="custom-error-messages"></a>

### Спеціальні повідомлення про помилки

За потреби ви можете використовувати спеціальні повідомлення про помилки для перевірки замість типових. Існує кілька способів вказати власні повідомлення. По-перше, ви можете передавати власні повідомлення як третій аргумент до`Validator::make`метод:

    $messages = [
        'required' => 'The :attribute field is required.',
    ];

    $validator = Validator::make($input, $rules, $messages);

У цьому прикладі`:attribute`заповнювач буде замінено фактичною назвою поля, що перевіряється. Ви також можете використовувати інші заповнювачі у повідомленнях про перевірку. Наприклад:

    $messages = [
        'same' => 'The :attribute and :other must match.',
        'size' => 'The :attribute must be exactly :size.',
        'between' => 'The :attribute value :input is not between :min - :max.',
        'in' => 'The :attribute must be one of the following types: :values',
    ];

<a name="specifying-a-custom-message-for-a-given-attribute"></a>

#### Вказівка ​​користувацького повідомлення для заданого атрибута

Іноді вам може знадобитися вказати власне повідомлення про помилку лише для певного поля. Ви можете зробити це, використовуючи позначення "крапка". Спочатку вкажіть ім’я атрибута, а потім правило:

    $messages = [
        'email.required' => 'We need to know your e-mail address!',
    ];

<a name="localization"></a>

#### Вказівка ​​користувацьких повідомлень у мовних файлах

У більшості випадків ви, ймовірно, будете вказувати власні повідомлення у мовному файлі, а не передавати їх безпосередньо до`Validator`. Для цього додайте свої повідомлення до`custom`масив у`resources/lang/xx/validation.php`мовний файл.

    'custom' => [
        'email' => [
            'required' => 'We need to know your e-mail address!',
        ],
    ],

<a name="specifying-custom-attribute-values"></a>

#### Вказівка ​​користувацьких значень атрибутів

Якщо ви хочете`:attribute`Частина вашого повідомлення про перевірку, яку потрібно замінити на ім'я власного атрибута, ви можете вказати власне ім'я в`attributes`масив вашого`resources/lang/xx/validation.php`мовний файл:

    'attributes' => [
        'email' => 'email address',
    ],

Ви також можете передати користувацькі атрибути як четвертий аргумент для`Validator::make`метод:

    $customAttributes = [
        'email' => 'email address',
    ];

    $validator = Validator::make($input, $rules, $messages, $customAttributes);

<a name="specifying-custom-values-in-language-files"></a>

#### Вказівка ​​користувацьких значень у мовних файлах

Іноді вам може знадобитися`:value`частина вашого повідомлення про перевірку, яку потрібно замінити користувацьким поданням значення. Наприклад, розглянемо наступне правило, яке визначає, що номер кредитної картки потрібен, якщо`payment_type`має значення`cc`:

    $request->validate([
        'credit_card_number' => 'required_if:payment_type,cc'
    ]);

Якщо це правило перевірки не вдається, воно видасть таке повідомлення про помилку:

    The credit card number field is required when payment type is cc.

Замість відображення`cc`як значення типу платежу ви можете вказати спеціальне подання вартості у вашому`validation`мовний файл, визначивши a`values`масив:

    'values' => [
        'payment_type' => [
            'cc' => 'credit card'
        ],
    ],

Тепер, якщо правило перевірки не вдається, воно видасть таке повідомлення:

    The credit card number field is required when payment type is credit card.

<a name="available-validation-rules"></a>

## Доступні правила перевірки

Нижче наведено перелік усіх доступних правил перевірки та їх функції:

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="collection-method-list" markdown="1">

[Прийнято](#rule-accepted)[Активна URL-адреса](#rule-active-url)[Після (Дата)](#rule-after)[Після або рівне (дата)](#rule-after-or-equal)[Альфа](#rule-alpha)[Альфа-тире](#rule-alpha-dash)[Буквено-цифрові](#rule-alpha-num)[Масив](#rule-array)[Під заставу](#rule-bail)[До (дата)](#rule-before)[До або рівне (дата)](#rule-before-or-equal)[Між](#rule-between)[Логічна](#rule-boolean)[Підтверджено](#rule-confirmed)[Дата](#rule-date)[Дата дорівнює](#rule-date-equals)[Формат дати](#rule-date-format)[Інший](#rule-different)[Цифри](#rule-digits)[Цифри між](#rule-digits-between)[Розміри (файли зображень)](#rule-dimensions)[Виразний](#rule-distinct)[Електронна пошта](#rule-email)[Закінчується с](#rule-ends-with)[Виключити Якщо](#rule-exclude-if)[Виключити хіба що](#rule-exclude-unless)[Існує (база даних)](#rule-exists)[Файл](#rule-file)[Заповнені](#rule-filled)[Більше ніж](#rule-gt)[Більший або рівний](#rule-gte)[Зображення (файл)](#rule-image)[В](#rule-in)[У масиві](#rule-in-array)[Ціле число](#rule-integer)[IP-адреса](#rule-ip)[JSON](#rule-json)[Less Than](#rule-lt)[Менше або рівне](#rule-lte)[Макс](#rule-max)[Типи MIME](#rule-mimetypes)[Тип MIME за розширенням файлу](#rule-mimes)[Хв](#rule-min)[Кілька](#multiple-of)
[Не в](#rule-not-in)[Не регулярний вираз](#rule-not-regex)[Допустимий](#rule-nullable)[Числовий](#rule-numeric)[Пароль](#rule-password)[Присутні](#rule-present)[Регулярний вираз](#rule-regex)[вимагається](#rule-required)[Потрібно Якщо](#rule-required-if)[Обов’язково, окрім випадків](#rule-required-unless)[Потрібно с](#rule-required-with)[Потрібно з усіма](#rule-required-with-all)[Потрібно без](#rule-required-without)[Потрібно без усіх](#rule-required-without-all)[Те саме](#rule-same)[Розмір](#rule-size)[Іноді](#conditionally-adding-rules)[Починається з](#rule-starts-with)[Рядок](#rule-string)[Часовий пояс](#rule-timezone)[Унікальний (база даних)](#rule-unique)[URL](#rule-url)[UUID](#rule-uuid)

</div>

<a name="rule-accepted"></a>

#### прийнято

Поле, що перевіряється, має бути_так_,_на_,_1_, або_правда_. Це корисно для перевірки прийняття "Умов використання".

<a name="rule-active-url"></a>

#### active_url

Поле, що перевіряється, повинно мати дійсний запис A або AAAA відповідно до`dns_get_record`Функція PHP. Ім'я хосту наданої URL-адреси витягується за допомогою`parse_url`Функція PHP перед передачею`dns_get_record`.

<a name="rule-after"></a>

#### після:_дата_

Поле, що перевіряється, має мати значення після заданої дати. Дати будуть передані в`strtotime`Функція PHP:

    'start_date' => 'required|date|after:tomorrow'

Замість того, щоб передавати рядок дати, яку слід оцінити`strtotime`, Ви можете вказати інше поле для порівняння з датою:

    'finish_date' => 'required|date|after:start_date'

<a name="rule-after-or-equal"></a>

#### після\_або\_дорівнює:_дата_

Поле, що перевіряється, має мати значення після або дорівнювати даній даті. Для отримання додаткової інформації див[після](#rule-after)правило.

<a name="rule-alpha"></a>

#### альфа

Поле, що перевіряється, повинно мати повністю алфавітні символи.

<a name="rule-alpha-dash"></a>

#### alpha_dash

Поле, що перевіряється, може мати буквено-цифрові символи, а також тире та підкреслення.

<a name="rule-alpha-num"></a>

#### Alpha_num

Поле, що перевіряється, повинно мати повністю алфавітно-цифрові символи.

<a name="rule-array"></a>

#### масив

Поле, що перевіряється, має бути PHP`array`.

<a name="rule-bail"></a>

#### застава

Зупиніть запуск правил перевірки після першої помилки перевірки.

<a name="rule-before"></a>

#### до:_дата_

Поле, що перевіряється, має мати значення, яке передує даній даті. Дати будуть передані в PHP`strtotime`функція. Крім того, як[`after`](#rule-after)правило, ім'я іншого поля, що перевіряється, може бути вказане як значення`date`.

<a name="rule-before-or-equal"></a>

#### раніше\_або\_дорівнює:_дата_

Поле, що перевіряється, має мати значення, яке передує даній даті або дорівнює їй. Дати будуть передані в PHP`strtotime`функція. Крім того, як[`after`](#rule-after)правило, ім'я іншого поля, що перевіряється, може бути вказане як значення`date`.

<a name="rule-between"></a>

#### між:_хв_,_макс_

Поле, що перевіряється, має мати розмір між заданим_хв_і_макс_. Рядки, числа, масиви та файли оцінюються так само, як і[`size`](#rule-size)правило.

<a name="rule-boolean"></a>

#### логічний

Поле, що перевіряється, повинно мати можливість бути доданим як логічне значення. Приймаються введення`true`,`false`,`1`,`0`,`"1"`, і`"0"`.

<a name="rule-confirmed"></a>

#### підтверджено

Поле, що перевіряється, повинно мати відповідне поле`foo_confirmation`. Наприклад, якщо поле для перевірки є`password`, відповідність`password_confirmation`поле повинно бути присутнім у введенні.

<a name="rule-date"></a>

#### дата

Поле, що перевіряється, має бути дійсною, не відносною датою відповідно до`strtotime`Функція PHP.

<a name="rule-date-equals"></a>

#### дата_дорівнює: \_date_

Поле, що перевіряється, має дорівнювати вказаній даті. Дати будуть передані в PHP`strtotime`функція.

<a name="rule-date-format"></a>

#### дата_формат: \_format_

Поле, що перевіряється, має відповідати заданому_формат_. Ви повинні використовувати**або**`date`або`date_format`під час перевірки поля, а не обох. Це правило перевірки підтримує всі формати, що підтримуються PHP[Дата, час](https://www.php.net/manual/en/class.datetime.php)клас.

<a name="rule-different"></a>

#### інший:_поле_

Поле, що перевіряється, має мати інше значення, ніж_поле_.

<a name="rule-digits"></a>

#### цифри:_значення_

Поле, що перевіряється, має бути_числовий_і повинна мати точну довжину_значення_.

<a name="rule-digits-between"></a>

#### цифри_між: \_min_,_макс_

Поле, що перевіряється, має бути_числовий_і повинна мати довжину між заданими_хв_і_макс_.

<a name="rule-dimensions"></a>

#### розміри

Файл, що перевіряється, повинен бути зображенням, що відповідає обмеженням розмірів, як зазначено параметрами правила:

    'avatar' => 'dimensions:min_width=100,min_height=200'

Доступні обмеження:_хв\_ширина_,_макс\_ширина_,_хв\_висота_,_макс\_висота_,_ширина_,_висота_,_співвідношення_.

A_співвідношення_обмеження має бути представлене як ширина, поділена на висоту. Це може бути вказано або твердженням типу`3/2`або поплавок типу`1.5`:

    'avatar' => 'dimensions:ratio=3/2'

Оскільки це правило вимагає кількох аргументів, ви можете використовувати`Rule::dimensions`метод вільно побудувати правило:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'avatar' => [
            'required',
            Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
        ],
    ]);

<a name="rule-distinct"></a>

#### виразний

При роботі з масивами поле, що перевіряється, не повинно мати жодних повторюваних значень.

    'foo.*.id' => 'distinct'

<a name="rule-email"></a>

#### електронною поштою

Поле, що перевіряється, має бути відформатоване як електронна адреса. Під капотом це правило перевірки використовує[`egulias/email-validator`](https://github.com/egulias/EmailValidator)пакет для перевірки адреси електронної пошти. За замовчуванням`RFCValidation`застосовується валідатор, але ви можете застосувати й інші стилі перевірки:

    'email' => 'email:rfc,dns'

У наведеному вище прикладі буде застосовано`RFCValidation`і`DNSCheckValidation`перевірки. Ось повний перелік стилів перевірки, які ви можете застосувати:

<div class="content-list" markdown="1">
- `rfc`: `RFCValidation`
- `strict`: `NoRFCWarningsValidation`
- `dns`: `DNSCheckValidation`
- `spoof`: `SpoofCheckValidation`
- `filter`: `FilterEmailValidation`
</div>

`filter`валідатор, який використовує PHP`filter_var`функція під капотом, поставляється з Laravel і є поведінкою Laravel до 5,8.`dns`і`spoof`валідатори вимагають PHP`intl`розширення.

<a name="rule-ends-with"></a>

#### закінчується_з: \_foo_,_бар_,...

Поле, що перевіряється, має закінчуватися одним із заданих значень.

<a name="rule-exclude-if"></a>

#### виключити_if: \_anotherfield_,_значення_

Поле, що перевіряється, буде виключене із даних запиту, які повертає`validate`і`validated`методи, якщо_інше поле_поле дорівнює_значення_.

<a name="rule-exclude-unless"></a>

#### виключити_хіба що: \_ані інше поле_,_значення_

Поле, що перевіряється, буде виключене із даних запиту, які повертає`validate`і`validated`методи хіба що_інше поле_поле дорівнює_значення_.

<a name="rule-exists"></a>

#### існує:_таблиця_,_стовпець_

Поле, що перевіряється, повинно існувати в даній таблиці бази даних.

<a name="basic-usage-of-exists-rule"></a>

#### Основне використання правила існуючих правил

    'state' => 'exists:states'

Якщо`column`параметр не вказаний, буде використано назву поля.

<a name="specifying-a-custom-column-name"></a>

#### Вказівка ​​власного імені стовпця

    'state' => 'exists:states,abbreviation'

Іноді вам може знадобитися вказати конкретне підключення до бази даних, яке буде використовуватися для`exists`запит. Ви можете досягти цього, додавши ім'я з'єднання до імені таблиці, використовуючи синтаксис "крапка":

    'email' => 'exists:connection.staff,email'

Замість того, щоб безпосередньо вказувати назву таблиці, ви можете вказати модель Eloquent, яку слід використовувати для визначення назви таблиці:

    'user_id' => 'exists:App\Models\User,id'

Якщо ви хочете налаштувати запит, який виконується правилом перевірки, ви можете використовувати`Rule`клас, щоб вільно визначити правило. У цьому прикладі ми також визначимо правила перевірки як масив замість того, щоб використовувати`|`символ для їх розмежування:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::exists('staff')->where(function ($query) {
                $query->where('account_id', 1);
            }),
        ],
    ]);

<a name="rule-file"></a>

#### файл

Поле, що перевіряється, має бути успішно завантаженим файлом.

<a name="rule-filled"></a>

#### заповнені

Поле, що перевіряється, не повинно бути порожнім, коли воно є.

<a name="rule-gt"></a>

#### gt:_поле_

Поле, що перевіряється, має бути більше заданого_поле_. Два поля мають бути одного типу. Рядки, числа, масиви та файли обчислюються за тими ж умовами, що і[`size`](#rule-size)правило.

<a name="rule-gte"></a>

#### gte:_поле_

Поле, що перевіряється, має бути більше або дорівнює заданому_поле_. Два поля мають бути одного типу. Рядки, числа, масиви та файли обчислюються за тими ж умовами, що і[`size`](#rule-size)правило.

<a name="rule-image"></a>

#### зображення

Файл, що перевіряється, повинен бути зображенням (jpeg, png, bmp, gif, svg або webp)

<a name="rule-in"></a>

#### у:_foo_,_бар_,...

Поле, що перевіряється, має бути включене до наведеного списку значень. Оскільки це правило часто вимагає від вас`implode`масив,`Rule::in`метод може бути використаний для вільного побудови правила:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'zones' => [
            'required',
            Rule::in(['first-zone', 'second-zone']),
        ],
    ]);

<a name="rule-in-array"></a>

#### в_масив: \_anotherfield_.\*

Поле, що перевіряється, повинно існувати у_інше поле_значення.

<a name="rule-integer"></a>

#### ціле число

Поле, що перевіряється, має бути цілим числом.

> {note} Це правило перевірки не підтверджує, що вхідні дані мають тип змінної "ціле число", а лише те, що введенням є рядок або числове значення, яке містить ціле число.

<a name="rule-ip"></a>

#### ip

Поле, що перевіряється, має бути IP-адресою.

<a name="ipv4"></a>

#### ipv4

Поле, що перевіряється, має бути адресою IPv4.

<a name="ipv6"></a>

#### ipv6

Поле, що перевіряється, має бути адресою IPv6.

<a name="rule-json"></a>

#### json

Поле, що перевіряється, має бути дійсним рядком JSON.

<a name="rule-lt"></a>

#### lt:_поле_

Поле, що перевіряється, має бути менше заданого_поле_. Два поля мають бути одного типу. Рядки, числа, масиви та файли обчислюються за тими ж умовами, що і[`size`](#rule-size)правило.

<a name="rule-lte"></a>

#### lte:_поле_

Поле, що перевіряється, має бути меншим або рівним заданому_поле_. Два поля мають бути одного типу. Рядки, числа, масиви та файли обчислюються за тими ж умовами, що і[`size`](#rule-size)правило.

<a name="rule-max"></a>

#### макс:_значення_

Поле, що перевіряється, має бути менше або дорівнювати максимуму_значення_. Рядки, числа, масиви та файли оцінюються так само, як і[`size`](#rule-size)правило.

<a name="rule-mimetypes"></a>

#### міметипи:_текст / звичайний_,...

Файл, що перевіряється, повинен відповідати одному з поданих типів MIME:

    'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'

Щоб визначити тип MIME завантаженого файлу, його вміст буде прочитано, і фреймворк спробує вгадати тип MIME, який може відрізнятися від наданого клієнтом типу MIME.

<a name="rule-mimes"></a>

#### міми:_foo_,_бар_,...

Файл, що перевіряється, повинен мати тип MIME, що відповідає одному з перелічених розширень.

<a name="basic-usage-of-mime-rule"></a>

#### Основне використання правила MIME

    'photo' => 'mimes:jpeg,bmp,png'

Незважаючи на те, що вам потрібно лише вказати розширення, це правило фактично перевіряє тип MIME файлу, читаючи вміст файлу та вгадуючи його тип MIME.

Повний перелік типів MIME та відповідних розширень можна знайти за адресою:<https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types>

<a name="rule-min"></a>

#### хв:_значення_

Поле, що перевіряється, має містити мінімум_значення_. Рядки, числа, масиви та файли оцінюються так само, як і[`size`](#rule-size)правило.

<a name="multiple-of"></a>

#### множинні_з: \_value_

Поле, що перевіряється, має бути кратним_значення_.

<a name="rule-not-in"></a>

#### ні_у: \_foo_,_бар_,...

Поле, що перевіряється, не повинно бути включене до поданого списку значень.`Rule::notIn`метод може бути використаний для вільного побудови правила:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'toppings' => [
            'required',
            Rule::notIn(['sprinkles', 'cherries']),
        ],
    ]);

<a name="rule-not-regex"></a>

#### ні_регулярний вираз: візерунок_

Поле, що перевіряється, не повинно відповідати заданому регулярному виразу.

Внутрішньо це правило використовує PHP`preg_match`функція. Зазначений шаблон повинен відповідати тому самому форматуванню, що вимагається`preg_match`і, таким чином, також включати дійсні роздільники Наприклад:`'email' => 'not_regex:/^.+$/i'`.

**Примітка:**При використанні`regex`/`not_regex`шаблонів, може знадобитися вказати правила в масиві замість використання роздільників контурів, особливо якщо регулярний вираз містить символ конвеєра.

<a name="rule-nullable"></a>

#### нульовий

Поле, що перевіряється, може бути`null`. Це особливо корисно під час перевірки примітивів, таких як рядки та цілі числа, які можуть містити`null`значення.

<a name="rule-numeric"></a>

#### числовий

Поле, що перевіряється, має бути числовим.

<a name="rule-password"></a>

#### пароль

Поле, що перевіряється, повинно відповідати паролю користувача, що пройшов аутентифікацію. Ви можете вказати захист автентифікації, використовуючи перший параметр правила:

    'password' => 'password:api'

<a name="rule-present"></a>

#### сьогодення

Поле, що перевіряється, має бути присутнім у вхідних даних, але може бути порожнім.

<a name="rule-regex"></a>

#### регулярний вираз:_візерунок_

Поле, що перевіряється, має відповідати заданому регулярному виразу.

Внутрішньо це правило використовує PHP`preg_match`функція. Зазначений шаблон повинен відповідати тому самому форматуванню, що вимагається`preg_match`і, таким чином, також включати дійсні роздільники Наприклад:`'email' => 'regex:/^.+@.+$/i'`.

**Note:**При використанні`regex`/`not_regex`шаблонів, може знадобитися вказати правила в масиві замість використання роздільників контурів, особливо якщо регулярний вираз містить символ конвеєра.

<a name="rule-required"></a>

#### вимагається

Поле, що перевіряється, має бути присутнім у вхідних даних, а не порожнім. Поле вважається "порожнім", якщо виконується одна з наступних умов:

<div class="content-list" markdown="1">

-   Значення`null`.
-   Значення - це порожній рядок.
-   Значення є порожнім масивом або порожнім`Countable`об'єкт.
-   Значення - це завантажений файл без шляху.

</div>

<a name="rule-required-if"></a>

#### вимагається_if: \_anotherfield_,_значення_,...

Поле, що перевіряється, має бути присутнім і не порожнім, якщо_інше поле_поле дорівнює будь-якому_значення_.

Якщо ви хочете побудувати більш складну умову для`required_if`правило, ви можете використовувати`Rule::requiredIf`метод. Цей метод приймає логічне значення або Закриття. Коли пройде закриття, закриття має повернутися`true`або`false`щоб вказати, чи потрібно поле для перевірки:

    use Illuminate\Validation\Rule;

    Validator::make($request->all(), [
        'role_id' => Rule::requiredIf($request->user()->is_admin),
    ]);

    Validator::make($request->all(), [
        'role_id' => Rule::requiredIf(function () use ($request) {
            return $request->user()->is_admin;
        }),
    ]);

<a name="rule-required-unless"></a>

#### вимагається_хіба що: \_ані інше поле_,_значення_,...

Поле, що перевіряється, має бути присутнім і не бути порожнім, якщо не вказано_інше поле_поле дорівнює будь-якому_значення_.

<a name="rule-required-with"></a>

#### вимагається_з: \_foo_,_бар_,...

Поле, що перевіряється, має бути присутнім і не порожнім_тільки якщо_будь-яке інше вказане поле присутнє.

<a name="rule-required-with-all"></a>

#### вимагається_with_all: \_foo_,_бар_,...

Поле, що перевіряється, має бути присутнім і не порожнім_тільки якщо_усі інші вказані поля присутні.

<a name="rule-required-without"></a>

#### вимагається_без: \_foo_,_бар_,...

Поле, що перевіряється, має бути присутнім і не порожнім_лише коли_жодне з інших зазначених полів відсутнє.

<a name="rule-required-without-all"></a>

#### вимагається_без\_всі: \_foo_,_бар_,...

Поле, що перевіряється, має бути присутнім і не порожнім_лише коли_всі інші вказані поля відсутні.

<a name="rule-same"></a>

#### те саме:_поле_

Дане_поле_має відповідати полю, що перевіряється.

<a name="rule-size"></a>

#### розмір:_значення_

Поле, що перевіряється, має мати розмір, що відповідає заданому_значення_. Для рядкових даних,_значення_відповідає кількості символів. Для числових даних:_значення_відповідає заданому цілочисельному значенню (атрибут також повинен мати`numeric`або`integer`правило). Для масиву,_розмір_відповідає`count`масиву. Для файлів,_розмір_відповідає розміру файлу в кілобайтах. Давайте розглянемо кілька прикладів:

    // Validate that a string is exactly 12 characters long...
    'title' => 'size:12';

    // Validate that a provided integer equals 10...
    'seats' => 'integer|size:10';

    // Validate that an array has exactly 5 elements...
    'tags' => 'array|size:5';

    // Validate that an uploaded file is exactly 512 kilobytes...
    'image' => 'file|size:512';

<a name="rule-starts-with"></a>

#### починається_з: \_foo_,_бар_,...

Поле, що перевіряється, повинно починатися з одного із заданих значень.

<a name="rule-string"></a>

#### рядок

Поле, що перевіряється, має бути рядком. Якщо ви хочете, щоб поле також було`null`, вам слід призначити`nullable`правило до поля.

<a name="rule-timezone"></a>

#### часовий пояс

Поле, що перевіряється, має бути дійсним ідентифікатором часового поясу відповідно до`timezone_identifiers_list`Функція PHP.

<a name="rule-unique"></a>

#### унікальний:_таблиця_,_стовпець_,_крім_,_idColumn_

Поле, що перевіряється, не повинно існувати в даній таблиці бази даних.

**Вказівка ​​власної назви таблиці / стовпця:**

Замість того, щоб безпосередньо вказувати назву таблиці, ви можете вказати модель Eloquent, яку слід використовувати для визначення назви таблиці:

    'email' => 'unique:App\Models\User,email_address'

`column`Опція може бути використана для вказівки відповідного стовпця бази даних поля. Якщо`column`параметр не вказаний, буде використано назву поля.

    'email' => 'unique:users,email_address'

**Спеціальне підключення до бази даних**

Іноді вам може знадобитися встановити користувальницьке підключення для запитів до бази даних, зроблених засобом перевірки. Як видно вище, налаштування`unique:users`як правило перевірки використовуватиме підключення до бази даних за замовчуванням для запиту до бази даних. Щоб замінити це, вкажіть підключення та назву таблиці, використовуючи синтаксис "крапка":

    'email' => 'unique:connection.users,email_address'

**Примушування унікального правила ігнорувати даний ідентифікатор:**

Іноді, можливо, ви захочете проігнорувати заданий ідентифікатор під час унікальної перевірки. Наприклад, розглянемо екран "оновлення профілю", який включає ім'я користувача, адресу електронної пошти та місцезнаходження. Можливо, ви захочете перевірити, що адреса електронної пошти унікальна. Однак, якщо користувач змінює лише поле імені, а не поле електронної пошти, ви не хочете, щоб виникала помилка перевірки, оскільки користувач уже є власником адреси електронної пошти.

Щоб доручити валідатору ігнорувати ідентифікатор користувача, ми використаємо`Rule`клас, щоб вільно визначити правило. У цьому прикладі ми також визначимо правила перевірки як масив замість того, щоб використовувати`|`символ для розмежування правил:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::unique('users')->ignore($user->id),
        ],
    ]);

> {note} Ви ніколи не повинні передавати будь-які введені користувачем запити, введені в`ignore`метод. Натомість слід передавати лише згенерований системою унікальний ідентифікатор, такий як автоматично зростаючий ідентифікатор або UUID, з екземпляра моделі Eloquent. В іншому випадку ваш додаток буде вразливим до атаки введення SQL.

Замість передачі значення ключа моделі в`ignore`метод, ви можете передати весь екземпляр моделі. Laravel автоматично витягне ключ із моделі:

    Rule::unique('users')->ignore($user)

Якщо у вашій таблиці використовується ім'я стовпця первинного ключа, відмінне від`id`, Ви можете вказати назву стовпця під час виклику`ignore`метод:

    Rule::unique('users')->ignore($user->id, 'user_id')

За замовчуванням`unique`правило перевіряє унікальність стовпця, що відповідає імені атрибута, що перевіряється. Однак ви можете передати інше ім'я стовпця як другий аргумент`unique`метод:

    Rule::unique('users', 'email_address')->ignore($user->id),

**Додавання додаткових речень Where:**

Ви також можете вказати додаткові обмеження запиту, налаштувавши запит за допомогою`where`метод. Наприклад, додамо обмеження, яке перевіряє`account_id`є`1`:

    'email' => Rule::unique('users')->where(function ($query) {
        return $query->where('account_id', 1);
    })

<a name="rule-url"></a>

#### url

Поле, що перевіряється, має бути дійсною URL-адресою.

<a name="rule-uuid"></a>

#### uuid

Поле, що перевіряється, має бути дійсним RFC 4122 (версія 1, 3, 4 або 5), універсально унікальним ідентифікатором (UUID).

<a name="conditionally-adding-rules"></a>

## Умовно додавання правил

<a name="skipping-validation-when-fields-have-certain-values"></a>

#### Пропуск перевірки, коли поля мають певні значення

Ви можете іноді побажати не перевіряти дане поле, якщо інше поле має задане значення. Ви можете досягти цього за допомогою`exclude_if`правило перевірки. У цьому прикладі`appointment_date`і`doctor_name`поля не перевірятимуться, якщо`has_appointment`поле має значення`false`:

    $v = Validator::make($data, [
        'has_appointment' => 'required|bool',
        'appointment_date' => 'exclude_if:has_appointment,false|required|date',
        'doctor_name' => 'exclude_if:has_appointment,false|required|string',
    ]);

Крім того, ви можете використовувати`exclude_unless`правило не перевіряти дане поле, якщо інше поле не має заданого значення:

    $v = Validator::make($data, [
        'has_appointment' => 'required|bool',
        'appointment_date' => 'exclude_unless:has_appointment,true|required|date',
        'doctor_name' => 'exclude_unless:has_appointment,true|required|string',
    ]);

<a name="validating-when-present"></a>

#### Перевірка при наявності

У деяких ситуаціях, можливо, ви захочете запустити перевірки перевірки щодо поля**лише**якщо це поле присутнє у вхідному масиві. Щоб швидко досягти цього, додайте`sometimes`правило до вашого списку правил:

    $v = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);

У наведеному вище прикладі`email`поле буде перевірено, лише якщо воно присутнє в`$data`масив.

> {tip} Якщо ви намагаєтесь перевірити поле, яке повинно бути завжди, але може бути порожнім, відвідайте[це примітка щодо необов’язкових полів](#a-note-on-optional-fields)

<a name="complex-conditional-validation"></a>

#### Складна умовна перевірка

Іноді вам може знадобитися додати правила перевірки на основі більш складної умовної логіки. Наприклад, можливо, ви захочете зажадати дане поле лише у тому випадку, якщо інше поле має більше значення, ніж 100. Або, можливо, вам знадобляться два поля, щоб мати задане значення, лише коли є інше поле. Додавання цих правил перевірки не повинно викликати труднощів. Спочатку створіть`Validator`екземпляр із вашим_статичні правила_що ніколи не змінюється:

    $v = Validator::make($data, [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);

Припустимо, наш веб-додаток призначений для збирачів ігор. Якщо збирач ігор реєструється в нашому додатку, і у них є понад 100 ігор, ми хочемо, щоб вони пояснили, чому вони володіють такою кількістю ігор. Наприклад, можливо, вони ведуть магазин перепродажу ігор, а може, їм просто подобається колекціонувати. Щоб умовно додати цю вимогу, ми можемо використовувати`sometimes`метод на`Validator`інстанції.

    $v->sometimes('reason', 'required|max:500', function ($input) {
        return $input->games >= 100;
    });

Перший аргумент, переданий в`sometimes`метод - це назва поля, яке ми умовно перевіряємо. Другий аргумент - це список правил, які ми хочемо додати. Якщо`Closure`передається, коли повертається третій аргумент`true`, правила будуть додані. Цей метод полегшує створення складних умовних перевірок. Ви навіть можете додати умовні перевірки для кількох полів одночасно:

    $v->sometimes(['reason', 'cost'], 'required', function ($input) {
        return $input->games >= 100;
    });

> {tip} The`$input`параметр переданий у ваш`Closure`буде екземпляром`Illuminate\Support\Fluent`і може використовуватися для доступу до ваших даних та файлів.

<a name="validating-arrays"></a>

## Перевірка масивів

Перевірка полів введення форми на основі масиву не повинна бути болем. Ви можете використовувати "крапкове позначення" для перевірки атрибутів у масиві. Наприклад, якщо вхідний запит HTTP містить файл`photos[profile]`поле, ви можете перевірити це так:

    $validator = Validator::make($request->all(), [
        'photos.profile' => 'required|image',
    ]);

Ви також можете перевірити кожен елемент масиву. Наприклад, щоб перевірити, що кожне повідомлення електронної пошти в даному полі введення масиву є унікальним, ви можете зробити наступне:

    $validator = Validator::make($request->all(), [
        'person.*.email' => 'email|unique:users',
        'person.*.first_name' => 'required_with:person.*.last_name',
    ]);

Так само ви можете використовувати`*`символ при вказівці повідомлень валідації у ваших мовних файлах, що робить легким використання одного повідомлення перевірки для полів на основі масиву:

    'custom' => [
        'person.*.email' => [
            'unique' => 'Each person must have a unique e-mail address',
        ]
    ],

<a name="custom-validation-rules"></a>

## Спеціальні правила перевірки

<a name="using-rule-objects"></a>

### Використання об’єктів правила

Laravel пропонує безліч корисних правил перевірки; однак, можливо, ви захочете вказати деякі власні. Одним із методів реєстрації власних правил перевірки є використання об'єктів правил. Щоб створити новий об'єкт правила, ви можете використовувати`make:rule`Artisan командування. Давайте використаємо цю команду, щоб сформувати правило, яке перевіряє рядок у верхньому регістрі. Laravel помістить нове правило в`app/Rules`каталог:

    php artisan make:rule Uppercase

Після створення правила ми готові визначити його поведінку. Об'єкт правила містить два методи:`passes`і`message`.`passes`метод отримує значення і ім'я атрибута і повинен повернутись`true`або`false`залежно від того, чи є значення атрибута дійсним чи ні.`message`метод повинен повертати повідомлення про помилку перевірки, яке слід використовувати, коли перевірка не вдається:

    <?php

    namespace App\Rules;

    use Illuminate\Contracts\Validation\Rule;

    class Uppercase implements Rule
    {
        /**
         * Determine if the validation rule passes.
         *
         * @param  string  $attribute
         * @param  mixed  $value
         * @return bool
         */
        public function passes($attribute, $value)
        {
            return strtoupper($value) === $value;
        }

        /**
         * Get the validation error message.
         *
         * @return string
         */
        public function message()
        {
            return 'The :attribute must be uppercase.';
        }
    }

Ви можете зателефонувати до`trans`помічник від вашого`message`метод, якщо ви хочете повернути повідомлення про помилку з ваших файлів перекладу:

    /**
     * Get the validation error message.
     *
     * @return string
     */
    public function message()
    {
        return trans('validation.uppercase');
    }

Після того, як правило визначено, ви можете приєднати його до валідатора, передавши екземпляр об’єкта правила разом з іншими правилами перевірки:

    use App\Rules\Uppercase;

    $request->validate([
        'name' => ['required', 'string', new Uppercase],
    ]);

<a name="using-closures"></a>

### Використання замикань

Якщо вам потрібна функціональність спеціального правила лише один раз у всій програмі, ви можете використовувати Закриття замість об’єкта правила. Закриття отримує ім’я атрибута, значення атрибута та a`$fail`зворотний виклик, який слід викликати, якщо перевірка не вдається:

    $validator = Validator::make($request->all(), [
        'title' => [
            'required',
            'max:255',
            function ($attribute, $value, $fail) {
                if ($value === 'foo') {
                    $fail($attribute.' is invalid.');
                }
            },
        ],
    ]);

<a name="using-extensions"></a>

### Використання розширень

Інший метод реєстрації власних правил перевірки - використання`extend`метод на`Validator`[фасад](/docs/{{version}}/facades). Давайте використаємо цей метод у межах[постачальник послуг](/docs/{{version}}/providers)для реєстрації власного правила перевірки:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Validator;

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
            Validator::extend('foo', function ($attribute, $value, $parameters, $validator) {
                return $value == 'foo';
            });
        }
    }

Спеціальний валідатор Закриття отримує чотири аргументи: ім'я`$attribute`перевіряється,`$value`атрибута, масив`$parameters`перейшло до правила, а`Validator`інстанції.

Ви також можете передати клас і метод до`extend`метод замість Закриття:

    Validator::extend('foo', 'FooValidator@validate');

<a name="defining-the-error-message"></a>

#### Визначення повідомлення про помилку

Вам також потрібно буде визначити повідомлення про помилку для власного правила. Ви можете зробити це, використовуючи вбудований спеціальний масив повідомлень, або додавши запис у файл мови перевірки. Це повідомлення слід розміщувати на першому рівні масиву, а не в межах`custom`масив, що стосується лише повідомлень про помилки, характерних для атрибутів:

    "foo" => "Your input was invalid!",

    "accepted" => "The :attribute must be accepted.",

    // The rest of the validation error messages...

Створюючи власне правило перевірки, іноді може знадобитися визначити власні заміни заповнювачів для повідомлень про помилки. Ви можете зробити це, створивши власний Валідатор, як описано вище, а потім зателефонувавши до`replacer`метод на`Validator`фасад. Ви можете зробити це в межах`boot`метод a[постачальник послуг](/docs/{{version}}/providers):

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Validator::extend(...);

        Validator::replacer('foo', function ($message, $attribute, $rule, $parameters) {
            return str_replace(...);
        });
    }

<a name="implicit-extensions"></a>

### Неявне розширення

За замовчуванням, коли атрибут, що перевіряється, відсутній або містить порожній рядок, звичайні правила перевірки, включаючи власні розширення, не запускаються. Наприклад,[`unique`](#rule-unique)правило не буде запущене проти порожнього рядка:

    $rules = ['name' => 'unique:users,name'];

    $input = ['name' => ''];

    Validator::make($input, $rules)->passes(); // true

Щоб правило працювало навіть тоді, коли атрибут порожній, правило повинно означати, що атрибут є обов’язковим. Щоб створити таке "неявне" розширення, використовуйте`Validator::extendImplicit()`метод:

    Validator::extendImplicit('foo', function ($attribute, $value, $parameters, $validator) {
        return $value == 'foo';
    });

> {note} Тільки "неявне" розширення_передбачає_що атрибут є обов’язковим. Чи справді він анулює відсутність чи порожній атрибут, вирішувати вам.

<a name="implicit-rule-objects"></a>

#### Неявні об'єкти правила

Якщо ви хочете, щоб об'єкт правила запускався, коли атрибут порожній, вам слід реалізувати файл`Illuminate\Contracts\Validation\ImplicitRule`інтерфейс. Цей інтерфейс служить "інтерфейсом маркера" для валідатора; отже, він не містить жодних методів, які вам потрібно реалізувати.
