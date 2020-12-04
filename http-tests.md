# Тести HTTP

-   [Вступ](#introduction)
    -   [Налаштування заголовків запитів](#customizing-request-headers)
    -   [Cookies](#cookies)
    -   [Відповіді на Debugging](#debugging-responses)
-   [Сесія / Аутентифікація](#session-and-authentication)
-   [Тестування JSON API](#testing-json-apis)
-   [Тестування завантаження файлів](#testing-file-uploads)
-   [Перевірка Views](#testing-views)
-   [Доступні Assertions](#available-assertions)
    -   [Assertions відповіді](#response-assertions)
    -   [Assertions автентифікації](#authentication-assertions)

<a name="introduction"></a>

## Вступ

Laravel пропонує дуже вільний API для надсилання HTTP-запитів до вашого додатку та вивчення результатів. Наприклад, погляньте на тест функції, визначений нижче:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * A basic test example.
         *
         * @return void
         */
        public function testBasicTest()
        {
            $response = $this->get('/');

            $response->assertStatus(200);
        }
    }

`get`метод робить a`GET`запит до програми, тоді як`assertStatus`метод стверджує, що повернута відповідь повинна мати вказаний код стану HTTP. На додаток до цього простого Assertions, Laravel також містить безліч Assertions для перевірки заголовків відповідей, вмісту, структури JSON тощо.

<a name="customizing-request-headers"></a>

### Налаштування заголовків запитів

Ви можете використовувати`withHeaders`метод для налаштування заголовків запиту до того, як його буде надіслано до програми. Це дозволяє додавати будь-які власні заголовки, які ви хочете, до запиту:

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->withHeaders([
                'X-Header' => 'Value',
            ])->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(201)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }

> {tip} Проміжне програмне забезпечення CSRF автоматично вимикається під час запуску тестів.

<a name="cookies"></a>

### Cookies

Ви можете використовувати`withCookie`або`withCookies`методи встановлення значень файлів cookie перед поданням запиту.`withCookie`метод приймає як два аргументи ім'я та значення файлу cookie, тоді як файл`withCookies`метод приймає масив пар ім'я / значення:

    <?php

    class ExampleTest extends TestCase
    {
        public function testCookies()
        {
            $response = $this->withCookie('color', 'blue')->get('/');

            $response = $this->withCookies([
                'color' => 'blue',
                'name' => 'Taylor',
            ])->get('/');
        }
    }

<a name="debugging-responses"></a>

### Відповіді на Debugging

Після подання запиту на тестування до вашої програми,`dump`,`dumpHeaders`, і`dumpSession`методи можуть бути використані для вивчення та Debugging вмісту відповіді:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * A basic test example.
         *
         * @return void
         */
        public function testBasicTest()
        {
            $response = $this->get('/');

            $response->dumpHeaders();

            $response->dumpSession();

            $response->dump();
        }
    }

<a name="session-and-authentication"></a>

## Сесія / Аутентифікація

Laravel надає кілька помічників для роботи з сеансом під час тестування HTTP. По-перше, ви можете встановити дані сеансу для заданого масиву, використовуючи`withSession`метод. Це корисно для завантаження сеансу даними перед поданням запиту до вашої програми:

    <?php

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $response = $this->withSession(['foo' => 'bar'])
                             ->get('/');
        }
    }

Одним із загальних видів використання сеансу є підтримка стану для аутентифікованого користувача.`actingAs`helper метод забезпечує простий спосіб автентифікації даного користувача як поточного користувача. Наприклад, ми можемо використовувати a[модельний Factory](/docs/{{version}}/database-testing#writing-factories)для створення та автентифікації користувача:

    <?php

    use App\Models\User;

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $user = User::factory()->create();

            $response = $this->actingAs($user)
                             ->withSession(['foo' => 'bar'])
                             ->get('/');
        }
    }

Ви також можете вказати, який захист слід використовувати для автентифікації даного користувача, передавши ім'я охорони як другий аргумент`actingAs`метод:

    $this->actingAs($user, 'api')

<a name="testing-json-apis"></a>

## Тестування JSON API

Laravel також надає кілька помічників для тестування API JSON та їх відповідей. Наприклад,`json`,`getJson`,`postJson`,`putJson`,`patchJson`,`deleteJson`, і`optionsJson`методи можуть бути використані для видачі запитів JSON з різними HTTP-дієсловами. Ви також можете легко передавати дані та заголовки цим методам. Для початку давайте напишемо тест, щоб скласти`POST`запит до`/user`і стверджуйте, що очікувані дані були повернуті:

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->postJson('/user', ['name' => 'Sally']);

            $response
                ->assertStatus(201)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }

> {tip} The`assertJson`метод перетворює відповідь на масив і використовує`PHPUnit::assertArraySubset`щоб переконатися, що даний масив існує у відповіді JSON, що повертається додатком. Отже, якщо у відповіді JSON є інші властивості, цей тест все одно пройде, доки присутній даний фрагмент.

Крім того, дані відповіді JSON можуть бути доступні як змінні масиву відповіді:

    $this->assertTrue($response['created']);

<a name="verifying-exact-match"></a>

### Перевірка точного збігу JSON

Якщо ви хочете перевірити, що даний масив є**точний**для JSON, що повертається додатком, вам слід використовувати`assertExactJson`метод:

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(201)
                ->assertExactJson([
                    'created' => true,
                ]);
        }
    }

<a name="verifying-json-paths"></a>

### Перевірка шляхів JSON

Якщо ви хочете перевірити, що відповідь JSON містить деякі дані за вказаним шляхом, вам слід скористатися`assertJsonPath`метод:

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(201)
                ->assertJsonPath('team.owner.name', 'foo');
        }
    }

