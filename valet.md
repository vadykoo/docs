# Laravel Valet

-   [Вступ](#introduction)
    -   [Valet Or Homestead](#valet-or-homestead)
-   [Встановлення](#installation)
    -   [Оновлення](#upgrading)
-   [Обслуговування сайтів](#serving-sites)
    -   [Команда "Парк"](#the-park-command)
    -   [Команда "Посилання"](#the-link-command)
    -   [Захист сайтів за допомогою TLS](#securing-sites)
-   [Спільний доступ до сайтів](#sharing-sites)
-   [Змінні середовища, специфічні для сайту](#site-specific-environment-variables)
-   [Послуги проксі-сервера](#proxying-services)
-   [Спеціальні драйвери для камердинера](#custom-valet-drivers)
    -   [Місцеві водії](#local-drivers)
-   [Інші команди Valet](#other-valet-commands)
-   [Каталоги та файли Valet](#valet-directories-and-files)

<a name="introduction"></a>

## Introduction

Valet - середовище розробки Laravel для мінімалістів Mac. Ні бродяга, ні`/etc/hosts`файл. Ви навіть можете публічно ділитися своїми веб-сайтами за допомогою місцевих тунелів._Так, нам теж це подобається._

Laravel Valet налаштовує ваш Mac завжди працювати[Nginx](https://www.nginx.com/)у фоновому режимі, коли машина запускається. Потім, використовуючи[DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq), Valet проксі-сервери всі запити на`*.test`домену, щоб вказувати на сайти, встановлені на вашому локальному комп'ютері.

Іншими словами, надзвичайно швидке середовище розробки Laravel, яке використовує приблизно 7 МБ оперативної пам'яті. Valet не є повною заміною Vagrant або Homestead, але пропонує чудову альтернативу, якщо ви хочете гнучкі основи, віддаєте перевагу екстремальній швидкості або працюєте на машині з обмеженою кількістю оперативної пам'яті.

Незрозуміла підтримка Valet включає, але не обмежується:

<style>
    #valet-support > ul {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        line-height: 1.9;
    }
</style>

<div id="valet-support" markdown="1">
- [Laravel](https://laravel.com)
- [Lumen](https://lumen.laravel.com)
- [Bedrock](https://roots.io/bedrock/)
- [CakePHP 3](https://cakephp.org)
- [Concrete5](https://www.concrete5.org/)
- [Contao](https://contao.org/en/)
- [Craft](https://craftcms.com)
- [Drupal](https://www.drupal.org/)
- [ExpressionEngine](https://www.expressionengine.com/)
- [Jigsaw](https://jigsaw.tighten.co)
- [Joomla](https://www.joomla.org/)
- [Katana](https://github.com/themsaid/katana)
- [Kirby](https://getkirby.com/)
- [Magento](https://magento.com/)
- [OctoberCMS](https://octobercms.com/)
- [Sculpin](https://sculpin.io/)
- [Slim](https://www.slimframework.com)
- [Statamic](https://statamic.com)
- Static HTML
- [Symfony](https://symfony.com)
- [WordPress](https://wordpress.org)
- [Zend](https://framework.zend.com)
</div>

Тим не менш, ви можете продовжити службу Valet своїм[користувацькі драйвери](#custom-valet-drivers).

<a name="valet-or-homestead"></a>

### Valet Or Homestead

Як ви вже знаєте, Laravel пропонує[Садиба](/docs/{{version}}/homestead), ще одне місцеве середовище для розвитку Laravel. Хомстед та Валет відрізняються між собою за призначенням аудиторії та підходом до місцевого розвитку. Homestead пропонує цілу віртуальну машину Ubuntu з автоматизованою конфігурацією Nginx. Homestead - чудовий вибір, якщо ви хочете повністю віртуалізоване середовище розробки Linux або перебуваєте на Windows / Linux.

Valet підтримує лише Mac і вимагає встановлення PHP та сервера баз даних безпосередньо на локальній машині. Цього легко досягти за допомогою[Саморобний](https://brew.sh/)з командами типу`brew install php`і`brew install mysql`. Valet забезпечує надзвичайно швидке місцеве середовище розробки з мінімальним споживанням ресурсів, тому чудово підходить для розробників, яким потрібен лише PHP / MySQL і не потрібне повністю віртуалізоване середовище розробки.

І Valet, і Homestead - це чудовий вибір для налаштування середовища розробки Laravel. Який із них ви оберете, буде залежати від вашого особистого смаку та потреб вашої команди.

<a name="installation"></a>

## Встановлення

**Valet вимагає macOS і[Саморобний](https://brew.sh/). Перед установкою слід переконатися, що жодні інші програми, такі як Apache або Nginx, не прив’язують до порту 80 вашої локальної машини.**

<div class="content-list" markdown="1">
- Install or update [Homebrew](https://brew.sh/) to the latest version using `brew update`.
- Install PHP 8.0 using Homebrew via `brew install php`.
- Install [Composer](https://getcomposer.org).
- Install Valet with Composer via `composer global require laravel/valet`. Make sure the `~/.composer/vendor/bin` directory is in your system's "PATH".
- Run the `valet install` command. This will configure and install Valet and DnsMasq, and register Valet's daemon to launch when your system starts.
</div>

Після встановлення програми Valet спробуйте пінгувати будь-яку`*.test`домену на вашому терміналі за допомогою команди, наприклад`ping foobar.test`. Якщо Valet встановлено правильно, ви побачите, як цей домен відповідає`127.0.0.1`.

Valet автоматично запускає свій демон щоразу, коли ваша машина завантажується. Не потрібно бігати`valet start`або`valet install`ще раз, як тільки початкове встановлення Valet буде завершено.

<a name="database"></a>

#### База даних

Якщо вам потрібна база даних, спробуйте MySQL, запустивши`brew install mysql@5.7`у вашому командному рядку. Після встановлення MySQL ви можете запустити його за допомогою`brew services start mysql@5.7`команди. Потім ви можете підключитися до бази даних за адресою`127.0.0.1`за допомогою`root`ім'я користувача та порожній рядок для пароля.

<a name="php-versions"></a>

#### Версії PHP

Valet дозволяє переключати версії PHP за допомогою`valet use php@version`команди. Valet встановить вказану версію PHP через Brew, якщо вона ще не встановлена:

    valet use php@7.2

    valet use php

> {note} Valet обслуговує лише одну версію PHP одночасно, навіть якщо у вас встановлено кілька версій PHP.

<a name="resetting-your-installation"></a>

#### Скидання встановлення

Якщо у вас виникли проблеми з належним запуском інсталяції Valet, запустіть`composer global update`команда, за якою слідує`valet install`скине вашу установку і може вирішити різноманітні проблеми. У рідкісних випадках може знадобитися "жорсткий скидання" Valet, виконавши`valet uninstall --force`слідом за ним`valet install`.

<a name="upgrading"></a>

### Оновлення

Ви можете оновити установку Valet за допомогою`composer global update`у вашому терміналі. Після оновлення є гарною практикою запускати`valet install`команда, щоб Valet міг додатково оновити ваші файли конфігурації, якщо це необхідно.

<a name="serving-sites"></a>

## Обслуговування сайтів

Після встановлення Valet ви готові розпочати обслуговування сайтів. Valet надає дві команди, які допоможуть вам обслуговувати ваші сайти Laravel:`park`і`link`.

<a name="the-park-command"></a>

#### `park`Команда

<div class="content-list" markdown="1">
- Create a new directory on your Mac by running something like `mkdir ~/Sites`. Next, `cd ~/Sites` and run `valet park`. This command will register your current working directory as a path that Valet should search for sites.
- Next, create a new Laravel site within this directory: `laravel new blog`.
- Open `http://blog.test` in your browser.
</div>

**Це все.**Тепер будь-який проект Laravel, який ви створюєте у своєму "припаркованому" каталозі, буде автоматично обслуговуватися за допомогою`http://folder-name.test`конвенції.

<a name="the-link-command"></a>

#### `link`Команда

`link`команда може також використовуватися для обслуговування ваших сайтів Laravel. Ця команда корисна, якщо ви хочете обслуговувати один сайт у каталозі, а не весь каталог.

<div class="content-list" markdown="1">
- To use the command, navigate to one of your projects and run `valet link app-name` in your terminal. Valet will create a symbolic link in `~/.config/valet/Sites` which points to your current working directory.
- After running the `link` command, you can access the site in your browser at `http://app-name.test`.
</div>

Щоб переглянути список усіх зв'язаних каталогів, запустіть`valet links`команди. Ви можете використовувати`valet unlink app-name`знищити символічне посилання.

> {tip} Ви можете використовувати`valet link`для обслуговування одного проекту з кількох (під) доменів. Щоб додати субдомен або інший домен до запуску проекту`valet link subdomain.app-name`з папки проекту.

<a name="securing-sites"></a>

#### Захист сайтів за допомогою TLS

За замовчуванням Valet обслуговує сайти через звичайний HTTP. Однак, якщо ви хочете обслуговувати сайт через зашифрований TLS за допомогою HTTP / 2, використовуйте`secure`команди. Наприклад, якщо ваш веб-сайт обслуговується службою Valet на`laravel.test`домену, вам слід виконати таку команду, щоб захистити його:

    valet secure laravel

Щоб "незахистити" сайт і повернутися до обслуговування трафіку через звичайний HTTP, використовуйте`unsecure`команди. Подобається`secure`команда, ця команда приймає ім'я хосту, яке ви хочете незахистити:

    valet unsecure laravel

<a name="sharing-sites"></a>

## Спільний доступ до сайтів

Valet навіть включає команду ділитися своїми локальними веб-сайтами зі світом, забезпечуючи простий спосіб протестувати свій сайт на мобільних пристроях або поділитися ним із членами команди та клієнтами. Після встановлення програми Valet додаткове встановлення програмного забезпечення не потрібно.

<a name="sharing-sites-via-ngrok"></a>

### Обмін сайтами через Ngrok

Щоб надати спільний доступ до сайту, перейдіть до каталогу сайту у вашому терміналі та запустіть`valet share`команди. Загальнодоступна URL-адреса буде вставлена ​​у ваш буфер обміну і готова вставити безпосередньо у ваш браузер або поділитися з командою.

Щоб припинити ділитися своїм веб-сайтом, натисніть`Control + C`скасувати процес.

> {tip} Ви можете передати додаткові параметри команді спільного доступу, наприклад`valet share --region=eu`. Для отримання додаткової інформації зверніться до[документація ngrok](https://ngrok.com/docs).

<a name="sharing-sites-via-expose"></a>

### Спільний доступ до сайтів через Expose

Якщо у вас є[Виставляти](https://beyondco.de/docs/expose)встановлений, ви можете поділитися своїм веб-сайтом, перейшовши до каталогу сайту у вашому терміналі та запустивши`expose`команди. Для отримання додаткових параметрів командного рядка, які він підтримує, зверніться до документації для експозиції. Після спільного використання веб-сайту Expose відобразить загальну URL-адресу, яку ви можете використовувати на своїх інших пристроях або серед членів команди.

Щоб припинити ділитися своїм веб-сайтом, натисніть`Control + C`скасувати процес.

<a name="sharing-sites-on-your-local-network"></a>

### Спільний доступ до сайтів у вашій локальній мережі

Valet обмежує вхідний трафік внутрішнім`127.0.0.1`інтерфейс за замовчуванням. Таким чином ваша машина розробки не піддається ризику безпеки через Інтернет.

Якщо ви хочете дозволити іншим пристроям у вашій локальній мережі отримувати доступ до сайтів Valet на вашому комп'ютері через IP-адресу вашого пристрою (наприклад:`192.168.1.10/app-name.test`), вам потрібно буде вручну відредагувати відповідний файл конфігурації Nginx для цього сайту, щоб зняти обмеження на`listen`директивою, видаливши`127.0.0.1:`префікс директиви для портів 80 та 443.

Якщо ви не бігали`valet secure`у проекті ви можете відкрити доступ до мережі для всіх сайтів, що не є HTTPS, відредагувавши`/usr/local/etc/nginx/valet/valet.conf`файл. Однак, якщо ви обслуговуєте сайт проекту через HTTPS (ви вже запускали`valet secure`для сайту), то вам слід відредагувати`~/.config/valet/Nginx/app-name.test`файл.

Після оновлення конфігурації Nginx запустіть`valet restart`команда застосувати зміни конфігурації.

<a name="site-specific-environment-variables"></a>

## Змінні середовища, специфічні для сайту

Деякі програми, що використовують інші фреймворки, можуть залежати від змінних середовища сервера, але не забезпечують можливості для цих змінних бути налаштованими у вашому проекті. Valet дозволяє налаштувати змінні середовища для певного сайту, додавши a`.valet-env.php`файл у корені вашого проекту. Ці змінні будуть додані до`$_SERVER`глобальний масив:

    <?php

    // Set $_SERVER['key'] to "value" for the foo.test site...
    return [
        'foo' => [
            'key' => 'value',
        ],
    ];

    // Set $_SERVER['key'] to "value" for all sites...
    return [
        '*' => [
            'key' => 'value',
        ],
    ];

<a name="proxying-services"></a>

## Послуги проксі-сервера

Іноді вам може знадобитися додати домен Valet до іншої служби на вашому локальному комп'ютері. Наприклад, іноді вам може знадобитися запустити Valet, одночасно запустивши окремий сайт у Docker; однак Valet і Docker не можуть одночасно прив'язуватися до порту 80.

Для вирішення цього питання ви можете використовувати`proxy`команда згенерувати проксі. Наприклад, ви можете проксі-сервер весь трафік з`http://elasticsearch.test`до`http://127.0.0.1:9200`:

    valet proxy elasticsearch http://127.0.0.1:9200

Ви можете видалити проксі через`unproxy`команда:

    valet unproxy elasticsearch

Ви можете використовувати`proxies`команда для переліку всіх конфігурацій веб-сайтів, які проксі-сервером:

    valet proxies

<a name="custom-valet-drivers"></a>

## Спеціальні драйвери для камердинера

Ви можете написати власний "драйвер" Valet для обслуговування додатків PHP, що працюють на іншому фреймворку або CMS, що не підтримується Valet. Коли ви встановлюєте Valet, a`~/.config/valet/Drivers`створюється каталог, який містить`SampleValetDriver.php`файл. Цей файл містить зразок реалізації драйвера, щоб продемонструвати, як написати власний драйвер. Написання драйвера вимагає лише трьох методів:`serves`,`isStaticFile`, і`frontControllerPath`.

Всі три методи отримують`$sitePath`,`$siteName`, і`$uri`значення як їх аргументи.`$sitePath`- це повністю кваліфікований шлях до веб-сайту, який обслуговується на вашому комп'ютері, наприклад`/Users/Lisa/Sites/my-project`.`$siteName`є частиною домену "хост" / "ім'я сайту" (`my-project`).`$uri`URI вхідного запиту (`/foo/bar`).

Після завершення користувацького драйвера Valet помістіть його в`~/.config/valet/Drivers`каталог за допомогою`FrameworkValetDriver.php`конвенція іменування. Наприклад, якщо ви пишете власний драйвер камердинера для WordPress, ім'я вашого файлу має бути`WordPressValetDriver.php`.

Let's take a look at a sample implementation of each method your custom Valet driver should implement.

<a name="the-serves-method"></a>

#### `serves`Метод

`serves`метод повинен повернутися`true`якщо ваш драйвер повинен обробляти вхідний запит. В іншому випадку метод повинен повернутися`false`. Отже, в рамках цього методу ви повинні спробувати визначити, чи є даний`$sitePath`містить проект типу, якому ви намагаєтесь обслуговувати.

Наприклад, давайте зробимо вигляд, що пишемо a`WordPressValetDriver`. Наші`serves`метод може виглядати приблизно так:

    /**
     * Determine if the driver serves the request.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return bool
     */
    public function serves($sitePath, $siteName, $uri)
    {
        return is_dir($sitePath.'/wp-admin');
    }

<a name="the-isstaticfile-method"></a>

#### `isStaticFile`Метод

`isStaticFile`слід визначити, чи вхідний запит стосується файлу, який є "статичним", наприклад, зображення чи таблиці стилів. Якщо файл статичний, метод повинен повернути повністю визначений шлях до статичного файлу на диску. Якщо вхідний запит не стосується статичного файлу, метод повинен повернутися`false`:

    /**
     * Determine if the incoming request is for a static file.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string|false
     */
    public function isStaticFile($sitePath, $siteName, $uri)
    {
        if (file_exists($staticFilePath = $sitePath.'/public/'.$uri)) {
            return $staticFilePath;
        }

        return false;
    }

> {примітка}`isStaticFile`метод буде викликаний, лише якщо`serves`метод повертає`true`для вхідного запиту та URI запиту немає`/`.

<a name="the-frontcontrollerpath-method"></a>

#### `frontControllerPath`Метод

`frontControllerPath`метод повинен повертати повністю кваліфікований шлях до "фронт-контролера" вашої програми, який зазвичай є вашим файлом "index.php" або еквівалентом:

    /**
     * Get the fully resolved path to the application's front controller.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string
     */
    public function frontControllerPath($sitePath, $siteName, $uri)
    {
        return $sitePath.'/public/index.php';
    }

<a name="local-drivers"></a>

### Місцеві водії

Якщо ви хочете визначити власний драйвер Valet для однієї програми, створіть`LocalValetDriver.php`у кореневому каталозі програми. Ваш користувальницький драйвер може розширити базу`ValetDriver`класу або розширити наявний драйвер для конкретної програми, такий як`LaravelValetDriver`:

    class LocalValetDriver extends LaravelValetDriver
    {
        /**
         * Determine if the driver serves the request.
         *
         * @param  string  $sitePath
         * @param  string  $siteName
         * @param  string  $uri
         * @return bool
         */
        public function serves($sitePath, $siteName, $uri)
        {
            return true;
        }

        /**
         * Get the fully resolved path to the application's front controller.
         *
         * @param  string  $sitePath
         * @param  string  $siteName
         * @param  string  $uri
         * @return string
         */
        public function frontControllerPath($sitePath, $siteName, $uri)
        {
            return $sitePath.'/public_html/index.php';
        }
    }

<a name="other-valet-commands"></a>

## Інші команди Valet

| Команда           | Опис                                                                                                                     |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `valet forget`    | Запустіть цю команду з "припаркованого" каталогу, щоб видалити її зі списку припаркованих каталогів.                     |
| `valet log`       | Перегляньте список журналів, написаних службами Valet.                                                                   |
| `valet paths`     | Перегляньте всі свої "припарковані" стежки.                                                                              |
| `valet restart`   | Перезапустіть демон Valet.                                                                                               |
| `valet start`     | Запустіть демон Valet.                                                                                                   |
| `valet stop`      | Зупиніть демон Valet.                                                                                                    |
| `valet trust`     | Додайте файли sudoers для Brew та Valet, щоб дозволити запускати команди Valet без запиту на введення паролів.           |
| `valet uninstall` | Видалити Valet: Відображає інструкції щодо ручного видалення; або пройти`--force`для агресивного видалення всього Valet. |

<a name="valet-directories-and-files"></a>

## Каталоги та файли Valet

Ви можете знайти такий каталог та інформацію про файли корисними під час вирішення проблем із середовищем Valet:

| Файл / Шлях                                                    | Опис                                                                                                                             |
| -------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `~/.config/valet/`                                             | Містить усі конфігурації Valet. Можливо, ви захочете зберегти резервну копію цієї папки.                                         |
| `~/.config/valet/dnsmasq.d/`                                   | Містить конфігурацію DNSMasq.                                                                                                    |
| `~/.config/valet/Drivers/`                                     | Містить спеціальні драйвери Valet.                                                                                               |
| `~/.config/valet/Extensions/`                                  | Містить спеціальні розширення / команди Valet.                                                                                   |
| `~/.config/valet/Nginx/`                                       | Містить усі конфігурації сайту, створені службою Valet. Ці файли відновлюються під час запуску`install`,`secure`, і`tld`команди. |
| `~/.config/valet/Sites/`                                       | Містить усі символічні посилання для пов'язаних проектів.                                                                        |
| `~/.config/valet/config.json`                                  | Головний файл конфігурації Valet                                                                                                 |
| `~/.config/valet/valet.sock`                                   | Сокет PHP-FPM, використовуваний конфігурацією Nginx від Valet. Це буде існувати, лише якщо PHP працює належним чином.            |
| `~/.config/valet/Log/fpm-php.www.log`                          | Журнал користувача щодо помилок PHP.                                                                                             |
| `~/.config/valet/Log/nginx-error.log`                          | Журнал користувачів щодо помилок Nginx.                                                                                          |
| `/usr/local/var/log/php-fpm.log`                               | Системний журнал помилок PHP-FPM.                                                                                                |
| `/usr/local/var/log/nginx`                                     | Містить журнали доступу та помилок Nginx.                                                                                        |
| `/usr/local/etc/php/X.X/conf.d`                                | Містить`*.ini`файли для різних налаштувань конфігурації PHP.                                                                     |
| `/usr/local/etc/php/X.X/php-fpm.d/valet-fpm.conf`              | Файл конфігурації пулу PHP-FPM.                                                                                                  |
| `~/.composer/vendor/laravel/valet/cli/stubs/secure.valet.conf` | Конфігурація Nginx за замовчуванням, яка використовується для створення сертифікатів сайту.                                      |
