# Обробка помилок

-   [Вступ](#introduction)
-   [Конфігурація](#configuration)
-   [Обробник винятків](#the-exception-handler)
    -   [Звітування про винятки](#reporting-exceptions)
    -   [Візуалізація винятків](#rendering-exceptions)
    -   [Звітні та візуальні винятки](#renderable-exceptions)
-   [Винятки HTTP](#http-exceptions)
    -   [Спеціальні сторінки помилок HTTP](#custom-http-error-pages)

<a name="introduction"></a>

## Вступ

Коли ви запускаєте новий проект Laravel, для вас вже налаштовано обробку помилок та винятків.`App\Exceptions\Handler`клас - це місце, де всі винятки, ініційовані вашою програмою, реєструються, а потім відображаються назад користувачеві. Ми поглибимось глибше в цей клас у цій документації.

<a name="configuration"></a>

## Конфігурація

`debug`варіант у вашому`config/app.php`Файл конфігурації визначає, скільки інформації про помилку насправді відображається користувачеві. За замовчуванням ця опція встановлена ​​відповідно до значення`APP_DEBUG`змінної середовища, яка зберігається у вашому`.env`файл.

Для місцевого розвитку слід встановити`APP_DEBUG`середовище змінна до`true`. У вашому виробничому середовищі це значення повинно бути завжди`false`. Якщо для значення встановлено значення`true`у виробництві ви ризикуєте розкрити чутливі значення конфігурації кінцевим користувачам вашої програми.

<a name="the-exception-handler"></a>

## Обробник винятків

<a name="reporting-exceptions"></a>

### Звітування про винятки

Усі винятки обробляє`App\Exceptions\Handler`клас. Цей клас містить`register`метод, де ви можете зареєструвати користувальницькі виклики репортерів та рендерів. Ми детально розглянемо кожне з цих понять. Звітування про винятки використовується для реєстрації винятків або надсилання їх до зовнішньої служби, наприклад[Спалах](https://flareapp.io),[Багснаг](https://bugsnag.com)або[Вартовий](https://github.com/getsentry/sentry-laravel). За замовчуванням винятки реєструватимуться на основі вашого[лісозаготівля](/docs/{{version}}/logging)конфігурації. Однак ви можете реєструвати винятки, як завгодно.

Наприклад, якщо вам потрібно повідомляти про різні типи винятків різними способами, ви можете використовувати файл`reportable`метод реєстрації Закриття, яке слід виконати, коли потрібно повідомити про виняток даного типу. Laravel визначить, який тип виключення подано у звіті про закриття, вивчивши підказку про закриття:

    use App\Exceptions\CustomException;

    /**
     * Register the exception handling callbacks for the application.
     *
     * @return void
     */
    public function register()
    {
        $this->reportable(function (CustomException $e) {
            //
        });
    }

Коли ви реєструєте користувальницький виклик звітування за допомогою`reportable`методу, Laravel все одно реєструватиме виняток, використовуючи конфігурацію ведення журналу за промовчанням для програми. Якщо ви хочете зупинити розповсюдження винятку до стеку реєстрації за замовчуванням, ви можете використовувати`stop`метод при визначенні вашого звітування про зворотний дзвінок або повернення`false`із зворотного дзвінка:

    $this->reportable(function (CustomException $e) {
        //
    })->stop();

    $this->reportable(function (CustomException $e) {
        return false;
    });

> {tip} Щоб налаштувати звітування про винятки для даного винятку, ви також можете розглянути можливість використання[звітні винятки](/docs/{{version}}/errors#renderable-exceptions)

<a name="global-log-context"></a>

#### Глобальний контекст журналу

Якщо доступно, Laravel автоматично додає ідентифікатор поточного користувача до повідомлення журналу кожного винятку як контекстні дані. Ви можете визначити свої власні глобальні контекстні дані, замінивши`context`метод вашої програми`App\Exceptions\Handler`клас. Ця інформація буде включена в повідомлення журналу кожного винятку, написане вашим додатком:

    /**
     * Get the default context variables for logging.
     *
     * @return array
     */
    protected function context()
    {
        return array_merge(parent::context(), [
            'foo' => 'bar',
        ]);
    }

<a name="the-report-helper"></a>

#### `report`Помічник

Іноді вам може знадобитися повідомити про виняток, але продовжувати обробляти поточний запит.`report`допоміжна функція дозволяє швидко повідомити про виняток, використовуючи ваш обробник винятків, не відображаючи сторінку помилки:

    public function isValid($value)
    {
        try {
            // Validate the value...
        } catch (Throwable $e) {
            report($e);

            return false;
        }
    }

<a name="ignoring-exceptions-by-type"></a>

#### Ігнорування винятків за типами

`$dontReport`властивість обробника винятків містить масив типів винятків, які не реєструватимуться. Наприклад, винятки, спричинені помилками 404, а також кілька інших типів помилок, не записуються до ваших журналів. За необхідності ви можете додати до цього масиву інші типи винятків:

    /**
     * A list of the exception types that should not be reported.
     *
     * @var array
     */
    protected $dontReport = [
        \Illuminate\Auth\AuthenticationException::class,
        \Illuminate\Auth\Access\AuthorizationException::class,
        \Symfony\Component\HttpKernel\Exception\HttpException::class,
        \Illuminate\Database\Eloquent\ModelNotFoundException::class,
        \Illuminate\Validation\ValidationException::class,
    ];

<a name="rendering-exceptions"></a>

### Візуалізація винятків

За замовчуванням обробник винятків Laravel перетворить винятки на відповідь HTTP для вас. Однак ви можете зареєструвати спеціальне закриття візуалізації для винятків певного типу. Ви можете досягти цього за допомогою`renderable`метод обробника винятків. Laravel визначить, який тип виключення робить Закриття, вивчивши підказку про Закриття:

    use App\Exceptions\CustomException;

    /**
     * Register the exception handling callbacks for the application.
     *
     * @return void
     */
    public function register()
    {
        $this->renderable(function (CustomException $e, $request) {
            return response()->view('errors.custom', [], 500);
        });
    }

<a name="renderable-exceptions"></a>

### Звітні та візуальні винятки

Замість перевірки типу винятків у обробнику винятків`report`і`render`методи, які ви можете визначити`report`і`render`методів безпосередньо на власний виняток. Коли ці методи існують, їх буде автоматично викликати фреймворк:

    <?php

    namespace App\Exceptions;

    use Exception;

    class RenderException extends Exception
    {
        /**
         * Report the exception.
         *
         * @return void
         */
        public function report()
        {
            //
        }

        /**
         * Render the exception into an HTTP response.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function render($request)
        {
            return response(...);
        }
    }

Якщо ваш виняток містить власну логіку звітування, яка виникає лише при дотриманні певних умов, можливо, вам доведеться доручити Laravel повідомляти про виняток за допомогою конфігурації обробки винятків за замовчуванням. Для цього ви можете повернутися`false`з винятків`report`метод:

    /**
     * Report the exception.
     *
     * @return bool|void
     */
    public function report()
    {
        // Determine if the exception needs custom reporting...

        return false;
    }

> {tip} Ви можете ввести натяк на будь-які необхідні залежності`report`метод, і вони будуть автоматично введені в метод методом Ларавеля[службовий контейнер](/docs/{{version}}/container).

<a name="http-exceptions"></a>

## Винятки HTTP

Деякі винятки описують коди помилок HTTP із сервера. Наприклад, це може бути помилка "сторінку не знайдено" (404), "несанкціонована помилка" (401) або навіть помилка розробника, що згенерувала 500. Для того, щоб згенерувати таку відповідь з будь-якої точки вашої програми, ви можете використовувати`abort`помічник:

    abort(404);

<a name="custom-http-error-pages"></a>

### Спеціальні сторінки помилок HTTP

Laravel дозволяє легко відображати власні сторінки помилок для різних кодів стану HTTP. Наприклад, якщо ви хочете налаштувати сторінку помилки для 404 кодів стану HTTP, створіть`resources/views/errors/404.blade.php`. Цей файл буде подано для всіх 404 помилок, згенерованих вашим додатком. Представлення даних у цьому каталозі мають бути названі відповідно до коду стану HTTP, якому вони відповідають.`HttpException`інстанція, піднята`abort`функція буде передана поданню як`$exception`змінна:

    <h2>{{ $exception->getMessage() }}</h2>

Ви можете опублікувати шаблони сторінок помилок Laravel за допомогою`vendor:publish`Ремісниче командування. Після опублікування шаблонів ви можете налаштувати їх на свій смак:

    php artisan vendor:publish --tag=laravel-errors