<a name="testing-file-uploads"></a>

## Тестування завантаження файлів

`Illuminate\Http\UploadedFile`клас забезпечує a`fake`метод, який може бути використаний для створення фіктивних файлів або зображень для тестування. Це в поєднанні з`Storage` facade's `fake`метод значно спрощує перевірку завантаження файлів. Наприклад, ви можете поєднати ці дві функції, щоб легко перевірити форму для завантаження аватара:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function testAvatarUpload()
        {
            Storage::fake('avatars');

            $file = UploadedFile::fake()->image('avatar.jpg');

            $response = $this->json('POST', '/avatar', [
                'avatar' => $file,
            ]);

            // Assert the file was stored...
            Storage::disk('avatars')->assertExists($file->hashName());

            // Assert a file does not exist...
            Storage::disk('avatars')->assertMissing('missing.jpg');
        }
    }

<a name="fake-file-customization"></a>

#### Підроблені налаштування файлів

При створенні файлів за допомогою`fake`методом, ви можете вказати ширину, висоту та розмір зображення, щоб краще перевірити ваші правила перевірки:

    UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);

На додаток до створення зображень, ви можете створювати файли будь-якого іншого типу за допомогою`create`метод:

    UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);

Якщо потрібно, ви можете здати`$mimeType`аргумент методу для явного визначення типу MIME, який повинен бути повернутий файлом:

    UploadedFile::fake()->create('document.pdf', $sizeInKilobytes, 'application/pdf');

<a name="testing-views"></a>

## Перевірка Views

Laravel дозволяє візуалізувати View ізольовано, не роблячи змодельований HTTP-запит до програми. Для цього ви можете використовувати`view`у вашому тесті.`view`метод приймає ім'я View та необов'язковий масив даних. Метод повертає екземпляр`Illuminate\Testing\TestView`, який пропонує кілька методів, щоб зручно висловлювати Assertions про вміст View:

    public function testWelcomeView()
    {
        $view = $this->view('welcome', ['name' => 'Taylor']);

        $view->assertSee('Taylor');
    }

`TestView`об'єкт забезпечує наступні методи Assertions:`assertSee`,`assertSeeInOrder`,`assertSeeText`,`assertSeeTextInOrder`,`assertDontSee`, і`assertDontSeeText`.

Якщо потрібно, ви можете отримати необроблений, відтворений вміст View, додавши файл`TestView`екземпляр до рядка:

    $contents = (string) $this->view('welcome');

<a name="sharing-errors"></a>

#### Помилки спільного використання

Деякі перегляди можуть залежати від помилок, які поділяються у загальній сумці помилок, наданій Laravel. Щоб зволожити пакет помилок повідомленнями про помилки, ви можете використовувати`withViewErrors`метод:

    $view = $this->withViewErrors([
        'name' => ['Please provide a valid name.']
    ])->view('form');

    $view->assertSee('Please provide a valid name.');

<a name="rendering-blade-components"></a>

#### Візуалізація леза та компонентів

Якщо потрібно, ви можете використовувати`blade`метод для оцінки та візуалізації необробленого рядка Blade. Подобається`view`метод,`blade`метод повертає екземпляр`Illuminate\Testing\TestView`:

    $view = $this->blade(
        '<x-component :name="$name" />',
        ['name' => 'Taylor']
    );

    $view->assertSee('Taylor');

