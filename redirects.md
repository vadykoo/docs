# Перенаправлення HTTP

-   [Створення переадресацій](#creating-redirects)
-   [Перенаправлення на іменовані маршрути](#redirecting-named-routes)
-   [Перенаправлення на дії контролера](#redirecting-controller-actions)
-   [Redirecting With Flashed Session Data](#redirecting-with-flashed-session-data)

<a name="creating-redirects"></a>

## Створення переадресацій

Переадресаційні відповіді є прикладами`Illuminate\Http\RedirectResponse`класу та містять відповідні заголовки, необхідні для перенаправлення користувача на іншу URL-адресу. Є кілька способів генерувати a`RedirectResponse`екземпляр. Найпростіший метод - використання глобального`redirect`помічник:

    Route::get('dashboard', function () {
        return redirect('home/dashboard');
    });

Іноді вам може знадобитися перенаправити користувача у попереднє місце розташування, наприклад, коли надіслана форма недійсна. Ви можете зробити це, використовуючи глобальний`back`допоміжна функція. Оскільки ця функція використовує[сесія](/docs/{{version}}/session), переконайтеся, що маршрут викликає`back`функція використовує`web`групу проміжного програмного забезпечення або застосував усі проміжні програми сеансу:

    Route::post('user/profile', function () {
        // Validate the request...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>

## Перенаправлення на іменовані маршрути

Коли ви телефонуєте на`redirect`помічник без параметрів, екземпляр`Illuminate\Routing\Redirector`повертається, що дозволяє вам викликати будь-який метод на`Redirector`екземпляр. Наприклад, для створення`RedirectResponse`до вказаного маршруту, ви можете використовувати`route`метод:

    return redirect()->route('login');

Якщо ваш маршрут має параметри, ви можете передати їх як другий аргумент до`route`метод:

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', ['id' => 1]);

<a name="populating-parameters-via-eloquent-models"></a>

#### Заповнення параметрів за допомогою Eloquent моделей

Якщо ви переспрямовуєте на маршрут із параметром "ID", який заповнюється з красномовної моделі, ви можете передати саму модель. Ідентифікатор буде витягнуто автоматично:

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', [$user]);

Якщо ви хочете налаштувати значення, яке розміщується в параметрі маршруту, вам слід замінити`getRouteKey`метод на вашій красномовній моделі:

    /**
     * Get the value of the model's route key.
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>

## Перенаправлення на дії контролера

Ви також можете генерувати переспрямування на[дії контролера](/docs/{{version}}/controllers). Для цього передайте контролер та ім'я дії в`action`метод:

    use App\Http\Controllers\HomeController;

    return redirect()->action([HomeController::class, 'index']);

Якщо ваш маршрут контролера вимагає параметрів, ви можете передати їх як другий аргумент до`action`метод:

    return redirect()->action(
        [UserController::class, 'profile'], ['id' => 1]
    );

<a name="redirecting-with-flashed-session-data"></a>

## Переадресація за допомогою прошитих даних сеансу

Перенаправлення на нову URL-адресу та[перепрошивка даних до сеансу](/docs/{{version}}/session#flash-data)зазвичай робляться одночасно. Як правило, це робиться після успішного виконання дії під час прошивки повідомлення про успіх до сеансу. Для зручності ви можете створити файл`RedirectResponse`дані екземпляра та флеш-пам’яті для сеансу в одному, вільному ланцюжку методів:

    Route::post('user/profile', function () {
        // Update the user's profile...

        return redirect('dashboard')->with('status', 'Profile updated!');
    });

Після перенаправлення користувача ви можете відобразити блимає повідомлення з[сесія](/docs/{{version}}/session). Наприклад, використовуючи[Blade синтаксису](/docs/{{version}}/blade):

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif
