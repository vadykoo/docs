# Laravel Cashier Paddle

-   [Вступ](#introduction)
-   [Оновлення Cashier](#upgrading-cashier)
-   [Встановлення](#installation)
-   [Конфігурація](#configuration)
    -   [Billable Модель](#billable-model)
    -   [Ключі API](#api-keys)
    -   [Paddle JS](#paddle-js)
    -   [Конфігурація валюти](#currency-configuration)
-   [Основні поняття](#core-concepts)
    -   [Платні посилання](#pay-links)
    -   [Inline оплата](#inline-checkout)
    -   [Ідентифікація користувача](#user-identification)
-   [Ціни](#prices)
-   [Клієнти](#customers)
    -   [Клієнт Defaults](#customer-defaults)
-   [Підписки](#subscriptions)
    -   [Створення підписки](#creating-subscriptions)
    -   [Перевірка статусу підписки](#checking-subscription-status)
    -   [Одноразова плата за передплату](#subscription-single-charges)
    -   [Оновлення платіжної інформації](#updating-payment-information)
    -   [Зміна планів](#changing-plans)
    -   [Кількість підписок](#subscription-quantity)
    -   [Призупинення передплати](#pausing-subscriptions)
    -   [Скасування передплати](#cancelling-subscriptions)
-   [Випробувальні підписки](#subscription-trials)
    -   [Зі способом оплати вперед](#with-payment-method-up-front)
    -   [Без способу оплати](#without-payment-method-up-front)
-   [Обробка Paddleвих веб-хуків](#handling-paddle-webhooks)
    -   [Визначення обробників подій Webhook](#defining-webhook-event-handlers)
    -   [Невдалі підписки](#handling-failed-subscriptions)
    -   [Перевірка підписів Webhook](#verifying-webhook-signatures)
-   [Одноразові збори](#single-charges)
    -   [Проста зарядка](#simple-charge)
    -   [Зарядні вироби](#charging-products)
    -   [Повернення коштів за замовлення](#refunding-orders)
-   [Receipts](#receipts)
    -   [Минулі та майбутні платежі](#past-and-upcoming-payments)
-   [Обробка невдалих платежів](#handling-failed-payments)
-   [Тестування](#testing)

<a name="introduction"></a>
## Вступ

Laravel Cashier Paddle забезпечує виразний, вільний інтерфейс для[Весла](https://paddle.com)послуги оплати передплати. Він обробляє майже весь код оплати передплати, який ви боїтеся. На додаток до основного управління передплатою, Касир може обробляти: купони, обмін підпискою, "кількість" передплати, пільгові періоди скасування тощо.

Під час співпраці з касою ми рекомендуємо вам також звертатися до Paddle's[посібники користувача](https://developer.paddle.com/guides)і[Документація API](https://developer.paddle.com/api-reference/intro).
<a name="upgrading-cashier"></a>

## Оновлення Cashier

Під час оновлення до нової версії Каси важливо уважно переглянути[посібник з оновлення](https://github.com/laravel/cashier-paddle/blob/master/UPGRADE.md).

<a name="installation"></a>

## Встановлення

По-перше, вимагайте пакет Cashier для Paddle with Composer:

    composer require laravel/cashier-paddle

> {note} Щоб гарантувати, що Касир правильно обробляє всі події Paddle, пам’ятайте[налаштувати обробку веб-хука Касира](#handling-paddle-webhooks).

<a name="database-migrations"></a>

#### Міграції баз даних

Постачальник послуг каси реєструє власний каталог міграції бази даних, тому не забудьте перенести вашу базу даних після встановлення пакету. Міграція касира створить нову`customers`таблиця. Крім того, новий`subscriptions`буде створена таблиця для зберігання всіх підписок вашого клієнта. Нарешті, новий`receipts`буде створена таблиця для зберігання всієї інформації про квитанцію:

    php artisan migrate

Якщо вам потрібно переписати міграції, що постачаються разом із пакетом Cashier, ви можете опублікувати їх за допомогою`vendor:publish`Команда ремісників:

    php artisan vendor:publish --tag="cashier-migrations"

Якщо ви хочете запобігти повному запуску міграції Каси, ви можете скористатися`ignoreMigrations`надані Касиром. Як правило, цей метод слід викликати в`register`метод вашого`AppServiceProvider`:

    use Laravel\Paddle\Cashier;

    Cashier::ignoreMigrations();

<a name="configuration"></a>

## Конфігурація

<a name="billable-model"></a>

### Billable Модель

Перед використанням каси ви повинні додати`Billable`властивість до визначення вашої моделі користувача. Ця риса пропонує різні методи, що дозволяють виконувати загальні завдання щодо виставлення рахунків, такі як створення підписок, застосування купонів та оновлення інформації про спосіб оплати:

    use Laravel\Paddle\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

Якщо у вас є платні сутності, які не є користувачами, ви також можете додати ознаку до цих класів:

    use Laravel\Paddle\Billable;

    class Team extends Model
    {
        use Billable;
    }

<a name="api-keys"></a>

### Ключі API

Далі слід налаштувати ключі Paddle у вашому`.env`файл. Ви можете отримати свої ключі API Paddle з панелі управління Paddle:

    PADDLE_VENDOR_ID=your-paddle-vendor-id
    PADDLE_VENDOR_AUTH_CODE=your-paddle-vendor-auth-code
    PADDLE_PUBLIC_KEY="your-paddle-public-key"

<a name="paddle-js"></a>

### Paddle JS

Paddle покладається на власну бібліотеку JavaScript, щоб ініціювати віджет перевірки Paddle. Ви можете завантажити бібліотеку JavaScript, розмістивши`@paddleJS`директиву безпосередньо перед закриттям макета вашого додатка`</head>`тег:

    <head>
        ...

        @paddleJS
    </head>

<a name="currency-configuration"></a>

### Конфігурація валюти

Валютою каси за замовчуванням є долари США (USD). Ви можете змінити валюту за замовчуванням, встановивши`CASHIER_CURRENCY`змінна середовища:

    CASHIER_CURRENCY=EUR

Окрім налаштування валюти Каси, Ви також можете вказати локаль, який використовуватиметься при форматуванні грошових значень для відображення на рахунках-фактурах. Внутрішньо Касир використовує[PHP`NumberFormatter`клас](https://www.php.net/manual/en/class.numberformatter.php)щоб встановити локаль валюти:

    CASHIER_CURRENCY_LOCALE=nl_BE

> {note} Для того, щоб використовувати локалі, крім`en`, забезпечити`ext-intl`Розширення PHP встановлено та налаштовано на вашому сервері.

<a name="core-concepts"></a>

## Основні поняття

<a name="pay-links"></a>

### Платні посилання

Paddle не має розширеного API CRUD для зміни стану. Тому більшість взаємодій з Paddle здійснюються через[його віджет оформлення замовлення](https://developer.paddle.com/guides/how-tos/checkout/paddle-checkout). Перш ніж ми зможемо відобразити віджет оплати, ми створимо "посилання на оплату" за допомогою Каси:

    $user = User::find(1);

    $payLink = $user->newSubscription('default', $premium = 34567)
        ->returnTo(route('home'))
        ->create();

    return view('billing', ['payLink' => $payLink]);

Касир включає в себе`paddle-button`Компонент леза. Ми можемо передати URL-адресу посилання на оплату цьому компоненту як "підставку". Після натискання цієї кнопки відображатиметься віджет оформлення замовлення Paddle:

    <x-paddle-button :url="$payLink" class="px-8 py-4">
        Subscribe
    </x-paddle-button>

За замовчуванням на ньому відображатиметься кнопка зі стандартним стилем Paddle. Ви можете видалити всі стилі Paddle, додавши`data-theme="none"`атрибут компонента:

    <x-paddle-button :url="$payLink" class="px-8 py-4" data-theme="none">
        Subscribe
    </x-paddle-button>

Віджет виплати весла є асинхронним. Після того, як користувач створить або оновить передплату у віджеті, Paddle надішле веб-хуки нашої програми, щоб ми могли належним чином оновити стан передплати у власній базі даних. Тому важливо, щоб ви були належним чином[налаштувати веб-хуки](#handling-paddle-webhooks)щоб пристосуватись до змін штату від Весла.

Після зміни стану передплати затримка отримання відповідного веб-хука, як правило, мінімальна, але вам слід врахувати це у своїй програмі, враховуючи, що передплата вашого користувача може бути недоступна відразу після завершення оформлення замовлення.

Для отримання додаткової інформації ви можете переглянути[документація API Paddle щодо генерації посилань на оплату](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink).

<a name="inline-checkout"></a>

### Вбудована оплата

Якщо ви не хочете використовувати віджет для оформлення замовлення в стилі "overlay", Paddle також має можливість відображати віджет вбудовано. Хоча такий підхід не дозволяє вам налаштувати жодне з полів HTML каси, він дозволяє вбудовувати віджет у вашу програму.

Щоб полегшити вам початок роботи з вбудованою касою, Cashier включає в себе`paddle-checkout`Компонент леза. Для початку вам слід[сформувати посилання на оплату](#pay-links)і передайте посилання на оплату компоненту`override`атрибут:

    <x-paddle-checkout :override="$payLink" class="w-full" />

Щоб відрегулювати висоту вбудованого компонента оформлення замовлення, ви можете передати`height`атрибут компонента Blade:

    <x-paddle-checkout :override="$payLink" class="w-full" height="500" />

<a name="inline-checkout-without-pay-links"></a>

#### Вбудована оплата без посилань на оплату

Крім того, ви можете налаштувати віджет за допомогою власних параметрів замість використання посилання на оплату:

    $options = [
        'product' => $productId,
        'title' => 'Product Title',
    ];

    <x-paddle-checkout :options="$options" class="w-full" />

Будь ласка, зверніться до Paddle's[путівник по Inline Checkout](https://developer.paddle.com/guides/how-tos/checkout/inline-checkout)а також їх[Посилання на параметри](https://developer.paddle.com/reference/paddle-js/parameters)для отримання детальної інформації про доступні варіанти.

> {note} Якщо ви хочете також використовувати`passthrough`параметр під час вказівки користувацьких параметрів, ви повинні надати масив ключ / значення, оскільки Cashier буде автоматично обробляти перетворення масиву в рядок JSON. Крім того,`customer_id`Параметр passthrough зарезервований для внутрішнього використання Касира.

<a name="user-identification"></a>

### Ідентифікація користувача

На відміну від Stripe, користувачі Paddle унікальні для всього Paddle, а не унікальні для кожного акаунта Paddle. Через це API Paddle наразі не надають способу оновлення даних користувача, таких як їх електронна адреса. При формуванні посилань на оплату Paddle визначає користувачів за допомогою`customer_email`параметр. Створюючи передплату, Paddle спробує встановити відповідність наданого користувачем електронного листа існуючому користувачеві Paddle.

У світлі такої поведінки є кілька важливих речей, про які слід пам’ятати, користуючись Cashier та Paddle. По-перше, ви повинні пам’ятати, що навіть якщо підписки в Cashier пов’язані з одним користувачем програми,**їх можна прив'язати до різних користувачів у внутрішніх системах Paddle**. По-друге, кожна передплата має свою підключену інформацію про спосіб оплати, а також може мати різні адреси електронної пошти у внутрішніх системах Paddle (залежно від того, яка електронна пошта була призначена користувачеві під час створення передплати).

Тому, показуючи підписки, ви завжди повинні повідомляти користувачеві, яка електронна адреса чи інформація про спосіб оплати підключена до передплати на основі підписки. Отримати цю інформацію можна наступними методами на`Subscription`модель:

    $subscription = $user->subscription('default');

    $customerEmailAddress = $subscription->paddleEmail();
    $paymentMethod = $subscription->paymentMethod();
    $cardBrand = $subscription->cardBrand();
    $cardLastFour = $subscription->cardLastFour();
    $cardExpirationDate = $subscription->cardExpirationDate();

Наразі неможливо змінити електронну адресу користувача за допомогою API Paddle. Коли користувач хоче оновити свою електронну адресу в Paddle, єдиний спосіб зробити це - зв’язатися зі службою підтримки Paddle. Спілкуючись із Paddle, вони повинні надати`paddleEmail`значення передплати на допомогу Paddle в оновленні правильного користувача.

<a name="prices"></a>

## Ціни

Paddle дозволяє налаштовувати ціни на валюту, по суті, дозволяючи налаштовувати різні ціни для різних країн. Cashier Paddle дозволяє отримати всі ціни на певний товар за допомогою`productPrices`метод:

    use Laravel\Paddle\Cashier;

    // Retrieve prices for two products...
    $prices = Cashier::productPrices([123, 456]);

Валюта визначатиметься на основі IP-адреси запиту; однак за бажанням ви можете вказати конкретну країну для отримання цін на:

    use Laravel\Paddle\Cashier;

    // Retrieve prices for two products...
    $prices = Cashier::productPrices([123, 456], ['customer_country' => 'BE']);

Отримавши ціни, ви можете відображати їх як завгодно:

    <ul>
        @foreach ($prices as $price)
            <li>{{ $price->product_title }} - {{ $price->price()->gross() }}</li>
        @endforeach
    </ul>

Ви також можете відобразити ціну нетто (без податку) та окремо відобразити суму податку:

    <ul>
        @foreach ($prices as $price)
            <li>{{ $price->product_title }} - {{ $price->price()->net() }} (+ {{ $price->price()->tax() }} tax)</li>
        @endforeach
    </ul>

Якщо ви отримали ціни на тарифні плани, ви можете відобразити їх початкову та періодичну ціну окремо:

    <ul>
        @foreach ($prices as $price)
            <li>{{ $price->product_title }} - Initial: {{ $price->initialPrice()->gross() }} - Recurring: {{ $price->recurringPrice()->gross() }}</li>
        @endforeach
    </ul>

Для отримання додаткової інформації,[перевірте документацію API Paddle щодо цін](https://developer.paddle.com/api-reference/checkout-api/prices/getprices).

<a name="prices-customers"></a>

#### Клієнти

Якщо користувач вже є клієнтом, і ви хочете відобразити ціни, що стосуються цього клієнта, ви можете зробити це, отримавши ціни безпосередньо з екземпляра клієнта:

    use App\Models\User;

    // Retrieve prices for two products...
    $prices = User::find(1)->productPrices([123, 456]);

Внутрішньо Касир використовуватиме користувальницькі[`paddleCountry`метод](#customer-defaults)отримати ціни в їх валюті. Так, наприклад, користувач, який проживає в США, бачитиме ціни в доларах США, тоді як користувач у Бельгії бачитиме ціни в євро. Якщо не вдається знайти відповідну валюту, буде використана валюта за замовчуванням продукту. Ви можете налаштувати всі ціни на товар або план передплати на панелі керування Paddle.

<a name="prices-coupons"></a>

#### Купони

Ви також можете вибрати відображення цін після зниження купона. При дзвінку в`productPrices`методу, купони можуть передаватися у вигляді рядка, розділеного комами:

    use Laravel\Paddle\Cashier;

    $prices = Cashier::productPrices([123, 456], ['coupons' => 'SUMMERSALE,20PERCENTOFF']);

Потім відобразіть розраховані ціни за допомогою`price`метод:

    <ul>
        @foreach ($prices as $price)
            <li>{{ $price->product_title }} - {{ $price->price()->gross() }}</li>
        @endforeach
    </ul>

Ви можете відображати оригінальні перелічені ціни (без купонних знижок) за допомогою`listPrice`метод:

    <ul>
        @foreach ($prices as $price)
            <li>{{ $price->product_title }} - {{ $price->listPrice()->gross() }}</li>
        @endforeach
    </ul>

> {note} При використанні API цін, Paddle дозволяє застосовувати купони лише до продуктів одноразової покупки, а не до планів підписки.

<a name="customers"></a>

## Клієнти

<a name="customer-defaults"></a>

### Клієнт Defaults

Каса дозволяє встановити деякі корисні значення за замовчуванням для вашого клієнта при створенні посилань на оплату. Встановлення цих значень за замовчуванням дозволяє попередньо заповнити адресу електронної пошти, країну та поштовий індекс клієнта, щоб вони могли негайно перейти до платіжної частини віджета оформлення замовлення. Ви можете встановити ці значення за замовчуванням, замінивши наведені нижче способи для вашого платного користувача:

    /**
     * Get the customer's email address to associate with Paddle.
     *
     * @return string|null
     */
    public function paddleEmail()
    {
        return $this->email;
    }

    /**
     * Get the customer's country to associate with Paddle.
     *
     * This needs to be a 2 letter code. See the link below for supported countries.
     *
     * @return string|null
     * @link https://developer.paddle.com/reference/platform-parameters/supported-countries
     */
    public function paddleCountry()
    {
        //
    }

    /**
     * Get the customer's postcode to associate with Paddle.
     *
     * See the link below for countries which require this.
     *
     * @return string|null
     * @link https://developer.paddle.com/reference/platform-parameters/supported-countries#countries-requiring-postcode
     */
    public function paddlePostcode()
    {
        //
    }

Ці значення за замовчуванням будуть використовуватися для кожної дії в Касі, яка генерує a[посилання на оплату](#pay-links).

<a name="subscriptions"></a>

## Підписки

<a name="creating-subscriptions"></a>

### Створення підписки

Щоб створити передплату, спочатку отримайте екземпляр вашої платної моделі, який зазвичай буде екземпляром`App\Models\User`. Отримавши екземпляр моделі, ви можете використовувати`newSubscription`метод створення посилання на оплату підписки на модель:

    $user = User::find(1);

    $payLink = $user->newSubscription('default', $premium = 12345)
        ->returnTo(route('home'))
        ->create();

    return view('billing', ['payLink' => $payLink]);

Перший аргумент, переданий в`newSubscription`метод повинен бути назвою передплати. Якщо ваша програма пропонує лише одну передплату, ви можете зателефонувати за цим номером`default`або`primary`. Другим аргументом є конкретний план, на який користувач підписується. Це значення має відповідати ідентифікатору плану в Paddle.`returnTo`метод приймає URL-адресу, на яку ваш користувач буде перенаправлений після успішного завершення оформлення замовлення.

`create`метод створить посилання на оплату, яке можна використовувати для створення кнопки оплати. Кнопку оплати можна створити за допомогою`paddle-button`Компонент леза, який постачається з Cashier Paddle:

    <x-paddle-button :url="$payLink" class="px-8 py-4">
        Subscribe
    </x-paddle-button>

Після того, як користувач закінчив оплату, a`subscription_created`webhook буде відправлений з Paddle. Касир отримає цей веб-хук та налаштує підписку для вашого клієнта. Для того, щоб переконатися, що всі веб-хуки належним чином отримані та оброблені вашою програмою, переконайтеся, що у вас є належним чином[налаштування обробки веб-хука - -](#handling-paddle-webhooks).

<a name="additional-details"></a>

#### Додаткові деталі

Якщо ви хочете вказати додаткові дані про клієнта або передплату, ви можете зробити це, передавши їх як масив ключа / значення в`create`метод:

    $payLink = $user->newSubscription('default', $monthly = 12345)
        ->returnTo(route('home'))
        ->create([
            'vat_number' => $vatNumber,
        ]);

Щоб дізнатись більше про додаткові поля, які підтримує Paddle, перегляньте документацію Paddle на[генерування посилань на оплату](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink).

<a name="subscriptions-coupons"></a>

#### Купони

Якщо ви хочете застосувати купон під час створення підписки, ви можете використовувати`withCoupon`метод:

    $payLink = $user->newSubscription('default', $monthly = 12345)
        ->returnTo(route('home'))
        ->withCoupon('code')
        ->create();

<a name="metadata"></a>

#### Метадані

Ви також можете передавати масив метаданих за допомогою`withMetadata`метод:

    $payLink = $user->newSubscription('default', $monthly = 12345)
        ->returnTo(route('home'))
        ->withMetadata(['key' => 'value'])
        ->create();

> {note} При наданні метаданих уникайте їх використання`subscription_name`як ключ метаданих. Цей ключ зарезервований для внутрішнього використання Касиром.

<a name="checking-subscription-status"></a>

### Перевірка статусу підписки

Після того, як користувач підписався на вашу програму, ви можете перевірити стан її підписки за допомогою різноманітних зручних методів. По-перше,`subscribed`метод повертає`true`якщо у користувача активна підписка, навіть якщо підписка наразі перебуває у пробному періоді:

    if ($user->subscribed('default')) {
        //
    }

`subscribed`метод також робить чудовим кандидатом на[маршрут проміжного програмного забезпечення](/docs/{{version}}/middleware), що дозволяє фільтрувати доступ до маршрутів та контролерів на основі статусу передплати користувача:

    public function handle($request, Closure $next)
    {
        if ($request->user() && ! $request->user()->subscribed('default')) {
            // This user is not a paying customer...
            return redirect('billing');
        }

        return $next($request);
    }

Якщо ви хочете визначити, чи перебуває користувач все ще в пробному періоді, ви можете використовувати`onTrial`метод. Цей метод може бути корисним для відображення попередження користувачеві про те, що він все ще перебуває на пробному періоді:

    if ($user->subscription('default')->onTrial()) {
        //
    }

`subscribedToPlan`метод може бути використаний для визначення того, чи підписаний користувач на певний план на основі заданого ідентифікатора плану веслування. У цьому прикладі ми визначимо, чи користувач`default`передплата активно підписується на щомісячний план:

    if ($user->subscribedToPlan($monthly = 12345, 'default')) {
        //
    }

Передаючи масив до`subscribedToPlan`методом, ви можете визначити, чи користувач`default`передплата активно підписується на місячний або річний план:

    if ($user->subscribedToPlan([$monthly = 12345, $yearly = 54321], 'default')) {
        //
    }

`recurring`метод може бути використаний для визначення того, чи користувач підписаний на даний момент і чи більше він не перебуває у пробному періоді:

    if ($user->subscription('default')->recurring()) {
        //
    }

<a name="cancelled-subscription-status"></a>

#### Статус скасованої передплати

Щоб визначити, чи був користувач колись активним абонентом, але скасував підписку, ви можете скористатися`cancelled`метод:

    if ($user->subscription('default')->cancelled()) {
        //
    }

Ви також можете визначити, чи скасував користувач підписку, але все ще перебуває на "пільговому періоді", поки підписка повністю не закінчиться. Наприклад, якщо користувач скасовує підписку 5 березня, термін дії якої спочатку планувався до 10 березня, користувач перебуває у "пільговому періоді" до 10 березня. Зверніть увагу, що`subscribed`метод все ще повертається`true`протягом цього часу:

    if ($user->subscription('default')->onGracePeriod()) {
        //
    }

Щоб визначити, скасував користувач підписку і чи не перебуває він у межах "пільгового періоду", ви можете скористатися`ended`метод:

    if ($user->subscription('default')->ended()) {
        //
    }

<a name="subscription-scopes"></a>

#### Сфери передплати

Більшість станів передплати також доступні у вигляді обсягу запитів, так що ви можете легко запитувати базу даних щодо підписок, які перебувають у певному стані:

    // Get all active subscriptions...
    $subscriptions = Subscription::query()->active()->get();

    // Get all of the cancelled subscriptions for a user...
    $subscriptions = $user->subscriptions()->cancelled()->get();

Повний список доступних областей доступний нижче:

    Subscription::query()->active();
    Subscription::query()->onTrial();
    Subscription::query()->notOnTrial();
    Subscription::query()->pastDue();
    Subscription::query()->recurring();
    Subscription::query()->ended();
    Subscription::query()->paused();
    Subscription::query()->notPaused();
    Subscription::query()->onPausedGracePeriod();
    Subscription::query()->notOnPausedGracePeriod();
    Subscription::query()->cancelled();
    Subscription::query()->notCancelled();
    Subscription::query()->onGracePeriod();
    Subscription::query()->notOnGracePeriod();

<a name="past-due-status"></a>

#### Статус простроченого терміну

Якщо плата не вдається здійснити підписку, вона буде позначена як`past_due`. Коли ваша передплата знаходиться в такому стані, вона не буде активною, поки клієнт не оновить свою платіжну інформацію. Ви можете визначити, чи прострочена передплата за допомогою`pastDue`в екземплярі передплати:

    if ($user->subscription('default')->pastDue()) {
        //
    }

Коли термін підписки прострочений, ви повинні доручити користувачеві[оновити свої платіжні дані](#updating-payment-information). Ви можете налаштувати спосіб обробки прострочених підписок у вашому[Налаштування підписки на Paddle](https://vendors.paddle.com/subscription-settings).

Якщо ви хочете, щоб підписки все ще вважалися активними, коли вони є`past_due`, ви можете використовувати`keepPastDueSubscriptionsActive`спосіб, наданий Касиром. Як правило, цей метод слід викликати в`register`метод вашого`AppServiceProvider`:

    use Laravel\Paddle\Cashier;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        Cashier::keepPastDueSubscriptionsActive();
    }

> {note} Коли підписка знаходиться в`past_due`заявіть, що його не можна змінити, поки не буде оновлено платіжну інформацію. Тому`swap`і`updateQuantity`методи видадуть виняток, коли передплата знаходиться в`past_due`держава.

<a name="subscription-single-charges"></a>

### Одноразова плата за передплату

Одноразова плата за передплату дозволяє платити передплатникам одноразовою оплатою поверх їхніх підписок:

    $response = $user->subscription('default')->charge(12.99, 'Support Add-on');

На відміну від[одноразові заряди](#single-charges), цей спосіб негайно стягуватиме з клієнта збережений спосіб оплати за підписку. Сума плати завжди вказана у валюті, для якої встановлено підписку.

<a name="updating-payment-information"></a>

### Оновлення платіжної інформації

Paddle завжди зберігає спосіб оплати за підписку. Якщо ви хочете оновити спосіб оплати за замовчуванням для передплати, спочатку слід створити "URL-адресу оновлення" передплати, використовуючи`updateUrl`метод на моделі підписки:

    $user = App\Models\User::find(1);

    $updateUrl = $user->subscription('default')->updateUrl();

Потім ви можете використовувати створену URL-адресу в поєднанні з наданою касою`paddle-button`Компонент Blade, що дозволяє користувачеві ініціювати віджет Paddle та оновлювати свою платіжну інформацію:

    <x-paddle-button :url="$updateUrl" class="px-8 py-4">
        Update Card
    </x-paddle-button>

Коли користувач закінчить оновлення своєї інформації, a`subscription_updated`webhook буде надіслано компанією Paddle, а деталі підписки будуть оновлені у базі даних вашої програми.

<a name="changing-plans"></a>

### Зміна планів

Після того, як користувач підписався на вашу програму, він іноді може захотіти перейти на новий план підписки. Щоб замінити користувача на нову підписку, вам слід передати ідентифікатор плану Paddle на підписку`swap`метод:

    $user = App\Models\User::find(1);

    $user->subscription('default')->swap($premium = 34567);

Якщо користувач пробується, пробний термін буде збережений. Крім того, якщо для передплати існує "кількість", ця кількість також буде збережена.

Якщо ви хочете поміняти місцями та скасувати будь-який пробний період, який зараз перебуває користувач, ви можете використовувати`skipTrial`метод:

    $user->subscription('default')
            ->skipTrial()
            ->swap($premium = 34567);

Якщо ви хочете поміняти місцями та негайно виставити рахунок користувачеві, а не чекати наступного платіжного циклу, ви можете використовувати`swapAndInvoice`метод:

    $user = App\Models\User::find(1);

    $user->subscription('default')->swapAndInvoice($premium = 34567);

<a name="prorations"></a>

#### Пропорції

За замовчуванням Paddle пропорційно стягує плату під час обміну між планами.`noProrate`метод може бути використаний для оновлення передплати без пропорційних витрат:

    $user->subscription('default')->noProrate()->swap($premium = 34567);

<a name="subscription-quantity"></a>

### Кількість передплати

Іноді на підписки впливає "кількість". Наприклад, за вашу програму може стягуватися 10 доларів на місяць**на користувача**на рахунку. Щоб легко збільшити або зменшити кількість передплати, використовуйте`incrementQuantity`і`decrementQuantity`методи:

    $user = User::find(1);

    $user->subscription('default')->incrementQuantity();

    // Add five to the subscription's current quantity...
    $user->subscription('default')->incrementQuantity(5);

    $user->subscription('default')->decrementQuantity();

    // Subtract five to the subscription's current quantity...
    $user->subscription('default')->decrementQuantity(5);

Крім того, ви можете встановити певну кількість за допомогою`updateQuantity`метод:

    $user->subscription('default')->updateQuantity(10);

`noProrate`метод може бути використаний для оновлення кількості передплати без збільшення пропорцій:

    $user->subscription('default')->noProrate()->updateQuantity(10);

<a name="pausing-subscriptions"></a>
### Призупинення підписок

Щоб призупинити передплату, зателефонуйте на номер`pause`метод підписки користувача:

    $user->subscription('default')->pause();

Коли підписка призупинена, Касир автоматично встановлює`paused_from`у вашій базі даних. Цей стовпець використовується, щоб знати, коли`paused`метод повинен почати повертатися`true`. Наприклад, якщо клієнт призупинить підписку 1 березня, але підписка не планувалась повторити до 5 березня,`paused`метод продовжить повертатися`false`до 5 березня.

Ви можете визначити, чи користувач призупинив свою підписку, але все ще перебуває на "пільговому періоді", використовуючи`onPausedGracePeriod`метод:

    if ($user->subscription('default')->onPausedGracePeriod()) {
        //
    }

Щоб відновити призупинену підписку, ви можете зателефонувати до`unpause`метод підписки користувача:

    $user->subscription('default')->unpause();

> {note} Підписку не можна змінювати, поки вона призупинена. Якщо ви хочете перейти на інший план або оновити кількість, спочатку слід відновити підписку.

<a name="cancelling-subscriptions"></a>

### Скасування передплати

Щоб скасувати передплату, зателефонуйте на номер`cancel`метод підписки користувача:

    $user->subscription('default')->cancel();

Коли підписку скасовано, Касир автоматично встановить`ends_at`у вашій базі даних. Цей стовпець використовується, щоб знати, коли`subscribed`метод повинен почати повертатися`false`. Наприклад, якщо клієнт скасовує підписку 1 березня, але передплата закінчувалася до 5 березня,`subscribed`метод продовжить повертатися`true`до 5 березня.

Ви можете визначити, скасував користувач підписку, але все ще перебуває на "пільговому періоді", використовуючи`onGracePeriod`метод:

    if ($user->subscription('default')->onGracePeriod()) {
        //
    }

Якщо ви бажаєте негайно скасувати підписку, ви можете зателефонувати до`cancelNow`метод підписки користувача:

    $user->subscription('default')->cancelNow();

> {note} Підписка на Paddle не може бути відновлена ​​після скасування. Якщо ваш клієнт бажає відновити підписку, йому доведеться підписатися на нову підписку.

<a name="subscription-trials"></a>

## Випробувальні підписки

<a name="with-payment-method-up-front"></a>

### Зі способом оплати вперед

> {note} Під час випробування та збору деталей способу оплати заздалегідь Paddle запобігає будь-яким змінам передплати, таким як обмін планами або оновлення кількості. Якщо ви хочете дозволити клієнту обмінятися планами під час пробного періоду, передплату потрібно скасувати та відновити.

Якщо ви хочете запропонувати пробні періоди своїм клієнтам, збираючи інформацію про спосіб оплати заздалегідь, вам слід використовувати`trialDays`спосіб створення підписки на оплату посилань:

    $user = User::find(1);

    $payLink = $user->newSubscription('default', $monthly = 12345)
                ->returnTo(route('home'))
                ->trialDays(10)
                ->create();

    return view('billing', ['payLink' => $payLink]);

Цей метод встановить дату закінчення пробного періоду в записі підписки в базі даних, а також вкаже Paddle не розпочинати виставлення рахунків клієнту до цієї дати.

> {note} Якщо передплата клієнта не буде скасована до дати закінчення пробного періоду, з них буде стягнуто плату, як тільки закінчиться пробна версія, тому ви повинні обов’язково повідомити своїх користувачів про дату закінчення пробного періоду.

Ви можете визначити, чи перебуває користувач протягом пробного періоду, використовуючи будь-який із`onTrial`метод екземпляра користувача або`onTrial`метод екземпляра передплати. Два наведені нижче приклади мають однакову поведінку:

    if ($user->onTrial('default')) {
        //
    }

    if ($user->subscription('default')->onTrial()) {
        //
    }

<a name="defining-trial-days-in-paddle-cashier"></a>

#### Визначення судових днів у веслуванні / касі

You may choose to define how many trial days your plan's receive in the Paddle dashboard or always pass them explicitly using Cashier. If you choose to define your plan's trial days in Paddle you should be aware that new subscriptions, including new subscriptions for a customer that had a subscription in the past, will always receive a trial period unless you explicitly call the `trialDays(0)`метод.

<a name="without-payment-method-up-front"></a>

### Без способу оплати

Якщо ви хочете запропонувати пробні періоди без попереднього збору інформації про спосіб оплати користувача, ви можете встановити`trial_ends_at`стовпець у записі клієнта, прикріплений до вашого користувача до бажаної дати закінчення пробного періоду. Зазвичай це робиться під час реєстрації користувача:

    $user = User::create([
        // Other user properties...
    ]);

    $user->createAsCustomer([
        'trial_ends_at' => now()->addDays(10)
    ]);

Касир називає цей тип судового розгляду «загальним випробуванням», оскільки він не додається до жодної існуючої передплати.`onTrial`метод на`User`екземпляр повернеться`true`якщо поточна дата не минула значення`trial_ends_at`:

    if ($user->onTrial()) {
        // User is within their trial period...
    }

Коли ви будете готові створити фактичну передплату для користувача, ви можете використовувати`newSubscription`метод, як зазвичай:

    $user = User::find(1);

    $payLink = $user->newSubscription('default', $monthly = 12345)
        ->returnTo(route('home'))
        ->create();

Щоб отримати дату завершення пробного періоду для користувача, ви можете використовувати`trialEndsAt`метод. Цей метод повертає екземпляр дати Carbon, якщо користувач пробний або`null`якщо їх немає. Ви також можете передати необов’язковий параметр імені підписки, якщо хочете отримати дату закінчення пробного періоду для певної підписки, крім стандартної:

    if ($user->onTrial()) {
        $trialEndsAt = $user->trialEndsAt('main');
    }

Ви також можете використовувати`onGenericTrial`метод, якщо ви хочете конкретно знати, що користувач перебуває у межах "загального" пробного періоду і ще не створив фактичну передплату:

    if ($user->onGenericTrial()) {
        // User is within their "generic" trial period...
    }

> {note} Неможливо продовжити або змінити пробний період підписки на Paddle після її створення.

<a name="handling-paddle-webhooks"></a>

## Обробка Paddleвих веб-хуків

> {tip} Ви можете використовувати[Валетт`share`команди](https://laravel.com/docs/{{version}}/valet#sharing-sites)для тестування веб-хуків під час локальної розробки.

Paddle може повідомляти вашу заявку про різноманітні події через веб-хуки. За замовчуванням маршрут, який вказує на контролер веб-хука Каси, налаштовується через постачальника послуг Каси. Цей контролер обробляє всі вхідні запити веб-хука.

За замовчуванням цей контролер буде автоматично обробляти скасування підписок, у яких занадто багато невдалих платежів ([як визначено в налаштуваннях підписки на Paddle](https://vendors.paddle.com/subscription-settings)), оновлення передплати та зміни способу оплати; однак, як ми скоро дізнаємось, ви можете розширити цей контролер для обробки будь-якої події веб-хука, яка вам подобається.

Щоб ваша програма могла обробляти веб-хуки Paddle, обов’язково[налаштуйте URL-адресу веб-хука на панелі управління Paddle](https://vendors.paddle.com/alerts-webhooks). За замовчуванням контролер веб-хука Касир прослуховує`/paddle/webhook`Шлях URL. Повний список усіх веб-хуків, які слід налаштувати на панелі керування Paddle:

-   Підписка створена
-   Підписка оновлена
-   Підписку скасовано
-   Оплата виконана
-   Оплата передплати виконана

> {note} Обов’язково захистіть вхідні запити за допомогою каси[перевірка підпису](/docs/{{version}}/cashier-paddle#verifying-webhook-signatures)проміжне програмне забезпечення.

<a name="webhooks-csrf-protection"></a>

#### Захист веб-хуків та CSRF

Оскільки веб-хуки Paddle повинні обходити Laravel[Захист CSRF](/docs/{{version}}/csrf), не забудьте вказати URI як виняток у вашому`VerifyCsrfToken`проміжне програмне забезпечення або перелічіть маршрут за межами`web`група проміжного програмного забезпечення:

    protected $except = [
        'paddle/*',
    ];

<a name="defining-webhook-event-handlers"></a>

### Визначення обробників подій Webhook

Каса автоматично обробляє скасування передплати за невдалі платежі, але якщо у вас є додаткові події веб-хука, які ви хотіли б обробити, вам слід продовжити`WebhookController`. Назви ваших методів повинні відповідати очікуваним умовам Касира, зокрема, методи повинні мати префікс`handle`і назва "справи верблюда" веб-гачка, з яким ви хочете обробити. Наприклад, якщо ви хочете обробляти файл`payment_succeeded`webhook, вам слід додати a`handlePaymentSucceeded`метод до контролера:

    <?php

    namespace App\Http\Controllers;

    use Laravel\Paddle\Http\Controllers\WebhookController as CashierController;

    class WebhookController extends CashierController
    {
        /**
         * Handle payment succeeded.
         *
         * @param  array  $payload
         * @return void
         */
        public function handlePaymentSucceeded($payload)
        {
            // Handle The Event
        }
    }

Далі визначте маршрут до свого контролера Каси в межах вашого`routes/web.php`файл. Це перезапише маршрут, що входить до Каси:

    use App\Http\Controllers\WebhookController;

    Route::post('paddle/webhook', WebhookController::class);

Касир виділяє`Laravel\Paddle\Events\WebhookReceived`подія, коли отримано веб-хук і`Laravel\Paddle\Events\WebhookHandled`подія, коли оброблявся веб-хук. Обидві події містять повне навантаження веб-хука Paddle.

Касир також проводить події, присвячені типу отриманого веб-хука. На додаток до повного корисного навантаження від Paddle, вони також містять відповідні моделі, які використовувались для обробки веб-хука, такі як платна модель, передплата або квитанція:

<div class="content-list" markdown="1">
- `PaymentSucceeded`
- `SubscriptionPaymentSucceeded`
- `SubscriptionCreated`
- `SubscriptionUpdated`
- `SubscriptionCancelled`
</div>

Ви також можете замінити вбудований маршрут веб-хука за замовчуванням, встановивши`CASHIER_WEBHOOK`змінну env у вашому`.env`файл. Це значення має бути повною URL-адресою маршруту вашого веб-хука і має відповідати URL-адресі, встановленій на панелі керування Paddle:

    CASHIER_WEBHOOK=https://example.com/my-paddle-webhook-url

<a name="handling-failed-subscriptions"></a>

### Невдалі підписки

Що робити, якщо термін дії кредитної картки клієнта закінчується? Не хвилюйтеся - контролер Webhook каси скасує передплату клієнта для вас. Невдалі платежі будуть автоматично зафіксовані та оброблені контролером. Контролер скасовує передплату клієнта, коли Paddle виявляє, що передплата не вдалася (зазвичай після трьох невдалих спроб оплати).

<a name="verifying-webhook-signatures"></a>

### Перевірка підписів Webhook

Для захисту веб-хуків ви можете використовувати[Підписи веб-гачка Весла](https://developer.paddle.com/webhook-reference/verifying-webhooks). Для зручності Cashier автоматично включає проміжне програмне забезпечення, яке підтверджує, що вхідний запит веб-хука Paddle є дійсним.

Щоб увімкнути перевірку веб-хука, переконайтеся, що`PADDLE_PUBLIC_KEY`змінна середовища встановлена ​​у вашому`.env`файл. Відкритий ключ можна отримати з інформаційної панелі вашого акаунта Paddle.

<a name="single-charges"></a>

## Одноразові збори

<a name="simple-charge"></a>

### Проста зарядка

Якщо ви хочете виставити клієнту одноразову оплату, ви можете використовувати`charge`метод на екземплярі платної моделі, щоб сформувати посилання на оплату за плату.`charge`метод приймає суму комісії (float) як перший аргумент, а опис комісії як другий аргумент:

    $payLink = $user->charge(12.99, 'Product Title');

    return view('pay', ['payLink' => $payLink]);

Після генерації посилання на оплату ви можете використовувати надану касу`paddle-button`Компонент леза, що дозволяє користувачеві ініціювати віджет Paddle і завершити зарядку:

    <x-paddle-button :url="$payLink" class="px-8 py-4">
        Buy
    </x-paddle-button>

`charge`Метод приймає масив як свій третій аргумент, дозволяючи вам передавати будь-які параметри, які ви хочете, для базового створення посилання на оплату Paddle. Будь ласка, проконсультуйтесь[документація на Paddle](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink)щоб дізнатись більше про варіанти, доступні вам під час створення нарахувань:

    $payLink = $user->charge(12.99, 'Product Title', [
        'custom_option' => $value,
    ]);

Плата стягується у валюті, зазначеній у`cashier.currency`варіант конфігурації. За замовчуванням для цього значення встановлено USD. Ви можете замінити валюту за замовчуванням, встановивши`CASHIER_CURRENCY`у вашому`.env`файл:

    CASHIER_CURRENCY=EUR

Ви також можете[замінити ціни на валюту](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink#price-overrides)за допомогою динамічної системи відповідності цін на Paddle. Для цього передайте масив цін замість фіксованої суми:

    $payLink = $user->charge([
        'USD:19.99',
        'EUR:15.99',
    ], 'Product Title');

<a name="charging-products"></a>

### Зарядні вироби

Якщо ви хочете зробити одноразову плату за певний продукт, налаштований в Paddle, ви можете використовувати`chargeProduct`метод на екземплярі платної моделі для формування посилання на оплату:

    $payLink = $user->chargeProduct($productId);

    return view('pay', ['payLink' => $payLink]);

Потім ви можете надати посилання на оплату`paddle-button`компонент, що дозволяє користувачеві ініціалізувати віджет Paddle:

    <x-paddle-button :url="$payLink" class="px-8 py-4">
        Buy
    </x-paddle-button>

`chargeProduct`Метод приймає масив як другий аргумент, дозволяючи передавати будь-які параметри, які ви хочете, для базового створення посилання на оплату за допомогою Paddle. Будь ласка, проконсультуйтесь[документація на Paddle](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink)щодо варіантів, доступних вам під час стягнення плати:

    $payLink = $user->chargeProduct($productId, [
        'custom_option' => $value,
    ]);

<a name="refunding-orders"></a>

### Повернення коштів за замовлення

Якщо вам потрібно повернути гроші за замовлення Paddle, ви можете скористатися`refund`метод. Цей метод приймає ідентифікатор замовлення весла як перший аргумент. Ви можете отримати квитанції для даної платної організації, використовуючи`receipts`метод:

    $receipt = $user->receipts()->first();

    $refundRequestId = $user->refund($receipt->order_id);

Ви також можете за бажанням вказати конкретну суму для відшкодування, а також причину відшкодування:

    $receipt = $user->receipts()->first();

    $refundRequestId = $user->refund(
        $receipt->order_id, 5.00, 'Unused product time'
    );

> {tip} Ви можете використовувати`$refundRequestId`як посилання на повернення коштів при зверненні до служби підтримки Paddle.

<a name="receipts"></a>

## Квитанції

Ви можете легко отримати масив квитанцій оплачуваної моделі за допомогою`receipts`метод:

    $receipts = $user->receipts();

Перераховуючи квитанції для клієнта, ви можете використовувати допоміжні методи квитанції для відображення відповідної інформації про квитанцію. Наприклад, ви можете перерахувати кожну квитанцію в таблиці, що дозволить користувачеві легко завантажити будь-яку квитанцію:

    <table>
        @foreach ($receipts as $receipt)
            <tr>
                <td>{{ $receipt->paid_at->toFormattedDateString() }}</td>
                <td>{{ $receipt->amount() }}</td>
                <td><a href="{{ $receipt->receipt_url }}" target="_blank">Download</a></td>
            </tr>
        @endforeach
    </table>

<a name="past-and-upcoming-payments"></a>

### Минулі та майбутні платежі

Ви можете використовувати`lastPayment`і`nextPayment`методи відображення минулих або майбутніх платежів клієнта за періодичні підписки:

    $subscription = $user->subscription('default');

    $lastPayment = $subscription->lastPayment();
    $nextPayment = $subscription->nextPayment();

Обидва ці методи повернуть екземпляр`Laravel\Paddle\Payment`; однак,`nextPayment`повернеться`null`коли платіжний цикл закінчився (наприклад, коли скасовано підписку):

    Next payment: {{ $nextPayment->amount() }} due on {{ $nextPayment->date()->format('d/m/Y') }}

<a name="handling-failed-payments"></a>

## Обробка невдалих платежів

Платежі підписки зазнають невдачі з різних причин, таких як картки, термін дії яких закінчився, або картка, у якої недостатньо коштів. Коли це трапляється, ми рекомендуємо дозволити Paddle самостійно обробляти невдалі платежі. Зокрема, ви можете[налаштувати автоматичні електронні листи на рахунки Paddle](https://vendors.paddle.com/subscription-settings)на панелі приладів Paddle.

Крім того, ви можете виконати більш точну настройку, вловлюючи[`subscription_payment_failed`](https://developer.paddle.com/webhook-reference/subscription-alerts/subscription-payment-failed)webhook і ввімкнення опції "Не вдалося здійснити оплату передплати" в налаштуваннях Webhook на інформаційній панелі Paddle:

    <?php

    namespace App\Http\Controllers;

    use Laravel\Paddle\Http\Controllers\WebhookController as CashierController;

    class WebhookController extends CashierController
    {
        /**
         * Handle subscription payment failed.
         *
         * @param  array  $payload
         * @return void
         */
        public function handleSubscriptionPaymentFailed($payload)
        {
            // Handle the failed subscription payment...
        }
    }

<a name="testing"></a>

## Тестування

На даний момент у Paddle відсутній належний CRUD API, тому вам доведеться вручну протестувати свій платіжний потік. Paddle також не має середовища розробника, тому будь-які платежі, які ви робите, є реальними. Для того, щоб обійти цю проблему, ми рекомендуємо використовувати купони зі 100% знижкою або безкоштовні товари під час тестування.