Ви можете використовувати`component`метод оцінки та візуалізації компонента леза. Подобається`view`метод,`component`метод повертає екземпляр`Illuminate\Testing\TestView`:

    $view = $this->component(Profile::class, ['name' => 'Taylor']);

    $view->assertSee('Taylor');

<a name="available-assertions"></a>

## Доступні Assertions

<a name="response-assertions"></a>

### Assertions відповіді

Laravel пропонує різноманітні власні методи Assertions для вашого[PHPUnit](https://phpunit.de/)тести характеристик. До цих Assertiors можна отримати доступ у відповіді, що повертається з`json`,`get`,`post`,`put`, і`delete`методи випробування:

<style>
    .collection-method-list > p {
        column-count: 2; -moz-column-count: 2; -webkit-column-count: 2;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="collection-method-list" markdown="1">

[assertCookie](#assert-cookie)[assertCookieExpired](#assert-cookie-expired)[затвердити Файл cookie не закінчився](#assert-cookie-not-expired)[assertCookieMissing](#assert-cookie-missing)[assertCreated](#assert-created)[assertDontSee](#assert-dont-see)[assertDontSeeText](#assert-dont-see-text)[assertExactJson](#assert-exact-json)[стверджуватиЗаборонено](#assert-forbidden)[assertHeader](#assert-header)[assertHeaderMissing](#assert-header-missing)[assertJson](#assert-json)[assertJsonCount](#assert-json-count)[assertJsonFragment](#assert-json-fragment)[assertJsonMissing](#assert-json-missing)[assertJsonMissingExact](#assert-json-missing-exact)[assertJson Відсутні помилки перевірки](#assert-json-missing-validation-errors)[assertJsonPath](#assert-json-path)[assertJsonStructure](#assert-json-structure)[assertJsonValidationErrors](#assert-json-validation-errors)[assertLocation](#assert-location)[assertNoContent](#assert-no-content)[assertNotFound](#assert-not-found)[assertOk](#assert-ok)[assertPlainCookie](#assert-plain-cookie)[assertRedirect](#assert-redirect)[затвердитиДив](#assert-see)[assertSeeInOrder](#assert-see-in-order)[assertSeeText](#assert-see-text)[assertSeeTextInOrder](#assert-see-text-in-order)[assertSessionHas](#assert-session-has)[assertSessionHasInput](#assert-session-has-input)[assertSessionHasAll](#assert-session-has-all)[assertSessionHasErrors](#assert-session-has-errors)[assertSessionHasErrors](#assert-session-has-errors-in)[assertSessionHasErrors](#assert-session-has-no-errors)[assertSessionDoesntHaveErrors](#assert-session-doesnt-have-errors)[assertSessionMissing](#assert-session-missing)[assertStatus](#assert-status)[assertSuccessful](#assert-successful)[assertUauthorized](#assert-unauthorized)[assertViewHas](#assert-view-has)[assertViewHasAll](#assert-view-has-all)[assertViewIs](#assert-view-is)[assertViewMissing](#assert-view-missing)

</div>

<a name="assert-cookie"></a>

#### assertCookie

Стверджуйте, що відповідь містить вказаний файл cookie:

    $response->assertCookie($cookieName, $value = null);

<a name="assert-cookie-expired"></a>

#### assertCookieExpired

Стверджуйте, що відповідь містить вказаний файл cookie і термін його дії минув:

    $response->assertCookieExpired($cookieName);

<a name="assert-cookie-not-expired"></a>

#### затвердити Файл cookie не закінчився

Стверджуйте, що відповідь містить вказаний файл cookie і термін його дії не минув:

    $response->assertCookieNotExpired($cookieName);

<a name="assert-cookie-missing"></a>

#### assertCookieMissing

Стверджуємо, що відповідь не містить вказаний файл cookie:

    $response->assertCookieMissing($cookieName);

<a name="assert-created"></a>

#### assertCreated

Стверджуйте, що відповідь має 201 код стану:

    $response->assertCreated();

<a name="assert-dont-see"></a>

#### assertDontSee

Стверджуйте, що заданий рядок не міститься у відповіді. Це Assertions автоматично уникне даного рядка, якщо ви не передасте другий аргумент`false`:

    $response->assertDontSee($value, $escaped = true);

<a name="assert-dont-see-text"></a>

#### assertDontSeeText

Стверджуйте, що заданий рядок не міститься в тексті відповіді. Це Assertions автоматично уникне даного рядка, якщо ви не передасте другий аргумент`false`:

    $response->assertDontSeeText($value, $escaped = true);

<a name="assert-exact-json"></a>

#### assertExactJson

Стверджуйте, що відповідь містить точну відповідність даних JSON:

    $response->assertExactJson(array $data);

<a name="assert-forbidden"></a>

#### стверджуватиЗаборонено

Стверджуйте, що відповідь має заборонений (403) код стану:

    $response->assertForbidden();

<a name="assert-header"></a>

#### assertHeader

Стверджуйте, що вказаний заголовок присутній у відповіді:

    $response->assertHeader($headerName, $value = null);

<a name="assert-header-missing"></a>

#### assertHeaderMissing

Стверджуйте, що вказаного заголовка немає у відповіді:

    $response->assertHeaderMissing($headerName);

<a name="assert-json"></a>

#### assertJson

Стверджуємо, що відповідь містить дані JSON:

    $response->assertJson(array $data, $strict = false);

<a name="assert-json-count"></a>

#### assertJsonCount

Стверджуємо, що відповідь JSON має масив із очікуваною кількістю елементів за даним ключем:

    $response->assertJsonCount($count, $key = null);

<a name="assert-json-fragment"></a>

#### assertJsonFragment

Стверджуємо, що відповідь містить вказаний фрагмент JSON:

    $response->assertJsonFragment(array $data);

<a name="assert-json-missing"></a>

#### assertJsonMissing

Стверджуємо, що відповідь не містить вказаний фрагмент JSON:

    $response->assertJsonMissing(array $data);

<a name="assert-json-missing-exact"></a>

#### assertJsonMissingExact

Стверджуємо, що відповідь не містить точного фрагмента JSON:

    $response->assertJsonMissingExact(array $data);

<a name="assert-json-missing-validation-errors"></a>

#### assertJson Відсутні помилки перевірки

Стверджуємо, що відповідь не має помилок перевірки JSON для даних ключів:

    $response->assertJsonMissingValidationErrors($keys);

<a name="assert-json-path"></a>

#### assertJsonPath

Стверджуйте, що відповідь містить дані щодо вказаного шляху:

    $response->assertJsonPath($path, array $data, $strict = true);

<a name="assert-json-structure"></a>

#### assertJsonStructure

Стверджуємо, що відповідь має задану структуру JSON:

    $response->assertJsonStructure(array $structure);

<a name="assert-json-validation-errors"></a>

#### assertJsonValidationErrors

Стверджуємо, що відповідь має дані про помилки перевірки JSON:

    $response->assertJsonValidationErrors(array $data);

<a name="assert-location"></a>

#### assertLocation

Стверджуйте, що відповідь має задане значення URI в`Location`заголовок:

    $response->assertLocation($uri);

<a name="assert-no-content"></a>

#### assertNoContent

Стверджуйте, що відповідь має заданий код стану і не містить вмісту.

    $response->assertNoContent($status = 204);

<a name="assert-not-found"></a>

#### assertNotFound

Стверджуйте, що відповідь має не знайдений код стану:

    $response->assertNotFound();

<a name="assert-ok"></a>

#### assertOk

Стверджуйте, що відповідь має код стану 200:

    $response->assertOk();

<a name="assert-plain-cookie"></a>

#### assertPlainCookie

Стверджуємо, що відповідь містить вказаний файл cookie (незашифрований):

    $response->assertPlainCookie($cookieName, $value = null);

<a name="assert-redirect"></a>

#### assertRedirect

Стверджуйте, що відповідь є переспрямуванням на даний URI:

    $response->assertRedirect($uri);

<a name="assert-see"></a>

#### затвердитиДив

Стверджуйте, що заданий рядок міститься у відповіді. Це Assertions автоматично уникне даного рядка, якщо ви не передасте другий аргумент`false`:

    $response->assertSee($value, $escaped = true);

<a name="assert-see-in-order"></a>

#### assertSeeInOrder

Стверджуйте, що задані рядки містяться в порядку відповіді у відповіді. Це Assertions автоматично уникне даних рядків, якщо ви не передасте другий аргумент`false`:

    $response->assertSeeInOrder(array $values, $escaped = true);

<a name="assert-see-text"></a>

#### assertSeeText

Стверджуйте, що заданий рядок міститься в тексті відповіді. Це Assertions автоматично уникне даного рядка, якщо ви не передасте другий аргумент`false`:

    $response->assertSeeText($value, $escaped = true);

<a name="assert-see-text-in-order"></a>

#### assertSeeTextInOrder

Стверджуйте, що дані рядки містяться в порядку відповіді у тексті відповіді. Це Assertions автоматично уникне даних рядків, якщо ви не передасте другий аргумент`false`:

    $response->assertSeeTextInOrder(array $values, $escaped = true);

<a name="assert-session-has"></a>

#### assertSessionHas

Стверджуйте, що сеанс містить вказаний фрагмент даних:

    $response->assertSessionHas($key, $value = null);

<a name="assert-session-has-input"></a>

#### assertSessionHasInput

Стверджуємо, що сеанс має задане значення у прошитому масиві вводу:

    $response->assertSessionHasInput($key, $value = null);

<a name="assert-session-has-all"></a>

#### assertSessionHasAll

Стверджуємо, що сеанс має заданий список значень:

    $response->assertSessionHasAll(array $data);

<a name="assert-session-has-errors"></a>

#### assertSessionHasErrors

Стверджуйте, що сеанс містить помилку для даного`$keys`. Якщо`$keys`є асоціативним масивом, стверджуємо, що сеанс містить певне повідомлення про помилку (значення) для кожного поля (ключа):

    $response->assertSessionHasErrors(array $keys, $format = null, $errorBag = 'default');

<a name="assert-session-has-errors-in"></a>

#### assertSessionHasErrors

Стверджуйте, що сеанс містить помилку для даного`$keys`, у межах певного пакету помилок. Якщо`$keys`є асоціативним масивом, стверджуємо, що сеанс містить певне повідомлення про помилку (значення) для кожного поля (ключа), що знаходиться в пакеті помилок:

    $response->assertSessionHasErrorsIn($errorBag, $keys = [], $format = null);

<a name="assert-session-has-no-errors"></a>

#### assertSessionHasErrors

Стверджуйте, що сеанс не має помилок:

    $response->assertSessionHasNoErrors();

<a name="assert-session-doesnt-have-errors"></a>

#### assertSessionDoesntHaveErrors

Стверджуємо, що сеанс не має помилок для даних ключів:

    $response->assertSessionDoesntHaveErrors($keys = [], $format = null, $errorBag = 'default');

<a name="assert-session-missing"></a>

#### assertSessionMissing

Стверджуйте, що сеанс не містить заданий ключ:

    $response->assertSessionMissing($key);

<a name="assert-status"></a>

#### assertStatus

Стверджуйте, що відповідь має заданий код:

    $response->assertStatus($code);

<a name="assert-successful"></a>

#### assertSuccessful

Стверджуйте, що відповідь має успішний (> = 200 та &lt;300) код стану:

    $response->assertSuccessful();

<a name="assert-unauthorized"></a>

#### assertUauthorized

Стверджуйте, що відповідь має несанкціонований (401) код стану:

    $response->assertUnauthorized();

<a name="assert-view-has"></a>

#### assertViewHas

Стверджуйте, що View відповіді отримало частину даних:

    $response->assertViewHas($key, $value = null);

Крім того, дані перегляду можуть бути доступні як змінні масиву у відповіді:

    $this->assertEquals('Taylor', $response['name']);

<a name="assert-view-has-all"></a>

#### assertViewHasAll

Стверджуйте, що View відповідей має заданий список даних:

    $response->assertViewHasAll(array $data);

<a name="assert-view-is"></a>

#### assertViewIs

Стверджуйте, що даний вигляд було повернуто маршрутом:

    $response->assertViewIs($value);

<a name="assert-view-missing"></a>

#### assertViewMissing

Запевнити, що у поданні відповіді відсутній фрагмент пов’язаних даних:

    $response->assertViewMissing($key);

<a name="authentication-assertions"></a>

### Ствердження автентифікації

Laravel також надає різноманітні Assertions щодо вашої автентифікації[PHPUnit](https://phpunit.de/)тести функцій:

| Метод                                                                 | Опис                                               |
| --------------------------------------------------------------------- | -------------------------------------------------- |
| `$this->assertAuthenticated($guard = null);`                          | Стверджуйте, що користувач автентифікований.       |
| `$this->assertGuest($guard = null);`                                  | Стверджуйте, що користувач не автентифікований.    |
| `$this->assertAuthenticatedAs($user, $guard = null);`                 | Стверджуйте, що даний користувач автентифікований. |
| `$this->assertCredentials(array $credentials, $guard = null);`        | Стверджуйте, що дані облікові дані є дійсними.     |
| `$this->assertInvalidCredentials(array $credentials, $guard = null);` | Стверджуйте, що дані облікові дані недійсні.       |
