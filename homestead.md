# Садиба Laravel

-   [Вступ](#introduction)
-   [Встановлення та налаштування](#installation-and-setup)
    -   [Перші кроки](#first-steps)
    -   [Налаштування Homestead](#configuring-homestead)
    -   [Запуск кошика Vagrant](#launching-the-vagrant-box)
    -   [Встановлення проекту](#per-project-installation)
    -   [Встановлення додаткових функцій](#installing-optional-features)
    -   [Псевдоніми](#aliases)
-   [Щоденне використання](#daily-usage)
    -   [Доступ до Homestead в усьому світі](#accessing-homestead-globally)
    -   [Підключення через SSH](#connecting-via-ssh)
    -   [Підключення до баз даних](#connecting-to-databases)
    -   [Резервне копіювання баз даних](#database-backups)
    -   [Snapshots бази даних](#database-snapshots)
    -   [Додавання додаткових сайтів](#adding-additional-sites)
    -   [Змінні середовища](#environment-variables)
    -   [Wildcard SSL](#wildcard-ssl)
    -   [Налаштування розкладу Cron](#configuring-cron-schedules)
    -   [Налаштування Mailhog](#configuring-mailhog)
    -   [Налаштування Minio](#configuring-minio)
    -   [Порти](#ports)
    -   [Спільне використання Environment](#sharing-your-environment)
    -   [Кілька версій PHP](#multiple-php-versions)
    -   [Веб-сервери](#web-servers)
    -   [Пошта](#mail)
    -   [Dusk Laravel](#laravel-dusk)
-   [Debug та профілювання](#debugging-and-profiling)
    -   [Debug веб-запитів за допомогою Xdebug](#debugging-web-requests)
    -   [Debug програм CLI](#debugging-cli-applications)
    -   [Профілювання додатків за допомогою Blackfire](#profiling-applications-with-blackfire)
-   [Мережеві інтерфейси](#network-interfaces)
-   [Розширення Homestead](#extending-homestead)
-   [Оновлення Homestead](#updating-homestead)
-   [Налаштування провайдера](#provider-specific-settings)
    -   [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>

## Вступ

Laravel прагне зробити весь досвід розробки PHP чудовим, включаючи ваше місцеве середовище розробки.[Vagrantа](https://www.vagrantup.com)забезпечує простий, елегантний спосіб управління та забезпечення віртуальних машин.

Laravel Homestead - це офіційна, заздалегідь упакована коробка Vagrant, яка забезпечує чудове середовище для розробки, не вимагаючи встановлення PHP, веб-сервера та будь-якого іншого серверного програмного забезпечення на локальній машині. Більше не турбуйтеся про те, щоб зіпсувати свою операційну систему! Бродячі ящики повністю одноразові. Якщо щось піде не так, ви можете знищити та заново створити вікно за лічені хвилини!

Homestead працює на будь-якій системі Windows, Mac або Linux і включає Nginx, PHP, MySQL, PostgreSQL, Redis, Memcached, Node та всі інші смачні смаки, необхідні для розробки чудових додатків Laravel.

> {note} Якщо ви використовуєте Windows, можливо, вам доведеться увімкнути апаратну віртуалізацію (VT-x). Зазвичай його можна увімкнути через BIOS. Якщо ви використовуєте Hyper-V в системі UEFI, можливо, вам доведеться додатково вимкнути Hyper-V, щоб отримати доступ до VT-x.

<a name="included-software"></a>

### Включене програмне забезпечення

<style>
    #software-list > ul {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 5em; -moz-column-gap: 5em; -webkit-column-gap: 5em;
        line-height: 1.9;
    }
</style>

<div id="software-list" markdown="1">
- Ubuntu 18.04 (`master` branch)
- Ubuntu 20.04 (`20.04` branch)
- Git
- PHP 8.0
- PHP 7.4
- PHP 7.3
- PHP 7.2
- PHP 7.1
- PHP 7.0
- PHP 5.6
- Nginx
- MySQL
- lmm for MySQL or MariaDB database snapshots
- Sqlite3
- PostgreSQL (9.6, 10, 11, 12)
- Composer
- Node (With Yarn, Bower, Grunt, and Gulp)
- Redis
- Memcached
- Beanstalkd
- Mailhog
- avahi
- ngrok
- Xdebug
- XHProf / Tideways / XHGui
- wp-cli
</div>

<a name="optional-software"></a>

### Необов’язкове програмне забезпечення

<style>
    #software-list > ul {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 5em; -moz-column-gap: 5em; -webkit-column-gap: 5em;
        line-height: 1.9;
    }
</style>

<div id="software-list" markdown="1">
- Apache
- Blackfire
- Cassandra
- Chronograf
- CouchDB
- Crystal & Lucky Framework
- Docker
- Elasticsearch
- Gearman
- Go
- Grafana
- InfluxDB
- MariaDB
- MinIO
- MongoDB
- MySQL 8
- Neo4j
- Oh My Zsh
- Open Resty
- PM2
- Python
- RabbitMQ
- Solr
- Webdriver & Laravel Dusk Utilities
</div>

<a name="installation-and-setup"></a>

## Встановлення та налаштування

<a name="first-steps"></a>

### Перші кроки

Перед запуском середовища Homestead ви повинні встановити[VirtualBox 6.x](https://www.virtualbox.org/wiki/Downloads),[VMWare](https://www.vmware.com),[Паралелі](https://www.parallels.com/products/desktop/)або[Хопер-В](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v)так само, як[Vagrantа](https://www.vagrantup.com/downloads.html). Всі ці програмні пакети забезпечують прості у використанні візуальні програми для встановлення всіх популярних операційних систем.

Для використання постачальника послуг VMware вам потрібно буде придбати як VMware Fusion / Workstation, так і[Плагін VMware Vagrant](https://www.vagrantup.com/vmware). Незважаючи на те, що це не безкоштовно, VMware може забезпечити швидшу продуктивність спільних папок зразу.

Щоб скористатися постачальником Parallels, вам потрібно буде встановити[Плагін Parallels Vagrant](https://github.com/Parallels/vagrant-parallels). Це безкоштовно.

Тому що[Обмеження Vagrant](https://www.vagrantup.com/docs/hyperv/limitations.html), постачальник Hyper-V ігнорує всі налаштування мережі.

<a name="installing-the-homestead-vagrant-box"></a>

#### Встановлення The Homestead Vagrant Box

Після встановлення VirtualBox / VMware та Vagrant слід додати файл`laravel/homestead`вікна до вашої установки Vagrant, використовуючи наступну команду у вашому терміналі. Завантаження коробки займе кілька хвилин, залежно від швидкості підключення до Інтернету:

    vagrant box add laravel/homestead

Якщо ця команда не вдається, переконайтеся, що ваша установка Vagrant є сучасною.

> {note} Homestead періодично видає для тестування поля "альфа" / "бета", які можуть заважати роботі`vagrant box add`команди. Якщо у вас виникли проблеми із запуском`vagrant box add`, ви можете запустити`vagrant up`команда і правильне вікно буде завантажено, коли Vagrant спробує запустити віртуальну машину.

<a name="installing-homestead"></a>

#### Встановлення Homestead

Ви можете встановити Homestead, клонуючи сховище на хост-машині. Подумайте про клонування сховища у файл`Homestead`в домашній директорії, оскільки вікно Homestead буде хостом для всіх ваших проектів Laravel:

    git clone https://github.com/laravel/homestead.git ~/Homestead

Ви повинні ознайомитися з тегова версія Homestead з`master`гілка не завжди може бути стабільною. Ви можете знайти останню стабільну версію на[Сторінка випуску GitHub](https://github.com/laravel/homestead/releases). Крім того, ви можете замовити`release`гілка, яка завжди містить останній стабільний випуск:

    cd ~/Homestead

    git checkout release

Після клонування сховища Homestead, запустіть`bash init.sh`команда з каталогу Homestead для створення`Homestead.yaml`файл конфігурації.`Homestead.yaml`файл буде розміщено в каталозі Homestead:

    // Mac / Linux...
    bash init.sh

    // Windows...
    init.bat

<a name="configuring-homestead"></a>

### Налаштування Homestead

<a name="setting-your-provider"></a>

#### Налаштування вашого провайдера

`provider`ключ у вашому`Homestead.yaml`файл вказує, якого постачальника послуг Vagrant слід використовувати:`virtualbox`,`vmware_fusion`,`vmware_workstation`,`parallels`або`hyperv`. Ви можете встановити це для провайдера, який вам більше подобається:

    provider: virtualbox

<a name="configuring-shared-folders"></a>

#### Налаштування спільних папок

`folders`власність`Homestead.yaml`у файлі перелічені всі папки, якими ви хочете поділитися із середовищем Homestead Оскільки файли в цих папках змінюються, вони будуть синхронізовані між локальною машиною та середовищем Homestead. Ви можете налаштувати стільки спільних папок, скільки потрібно:

    folders:
        - map: ~/code/project1
          to: /home/vagrant/project1

> {note} Користувачам Windows не слід використовувати`~/`path синтаксис і замість цього повинні використовувати повний шлях до свого проекту, наприклад`C:\Users\user\Code\project1`.

Завжди слід зіставляти окремі проекти з їх власними папками, а не цілими`~/code`папку. Коли ви зіставляєте папку, віртуальна машина повинна відстежувати всі дискові операції введення-виведення_кожен_файл у папці. Це призводить до проблем із продуктивністю, якщо у вас є велика кількість файлів у папці.

    folders:
        - map: ~/code/project1
          to: /home/vagrant/project1

        - map: ~/code/project2
          to: /home/vagrant/project2

> {note} Ніколи не слід монтувати`.`(поточний каталог) при використанні Homestead. Це призводить до того, що Vagrant не відображає поточну папку`/vagrant`і порушить додаткові функції та призведе до несподіваних результатів під час резервування.

Для того, щоб[Той самий](https://www.vagrantup.com/docs/synced-folders/nfs.html), вам потрібно лише додати простий прапор до конфігурації синхронізованої папки:

    folders:
        - map: ~/code/project1
          to: /home/vagrant/project1
          type: "nfs"

> {note} Під час використання NFS у Windows, вам слід розглянути можливість встановлення[Vagrantа-winnfsd](https://github.com/winnfsd/vagrant-winnfsd)підключати. Цей плагін підтримуватиме правильні дозволи користувачів / груп для файлів та каталогів у полі Homestead.

Ви також можете передати будь-які варіанти, підтримувані Vagrant's[Синхронізовані папки](https://www.vagrantup.com/docs/synced-folders/basic_usage.html)перерахувавши їх під`options`ключ:

    folders:
        - map: ~/code/project1
          to: /home/vagrant/project1
          type: "rsync"
          options:
              rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
              rsync__exclude: ["node_modules"]

<a name="configuring-nginx-sites"></a>

#### Налаштування сайтів Nginx

Не знайомий з Nginx? Нема проблем.`sites`властивість дозволяє легко зіставити "домен" із папкою у вашому оточенні Homestead. Зразок конфігурації сайту включений до`Homestead.yaml`файл. Знову ж таки, ви можете додати скільки завгодно сайтів до свого середовища Homestead, скільки потрібно. Homestead може слугувати зручним віртуальним середовищем для кожного проекту Laravel, над яким ви працюєте:

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public

Якщо змінити`sites`майна після надання коробки "Садиба", вам слід повторно запустити`vagrant reload --provision`оновити конфігурацію Nginx на віртуальній машині.

> {примітка} Сценарії Homestead побудовані так, щоб вони були якомога ідемпотентнішими. Однак якщо у вас виникають проблеми під час резервування, вам слід знищити та відновити машину через`vagrant destroy && vagrant up`.

<a name="enable-disable-services"></a>

#### Увімкнути / Вимкнути служби

Homestead за замовчуванням запускає кілька служб; однак ви можете налаштувати, які служби вмикаються чи вимикаються під час надання. Наприклад, ви можете увімкнути PostgreSQL і вимкнути MySQL:

    services:
        - enabled:
            - "postgresql@12-main"
        - disabled:
            - "mysql"

Зазначені послуги будуть запущені або припинені на основі їх замовлення в`enabled`і`disabled`директиви.

<a name="hostname-resolution"></a>

#### Дозвіл імені хосту

Homestead публікує імена хостів`mDNS`для автоматичного дозволу хоста. Якщо встановити`hostname: homestead`у вашому`Homestead.yaml`файл, хост буде доступний за адресою`homestead.local`. До складу настільних дистрибутивів MacOS, iOS та Linux входять`mDNS`підтримка за замовчуванням. Windows вимагає встановлення[Послуги друку Bonjour для Windows](https://support.apple.com/kb/DL999?viewlocale=en_US&locale=en_US).

Використання автоматичних імен хостів найкраще підходить для інсталяцій Homestead "за проектом". Якщо ви розміщуєте кілька сайтів на одному екземплярі Homestead, ви можете додати "домени" для своїх веб-сайтів до`hosts`файл на вашому комп'ютері.`hosts`файл перенаправить запити на ваші сайти Homestead у вашу машину Homestead. На Mac та Linux цей файл знаходиться за адресою`/etc/hosts`. У Windows він знаходиться за адресою`C:\Windows\System32\drivers\etc\hosts`. Рядки, які ви додаєте до цього файлу, матимуть такий вигляд:

    192.168.10.10  homestead.test

Переконайтесь, що вказана IP-адреса вказана у вашому`Homestead.yaml`файл. Після додавання домену до вашого`hosts`файл і запустив поле Vagrant, ви зможете отримати доступ до сайту через веб-браузер:

    http://homestead.test

<a name="launching-the-vagrant-box"></a>

### Запуск кошика Vagrant

Після редагування`Homestead.yaml`на ваш смак, запустіть`vagrant up`з вашого каталогу Homestead. Vagrant завантажить віртуальну машину та автоматично налаштує ваші спільні папки та сайти Nginx.

Щоб знищити машину, ви можете використовувати`vagrant destroy --force`команди.

<a name="per-project-installation"></a>

### За встановлення проекту

Замість того, щоб встановлювати Homestead глобально та ділитися одним і тим же полем Homestead у всіх своїх проектах, ви можете замість цього налаштувати екземпляр Homestead для кожного проекту, яким ви керуєте. Встановлення Homestead для проекту може бути корисним, якщо ви хочете відправити`Vagrantfile`з вашим проектом, дозволяючи іншим, хто працює над проектом`vagrant up`.

Щоб встановити Homestead безпосередньо у свій проект, вимагайте його, використовуючи Composer:

    composer require laravel/homestead --dev

Після встановлення Homestead використовуйте`make`команда для створення`Vagrantfile`і`Homestead.yaml`у вашому проектному корені.`make`команда автоматично налаштує`sites`і`folders`директиви в`Homestead.yaml`файл.

Mac / Linux:

    php vendor/bin/homestead make

Windows:

    vendor\\bin\\homestead make

Далі запустіть`vagrant up`у своєму терміналі та отримайте доступ до свого проекту за адресою`http://homestead.test`у вашому браузері. Пам'ятайте, вам все одно потрібно буде додати`/etc/hosts`запис файлу для`homestead.test`або вибраний вами домен, якщо ви не використовуєте автоматичний[дозвіл імені хосту](#hostname-resolution).

<a name="installing-optional-features"></a>

### Встановлення додаткових функцій

Додаткове програмне забезпечення встановлюється за допомогою параметра "особливості" у вашому файлі конфігурації Homestead. Більшість функцій можна ввімкнути або вимкнути за допомогою логічного значення, тоді як деякі функції дозволяють кілька варіантів конфігурації:

    features:
        - blackfire:
            server_id: "server_id"
            server_token: "server_value"
            client_id: "client_id"
            client_token: "client_value"
        - cassandra: true
        - chronograf: true
        - couchdb: true
        - crystal: true
        - docker: true
        - elasticsearch:
            version: 7.9.0
        - gearman: true
        - golang: true
        - grafana: true
        - influxdb: true
        - mariadb: true
        - minio: true
        - mongodb: true
        - mysql8: true
        - neo4j: true
        - ohmyzsh: true
        - openresty: true
        - pm2: true
        - python: true
        - rabbitmq: true
        - solr: true
        - webdriver: true

<a name="mariadb"></a>

#### MariaDB

Увімкнення MariaDB призведе до видалення MySQL та встановлення MariaDB. MariaDB служить заміною MySQL, тому вам все одно слід використовувати`mysql`драйвер бази даних у конфігурації бази даних вашої програми.

<a name="mongodb"></a>

#### MongoDB

За замовчуванням установка MongoDB встановить для імені користувача бази даних значення`homestead`та відповідний пароль до`secret`.

<a name="elasticsearch"></a>

#### Еластичний пошук

Ви можете вказати підтримувану версію Elasticsearch, яка повинна мати точний номер версії (major.minor.patch). Інсталяція за замовчуванням створить кластер із назвою 'садиба'. Ви ніколи не повинні віддавати Elasticsearch більше половини пам'яті операційної системи, тому переконайтеся, що ваш апарат Homestead має принаймні вдвічі більше розподілу Elasticsearch.

> {tip} Перевірте[Документація Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current)щоб дізнатися, як налаштувати конфігурацію.

<a name="neo4j"></a>

#### Neo4j

За замовчуванням установка Neo4j встановить ім’я користувача бази даних`homestead`і відповідний пароль до`secret`. Щоб отримати доступ до браузера Neo4j, відвідайте`http://homestead.test:7474`через веб-браузер. Порти`7687`(Болт),`7474`(HTTP) та`7473`(HTTPS) готові обслуговувати запити від клієнта Neo4j.

<a name="aliases"></a>

### Псевдоніми

Ви можете додати псевдоніми Bash до своєї машини Homestead, змінивши`aliases`у вашому каталозі Homestead:

    alias c='clear'
    alias ..='cd ..'

Після оновлення`aliases`файл, вам слід повторно забезпечити машину Homestead за допомогою`vagrant reload --provision`команди. Це забезпечить доступність нових псевдонімів на машині.

<a name="daily-usage"></a>

## Щоденне використання

<a name="accessing-homestead-globally"></a>

### Доступ до Homestead в усьому світі

Іноді вам може знадобитися`vagrant up`вашу машину Homestead з будь-якої точки вашої файлової системи. Ви можете зробити це в системах Mac / Linux, додавши функцію Bash до свого профілю Bash. У Windows ви можете зробити це, додавши "пакетний" файл до вашого`PATH`. Ці сценарії дозволять вам запустити будь-яку команду Vagrant з будь-якої точки вашої системи та автоматично направлять цю команду на вашу інсталяцію Homestead:

<a name="mac-linux"></a>

#### Mac / Linux

    function homestead() {
        ( cd ~/Homestead && vagrant $* )
    }

Не забудьте налаштувати`~/Homestead`шлях у функції до місця фактичного встановлення Homestead. Після встановлення функції ви можете запускати такі команди, як`homestead up`або`homestead ssh`з будь-якої точки вашої системи.

<a name="windows"></a>

#### Windows

Створити`homestead.bat`пакетний файл в будь-якому місці на вашому комп'ютері з таким вмістом:

    @echo off

    set cwd=%cd%
    set homesteadVagrant=C:\Homestead

    cd /d %homesteadVagrant% && vagrant %*
    cd /d %cwd%

    set cwd=
    set homesteadVagrant=

Обов’язково налаштуйте приклад`C:\Homestead`шлях у сценарії до фактичного місця встановлення вашого Homestead. Після створення файлу додайте розташування файлу до вашого`PATH`. Потім ви можете запускати такі команди, як`homestead up`або`homestead ssh`з будь-якої точки вашої системи.

<a name="connecting-via-ssh"></a>

### Підключення через SSH

Ви можете перенести SSH на свою віртуальну машину, видавши файл`vagrant ssh`терміналу з вашого каталогу Homestead.

Але оскільки вам, напевно, доведеться часто SSH додавати до вашої машини Homestead, подумайте про додавання описаної вище «функції» до вашої хост-машини для швидкого SSH у поле Homestead.

<a name="connecting-to-databases"></a>

### Підключення до баз даних

A`homestead`база даних налаштована як для MySQL, так і для PostgreSQL нестандартно. Щоб підключитися до бази даних MySQL або PostgreSQL з клієнта бази даних на хост-машині, слід підключитися до`127.0.0.1`і порт`33060`(MySQL) або`54320`(PostgreSQL). Ім'я користувача та пароль для обох баз даних`homestead`/`secret`.

> {note} Ці нестандартні порти слід використовувати лише під час підключення до баз даних із хост-машини. Ви будете використовувати за замовчуванням порти 3306 та 5432 у вашому файлі конфігурації бази даних Laravel, оскільки Laravel запущений_в межах_віртуальна машина.

<a name="database-backups"></a>

### Резервне копіювання баз даних

Homestead може автоматично створювати резервні копії бази даних, коли ваш ящик Vagrant буде знищений. Щоб використовувати цю функцію, ви повинні використовувати Vagrant 2.1.0 або новішої версії. Або, якщо ви використовуєте стару версію Vagrant, вам слід встановити`vagrant-triggers`підключати. Щоб увімкнути автоматичне резервне копіювання бази даних, додайте наступний рядок до вашого`Homestead.yaml`файл:

    backup: true

Після налаштування Homestead експортуватиме ваші бази даних до`mysql_backup`і`postgres_backup`каталоги, коли`vagrant destroy`команда виконується. Ці каталоги можна знайти в папці, де ви клонували Homestead, або в кореневій частині вашого проекту, якщо ви використовуєте[за встановлення проекту](#per-project-installation)метод.

<a name="database-snapshots"></a>

### Snapshots бази даних

Homestead підтримує заморожування стану баз даних MySQL та MariaDB та розгалуження між ними за допомогою[Логічний менеджер MySQL](https://github.com/Lullabot/lmm). Наприклад, уявіть, як ви працюєте на сайті з багатогігабайтною базою даних. Ви можете імпортувати базу даних і зробити знімок. Виконавши певну роботу та створивши тестовий вміст локально, ви можете швидко відновити початковий стан.

Під капотом LMM використовує тонку функцію знімка LVM з підтримкою копіювання на запис. На практиці це означає, що зміна одного рядка в таблиці призведе лише до того, що внесені вами зміни будуть записані на диск, заощаджуючи значний час та місце на диску під час відновлення.

Оскільки`lmm`взаємодіє з LVM, його потрібно запускати як`root`. Щоб переглянути всі доступні команди, запустіть`sudo lmm`всередині вашої коробки Vagrant. Загальний робочий процес виглядає так:

1.  Імпортуйте базу даних за замовчуванням`master`лмм гілка.
2.  Збережіть знімок незміненої бази даних за допомогою`sudo lmm branch prod-YYYY-MM-DD`.
3.  Змінити базу даних.
4.  Біжи`sudo lmm merge prod-YYYY-MM-DD`скасувати всі зміни.
5.  Біжи`sudo lmm delete <branch>`видалити непотрібні гілки.

<a name="adding-additional-sites"></a>

### Додавання додаткових сайтів

Після того, як ваше середовище Homestead буде створено та запущено, ви можете додати додаткові сайти Nginx для своїх програм Laravel. Ви можете запускати скільки завгодно установок Laravel в одному середовищі Homestead. Щоб додати додатковий сайт, додайте його до вашого`Homestead.yaml`файл:

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public
        - map: another.test
          to: /home/vagrant/project2/public

Якщо Vagrant не керує автоматично вашим файлом "хостів", можливо, вам доведеться також додати новий сайт до цього файлу:

    192.168.10.10  homestead.test
    192.168.10.10  another.test

Після додавання сайту запустіть`vagrant reload --provision`з вашого каталогу Homestead.

<a name="site-types"></a>

#### Типи сайтів

Homestead підтримує декілька типів сайтів, які дозволяють легко запускати проекти, які не засновані на Laravel. Наприклад, ми можемо легко додати додаток Symfony до Homestead за допомогою`symfony2`тип сайту:

    sites:
        - map: symfony2.test
          to: /home/vagrant/my-symfony-project/web
          type: "symfony2"

Доступні типи сайтів:`apache`,`apigility`,`expressive`,`laravel`(за замовчуванням),`proxy`,`silverstripe`,`statamic`,`symfony2`,`symfony4`, і`zf`.

<a name="site-parameters"></a>

#### Параметри сайту

Ви можете додати додатковий Nginx`fastcgi_param`значення для вашого сайту через`params`директива сайту. Наприклад, ми додамо a`FOO`параметр зі значенням`BAR`:

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public
          params:
              - key: FOO
                value: BAR

<a name="environment-variables"></a>

### Змінні середовища

Ви можете встановити глобальні змінні середовища, додавши їх до вашого`Homestead.yaml`файл:

    variables:
        - key: APP_ENV
          value: local
        - key: FOO
          value: bar

Після оновлення`Homestead.yaml`, не забудьте повторно забезпечити машину, запустивши`vagrant reload --provision`. Це оновить конфігурацію PHP-FPM для всіх встановлених версій PHP, а також оновить середовище для`vagrant`користувач.

<a name="wildcard-ssl"></a>

### Wildcard SSL

Homestead конфігурує самопідписаний сертифікат SSL для кожного сайту, визначеного в`sites:`розділ вашого`Homestead.yaml`файл. Якщо ви хочете створити підстановочний SSL-сертифікат для сайту, ви можете додати`wildcard`варіант конфігурації цього сайту. За замовчуванням сайт використовуватиме сертифікат підстановки_натомість_сертифіката конкретного домену:

    - map: foo.domain.test
      to: /home/vagrant/domain
      wildcard: "yes"

Якщо`use_wildcard`для параметра встановлено значення`no`, сертифікат підстановки буде створений, але не використовуватиметься:

    - map: foo.domain.test
      to: /home/vagrant/domain
      wildcard: "yes"
      use_wildcard: "no"

<a name="configuring-cron-schedules"></a>

### Налаштування розкладів Cron

Laravel забезпечує зручний спосіб[розклад роботи Cron](/docs/{{version}}/scheduling)шляхом складання розкладу на один`schedule:run`Реміснича команда повинна виконуватися щохвилини.`schedule:run`команда вивчить графік роботи, визначений у вашому`App\Console\Kernel`клас, щоб визначити, які завдання слід запускати.

Якщо ви хочете`schedule:run`Команда, яку потрібно запустити для сайту Homestead, ви можете встановити`schedule`варіант до`true`при визначенні сайту:

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public
          schedule: true

Робота Cron для сайту буде визначена в`/etc/cron.d`папка віртуальної машини.

<a name="configuring-mailhog"></a>

### Налаштування Mailhog

Mailhog дозволяє легко перехопити вихідну електронну пошту та перевірити її, фактично не надсилаючи повідомлення одержувачам. Для початку оновіть свій`.env`файл, щоб використовувати такі налаштування пошти:

    MAIL_MAILER=smtp
    MAIL_HOST=localhost
    MAIL_PORT=1025
    MAIL_USERNAME=null
    MAIL_PASSWORD=null
    MAIL_ENCRYPTION=null

Після налаштування Mailhog ви можете отримати доступ до інформаційної панелі Mailhog за адресою`http://localhost:8025`.

<a name="configuring-minio"></a>

### Налаштування Minio

Minio - це сервер зберігання даних з відкритим кодом із сумісним API Amazon S3. Щоб встановити Minio, оновіть свій`Homestead.yaml`файл із наступним параметром конфігурації в[особливості](#installing-optional-features)розділ:

    minio: true

За замовчуванням Minio доступний через порт 9600. Ви можете отримати доступ до панелі керування Minio, відвідавши`http://localhost:9600/`. Ключ доступу за замовчуванням -`homestead`, а секретним ключем за замовчуванням є`secretkey`. Під час доступу до Minio ви завжди повинні використовувати регіон`us-east-1`.

Для використання Minio вам потрібно буде налаштувати конфігурацію диска S3 у вашому`config/filesystems.php`файл конфігурації. Вам потрібно буде додати`use_path_style_endpoint`опцію до конфігурації диска, а також змінити`url`ключ до`endpoint`:

    's3' => [
        'driver' => 's3',
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION'),
        'bucket' => env('AWS_BUCKET'),
        'endpoint' => env('AWS_URL'),
        'use_path_style_endpoint' => true,
    ]

Нарешті, переконайтеся, що ваш`.env`файл має такі опції:

    AWS_ACCESS_KEY_ID=homestead
    AWS_SECRET_ACCESS_KEY=secretkey
    AWS_DEFAULT_REGION=us-east-1
    AWS_URL=http://localhost:9600

Щоб забезпечити сегменти, додайте a`buckets`директива до вашого конфігураційного файлу Homestead:

    buckets:
        - name: your-bucket
          policy: public
        - name: your-private-bucket
          policy: none

Підтримується`policy`значення включають:`none`,`download`,`upload`, і`public`.

<a name="ports"></a>

### Порти

За замовчуванням до вашого середовища Homestead пересилаються такі порти:

<div class="content-list" markdown="1">
- **SSH:** 2222 &rarr; Forwards To 22
- **ngrok UI:** 4040 &rarr; Forwards To 4040
- **HTTP:** 8000 &rarr; Forwards To 80
- **HTTPS:** 44300 &rarr; Forwards To 443
- **MySQL:** 33060 &rarr; Forwards To 3306
- **PostgreSQL:** 54320 &rarr; Forwards To 5432
- **MongoDB:** 27017 &rarr; Forwards To 27017
- **Mailhog:** 8025 &rarr; Forwards To 8025
- **Minio:** 9600 &rarr; Forwards To 9600
</div>

<a name="forwarding-additional-ports"></a>

#### Переадресація додаткових портів

За бажанням ви можете переслати додаткові порти у поле Vagrant, а також вказати їх протокол:

    ports:
        - send: 50000
          to: 5000
        - send: 7777
          to: 777
          protocol: udp

<a name="sharing-your-environment"></a>

### Спільне використання Environment

Іноді, можливо, ви захочете поділитися тим, над чим зараз працюєте, з колегами чи клієнтами. Vagrant має вбудований спосіб підтримати це через`vagrant share`; однак це не спрацює, якщо у вас налаштовано кілька сайтів`Homestead.yaml`файл.

Для вирішення цієї проблеми Homestead включає свою власну`share`команди. Для початку вставте SSH у свою машину Homestead через`vagrant ssh`і біжи`share homestead.test`. Це поділиться`homestead.test`сайту з вашого`Homestead.yaml`файл конфігурації. Ви можете замінити будь-який інший налаштований веб-сайт`homestead.test`:

    share homestead.test

Після запуску команди ви побачите екран Ngrok, який містить журнал активності та загальнодоступні URL-адреси спільного сайту. Якщо ви хочете вказати спеціальний регіон, субдомен або інший варіант виконання Ngrok, ви можете додати їх до свого`share`команда:

    share homestead.test -region=eu -subdomain=laravel

> {note} Пам'ятайте, Vagrant за своєю суттю небезпечний, і ви піддаєте свою віртуальну машину Інтернету під час запуску`share`команди.

<a name="multiple-php-versions"></a>

### Кілька версій PHP

Homestead 6 представив підтримку декількох версій PHP на одній і тій же віртуальній машині. Ви можете вказати, яку версію PHP використовувати для певного сайту у вашому`Homestead.yaml`файл. Доступні версії PHP: "5.6", "7.0", "7.1", "7.2", "7.3" і "7.4" (за замовчуванням):

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public
          php: "7.1"

Крім того, ви можете використовувати будь-яку з підтримуваних версій PHP через CLI:

    php5.6 artisan list
    php7.0 artisan list
    php7.1 artisan list
    php7.2 artisan list
    php7.3 artisan list
    php7.4 artisan list

Ви також можете оновити версію CLI за замовчуванням, видавши такі команди з віртуальної машини Homestead:

    php56
    php70
    php71
    php72
    php73
    php74

<a name="web-servers"></a>

### Веб-сервери

Homestead використовує веб-сервер Nginx за замовчуванням. Однак він може встановити Apache, якщо`apache`вказано як тип сайту. Хоча обидва веб-сервери можуть бути встановлені одночасно, вони не можуть бути обома_біг_в той самий час.`flip`Команда shell доступна для полегшення процесу перемикання між веб-серверами.`flip`Команда автоматично визначає, який веб-сервер запущений, вимикає його, а потім запускає інший сервер. Щоб скористатися цією командою, вставте SSH у машину Homestead і запустіть команду у своєму терміналі:

    flip

<a name="mail"></a>

### Пошта

До складу Homestead входить агент передачі пошти Postfix, який прослуховує порт`1025`за замовчуванням. Отже, ви можете доручити своїй програмі використовувати`smtp`поштовий драйвер на`localhost`порт`1025`. Тоді вся надіслана пошта буде оброблятися Postfix і перехоплюватися Mailhog. Щоб переглянути надіслані електронні листи, відкрийте[http: // localhost: 8025](http://localhost:8025)у веб-браузері.

<a name="laravel-dusk"></a>

### Dusk Laravel

Для того, щоб бігти[Dusk Laravel](/docs/{{version}}/dusk)тести в Homestead, вам слід увімкнути[`webdriver`особливість](#installing-optional-features)у вашій конфігурації Homestead:

      features:
          - webdriver: true

Не забудьте забезпечити свою віртуальну машину Homestead згодом, щоб забезпечити`webdriver`функція повністю встановлена.

<a name="debugging-and-profiling"></a>

## Debug та профілювання

<a name="debugging-web-requests"></a>

### Debug веб-запитів за допомогою Xdebug

Homestead включає підтримку використання Debug кроків[Xdebug](https://xdebug.org). Наприклад, ви можете завантажити веб-сторінку з браузера, і PHP підключиться до вашої IDE, щоб дозволити перевірку та модифікацію запущеного коду.

За замовчуванням Xdebug вже запущений і готовий приймати підключення. Якщо вам потрібно увімкнути Xdebug на CLI, запустіть`sudo phpenmod xdebug`у вашому вікні Vagrant. Далі, дотримуйтесь інструкцій IDE, щоб увімкнути Debug. Нарешті, налаштуйте ваш браузер для запуску Xdebug із розширенням або[букмарклет](https://www.jetbrains.com/phpstorm/marklets/).

> {note} Xdebug призводить до того, що PHP працює значно повільніше. Щоб вимкнути Xdebug, запустіть`sudo phpdismod xdebug`і перезапустіть службу FPM.

<a name="debugging-cli-applications"></a>

### Debug програм CLI

Для Debug програми PHP CLI використовуйте`xphp`псевдонім оболонки всередині вашої коробки Vagrant:

    xphp path/to/script

<a name="autostarting-xdebug"></a>

#### Автозапуск Xdebug

Під час Debug функціональних тестів, які роблять запити на веб-сервер, легше автоматично запускати Debug, а не модифікувати тести для проходження через спеціальний заголовок або файл cookie для запуску Debug. Щоб змусити Xdebug запускатися автоматично, внесіть зміни`/etc/php/7.x/fpm/conf.d/20-xdebug.ini`всередині вашої коробки Vagrant та додайте таку конфігурацію:

    ; If Homestead.yaml contains a different subnet for the IP address, this address may be different...
    xdebug.remote_host = 192.168.10.1
    xdebug.remote_autostart = 1

<a name="profiling-applications-with-blackfire"></a>

### Профілювання додатків за допомогою Blackfire

[Blackfire](https://blackfire.io/docs/introduction)є послугою SaaS для профілювання веб-запитів та додатків CLI та написання Assertiors про ефективність. Він пропонує інтерактивний користувальницький інтерфейс, який відображає дані профілю в графіках викликів та часових шкалах. Він створений для використання в розробці, постановці та виробництві, без накладних витрат для кінцевих споживачів. Він забезпечує перевірку продуктивності, якості та безпеки коду та`php.ini`конфігураційні установки.

[Програвач Blackfire](https://blackfire.io/docs/player/index)- це програма для веб-сканування, веб-тестування та веб-скрепінгу з відкритим кодом, яка може працювати спільно з Blackfire для створення сценаріїв профілювання сценаріїв.

Щоб увімкнути Blackfire, використовуйте параметр "особливості" у файлі конфігурації Homestead:

    features:
        - blackfire:
            server_id: "server_id"
            server_token: "server_value"
            client_id: "client_id"
            client_token: "client_value"

Повноваження сервера Blackfire та клієнтські дані[потрібен обліковий запис користувача](https://blackfire.io/signup). Blackfire пропонує різні варіанти профілювання програми, включаючи інструмент CLI та розширення браузера. Будь ласка[перегляньте документацію Blackfire, щоб отримати докладнішу інформацію](https://blackfire.io/docs/cookbooks/index).

<a name="profiling-php-performance-using-xhgui"></a>

### Профілювання продуктивності PHP за допомогою XHGui

[XHGui](https://www.github.com/perftools/xhgui)- це користувальницький інтерфейс для вивчення продуктивності ваших програм PHP. Щоб увімкнути XHGui, додайте`xhgui: 'true'`до конфігурації вашого сайту:

    sites:
        -
            map: your-site.test
            to: /home/vagrant/your-site/public
            type: "apache"
            xhgui: 'true'

Якщо сайт вже існує, переконайтеся, що він запущений`vagrant provision`після оновлення конфігурації.

Щоб профілювати веб-запит, додайте`xhgui=on`як параметр запиту до запиту. XHGui автоматично приєднає до відповіді файл cookie, щоб наступні запити не потребували значення рядка запиту. Ви можете переглянути результати свого профілю заявки, перейшовши за адресою`http://your-site.test/xhgui`.

Щоб профілювати запит CLI за допомогою XHGui, додайте команді префікс до`XHGUI=on`:

    XHGUI=on path/to/script

Результати профілю CLI можна переглядати так само, як результати веб-профілю.

Зверніть увагу, що акт профілювання уповільнює виконання сценарію, і абсолютний час може бути вдвічі більшим, ніж реальні запити. Тому завжди порівнюйте процентні покращення, а не абсолютні цифри. Також пам’ятайте, що час виконання включає будь-який час, проведений на паузі в налагоджувачі.

Оскільки профілі продуктивності займають значне місце на диску, вони автоматично видаляються через кілька днів.

<a name="network-interfaces"></a>

## Мережеві інтерфейси

`networks`власність`Homestead.yaml`налаштовує мережеві інтерфейси для вашого середовища Homestead. Ви можете налаштувати стільки інтерфейсів, скільки потрібно:

    networks:
        - type: "private_network"
          ip: "192.168.10.20"

Щоб увімкнути a[мостовий](https://www.vagrantup.com/docs/networking/public_network.html)інтерфейс, налаштуйте a`bridge`налаштування та змініть тип мережі на`public_network`:

    networks:
        - type: "public_network"
          ip: "192.168.10.20"
          bridge: "en1: Wi-Fi (AirPort)"

Для того, щоб[DHCP](https://www.vagrantup.com/docs/networking/public_network.html), просто видаліть`ip`опція з вашої конфігурації:

    networks:
        - type: "public_network"
          bridge: "en1: Wi-Fi (AirPort)"

<a name="extending-homestead"></a>

## Розширення Homestead

Ви можете продовжити Homestead за допомогою`after.sh`скрипт у кореневій частині каталогу Homestead. У цей файл ви можете додати будь-які команди оболонки, необхідні для правильної настройки та налаштування вашої віртуальної машини.

Під час налаштування Homestead Ubuntu може запитати вас, чи хочете ви зберегти оригінальну конфігурацію пакета або перезаписати його новим файлом конфігурації. Щоб цього уникнути, слід використовувати наступну команду під час встановлення пакунків, щоб уникнути перезапису будь-якої конфігурації, раніше написаної Homestead:

    sudo apt-get -y \
        -o Dpkg::Options::="--force-confdef" \
        -o Dpkg::Options::="--force-confold" \
        install your-package

<a name="user-customizations"></a>

### Налаштування користувача

Використовуючи Homestead в командному середовищі, ви можете налаштувати Homestead, щоб краще відповідати вашому стилю особистого розвитку. Ви можете створити`user-customizations.sh`файл у кореневій частині вашого каталогу Homestead (Той самий каталог, що містить ваш`Homestead.yaml`). У цьому файлі ви можете зробити будь-яке налаштування, яке хочете; однак,`user-customizations.sh`не повинен контролюватися версіями.

<a name="updating-homestead"></a>

## Оновлення Homestead

Перш ніж приступати до оновлення Homestead, переконайтеся, що ви видалили свою поточну віртуальну машину, виконавши таку команду в каталозі Homestead:

    vagrant destroy

Далі вам потрібно оновити вихідний код Homestead. Якщо ви клонували сховище, ви можете виконати наступні команди в тому місці, де ви спочатку клонували сховище:

    git fetch

    git pull origin release

Ці команди витягують останній код Homestead зі сховища GitHub, отримують останні теги, а потім перевіряють останній тегований випуск. Ви можете знайти останню стабільну версію версії на[Сторінка випусків GitHub](https://github.com/laravel/homestead/releases).

Якщо ви встановили Homestead через ваш проект`composer.json`файл, ви повинні переконатися, що ваш`composer.json`файл містить`"laravel/homestead": "^11"`та оновіть свої залежності:

    composer update

Потім вам слід оновити поле Vagrant за допомогою`vagrant box update`команда:

    vagrant box update

Далі вам слід запустити`bash init.sh`з каталогу Homestead для оновлення деяких додаткових файлів конфігурації. Вас запитають, чи хочете ви перезаписати існуючі`Homestead.yaml`,`after.sh`, і`aliases`файли:

    // Mac / Linux...
    bash init.sh

    // Windows...
    init.bat

Нарешті, вам потрібно буде регенерувати свій ящик Homestead, щоб використовувати останню установку Vagrant:

    vagrant up

<a name="provider-specific-settings"></a>

## Налаштування провайдера

<a name="provider-specific-virtualbox"></a>

### VirtualBox

<a name="natdnshostresolver"></a>

#### `natdnshostresolver`

За замовчуванням Homestead налаштовує`natdnshostresolver`встановивши на`on`. Це дозволяє Homestead використовувати налаштування DNS вашої операційної системи. Якщо ви хочете замінити цю поведінку, додайте наступні рядки до вашого`Homestead.yaml`файл:

    provider: virtualbox
    natdnshostresolver: 'off'

<a name="symbolic-links-on-windows"></a>

#### Символічні посилання в Windows

Якщо символічні посилання не працюють належним чином на вашому комп'ютері Windows, можливо, вам доведеться додати наступний блок до вашого`Vagrantfile`:

    config.vm.provider "virtualbox" do |v|
        v.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
    end
