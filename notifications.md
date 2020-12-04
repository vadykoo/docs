# Повідомлення

-   [Вступ](#introduction)
-   [Створення сповіщень](#creating-notifications)
-   [Надсилання сповіщень](#sending-notifications)
    -   [Using The Notifiable Trait](#using-the-notifiable-trait)
    -   [Використання фасаду повідомлень](#using-the-notification-facade)
    -   [Визначення каналів доставки](#specifying-delivery-channels)
    -   [Черга сповіщень](#queueing-notifications)
    -   [Повідомлення на вимогу](#on-demand-notifications)
-   [Повідомлення поштою](#mail-notifications)
    -   [Форматування поштових повідомлень](#formatting-mail-messages)
    -   [Налаштування відправника](#customizing-the-sender)
    -   [Налаштування одержувача](#customizing-the-recipient)
    -   [Налаштування теми](#customizing-the-subject)
    -   [Налаштування пошти](#customizing-the-mailer)
    -   [Налаштування шаблонів](#customizing-the-templates)
    -   [Вкладення](#mail-attachments)
    -   [Попередній перегляд створених сповіщень](#previewing-mail-notifications)
-   [Повідомлення поштової розсилки](#markdown-mail-notifications)
    -   [Створення повідомлення](#generating-the-message)
    -   [Написання повідомлення](#writing-the-message)
    -   [Налаштування компонентів](#customizing-the-components)
-   [Notification бази даних](#database-notifications)
    -   [Передумови](#database-prerequisites)
    -   [Форматування сповіщень баз даних](#formatting-database-notifications)
    -   [Доступ до сповіщень](#accessing-the-notifications)
    -   [Позначення повідомлень як прочитаних](#marking-notifications-as-read)
-   [Broadcast Notifications](#broadcast-notifications)
    -   [Передумови](#broadcast-prerequisites)
    -   [Форматування сповіщень про Broadcast](#formatting-broadcast-notifications)
    -   [Прослуховування сповіщень](#listening-for-notifications)
-   [SMS-Notification](#sms-notifications)
    -   [Передумови](#sms-prerequisites)
    -   [Форматування сповіщень SMS](#formatting-sms-notifications)
    -   [Форматування сповіщень із шорткодом](#formatting-shortcode-notifications)
    -   [Налаштування номера "Від"](#customizing-the-from-number)
    -   [Routing SMS-сповіщень](#routing-sms-notifications)
-   [Slack Notification](#slack-notifications)
    -   [Передумови](#slack-prerequisites)
    -   [Форматування слабких сповіщень](#formatting-slack-notifications)
    -   [Slack вкладення](#slack-attachments)
    -   [Routing Slack сповіщень](#routing-slack-notifications)
-   [Локалізація сповіщень](#localizing-notifications)
-   [Події Notification](#notification-events)
-   [Спеціальні канали](#custom-channels)

<a name="introduction"></a>

## Вступ

На додаток до підтримки для[відправка електронної пошти](/docs/{{version}}/mail), Laravel надає підтримку надсилання повідомлень за різними каналами доставки, включаючи пошту, SMS (через[Вонаж](https://www.vonage.com/communications-apis/), раніше відомий як Nexmo), і[Ослаблення](https://slack.com). Повідомлення також можуть зберігатися у базі даних, щоб вони могли відображатися у вашому веб-інтерфейсі.

Як правило, Notification повинні бути короткими, інформаційними повідомленнями, які сповіщають користувачів про щось, що сталося у вашій програмі. Наприклад, якщо ви пишете заявку на виставлення рахунку, ви можете надіслати Notification "Оплачений рахунок-фактура" своїм користувачам за допомогою електронної пошти та SMS-каналів.

<a name="creating-notifications"></a>

## Створення сповіщень

У Laravel кожне повідомлення представлене одним класом (зазвичай зберігається в`app/Notifications`каталог). Не хвилюйтеся, якщо ви не бачите цей каталог у своїй програмі, він буде створений для вас під час запуску`make:notification`Команда ремісників:

    php artisan make:notification InvoicePaid

Ця команда помістить новий клас сповіщень у ваш`app/Notifications`каталог. Кожен клас повідомлень містить a`via`і змінну кількість методів побудови повідомлень (таких як`toMail`або`toDatabase`), які перетворюють повідомлення на повідомлення, оптимізоване для цього конкретного каналу.

<a name="sending-notifications"></a>

## Надсилання сповіщень

<a name="using-the-notifiable-trait"></a>

### Using The Notifiable Trait

Повідомлення можуть надсилатися двома способами: за допомогою`notify`метод`Notifiable`ознака або використання`Notification`[фасад](/docs/{{version}}/facades). Спочатку давайте дослідимо, використовуючи ознаку:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;
    }

Ця ознака використовується за замовчуванням`App\Models\User`модель і містить один метод, який можна використовувати для надсилання сповіщень:`notify`.`notify`метод очікує отримати екземпляр Notification:

    use App\Notifications\InvoicePaid;

    $user->notify(new InvoicePaid($invoice));

> {tip} Пам'ятайте, ви можете використовувати`Illuminate\Notifications\Notifiable`властивість будь-якої з ваших моделей. Ви не обмежуєтесь лише включенням цього на свій`User`модель.

<a name="using-the-notification-facade"></a>

### Використання фасаду повідомлень

Крім того, ви можете надсилати Notification через`Notification`[фасад](/docs/{{version}}/facades). Це корисно в першу чергу, коли вам потрібно надіслати Notification до кількох об’єктів, про які потрібно повідомити, наприклад до колекції користувачів. Щоб надіслати Notification за допомогою фасаду, передайте всі об'єкти, що підлягають повідомленню, та екземпляр Notification в`send`метод:

    Notification::send($users, new InvoicePaid($invoice));

<a name="specifying-delivery-channels"></a>

### Визначення каналів доставки

Кожен клас повідомлень має`via`метод, який визначає, за якими каналами буде надходити повідомлення. Повідомлення можуть надсилатися на`mail`,`database`,`broadcast`,`nexmo`, і`slack`каналів.

> {tip} Якщо ви хочете скористатися іншими каналами доставки, такими як Telegram або Pusher, перегляньте спільноту[Веб-сайт Laravel Notification Channels](http://laravel-notification-channels.com).

`via`метод отримує a`$notifiable`екземпляр, який буде екземпляром класу, до якого надсилається Notification. Ви можете використовувати`$notifiable`щоб визначити, за якими каналами слід надсилати повідомлення:

    /**
     * Get the notification's delivery channels.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function via($notifiable)
    {
        return $notifiable->prefers_sms ? ['nexmo'] : ['mail', 'database'];
    }

<a name="queueing-notifications"></a>

### Черга сповіщень

> {note} Перед тим, як сповіщати про чергу, вам слід налаштувати свою чергу та[почати робітника](/docs/{{version}}/queues).

Надсилання сповіщень може зайняти час, особливо якщо каналу потрібен зовнішній виклик API для доставки Notification. Щоб пришвидшити час відповіді вашої програми, нехай ваше повідомлення буде в черзі, додавши`ShouldQueue`інтерфейс і`Queueable`риса до вашого класу. Інтерфейс та ознака вже імпортовані для всіх повідомлень, створених за допомогою`make:notification`, тому ви можете негайно додати їх до свого класу сповіщень:

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Notifications\Notification;

    class InvoicePaid extends Notification implements ShouldQueue
    {
        use Queueable;

        // ...
    }

Одного разу`ShouldQueue`Інтерфейс додано до вашого Notification, ви можете надіслати Notification як зазвичай. Laravel виявить`ShouldQueue`інтерфейс класу і автоматично поставити в чергу доставку повідомлення:

    $user->notify(new InvoicePaid($invoice));

Якщо ви хочете затримати доставку повідомлення, ви можете встановити ланцюжок`delay`методу до вашої інстанції Notification:

    $delay = now()->addMinutes(10);

    $user->notify((new InvoicePaid($invoice))->delay($delay));

Ви можете передати масив до`delay`спосіб вказати величину затримки для певних каналів:

    $user->notify((new InvoicePaid($invoice))->delay([
        'mail' => now()->addMinutes(5),
        'sms' => now()->addMinutes(10),
    ]));

Під час отримання повідомлень у черзі буде створено завдання в черзі для кожного поєднання одержувача та каналу. Наприклад, шість завдань буде відправлено до черги, якщо ваше Notification має трьох одержувачів і два канали.

<a name="customizing-notification-channel-queues"></a>

#### Налаштування черг каналів сповіщень

Якщо ви хочете вказати конкретну чергу, яку слід використовувати для кожного каналу сповіщень, що підтримується Notificationм, ви можете визначити a`viaQueues`у вашому повідомленні. Цей метод повинен повернути масив пар назв каналів / назв черги:

    /**
     * Determine which queues should be used for each notification channel.
     *
     * @return array
     */
    public function viaQueues()
    {
        return [
            'mail' => 'mail-queue',
            'slack' => 'slack-queue',
        ];
    }

<a name="on-demand-notifications"></a>

### Повідомлення на вимогу

Іноді вам може знадобитися надіслати повідомлення комусь, хто не зберігається як "користувач" вашої програми. Використання`Notification::route`методом фасаду, ви можете вказати спеціальну інформацію про маршрутизацію сповіщень перед надсиланням повідомлення:

    Notification::route('mail', 'taylor@example.com')
                ->route('nexmo', '5555555555')
                ->route('slack', 'https://hooks.slack.com/services/...')
                ->notify(new InvoicePaid($invoice));

<a name="mail-notifications"></a>

## Повідомлення поштою

<a name="formatting-mail-messages"></a>

### Форматування поштових повідомлень

Якщо повідомлення підтримує надсилання у вигляді електронного листа, слід визначити`toMail`в класі сповіщень. Цей метод отримає a`$notifiable`і повинен повернути файл`Illuminate\Notifications\Messages\MailMessage`екземпляр. Поштові повідомлення можуть містити рядки тексту, а також "заклик до дії". Давайте розглянемо приклад`toMail`метод:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->greeting('Hello!')
                    ->line('One of your invoices has been paid!')
                    ->action('View Invoice', $url)
                    ->line('Thank you for using our application!');
    }

> {tip} Примітка, яку ми використовуємо`$this->invoice->id`в нашому`toMail`метод. Ви можете передавати будь-які дані, необхідні для створення повідомлення, у конструктор Notification.

У цьому прикладі ми реєструємо привітання, рядок тексту, заклик до дії, а потім ще один рядок тексту. Ці методи передбачені`MailMessage`object дозволяє спростити та швидко форматувати невеликі транзакційні електронні листи. Потім поштовий канал переведе компоненти повідомлень у приємний, чуйний HTML-шаблон електронної пошти зі звичайним текстом. Ось приклад електронного листа, створеного`mail`канал:

<img src="https://laravel.com/img/docs/notification-example-2.png">

> {tip} Під час надсилання сповіщень поштою обов’язково встановіть`name` value in your `config/app.php`файл конфігурації. Це значення буде використано у верхньому та нижньому колонтитулі ваших повідомлень про поштові Notification.

<a name="other-notification-formatting-options"></a>

#### Інші параметри форматування сповіщень

Замість того, щоб визначати "рядки" тексту в класі сповіщень, ви можете використовувати`view`метод, щоб вказати власний шаблон, який слід використовувати для відображення повідомлення електронної пошти:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)->view(
            'emails.name', ['invoice' => $this->invoice]
        );
    }

Ви можете вказати текстовий вигляд для поштового повідомлення, передавши ім'я View як другий елемент масиву, який надається`view`метод`MailMessage`:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)->view(
            ['emails.name.html', 'emails.name.plain'],
            ['invoice' => $this->invoice]
        );
    }

Крім того, ви можете повернути повну[доступний об'єкт](/docs/{{version}}/mail)від`toMail`метод:

    use App\Mail\InvoicePaid as Mailable;

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return Mailable
     */
    public function toMail($notifiable)
    {
        return (new Mailable($this->invoice))->to($notifiable->email);
    }

<a name="error-messages"></a>

#### Повідомлення про помилки

Деякі Notification інформують користувачів про помилки, наприклад, про невдалу оплату рахунка-фактури. Ви можете вказати, що поштове повідомлення стосується помилки, зателефонувавши на`error`метод при побудові вашого повідомлення. При використанні`error`у поштовому повідомленні кнопка із закликом до дії буде червоною, а не синьою:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Message
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->error()
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-sender"></a>

### Налаштування відправника

За замовчуванням адреса відправника / адреса електронної пошти визначена в`config/mail.php`файл конфігурації. Однак ви можете вказати адресу from для конкретного Notification за допомогою`from`метод:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->from('test@example.com', 'Example')
                    ->line('...');
    }

<a name="customizing-the-recipient"></a>

### Налаштування одержувача

При надсиланні повідомлень через`mail`канал, система Notification автоматично шукатиме`email`власність на вашу юридичну особу. Ви можете налаштувати, яка електронна адреса використовується для надсилання Notification, визначивши a`routeNotificationForMail`метод на сутність:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the mail channel.
         *
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return array|string
         */
        public function routeNotificationForMail($notification)
        {
            // Return email address only...
            return $this->email_address;

            // Return name and email address...
            return [$this->email_address => $this->name];
        }
    }

<a name="customizing-the-subject"></a>

### Налаштування теми

За замовчуванням темою повідомлення електронної пошти є назва класу Notification, відформатоване як "заголовок". Отже, якщо названо ваш клас сповіщень`InvoicePaid`, темою повідомлення буде`Invoice Paid`. Якщо ви хочете вказати явну тему для повідомлення, ви можете зателефонувати за номером`subject`метод при побудові вашого повідомлення:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-mailer"></a>

### Налаштування пошти

За замовчуванням Notification електронною поштою надсилатиметься із драйвером за замовчуванням, визначеним у`config/mail.php`файл конфігурації. Однак ви можете вказати іншу поштову скриньку під час виконання, зателефонувавши до`mailer`метод при побудові вашого повідомлення:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->mailer('postmark')
                    ->line('...');
    }

<a name="customizing-the-templates"></a>

### Налаштування шаблонів

Ви можете змінити HTML і шаблон простого тексту, що використовуються у Notificationх електронною поштою, опублікувавши ресурси пакета сповіщень. Після запуску цієї команди шаблони повідомлень про пошту будуть розміщені в`resources/views/vendor/notifications`каталог:

    php artisan vendor:publish --tag=laravel-notifications

<a name="mail-attachments"></a>

### Вкладення

Щоб додати вкладення до Notification електронною поштою, використовуйте`attach`метод під час створення вашого повідомлення.`attach`метод приймає повний (абсолютний) шлях до файлу як перший аргумент:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->greeting('Hello!')
                    ->attach('/path/to/file');
    }

Прикріплюючи файли до повідомлення, ви також можете вказати відображуване ім'я та / або тип MIME, передавши файл`array`як другий аргумент`attach`метод:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->greeting('Hello!')
                    ->attach('/path/to/file', [
                        'as' => 'name.pdf',
                        'mime' => 'application/pdf',
                    ]);
    }

> {tip} На відміну від вкладання файлів у доступні об’єкти, ви не можете вкладати файл безпосередньо з диска зберігання за допомогою`attachFromStorage`. Вам краще скористатися`attach`метод з абсолютним шляхом до файлу на диску зберігання. Як варіант, ви можете повернути a[доступний](/docs/{{version}}/mail#generating-mailables)від`toMail`метод.

<a name="raw-data-attachments"></a>

#### Вкладені файли необроблених даних

`attachData`метод може бути використаний для приєднання необробленого рядка байтів як вкладення:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->greeting('Hello!')
                    ->attachData($this->pdf, 'name.pdf', [
                        'mime' => 'application/pdf',
                    ]);
    }

<a name="previewing-mail-notifications"></a>

### Попередній перегляд створених сповіщень

При розробці шаблону Notification про пошту зручно швидко переглядати відтворене поштове повідомлення у вашому браузері, як типовий шаблон Blade. З цієї причини Laravel дозволяє повертати будь-яке поштове повідомлення, генероване поштовим повідомленням, безпосередньо із Закриття маршруту або контролера. Коли a`MailMessage`повернеться, він буде відтворений і відображений у браузері, що дозволить вам швидко переглянути його дизайн без необхідності надсилати його на фактичну електронну адресу:

    Route::get('mail', function () {
        $invoice = App\Invoice::find(1);

        return (new App\Notifications\InvoicePaid($invoice))
                    ->toMail($invoice->user);
    });

<a name="markdown-mail-notifications"></a>

## Повідомлення поштової розсилки

Notification про поштову розсилку дозволяють скористатися заздалегідь побудованими шаблонами сповіщень про пошту, надаючи вам більше свободи для написання довших, налаштованих повідомлень. Оскільки повідомлення написані в Markdown, Laravel може рендерити чудові, чуйні HTML-шаблони для повідомлень, а також автоматично генерувати аналог простого тексту.

<a name="generating-the-message"></a>

### Створення повідомлення

Щоб створити Notification з відповідним шаблоном Markdown, ви можете використовувати`--markdown`варіант`make:notification`Команда ремісників:

    php artisan make:notification InvoicePaid --markdown=mail.invoice.paid

Як і всі інші Notification поштою, Notification, які використовують шаблони Markdown, повинні визначати a`toMail`метод їх класу сповіщень. Однак замість використання`line`і`action`методи побудови повідомлення, використовуйте`markdown`метод, щоб вказати назву шаблону Markdown, який слід використовувати:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->subject('Invoice Paid')
                    ->markdown('mail.invoice.paid', ['url' => $url]);
    }

<a name="writing-the-message"></a>

### Написання повідомлення

Повідомлення Markdown в пошті використовують комбінацію компонентів Blade та синтаксису Markdown, які дозволяють легко створювати повідомлення, використовуючи заздалегідь створені компоненти сповіщень Laravel:

    @component('mail::message')
    # Invoice Paid

    Your invoice has been paid!

    @component('mail::button', ['url' => $url])
    View Invoice
    @endcomponent

    Thanks,<br>
    {{ config('app.name') }}
    @endcomponent

<a name="button-component"></a>

#### Кнопка Компонент

Компонент кнопки відображає відцентрове посилання кнопки. Компонент приймає два аргументи, a`url`та необов’язково`color`. Підтримувані кольори`blue`,`green`, і`red`. Ви можете додати до Notification скільки завгодно компонентів кнопок:

    @component('mail::button', ['url' => $url, 'color' => 'green'])
    View Invoice
    @endcomponent

<a name="panel-component"></a>

#### Компонент панелі

Компонент панелі відображає даний блок тексту на панелі, яка має дещо інший колір фону, ніж решта Notification. Це дозволяє звернути увагу на певний блок тексту:

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

Ви можете експортувати всі компоненти Notification Markdown у власну програму для налаштування. Щоб експортувати компоненти, використовуйте`vendor:publish`Реміснича команда видавати`laravel-mail`тег активу:

    php artisan vendor:publish --tag=laravel-mail

Ця команда опублікує компоненти пошти Markdown у`resources/views/vendor/mail`каталог.`mail`каталог міститиме`html`і a`text`каталог, кожен із відповідних Views кожного доступного компонента. Ви можете налаштувати ці компоненти як завгодно.

<a name="customizing-the-css"></a>

#### Налаштування CSS

Після експорту компонентів файл`resources/views/vendor/mail/html/themes`каталог міститиме файл`default.css`файл. Ви можете налаштувати CSS у цьому файлі, і ваші стилі будуть автоматично вставлені в HTML-зображення ваших сповіщень Markdown.

Якщо ви хочете створити абсолютно нову тему для компонентів Laravel Markdown, ви можете розмістити файл CSS у`html/themes`каталог. Після іменування та збереження файлу CSS оновіть файл`theme`варіант`mail`конфігураційний файл відповідно до назви вашої нової теми.

Щоб налаштувати тему для окремого Notification, ви можете зателефонувати до`theme`метод під час створення поштового повідомлення Notification.`theme`метод приймає назву теми, яку слід використовувати під час надсилання Notification:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->theme('invoice')
                    ->subject('Invoice Paid')
                    ->markdown('mail.invoice.paid', ['url' => $url]);
    }

<a name="database-notifications"></a>

## Notification бази даних

<a name="database-prerequisites"></a>

### Передумови

`database`канал сповіщень зберігає інформацію Notification в таблиці бази даних. Ця таблиця міститиме таку інформацію, як тип Notification, а також користувацькі дані JSON, що описують Notification.

Ви можете запитати таблицю, щоб відобразити Notification в інтерфейсі користувача вашої програми. Але перед тим, як це зробити, вам потрібно буде створити таблицю бази даних для зберігання ваших сповіщень. Ви можете використовувати`notifications:table`команда для генерації міграції за допомогою відповідної схеми таблиці:

    php artisan notifications:table

    php artisan migrate

<a name="formatting-database-notifications"></a>

### Форматування сповіщень баз даних

Якщо Notification підтримує збереження в таблиці бази даних, слід визначити a`toDatabase`або`toArray`в класі сповіщень. Цей метод отримає a`$notifiable`сутність і повинен повертати звичайний масив PHP. Повернутий масив буде закодований як JSON і збережений у`data`стовпець вашого`notifications`таблиця. Давайте розглянемо приклад`toArray`метод:

    /**
     * Get the array representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function toArray($notifiable)
    {
        return [
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ];
    }

<a name="todatabase-vs-toarray"></a>

#### `toDatabase`Проти`toArray`

`toArray`метод також використовується методом`broadcast`канал, щоб визначити, які дані транслювати на ваш клієнт JavaScript. Якщо ви хочете мати два різних View масиву для`database`і`broadcast`канали, вам слід визначити a`toDatabase`метод замість a`toArray`метод.

<a name="accessing-the-notifications"></a>

### Доступ до сповіщень

Після того, як Notification зберігаються в базі даних, вам потрібен зручний спосіб отримати доступ до них із ваших об’єктів, про які потрібно повідомити.`Illuminate\Notifications\Notifiable`ознака, яка включена за замовчуванням Laravel`App\Models\User`модель, включає a`notifications`Красномовний зв’язок, який повертає повідомлення для сутності. Щоб отримати Notification, ви можете отримати доступ до цього методу, як і до будь-якого іншого красномовного зв'язку. За замовчуванням Notification сортуються за`created_at`позначка часу:

    $user = App\Models\User::find(1);

    foreach ($user->notifications as $notification) {
        echo $notification->type;
    }

Якщо ви хочете отримати лише "непрочитані" повідомлення, ви можете використовувати`unreadNotifications`відносини. Знову ж таки, ці Notification буде відсортовано за`created_at`позначка часу:

    $user = App\Models\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        echo $notification->type;
    }

> {tip} Щоб отримати доступ до сповіщень із вашого клієнта JavaScript, слід визначити контролер сповіщень для своєї програми, який повертає Notification для об’єкта, що підлягає повідомленню, наприклад поточного користувача. Потім ви можете зробити запит HTTP на URI цього контролера від свого клієнта JavaScript.

<a name="marking-notifications-as-read"></a>

### Позначення повідомлень як прочитаних

Як правило, ви хочете позначити повідомлення як "прочитане", коли користувач його переглядає.`Illuminate\Notifications\Notifiable`ознака забезпечує a`markAsRead`метод, який оновлює`read_at`стовпець у записі бази даних Notification:

    $user = App\Models\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        $notification->markAsRead();
    }

Однак, замість перегляду кожного повідомлення, ви можете використовувати`markAsRead`метод безпосередньо на колекції сповіщень:

    $user->unreadNotifications->markAsRead();

Ви також можете використовувати запит масового оновлення, щоб позначити всі Notification як прочитані, не отримуючи їх із бази даних:

    $user = App\Models\User::find(1);

    $user->unreadNotifications()->update(['read_at' => now()]);

Ви можете`delete`Notification, щоб повністю видалити їх із таблиці:

    $user->notifications()->delete();

<a name="broadcast-notifications"></a>

## Broadcast Notifications

<a name="broadcast-prerequisites"></a>

### Передумови

Перш ніж транслювати Notification, слід налаштувати та ознайомитись із повідомленнями Laravel[трансляція подій](/docs/{{version}}/broadcasting)послуги. Трансляція подій забезпечує спосіб реагування на серверні події Laravel із вашого клієнта JavaScript.

<a name="formatting-broadcast-notifications"></a>

### Форматування сповіщень про Broadcast

`broadcast`канал транслює повідомлення за допомогою Laravel[трансляція подій](/docs/{{version}}/broadcasting)служби, що дозволяє вашому клієнту JavaScript отримувати Notification в режимі реального часу. Якщо повідомлення підтримує Broadcast, ви можете визначити a`toBroadcast`в класі сповіщень. Цей метод отримає a`$notifiable`сутність і повинна повернути a`BroadcastMessage`екземпляр. Якщо`toBroadcast`метод не існує,`toArray`метод буде використаний для збору даних, які слід транслювати. Повернуті дані будуть закодовані як JSON і передаватимуться на ваш клієнт JavaScript. Давайте розглянемо приклад`toBroadcast`метод:

    use Illuminate\Notifications\Messages\BroadcastMessage;

    /**
     * Get the broadcastable representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return BroadcastMessage
     */
    public function toBroadcast($notifiable)
    {
        return new BroadcastMessage([
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ]);
    }

<a name="broadcast-queue-configuration"></a>

#### Конфігурація черги мовлення

Усі Notification про Broadcast в черзі на Broadcast. Якщо ви хочете налаштувати підключення черги або ім'я черги, що використовується для черги операції трансляції, ви можете використовувати`onConnection`і`onQueue`методи`BroadcastMessage`:

    return (new BroadcastMessage($data))
                    ->onConnection('sqs')
                    ->onQueue('broadcasts');

<a name="customizing-the-notification-type"></a>

#### Налаштування типу повідомлення

Окрім зазначених вами даних, усі Broadcast Notifications також мають символ`type`поле, що містить повну назву класу Notification. Якщо ви хочете налаштувати повідомлення`type`що надається вашому клієнту JavaScript, ви можете визначити`broadcastType`метод у класі сповіщень:

    use Illuminate\Notifications\Messages\BroadcastMessage;

    /**
     * Get the type of the notification being broadcast.
     *
     * @return string
     */
    public function broadcastType()
    {
        return 'broadcast.message';
    }

<a name="listening-for-notifications"></a>

### Прослуховування сповіщень

Notification транслюватимуться на приватному каналі, відформатованому за допомогою`{notifiable}.{id}`конвенції. Отже, якщо ви надсилаєте повідомлення на`App\Models\User`екземпляр з ідентифікатором`1`, повідомлення буде транслюватися на`App.Models.User.1`приватний канал. При використанні[Laravel Echo](/docs/{{version}}/broadcasting), Ви можете легко прослуховувати Notification на каналі за допомогою`notification`допоміжний метод:

    Echo.private('App.Models.User.' + userId)
        .notification((notification) => {
            console.log(notification.type);
        });

<a name="customizing-the-notification-channel"></a>

#### Налаштування каналу сповіщень

Якщо ви хочете налаштувати, на яких каналах суб’єкт, що підлягає повідомленню, отримує Notification про Broadcast, ви можете визначити a`receivesBroadcastNotificationsOn`метод на об'єкт, що підлягає повідомленню:

    <?php

    namespace App\Models;

    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * The channels the user receives notification broadcasts on.
         *
         * @return string
         */
        public function receivesBroadcastNotificationsOn()
        {
            return 'users.'.$this->id;
        }
    }

<a name="sms-notifications"></a>

## SMS-Notification

<a name="sms-prerequisites"></a>

### Передумови

Надсилання SMS-сповіщень у Laravel забезпечується[Nexmo](https://www.nexmo.com/). Перш ніж ви зможете надсилати Notification через Nexmo, вам потрібно встановити`laravel/nexmo-notification-channel`Пакет композитора:

    composer require laravel/nexmo-notification-channel

Це також встановить[`nexmo/laravel`](https://github.com/Nexmo/nexmo-laravel)пакет. Цей пакет включає[власний файл конфігурації](https://github.com/Nexmo/nexmo-laravel/blob/master/config/nexmo.php). Ви можете використовувати`NEXMO_KEY`і`NEXMO_SECRET`змінні середовища для встановлення вашого відкритого та секретного ключа Nexmo.

Далі вам потрібно буде додати параметр конфігурації до вашого`config/services.php`файл конфігурації. Для початку можна скопіювати приклад конфігурації нижче:

    'nexmo' => [
        'sms_from' => '15556666666',
    ],

`sms_from`Опція - це номер телефону, з якого будуть надсилатися ваші SMS-повідомлення. Ви повинні створити номер телефону для своєї програми на панелі керування Nexmo.

<a name="formatting-sms-notifications"></a>

### Форматування сповіщень SMS

Якщо Notification підтримує надсилання у вигляді SMS, слід визначити a`toNexmo`в класі сповіщень. Цей метод отримає a`$notifiable`і повинен повернути файл`Illuminate\Notifications\Messages\NexmoMessage`примірник:

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content');
    }

<a name="formatting-shortcode-notifications"></a>

### Форматування сповіщень із шорткодом

Laravel також підтримує надсилання сповіщень із шорткодом, які є заздалегідь визначеними шаблонами повідомлень у вашому обліковому записі Nexmo. Ви можете вказати тип повідомлення (`alert`,`2fa`, або`marketing`), а також користувацькі значення, які заповнюватимуть шаблон:

    /**
     * Get the Nexmo / Shortcode representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function toShortcode($notifiable)
    {
        return [
            'type' => 'alert',
            'custom' => [
                'code' => 'ABC123',
            ];
        ];
    }

> {tip} Подобається[Routing SMS-сповіщень](#routing-sms-notifications), вам слід реалізувати`routeNotificationForShortcode`метод на вашій моделі, що підлягає повідомленню.

<a name="unicode-content"></a>

#### Вміст Unicode

Якщо ваше SMS-повідомлення буде містити символи Unicode, вам слід зателефонувати до`unicode`метод при побудові`NexmoMessage`примірник:

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your unicode message')
                    ->unicode();
    }

<a name="customizing-the-from-number"></a>

### Налаштування номера "Від"

Якщо ви хочете надіслати деякі Notification з номера телефону, який відрізняється від номера телефону, вказаного у вашому`config/services.php`файл, ви можете використовувати файл`from`метод на a`NexmoMessage`примірник:

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content')
                    ->from('15554443333');
    }

<a name="routing-sms-notifications"></a>

### Routing SMS-сповіщень

Щоб направити Notification Nexmo на відповідний номер телефону, визначте a`routeNotificationForNexmo`метод на вашому об’єкті, про який потрібно повідомити:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the Nexmo channel.
         *
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return string
         */
        public function routeNotificationForNexmo($notification)
        {
            return $this->phone_number;
        }
    }

<a name="slack-notifications"></a>

## Slack Notification

<a name="slack-prerequisites"></a>

### Передумови

Перш ніж ви зможете надсилати Notification через Slack, ви повинні встановити канал сповіщень через Composer:

    composer require laravel/slack-notification-channel

Вам також потрібно буде налаштувати файл["Вхідний веб-хук"](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks)інтеграція для вашої команди Slack. Ця інтеграція надасть вам URL-адресу, яку ви можете використовувати, коли[Routing повідомлень Slack](#routing-slack-notifications).

<a name="formatting-slack-notifications"></a>

### Форматування слабких сповіщень

Якщо Notification підтримує надсилання як повідомлення Slack, слід визначити a`toSlack`в класі сповіщень. Цей метод отримає a`$notifiable`і повинен повернути файл`Illuminate\Notifications\Messages\SlackMessage`екземпляр. Slack повідомлення можуть містити текстовий вміст, а також "вкладення", що форматує додатковий текст або масив полів. Давайте поглянемо на основне`toSlack`приклад:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->content('One of your invoices has been paid!');
    }

<a name="customizing-the-sender-recipient"></a>

#### Налаштування відправника та одержувача

Ви можете використовувати`from`і`to`методи налаштування відправника та одержувача.`from`метод приймає ім'я користувача та ідентифікатор смайликів, тоді як`to`метод приймає канал або ім'я користувача:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->from('Ghost', ':ghost:')
                    ->to('#other')
                    ->content('This will be sent to #other');
    }

Ви також можете використовувати зображення як свій логотип замість емодзі:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->from('Laravel')
                    ->image('https://laravel.com/img/favicon/favicon.ico')
                    ->content('This will display the Laravel logo next to the message');
    }

<a name="slack-attachments"></a>

### Slack вкладення

Ви також можете додати "вкладення" до повідомлень Slack. Вкладені файли мають більше можливостей форматування, ніж прості текстові повідомлення. У цьому прикладі ми надішлемо Notification про помилку про виняток, який стався в програмі, включаючи посилання для перегляду детальної інформації про виняток:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);

        return (new SlackMessage)
                    ->error()
                    ->content('Whoops! Something went wrong.')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Exception: File Not Found', $url)
                                   ->content('File [background.jpg] was not found.');
                    });
    }

Вкладені файли також дозволяють вказати масив даних, який повинен бути представлений користувачеві. Наведені дані будуть представлені у форматі таблиці для зручності читання:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/invoices/'.$this->invoice->id);

        return (new SlackMessage)
                    ->success()
                    ->content('One of your invoices has been paid!')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Invoice 1322', $url)
                                   ->fields([
                                        'Title' => 'Server Expenses',
                                        'Amount' => '$1,234',
                                        'Via' => 'American Express',
                                        'Was Overdue' => ':-1:',
                                    ]);
                    });
    }

<a name="markdown-attachment-content"></a>

#### Зміст вкладення розмітки

Якщо деякі поля вкладених файлів містять Markdown, ви можете використовувати`markdown`метод, щоб доручити Slack проаналізувати та відобразити дані поля вкладень як текст у форматі Markdown. Значення, прийняті цим методом:`pretext`,`text`, та / або`fields`. Для отримання додаткової інформації про форматування вкладених файлів Slack перегляньте[Документація API Slack](https://api.slack.com/docs/message-formatting#message_formatting):

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);

        return (new SlackMessage)
                    ->error()
                    ->content('Whoops! Something went wrong.')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Exception: File Not Found', $url)
                                   ->content('File [background.jpg] was *not found*.')
                                   ->markdown(['text']);
                    });
    }

<a name="routing-slack-notifications"></a>

### Routing Slack сповіщень

Щоб направити Notification Slack у належне місце, визначте a`routeNotificationForSlack`метод на вашому об’єкті, про який потрібно повідомити. Це повинно повернути URL-адресу веб-хука, на яку має надходити повідомлення. URL-адреси Webhook можуть бути сформовані додаванням служби "Вхідний Webhook" до вашої команди Slack:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the Slack channel.
         *
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return string
         */
        public function routeNotificationForSlack($notification)
        {
            return 'https://hooks.slack.com/services/...';
        }
    }

<a name="localizing-notifications"></a>

## Локалізація сповіщень

Laravel дозволяє надсилати Notification мовою, відмінною від поточної мови, і навіть запам'ятовуватиме цю локаль, якщо Notification буде поставлено в чергу.

Для цього,`Illuminate\Notifications\Notification`клас пропонує a`locale`спосіб встановити потрібну мову. Програма зміниться на цю локаль під час форматування Notification, а потім повернеться до попередньої мови після завершення форматування:

    $user->notify((new InvoicePaid($invoice))->locale('es'));

Локалізація декількох повідомлюваних записів також може бути досягнута за допомогою`Notification`фасад:

    Notification::locale('es')->send($users, new InvoicePaid($invoice));

<a name="user-preferred-locales"></a>

### Локалі, що надаються користувачем

Іноді програми зберігають улюблений регіон кожного користувача. Впроваджуючи`HasLocalePreference`контракт на вашу модель, що підлягає повідомленню, ви можете доручити Laravel використовувати цю збережену локаль під час надсилання повідомлення:

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

Після того, як ви впровадите інтерфейс, Laravel автоматично використовуватиме бажану локаль під час надсилання сповіщень та доступних моделей до моделі. Тому немає необхідності телефонувати до`locale`метод при використанні цього інтерфейсу:

    $user->notify(new InvoicePaid($invoice));

<a name="notification-events"></a>

## Події Notification

Коли повідомлення надіслано,`Illuminate\Notifications\Events\NotificationSent`подія запускається системою Notification. Він містить сутність, про яку потрібно повідомити, та сам екземпляр Notification. Ви можете зареєструвати слухачів для цієї події у своєму`EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Notifications\Events\NotificationSent' => [
            'App\Listeners\LogNotification',
        ],
    ];

> {tip} Після реєстрації слухачів у вашому`EventServiceProvider`, використовувати`event:generate`Команда Artisan для швидкого створення класів слухачів.

У програмі прослуховування подій ви можете отримати доступ до`notifiable`,`notification`, і`channel`властивості події, щоб дізнатись більше про одержувача Notification або саме Notification:

    /**
     * Handle the event.
     *
     * @param  NotificationSent  $event
     * @return void
     */
    public function handle(NotificationSent $event)
    {
        // $event->channel
        // $event->notifiable
        // $event->notification
        // $event->response
    }

<a name="custom-channels"></a>

## Спеціальні канали

Laravel постачається з декількома каналами сповіщень, але ви можете написати власні драйвери для доставки повідомлень за іншими каналами. Laravel робить це просто. Для початку визначте клас, який містить`send`метод. Метод повинен отримувати два аргументи: a`$notifiable`і a`$notification`:

    <?php

    namespace App\Channels;

    use Illuminate\Notifications\Notification;

    class VoiceChannel
    {
        /**
         * Send the given notification.
         *
         * @param  mixed  $notifiable
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return void
         */
        public function send($notifiable, Notification $notification)
        {
            $message = $notification->toVoice($notifiable);

            // Send notification to the $notifiable instance...
        }
    }

Після визначення класу вашого каналу сповіщень ви можете повернути назву класу з`via`спосіб будь-якого з ваших сповіщень:

    <?php

    namespace App\Notifications;

    use App\Channels\Messages\VoiceMessage;
    use App\Channels\VoiceChannel;
    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Notifications\Notification;

    class InvoicePaid extends Notification
    {
        use Queueable;

        /**
         * Get the notification channels.
         *
         * @param  mixed  $notifiable
         * @return array|string
         */
        public function via($notifiable)
        {
            return [VoiceChannel::class];
        }

        /**
         * Get the voice representation of the notification.
         *
         * @param  mixed  $notifiable
         * @return VoiceMessage
         */
        public function toVoice($notifiable)
        {
            // ...
        }
    }
