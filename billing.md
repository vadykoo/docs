# Каса Laravel

-   [Вступ](#introduction)
-   [Оновлення каси](#upgrading-cashier)
-   [Встановлення](#installation)
-   [Конфігурація](#configuration)
    -   [Billable Model](#billable-model)
    -   [Ключі API](#api-keys)
    -   [Конфігурація валюти](#currency-configuration)
    -   [Лісозаготівля](#logging)
-   [Клієнти](#customers)
    -   [Отримання клієнтів](#retrieving-customers)
    -   [Створення клієнтів](#creating-customers)
    -   [Оновлення клієнтів](#updating-customers)
    -   [Портал виставлення рахунків](#billing-portal)
-   [методи оплати](#payment-methods)
    -   [Зберігання способів оплати](#storing-payment-methods)
    -   [Отримання способів оплати](#retrieving-payment-methods)
    -   [Визначення, чи має користувач спосіб оплати](#check-for-a-payment-method)
    -   [Оновлення способу оплати за замовчуванням](#updating-the-default-payment-method)
    -   [Додавання способів оплати](#adding-payment-methods)
    -   [Видалення способів оплати](#deleting-payment-methods)
-   [Підписки](#subscriptions)
    -   [Створення підписки](#creating-subscriptions)
    -   [Перевірка статусу передплати](#checking-subscription-status)
    -   [Зміна планів](#changing-plans)
    -   [Кількість передплати](#subscription-quantity)
    -   [Підписки на кілька планів](#multiplan-subscriptions)
    -   [Податки на підписку](#subscription-taxes)
    -   [Дата прив’язки підписки](#subscription-anchor-date)
    -   [Скасування передплати](#cancelling-subscriptions)
    -   [Відновлення передплати](#resuming-subscriptions)
-   [Випробувальні підписки](#subscription-trials)
    -   [Зі способом оплати вперед](#with-payment-method-up-front)
    -   [Без способу оплати](#without-payment-method-up-front)
    -   [Розширення випробувань](#extending-trials)
-   [Обробка смугових веб-хуків](#handling-stripe-webhooks)
    -   [Визначення обробників подій Webhook](#defining-webhook-event-handlers)
    -   [Невдалі підписки](#handling-failed-subscriptions)
    -   [Перевірка підписів Webhook](#verifying-webhook-signatures)
-   [Одноразові збори](#single-charges)
    -   [Проста зарядка](#simple-charge)
    -   [Стягніть рахунок-фактуру](#charge-with-invoice)
    -   [Повернення коштів](#refunding-charges)
-   [Рахунки-фактури](#invoices)
    -   [Отримання рахунків-фактур](#retrieving-invoices)
    -   [Створення PDF-фактур](#generating-invoice-pdfs)
-   [Обробка невдалих платежів](#handling-failed-payments)
-   [Сильна автентифікація клієнта (SCA)](#strong-customer-authentication)
    -   [Платежі, що вимагають додаткового підтвердження](#payments-requiring-additional-confirmation)
    -   [Повідомлення про оплату поза сесією](#off-session-payment-notifications)
-   [Смугастий SDK](#stripe-sdk)
-   [Тестування](#testing)

<a name="introduction"></a>

## Вступ

Laravel Cashier надає виразний, вільний інтерфейс для[Смугастий](https://stripe.com)послуги оплати передплати. Він обробляє майже весь звітний код передплати, перед яким ви боїтеся писати. На додаток до основного управління передплатою, Касир може обробляти купони, обмінюватися передплатою, «кількістю» передплати, пільговими періодами скасування та навіть генерувати PDF-файли рахунків-фактур.

<a name="upgrading-cashier"></a>

## Оновлення каси

Під час оновлення до нової версії Каси важливо уважно переглянути[посібник з оновлення](https://github.com/laravel/cashier-stripe/blob/master/UPGRADE.md).

> {note} Щоб запобігти порушенням змін, Cashier використовує фіксовану версію API Stripe. Cashier 12 використовує версію API Stripe`2020-03-02`. Версія API Stripe буде оновлюватися в незначних версіях, щоб використовувати нові функції та вдосконалення Stripe.

<a name="installation"></a>

## Встановлення

По-перше, вимагайте пакет Cashier для Stripe with Composer:

    composer require laravel/cashier

> {note} Щоб касир правильно обробляв усі події Stripe, пам’ятайте[налаштувати обробку веб-хука Касира](#handling-stripe-webhooks).

<a name="database-migrations"></a>

#### Міграції баз даних

Постачальник послуг каси реєструє власний каталог міграції бази даних, тому не забудьте перенести вашу базу даних після встановлення пакету. Міграція каси додасть кілька стовпців до вашого`users`таблиці, а також створити нову`subscriptions`таблиця, щоб вмістити всі передплати вашого клієнта:

    php artisan migrate

Якщо вам потрібно переписати міграції, які постачаються з пакетом Cashier, ви можете опублікувати їх за допомогою`vendor:publish`Команда ремісників:

    php artisan vendor:publish --tag="cashier-migrations"

Якщо ви хочете повністю запобігти запуску міграції Каси, ви можете скористатися`ignoreMigrations`надані Касиром. Як правило, цей метод слід викликати в`register`метод вашого`AppServiceProvider`:

    use Laravel\Cashier\Cashier;

    Cashier::ignoreMigrations();

> {note} Stripe рекомендує, щоб будь-який стовпець, що використовується для зберігання ідентифікаторів Stripe, був чутливим до регістру. Тому слід забезпечити сортування стовпців для`stripe_id`для стовпця встановлено, наприклад,`utf8_bin`в MySQL. Більше інформації можна знайти[у документації Stripe](https://stripe.com/docs/upgrades#what-changes-does-stripe-consider-to-be-backwards-compatible).

<a name="configuration"></a>

## Конфігурація

<a name="billable-model"></a>

### Модель, що підлягає оплаті

Перед використанням Каси додайте`Billable`риса для визначення вашої моделі. Ця ознака пропонує різні методи, що дозволяють виконувати загальні завдання щодо виставлення рахунків, такі як створення підписок, застосування купонів та оновлення інформації про спосіб оплати:

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

Касир припускає, що вашою платною моделлю буде`App\Models\User` class that ships with Laravel. If you wish to change this you can specify a different model in your `.env`файл:

    CASHIER_MODEL=App\Models\User

> {note} Якщо ви використовуєте модель, відмінну від поставленої Laravel`App\Models\User`модель, вам потрібно буде опублікувати та змінити файл[міграції](#installation)надається відповідно до назви таблиці вашої альтернативної моделі.

<a name="api-keys"></a>

### Ключі API

Далі слід налаштувати свої клавіші Stripe у вашому`.env`файл. Ключі API Stripe можна отримати на панелі керування Stripe.

    STRIPE_KEY=your-stripe-key
    STRIPE_SECRET=your-stripe-secret

<a name="currency-configuration"></a>

### Конфігурація валюти

Валютою каси за замовчуванням є долари США (USD). Ви можете змінити валюту за замовчуванням, встановивши`CASHIER_CURRENCY`змінна середовища:

    CASHIER_CURRENCY=eur

Окрім налаштування валюти Каси, Ви також можете вказати локаль, який використовуватиметься при форматуванні грошових значень для відображення на рахунках-фактурах. Внутрішньо Касир використовує[PHP`NumberFormatter`клас](https://www.php.net/manual/en/class.numberformatter.php)щоб встановити локаль валюти:

    CASHIER_CURRENCY_LOCALE=nl_BE

> {note} Для того, щоб використовувати локалі, крім`en`, забезпечити`ext-intl`Розширення PHP встановлено та налаштовано на вашому сервері.

<a name="logging"></a>

#### Лісозаготівля

Каса дозволяє вказати канал журналу, який використовуватиметься при реєстрації всіх винятків, пов’язаних із Stripe. Ви можете вказати канал журналу за допомогою`CASHIER_LOGGER`змінна середовища:

    CASHIER_LOGGER=stack

<a name="customers"></a>

## Клієнти

<a name="retrieving-customers"></a>

### Отримання клієнтів

Ви можете отримати клієнта за його ідентифікатором Stripe, використовуючи`Cashier::findBillable`метод. Це поверне екземпляр моделі Billable:

    use Laravel\Cashier\Cashier;

    $user = Cashier::findBillable($stripeId);

<a name="creating-customers"></a>

### Створення клієнтів

Іноді, можливо, ви захочете створити клієнта Stripe, не починаючи передплату. Ви можете досягти цього за допомогою`createAsStripeCustomer`метод:

    $stripeCustomer = $user->createAsStripeCustomer();

Після створення клієнта в Stripe ви можете розпочати підписку пізніше. Ви також можете використовувати необов’язковий`$options`масив для передачі будь-яких додаткових параметрів, які підтримуються Stripe API:

    $stripeCustomer = $user->createAsStripeCustomer($options);

Ви можете використовувати`asStripeCustomer`метод, якщо ви хочете повернути об'єкт клієнта, якщо платний об'єкт вже є клієнтом у Stripe:

    $stripeCustomer = $user->asStripeCustomer();

`createOrGetStripeCustomer`метод може бути використаний, якщо ви хочете повернути об'єкт замовника, але не впевнені, чи платник є вже клієнтом у Stripe. Цей метод створить нового клієнта в Stripe, якщо такий ще не існує:

    $stripeCustomer = $user->createOrGetStripeCustomer();

<a name="updating-customers"></a>

### Оновлення клієнтів

Іноді, можливо, ви захочете оновити клієнта Stripe безпосередньо додатковою інформацією. Ви можете досягти цього за допомогою`updateStripeCustomer`метод:

    $stripeCustomer = $user->updateStripeCustomer($options);

<a name="billing-portal"></a>

### Портал виставлення рахунків

Смугасті пропозиції[простий спосіб створити платіжний портал](https://stripe.com/docs/billing/subscriptions/customer-portal)так що ваш клієнт може керувати підпискою, способами оплати та переглядати історію виставлення рахунків. Ви можете перенаправити своїх користувачів на платіжний портал за допомогою`redirectToBillingPortal`метод від контролера або маршруту:

    use Illuminate\Http\Request;

    public function billingPortal(Request $request)
    {
        return $request->user()->redirectToBillingPortal();
    }

За замовчуванням, коли користувач закінчує керувати підпискою, він може повернутися до`home`маршрут вашої заявки. Ви можете надати користувацьку URL-адресу, до якої користувач повинен повернутися, передавши URL-адресу як аргумент для`redirectToBillingPortal`метод:

    use Illuminate\Http\Request;

    public function billingPortal(Request $request)
    {
        return $request->user()->redirectToBillingPortal(
            route('billing')
        );
    }

Якщо ви хочете згенерувати лише URL-адресу платіжного порталу, ви можете скористатися`billingPortalUrl`метод:

    $url = $user->billingPortalUrl(route('billing'));

<a name="payment-methods"></a>

## методи оплати

<a name="storing-payment-methods"></a>

### Зберігання способів оплати

Для того, щоб створити передплату або здійснити одноразові платежі за Stripe, вам потрібно буде зберегти спосіб оплати та отримати його ідентифікатор із Stripe. Підхід, який використовується для досягнення, відрізняється залежно від того, чи плануєте ви використовувати спосіб оплати для підписки або одноразової оплати, тому ми розглянемо обидва нижче.

<a name="payment-methods-for-subscriptions"></a>

#### Способи оплати передплати

При зберіганні кредитних карток у клієнта для подальшого використання необхідно використовувати API Stripe Setup Intents для надійного збору інформації про спосіб оплати клієнта. "Намір налаштування" означає, що Stripe має намір стягнути плату з способу оплати клієнта. Касирський`Billable`риса включає в себе`createSetupIntent`щоб легко створити новий намір налаштування. Вам слід зателефонувати за цим способом із маршруту або контролера, який надасть форму, яка збирає деталі способу оплати вашого клієнта:

    return view('update-payment-method', [
        'intent' => $user->createSetupIntent()
    ]);

Після того, як ви створили намір налаштування та передали його представленням, вам слід прикріпити його секрет до елемента, який збиратиме спосіб оплати. Наприклад, розгляньте цю форму "оновити спосіб оплати":

    <input id="card-holder-name" type="text">

    <!-- Stripe Elements Placeholder -->
    <div id="card-element"></div>

    <button id="card-button" data-secret="{{ $intent->client_secret }}">
        Update Payment Method
    </button>

Далі, бібліотека Stripe.js може бути використана для приєднання елемента смужки до форми та надійного збору платіжних даних клієнта:

    <script src="https://js.stripe.com/v3/"></script>

    <script>
        const stripe = Stripe('stripe-public-key');

        const elements = stripe.elements();
        const cardElement = elements.create('card');

        cardElement.mount('#card-element');
    </script>

Далі картку можна перевірити та отримати захищений «ідентифікатор способу оплати» у Stripe за допомогою[Смугастий`confirmCardSetup`метод](https://stripe.com/docs/js/setup_intents/confirm_card_setup):

    const cardHolderName = document.getElementById('card-holder-name');
    const cardButton = document.getElementById('card-button');
    const clientSecret = cardButton.dataset.secret;

    cardButton.addEventListener('click', async (e) => {
        const { setupIntent, error } = await stripe.confirmCardSetup(
            clientSecret, {
                payment_method: {
                    card: cardElement,
                    billing_details: { name: cardHolderName.value }
                }
            }
        );

        if (error) {
            // Display "error.message" to the user...
        } else {
            // The card has been verified successfully...
        }
    });

Після перевірки картки Stripe ви можете передати отриману`setupIntent.payment_method`ідентифікатор вашої програми Laravel, де його можна прикріпити до замовника. Спосіб оплати може бути будь-яким[додано як новий спосіб оплати](#adding-payment-methods)або[використовується для оновлення способу оплати за замовчуванням](#updating-the-default-payment-method). Ви також можете негайно використовувати ідентифікатор способу оплати для[створити нову передплату](#creating-subscriptions).

> {tip} Якщо вам потрібна додаткова інформація про наміри налаштування та збір даних про оплату клієнтів, будь ласка[перегляньте цей огляд, наданий Stripe](https://stripe.com/docs/payments/save-and-reuse#php).

<a name="payment-methods-for-single-charges"></a>

#### Способи оплати одноразових платежів

Звичайно, під час одноразового стягнення платіжного способу клієнта нам потрібно буде використовувати ідентифікатор способу оплати лише один раз. Через обмеження Stripe ви не можете використовувати збережений спосіб оплати клієнта за замовчуванням для одноразових платежів. Ви повинні дозволити клієнту вводити дані свого способу оплати за допомогою бібліотеки Stripe.js. Наприклад, розглянемо таку форму:

    <input id="card-holder-name" type="text">

    <!-- Stripe Elements Placeholder -->
    <div id="card-element"></div>

    <button id="card-button">
        Process Payment
    </button>

Далі, бібліотека Stripe.js може бути використана для приєднання елемента смужки до форми та надійного збору платіжних даних клієнта:

    <script src="https://js.stripe.com/v3/"></script>

    <script>
        const stripe = Stripe('stripe-public-key');

        const elements = stripe.elements();
        const cardElement = elements.create('card');

        cardElement.mount('#card-element');
    </script>

Далі картку можна перевірити та отримати захищений «ідентифікатор способу оплати» у Stripe за допомогою[Смугастий`createPaymentMethod`метод](https://stripe.com/docs/stripe-js/reference#stripe-create-payment-method):

    const cardHolderName = document.getElementById('card-holder-name');
    const cardButton = document.getElementById('card-button');

    cardButton.addEventListener('click', async (e) => {
        const { paymentMethod, error } = await stripe.createPaymentMethod(
            'card', cardElement, {
                billing_details: { name: cardHolderName.value }
            }
        );

        if (error) {
            // Display "error.message" to the user...
        } else {
            // The card has been verified successfully...
        }
    });

Якщо картку успішно перевірено, ви можете передати`paymentMethod.id`до вашої заявки Laravel та обробіть a[одна зарядка](#simple-charge).

<a name="retrieving-payment-methods"></a>

### Отримання способів оплати

`paymentMethods`метод на екземплярі моделі Billable повертає колекцію`Laravel\Cashier\PaymentMethod`екземпляри:

    $paymentMethods = $user->paymentMethods();

Щоб отримати спосіб оплати за замовчуванням,`defaultPaymentMethod`може бути використаний метод:

    $paymentMethod = $user->defaultPaymentMethod();

Ви також можете отримати певний спосіб оплати, який належить моделі Billable, використовуючи`findPaymentMethod`метод:

    $paymentMethod = $user->findPaymentMethod($paymentMethodId);

<a name="check-for-a-payment-method"></a>

### Визначення, чи має користувач спосіб оплати

Щоб визначити, чи має модель, що підлягає оплаті, метод оплати за замовчуванням, прикріплений до її рахунку, за допомогою`hasDefaultPaymentMethod`метод:

    if ($user->hasDefaultPaymentMethod()) {
        //
    }

Щоб визначити, чи має модель, яка підлягає оплаті, принаймні один спосіб оплати, прикріплений до її рахунку, за допомогою`hasPaymentMethod`метод:

    if ($user->hasPaymentMethod()) {
        //
    }

<a name="updating-the-default-payment-method"></a>

### Оновлення способу оплати за замовчуванням

`updateDefaultPaymentMethod`метод може бути використаний для оновлення інформації про спосіб оплати за замовчуванням. Цей метод приймає ідентифікатор способу оплати Stripe та призначає новий спосіб оплати як спосіб оплати за замовчуванням за замовчуванням:

    $user->updateDefaultPaymentMethod($paymentMethod);

Щоб синхронізувати інформацію про спосіб оплати за замовчуванням із інформацією про спосіб оплати за замовчуванням у Stripe, ви можете використовувати`updateDefaultPaymentMethodFromStripe`метод:

    $user->updateDefaultPaymentMethodFromStripe();

> {note} Спосіб оплати за замовчуванням для клієнта може використовуватися лише для виставлення рахунків та створення нових підписок. Через обмеження Stripe він не може використовуватися для одноразового заряджання.

<a name="adding-payment-methods"></a>

### Додавання способів оплати

Щоб додати новий спосіб оплати, ви можете зателефонувати за номером`addPaymentMethod`для платного користувача, передаючи ідентифікатор способу оплати:

    $user->addPaymentMethod($paymentMethod);

> {tip} Щоб дізнатись, як отримати ідентифікатори способу оплати, перегляньте[документація щодо зберігання способу оплати](#storing-payment-methods).

<a name="deleting-payment-methods"></a>

### Видалення способів оплати

Щоб видалити спосіб оплати, ви можете зателефонувати за номером`delete`метод на`Laravel\Cashier\PaymentMethod`екземпляр, який ви хочете видалити:

    $paymentMethod->delete();

`deletePaymentMethods`метод видалить всю інформацію про спосіб оплати для моделі Billable:

    $user->deletePaymentMethods();

> {note} Якщо у користувача активна підписка, вам слід заборонити йому видаляти спосіб оплати за замовчуванням.

<a name="subscriptions"></a>

## Підписки

Підписки забезпечують можливість налаштування періодичних платежів для ваших клієнтів і підтримують кілька планів підписки, кількість підписки, пробні версії тощо.

<a name="creating-subscriptions"></a>

### Створення підписки

Щоб створити передплату, спочатку отримайте екземпляр вашої платної моделі, який зазвичай буде екземпляром`App\Models\User`. Отримавши екземпляр моделі, ви можете використовувати`newSubscription`метод створення передплати на модель:

    $user = User::find(1);

    $user->newSubscription('default', 'price_premium')->create($paymentMethod);

Перший аргумент, переданий в`newSubscription`метод повинен бути назвою передплати. Якщо ваша програма пропонує лише одну передплату, ви можете зателефонувати за цим номером`default`або`primary`. Другим аргументом є конкретний план, на який користувач підписується. Це значення повинно відповідати ідентифікатору ціни плану в смузі.

`create`метод, який приймає[ідентифікатор способу оплати у смузі](#storing-payment-methods)або Смуга`PaymentMethod`об'єкт, розпочне передплату, а також оновить вашу базу даних за допомогою ідентифікатора клієнта та іншої відповідної платіжної інформації.

> {note} Передача ідентифікатора способу оплати безпосередньо до`create()`метод підписки також автоматично додасть його до збережених користувачем способів оплати.

<a name="subscription-quantities"></a>

#### Кількості

Якщо ви хочете встановити певну кількість для плану під час створення передплати, ви можете використовувати`quantity`метод:

    $user->newSubscription('default', 'price_monthly')
         ->quantity(5)
         ->create($paymentMethod);

<a name="additional-details"></a>

#### Додаткові деталі

Якщо ви хочете вказати додаткові дані про клієнта або передплату, ви можете зробити це, передавши їх як другий та третій аргументи до`create`метод:

    $user->newSubscription('default', 'price_monthly')->create($paymentMethod, [
        'email' => $email,
    ], [
        'metadata' => ['note' => 'Some extra information.'],
    ]);

Щоб дізнатись більше про додаткові поля, які підтримує Stripe, ознайомтеся з документацією Stripe на[створення клієнтів](https://stripe.com/docs/api#create_customer)і[створення передплати](https://stripe.com/docs/api/subscriptions/create).

<a name="coupons"></a>

#### Купони

Якщо ви хочете застосувати купон під час створення підписки, ви можете використовувати`withCoupon`метод:

    $user->newSubscription('default', 'price_monthly')
         ->withCoupon('code')
         ->create($paymentMethod);

<a name="adding-subscriptions"></a>

#### Додавання передплат

Якщо ви хочете додати підписку для клієнта, у якого вже встановлений метод оплати за замовчуванням, ви можете використовувати`add`метод при використанні`newSubscription`метод:

    $user = User::find(1);

    $user->newSubscription('default', 'price_premium')->add();

<a name="checking-subscription-status"></a>

### Перевірка статусу передплати

Після того, як користувач підписався на вашу програму, ви можете легко перевірити стан її підписки, використовуючи різноманітні зручні методи. По-перше,`subscribed`метод повертає`true`якщо у користувача активна підписка, навіть якщо підписка наразі перебуває у пробному періоді:

    if ($user->subscribed('default')) {
        //
    }

`subscribed`метод також робить чудового кандидата на[маршрут проміжного програмного забезпечення](/docs/{{version}}/middleware), що дозволяє фільтрувати доступ до маршрутів та контролерів на основі статусу передплати користувача:

    public function handle($request, Closure $next)
    {
        if ($request->user() && ! $request->user()->subscribed('default')) {
            // This user is not a paying customer...
            return redirect('billing');
        }

        return $next($request);
    }

Якщо ви хочете визначити, чи перебуває користувач все ще в пробному періоді, ви можете використовувати`onTrial`метод. Цей метод може бути корисним для відображення попередження користувачеві про те, що він все ще перебуває у пробному періоді:

    if ($user->subscription('default')->onTrial()) {
        //
    }

`subscribedToPlan`метод може бути використаний для визначення того, чи підписаний користувач на певний план на основі заданого ідентифікатора ціни смуги. У цьому прикладі ми визначимо, чи користувач`default`передплата активно підписується на`monthly`план:

    if ($user->subscribedToPlan('price_monthly', 'default')) {
        //
    }

Передаючи масив до`subscribedToPlan`методом, ви можете визначити, чи користувач`default`передплата активно підписується на`monthly`або`yearly`план:

    if ($user->subscribedToPlan(['price_monthly', 'price_yearly'], 'default')) {
        //
    }

`recurring`метод може бути використаний, щоб визначити, чи підписаний користувач на даний момент і чи не перебуває він у пробному періоді:

    if ($user->subscription('default')->recurring()) {
        //
    }

> {note} Якщо у користувача є дві підписки з однаковим іменем, остання підписка завжди буде повернута`subscription`метод. Наприклад, у користувача можуть бути названі два записи передплати`default`; однак одна з підписок може бути старою, термін дії якої минув, тоді як інша - поточна, активна підписка. Остання підписка завжди буде повернута, тоді як попередні підписки зберігаються в базі даних для історичного перегляду.

<a name="cancelled-subscription-status"></a>

#### Статус скасованої передплати

Щоб визначити, чи був користувач колись активним передплатником, але скасував свою підписку, ви можете використовувати`cancelled`метод:

    if ($user->subscription('default')->cancelled()) {
        //
    }

Ви також можете визначити, чи скасував користувач свою підписку, але все ще перебуває на "пільговому періоді", поки підписка повністю не закінчиться. Наприклад, якщо користувач скасовує підписку на 5 березня, термін дії якої спочатку планувався до 10 березня, користувач перебуває у "пільговому періоді" до 10 березня. Зверніть увагу, що`subscribed`метод все ще повертається`true`протягом цього часу:

    if ($user->subscription('default')->onGracePeriod()) {
        //
    }

To determine if the user has cancelled their subscription and is no longer within their "grace period", you may use the `ended`метод:

    if ($user->subscription('default')->ended()) {
        //
    }

<a name="subscription-scopes"></a>

#### Сфери передплати

Більшість станів передплати також доступні у вигляді обсягу запитів, так що ви можете легко здійснити запит у своїй базі даних щодо підписок, які перебувають у певному стані:

    // Get all active subscriptions...
    $subscriptions = Subscription::query()->active()->get();

    // Get all of the cancelled subscriptions for a user...
    $subscriptions = $user->subscriptions()->cancelled()->get();

Повний список доступних областей доступний нижче:

    Subscription::query()->active();
    Subscription::query()->cancelled();
    Subscription::query()->ended();
    Subscription::query()->incomplete();
    Subscription::query()->notCancelled();
    Subscription::query()->notOnGracePeriod();
    Subscription::query()->notOnTrial();
    Subscription::query()->onGracePeriod();
    Subscription::query()->onTrial();
    Subscription::query()->pastDue();
    Subscription::query()->recurring();

<a name="incomplete-and-past-due-status"></a>

#### Неповний та прострочений статус

Якщо підписка вимагає вторинної платіжної дії після створення, підписка буде позначена як`incomplete`. Статуси підписки зберігаються в`stripe_status`стовпець Каси`subscriptions`таблиця бази даних.

Подібним чином, якщо під час обміну планами потрібна дія вторинного платежу, передплата буде позначена як`past_due`. Коли ваша підписка знаходиться в будь-якому з цих штатів, вона не буде активною, доки клієнт не підтвердить свою оплату. Перевірити, чи має передплата неповний платіж, можна за допомогою`hasIncompletePayment`метод на моделі Billable або екземплярі передплати:

    if ($user->hasIncompletePayment('default')) {
        //
    }

    if ($user->subscription('default')->hasIncompletePayment()) {
        //
    }

Якщо підписка має неповний платіж, вам слід направити користувача на сторінку підтвердження платежу Каси, передавши`latestPayment`ідентифікатор. Ви можете використовувати`latestPayment`метод, доступний в екземплярі передплати для отримання цього ідентифікатора:

    <a href="{{ route('cashier.payment', $subscription->latestPayment()->id) }}">
        Please confirm your payment.
    </a>

Якщо ви хочете, щоб підписка все ще вважалася активною, коли вона знаходиться в`past_due`штату, ви можете використовувати`keepPastDueSubscriptionsActive`спосіб, наданий Касиром. Як правило, цей метод слід викликати в`register`метод вашого`AppServiceProvider`:

    use Laravel\Cashier\Cashier;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        Cashier::keepPastDueSubscriptionsActive();
    }

> {note} Коли підписка знаходиться в`incomplete`заявіть, що його не можна змінити, поки не буде підтверджено платіж. Тому`swap`і`updateQuantity`методи видадуть виняток, коли передплата знаходиться в`incomplete`держава.

<a name="changing-plans"></a>

### Зміна планів

Після того, як користувач підписався на вашу програму, він іноді може захотіти перейти на новий план підписки. Щоб замінити користувача на нову підписку, передайте ідентифікатор ціни плану на`swap`метод:

    $user = App\Models\User::find(1);

    $user->subscription('default')->swap('provider-price-id');

Якщо користувач проходить пробний період, пробний період буде збережено. Крім того, якщо для підписки існує "кількість", ця кількість також буде збережена.

Якщо ви хочете поміняти місцями та скасувати будь-який пробний період, який зараз перебуває користувач, ви можете використовувати`skipTrial`метод:

    $user->subscription('default')
            ->skipTrial()
            ->swap('provider-price-id');

Якщо ви хочете поміняти місцями та негайно виставити рахунок користувачеві, а не чекати наступного платіжного циклу, ви можете використовувати`swapAndInvoice`метод:

    $user = App\Models\User::find(1);

    $user->subscription('default')->swapAndInvoice('provider-price-id');

<a name="prorations"></a>

#### Пропорції

За замовчуванням Stripe пропорційно стягує плату при обміні між планами.`noProrate`метод може бути використаний для оновлення передплати без пропорційних витрат:

    $user->subscription('default')->noProrate()->swap('provider-price-id');

Щоб отримати додаткову інформацію про розподіл передплати, зверніться до[Смугаста документація](https://stripe.com/docs/billing/subscriptions/prorations).

> {note} Виконання`noProrate`метод до`swapAndInvoice`метод не вплине на розподіл. Рахунок завжди буде виставлений.

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

Крім того, ви можете встановити конкретну кількість за допомогою`updateQuantity`метод:

    $user->subscription('default')->updateQuantity(10);

`noProrate`метод може бути використаний для оновлення кількості передплати без збільшення пропорцій:

    $user->subscription('default')->noProrate()->updateQuantity(10);

Щоб отримати додаткову інформацію про кількість передплати, зверніться до[Смугаста документація](https://stripe.com/docs/subscriptions/quantities).

> {note} Зверніть увагу, що при роботі з багатоплановими підписками для зазначених вище методів кількості потрібен додатковий параметр "план".

<a name="multiplan-subscriptions"></a>

### Підписки на кілька планів

[Багатопланові підписки](https://stripe.com/docs/billing/subscriptions/multiplan)дозволяють призначати кілька тарифних планів для однієї передплати. Наприклад, уявіть, що ви створюєте додаток служби підтримки клієнтів, який має базову підписку 10 доларів на місяць, але пропонує додатковий план живого чату на додаткові 15 доларів на місяць:

    $user = User::find(1);

    $user->newSubscription('default', [
        'price_monthly',
        'chat-plan',
    ])->create($paymentMethod);

Тепер у клієнта буде два плани`default`передплата. За обидва тарифи буде стягуватися плата відповідно до відповідних інтервалів виставлення рахунків. Ви також можете використовувати`quantity`спосіб вказати конкретну кількість для кожного плану:

    $user = User::find(1);

    $user->newSubscription('default', ['price_monthly', 'chat-plan'])
        ->quantity(5, 'chat-plan')
        ->create($paymentMethod);

Або ви можете динамічно додавати додатковий план та його кількість, використовуючи`plan`метод:

    $user = User::find(1);

    $user->newSubscription('default', 'price_monthly')
        ->plan('chat-plan', 5)
        ->create($paymentMethod);

Крім того, ви можете додати новий план до існуючої передплати пізніше:

    $user = User::find(1);

    $user->subscription('default')->addPlan('chat-plan');

У наведеному вище прикладі буде додано новий план, і клієнту буде виставлено рахунок за нього під час наступного платіжного циклу. Якщо ви хочете негайно виставити рахунок клієнту, ви можете скористатися ним`addPlanAndInvoice`метод:

    $user->subscription('default')->addPlanAndInvoice('chat-plan');

Якщо ви хочете додати план із конкретною кількістю, ви можете передати величину як другий параметр`addPlan`або`addPlanAndInvoice`методи:

    $user = User::find(1);

    $user->subscription('default')->addPlan('chat-plan', 5);

Ви можете видалити плани з підписок за допомогою`removePlan`метод:

    $user->subscription('default')->removePlan('chat-plan');

> {note} Ви не можете видалити останній план підписки. Натомість вам слід просто скасувати підписку.

<a name="swapping"></a>

### Обмін

Ви також можете змінити плани, приєднані до багатопланової передплати. Наприклад, уявіть, що ви на`basic-plan`передплата з`chat-plan`і ви хочете перейти на`pro-plan`план:

    $user = User::find(1);

    $user->subscription('default')->swap(['pro-plan', 'chat-plan']);

При виконанні наведеного вище коду основний елемент передплати з`basic-plan`видаляється, а той із`chat-plan`зберігається. Крім того, новий елемент передплати на новий`pro-plan`створюється.

Ви також можете вказати параметри елементів передплати. Наприклад, вам може знадобитися вказати кількості плану підписки:

    $user = User::find(1);

    $user->subscription('default')->swap([
        'pro-plan' => ['quantity' => 5],
        'chat-plan'
    ]);

Якщо ви хочете поміняти один план на передплаті, ви можете зробити це за допомогою`swap`метод на самому елементі передплати. Цей підхід корисний, якщо ви, наприклад, хочете зберегти всі існуючі метадані в елементі передплати.

    $user = User::find(1);

    $user->subscription('default')
            ->findItemOrFail('basic-plan')
            ->swap('pro-plan');

<a name="proration"></a>

#### Прорація

За замовчуванням Stripe буде пропорційно стягувати плату під час додавання або видалення планів з передплати. Якщо ви хочете внести коригування плану без пропорції, вам слід встановити ланцюжок`noProrate`методу на вашій операції плану:

    $user->subscription('default')->noProrate()->removePlan('chat-plan');

<a name="swapping-quantities"></a>

#### Кількості

Якщо ви хочете оновити кількість за окремими тарифними планами, ви можете зробити це за допомогою[існуючі кількісні методи](#subscription-quantity)і передаючи ім'я плану як додатковий аргумент методу:

    $user = User::find(1);

    $user->subscription('default')->incrementQuantity(5, 'chat-plan');

    $user->subscription('default')->decrementQuantity(3, 'chat-plan');

    $user->subscription('default')->updateQuantity(10, 'chat-plan');

> {note} Якщо у вас встановлено кілька тарифів на підписку,`stripe_plan`і`quantity`атрибути на`Subscription`модель буде`null`. Щоб отримати доступ до індивідуальних планів, вам слід використовувати`items`відносини, доступні на`Subscription`модель.

<a name="subscription-items"></a>

#### Елементи передплати

Якщо у підписки є кілька планів, вона буде мати кілька елементів підписки, що зберігаються у вашій базі даних`subscription_items`таблиця. Ви можете отримати доступ до них через`items`відносини за передплатою:

    $user = User::find(1);

    $subscriptionItem = $user->subscription('default')->items->first();

    // Retrieve the Stripe plan and quantity for a specific item...
    $stripePlan = $subscriptionItem->stripe_plan;
    $quantity = $subscriptionItem->quantity;

Ви також можете отримати конкретний план за допомогою`findItemOrFail`метод:

    $user = User::find(1);

    $subscriptionItem = $user->subscription('default')->findItemOrFail('chat-plan');

<a name="subscription-taxes"></a>

### Податки на підписку

Щоб визначити ставки податку, які користувач сплачує за передплату, застосуйте`taxRates`метод на вашій платіжній моделі та поверніть масив із ідентифікаторами податкових ставок. Ви можете визначити ці ставки податку в[панель приладів Stripe](https://dashboard.stripe.com/test/tax-rates):

    public function taxRates()
    {
        return ['tax-rate-id'];
    }

`taxRates`Метод дозволяє застосовувати ставку податку на основі моделі, що може бути корисним для бази користувачів, яка охоплює кілька країн і податкові ставки. Якщо ви працюєте з багатоплановими підписками, ви можете визначити різні ставки податку для кожного плану, застосувавши a`planTaxRates`метод на вашій оплачуваній моделі:

    public function planTaxRates()
    {
        return [
            'plan-id' => ['tax-rate-id'],
        ];
    }

> {примітка}`taxRates`метод застосовується лише до плати за передплату. Якщо ви використовуєте Касу для одноразових платежів, вам потрібно буде вручну вказати ставку податку на той момент.

<a name="syncing-tax-rates"></a>

#### Синхронізація податкових ставок

При зміні жорстко закодованих ідентифікаторів податкових ставок, що повертаються`taxRates`методом, податкові налаштування для будь-яких існуючих підписок для користувача залишаться незмінними. Якщо ви хочете оновити податкову вартість для існуючих підписок із повернутими`taxTaxRates`значення, вам слід викликати`syncTaxRates`метод на екземплярі передплати користувача:

    $user->subscription('default')->syncTaxRates();

Це також синхронізує будь-які ставки податку на підписку, тому переконайтеся, що ви також правильно змінили`planTaxRates`метод.

<a name="tax-exemption"></a>

#### Звільнення від оподаткування

Касир також пропонує методи визначення, чи клієнт звільнений від сплати податку, зателефонувавши Stripe API.`isNotTaxExempt`,`isTaxExempt`, і`reverseChargeApplies`методи, доступні на платній моделі:

    $user = User::find(1);

    $user->isTaxExempt();
    $user->isNotTaxExempt();
    $user->reverseChargeApplies();

Ці методи також доступні на будь-яких`Laravel\Cashier\Invoice`об'єкт. Однак при виклику цих методів на`Invoice`об'єкт, методи визначатимуть статус звільнення під час створення рахунка-фактури.

<a name="subscription-anchor-date"></a>

### Дата прив’язки підписки

За замовчуванням прив’язкою платіжного циклу є дата створення передплати, або якщо використовується пробний період, дата закінчення пробної версії. Якщо ви хочете змінити дату прив’язки рахунків, ви можете використовувати`anchorBillingCycleOn`метод:

    use App\Models\User;
    use Carbon\Carbon;

    $user = User::find(1);

    $anchor = Carbon::parse('first day of next month');

    $user->newSubscription('default', 'price_premium')
                ->anchorBillingCycleOn($anchor->startOfDay())
                ->create($paymentMethod);

Для отримання додаткової інформації щодо управління циклами виставлення рахунків передплати зверніться до[Документація циклу виставлення рахунків у смужку](https://stripe.com/docs/billing/subscriptions/billing-cycle)

<a name="cancelling-subscriptions"></a>

### Скасування передплати

Щоб скасувати передплату, зателефонуйте на`cancel`метод підписки користувача:

    $user->subscription('default')->cancel();

Коли підписку скасовано, Касир автоматично встановить`ends_at`у вашій базі даних. Цей стовпець використовується, щоб знати, коли`subscribed`метод повинен почати повертатися`false`. Наприклад, якщо клієнт скасовує підписку 1 березня, але передплата закінчувалася до 5 березня,`subscribed`метод продовжить повертатися`true`до 5 березня.

Ви можете визначити, скасував користувач підписку, але все ще перебуває на "пільговому періоді", використовуючи`onGracePeriod`метод:

    if ($user->subscription('default')->onGracePeriod()) {
        //
    }

Якщо ви бажаєте негайно скасувати підписку, зателефонуйте на номер`cancelNow`метод підписки користувача:

    $user->subscription('default')->cancelNow();

<a name="resuming-subscriptions"></a>

### Відновлення передплати

Якщо користувач скасував свою підписку, і ви хочете відновити її, скористайтесь`resume`метод. Користувач**повинен**все ще перебувати у своєму пільговому періоді, щоб відновити підписку:

    $user->subscription('default')->resume();

Якщо користувач скасує підписку, а потім відновить підписку до закінчення терміну дії підписки, оплата за них не стягуватиметься негайно. Натомість їх передплата буде повторно активована, і вони будуть виставлені рахунки за початковим розрахунковим циклом.

<a name="subscription-trials"></a>

## Випробувальні підписки

<a name="with-payment-method-up-front"></a>

### Зі способом оплати вперед

Якщо ви хочете запропонувати пробні періоди своїм клієнтам, одночасно збираючи інформацію про спосіб оплати, вам слід скористатися`trialDays`під час створення підписки:

    $user = User::find(1);

    $user->newSubscription('default', 'price_monthly')
                ->trialDays(10)
                ->create($paymentMethod);

Цей метод встановить дату закінчення пробного періоду в записі підписки в базі даних, а також вкаже Stripe не розпочинати виставлення рахунків клієнту до цієї дати. При використанні`trialDays`методом, Касир замінить будь-який пробний період за промовчанням, налаштований для плану в Stripe.

> {note} Якщо передплата клієнта не буде скасована до дати закінчення пробного періоду, з них буде стягнуто плату, як тільки закінчиться пробна версія, тому ви повинні обов’язково повідомити своїх користувачів про дату закінчення пробного періоду.

`trialUntil`метод дозволяє надати a`DateTime`екземпляр, щоб вказати, коли повинен закінчитися пробний період:

    use Carbon\Carbon;

    $user->newSubscription('default', 'price_monthly')
                ->trialUntil(Carbon::now()->addDays(10))
                ->create($paymentMethod);

Ви можете визначити, чи перебуває користувач протягом пробного періоду, використовуючи будь-який із`onTrial`метод екземпляра користувача, або`onTrial`метод екземпляра передплати. Два наведені нижче приклади однакові:

    if ($user->onTrial('default')) {
        //
    }

    if ($user->subscription('default')->onTrial()) {
        //
    }

<a name="defining-trial-days-in-stripe-cashier"></a>

#### Визначення пробних днів у смузі / касі

Ви можете визначити, скільки пробних днів отримує ваш план на інформаційній панелі Stripe, або завжди явно передавати їх за допомогою Cashier. Якщо ви вирішите визначити пробні дні свого плану в Stripe, ви повинні пам’ятати, що нові підписки, включаючи нові підписки для клієнта, який раніше підписувався, завжди отримуватимуть пробний період, якщо ви явно не зателефонуєте`trialDays(0)`метод.

<a name="without-payment-method-up-front"></a>

### Без способу оплати

Якщо ви хочете запропонувати пробні періоди без попереднього збору інформації про спосіб оплати користувача, ви можете встановити`trial_ends_at`у стовпці запису користувача до бажаної дати закінчення пробного періоду. Зазвичай це робиться під час реєстрації користувача:

    $user = User::create([
        // Populate other user properties...
        'trial_ends_at' => now()->addDays(10),
    ]);

> {note} Обов’язково додайте a[мутатор дати](/docs/{{version}}/eloquent-mutators#date-mutators)для`trial_ends_at`до визначення вашої моделі.

Касир називає цей тип судового розгляду «загальним випробуванням», оскільки він не додається до жодної існуючої передплати.`onTrial`метод на`User`екземпляр повернеться`true`якщо поточна дата не минула значення`trial_ends_at`:

    if ($user->onTrial()) {
        // User is within their trial period...
    }

Коли ви будете готові створити фактичну передплату для користувача, ви можете використовувати`newSubscription`метод, як зазвичай:

    $user = User::find(1);

    $user->newSubscription('default', 'price_monthly')->create($paymentMethod);

Щоб отримати дату закінчення пробного періоду для користувача, ви можете використовувати`trialEndsAt`метод. Цей метод повертає примірник дати Carbon, якщо користувач перебуває на пробній версії або`null`якщо їх немає. Ви також можете передати необов’язковий параметр імені підписки, якщо хочете отримати дату закінчення пробного періоду для певної підписки, крім стандартної:

    if ($user->onTrial()) {
        $trialEndsAt = $user->trialEndsAt('main');
    }

Ви також можете використовувати`onGenericTrial`метод, якщо ви хочете конкретно знати, що користувач перебуває у межах свого "загального" пробного періоду і ще не створив фактичну передплату:

    if ($user->onGenericTrial()) {
        // User is within their "generic" trial period...
    }

<a name="extending-trials"></a>

### Розширення випробувань

`extendTrial`метод дозволяє продовжити пробний період підписки після її створення:

    // End the trial 7 days from now...
    $subscription->extendTrial(
        now()->addDays(7)
    );

    // Add an additional 5 days to the trial...
    $subscription->extendTrial(
        $subscription->trial_ends_at->addDays(5)
    );

Якщо пробна версія вже закінчилася, а клієнту вже виставляється рахунок за підписку, ви все одно можете запропонувати їм розширену пробну версію. Час, проведений протягом пробного періоду, буде вираховуватися із наступного рахунку клієнта.

<a name="handling-stripe-webhooks"></a>

## Обробка смугових веб-хуків

> {tip} Ви можете використовувати[смужка CLI](https://stripe.com/docs/stripe-cli)для тестування веб-хуків під час локальної розробки.

Stripe може повідомляти вашу заявку про різноманітні події через веб-хуки. За замовчуванням маршрут, який вказує на контролер веб-хука Каси, налаштовується через постачальника послуг Каси. Цей контролер обробляє всі вхідні запити веб-хука.

За замовчуванням цей контролер автоматично обробляє скасування підписок, у яких занадто багато невдалих платежів (як це визначено в налаштуваннях Stripe), оновлення клієнтів, видалення клієнтів, оновлення передплати та зміни способу оплати; однак, як ми скоро дізнаємось, ви можете розширити цей контролер для обробки будь-якої події веб-хука, яка вам подобається.

Щоб ваша програма могла обробляти веб-хуки Stripe, налаштуйте URL-адресу веб-хука на панелі керування Stripe. За замовчуванням контролер веб-хука Касир прослуховує`/stripe/webhook`Шлях URL. Повний список усіх веб-хуків, які слід налаштувати на панелі керування Stripe:

-   `customer.subscription.updated`
-   `customer.subscription.deleted`
-   `customer.updated`
-   `customer.deleted`
-   `invoice.payment_action_required`

> {note} Обов’язково захистіть вхідні запити за допомогою каси[перевірка підпису](/docs/{{version}}/billing#verifying-webhook-signatures)проміжне програмне забезпечення.

<a name="webhooks-csrf-protection"></a>

#### Захист веб-хуків та CSRF

Оскільки веб-хуки Stripe повинні обійти Laravel[Захист CSRF](/docs/{{version}}/csrf), не забудьте вказати URI як виняток у вашому`VerifyCsrfToken`проміжне програмне забезпечення або перелічіть маршрут за межами`web`група проміжного програмного забезпечення:

    protected $except = [
        'stripe/*',
    ];

<a name="defining-webhook-event-handlers"></a>

### Визначення обробників подій Webhook

Каса автоматично обробляє скасування передплати за невдалі платежі, але якщо у вас є додаткові події веб-хука, які ви хотіли б обробити, розширте контролер Webhook. Назви ваших методів повинні відповідати очікуваній умові Касира, зокрема, до методів слід вводити префікс`handle`і назва "справи верблюда" веб-гачка, з яким ви хочете обробити. Наприклад, якщо ви хочете обробляти файл`invoice.payment_succeeded`webhook, вам слід додати a`handleInvoicePaymentSucceeded`метод до контролера:

    <?php

    namespace App\Http\Controllers;

    use Laravel\Cashier\Http\Controllers\WebhookController as CashierController;

    class WebhookController extends CashierController
    {
        /**
         * Handle invoice payment succeeded.
         *
         * @param  array  $payload
         * @return \Symfony\Component\HttpFoundation\Response
         */
        public function handleInvoicePaymentSucceeded($payload)
        {
            // Handle The Event
        }
    }

Далі визначте маршрут до свого контролера Каси в межах вашого`routes/web.php`файл. Це перезапише відправлений маршрут за замовчуванням:

    use App\Http\Controllers\WebhookController;

    Route::post(
        'stripe/webhook',
        [WebhookController::class, 'handleWebhook']
    );

Касир виділяє`Laravel\Cashier\Events\WebhookReceived`подія, коли отримано веб-хук, та`Laravel\Cashier\Events\WebhookHandled`подія, коли веб-хук обробляв Касир. Обидві події містять повне корисне навантаження веб-хука Stripe.

<a name="handling-failed-subscriptions"></a>

### Невдалі підписки

Що робити, якщо термін дії кредитної картки клієнта закінчується? Не хвилюйтеся - контролер Webhook каси скасує передплату клієнта для вас. Невдалі платежі будуть автоматично зафіксовані та оброблені контролером. Контролер скасує передплату клієнта, коли Stripe виявить, що підписка не вдалася (зазвичай після трьох невдалих спроб оплати).

<a name="verifying-webhook-signatures"></a>

### Перевірка підписів Webhook

Для захисту веб-хуків ви можете використовувати[Підписи веб-хука Стріпа](https://stripe.com/docs/webhooks/signatures). Для зручності Cashier автоматично включає проміжне програмне забезпечення, яке підтверджує, що вхідний запит веб-хука Stripe є дійсним.

Щоб увімкнути перевірку веб-хука, переконайтеся, що`STRIPE_WEBHOOK_SECRET`змінна середовища встановлена ​​у вашому`.env`файл. Веб-хук`secret`може бути отримано з інформаційної панелі вашого облікового запису Stripe.

<a name="single-charges"></a>

## Одноразові збори

<a name="simple-charge"></a>

### Проста зарядка

> {примітка}`charge`метод приймає суму, яку ви хочете стягнути в**найменший знаменник валюти, що використовується вашою заявкою**.

Якщо ви хочете зробити одноразову комісію за спосіб оплати підписаного клієнта, ви можете використовувати`charge`метод на екземплярі платної моделі. Вам потрібно буде[надайте ідентифікатор способу оплати](#storing-payment-methods)як другий аргумент:

    // Stripe Accepts Charges In Cents...
    $stripeCharge = $user->charge(100, $paymentMethod);

`charge`метод приймає масив як свій третій аргумент, дозволяючи передавати будь-які параметри, які ви хочете, для базового створення заряду Stripe. Зверніться до документації Stripe щодо варіантів, доступних вам під час створення нарахувань:

    $user->charge(100, $paymentMethod, [
        'custom_option' => $value,
    ]);

Ви також можете використовувати`charge`метод без основного замовника або користувача:

    use App\Models\User;

    $stripeCharge = (new User)->charge(100, $paymentMethod);

`charge`метод видасть виняток, якщо заряд не вдасться. Якщо звинувачення успішне, примірник`Laravel\Cashier\Payment`буде повернуто з методу:

    try {
        $payment = $user->charge(100, $paymentMethod);
    } catch (Exception $e) {
        //
    }

<a name="charge-with-invoice"></a>

### Стягніть рахунок-фактуру

Іноді вам може знадобитися зробити одноразову оплату, але також сформувати рахунок-фактуру, щоб ви могли запропонувати замовнику квитанцію у форматі PDF.`invoiceFor`метод дозволяє зробити саме це. Наприклад, давайте виставляємо рахунки клієнту 5,00 доларів США за "одноразову плату":

    // Stripe Accepts Charges In Cents...
    $user->invoiceFor('One Time Fee', 500);

Рахунок-фактура буде негайно стягнуто з використання способу оплати користувача за замовчуванням.`invoiceFor`метод також приймає масив як свій третій аргумент. Цей масив містить варіанти виставлення рахунка для позиції рахунка-фактури. Четвертий аргумент, прийнятий методом, також є масивом. Цей остаточний аргумент приймає варіанти виставлення рахунків для самого рахунку-фактури:

    $user->invoiceFor('Stickers', 500, [
        'quantity' => 50,
    ], [
        'default_tax_rates' => ['tax-rate-id'],
    ]);

> {примітка}`invoiceFor`метод створить рахунок-фактуру Stripe, який повторить спроби невдалих спроб виставлення рахунків. Якщо ви не хочете, щоб рахунки-фактури повторювали невдалі платежі, вам доведеться закрити їх за допомогою API Stripe після першого невдалого стягнення плати.

<a name="refunding-charges"></a>

### Повернення коштів

Якщо вам потрібно повернути плату за Stripe, ви можете використовувати`refund`метод. Цей метод приймає ідентифікатор наміру платежу в якості першого аргументу:

    $payment = $user->charge(100, $paymentMethod);

    $user->refund($payment->id);

<a name="invoices"></a>

## Рахунки-фактури

<a name="retrieving-invoices"></a>

### Отримання рахунків-фактур

Ви можете легко отримати масив рахунків-фактур, що оплачується, за допомогою`invoices`метод:

    $invoices = $user->invoices();

    // Include pending invoices in the results...
    $invoices = $user->invoicesIncludingPending();

Ви можете використовувати`findInvoice`спосіб отримання конкретного рахунку-фактури:

    $invoice = $user->findInvoice($invoiceId);

<a name="displaying-invoice-information"></a>

#### Відображення інформації про рахунок-фактуру

Перераховуючи рахунки для замовника, ви можете використовувати допоміжні методи рахунку-фактури для відображення відповідної інформації про рахунок. Наприклад, ви можете перерахувати кожен рахунок-фактуру в таблиці, що дозволить користувачеві легко завантажити будь-який з них:

    <table>
        @foreach ($invoices as $invoice)
            <tr>
                <td>{{ $invoice->date()->toFormattedDateString() }}</td>
                <td>{{ $invoice->total() }}</td>
                <td><a href="/user/invoice/{{ $invoice->id }}">Download</a></td>
            </tr>
        @endforeach
    </table>

<a name="generating-invoice-pdfs"></a>

### Створення PDF-фактур

Зсередини маршруту або контролера використовуйте`downloadInvoice`метод створення PDF-фактури для завантаження. Цей метод автоматично генерує правильну відповідь HTTP для відправки завантаження у браузер:

    use Illuminate\Http\Request;

    Route::get('user/invoice/{invoice}', function (Request $request, $invoiceId) {
        return $request->user()->downloadInvoice($invoiceId, [
            'vendor' => 'Your Company',
            'product' => 'Your Product',
        ]);
    });

`downloadInvoice`метод також допускає необов'язкове власне ім'я файлу як третій параметр. Це ім'я файлу буде автоматично суфіксом`.pdf`для вас:

    return $request->user()->downloadInvoice($invoiceId, [
        'vendor' => 'Your Company',
        'product' => 'Your Product',
    ], 'my-invoice');

<a name="handling-failed-payments"></a>

## Обробка невдалих платежів

Іноді платежі за передплату або одноразові платежі можуть не вдатися. Коли це станеться, Касир кине знак`IncompletePayment`виняток, який повідомляє вас, що це сталося. Вловивши цей виняток, у вас є два варіанти, як діяти далі.

По-перше, ви можете перенаправити клієнта на спеціальну сторінку підтвердження платежу, яка додається до каси. Ця сторінка вже має пов’язаний маршрут, який зареєстрований через постачальника послуг каси. Отже, ви можете зловити`IncompletePayment`виключення та перенаправлення на сторінку підтвердження платежу:

    use Laravel\Cashier\Exceptions\IncompletePayment;

    try {
        $subscription = $user->newSubscription('default', $planId)
                                ->create($paymentMethod);
    } catch (IncompletePayment $exception) {
        return redirect()->route(
            'cashier.payment',
            [$exception->payment->id, 'redirect' => route('home')]
        );
    }

На сторінці підтвердження платежу клієнту буде запропоновано ще раз ввести дані своєї кредитної картки та виконати будь-які додаткові дії, які вимагає Stripe, наприклад, підтвердження "3D Secure". Після підтвердження оплати користувач буде перенаправлений на URL-адресу, надану`redirect`параметр, зазначений вище. При перенаправленні,`message`(рядок) та`success`(цілочисельні) змінні рядка запиту будуть додані до URL-адреси.

Крім того, ви можете дозволити Stripe обробляти для вас підтвердження платежу. У цьому випадку замість переадресації на сторінку підтвердження платежу ви можете[налаштування електронних листів автоматичного виставлення рахунків](https://dashboard.stripe.com/account/billing/automatic)на панелі приладів Stripe. Однак якщо`IncompletePayment`винятком виявлено, ви все одно повинні повідомити користувача, що він отримає електронне повідомлення з подальшими інструкціями щодо підтвердження платежу.

Винятки з оплати можуть бути використані для таких способів:`charge`,`invoiceFor`, і`invoice`на`Billable`користувач. При обробці підписок файл`create`метод на`SubscriptionBuilder`, і`incrementAndInvoice`і`swapAndInvoice`методи на`Subscription`модель може створювати винятки.

В даний час існує два типи винятків із платежів, які поширюються`IncompletePayment`. За потреби ви можете зловити їх окремо, щоб налаштувати взаємодію з користувачем:

<div class="content-list" markdown="1">
- `PaymentActionRequired`: this indicates that Stripe requires extra verification in order to confirm and process a payment.
- `PaymentFailure`: this indicates that a payment failed for various other reasons, such as being out of available funds.
</div>

<a name="strong-customer-authentication"></a>

## Надійна аутентифікація клієнта

Якщо ваш бізнес базується в Європі, вам доведеться дотримуватися вимог жорсткої автентифікації клієнтів (SCA). Ці правила були введені у вересні 2019 року Європейським Союзом для запобігання шахрайству з платежами. На щастя, Stripe та Cashier підготовлені до створення додатків, сумісних із SCA.

> {note} Перш ніж розпочати, перегляньте[Посібник Stripe щодо PSD2 та SCA](https://stripe.com/guides/strong-customer-authentication)а також їх[документація щодо нових API SCA](https://stripe.com/docs/strong-customer-authentication).

<a name="payments-requiring-additional-confirmation"></a>

### Платежі, що вимагають додаткового підтвердження

Положення SCA часто вимагають додаткової перевірки для підтвердження та обробки платежу. Коли це станеться, Касир кине`PaymentActionRequired`виняток, який повідомляє вас про необхідність додаткової перевірки. Більше інформації про те, як обробляти ці винятки, можна знайти[тут](#handling-failed-payments).

<a name="incomplete-and-past-due-state"></a>

#### Незавершений та прострочений строк

Коли платіж потребує додаткового підтвердження, передплата залишатиметься в`incomplete`або`past_due`держави, як вказує його`stripe_status`стовпець бази даних. Каса автоматично активує передплату клієнта через веб-хук, як тільки підтвердження оплати буде завершено.

Для отримання додаткової інформації про`incomplete`і`past_due`штати, будь ласка, зверніться до[наша додаткова документація](#incomplete-and-past-due-status).

<a name="off-session-payment-notifications"></a>

### Повідомлення про оплату поза сесією

Оскільки правила SCA вимагають від клієнтів час від часу перевіряти свої платіжні реквізити, навіть коли підписка активна, Касир може надіслати клієнту сповіщення про платіж, коли потрібно підтвердження оплати поза сесією. Наприклад, це може статися, коли передплата поновлюється. Повідомлення про оплату каси можна ввімкнути, встановивши`CASHIER_PAYMENT_NOTIFICATION`середовища до класу сповіщень. За замовчуванням це сповіщення вимкнено. Звичайно, Cashier включає клас сповіщень, який ви можете використовувати для цієї мети, але за бажанням ви можете надати власний клас сповіщень:

    CASHIER_PAYMENT_NOTIFICATION=Laravel\Cashier\Notifications\ConfirmPayment

Щоб переконатися, що надсилаються повідомлення про підтвердження оплати поза сесією, перевірте це[Смугасті веб-хуки налаштовані](#handling-stripe-webhooks)для вашої заявки та`invoice.payment_action_required`webhook увімкнено на вашій інформаційній панелі Stripe. Крім того, ваш`Billable`модель також повинна використовувати Laravel`Illuminate\Notifications\Notifiable`риса.

> {note} Сповіщення надсилатимуться навіть тоді, коли клієнти здійснюють платіж вручну, що вимагає додаткового підтвердження. На жаль, Stripe не може дізнатися, що оплата була здійснена вручну або "поза сесією". Але клієнт просто побачить повідомлення "Оплата виконана", якщо він відвідає сторінку платежів після того, як уже підтвердить свій платіж. Клієнту не дозволяється випадково двічі підтвердити один і той же платіж і випадково здійснити другу оплату.

<a name="stripe-sdk"></a>

## Смугастий SDK

Багато об'єктів Каси є обгортками навколо об'єктів Stripe SDK. Якщо ви хочете взаємодіяти з об'єктами Stripe безпосередньо, ви можете зручно отримати їх за допомогою`asStripe`метод:

    $stripeSubscription = $subscription->asStripeSubscription();

    $stripeSubscription->application_fee_percent = 5;

    $stripeSubscription->save();

Ви також можете використовувати`updateStripeSubscription`метод оновлення підписки Stripe безпосередньо:

    $subscription->updateStripeSubscription(['application_fee_percent' => 5]);

<a name="testing"></a>

## Тестування

Під час тестування програми, яка використовує Cashier, ви можете знущатися над фактичними HTTP-запитами до Stripe API; однак для цього потрібно частково повторно реалізувати власну поведінку Касира. Тому ми рекомендуємо дозволити вашим тестам вражати фактичний Stripe API. Хоча це відбувається повільніше, воно забезпечує більшу впевненість у тому, що ваш додаток працює належним чином, і будь-які повільні тести можуть бути розміщені у власній групі тестування PHPUnit.

Під час тестування пам’ятайте, що сам Касир вже має чудовий пакет тестів, тому вам слід зосередитись лише на тестуванні підписки та потоку платежів у вашому власному додатку, а не на кожній базовій поведінці Касира.

Для початку додайте**тестування**версія вашої смужки секрет для вашого`phpunit.xml`файл:

    <env name="STRIPE_SECRET" value="sk_test_<your-key>"/>

Тепер, коли ви взаємодієте з Cashier під час тестування, він надсилатиме фактичні запити API до вашого середовища тестування Stripe. Для зручності слід попередньо заповнити свій тестовий акаунт Stripe підписками / планами, які потім можна використовувати під час тестування.

> {tip} Для тестування різноманітних сценаріїв виставлення рахунків, таких як відмови та помилки на кредитних картках, ви можете використовувати широкий спектр[тестування номерів карток та жетонів](https://stripe.com/docs/testing)надає Stripe.
