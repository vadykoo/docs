# Встановлення

-   [Встановлення](#installation)
    -   [Вимоги до сервера](#server-requirements)
    -   [Встановлення Laravel](#installing-laravel)
    -   [Конфігурація](#configuration)
-   [Конфігурація веб-сервера](#web-server-configuration)
    -   [Конфігурація каталогу](#directory-configuration)
    -   [Гарні URL-адреси](#pretty-urls)

<a name="installation"></a>

## Установка

<a name="server-requirements"></a>

### Вимоги до сервера

Фреймворк Laravel має кілька системних вимог. Всі ці вимоги задовольняються[ Homestead  Laravel](/docs/{{version}}/homestead)віртуальної машини, тому настійно рекомендується використовувати Homestead як місцеве середовище розробки Laravel.

Однак, якщо ви не використовуєте Homestead, вам потрібно переконатися, що ваш сервер відповідає таким вимогам:

<div class="content-list" markdown="1">
- PHP >= 7.3
- BCMath PHP Extension
- Ctype PHP Extension
- Fileinfo PHP Extension
- JSON PHP Extension
- Mbstring PHP Extension
- OpenSSL PHP Extension
- PDO PHP Extension
- Tokenizer PHP Extension
- XML PHP Extension
</div>

<a name="installing-laravel"></a>

### Встановлення Laravel

Laravel використовує[Composer](https://getcomposer.org)для управління його залежностями. Отже, перед використанням Laravel переконайтеся, що на вашому комп'ютері встановлено Composer.

<a name="via-laravel-installer"></a>

#### Через інсталятор Laravel

Спочатку завантажте інсталятор Laravel за допомогою Composer:

    composer global require laravel/installer

Переконайтеся, що ви помістили каталог Composer у свій`$PATH`так що виконуваний файл laravel може знаходитись у вашій системі. Цей каталог існує в різних місцях залежно від вашої операційної системи; Найчастіше в таких місцях:

<div class="content-list" markdown="1">
- macOS: `$HOME/.composer/vendor/bin`
- Windows: `%USERPROFILE%\AppData\Roaming\Composer\vendor\bin`
- GNU / Linux Distributions: `$HOME/.config/composer/vendor/bin` or `$HOME/.composer/vendor/bin`
</div>

Ви також можете знайти глобальний шлях встановлення composer, запустивши`composer global about`.

Після встановлення`laravel new`команда створить нову інсталяцію Laravel у вказаному каталозі. Наприклад,`laravel new blog`створить каталог з іменем`blog`що містить свіжу інсталяцію Laravel з усіма вже встановленими залежностями Laravel:

    laravel new blog

> {tip} Хочете створити проект Laravel із входом, реєстрацією та іншими функціями, він вже створеними для вас? Погляньте[Laravel Jetstream](https://jetstream.laravel.com).

<a name="via-composer-create-project"></a>

#### Через проект Composer Create

Ви також можете встановити Laravel, за допомогою Composer`create-project`команди у терміналі:

    composer create-project --prefer-dist laravel/laravel blog

<a name="local-development-server"></a>

#### Локальний сервер розробки

Якщо у вас PHP встановлений локально, і ви хочете використовувати вбудований сервер розробки PHP для обслуговування своєї програми, ви можете використовувати`serve` artisan командування. Ця команда запустить сервер розробки в`http://localhost:8000`:

    php artisan serve

Більш надійні варіанти локальної розробки доступні через[ Homestead ](/docs/{{version}}/homestead)і[Камердинер](/docs/{{version}}/valet).

<a name="configuration"></a>

### Конфігурація

<a name="public-directory"></a>

#### Публічний каталог

Після встановлення Laravel вам слід налаштувати документ / веб-корінь веб-сервера на`public`каталог.`index.php`у цьому каталозі виконує функції контролера для всіх HTTP-запитів, що надходять у вашу програму.

<a name="configuration-files"></a>

#### Файли конфігурації

Всі файли конфігурації фреймворку Laravel зберігаються в`config`каталозі. Кожена опція задокументована, тож сміливо переглядайте файли та знайомтесь із доступними вам параметрами.

<a name="directory-permissions"></a>

#### Дозволи каталогу

Після встановлення Laravel вам може знадобитися налаштувати деякі дозволи. Папки всередині`storage`та`bootstrap/cache`каталоги повинні бути записані на вашому веб-сервері, інакше Laravel не працюватиме. Якщо ви використовуєте[ Homestead ](/docs/{{version}}/homestead)віртуальної машини, ці дозволи вже мають бути встановлені.

<a name="application-key"></a>

#### Ключ програми

Наступним, що вам слід зробити після встановлення Laravel, є встановлення ключа програми на випадковий рядок. Якщо ви встановили Laravel через Composer або інсталятор Laravel, цей ключ для вас уже встановив`php artisan key:generate`команди.

Зазвичай цей рядок повинен мати довжину 32 символи. Клавішу можна встановити в`.env`файл середовища. Якщо ви не скопіювали файл`.env.example`файл у новий файл з іменем`.env`, ви повинні це зробити зараз.**Якщо ключ програми не встановлений, ваші сеанси користувача та інші зашифровані дані не будуть захищені!**

<a name="additional-configuration"></a>

#### Додаткова конфігурація

Laravel майже не потребує жодної іншої конфігурації. Ви можете розпочати розробку! Однак, можливо, ви захочете переглянути`config/app.php`файл та його документація. Він містить кілька варіантів, таких як`timezone`і`locale`які ви можете змінити відповідно до вашої заявки.

Ви також можете налаштувати кілька додаткових компонентів Laravel, таких як:

<div class="content-list" markdown="1">
- [Cache](/docs/{{version}}/cache#configuration)
- [Database](/docs/{{version}}/database#configuration)
- [Session](/docs/{{version}}/session#configuration)
</div>

<a name="web-server-configuration"></a>

## Конфігурація веб-сервера

<a name="directory-configuration"></a>

### Конфігурація каталогу

Laravel завжди повинен подаватися з кореня "веб-каталогу", налаштованого для вашого веб-сервера. Не слід намагатися обслуговувати додаток Laravel із підкаталогу "веб-каталогу". Спроба зробити це може відкрити конфіденційні файли, що знаходяться у вашій програмі.

<a name="pretty-urls"></a>

### Гарні URL-адреси

<a name="apache"></a>

#### Апачі

Laravel включає a`public/.htaccess`файл, який використовується для надання URL-адрес без`index.php`передній контролер на шляху. Перш ніж подавати Laravel разом з Apache, не забудьте включити`mod_rewrite`модуль так`.htaccess`файл буде виконаний сервером.

Якщо`.htaccess`файл, який постачається з Laravel, не працює з вашою установкою Apache, спробуйте наступну альтернативу:

    Options +FollowSymLinks -Indexes
    RewriteEngine On

    RewriteCond %{HTTP:Authorization} .
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]

<a name="nginx"></a>

#### Nginx

Якщо ви використовуєте Nginx, наступна директива у конфігурації вашого сайту спрямовуватиме всі запити до`index.php`передній контролер:

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

При використанні[ Homestead ](/docs/{{version}}/homestead)або[Камердинер](/docs/{{version}}/valet), гарні URL-адреси будуть автоматично налаштовані.
