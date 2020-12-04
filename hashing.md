# Хешування

-   [Вступ](#introduction)
-   [Конфігурація](#configuration)
-   [Базове використання](#basic-usage)

<a name="introduction"></a>

## Вступ

Ларавель`Hash`[фасад](/docs/{{version}}/facades)забезпечує безпечне хешування Bcrypt та Argon2 для зберігання паролів користувачів. Якщо ви використовуєте[Laravel Jetstream](https://jetstream.laravel.com)лесу для автентифікації, Bcrypt буде використовуватися для реєстрації та автентифікації за замовчуванням.

> {tip} Bcrypt - чудовий вибір для хешування паролів, оскільки його "робочий коефіцієнт" можна регулювати, а це означає, що час, необхідний для генерації хешу, може бути збільшений із збільшенням потужності обладнання.

<a name="configuration"></a>

## Конфігурація

Драйвер хешування за замовчуванням для вашої програми налаштовано в`config/hashing.php`файл конфігурації. На даний момент підтримуються три драйвери:[Bcrypt](https://en.wikipedia.org/wiki/Bcrypt)і[ми повернулися](https://en.wikipedia.org/wiki/Argon2)(Варіанти Argon2i та Argon2id).

> {note} Драйвер Argon2i вимагає PHP 7.2.0 або вище, а для Argon2id драйвер PHP 7.3.0 або вище.

<a name="basic-usage"></a>

## Базове використання

Ви можете хеш-пароль, зателефонувавши до`make`метод на`Hash`фасад:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;

    class UpdatePasswordController extends Controller
    {
        /**
         * Update the password for the user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // Validate the new password length...

            $request->user()->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();
        }
    }

<a name="adjusting-the-bcrypt-work-factor"></a>

#### Налаштування фактора роботи Bcrypt

Якщо ви використовуєте алгоритм Bcrypt, файл`make`Метод дозволяє управляти коефіцієнтом роботи алгоритму за допомогою`rounds`варіант; однак за замовчуванням прийнятно для більшості програм:

    $hashed = Hash::make('password', [
        'rounds' => 12,
    ]);

<a name="adjusting-the-argon2-work-factor"></a>

#### Налаштування фактора роботи Argon2

Якщо ви використовуєте алгоритм Argon2, файл`make`Метод дозволяє управляти коефіцієнтом роботи алгоритму за допомогою`memory`,`time`, і`threads` options; however, the defaults are acceptable for most applications:

    $hashed = Hash::make('password', [
        'memory' => 1024,
        'time' => 2,
        'threads' => 2,
    ]);

> {tip} Щоб отримати додаткову інформацію про ці варіанти, перегляньте[офіційна документація PHP](https://secure.php.net/manual/en/function.password-hash.php).

<a name="verifying-a-password-against-a-hash"></a>

#### Перевірка пароля проти хешу

`check`Метод дозволяє перевірити, що заданий текстовий рядок відповідає заданому хешу:

    if (Hash::check('plain-text', $hashedPassword)) {
        // The passwords match...
    }

<a name="checking-if-a-password-needs-to-be-rehashed"></a>

#### Перевірка, чи потрібно переробити пароль

`needsRehash`функція дозволяє визначити, чи змінився робочий коефіцієнт, який використовує хеш, після хешування пароля:

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }
