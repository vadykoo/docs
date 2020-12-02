# Сутінки Laravel

-   [Вступ](#introduction)
-   [Встановлення](#installation)
    -   [Керування встановленнями ChromeDriver](#managing-chromedriver-installations)
    -   [Використання інших браузерів](#using-other-browsers)
-   [Починаємо](#getting-started)
    -   [Створення тестів](#generating-tests)
    -   [Бігові тести](#running-tests)
    -   [Поводження з навколишнім середовищем](#environment-handling)
    -   [Створення браузерів](#creating-browsers)
    -   [Макроси браузера](#browser-macros)
    -   [Аутентифікація](#authentication)
    -   [Міграції баз даних](#migrations)
    -   [Печиво](#cookies)
    -   [Знімок екрана](#taking-a-screenshot)
    -   [Зберігання виводу консолі на диск](#storing-console-output-to-disk)
    -   [Зберігання джерела сторінки на диску](#storing-page-source-to-disk)
-   [Взаємодія з елементами](#interacting-with-elements)
    -   [Сутінкові селектори](#dusk-selectors)
    -   [Клацнувши посилання](#clicking-links)
    -   [Текст, значення та атрибути](#text-values-and-attributes)
    -   [Використання форм](#using-forms)
    -   [Вкладання файлів](#attaching-files)
    -   [Використання клавіатури](#using-the-keyboard)
    -   [Використання миші](#using-the-mouse)
    -   [Діалоги JavaScript](#javascript-dialogs)
    -   [Селектори масштабування](#scoping-selectors)
    -   [Очікування елементів](#waiting-for-elements)
    -   [Прокрутка елемента в поле зору](#scrolling-an-element-into-view)
    -   [Виконання JavaScript](#executing-javascript)
    -   [Висловлення тверджень Vue](#making-vue-assertions)
-   [Доступні твердження](#available-assertions)
-   [Сторінки](#pages)
    -   [Генерування сторінок](#generating-pages)
    -   [Налаштування сторінок](#configuring-pages)
    -   [Перехід до сторінок](#navigating-to-pages)
    -   [Стенографічні селектори](#shorthand-selectors)
    -   [Методи сторінок](#page-methods)
-   [Компоненти](#components)
    -   [Генерування компонентів](#generating-components)
    -   [Використання компонентів](#using-components)
-   [Постійна інтеграція](#continuous-integration)
    -   [CircleCI](#running-tests-on-circle-ci)
    -   [Кодування](#running-tests-on-codeship)
    -   [Героку К.І.](#running-tests-on-heroku-ci)
    -   [Тревіс К.І.](#running-tests-on-travis-ci)
    -   [Дії GitHub](#running-tests-on-github-actions)

<a name="introduction"></a>

## Вступ

Laravel Dusk забезпечує виразний, простий у використанні API автоматизації та тестування браузера. За замовчуванням Dusk не вимагає встановлення JDK або Selenium на вашому комп'ютері. Натомість Dusk використовує автономний[ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home)встановлення. Однак ви можете використовувати будь-який інший драйвер, сумісний із Selenium, який хочете.

<a name="installation"></a>

## Встановлення

Для початку слід додати`laravel/dusk`Залежність композитора від вашого проекту:

    composer require --dev laravel/dusk

> {note} Якщо ви реєструєте постачальника послуг Dusk вручну, вам слід**ніколи**зареєструйте його у своєму виробничому середовищі, оскільки це може призвести до того, що довільні користувачі зможуть пройти автентифікацію у вашій програмі.

Після встановлення пакунка Dusk запустіть`dusk:install`Команда ремісників:

    php artisan dusk:install

A`Browser`каталог буде створений у вашому`tests`каталог і міститиме приклад тесту. Далі встановіть`APP_URL`змінної середовища у вашому`.env`файл. Це значення має відповідати URL-адресі, яку ви використовуєте для доступу до своєї програми у браузері.

Для запуску тестів використовуйте`dusk`Ремісниче командування.`dusk`команда приймає будь-який аргумент, який також приймає`phpunit`команда:

    php artisan dusk

Якщо у вас були збої в тесті під час останнього запуску`dusk`Ви можете заощадити час, повторно запустивши невдалі тести, спочатку використовуючи`dusk:fails`команда:

    php artisan dusk:fails

<a name="managing-chromedriver-installations"></a>

### Керування встановленнями ChromeDriver

Якщо ви хочете встановити іншу версію ChromeDriver, відмінну від тієї, що входить до Laravel Dusk, ви можете використовувати`dusk:chrome-driver`команда:

    # Install the latest version of ChromeDriver for your OS...
    php artisan dusk:chrome-driver

    # Install a given version of ChromeDriver for your OS...
    php artisan dusk:chrome-driver 86

    # Install a given version of ChromeDriver for all supported OSs...
    php artisan dusk:chrome-driver --all

    # Install the version of ChromeDriver that matches the detected version of Chrome / Chromium for your OS...
    php artisan dusk:chrome-driver --detect

> {note} Сутінки вимагають`chromedriver`двійкові файли для виконання. Якщо у вас виникли проблеми із запуском «Сутінків», слід переконатися, що двійкові файли виконуються за допомогою наступної команди:`chmod -R 0755 vendor/laravel/dusk/bin/`.

<a name="using-other-browsers"></a>

### Використання інших браузерів

За замовчуванням Dusk використовує Google Chrome і автономний[ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home)встановлення для запуску тестів браузера. Тим не менш, ви можете запустити власний сервер Selenium і запустити тести на будь-який браузер, який хочете.

Для початку відкрийте`tests/DuskTestCase.php`файл, який є базовим тестовим кейсом для Вашої програми. У цьому файлі ви можете видалити виклик до`startChromeDriver`метод. Це зупинить Dusk від автоматичного запуску ChromeDriver:

    /**
     * Prepare for Dusk test execution.
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }

Далі ви можете змінити`driver`спосіб підключення до обраної вами URL-адреси та порту. Крім того, ви можете змінити "бажані можливості", які повинні передаватися WebDriver:

    /**
     * Create the RemoteWebDriver instance.
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
        );
    }

<a name="getting-started"></a>

## Починаємо

<a name="generating-tests"></a>

### Створення тестів

Щоб створити тест "Сутінки", використовуйте`dusk:make`Ремісниче командування. Створений тест буде розміщено в`tests/Browser`каталог:

    php artisan dusk:make LoginTest

<a name="running-tests"></a>

### Бігові тести

Для запуску тестів браузера використовуйте`dusk`Команда ремісників:

    php artisan dusk

Якщо у вас були збої в тесті під час останнього запуску`dusk`Ви можете заощадити час, повторно виконавши невдалі тести спочатку за допомогою`dusk:fails`команда:

    php artisan dusk:fails

`dusk`команда приймає будь-який аргумент, який зазвичай приймається тестовим запуском PHPUnit, дозволяючи запускати тести лише для даного[групи](https://phpunit.de/manual/current/en/appendixes.annotations.html#appendixes.annotations.group), тощо:

    php artisan dusk --group=foo

<a name="manually-starting-chromedriver"></a>

#### Запуск ChromeDriver вручну

За замовчуванням Dusk автоматично спробує запустити ChromeDriver. Якщо це не працює для вашої конкретної системи, ви можете вручну запустити ChromeDriver перед запуском`dusk`команди. Якщо ви вирішили запустити ChromeDriver вручну, вам слід прокоментувати наступний рядок вашого`tests/DuskTestCase.php`файл:

    /**
     * Prepare for Dusk test execution.
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }

Крім того, якщо ви запускаєте ChromeDriver з порту, відмінного від 9515, вам слід змінити`driver`метод того ж класу:

    /**
     * Create the RemoteWebDriver instance.
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:9515', DesiredCapabilities::chrome()
        );
    }

<a name="environment-handling"></a>

### Поводження з навколишнім середовищем

Щоб змусити Dusk використовувати власний файл середовища під час запуску тестів, створіть файл`.env.dusk.{environment}`файл у кореневій частині вашого проекту. Наприклад, якщо ви ініціюєте`dusk`команда від вашого`local`навколишнього середовища, вам слід створити a`.env.dusk.local`файл.

Під час запуску тестів Dusk зробить резервну копію вашого`.env`файл і перейменуйте ваше середовище Сутінки на`.env`. Після завершення тестів ваш`.env`файл буде відновлено.

<a name="creating-browsers"></a>

### Створення браузерів

Для початку давайте напишемо тест, який підтверджує, що ми можемо увійти в наш додаток. Після створення тесту ми можемо змінити його, щоб перейти на сторінку входу, ввести деякі облікові дані та натиснути кнопку «Вхід». Щоб створити екземпляр браузера, зателефонуйте до`browse`метод:

    <?php

    namespace Tests\Browser;

    use App\Models\User;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Laravel\Dusk\Chrome;
    use Tests\DuskTestCase;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;

        /**
         * A basic browser test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $user = User::factory()->create([
                'email' => 'taylor@laravel.com',
            ]);

            $this->browse(function ($browser) use ($user) {
                $browser->visit('/login')
                        ->type('email', $user->email)
                        ->type('password', 'password')
                        ->press('Login')
                        ->assertPathIs('/home');
            });
        }
    }

Як ви можете бачити на прикладі вище, файл`browse`метод приймає зворотний дзвінок. Екземпляр браузера автоматично передається цьому зворотному виклику з боку Сутінків і є основним об’єктом, що використовується для взаємодії та твердження щодо вашої програми.

<a name="creating-multiple-browsers"></a>

#### Створення декількох браузерів

Іноді вам може знадобитися кілька браузерів, щоб правильно провести тест. Наприклад, для перевірки екрана чату, який взаємодіє з веб-сокетами, може знадобитися кілька браузерів. Щоб створити декілька браузерів, "запитуйте" більше одного браузера в підписі зворотного дзвінка, наданому`browse`метод:

    $this->browse(function ($first, $second) {
        $first->loginAs(User::find(1))
              ->visit('/home')
              ->waitForText('Message');

        $second->loginAs(User::find(2))
               ->visit('/home')
               ->waitForText('Message')
               ->type('message', 'Hey Taylor')
               ->press('Send');

        $first->waitForText('Hey Taylor')
              ->assertSee('Jeffrey Way');
    });

<a name="resizing-browser-windows"></a>

#### Зміна розміру браузера Windows

Ви можете використовувати`resize`спосіб регулювання розміру вікна браузера:

    $browser->resize(1920, 1080);

`maximize`метод може бути використаний для максимізації вікна браузера:

    $browser->maximize();

`fitContent`метод змінить розмір вікна браузера відповідно до розміру вмісту:

    $browser->fitContent();

Коли тест не вдається, Dusk автоматично змінює розмір браузера відповідно до вмісту, перш ніж робити знімок екрана. Ви можете вимкнути цю функцію, зателефонувавши до`disableFitOnFailure`метод у вашому тесті:

    $browser->disableFitOnFailure();

Ви можете використовувати`move`спосіб переміщення вікна браузера в інше місце на екрані:

    $browser->move(100, 100);

<a name="browser-macros"></a>

### Макроси браузера

Якщо ви хочете визначити власний метод браузера, який ви зможете повторно використовувати в різних своїх тестах, ви можете використовувати`macro`метод на`Browser`клас. Як правило, вам слід викликати цей метод із[постачальника послуг](/docs/{{version}}/providers)`boot`метод:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Laravel\Dusk\Browser;

    class DuskServiceProvider extends ServiceProvider
    {
        /**
         * Register the Dusk's browser macros.
         *
         * @return void
         */
        public function boot()
        {
            Browser::macro('scrollToElement', function ($element = null) {
                $this->script("$('html, body').animate({ scrollTop: $('$element').offset().top }, 0);");

                return $this;
            });
        }
    }

`macro`Функція приймає ім'я як перший аргумент, а Закриття - як другий аргумент. Закриття макросу буде виконано під час виклику макросу як методу на`Browser`реалізація:

    $this->browse(function ($browser) use ($user) {
        $browser->visit('/pay')
                ->scrollToElement('#credit-card-details')
                ->assertSee('Enter Credit Card Details');
    });

<a name="authentication"></a>

### Аутентифікація

Часто ви будете тестувати сторінки, які потребують автентифікації. Ви можете використовувати Сутінки`loginAs`метод, щоб уникнути взаємодії з екраном входу під час кожного тесту.`loginAs`метод приймає ідентифікатор користувача або примірник моделі користувача:

    $this->browse(function ($first, $second) {
        $first->loginAs(User::find(1))
              ->visit('/home');
    });

> {note} Після використання`loginAs`метод, сеанс користувача буде підтримуватися для всіх тестів у файлі.

<a name="migrations"></a>

### Міграції баз даних

Коли ваш тест вимагає міграції, як, наприклад, приклад автентифікації вище, ви ніколи не повинні використовувати`RefreshDatabase`риса. Чай`RefreshDatabase`trait використовує транзакції бази даних, які не застосовуватимуться до запитів HTTP. Натомість використовуйте`DatabaseMigrations`риса:

    <?php

    namespace Tests\Browser;

    use App\Models\User;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Laravel\Dusk\Chrome;
    use Tests\DuskTestCase;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;
    }

> {note} Бази даних SQLite в пам’яті не можна використовувати під час виконання Dusk-тестів. Оскільки браузер виконується в рамках власного процесу, він не зможе отримати доступ до баз даних інших процесів у пам'яті.

<a name="cookies"></a>

### Печиво

Ви можете використовувати`cookie`спосіб отримати або встановити значення зашифрованого файлу cookie:

    $browser->cookie('name');

    $browser->cookie('name', 'Taylor');

Ви можете використовувати`plainCookie`спосіб отримати або встановити значення незашифрованого файлу cookie:

    $browser->plainCookie('name');

    $browser->plainCookie('name', 'Taylor');

Ви можете використовувати`deleteCookie`спосіб видалення даного файлу cookie:

    $browser->deleteCookie('name');

<a name="taking-a-screenshot"></a>

### Знімок екрана

Ви можете використовувати`screenshot`метод зробити знімок екрана та зберегти його із вказаною назвою файлу. Всі знімки екрана зберігатимуться в`tests/Browser/screenshots`каталог:

    $browser->screenshot('filename');

<a name="storing-console-output-to-disk"></a>

### Зберігання виводу консолі на диск

Ви можете використовувати`storeConsoleLog`метод запису виводу консолі на диск із заданою назвою файлу. Висновок консолі зберігатиметься в`tests/Browser/console`каталог:

    $browser->storeConsoleLog('filename');

<a name="storing-page-source-to-disk"></a>

### Зберігання джерела сторінки на диску

Ви можете використовувати`storeSource`метод записати поточне джерело сторінки на диск із вказаною назвою файлу. Джерело сторінки буде зберігатися в`tests/Browser/source`каталог:

    $browser->storeSource('filename');

<a name="interacting-with-elements"></a>

## Взаємодія з елементами

<a name="dusk-selectors"></a>

### Сутінкові селектори

Вибір хороших селекторів CSS для взаємодії з елементами є однією з найскладніших частин написання тестів Dusk. З часом зміни інтерфейсу можуть змусити селектори CSS, як показано нижче, порушити ваші тести:

    // HTML...

    <button>Login</button>

    // Test...

    $browser->click('.login-page .container div > button');

Сутінкові селектори дозволяють зосередитись на написанні ефективних тестів, а не запам'ятовуванні селекторів CSS. Щоб визначити селектор, додайте a`dusk`атрибут для вашого елемента HTML. Потім префіксуйте селектор префіксом`@`для маніпулювання прикріпленим елементом в рамках тесту Сутінки:

    // HTML...

    <button dusk="login-button">Login</button>

    // Test...

    $browser->click('@login-button');

<a name="clicking-links"></a>

### Клацнувши посилання

Щоб натиснути посилання, ви можете використовувати`clickLink`в екземплярі браузера.`clickLink`метод клацне посилання, що містить вказаний текст відображення:

    $browser->clickLink($linkText);

Ви можете використовувати`seeLink`метод, щоб визначити, чи на сторінці видно посилання, що містить вказаний текст відображення:

    if ($browser->seeLink($linkText)) {
        // ...
    }

> {note} Ці методи взаємодіють з jQuery. Якщо jQuery недоступний на сторінці, Dusk автоматично вводить його на сторінку, щоб він був доступний протягом тривалості тесту.

<a name="text-values-and-attributes"></a>

### Текст, значення та атрибути

<a name="retrieving-setting-values"></a>

#### Отримання та встановлення значень

Сутінки надають кілька методів взаємодії з поточним відображуваним текстом, значенням та атрибутами елементів на сторінці. Наприклад, щоб отримати "значення" елемента, який відповідає даному селектору, використовуйте`value`метод:

    // Retrieve the value...
    $value = $browser->value('selector');

    // Set the value...
    $browser->value('selector', 'value');

Ви можете використовувати`inputValue`спосіб отримати "значення" вхідного елемента, який має задане ім'я поля:

    // Retrieve the value of an input element...
    $inputValue = $browser->inputValue('field');

<a name="retrieving-text"></a>

#### Отримання тексту

`text`метод може бути використаний для отримання відображуваного тексту елемента, який відповідає заданому селектору:

    $text = $browser->text('selector');

<a name="retrieving-attributes"></a>

#### Отримання атрибутів

Нарешті,`attribute`метод може бути використаний для отримання атрибута елемента, що відповідає даному селектору:

    $attribute = $browser->attribute('selector', 'value');

<a name="using-forms"></a>

### Використання форм

<a name="typing-values"></a>

#### Введення значень

Сутінки надають різноманітні методи взаємодії з формами та елементами введення. Спочатку розглянемо приклад введення тексту в поле введення:

    $browser->type('email', 'taylor@laravel.com');

Зауважте, що, хоча метод і приймає такий, якщо це необхідно, нам не потрібно передавати селектор CSS в`type`метод. Якщо селектор CSS не передбачений, Сутінки шукатимуть поле введення із заданим`name`атрибут. Нарешті, Сутінки спробують знайти`textarea`з даним`name`атрибут.

Щоб додати текст до поля, не очищаючи його вміст, ви можете використовувати`append`метод:

    $browser->type('tags', 'foo')
            ->append('tags', ', bar, baz');

Ви можете очистити значення введення за допомогою`clear`метод:

    $browser->clear('email');

Ви можете доручити Даску повільно набирати текст за допомогою`typeSlowly`метод. За замовчуванням Dusk робить паузу на 100 мілісекунд між натисканнями клавіш. Щоб налаштувати проміжок часу між натисканнями клавіш, ви можете передати відповідну кількість мілісекунд як третій аргумент методу:

    $browser->typeSlowly('mobile', '+1 (202) 555-5555');

    $browser->typeSlowly('mobile', '+1 (202) 555-5555', 300);

Ви можете використовувати`appendSlowly`метод повільного додавання тексту:

    $browser->type('tags', 'foo')
            ->appendSlowly('tags', ', bar, baz');

<a name="dropdowns"></a>

#### Випадаючі меню

Щоб вибрати значення у випадаючому вікні вибору, ви можете використовувати`select`метод. Подобається`type`метод,`select`метод не вимагає повного селектора CSS. При передачі значення в`select`методу, вам слід передати базове значення параметра замість відображуваного тексту:

    $browser->select('size', 'Large');

Ви можете вибрати випадковий варіант, опустивши другий параметр:

    $browser->select('size');

<a name="checkboxes"></a>

#### Прапорці

Щоб "позначити" поле прапорця, ви можете використовувати`check`метод. Як і багато інших методів, пов'язаних із введенням, повний селектор CSS не потрібен. Якщо точної відповідності селектора не вдається знайти, Сутінки знайдуть прапорець із відповідністю`name`атрибут:

    $browser->check('terms');

    $browser->uncheck('terms');

<a name="radio-buttons"></a>

#### Кнопки радіо

Щоб "вибрати" опцію перемикача, ви можете використовувати`radio`метод. Як і багато інших методів, пов'язаних із введенням, повний селектор CSS не потрібен. Якщо не вдається знайти точну відповідність селектора, Сутінки шукатимуть радіо з відповідними`name`і`value`атрибути:

    $browser->radio('version', 'php8');

<a name="attaching-files"></a>

### Вкладання файлів

`attach`метод може бути використаний для вкладання файлу в файл`file`вхідний елемент. Як і багато інших методів, пов'язаних із введенням, повний селектор CSS не потрібен. Якщо не вдається знайти точну відповідність селектора, Сутінки шукатимуть файл, що відповідає`name`атрибут:

    $browser->attach('photo', __DIR__.'/photos/me.png');

> {note} Функція приєднання вимагає`Zip`Розширення PHP, яке буде встановлено та увімкнено на вашому сервері.

<a name="using-the-keyboard"></a>

### Using The Keyboard

`keys`Метод дозволяє надати більш складні вхідні послідовності для даного елемента, ніж зазвичай дозволяється`type`метод. Наприклад, ви можете тримати модифікаційні клавіші, що вводять значення. У цьому прикладі`shift`ключ буде утримуватися в той час, як`taylor`вводиться в елемент, що відповідає даному селектору. Після`taylor`набирається,`otwell`буде введено без будь-яких модифікаційних клавіш:

    $browser->keys('selector', ['{shift}', 'taylor'], 'otwell');

Ви навіть можете надіслати "гарячу клавішу" до основного селектора CSS, що містить вашу програму:

    $browser->keys('.app', ['{command}', 'j']);

> {tip} Усі клавіші-модифікатори загорнуті`{}`символів і відповідають константам, визначеним у`Facebook\WebDriver\WebDriverKeys`клас, який може бути[знайдено на GitHub](https://github.com/php-webdriver/php-webdriver/blob/master/lib/WebDriverKeys.php).

<a name="using-the-mouse"></a>

### Використання миші

<a name="clicking-on-elements"></a>

#### Клацання на елементах

`click`метод може бути використаний для "клацання" на елементі, що відповідає даному селектору:

    $browser->click('.selector');

`clickAtXPath`метод може бути використаний для "клацання" на елементі, що відповідає даному виразу XPath:

    $browser->clickAtXPath('//div[@class = "selector"]');

`clickAtPoint`метод може бути використаний для "клацання" на самому верхньому елементі за заданою парою координат щодо видимої області браузера:

    $browser->clickAtPoint(0, 0);

`doubleClick`метод може бути використаний для імітації подвійного "клацання" миші:

    $browser->doubleClick();

`rightClick`метод може бути використаний для імітації клацання правою кнопкою миші:

    $browser->rightClick();

    $browser->rightClick('.selector');

`clickAndHold`метод може бути використаний для імітації натискання та утримування кнопки миші. Подальший дзвінок до`releaseMouse`метод скасує таку поведінку та відпустить кнопку миші:

    $browser->clickAndHold()
            ->pause(1000)
            ->releaseMouse();

<a name="mouseover"></a>

#### Наведення миші

`mouseover`метод може бути використаний, коли вам потрібно навести мишу на елемент, що відповідає даному селектору:

    $browser->mouseover('.selector');

<a name="drag-drop"></a>

#### Перетягування

The `drag`метод може бути використаний для перетягування елемента, що відповідає даному селектору, до іншого елемента:

    $browser->drag('.from-selector', '.to-selector');

Або ви можете перетягнути елемент в одному напрямку:

    $browser->dragLeft('.selector', 10);
    $browser->dragRight('.selector', 10);
    $browser->dragUp('.selector', 10);
    $browser->dragDown('.selector', 10);

Нарешті, ви можете перетягнути елемент на заданий зсув:

    $browser->dragOffset('.selector', 10, 10);

<a name="javascript-dialogs"></a>

### Діалоги JavaScript

Сутінки надають різні методи взаємодії з діалоговими вікнами JavaScript:

    // Wait for a dialog to appear:
    $browser->waitForDialog($seconds = null);

    // Assert that a dialog has been displayed and that its message matches the given value:
    $browser->assertDialogOpened('value');

    // Type the given value in an open JavaScript prompt dialog:
    $browser->typeInDialog('Hello World');

Щоб закрити відкрите діалогове вікно JavaScript, натисніть кнопку OK:

    $browser->acceptDialog();

Щоб закрити відкрите діалогове вікно JavaScript, натисніть кнопку Скасувати (лише для діалогового вікна підтвердження):

    $browser->dismissDialog();

<a name="scoping-selectors"></a>

### Селектори масштабування

Іноді вам може знадобитися виконати кілька операцій, одночасно визначаючи обсяг усіх операцій у межах даного селектора. Наприклад, ви можете запевнити, що якийсь текст існує лише в таблиці, а потім натиснути кнопку в цій таблиці. Ви можете використовувати`with`метод досягнення цього. Всі операції, що виконуються в рамках зворотного виклику, наданого`with`метод буде обмежений до початкового селектора:

    $browser->with('.table', function ($table) {
        $table->assertSee('Hello World')
              ->clickLink('Delete');
    });

Можливо, вам іноді доведеться виконувати твердження поза поточною сферою. Ви можете використовувати`elsewhere`метод для досягнення цього:

     $browser->with('.table', function ($table) {
        // Current scope is `body .table`...
        $browser->elsewhere('.page-title', function ($title) {
            // Current scope is `body .page-title`...
            $title->assertSee('Hello World');
        });
     });

<a name="waiting-for-elements"></a>

### Очікування елементів

При тестуванні додатків, які широко використовують JavaScript, часто виникає необхідність "почекати", поки певні елементи або дані стануть доступними, перш ніж продовжувати тест. Сутінки роблять це важким явищем. Використовуючи різні методи, ви можете зачекати, поки елементи будуть видимими на сторінці, або навіть почекати, поки даний вираз JavaScript отримає значення`true`.

<a name="waiting"></a>

#### Чекаю

Якщо вам потрібно призупинити тест на задану кількість мілісекунд, використовуйте`pause`метод:

    $browser->pause(1000);

<a name="waiting-for-selectors"></a>

#### Очікування селекторів

`waitFor`метод може бути використаний для призупинення виконання тесту, поки елемент, що відповідає даному селектору CSS, не відображається на сторінці. За замовчуванням це призупиняє тест максимум на п’ять секунд, перш ніж створювати виняток. За необхідності ви можете передати спеціальний поріг очікування як другий аргумент методу:

    // Wait a maximum of five seconds for the selector...
    $browser->waitFor('.selector');

    // Wait a maximum of one second for the selector...
    $browser->waitFor('.selector', 1);

Ви також можете почекати, поки селектор не містить заданий текст:

    // Wait a maximum of five seconds for the selector to contain the given text...
    $browser->waitForTextIn('.selector', 'Hello World');

    // Wait a maximum of one second for the selector to contain the given text...
    $browser->waitForTextIn('.selector', 'Hello World', 1);

Ви також можете почекати, поки вказаний селектор не буде відсутній на сторінці:

    // Wait a maximum of five seconds until the selector is missing...
    $browser->waitUntilMissing('.selector');

    // Wait a maximum of one second until the selector is missing...
    $browser->waitUntilMissing('.selector', 1);

<a name="scoping-selectors-when-available"></a>

#### Селектори масштабування, коли вони доступні

Іноді, можливо, ви захочете почекати на даний селектор, а потім взаємодіяти з елементом, що відповідає селектору. Наприклад, ви можете почекати, поки не з’явиться модальне вікно, а потім натиснути кнопку «ОК» у модальному режимі.`whenAvailable`у цьому випадку може бути використаний метод. Усі операції з елементами, що виконуються в рамках даного зворотного виклику, будуть масштабовані до початкового селектора:

    $browser->whenAvailable('.modal', function ($modal) {
        $modal->assertSee('Hello World')
              ->press('OK');
    });

<a name="waiting-for-text"></a>

#### Очікування тексту

`waitForText`метод може бути використаний для того, щоб почекати, поки даний текст не відобразиться на сторінці:

    // Wait a maximum of five seconds for the text...
    $browser->waitForText('Hello World');

    // Wait a maximum of one second for the text...
    $browser->waitForText('Hello World', 1);

Ви можете використовувати`waitUntilMissingText`метод почекати, поки відображений текст буде видалено зі сторінки:

    // Wait a maximum of five seconds for the text to be removed...
    $browser->waitUntilMissingText('Hello World');

    // Wait a maximum of one second for the text to be removed...
    $browser->waitUntilMissingText('Hello World', 1);

<a name="waiting-for-links"></a>

#### Чекаю посилань

`waitForLink`метод може бути використаний, щоб зачекати, поки на сторінці відобразиться текст даного посилання:

    // Wait a maximum of five seconds for the link...
    $browser->waitForLink('Create');

    // Wait a maximum of one second for the link...
    $browser->waitForLink('Create', 1);

<a name="waiting-on-the-page-location"></a>

#### Очікування на сторінці Розташування

Коли робите твердження про шлях, наприклад`$browser->assertPathIs('/home')`, твердження може провалитися, якщо`window.location.pathname`оновлюється асинхронно. Ви можете використовувати`waitForLocation`метод дочекатися, коли розташування має задане значення:

    $browser->waitForLocation('/secret');

Ви також можете дочекатися місцезнаходження вказаного маршруту:

    $browser->waitForRoute($routeName, $parameters);

<a name="waiting-for-page-reloads"></a>

#### Waiting for Page Reloads

Якщо вам потрібно зробити твердження після перезавантаження сторінки, використовуйте`waitForReload`метод:

    $browser->click('.some-action')
            ->waitForReload()
            ->assertSee('something');

<a name="waiting-on-javascript-expressions"></a>

#### Очікування на вирази JavaScript

Іноді вам може знадобитися призупинити виконання тесту, поки заданий вираз JavaScript не отримає значення`true`. Ви можете легко досягти цього за допомогою`waitUntil`метод. Передаючи вираз цьому методу, вам не потрібно включати`return`ключове слово або крапка з комою:

    // Wait a maximum of five seconds for the expression to be true...
    $browser->waitUntil('App.dataLoaded');

    $browser->waitUntil('App.data.servers.length > 0');

    // Wait a maximum of one second for the expression to be true...
    $browser->waitUntil('App.data.servers.length > 0', 1);

<a name="waiting-on-vue-expressions"></a>

#### Очікування на вирази Vue

Наступні методи можна використовувати для очікування, поки даний атрибут компонента Vue отримає задане значення:

    // Wait until the component attribute contains the given value...
    $browser->waitUntilVue('user.name', 'Taylor', '@user');

    // Wait until the component attribute doesn't contain the given value...
    $browser->waitUntilVueIsNot('user.name', null, '@user');

<a name="waiting-with-a-callback"></a>

#### Очікування із зворотним дзвінком

Багато методів "очікування" в Сутінках покладаються на основні`waitUsing`метод. Ви можете використовувати цей метод безпосередньо, щоб дочекатися повернення даного зворотного дзвінка`true`.`waitUsing`метод приймає максимальну кількість секунд очікування, інтервал, за який слід оцінити Закриття, Закриття та необов’язкове повідомлення про помилку:

    $browser->waitUsing(10, 1, function () use ($something) {
        return $something->isReady();
    }, "Something wasn't ready in time.");

<a name="scrolling-an-element-into-view"></a>

### Прокрутка елемента в поле зору

Іноді ви не можете натиснути на елемент, оскільки він знаходиться поза зоною видимості браузера.`scrollIntoView`метод буде прокручувати вікно браузера, поки елемент у даному селекторі не буде у вікні перегляду:

    $browser->scrollIntoView('selector')
            ->click('selector');

<a name="executing-javascript"></a>

### Запуск JavaScript

Ви можете використовувати`script`метод для виконання JavaScript у браузері:

    $output = $browser->script('document.documentElement.scrollTop = 0');

    $output = $browser->script([
        'document.body.scrollTop = 0',
        'document.documentElement.scrollTop = 0',
    ]);

<a name="making-vue-assertions"></a>

### Висловлення тверджень Vue

Сутінки навіть дозволяють робити твердження про стан Росії[Переглянути](https://vuejs.org)дані компонентів. Наприклад, уявіть, що ваша програма містить такий компонент Vue:

    // HTML...

    <profile dusk="profile-component"></profile>

    // Component Definition...

    Vue.component('profile', {
        template: '<div>{{ user.name }}</div>',

        data: function () {
            return {
                user: {
                    name: 'Taylor'
                }
            };
        }
    });

Ви можете стверджувати про стан компонента Vue так:

    /**
     * A basic Vue test example.
     *
     * @return void
     */
    public function testVue()
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/')
                    ->assertVue('user.name', 'Taylor', '@profile-component');
        });
    }

<a name="available-assertions"></a>

## Доступні твердження

Сутінки містять різноманітні твердження, які ви можете зробити проти своєї заявки. Усі наявні твердження задокументовані у списку нижче:

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
[assertTitle](#assert-title)
[assertTitleContains](#assert-title-contains)
[assertUrlIs](#assert-url-is)
[assertSchemeIs](#assert-scheme-is)
[assertSchemeIsNot](#assert-scheme-is-not)
[assertHostIs](#assert-host-is)
[assertHostIsNot](#assert-host-is-not)
[assertPortIs](#assert-port-is)
[assertPortIsNot](#assert-port-is-not)
[assertPathBeginsWith](#assert-path-begins-with)
[assertPathIs](#assert-path-is)
[assertPathIsNot](#assert-path-is-not)
[assertRouteIs](#assert-route-is)
[assertQueryStringHas](#assert-query-string-has)
[assertQueryStringMissing](#assert-query-string-missing)
[assertFragmentIs](#assert-fragment-is)
[assertFragmentBeginsWith](#assert-fragment-begins-with)
[assertFragmentIsNot](#assert-fragment-is-not)
[assertHasCookie](#assert-has-cookie)
[assertHasPlainCookie](#assert-has-plain-cookie)
[assertCookieMissing](#assert-cookie-missing)
[assertPlainCookieMissing](#assert-plain-cookie-missing)
[assertCookieValue](#assert-cookie-value)
[assertPlainCookieValue](#assert-plain-cookie-value)
[assertSee](#assert-see)
[assertDontSee](#assert-dont-see)
[assertSeeIn](#assert-see-in)
[assertDontSeeIn](#assert-dont-see-in)
[assertScript](#assert-script)
[assertSourceHas](#assert-source-has)
[assertSourceMissing](#assert-source-missing)
[assertSeeLink](#assert-see-link)
[assertDontSeeLink](#assert-dont-see-link)
[assertInputValue](#assert-input-value)
[assertInputValueIsNot](#assert-input-value-is-not)
[assertChecked](#assert-checked)
[assertNotChecked](#assert-not-checked)
[assertRadioSelected](#assert-radio-selected)
[assertRadioNotSelected](#assert-radio-not-selected)
[assertSelected](#assert-selected)
[assertNotSelected](#assert-not-selected)
[assertSelectHasOptions](#assert-select-has-options)
[assertSelectMissingOption](#assert-select-missing-option)
[assertSelectMissingOptions](#assert-select-missing-options)
[assertSelectHasOption](#assert-select-has-option)
[assertValue](#assert-value)
[assertAttribute](#assert-attribute)
[assertAriaAttribute](#assert-aria-attribute)
[assertDataAttribute](#assert-data-attribute)
[assertVisible](#assert-visible)
[assertPresent](#assert-present)
[assertMissing](#assert-missing)
[assertDialogOpened](#assert-dialog-opened)
[assertEnabled](#assert-enabled)
[assertDisabled](#assert-disabled)
[assertButtonEnabled](#assert-button-enabled)
[assertButtonDisabled](#assert-button-disabled)
[assertFocused](#assert-focused)
[assertNotFocused](#assert-not-focused)
[assertAuthenticated](#assert-authenticated)
[assertGuest](#assert-guest)
[assertAuthenticatedAs](#assert-authenticated-as)
[assertVue](#assert-vue)
[assertVueIsNot](#assert-vue-is-not)
[assertVueContains](#assert-vue-contains)
[assertVueDoesNotContain](#assert-vue-does-not-contain)
</div>

<a name="assert-title"></a>

#### assertTitle

Стверджуйте, що заголовок сторінки відповідає даному тексту:

    $browser->assertTitle($title);

<a name="assert-title-contains"></a>

#### assertTitleContains

Стверджуйте, що заголовок сторінки містить поданий текст:

    $browser->assertTitleContains($title);

<a name="assert-url-is"></a>

#### стверджувати

Запевнити, що поточна URL-адреса (без рядка запиту) відповідає заданому рядку:

    $browser->assertUrlIs($url);

<a name="assert-scheme-is"></a>

#### assertSchemeIs

Стверджуємо, що поточна схема URL відповідає заданій схемі:

    $browser->assertSchemeIs($scheme);

<a name="assert-scheme-is-not"></a>

#### assertSchemeIsNot

Стверджуємо, що поточна схема URL-адреси не відповідає заданій схемі:

    $browser->assertSchemeIsNot($scheme);

<a name="assert-host-is"></a>

#### assertHostIs

Запевнити, що поточний хост URL-адреси відповідає даному хосту:

    $browser->assertHostIs($host);

<a name="assert-host-is-not"></a>

#### assertHostIsNot

Запевнити, що поточний хост URL-адреси не відповідає даному хосту:

    $browser->assertHostIsNot($host);

<a name="assert-port-is"></a>

#### assertPortIs

Запевнити, що поточний порт URL відповідає даному порту:

    $browser->assertPortIs($port);

<a name="assert-port-is-not"></a>

#### assertPortIsNot

Запевнити, що поточний порт URL-адреси не відповідає даному порту:

    $browser->assertPortIsNot($port);

<a name="assert-path-begins-with"></a>

#### assertPathBeginsWith

Запевнити, що поточний шлях до URL починається з заданого шляху:

    $browser->assertPathBeginsWith($path);

<a name="assert-path-is"></a>

#### assertPathIs

Запевнити, що поточний шлях відповідає даному шляху:

    $browser->assertPathIs('/home');

<a name="assert-path-is-not"></a>

#### assertPathIsNot

Запевнити, що поточний шлях не відповідає заданому шляху:

    $browser->assertPathIsNot('/home');

<a name="assert-route-is"></a>

#### assertRouteIs

Запевнити, що поточна URL-адреса збігається із вказаною URL-адресою із вказаним маршрутом:

    $browser->assertRouteIs($name, $parameters);

<a name="assert-query-string-has"></a>

#### assertQueryStringHas

Стверджуємо, що вказаний параметр рядка запиту присутній:

    $browser->assertQueryStringHas($name);

Стверджуємо, що заданий параметр рядка запиту присутній і має задане значення:

    $browser->assertQueryStringHas($name, $value);

<a name="assert-query-string-missing"></a>

#### assertQueryStringMissing

Стверджуємо, що заданий параметр рядка запиту відсутній:

    $browser->assertQueryStringMissing($name);

<a name="assert-fragment-is"></a>

#### assertFragmentIs

Стверджуємо, що поточний фрагмент відповідає даному фрагменту:

    $browser->assertFragmentIs('anchor');

<a name="assert-fragment-begins-with"></a>

#### assertFragmentBeginsWith

Стверджуємо, що поточний фрагмент починається з даного фрагмента:

    $browser->assertFragmentBeginsWith('anchor');

<a name="assert-fragment-is-not"></a>

#### assertFragmentIsNot

Стверджуємо, що поточний фрагмент не відповідає даному фрагменту:

    $browser->assertFragmentIsNot('anchor');

<a name="assert-has-cookie"></a>

#### assertHasCookie

Стверджуємо, що вказаний зашифрований файл cookie присутній:

    $browser->assertHasCookie($name);

<a name="assert-has-plain-cookie"></a>

#### assertHasPlainCookie

Стверджуємо, що даний незашифрований файл cookie присутній:

    $browser->assertHasPlainCookie($name);

<a name="assert-cookie-missing"></a>

#### assertCookieMissing

Стверджуємо, що вказаного зашифрованого файлу cookie немає:

    $browser->assertCookieMissing($name);

<a name="assert-plain-cookie-missing"></a>

#### assertPlainCookieMissing

Стверджуємо, що даного незашифрованого файлу cookie немає:

    $browser->assertPlainCookieMissing($name);

<a name="assert-cookie-value"></a>

#### assertCookieValue

Стверджуємо, що зашифрований файл cookie має задане значення:

    $browser->assertCookieValue($name, $value);

<a name="assert-plain-cookie-value"></a>

#### assertPlainCookieValue

Стверджуємо, що незашифрований файл cookie має задане значення:

    $browser->assertPlainCookieValue($name, $value);

<a name="assert-see"></a>

#### затвердитиДив

Стверджуйте, що даний текст присутній на сторінці:

    $browser->assertSee($text);

<a name="assert-dont-see"></a>

#### assertDontSee

Стверджуйте, що наведеного тексту немає на сторінці:

    $browser->assertDontSee($text);

<a name="assert-see-in"></a>

#### assertSeeIn

Стверджуйте, що даний текст присутній у селекторі:

    $browser->assertSeeIn($selector, $text);

<a name="assert-dont-see-in"></a>

#### assertDontSeeIn

Стверджуйте, що вказаний текст відсутній у селекторі:

    $browser->assertDontSeeIn($selector, $text);

<a name="assert-script"></a>

#### assertScript

Стверджуємо, що вказаний вираз JavaScript має значення:

    $browser->assertScript('window.isLoaded')
            ->assertScript('document.readyState', 'complete');

<a name="assert-source-has"></a>

#### assertSourceHas

Стверджуйте, що даний вихідний код присутній на сторінці:

    $browser->assertSourceHas($code);

<a name="assert-source-missing"></a>

#### assertSourceMissing

Стверджуйте, що даного вихідного коду немає на сторінці:

    $browser->assertSourceMissing($code);

<a name="assert-see-link"></a>

#### assertSeeLink

Стверджуйте, що дане посилання присутнє на сторінці:

    $browser->assertSeeLink($linkText);

<a name="assert-dont-see-link"></a>

#### assertDontSeeLink

Стверджуйте, що даного посилання немає на сторінці:

    $browser->assertDontSeeLink($linkText);

<a name="assert-input-value"></a>

#### assertInputValue

Стверджуємо, що дане поле введення має вказане значення:

    $browser->assertInputValue($field, $value);

<a name="assert-input-value-is-not"></a>

#### assertInputValueIsNot

Стверджуємо, що дане поле введення не має заданого значення:

    $browser->assertInputValueIsNot($field, $value);

<a name="assert-checked"></a>

#### assertChecked

Стверджуємо, що вказаний прапорець встановлений:

    $browser->assertChecked($field);

<a name="assert-not-checked"></a>

#### assertNotChecked

Стверджуйте, що вказаний прапорець не встановлений:

    $browser->assertNotChecked($field);

<a name="assert-radio-selected"></a>

#### assertRadioSelected

Стверджуйте, що вибрано дане радіополе:

    $browser->assertRadioSelected($field, $value);

<a name="assert-radio-not-selected"></a>

#### assertRadioNotSelected

Стверджуйте, що дане радіополе не вибрано:

    $browser->assertRadioNotSelected($field, $value);

<a name="assert-selected"></a>

#### assertSelected

Стверджуємо, що в даному випадаючому списку вибрано вказане значення:

    $browser->assertSelected($field, $value);

<a name="assert-not-selected"></a>

#### assertNotSelected

Стверджуємо, що в даному випадаючому списку не вибрано вказане значення:

    $browser->assertNotSelected($field, $value);

<a name="assert-select-has-options"></a>

#### assertSelectHasOptions

Стверджуємо, що даний масив значень доступний для вибору:

    $browser->assertSelectHasOptions($field, $values);

<a name="assert-select-missing-option"></a>

#### assertSelectMissingOption

Стверджуйте, що вказане значення недоступне для вибору:

    $browser->assertSelectMissingOption($field, $value);

<a name="assert-select-missing-options"></a>

#### assertSelectMissingOptions

Стверджуємо, що даний масив значень недоступний для вибору:

    $browser->assertSelectMissingOptions($field, $values);

<a name="assert-select-has-option"></a>

#### assertSelectHasOption

Стверджуйте, що вказане значення доступне для вибору у даному полі:

    $browser->assertSelectHasOption($field, $value);

<a name="assert-value"></a>

#### assertValue

Стверджуємо, що елемент, що відповідає даному селектору, має вказане значення:

    $browser->assertValue($selector, $value);

<a name="assert-attribute"></a>

#### assertAttribute

Стверджуємо, що елемент, що відповідає даному селектору, має задане значення у наданому атрибуті:

    $browser->assertAttribute($selector, $attribute, $value);

<a name="assert-aria-attribute"></a>

#### assertAriaAttribute

Стверджуємо, що елемент, що відповідає даному селектору, має задане значення в наданому атрибуті aria:

    $browser->assertAriaAttribute($selector, $attribute, $value);

Наприклад, враховуючи розмітку`<button aria-label="Add"></button>`, ви можете стверджувати проти`aria-label`атрибут так:

    $browser->assertAriaAttribute('button', 'label', 'Add')

<a name="assert-data-attribute"></a>

#### стверджувати атрибут даних

Запевнити, що елемент, що відповідає даному селектору, має задане значення в наданому атрибуті даних:

    $browser->assertDataAttribute($selector, $attribute, $value);

Наприклад, враховуючи розмітку`<tr id="row-1" data-content="attendees"></tr>`, ви можете стверджувати проти`data-label`атрибут так:

    $browser->assertDataAttribute('#row-1', 'content', 'attendees')

<a name="assert-visible"></a>

#### assertVisible

Запевнити, що видно елемент, що відповідає даному селектору:

    $browser->assertVisible($selector);

<a name="assert-present"></a>

#### assertPresent

Стверджуємо, що присутній елемент, що відповідає даному селектору:

    $browser->assertPresent($selector);

<a name="assert-missing"></a>

#### assertMissing

Стверджуємо, що елемент, що відповідає даному селектору, не видно:

    $browser->assertMissing($selector);

<a name="assert-dialog-opened"></a>

#### assertDialogOpened

Стверджуємо, що відкрито діалогове вікно JavaScript із даним повідомленням:

    $browser->assertDialogOpened($message);

<a name="assert-enabled"></a>

#### assertEnabled

Стверджуємо, що дане поле ввімкнено:

    $browser->assertEnabled($field);

<a name="assert-disabled"></a>

#### assertDisabled

Стверджуємо, що дане поле вимкнено:

    $browser->assertDisabled($field);

<a name="assert-button-enabled"></a>

#### assertButtonEnabled

Стверджуємо, що дана кнопка ввімкнена:

    $browser->assertButtonEnabled($button);

<a name="assert-button-disabled"></a>

#### Кнопка затвердження відключена

Стверджуйте, що дана кнопка відключена:

    $browser->assertButtonDisabled($button);

<a name="assert-focused"></a>

#### assertFocused

Стверджуйте, що дане поле зосереджене:

    $browser->assertFocused($field);

<a name="assert-not-focused"></a>

#### assertNotFocused

Стверджуйте, що дане поле не сфокусовано:

    $browser->assertNotFocused($field);

<a name="assert-authenticated"></a>

#### затвердити Аутентифіковано

Стверджуємо, що автентифікація користувача:

    $browser->assertAuthenticated();

<a name="assert-guest"></a>

#### assertGuest

Стверджуйте, що користувач не пройшов аутентифікацію:

    $browser->assertGuest();

<a name="assert-authenticated-as"></a>

#### assertAuthenticatedAs

Стверджуємо, що користувач аутентифікується як даний користувач:

    $browser->assertAuthenticatedAs($user);

<a name="assert-vue"></a>

#### assertVue

Стверджуємо, що дана властивість даних компонента Vue відповідає заданому значенню:

    $browser->assertVue($property, $value, $componentSelector = null);

<a name="assert-vue-is-not"></a>

#### assertVueIsNot

Стверджуємо, що дана властивість даних компонента Vue не відповідає заданому значенню:

    $browser->assertVueIsNot($property, $value, $componentSelector = null);

<a name="assert-vue-contains"></a>

#### assertVueContens

Стверджуємо, що дана властивість даних компонента Vue є масивом і містить задане значення:

    $browser->assertVueContains($property, $value, $componentSelector = null);

<a name="assert-vue-does-not-contain"></a>

#### assertVueDoesNotContain

Стверджуємо, що дана властивість даних компонента Vue є масивом і не містить заданого значення:

    $browser->assertVueDoesNotContain($property, $value, $componentSelector = null);

<a name="pages"></a>

## Сторінки

Іноді тести вимагають декількох складних дій, які слід виконувати послідовно. Це може ускладнити читання та розуміння ваших тестів. Сторінки дозволяють визначити виразні дії, які потім можна виконати на даній сторінці за допомогою одного методу. Сторінки також дозволяють визначати ярлики до загальних селекторів для вашої програми або однієї сторінки.

<a name="generating-pages"></a>

### Генерування сторінок

Щоб створити об'єкт сторінки, використовуйте`dusk:page`Ремісниче командування. Усі об'єкти сторінки будуть розміщені в`tests/Browser/Pages`каталог:

    php artisan dusk:page Login

<a name="configuring-pages"></a>

### Налаштування сторінок

За замовчуванням сторінки мають три методи:`url`,`assert`, і`elements`. Ми обговоримо`url`і`assert`методи зараз.`elements`метод буде[докладніше обговорюється нижче](#shorthand-selectors).

<a name="the-url-method"></a>

#### `url`Метод

`url`метод повинен повертати шлях до URL-адреси, що представляє сторінку. Сутінки використовуватимуть цю URL-адресу під час переходу на сторінку в браузері:

    /**
     * Get the URL for the page.
     *
     * @return string
     */
    public function url()
    {
        return '/login';
    }

<a name="the-assert-method"></a>

#### `assert`Метод

`assert`метод може робити будь-які твердження, необхідні для перевірки того, що браузер насправді знаходиться на даній сторінці. Завершувати цей метод не потрібно; однак ви можете вільно робити ці твердження, якщо хочете. Ці твердження будуть запускатися автоматично під час переходу на сторінку:

    /**
     * Assert that the browser is on the page.
     *
     * @return void
     */
    public function assert(Browser $browser)
    {
        $browser->assertPathIs($this->url());
    }

<a name="navigating-to-pages"></a>

### Перехід до сторінок

Після налаштування сторінки ви можете перейти до неї за допомогою`visit`метод:

    use Tests\Browser\Pages\Login;

    $browser->visit(new Login);

Ви можете використовувати`visitRoute`спосіб переходу до названого маршруту:

    $browser->visitRoute('login');

Ви можете переходити "назад" і "вперед", використовуючи`back`і`forward`методи:

    $browser->back();

    $browser->forward();

Ви можете використовувати`refresh`спосіб оновити сторінку:

    $browser->refresh();

Іноді ви вже можете бути на певній сторінці і вам потрібно "завантажити" селектори та методи сторінки в поточний контекст тесту. Це характерно для натискання кнопки та перенаправлення на дану сторінку без явного переходу до неї. У цій ситуації ви можете використовувати`on`спосіб завантаження сторінки:

    use Tests\Browser\Pages\CreatePlaylist;

    $browser->visit('/dashboard')
            ->clickLink('Create Playlist')
            ->on(new CreatePlaylist)
            ->assertSee('@create');

<a name="shorthand-selectors"></a>

### Стенографічні селектори

`elements`Метод сторінок дозволяє визначити швидкі, легко запам'ятовуються ярлики для будь-якого селектора CSS на вашій сторінці. Наприклад, визначимо ярлик для поля введення "електронна пошта" сторінки входу в програму:

    /**
     * Get the element shortcuts for the page.
     *
     * @return array
     */
    public function elements()
    {
        return [
            '@email' => 'input[name=email]',
        ];
    }

Тепер ви можете використовувати цей скорочений селектор у будь-якому місці, де ви б використовували повний селектор CSS:

    $browser->type('@email', 'taylor@laravel.com');

<a name="global-shorthand-selectors"></a>

#### Глобальні стенографічні селектори

Після встановлення Dusk, основи`Page`клас буде розміщений у вашому`tests/Browser/Pages`каталог. Цей клас містить`siteElements`метод, який може бути використаний для визначення глобальних скорочувальних селекторів, які повинні бути доступні на кожній сторінці у вашій програмі:

    /**
     * Get the global element shortcuts for the site.
     *
     * @return array
     */
    public static function siteElements()
    {
        return [
            '@element' => '#selector',
        ];
    }

<a name="page-methods"></a>

### Методи сторінок

На додаток до методів за замовчуванням, визначених на сторінках, ви можете визначити додаткові методи, які можна використовувати під час ваших тестів. Наприклад, уявімо, що ми створюємо програму управління музикою. Поширеною дією для однієї сторінки програми може бути створення списку відтворення. Замість того, щоб переписувати логіку для створення списку відтворення в кожному тесті, ви можете визначити a`createPlaylist`метод на класі сторінки:

    <?php

    namespace Tests\Browser\Pages;

    use Laravel\Dusk\Browser;

    class Dashboard extends Page
    {
        // Other page methods...

        /**
         * Create a new playlist.
         *
         * @param  \Laravel\Dusk\Browser  $browser
         * @param  string  $name
         * @return void
         */
        public function createPlaylist(Browser $browser, $name)
        {
            $browser->type('name', $name)
                    ->check('share')
                    ->press('Create Playlist');
        }
    }

Після визначення методу ви можете використовувати його в будь-якому тесті, який використовує сторінку. Екземпляр браузера буде автоматично переданий методу сторінки:

    use Tests\Browser\Pages\Dashboard;

    $browser->visit(new Dashboard)
            ->createPlaylist('My Playlist')
            ->assertSee('My Playlist');

<a name="components"></a>

## Компоненти

Компоненти схожі на «об'єкти сторінки» Dusk, але призначені для частин інтерфейсу та функціональних можливостей, які повторно використовуються у всій програмі, наприклад, панелі навігації або вікна сповіщень. Як такі, компоненти не прив’язані до певних URL-адрес.

<a name="generating-components"></a>

### Генерування компонентів

Щоб створити компонент, використовуйте`dusk:component`Ремісниче командування. Нові компоненти розміщуються в`tests/Browser/Components`каталог:

    php artisan dusk:component DatePicker

Як показано вище, "засіб вибору дати" - це приклад компонента, який може існувати у всій програмі на різних сторінках. Вручну писати логіку автоматизації браузера для вибору дати в десятках тестів у вашому наборі тестів може стати важко. Натомість ми можемо визначити компонент «Сутінки» для представлення пункту вибору дати, що дозволяє нам інкапсулювати цю логіку в компоненті:

    <?php

    namespace Tests\Browser\Components;

    use Laravel\Dusk\Browser;
    use Laravel\Dusk\Component as BaseComponent;

    class DatePicker extends BaseComponent
    {
        /**
         * Get the root selector for the component.
         *
         * @return string
         */
        public function selector()
        {
            return '.date-picker';
        }

        /**
         * Assert that the browser page contains the component.
         *
         * @param  Browser  $browser
         * @return void
         */
        public function assert(Browser $browser)
        {
            $browser->assertVisible($this->selector());
        }

        /**
         * Get the element shortcuts for the component.
         *
         * @return array
         */
        public function elements()
        {
            return [
                '@date-field' => 'input.datepicker-input',
                '@year-list' => 'div > div.datepicker-years',
                '@month-list' => 'div > div.datepicker-months',
                '@day-list' => 'div > div.datepicker-days',
            ];
        }

        /**
         * Select the given date.
         *
         * @param  \Laravel\Dusk\Browser  $browser
         * @param  int  $year
         * @param  int  $month
         * @param  int  $day
         * @return void
         */
        public function selectDate($browser, $year, $month, $day)
        {
            $browser->click('@date-field')
                    ->within('@year-list', function ($browser) use ($year) {
                        $browser->click($year);
                    })
                    ->within('@month-list', function ($browser) use ($month) {
                        $browser->click($month);
                    })
                    ->within('@day-list', function ($browser) use ($day) {
                        $browser->click($day);
                    });
        }
    }

<a name="using-components"></a>

### Використання компонентів

Після того, як компонент визначений, ми можемо легко вибрати дату в засобі вибору дати з будь-якого тесту. І якщо логіка, необхідна для вибору дати, змінюється, нам потрібно лише оновити компонент:

    <?php

    namespace Tests\Browser;

    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Laravel\Dusk\Browser;
    use Tests\Browser\Components\DatePicker;
    use Tests\DuskTestCase;

    class ExampleTest extends DuskTestCase
    {
        /**
         * A basic component test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->browse(function (Browser $browser) {
                $browser->visit('/')
                        ->within(new DatePicker, function ($browser) {
                            $browser->selectDate(2019, 1, 30);
                        })
                        ->assertSee('January');
            });
        }
    }

<a name="continuous-integration"></a>

## Постійна інтеграція

> {note} Перш ніж додавати файл конфігурації безперервної інтеграції, переконайтеся, що ваш`.env.testing`файл містить файл`APP_URL`запис зі значенням`http://127.0.0.1:8000`.

<a name="running-tests-on-circle-ci"></a>

### CircleCI

Якщо ви використовуєте CircleCI для запуску своїх сутінкових тестів, ви можете використовувати цей файл конфігурації як вихідну точку. Як і TravisCI, ми будемо використовувати`php artisan serve`команда для запуску вбудованого веб-сервера PHP:

    version: 2
    jobs:
        build:
            steps:
                - run: sudo apt-get install -y libsqlite3-dev
                - run: cp .env.testing .env
                - run: composer install -n --ignore-platform-reqs
                - run: php artisan key:generate
                - run: php artisan dusk:chrome-driver
                - run: npm install
                - run: npm run production
                - run: vendor/bin/phpunit

                - run:
                    name: Start Chrome Driver
                    command: ./vendor/laravel/dusk/bin/chromedriver-linux
                    background: true

                - run:
                    name: Run Laravel Server
                    command: php artisan serve
                    background: true

                - run:
                    name: Run Laravel Dusk Tests
                    command: php artisan dusk

                - store_artifacts:
                    path: tests/Browser/screenshots

                - store_artifacts:
                    path: tests/Browser/console

                - store_artifacts:
                    path: storage/logs

<a name="running-tests-on-codeship"></a>

### Кодування

Для запуску тестів "Сутінки"[Кодування](https://codeship.com), додайте наступні команди до свого проекту Codeship. Ці команди є лише відправною точкою, і ви можете додавати додаткові команди за потреби:

    phpenv local 7.3
    cp .env.testing .env
    mkdir -p ./bootstrap/cache
    composer install --no-interaction --prefer-dist
    php artisan key:generate
    php artisan dusk:chrome-driver
    nohup bash -c "php artisan serve 2>&1 &" && sleep 5
    php artisan dusk

<a name="running-tests-on-heroku-ci"></a>

### Героку К.І.

Для запуску тестів "Сутінки"[Героку К.І.](https://www.heroku.com/continuous-integration), додайте наступний пакет збірки Google Chrome та сценарії до вашого Heroku`app.json`файл:

    {
      "environments": {
        "test": {
          "buildpacks": [
            { "url": "heroku/php" },
            { "url": "https://github.com/heroku/heroku-buildpack-google-chrome" }
          ],
          "scripts": {
            "test-setup": "cp .env.testing .env",
            "test": "nohup bash -c './vendor/laravel/dusk/bin/chromedriver-linux > /dev/null 2>&1 &' && nohup bash -c 'php artisan serve > /dev/null 2>&1 &' && php artisan dusk"
          }
        }
      }
    }

<a name="running-tests-on-travis-ci"></a>

### Тревіс К.І.

Для запуску тестів "Сутінки" на[Тревіс К.І.](https://travis-ci.org), скористайтеся наступним`.travis.yml`конфігурації. Оскільки Travis CI не є графічним середовищем, нам потрібно буде зробити кілька додаткових кроків, щоб запустити браузер Chrome. Крім того, ми будемо використовувати`php artisan serve`для запуску вбудованого веб-сервера PHP:

    language: php

    php:
      - 7.3

    addons:
      chrome: stable

    install:
      - cp .env.testing .env
      - travis_retry composer install --no-interaction --prefer-dist --no-suggest
      - php artisan key:generate
      - php artisan dusk:chrome-driver

    before_script:
      - google-chrome-stable --headless --disable-gpu --remote-debugging-port=9222 http://localhost &
      - php artisan serve &

    script:
      - php artisan dusk

<a name="running-tests-on-github-actions"></a>

### Дії GitHub

Якщо ви використовуєте[Дії Github](https://github.com/features/actions)для запуску Ваших сутінкових тестів ви можете використовувати цей файл конфігурації як початкову точку. Як і TravisCI, ми будемо використовувати`php artisan serve`команда для запуску вбудованого веб-сервера PHP:

    name: CI
    on: [push]
    jobs:

      dusk-php:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v2
          - name: Prepare The Environment
            run: cp .env.example .env
          - name: Create Database
            run: |
              sudo systemctl start mysql
              mysql --user="root" --password="root" -e "CREATE DATABASE 'my-database' character set UTF8mb4 collate utf8mb4_bin;"
          - name: Install Composer Dependencies
            run: composer install --no-progress --no-suggest --prefer-dist --optimize-autoloader
          - name: Generate Application Key
            run: php artisan key:generate
          - name: Upgrade Chrome Driver
            run: php artisan dusk:chrome-driver `/opt/google/chrome/chrome --version | cut -d " " -f3 | cut -d "." -f1`
          - name: Start Chrome Driver
            run: ./vendor/laravel/dusk/bin/chromedriver-linux &
          - name: Run Laravel Server
            run: php artisan serve &
          - name: Run Dusk Tests
            env:
              APP_URL: "http://127.0.0.1:8000"
            run: php artisan dusk
          - name: Upload Screenshots
            if: failure()
            uses: actions/upload-artifact@v2
            with:
              name: screenshots
              path: tests/Browser/screenshots
          - name: Upload Console Logs
            if: failure()
            uses: actions/upload-artifact@v2
            with:
              name: console
              path: tests/Browser/console
