# Компіляція активів (Mix)

-   [Вступ](#introduction)
-   [Встановлення та налаштування](#installation)
-   [Running Mix](#running-mix)
-   [Робота з таблицями стилів](#working-with-stylesheets)
    -   [Less](#less)
    -   [Sass](#sass)
    -   [Stylus](#stylus)
    -   [PostCSS](#postcss)
    -   [Звичайний CSS](#plain-css)
    -   [Обробка URL-адрес](#url-processing)
    -   [Карти Source](#css-source-maps)
-   [Робота з JavaScript](#working-with-scripts)
    -   [Вилучення Vendor](#vendor-extraction)
    -   [Зреагуйте](#react)
    -   [Ваніль JS](#vanilla-js)
    -   [Спеціальна конфігурація Webpack](#custom-webpack-configuration)
-   [Копіювання файлів та каталогів](#copying-files-and-directories)
-   [Переробка версій / кеш-пам’яті](#versioning-and-cache-busting)
-   [Перезавантаження синхронізації браузера](#browsersync-reloading)
-   [Змінні середовища](#environment-variables)
-   [Повідомлення](#notifications)

<a name="introduction"></a>

## Вступ

[Laravel Mix](https://github.com/JeffreyWay/laravel-mix)забезпечує вільний API для визначення кроків побудови Webpack для вашого додатку Laravel за допомогою декількох поширених попередніх процесорів CSS та JavaScript. За допомогою простого ланцюжка методів ви можете вільно визначити конвеєр активів. Наприклад:

    mix.js('resources/js/app.js', 'public/js')
        .sass('resources/sass/app.scss', 'public/css');

Якщо ви коли-небудь були розгублені та пригнічені щодо початку роботи з Webpack та компіляцією активів, вам сподобається Laravel Mix. Однак вам не потрібно використовувати його під час розробки вашого додатка; Ви можете використовувати будь-який інструмент конвеєру активів, який хочете, а то й взагалі жоден.

<a name="installation"></a>

## Встановлення та налаштування

<a name="installing-node"></a>

#### Встановлення Node

Перш ніж запускати Mix, спочатку потрібно переконатися, що Node.js та NPM встановлені на вашому комп'ютері.

    node -v
    npm -v

За замовчуванням Laravel Homestead включає все необхідне; однак, якщо ви не використовуєте Vagrant, ви можете легко встановити останню версію Node і NPM, використовуючи прості графічні інсталятори від[їх сторінку завантаження](https://nodejs.org/en/download/).

<a name="laravel-mix"></a>

#### Laravel Mix

Залишається лише встановити Laravel Mix. У свіжій інсталяції Laravel ви знайдете`package.json`файл у кореневій частині вашої структури каталогів. За замовчуванням`package.json`файл містить усе необхідне для початку роботи. Думай про це як про своє`composer.json`файл, за винятком того, що він визначає залежності Node замість PHP. Ви можете встановити залежності, на які посилається, запустивши:

    npm install

<a name="running-mix"></a>

## Running Mix

Mix - це конфігураційний шар поверх[Веб-пакет](https://webpack.js.org), тому для запуску завдань Mix вам потрібно лише виконати один із сценаріїв NPM, який входить до складу Laravel за замовчуванням`package.json`файл:

    // Run all Mix tasks...
    npm run dev

    // Run all Mix tasks and minify output...
    npm run production

<a name="watching-assets-for-changes"></a>

#### Спостереження за активами для змін

`npm run watch`команда продовжить працювати у вашому терміналі та відстежуватиме всі відповідні файли на наявність змін. Потім Webpack автоматично перекомпілює ваші Assets, коли виявить зміни:

    npm run watch

Ви можете виявити, що в певних середовищах Webpack не оновлюється, коли файли змінюються. Якщо це так у вашій системі, подумайте про використання`watch-poll`команда:

    npm run watch-poll

<a name="working-with-stylesheets"></a>

## Робота з таблицями стилів

`webpack.mix.js`файл - це точка входу для компіляції всіх активів. Подумайте про це як про легку обгортку конфігурації навколо Webpack. Поєднання завдань можна об’єднати в ланцюжок, щоб точно визначити, як слід компілювати ваші Assets.

<a name="less"></a>

### Less

`less`метод може бути використаний для компіляції[Less](http://lesscss.org/)в CSS. Складемо наш основний`app.less`чувак`public/css/app.css`.

    mix.less('resources/less/app.less', 'public/css');

Кілька дзвінків на`less`метод може бути використаний для компіляції декількох файлів:

    mix.less('resources/less/app.less', 'public/css')
        .less('resources/less/admin.less', 'public/css');

Якщо ви хочете налаштувати ім'я файлу скомпільованого CSS, ви можете передати повний шлях до файлу як другий аргумент до`less`метод:

    mix.less('resources/less/app.less', 'public/stylesheets/styles.css');

Якщо вам потрібно замінити[основні опції Less плагінів](https://github.com/webpack-contrib/less-loader#options), ви можете передати об'єкт як третій аргумент`mix.less()`:

    mix.less('resources/less/app.less', 'public/css', {
        strictMath: true
    });

<a name="sass"></a>

### Sass

`sass`метод дозволяє компілювати[Sass](https://sass-lang.com/)в CSS. Ви можете використовувати метод приблизно так:

    mix.sass('resources/sass/app.scss', 'public/css');

Знову ж, як`less`методу, ви можете скомпілювати кілька файлів Sass у власні відповідні файли CSS і навіть налаштувати вихідний каталог отриманого CSS:

    mix.sass('resources/sass/app.sass', 'public/css')
        .sass('resources/sass/admin.sass', 'public/css/admin');

Додаткові[Параметри плагіна Node-Sass](https://github.com/sass/node-sass#options)може бути поданий як третій аргумент:

    mix.sass('resources/sass/app.sass', 'public/css', {
        precision: 5
    });

<a name="stylus"></a>

### Stylus

Подібно до Less і Sass,`stylus`метод дозволяє компілювати[Stylus](http://stylus-lang.com/)в CSS:

    mix.stylus('resources/stylus/app.styl', 'public/css');

Ви також можете встановити додаткові плагіни Stylus, такі як[Порушення](https://github.com/jescalan/rupture). Спочатку встановіть плагін через NPM (`npm install rupture`), а потім вимагайте цього у своєму дзвінку`mix.stylus()`:

    mix.stylus('resources/stylus/app.styl', 'public/css', {
        use: [
            require('rupture')()
        ]
    });

<a name="postcss"></a>

### PostCSS

[PostCSS](https://postcss.org/), потужний інструмент для перетворення вашого CSS, включений до Laravel Mix з коробки. За замовчуванням Mix використовує популярне[Автопрефікс](https://github.com/postcss/autoprefixer)плагін для автоматичного застосування всіх необхідних префіксів постачальника CSS3. Однак ви можете вільно додавати будь-які додаткові плагіни, які підходять для вашої програми. Спочатку встановіть потрібний плагін через NPM, а потім посилайтеся на нього у своєму`webpack.mix.js`файл:

    mix.sass('resources/sass/app.scss', 'public/css')
        .options({
            postCss: [
                require('postcss-css-variables')()
            ]
        });

<a name="plain-css"></a>

### Звичайний CSS

Якщо ви просто хочете об'єднати кілька простих таблиць стилів CSS в один файл, ви можете використовувати файл`styles`метод.

    mix.styles([
        'public/css/vendor/normalize.css',
        'public/css/vendor/videojs.css'
    ], 'public/css/all.css');

<a name="url-processing"></a>

### Обробка URL-адрес

Оскільки Laravel Mix побудований поверх Webpack, важливо зрозуміти кілька концепцій Webpack. Для компіляції CSS Webpack перепише і оптимізує будь-які`url()`дзвінки у ваших таблицях стилів. Хоча спочатку це може здатися дивним, це неймовірно потужна функціональність. Уявіть, що ми хочемо скомпілювати Sass, що включає відносну URL-адресу зображення:

    .example {
        background: url('../images/example.png');
    }

> {note} Абсолютні шляхи для будь-якого даного`url()`буде виключено з переписування URL-адрес. Наприклад,`url('/images/thing.png')`або`url('http://example.com/images/thing.png')`не буде змінено.

За замовчуванням знайдуть Laravel Mix та Webpack`example.png`, скопіюйте його у свій`public/images`, а потім перепишіть файл`url()`у створеній таблиці стилів. Таким чином, ваш скомпільований CSS буде таким:

    .example {
        background: url(/images/example.png?d41d8cd98f00b204e9800998ecf8427e);
    }

Якою б корисною не була ця функція, можливо, ваша існуюча структура папок вже налаштована так, як вам подобається. Якщо це так, ви можете вимкнути`url()`переписуємо так:

    mix.sass('resources/sass/app.scss', 'public/css')
        .options({
            processCssUrls: false
        });

З цим доповненням до вашого`webpack.mix.js`файл, Mix більше не відповідатиме жодному`url()`або скопіюйте ресурси у свій загальнодоступний каталог. Іншими словами, скомпільований CSS буде виглядати так само, як ви його набрали спочатку:

    .example {
        background: url("../images/thing.png");
    }

<a name="css-source-maps"></a>

### Карти Source

Хоча за замовчуванням вимкнено, вихідні карти можуть бути активовані за допомогою виклику`mix.sourceMaps()`метод у вашому`webpack.mix.js`файл. Незважаючи на те, що він має вартість компіляції / продуктивності, це забезпечить додаткову інформацію про Debugging інструментів розробника вашого браузера при використанні скомпільованих ресурсів.

    mix.js('resources/js/app.js', 'public/js')
        .sourceMaps();

<a name="style-of-source-mapping"></a>

#### Стиль зіставлення джерел

Webpack пропонує безліч[стилі відображення джерел](https://webpack.js.org/configuration/devtool/#devtool). За замовчуванням для стилю зіставлення Source Mix встановлено значення`eval-source-map`, що забезпечує швидкий час відновлення. Якщо ви хочете змінити стиль відображення, ви можете зробити це за допомогою`sourceMaps`метод:

    let productionSourceMaps = false;

    mix.js('resources/js/app.js', 'public/js')
        .sourceMaps(productionSourceMaps, 'source-map');

<a name="working-with-scripts"></a>

## Робота з JavaScript

Mix надає кілька функцій, які допоможуть вам працювати з вашими файлами JavaScript, такі як компіляція ECMAScript 2015, групування модулів, мініфікація та конкатенація простих файлів JavaScript. Ще краще, все це працює безперешкодно, не вимагаючи ні грама власної конфігурації:

    mix.js('resources/js/app.js', 'public/js');

За допомогою цього одного рядка коду ви тепер можете скористатися:

<div class="content-list" markdown="1">
- ES2015 syntax.
- Modules
- Compilation of `.vue` files.
- Minification for production environments.
</div>

<a name="vendor-extraction"></a>

### Вилучення Vendor

Одним з можливих недоліків об’єднання всіх специфічних для додатків JavaScript із вашими бібліотеками постачальників є те, що це ускладнює довгострокове кешування. Наприклад, одне оновлення коду вашої програми змусить браузер повторно завантажити всі ваші бібліотеки постачальників, навіть якщо вони не змінилися.

Якщо ви маєте намір регулярно оновлювати JavaScript вашого додатка, вам слід розглянути можливість вилучення всіх бібліотек постачальників у їх власний файл. Таким чином, зміна коду програми не вплине на кешування великого файлу`vendor.js`файл. Мікс`extract`метод робить це вітряком:

    mix.js('resources/js/app.js', 'public/js')
        .extract(['vue'])

`extract`метод приймає масив усіх бібліотек або модулів, які ви хочете розпакувати в`vendor.js`файл. На прикладі наведеного фрагмента, Mix створить такі файли:

<div class="content-list" markdown="1">
- `public/js/manifest.js`: *The Webpack manifest runtime*
- `public/js/vendor.js`: *Your vendor libraries*
- `public/js/app.js`: *Your application code*
</div>

Щоб уникнути помилок JavaScript, завантажте ці файли у належному порядку:

    <script src="/js/manifest.js"></script>
    <script src="/js/vendor.js"></script>
    <script src="/js/app.js"></script>

<a name="react"></a>

### Зреагуйте

Mix може автоматично встановити плагіни Babel, необхідні для підтримки React. Для початку замініть`mix.js()`дзвоніть за допомогою`mix.react()`:

    mix.react('resources/js/app.jsx', 'public/js');

За лаштунками Mix завантажить та включить відповідне`babel-preset-react`Плагін Babel.

<a name="vanilla-js"></a>

### Ваніль JS

Подібно до поєднання таблиць стилів з`mix.styles()`, Ви також можете комбінувати та зменшувати будь-яку кількість файлів JavaScript із`scripts()`метод:

    mix.scripts([
        'public/js/admin.js',
        'public/js/dashboard.js'
    ], 'public/js/all.js');

Цей параметр особливо корисний для застарілих проектів, де вам не потрібна компіляція Webpack для вашого JavaScript.

> {tip} Невелика варіація`mix.scripts()`є`mix.babel()`. Його підпис методу ідентичний`scripts`; однак, об'єднаний файл отримає компіляцію Babel, яка перекладе будь-який код ES2015 у ванільний JavaScript, який зрозуміють всі браузери.

<a name="custom-webpack-configuration"></a>

### Спеціальна конфігурація Webpack

За лаштунками Laravel Mix посилається на попередньо налаштований`webpack.config.js`файл, щоб якнайшвидше розпочати роботу. Іноді вам може знадобитися вручну змінити цей файл. Можливо, у вас є спеціальний завантажувач або плагін, на який потрібно посилатися, або, можливо, ви віддаєте перевагу використовувати Stylus замість Sass. У таких випадках у вас є два варіанти:

<a name="merging-custom-configuration"></a>

#### Злиття власної конфігурації

Мікс забезпечує корисне`webpackConfig`метод, який дозволяє об'єднати будь-які короткі заміни конфігурації Webpack. Це особливо привабливий вибір, оскільки він не вимагає копіювати та зберігати власну копію`webpack.config.js`файл.`webpackConfig`метод приймає об'єкт, який повинен містити будь-який[Конфігурація веб-пакета](https://webpack.js.org/configuration/)що ви хочете подати заявку.

    mix.webpackConfig({
        resolve: {
            modules: [
                path.resolve(__dirname, 'vendor/laravel/spark/resources/assets/js')
            ]
        }
    });

<a name="custom-configuration-files"></a>

#### Файли спеціальної конфігурації

Якщо ви хочете повністю налаштувати конфігурацію Webpack, скопіюйте файл`node_modules/laravel-mix/setup/webpack.config.js`файл у кореневий каталог вашого проекту. Далі вкажіть усі`--config`посилання у вашому`package.json`файл у щойно скопійований файл конфігурації. Якщо ви вирішите застосувати такий підхід до налаштування, будь-які майбутні оновлення Mix's`webpack.config.js`повинні бути вручну об’єднані у ваш налаштований файл.

<a name="copying-files-and-directories"></a>

## Копіювання файлів та каталогів

`copy`метод може бути використаний для копіювання файлів та каталогів у нові місця. Це може бути корисно, коли певний актив у вашому`node_modules`каталог потрібно перенести у ваш`public`папку.

    mix.copy('node_modules/foo/bar.css', 'public/css/bar.css');

При копіюванні каталогу файл`copy`метод згладить структуру каталогу. Щоб зберегти оригінальну структуру каталогу, слід використовувати`copyDirectory`замість цього:

    mix.copyDirectory('resources/img', 'public/img');

<a name="versioning-and-cache-busting"></a>

## Переробка версій / кеш-пам’яті

Багато розробників суфіксують свої скомпільовані ресурси з позначкою часу або унікальним маркером, щоб змусити браузери завантажувати нові ресурси, а не обслуговувати застарілі копії коду. Mix може впоратися з цим, використовуючи`version`метод.

`version`метод автоматично додає унікальний хеш до імен усіх скомпільованих файлів, що дозволяє зручніше перебити кеш:

    mix.js('resources/js/app.js', 'public/js')
        .version();

Після генерації версійного файлу ви не будете знати точної назви файлу. Отже, вам слід використовувати глобальний Laravel`mix`функція у вашому[погляди](/docs/{{version}}/views)завантажити належним чином хешований актив.`mix`функція автоматично визначає поточну назву хешованого файлу:

    <script src="{{ mix('/js/app.js') }}"></script>

Оскільки версійні файли, як правило, непотрібні при розробці, ви можете доручити процесу керування версіями запускатися лише протягом`npm run production`:

    mix.js('resources/js/app.js', 'public/js');

    if (mix.inProduction()) {
        mix.version();
    }

<a name="custom-mix-base-urls"></a>

#### Спеціальні базові URL-адреси міксів

Якщо ваші компільовані об’єкти Mix розгорнуті на CDN окремо від вашої програми, вам потрібно буде змінити базову URL-адресу, створену`mix`функція. Ви можете зробити це, додавши`mix_url`варіант конфігурації для вашого`config/app.php`файл конфігурації:

    'mix_url' => env('MIX_ASSET_URL', null)

Після налаштування URL-адреси змішування,`mix`функція буде префіксувати налаштовану URL-адресу при генерації URL-адрес для активів:

    https://cdn.example.com/js/app.js?id=1964becbdd96414518cd

<a name="browsersync-reloading"></a>

## Перезавантаження синхронізації браузера

[BrowserSync](https://browsersync.io/)може автоматично контролювати ваші файли на наявність змін та вводити ваші зміни у браузер, не вимагаючи оновлення вручну. Ви можете ввімкнути підтримку, зателефонувавши до`mix.browserSync()`метод:

    mix.browserSync('my-domain.test');

    // Or...

    // https://browsersync.io/docs/options
    mix.browserSync({
        proxy: 'my-domain.test'
    });

Ви можете передати рядок (проксі-сервер) або об’єкт (налаштування BrowserSync) цьому методу. Далі запустіть сервер розробників Webpack за допомогою`npm run watch`команди. Тепер, коли ви змінюєте скрипт або файл PHP, спостерігайте, як браузер миттєво оновлює сторінку, щоб відобразити ваші зміни.

<a name="environment-variables"></a>

## Змінні середовища

Ви можете вводити змінні середовища в Mix, додавши до вашого ключа префікс ключа`.env`файл з`MIX_`:

    MIX_SENTRY_DSN_PUBLIC=http://example.com

Після того, як змінна була визначена у вашому`.env`файл, ви можете отримати до нього доступ через`process.env`об'єкт. Якщо значення змінюється під час запуску`watch`завдання, вам потрібно буде перезапустити завдання:

    process.env.MIX_SENTRY_DSN_PUBLIC

<a name="notifications"></a>

## Повідомлення

Якщо доступний, Mix автоматично відображатиме повідомлення ОС для кожного пакета. Це дасть вам миттєвий зворотний зв’язок щодо успішної компіляції чи ні. Однак можуть траплятися випадки, коли ви віддаєте перевагу вимкнути ці Notification. Одним з таких прикладів може бути активація Mix на вашому виробничому сервері. Повідомлення можуть бути деактивовані через`disableNotifications`метод.

    mix.disableNotifications();
