# Пошта

-   [Вступ](#introduction)
    -   [Конфігурація](#configuration)
    -   [Передумови драйвера](#driver-prerequisites)
-   [Генерування Mailables](#generating-mailables)
-   [Написання Mailables](#writing-mailables)
    -   [Налаштування відправника](#configuring-the-sender)
    -   [Налаштування View](#configuring-the-view)
    -   [Переглянути дані](#view-data)
    -   [Вкладення](#attachments)
    -   [Вбудовані вкладення](#inline-attachments)
    -   [Налаштування повідомлення SwiftMailer](#customizing-the-swiftmailer-message)
-   [Націнка доступна](#markdown-mailables)
    -   [Генерування доступності націнки](#generating-markdown-mailables)
    -   [Написання повідомлень про націнку](#writing-markdown-messages)
    -   [Налаштування компонентів](#customizing-the-components)
-   [Відправлення пошти](#sending-mail)
    -   [Пошта в черзі](#queueing-mail)
-   [Візуалізація доступних](#rendering-mailables)
    -   [Попередній перегляд Mailables у браузері](#previewing-mailables-in-the-browser)
-   [Локалізація доступних](#localizing-mailables)
-   [Пошта та місцевий розвиток](#mail-and-local-development)
-   [Події](#events)

<a name="introduction"></a>

## Вступ

Laravel забезпечує чистий, простий API порівняно з популярним[SwiftMailer](https://swiftmailer.symfony.com/)бібліотека з драйверами для SMTP, Mailgun, Postmark, Amazon SES та`sendmail`, що дозволяє швидко розпочати надсилання пошти через локальний або хмарний сервіс на ваш вибір.

<a name="configuration"></a>

### Конфігурація

Послуги електронної пошти Laravel можна налаштувати через`mail`файл конфігурації. Кожен поштовий скринька, налаштований у цьому файлі, може мати власні параметри і навіть власний унікальний "транспорт", що дозволяє вашій програмі використовувати різні служби електронної пошти для надсилання певних повідомлень електронної пошти. Наприклад, ваша програма може використовувати поштовий штемпель для надсилання транзакційних повідомлень, а за допомогою Amazon SES - для масових повідомлень.

<a name="driver-prerequisites"></a>

### Передумови драйвера

Драйвери на основі API, такі як Mailgun та Postmark, часто простіші та швидші, ніж SMTP-сервери. Якщо можливо, слід скористатися одним із цих драйверів. Для всіх драйверів API потрібна бібліотека Guzzle HTTP, яку можна встановити через менеджер пакетів Composer:

    composer require guzzlehttp/guzzle

<a name="mailgun-driver"></a>

#### Водій поштової зброї

Щоб використовувати драйвер Mailgun, спочатку встановіть Guzzle, а потім встановіть`default`варіант у вашому`config/mail.php`файл конфігурації до`mailgun`. Далі переконайтеся, що ваш`config/services.php`файл конфігурації містить такі опції:

    'mailgun' => [
        'domain' => env('MAILGUN_DOMAIN'),
        'secret' => env('MAILGUN_SECRET'),
    ],

Якщо ви не використовуєте "США"[Регіон поштової зброї](https://documentation.mailgun.com/en/latest/api-intro.html#mailgun-regions), ви можете визначити кінцеву точку вашого регіону в`services`файл конфігурації:

    'mailgun' => [
        'domain' => env('MAILGUN_DOMAIN'),
        'secret' => env('MAILGUN_SECRET'),
        'endpoint' => env('MAILGUN_ENDPOINT', 'api.eu.mailgun.net'),
    ],

<a name="postmark-driver"></a>

#### Драйвер поштового штемпеля

Щоб використовувати драйвер поштового штемпеля, встановіть транспорт SwiftMailer Postmark через Composer:

    composer require wildbit/swiftmailer-postmark

Далі встановіть Guzzle і встановіть`default`варіант у вашому`config/mail.php`файл конфігурації до`postmark`. Нарешті, переконайтеся, що ваш`config/services.php`файл конфігурації містить такі опції:

    'postmark' => [
        'token' => env('POSTMARK_TOKEN'),
    ],

<a name="ses-driver"></a>

#### Драйвер SES

Щоб використовувати драйвер Amazon SES, спочатку потрібно встановити Amazon AWS SDK для PHP. Ви можете встановити цю бібліотеку, додавши наступний рядок до вашого`composer.json`файлів`require`розділу та запуску`composer update`команда:

    "aws/aws-sdk-php": "~3.0"

Далі встановіть`default`варіант у вашому`config/mail.php`файл конфігурації до`ses`і переконайтеся, що ваш`config/services.php`файл конфігурації містить такі опції:

    'ses' => [
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    ],

Якщо вам потрібно включити[додаткові опції](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-email-2010-12-01.html#sendrawemail)при виконанні СЕС`SendRawEmail`Ви можете визначити`options`масив у вашому`ses`конфігурація:

    'ses' => [
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
        'options' => [
            'ConfigurationSetName' => 'MyConfigurationSet',
            'Tags' => [
                [
                    'Name' => 'foo',
                    'Value' => 'bar',
                ],
            ],
        ],
    ],

<a name="generating-mailables"></a>

## Генерування Mailables

У Laravel кожен тип електронного листа, надісланий вашою програмою, представлений як клас "доступний". Ці класи зберігаються в`app/Mail`каталог. Не хвилюйтеся, якщо ви не бачите цей каталог у своїй програмі, оскільки він буде створений для вас, коли ви створите свій перший доступний клас за допомогою`make:mail`команда:

    php artisan make:mail OrderShipped

<a name="writing-mailables"></a>

## Написання Mailables

Вся конфігурація доступного класу виконується в`build`метод. У рамках цього методу ви можете викликати різні методи, такі як`from`,`subject`,`view`, і`attach`налаштувати презентацію та доставку електронного листа.

<a name="configuring-the-sender"></a>

### Налаштування відправника

<a name="using-the-from-method"></a>

#### Використання`from`Метод

Спочатку розглянемо налаштування відправника електронної пошти. Або, іншими словами, хто буде електронною поштою. Існує два способи налаштування відправника. По-перше, ви можете використовувати`from`метод у вашому доступному класі '`build`метод:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->view('emails.orders.shipped');
    }

<a name="using-a-global-from-address"></a>

#### Використання глобального`from`Адреса

Однак, якщо ваша програма використовує одну і ту ж адресу "з" для всіх своїх електронних листів, виклик телефону на`from`у кожному доступному класі, який ви створюєте. Замість цього ви можете вказати глобальну адресу "з" у вашому`config/mail.php`файл конфігурації. Ця адреса буде використана, якщо в межах доступного класу не вказана інша адреса "з":

    'from' => ['address' => 'example@example.com', 'name' => 'App Name'],

Крім того, ви можете визначити загальну адресу "reply_to" у вашому`config/mail.php`файл конфігурації:

    'reply_to' => ['address' => 'example@example.com', 'name' => 'App Name'],

<a name="configuring-the-view"></a>

### Налаштування View

В межах доступного класу '`build`метод, ви можете використовувати`view`метод, щоб вказати, який шаблон слід використовувати під час відтворення вмісту електронного листа. Оскільки в кожному електронному листі зазвичай використовується файл[Blade шаблон](/docs/{{version}}/blade)щоб відтворити його вміст, ви маєте повну потужність та зручність механізму шаблонізації Blade під час створення HTML-коду електронної пошти:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped');
    }

> {tip} Ви можете створити файл`resources/views/emails`каталог для розміщення всіх ваших шаблонів електронної пошти; однак ви можете вільно розміщувати їх там, де забажаєте`resources/views`каталог.

<a name="plain-text-emails"></a>

#### Звичайні текстові електронні листи

Якщо ви хочете визначити текстову версію електронної пошти, ви можете використовувати`text`метод. Подобається`view`метод,`text`метод приймає ім'я шаблону, яке буде використано для візуалізації вмісту електронного листа. Ви можете визначити як HTML, так і текстову версію свого повідомлення:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->text('emails.orders.shipped_plain');
    }

<a name="view-data"></a>

### Переглянути дані

<a name="via-public-properties"></a>

#### Через публічну власність

Як правило, вам доведеться передати у свій View деякі дані, які ви можете використовувати при отрисуванні HTML-коду електронного листа. Є два способи зробити дані доступними для вашого перегляду. По-перше, будь-яка загальнодоступна властивість, визначена у вашому доступному класі, буде автоматично доступною для перегляду. Так, наприклад, ви можете передати дані в конструктор вашого доступного класу і встановити ці дані на загальнодоступні властивості, визначені в класі:

    <?php

    namespace App\Mail;

    use App\Models\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * The order instance.
         *
         * @var Order
         */
        public $order;

        /**
         * Create a new message instance.
         *
         * @param  \App\Models\Order  $order
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped');
        }
    }

Як тільки дані будуть встановлені як загальнодоступне, вони будуть автоматично доступні у вашому поданні, тому ви зможете отримати до них доступ, як і до будь-яких інших даних у ваших шаблонах Blade:

    <div>
        Price: {{ $order->price }}
    </div>

<a name="via-the-with-method"></a>

#### Через`with`Метод:

Якщо ви хочете налаштувати формат даних електронної пошти перед тим, як їх надсилати до шаблону, ви можете вручну передати свої дані у View за допомогою`with`метод. Зазвичай ви все одно передаєте дані через конструктор доступного класу '; однак вам слід встановити для цих даних значення`protected`або`private`властивості, тому дані автоматично не стають доступними для шаблону. Потім, при виклику`with`метод, передайте масив даних, які ви хочете зробити доступними для шаблону:

    <?php

    namespace App\Mail;

    use App\Models\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * The order instance.
         *
         * @var \App\Models\Order
         */
        protected $order;

        /**
         * Create a new message instance.
         *
         * @param  \App\Models\Order $order
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->with([
                            'orderName' => $this->order->name,
                            'orderPrice' => $this->order->price,
                        ]);
        }
    }

Після передачі даних до`with`метод, він буде автоматично доступний у вашому поданні, тому ви можете отримати доступ до нього, як і до будь-яких інших даних у своїх шаблонах Blade:

    <div>
        Price: {{ $orderPrice }}
    </div>

<a name="attachments"></a>

### Вкладення

Щоб додати вкладення до електронного листа, використовуйте`attach`метод у доступному класі '`build`метод.`attach`метод приймає повний шлях до файлу як перший аргумент:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->attach('/path/to/file');
    }

Прикріплюючи файли до повідомлення, ви також можете вказати відображуване ім'я та / або тип MIME, передавши файл`array`як другий аргумент`attach`метод:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->attach('/path/to/file', [
                        'as' => 'name.pdf',
                        'mime' => 'application/pdf',
                    ]);
    }

<a name="attaching-files-from-disk"></a>

#### Вкладання файлів з диска

Якщо ви зберегли файл на одному зі своїх[диски файлової системи](/docs/{{version}}/filesystem), ви можете прикріпити його до електронного листа за допомогою`attachFromStorage`метод:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
       return $this->view('emails.orders.shipped')
                   ->attachFromStorage('/path/to/file');
    }

Якщо потрібно, ви можете вказати ім'я вкладення файлу та додаткові параметри, використовуючи другий та третій аргументи до`attachFromStorage`метод:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
       return $this->view('emails.orders.shipped')
                   ->attachFromStorage('/path/to/file', 'name.pdf', [
                       'mime' => 'application/pdf'
                   ]);
    }

`attachFromStorageDisk`метод може бути використаний, якщо вам потрібно вказати диск зберігання, відмінний від диска за замовчуванням:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
       return $this->view('emails.orders.shipped')
                   ->attachFromStorageDisk('s3', '/path/to/file');
    }

<a name="raw-data-attachments"></a>

#### Вкладені файли необроблених даних

`attachData`метод може бути використаний для приєднання необробленого рядка байтів як вкладення. Наприклад, ви можете скористатися цим методом, якщо створили PDF в пам'яті і хочете приєднати його до електронного листа, не записуючи на диск.`attachData`метод приймає необроблені байти даних як перший аргумент, ім'я файлу як другий аргумент і масив параметрів як третій аргумент:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->attachData($this->pdf, 'name.pdf', [
                        'mime' => 'application/pdf',
                    ]);
    }

<a name="inline-attachments"></a>

### Вбудовані вкладення

Вбудовувати вбудовані зображення в електронні листи, як правило, громіздко; однак Laravel пропонує зручний спосіб приєднати зображення до електронних листів та отримати відповідний CID. Щоб вставити вбудоване зображення, використовуйте`embed`метод на`$message`змінної в шаблоні електронної пошти. Laravel автоматично робить`$message`доступна для всіх ваших шаблонів електронної пошти, тому вам не потрібно турбуватися про передачу її вручну:

    <body>
        Here is an image:

        <img src="{{ $message->embed($pathToImage) }}">
    </body>

> {Примітка}`$message`Змінна недоступна у текстових повідомленнях, оскільки текстові повідомлення не використовують вбудовані вкладення.

<a name="embedding-raw-data-attachments"></a>

#### Вбудовування вкладень необроблених даних

Якщо у вас вже є необроблений рядок даних, який ви хочете вбудувати в шаблон електронної пошти, ви можете використовувати`embedData`метод на`$message`змінна:

    <body>
        Here is an image from raw data:

        <img src="{{ $message->embedData($data, $name) }}">
    </body>

<a name="customizing-the-swiftmailer-message"></a>

### Налаштування повідомлення SwiftMailer

`withSwiftMessage`метод`Mailable`базовий клас дозволяє зареєструвати зворотний виклик, який буде викликаний із необробленим екземпляром повідомлення SwiftMailer перед відправкою повідомлення. Це дає вам можливість налаштувати повідомлення перед його доставкою:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        $this->view('emails.orders.shipped');

        $this->withSwiftMessage(function ($message) {
            $message->getHeaders()
                    ->addTextHeader('Custom-Header', 'HeaderValue');
        });
    }

<a name="markdown-mailables"></a>

## Націнка доступна

Повідомлення, доступні для розмітки, дозволяють вам скористатися заздалегідь побудованими шаблонами та компонентами сповіщень про пошту у ваших доступних повідомленнях. Оскільки повідомлення написані в Markdown, Laravel може рендерити чудові, гнучкі HTML-шаблони для повідомлень, а також автоматично генерувати аналог у звичайному тексті.

<a name="generating-markdown-mailables"></a>

### Генерування доступності націнки

Щоб створити доступний за допомогою відповідного шаблону Markdown, ви можете використовувати`--markdown`варіант`make:mail`Команда ремісників:

    php artisan make:mail OrderShipped --markdown=emails.orders.shipped

Потім, при налаштуванні доступного в межах`build`метод, викличте`markdown`метод замість`view`метод.`markdown`метод приймає ім'я шаблону Markdown та необов'язковий масив даних, щоб зробити шаблон доступним:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->markdown('emails.orders.shipped');
    }

<a name="writing-markdown-messages"></a>

### Написання повідомлень про націнку

Доступні в Markdown комбінації компонентів Blade та синтаксису Markdown дозволяють легко створювати поштові повідомлення, використовуючи заздалегідь створені компоненти Laravel:

    @component('mail::message')
    # Order Shipped

    Your order has been shipped!

    @component('mail::button', ['url' => $url])
    View Order
    @endcomponent

    Thanks,<br>
    {{ config('app.name') }}
    @endcomponent

> {tip} Не використовуйте надмірні відступи під час написання електронних листів Markdown. Аналізатори розмітки відображатимуть відступ із вмістом як блоки коду.

<a name="button-component"></a>

#### Кнопка Компонент

Компонент кнопки відображає відцентрове посилання кнопки. Компонент приймає два аргументи, a`url`та необов’язково`color`. Підтримувані кольори`primary`,`success`, і`error`. Ви можете додати до повідомлення скільки завгодно компонентів кнопок:

    @component('mail::button', ['url' => $url, 'color' => 'success'])
    View Order
    @endcomponent

<a name="panel-component"></a>

#### Компонент панелі

Компонент панелі відображає даний блок тексту на панелі, яка має дещо інший колір тла, ніж решта повідомлення. Це дозволяє звернути увагу на певний блок тексту:

    @component('mail::panel')
    This is the panel content.
    @endcomponent

<a name="table-component"></a>

#### Компонент таблиці

Компонент таблиці дозволяє перетворити таблицю Markdown в таблицю HTML. Компонент приймає таблицю Markdown як свій вміст. Вирівнювання стовпців таблиці підтримується за допомогою синтаксису вирівнювання таблиці Markdown за замовчуванням:

    @component('mail::table')
    | Laravel       | Table         | Example  |
    | ------------- |:-------------:| --------:|
    | Col 2 is      | Centered      | $10      |
    | Col 3 is      | Right-Aligned | $20      |
    @endcomponent

<a name="customizing-the-components"></a>

### Налаштування компонентів

Ви можете експортувати всі компоненти пошти Markdown у власну програму для налаштування. Щоб експортувати компоненти, використовуйте`vendor:publish`Реміснича команда видавати`laravel-mail`тег активу:

    php artisan vendor:publish --tag=laravel-mail

Ця команда опублікує компоненти пошти Markdown у`resources/views/vendor/mail`каталог.`mail`каталог міститиме`html`і a`text`каталог, кожен із відповідних Views кожного доступного компонента. Ви можете налаштувати ці компоненти як завгодно.

<a name="customizing-the-css"></a>

#### Налаштування CSS

Після експорту компонентів файл`resources/views/vendor/mail/html/themes`каталог міститиме файл`default.css`файл. Ви можете налаштувати CSS у цьому файлі, і ваші стилі будуть автоматично вставлені в HTML-зображення ваших повідомлень електронної пошти Markdown.

Якщо ви хочете створити абсолютно нову тему для компонентів Laravel Markdown, ви можете розмістити файл CSS у`html/themes`каталог. Після іменування та збереження файлу CSS оновіть файл`theme`варіант`mail`файл конфігурації відповідно до назви вашої нової теми.

Щоб налаштувати тему для окремого доступного користувача, ви можете встановити`$theme`властивість класу "доступний" на ім'я теми, яку слід використовувати під час надсилання цієї книги.

<a name="sending-mail"></a>

## Відправлення пошти

Щоб надіслати повідомлення, використовуйте`to`метод на`Mail`[фасад](/docs/{{version}}/facades).`to`метод приймає електронну адресу, екземпляр користувача або колекцію користувачів. Якщо ви передасте об'єкт або колекцію об'єктів, поштовий скринька автоматично використовуватиме їх`email`і`name`властивості при встановленні одержувачів електронної пошти, тому переконайтесь, що ці атрибути доступні для ваших об’єктів. Після того, як ви вказали своїх одержувачів, ви можете передати екземпляр вашого доступного класу до`send`метод:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Mail\OrderShipped;
    use App\Models\Order;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Mail;

    class OrderController extends Controller
    {
        /**
         * Ship the given order.
         *
         * @param  Request  $request
         * @param  int  $orderId
         * @return Response
         */
        public function ship(Request $request, $orderId)
        {
            $order = Order::findOrFail($orderId);

            // Ship order...

            Mail::to($request->user())->send(new OrderShipped($order));
        }
    }

Ви не обмежуєтеся просто вказівкою одержувачів "до" під час надсилання повідомлення. Ви можете встановити одержувачів "на", "куб. Сс" та "приховану копію" у межах одного виклику методом ланцюжка:

    use Illuminate\Support\Facades\Mail;

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->send(new OrderShipped($order));

<a name="looping-over-recipients"></a>

#### Зациклювання над одержувачами

Іноді вам може знадобитися надіслати доступ до списку одержувачів, перебираючи масив одержувачів / адрес електронної пошти. Так як`to`Метод додає адреси електронної пошти до списку одержувачів, доступних для доступу, вам слід завжди створювати доступний екземпляр для кожного одержувача:

    foreach (['taylor@example.com', 'dries@example.com'] as $recipient) {
        Mail::to($recipient)->send(new OrderShipped($order));
    }

<a name="sending-mail-via-a-specific-mailer"></a>

#### Відправлення пошти за допомогою певної пошти

За замовчуванням Laravel використовуватиме поштову скриньку, налаштовану як`default`пошти у вашому`mail`файл конфігурації. Однак ви можете використовувати`mailer`спосіб відправити повідомлення за допомогою певної конфігурації поштової скриньки:

    Mail::mailer('postmark')
            ->to($request->user())
            ->send(new OrderShipped($order));

<a name="queueing-mail"></a>

### Пошта в черзі

<a name="queueing-a-mail-message"></a>

#### Черга поштового повідомлення

Оскільки надсилання повідомлень електронної пошти може суттєво подовжити час відгуку вашої програми, багато розробників вирішують поставити повідомлення електронної пошти в чергу для фонової відправки. Laravel полегшує це використання за допомогою вбудованого[API уніфікованої черги](/docs/{{version}}/queues). Щоб поставити поштове повідомлення в чергу, використовуйте`queue`метод на`Mail`фасаду після вказівки одержувачів повідомлення:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue(new OrderShipped($order));

Цей метод автоматично подбає про перенесення завдання в чергу, тому повідомлення надсилається у фоновому режимі. Вам потрібно буде[налаштувати черги](/docs/{{version}}/queues)перед використанням цієї функції.

<a name="delayed-message-queueing"></a>

#### Затримка черг повідомлень

Якщо ви хочете затримати доставку повідомлення електронної пошти, яке знаходиться в черзі, ви можете використовувати`later`метод. Першим аргументом є`later`метод приймає a`DateTime`екземпляр, який вказує, коли повідомлення має бути надіслано:

    $when = now()->addMinutes(10);

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->later($when, new OrderShipped($order));

<a name="pushing-to-specific-queues"></a>

#### Перехід до певних черг

Оскільки всі доступні класи, створені за допомогою`make:mail`команда використовує`Illuminate\Bus\Queueable`риса, ви можете зателефонувати до`onQueue`і`onConnection`методів на будь-якому доступному екземплярі класу, що дозволяє вказати підключення та назву черги для повідомлення:

    $message = (new OrderShipped($order))
                    ->onConnection('sqs')
                    ->onQueue('emails');

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue($message);

<a name="queueing-by-default"></a>

#### Черга за замовчуванням

Якщо у вас є доступні класи, які ви хочете завжди залишати в черзі, ви можете застосувати`ShouldQueue`контракт на клас. Тепер, навіть якщо ви телефонуєте на`send`метод під час розсилки, доступний все ще буде в черзі, оскільки він реалізує контракт:

    use Illuminate\Contracts\Queue\ShouldQueue;

    class OrderShipped extends Mailable implements ShouldQueue
    {
        //
    }

<a name="rendering-mailables"></a>

## Візуалізація доступних

Іноді вам може знадобитися захопити HTML-вміст доступного користувача, не надсилаючи його. Для цього ви можете зателефонувати до`render`спосіб доступності. Цей метод поверне обчислюваний вміст доступного у вигляді рядка:

    $invoice = App\Models\Invoice::find(1);

    return (new App\Mail\InvoicePaid($invoice))->render();

<a name="previewing-mailables-in-the-browser"></a>

### Попередній перегляд Mailables у браузері

Під час проектування шаблону, що доступний, зручно швидко переглядати візуалізований файл, доступний у вашому браузері, як типовий шаблон Blade. З цієї причини Laravel дозволяє повернути будь-який доступний безпосередньо з Закриття маршруту або контролера. Після повернення файлу, який доступний, він буде відтворений і відображений у браузері, що дозволить вам швидко переглянути його дизайн без необхідності відправляти його на фактичну електронну адресу:

    Route::get('mailable', function () {
        $invoice = App\Models\Invoice::find(1);

        return new App\Mail\InvoicePaid($invoice);
    });

> {Примітка}[Вбудовані вкладення](#inline-attachments)не відображатиметься, коли у вашому браузері здійснюється попередній перегляд Mailables. Щоб переглянути ці доступні файли, вам слід надіслати їх до програми тестування електронної пошти, наприклад[MailHog](https://github.com/mailhog/MailHog)або[Привіт](https://usehelo.com).

<a name="localizing-mailables"></a>

## Локалізація доступних

Laravel дозволяє надсилати доступні матеріали мовою, відмінною від поточної мови, і навіть запам’ятає цю локаль, якщо пошта буде в черзі.

Для цього,`Mail`Facadeпропонує a`locale`спосіб встановити потрібну мову. Програма зміниться на цю локаль після форматування доступної версії, а потім повернеться до попередньої локалі після завершення форматування:

    Mail::to($request->user())->locale('es')->send(
        new OrderShipped($order)
    );

<a name="user-preferred-locales"></a>

### Локалі, що надаються користувачем

Іноді програми зберігають улюблений регіон кожного користувача. Впроваджуючи`HasLocalePreference`уклавши контракт на одну або кілька ваших моделей, ви можете доручити Laravel використовувати цю збережену локаль під час надсилання пошти:

    use Illuminate\Contracts\Translation\HasLocalePreference;

    class User extends Model implements HasLocalePreference
    {
        /**
         * Get the user's preferred locale.
         *
         * @return string
         */
        public function preferredLocale()
        {
            return $this->locale;
        }
    }

Після того, як ви впровадите інтерфейс, Laravel автоматично використовуватиме бажану локаль під час надсилання доступних моделей та повідомлень до моделі. Тому немає необхідності телефонувати до`locale`метод при використанні цього інтерфейсу:

    Mail::to($request->user())->send(new OrderShipped($order));

<a name="mail-and-local-development"></a>

## Пошта та місцевий розвиток

Розробляючи програму, яка надсилає електронну пошту, ви, мабуть, не хочете фактично надсилати електронні листи на електронні адреси, що працюють. Laravel пропонує декілька способів "відключити" фактичне надсилання електронних листів під час локальної розробки.

<a name="log-driver"></a>

#### Драйвер журналу

Замість того, щоб надсилати електронні листи,`log`поштовий драйвер запише всі повідомлення електронної пошти у ваші журнали для перевірки. Щоб отримати докладнішу інформацію про налаштування програми для кожного середовища, перегляньте[конфігураційна документація](/docs/{{version}}/configuration#environment-configuration).

<a name="universal-to"></a>

#### Універсальний

Іншим рішенням, яке надає Laravel, є встановлення універсального одержувача всіх електронних листів, надісланих фреймворком. Таким чином, усі електронні листи, створені вашим додатком, будуть надіслані на певну адресу, а не на адресу, фактично вказану під час надсилання повідомлення. Це можна зробити за допомогою`to`варіант у вашому`config/mail.php`файл конфігурації:

    'to' => [
        'address' => 'example@example.com',
        'name' => 'Example'
    ],

<a name="mailtrap"></a>

#### Поштова пастка

Нарешті, ви можете скористатися такою послугою, як[Поштова пастка](https://mailtrap.io)та`smtp`драйвер для надсилання ваших повідомлень електронної пошти на "фіктивну" поштову скриньку, де ви можете переглянути їх у справжньому поштовому клієнті. Цей підхід має перевагу, дозволяючи вам фактично перевіряти остаточні електронні листи в засобі перегляду повідомлень Mailtrap.

<a name="events"></a>

## Події

Laravel запускає дві події в процесі надсилання поштових повідомлень.`MessageSending`подія запускається до надсилання повідомлення, тоді як`MessageSent`подія запускається після відправлення повідомлення. Пам’ятайте, ці події запускаються, коли відбувається пошта_надісланий_, а не тоді, коли воно в черзі. Ви можете зареєструвати слухач події для цієї події у своєму`EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Mail\Events\MessageSending' => [
            'App\Listeners\LogSendingMessage',
        ],
        'Illuminate\Mail\Events\MessageSent' => [
            'App\Listeners\LogSentMessage',
        ],
    ];
