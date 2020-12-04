# Зберігання файлів

-   [Вступ](#introduction)
-   [Конфігурація](#configuration)
    -   [Public диск](#the-public-disk)
    -   [Local Driver](#the-local-driver)
    -   [Передумови драйвера](#driver-prerequisites)
    -   [Кешування](#caching)
-   [Отримання екземплярів диска](#obtaining-disk-instances)
-   [Отримання файлів](#retrieving-files)
    -   [Завантаження файлів](#downloading-files)
    -   [URL-адреси файлів](#file-urls)
    -   [Шляхи до файлів](#file-paths)
    -   [Файлові метадані](#file-metadata)
-   [Зберігання файлів](#storing-files)
    -   [Завантаження файлів](#file-uploads)
    -   [Видимість файлу](#file-visibility)
-   [Видалення файлів](#deleting-files)
-   [Довідники](#directories)
-   [Спеціальні файлові системи](#custom-filesystems)

<a name="introduction"></a>

## Вступ

Laravel забезпечує потужну абстракцію файлової системи завдяки чудовому[Flysystem](https://github.com/thephpleague/flysystem)Пакет PHP Франка де Йонге. Інтеграція Laravel Flysystem забезпечує прості у використанні драйвери для роботи з локальними файловими системами та Amazon S3. Навіть краще, надзвичайно просто перемикатися між цими параметрами зберігання, оскільки API залишається незмінним для кожної системи.

<a name="configuration"></a>

## Конфігурація

Файл конфігурації файлової системи знаходиться за адресою`config/filesystems.php`. У цьому файлі ви можете налаштувати всі свої "диски". Кожен диск представляє певний драйвер зберігання та місце зберігання. Приклади конфігурацій для кожного підтримуваного драйвера містяться у файлі конфігурації. Отже, змініть конфігурацію, щоб відображати ваші уподобання та облікові дані.

Ви можете налаштувати скільки завгодно дисків і навіть мати кілька дисків, що використовують один і той же драйвер.

<a name="the-public-disk"></a>

### Public диск

`public`диск призначений для файлів, які будуть загальнодоступними. За замовчуванням`public`диск використовує`local`драйвер і зберігає ці файли в`storage/app/public`. Щоб зробити їх доступними з Інтернету, вам слід створити символічне посилання з`public/storage`до`storage/app/public`. Ця угода зберігатиме ваші загальнодоступні файли в одному каталозі, до яких можна легко ділитися між розгортаннями при використанні нульових систем розгортання, таких як[Надіслати](https://envoyer.io).

Для створення символічного посилання ви можете використовувати`storage:link`Команда ремісників:

    php artisan storage:link

Після збереження файлу та створення символічного посилання ви можете створити URL-адресу файлів за допомогою`asset`помічник:

    echo asset('storage/file.txt');

Ви можете налаштувати додаткові символічні посилання у своєму`filesystems`файл конфігурації. Кожне з налаштованих посилань буде створено під час запуску`storage:link`команда:

    'links' => [
        public_path('storage') => storage_path('app/public'),
        public_path('images') => storage_path('app/images'),
    ],

<a name="the-local-driver"></a>

### Local Driver

При використанні`local`драйвера, всі файлові операції відносяться до`root`каталог, визначений у вашому`filesystems`файл конфігурації. За замовчуванням це значення має значення`storage/app`каталог. Отже, наступний метод зберігає файл у`storage/app/file.txt`:

    Storage::disk('local')->put('file.txt', 'Contents');

<a name="permissions"></a>

#### Дозволи

`public`[видимість](#file-visibility)перекладає на`0755`для каталогів і`0644`для файлів. Ви можете змінити зіставлення дозволів у вашому`filesystems`файл конфігурації:

    'local' => [
        'driver' => 'local',
        'root' => storage_path('app'),
        'permissions' => [
            'file' => [
                'public' => 0664,
                'private' => 0600,
            ],
            'dir' => [
                'public' => 0775,
                'private' => 0700,
            ],
        ],
    ],

<a name="driver-prerequisites"></a>

### Передумови драйвера

<a name="composer-packages"></a>

#### Пакети композитора

Перш ніж використовувати драйвери SFTP або S3, вам потрібно встановити відповідний пакет через Composer:

-   SFTP:`league/flysystem-sftp ~1.0`
-   Amazon S3:`league/flysystem-aws-s3-v3 ~1.0`

Абсолютно необхідним для продуктивності є використання кешованого адаптера. Для цього вам знадобиться додатковий пакет:

-   Кешований адаптер:`league/flysystem-cached-adapter ~1.0`

<a name="s3-driver-configuration"></a>

#### Конфігурація драйвера S3

Інформація про конфігурацію драйвера S3 знаходиться у вашому`config/filesystems.php`файл конфігурації. Цей файл містить приклад масиву конфігурації для драйвера S3. Ви можете змінювати цей масив за допомогою власної конфігурації S3 та облікових даних. Для зручності ці змінні середовища відповідають умовам іменування, що використовуються AWS CLI.

<a name="ftp-driver-configuration"></a>

#### Конфігурація драйвера FTP

Інтеграція Laravel Flysystem чудово працює з FTP; однак, зразок конфігурації не входить у стандартну структуру фреймворку`filesystems.php`файл конфігурації. Якщо вам потрібно налаштувати файлову систему FTP, ви можете скористатися прикладом конфігурації нижче:

    'ftp' => [
        'driver' => 'ftp',
        'host' => 'ftp.example.com',
        'username' => 'your-username',
        'password' => 'your-password',

        // Optional FTP Settings...
        // 'port' => 21,
        // 'root' => '',
        // 'passive' => true,
        // 'ssl' => true,
        // 'timeout' => 30,
    ],

<a name="sftp-driver-configuration"></a>

#### Конфігурація драйвера SFTP

Інтеграція Laravel Flysystem чудово працює з SFTP; однак, зразок конфігурації не входить у стандартну структуру фреймворку`filesystems.php`файл конфігурації. Якщо вам потрібно налаштувати файлову систему SFTP, ви можете скористатися прикладом конфігурації нижче:

    'sftp' => [
        'driver' => 'sftp',
        'host' => 'example.com',
        'username' => 'your-username',
        'password' => 'your-password',

        // Settings for SSH key based authentication...
        // 'privateKey' => '/path/to/privateKey',
        // 'password' => 'encryption-password',

        // Optional SFTP Settings...
        // 'port' => 22,
        // 'root' => '',
        // 'timeout' => 30,
    ],

<a name="caching"></a>

### Кешування

Щоб увімкнути кешування для даного диска, ви можете додати файл`cache`директиву до параметрів конфігурації диска.`cache`Параметр повинен бути масивом параметрів кешування, що містить файл`disk`ім'я,`expire`час у секундах та кеш`prefix`:

    's3' => [
        'driver' => 's3',

        // Other Disk Options...

        'cache' => [
            'store' => 'memcached',
            'expire' => 600,
            'prefix' => 'cache-prefix',
        ],
    ],

<a name="obtaining-disk-instances"></a>

## Отримання екземплярів диска

`Storage`Facade може використовуватися для взаємодії з будь-яким із налаштованих вами дисків. Наприклад, ви можете використовувати`put`метод на фасаді для зберігання аватари на диску за замовчуванням. Якщо ви викликаєте методи на`Storage`Facadeбез попереднього дзвінка в`disk`метод, виклик методу автоматично передається на диск за замовчуванням:

    use Illuminate\Support\Facades\Storage;

    Storage::put('avatars/1', $fileContents);

Якщо ваша програма взаємодіє з кількома дисками, ви можете використовувати`disk`метод на`Storage`Facadeдля роботи з файлами на певному диску:

    Storage::disk('s3')->put('avatars/1', $fileContents);

<a name="retrieving-files"></a>

## Отримання файлів

`get`метод може бути використаний для отримання вмісту файлу. Сирий вміст рядка файлу буде повернено методом. Пам'ятайте, що всі шляхи до файлів повинні бути вказані щодо "кореневого" розташування, налаштованого для диска:

    $contents = Storage::get('file.jpg');

`exists`метод може бути використаний для визначення, чи існує файл на диску:

    $exists = Storage::disk('s3')->exists('file.jpg');

`missing`метод може бути використаний для визначення, чи на диску відсутній файл:

    $missing = Storage::disk('s3')->missing('file.jpg');

<a name="downloading-files"></a>

### Завантаження файлів

`download`метод може бути використаний для генерації відповіді, яка змушує браузер користувача завантажувати файл за вказаним шляхом.`download`метод приймає ім'я файлу як другий аргумент методу, який визначатиме ім'я файлу, яке бачить користувач, який завантажує файл. Нарешті, ви можете передати масив заголовків HTTP як третій аргумент методу:

    return Storage::download('file.jpg');

    return Storage::download('file.jpg', $name, $headers);

<a name="file-urls"></a>

### URL-адреси файлів

Ви можете використовувати`url`спосіб отримати URL-адресу для даного файлу. Якщо ви використовуєте`local`драйвера, це, як правило, просто передує`/storage`до заданого шляху та повернення до файлу відносної URL-адреси. Якщо ви використовуєте`s3`драйвера, буде повернено повністю кваліфіковану віддалену URL-адресу:

    use Illuminate\Support\Facades\Storage;

    $url = Storage::url('file.jpg');

При використанні`local`драйвер, усі файли, які мають бути загальнодоступними, повинні бути розміщені в`storage/app/public`каталог. Крім того, ви повинні[створити символічне посилання](#the-public-disk)в`public/storage`що вказує на`storage/app/public`каталог.

> {note} При використанні`local`драйвер, повернене значення`url`не кодується URL. З цієї причини ми рекомендуємо завжди зберігати файли, використовуючи імена, які створюватимуть дійсні URL-адреси.

<a name="temporary-urls"></a>

#### Тимчасові URL-адреси

Для файлів, що зберігаються за допомогою`s3`Ви можете створити тимчасову URL-адресу даного файлу за допомогою`temporaryUrl`метод. Цей метод приймає шлях і a`DateTime`екземпляр із зазначенням терміну дії URL-адреси:

    $url = Storage::temporaryUrl(
        'file.jpg', now()->addMinutes(5)
    );

Якщо вам потрібно вказати додаткові[Параметри запиту S3](https://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectGET.html#RESTObjectGET-requests), ви можете передати масив параметрів запиту як третій аргумент до`temporaryUrl`метод:

    $url = Storage::temporaryUrl(
        'file.jpg',
        now()->addMinutes(5),
        [
            'ResponseContentType' => 'application/octet-stream',
            'ResponseContentDisposition' => 'attachment; filename=file2.jpg',
        ]
    );

<a name="url-host-customization"></a>

#### Налаштування хосту URL-адреси

Якщо ви хочете заздалегідь визначити хост для URL-адрес файлів, створених за допомогою`Storage`фасад, ви можете додати`url`опція до масиву конфігурації диска:

    'public' => [
        'driver' => 'local',
        'root' => storage_path('app/public'),
        'url' => env('APP_URL').'/storage',
        'visibility' => 'public',
    ],

<a name="file-paths"></a>

### Шляхи до файлів

Ви можете використовувати`path`метод отримати шлях до заданого файлу. Якщо ви використовуєте`local`драйвер, це поверне абсолютний шлях до файлу. Якщо ви використовуєте`s3`драйвера, цей метод поверне відносний шлях до файлу в сегменті S3:

    use Illuminate\Support\Facades\Storage;

    $path = Storage::path('file.jpg');

<a name="file-metadata"></a>

### Файлові метадані

Окрім читання та запису файлів, Laravel може також надавати інформацію про самі файли. Наприклад,`size`метод може бути використаний для отримання розміру файлу в байтах:

    use Illuminate\Support\Facades\Storage;

    $size = Storage::size('file.jpg');

`lastModified`Метод повертає мітку часу UNIX останньої зміни файла:

    $time = Storage::lastModified('file.jpg');

<a name="storing-files"></a>

## Зберігання файлів

`put`метод може використовуватися для зберігання необробленого вмісту файлу на диску. Ви також можете пройти PHP`resource`до`put`метод, який використовуватиме базову підтримку потоку Flysystem. Пам'ятайте, що всі шляхи до файлів повинні бути вказані щодо "кореневого" розташування, налаштованого для диска:

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

<a name="automatic-streaming"></a>

#### Автоматичне потокове передавання

Якщо ви хочете, щоб Laravel автоматично керував потоковим передаванням даного файлу до вашого місця зберігання, ви можете використовувати`putFile`або`putFileAs`метод. Цей метод приймає або`Illuminate\Http\File`або`Illuminate\Http\UploadedFile`і автоматично переведе файл у потрібне місце:

    use Illuminate\Http\File;
    use Illuminate\Support\Facades\Storage;

    // Automatically generate a unique ID for file name...
    Storage::putFile('photos', new File('/path/to/photo'));

    // Manually specify a file name...
    Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');

Є кілька важливих речей, про які слід зазначити`putFile`метод. Зауважте, що ми вказали лише ім’я каталогу, а не ім’я файлу. За замовчуванням`putFile`метод генерує унікальний ідентифікатор, який служить іменем файлу. Розширення файлу буде визначено шляхом вивчення типу MIME файлу. Шлях до файлу поверне файл`putFile`метод, щоб ви могли зберігати шлях, включаючи згенероване ім’я файлу, у базі даних.

`putFile`і`putFileAs`методи також приймають аргумент, щоб вказати "видимість" збереженого файлу. Це особливо корисно, якщо ви зберігаєте файл на хмарному диску, такому як S3, і хочете, щоб файл був загальнодоступним:

    Storage::putFile('photos', new File('/path/to/photo'), 'public');

<a name="prepending-appending-to-files"></a>

#### Попереднє додавання та додавання до файлів

`prepend`і`append`методи дозволяють писати на початок або кінець файлу:

    Storage::prepend('file.log', 'Prepended Text');

    Storage::append('file.log', 'Appended Text');

<a name="copying-moving-files"></a>

#### Копіювання та переміщення файлів

`copy`метод може бути використаний для копіювання існуючого файлу в нове місце на диску, тоді як файл`move`метод може бути використаний для перейменування або переміщення існуючого файлу в нове місце:

    Storage::copy('old/file.jpg', 'new/file.jpg');

    Storage::move('old/file.jpg', 'new/file.jpg');

<a name="file-uploads"></a>

### Завантаження файлів

У веб-додатках одним із найпоширеніших випадків використання файлів є зберігання завантажених користувачами файлів, таких як зображення профілю, фотографії та документи. Laravel дозволяє дуже легко зберігати завантажені файли за допомогою`store`метод на завантаженому екземплярі файлу. Зателефонуйте`store`метод із шляхом, за яким ви хочете зберегти завантажений файл:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class UserAvatarController extends Controller
    {
        /**
         * Update the avatar for the user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            $path = $request->file('avatar')->store('avatars');

            return $path;
        }
    }

На цьому прикладі слід зазначити кілька важливих речей. Зауважте, що ми вказали лише ім’я каталогу, а не ім’я файлу. За замовчуванням`store`метод генерує унікальний ідентифікатор, який служить іменем файлу. Розширення файлу буде визначено шляхом вивчення типу MIME файлу. Шлях до файлу поверне файл`store`метод, щоб ви могли зберігати шлях, включаючи згенероване ім’я файлу, у базі даних.

Ви також можете зателефонувати до`putFile`метод на`Storage`фасаду, щоб виконати ту ж маніпуляцію файлами, що і в прикладі вище:

    $path = Storage::putFile('avatars', $request->file('avatar'));

<a name="specifying-a-file-name"></a>

#### Вказівка ​​імені файлу

Якщо ви не хочете, щоб ім'я файлу автоматично призначалося вашому збереженому файлу, ви можете використовувати файл`storeAs`метод, який отримує в якості аргументів шлях, ім’я файлу та (необов’язковий) диск:

    $path = $request->file('avatar')->storeAs(
        'avatars', $request->user()->id
    );

Ви також можете використовувати`putFileAs`метод на`Storage`фасад, який буде виконувати таку ж маніпуляцію файлами, як приклад вище:

    $path = Storage::putFileAs(
        'avatars', $request->file('avatar'), $request->user()->id
    );

> {note} Недруковані та недійсні символи Unicode автоматично видаляються із шляхів до файлів. Тому, можливо, ви захочете продезінфікувати шляхи до своїх файлів, перш ніж передавати їх методам зберігання файлів Laravel. Шляхи до файлів нормалізуються за допомогою`League\Flysystem\Util::normalizePath`метод.

<a name="specifying-a-disk"></a>

#### Вказівка ​​диска

За замовчуванням цей метод використовуватиме ваш диск за замовчуванням. Якщо ви хочете вказати інший диск, передайте ім'я диска як другий аргумент у файл`store`метод:

    $path = $request->file('avatar')->store(
        'avatars/'.$request->user()->id, 's3'
    );

Якщо ви використовуєте`storeAs`методу, ви можете передати ім'я диска як третій аргумент методу:

    $path = $request->file('avatar')->storeAs(
        'avatars',
        $request->user()->id,
        's3'
    );

<a name="other-file-information"></a>

#### Інша інформація про файл

Якщо ви хочете отримати оригінальну назву завантаженого файлу, ви можете зробити це за допомогою`getClientOriginalName`метод:

    $name = $request->file('avatar')->getClientOriginalName();

`extension`метод може бути використаний для отримання розширення файлу завантаженого файлу:

    $extension = $request->file('avatar')->extension();

<a name="file-visibility"></a>

### Видимість файлу

В інтеграції Laravel Flysystem "видимість" - це абстракція дозволів на файли на декількох платформах. Файли можуть бути оголошені`public`або`private`. Коли файл оголошено`public`, ви вказуєте, що файл, як правило, повинен бути доступним для інших. Наприклад, використовуючи драйвер S3, ви можете отримати URL-адреси для`public`файлів.

Ви можете встановити видимість при встановленні файлу через`put`метод:

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents, 'public');

Якщо файл вже зберігався, його видимість можна отримати та встановити за допомогою`getVisibility`і`setVisibility`методи:

    $visibility = Storage::getVisibility('file.jpg');

    Storage::setVisibility('file.jpg', 'public');

Під час взаємодії із завантаженими файлами ви можете використовувати файл`storePublicly`і`storePubliclyAs`методи зберігання завантаженого файлу за допомогою`public`видимість:

    $path = $request->file('avatar')->storePublicly('avatars', 's3');

    $path = $request->file('avatar')->storePubliclyAs(
        'avatars',
        $request->user()->id,
        's3'
    );

<a name="deleting-files"></a>

## Видалення файлів

`delete`метод приймає одне ім'я файлу або масив файлів для видалення з диска:

    use Illuminate\Support\Facades\Storage;

    Storage::delete('file.jpg');

    Storage::delete(['file.jpg', 'file2.jpg']);

За необхідності ви можете вказати диск, з якого слід видалити файл:

    use Illuminate\Support\Facades\Storage;

    Storage::disk('s3')->delete('folder_path/file_name.jpg');

<a name="directories"></a>

## Довідники

<a name="get-all-files-within-a-directory"></a>

#### Отримати всі файли в каталозі

`files`метод повертає масив усіх файлів у даному каталозі. Якщо ви хочете отримати список усіх файлів у даному каталозі, включаючи всі підкаталоги, ви можете використовувати`allFiles`метод:

    use Illuminate\Support\Facades\Storage;

    $files = Storage::files($directory);

    $files = Storage::allFiles($directory);

<a name="get-all-directories-within-a-directory"></a>

#### Отримати всі каталоги в каталозі

`directories`Метод повертає масив усіх каталогів у даному каталозі. Крім того, ви можете використовувати`allDirectories`спосіб отримати список усіх каталогів у даному каталозі та всіх його підкаталогів:

    $directories = Storage::directories($directory);

    // Recursive...
    $directories = Storage::allDirectories($directory);

<a name="create-a-directory"></a>

#### Створіть каталог

`makeDirectory`метод створить даний каталог, включаючи всі необхідні підкаталоги:

    Storage::makeDirectory($directory);

<a name="delete-a-directory"></a>

#### Видалити каталог

Нарешті,`deleteDirectory`метод може бути використаний для видалення каталогу та всіх його файлів:

    Storage::deleteDirectory($directory);

<a name="custom-filesystems"></a>

## Спеціальні файлові системи

Інтеграція Laravel Flysystem забезпечує готові драйвери для декількох "драйверів"; однак Flysystem цим не обмежується і має адаптери для багатьох інших систем зберігання. Ви можете створити власний драйвер, якщо хочете використовувати один із цих додаткових адаптерів у своїй програмі Laravel.

Для налаштування власної файлової системи вам знадобиться адаптер Flysystem. Давайте додамо адаптер Dropbox, який підтримується спільнотою, до нашого проекту:

    composer require spatie/flysystem-dropbox

Далі слід створити файл[постачальник послуг](/docs/{{version}}/providers)як от`DropboxServiceProvider`. У постачальника`boot`метод, ви можете використовувати`Storage`фасадні`extend`метод визначення користувацького драйвера:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Storage;
    use Illuminate\Support\ServiceProvider;
    use League\Flysystem\Filesystem;
    use Spatie\Dropbox\Client as DropboxClient;
    use Spatie\FlysystemDropbox\DropboxAdapter;

    class DropboxServiceProvider extends ServiceProvider
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
            Storage::extend('dropbox', function ($app, $config) {
                $client = new DropboxClient(
                    $config['authorization_token']
                );

                return new Filesystem(new DropboxAdapter($client));
            });
        }
    }

Перший аргумент`extend`метод - це ім'я драйвера, а другий - Закриття, яке отримує`$app`і`$config`змінні. Закриття розв'язувача повинно повернути екземпляр`League\Flysystem\Filesystem`.`$config`змінна містить значення, визначені в`config/filesystems.php`для вказаного диска.

Далі зареєструйте постачальника послуг у своєму`config/app.php`файл конфігурації:

    'providers' => [
        // ...
        App\Providers\DropboxServiceProvider::class,
    ];

Після створення та реєстрації постачальника послуг розширення ви можете використовувати`dropbox`драйвер у вашому`config/filesystems.php`файл конфігурації.
