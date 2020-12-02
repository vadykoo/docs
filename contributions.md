git 11a0798a095fefdd3c1e9627c4a209ab069285c5

---
# Керівництво по внеску

-   [Звіти про помилки](#bug-reports)
-   [Питання підтримки](#support-questions)
-   [Основна дискусія щодо розвитку](#core-development-discussion)
-   [Яке відділення?](#which-branch)
-   [Складені активи](#compiled-assets)
-   [Уразливості безпеки](#security-vulnerabilities)
-   [Стиль кодування](#coding-style)
    -   [PHPDoc](#phpdoc)
    -   [StyleCI](#styleci)
-   [Норми поведінки](#code-of-conduct)

<a name="bug-reports"></a>

## Звіти про помилки

Щоб заохотити активну співпрацю, Laravel настійно заохочує запити на виклик, а не лише повідомлення про помилки. "Звіти про помилки" також можуть надсилатися у формі запиту на вилучення, що містить невдалий тест.

Однак, якщо ви подаєте звіт про помилку, ваша проблема повинна містити заголовок та чіткий опис проблеми. Ви також повинні включити якомога більше відповідної інформації та зразок коду, який демонструє проблему. Мета звіту про помилку - полегшити для себе - та інших - відтворення помилки та розробку виправлення.

Пам’ятайте, звіти про помилки створюються з надією, що люди з такою ж проблемою зможуть співпрацювати з вами над її вирішенням. Не чекайте, що звіт про помилку автоматично побачить будь-яку активність або що інші перескочать, щоб виправити це. Створення звіту про помилку допомагає собі та іншим почати шлях виправлення проблеми. Якщо ви хочете чіп, ви можете допомогти, виправивши[будь-які помилки, перелічені в наших програмах відстеження](https://github.com/issues?q=is%3Aopen+is%3Aissue+label%3Abug+user%3Alaravel).

Вихідний код Laravel управляється на GitHub, і для кожного з проектів Laravel є сховища:

<div class="content-list" markdown="1">
- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Art](https://github.com/laravel/art)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Dusk](https://github.com/laravel/dusk)
- [Laravel Cashier Stripe](https://github.com/laravel/cashier)
- [Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle)
- [Laravel Echo](https://github.com/laravel/echo)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Horizon](https://github.com/laravel/horizon)
- [Laravel Jetstream](https://github.com/laravel/jetstream)
- [Laravel Passport](https://github.com/laravel/passport)
- [Laravel Sanctum](https://github.com/laravel/sanctum)
- [Laravel Scout](https://github.com/laravel/scout)
- [Laravel Socialite](https://github.com/laravel/socialite)
- [Laravel Telescope](https://github.com/laravel/telescope)
- [Laravel Website](https://github.com/laravel/laravel.com-next)
</div>

<a name="support-questions"></a>

## Питання підтримки

Трекери випусків GitHub Laravel не призначені для надання допомоги або підтримки Laravel. Натомість використовуйте один із таких каналів:

<div class="content-list" markdown="1">
- [GitHub Discussions](https://github.com/laravel/framework/discussions)
- [Laracasts Forums](https://laracasts.com/discuss)
- [Laravel.io Forums](https://laravel.io/forum)
- [StackOverflow](https://stackoverflow.com/questions/tagged/laravel)
- [Discord](https://discordapp.com/invite/KxwQuKb)
- [Larachat](https://larachat.co)
- [IRC](https://webchat.freenode.net/?nick=artisan&channels=%23laravel&prompt=1)
</div>

<a name="core-development-discussion"></a>

## Основна дискусія щодо розвитку

Ви можете запропонувати нові функції або вдосконалення існуючої поведінки Laravel в Laravel Ideas[емісійна дошка](https://github.com/laravel/ideas/issues). Якщо ви пропонуєте нову функцію, будь ласка, будьте готові застосувати хоча б частину коду, який був би необхідний для завершення функції.

Неформальна дискусія щодо помилок, нових функцій та впровадження існуючих функцій відбувається в`#internals`канал[Сервер Laravel Discord](https://discordapp.com/invite/mPZNm7A). Тейлор Отуелл, співробітник Laravel, зазвичай присутній на каналі в робочі дні з 8:00 до 17:00 (UTC-06: 00 або Америка / Чикаго), а спорадично присутній на каналі в інший час.

<a name="which-branch"></a>

## Яке відділення?

**Всі**виправлення помилок слід надсилати до останньої стабільної гілки або до[поточна філія LTS](/docs/{{version}}/releases#support-policy). Виправлення помилок повинно відбуватися**ніколи**відправити до`master`, якщо вони не виправлять функції, які існують лише у майбутньому випуску

**Неповнолітні**особливості, які є**повністю назад сумісний**з поточним випуском може бути відправлений в останню стабільну гілку.

**Майор**нові функції завжди слід надсилати до`master`, що містить майбутній випуск.

Якщо ви не впевнені, чи відповідає ваша функція мажору чи мінору, будь ласка, запитайте Тейлора Отуелла в`#internals`канал[Сервер Laravel Discord](https://discordapp.com/invite/mPZNm7A).

<a name="compiled-assets"></a>

## Складені активи

Якщо ви подаєте зміну, яка вплине на скомпільований файл, такий як більшість файлів у`resources/css`або`resources/js`з`laravel/laravel`сховище, не фіксуйте компільовані файли. Через їх великі розміри, вони не можуть бути реально перевірені супроводжуючим. Це можна використати як спосіб ін’єкції шкідливого коду в Laravel. Для того, щоб оборонно запобігти цьому, всі скомпільовані файли будуть генеруватися та фіксуватися супровідниками Laravel.

<a name="security-vulnerabilities"></a>

## Уразливості безпеки

Якщо ви виявили вразливість системи безпеки в Laravel, надішліть електронний лист Тейлору Отуеллу за адресою<a href="mailto:taylor@laravel.com">taylor@laravel.com</a>. Усі уразливості системи безпеки будуть негайно усунені.

<a name="coding-style"></a>

## Стиль кодування

Ларавел слідує за[PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md)стандарт кодування та[PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md)стандарт автоматичного завантаження.

<a name="phpdoc"></a>

### PHPDoc

Below is an example of a valid Laravel documentation block. Note that the `@param`за атрибутом слідують два пробіли, тип аргументу, ще два пробіли і, нарешті, ім'я змінної:

    /**
     * Register a binding with the container.
     *
     * @param  string|array  $abstract
     * @param  \Closure|string|null  $concrete
     * @param  bool  $shared
     * @return void
     *
     * @throws \Exception
     */
    public function bind($abstract, $concrete = null, $shared = false)
    {
        //
    }

<a name="styleci"></a>

### StyleCI

Не хвилюйтеся, якщо ваш стиль коду не ідеальний\![StyleCI](https://styleci.io/)автоматично об’єднає будь-які виправлення стилів у сховище Laravel після об’єднання запитів на витягування. Це дозволяє зосередитись на змісті внеску, а не на стилі коду.

<a name="code-of-conduct"></a>

## Норми поведінки

The Laravel code of conduct is derived from the Ruby code of conduct. Any violations of the code of conduct may be reported to Taylor Otwell ([taylor@laravel.com](mailto:taylor@laravel.com)):

<div class="content-list" markdown="1">
- Participants will be tolerant of opposing views.
- Participants must ensure that their language and actions are free of personal attacks and disparaging personal remarks.
- When interpreting the words and actions of others, participants should always assume good intentions.
- Behavior that can be reasonably considered harassment will not be tolerated.
</div>
