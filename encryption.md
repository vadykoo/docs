# Шифрування

-   [Вступ](#introduction)
-   [Конфігурація](#configuration)
-   [Використання шифрувача](#using-the-encrypter)

<a name="introduction"></a>

## Вступ

Шифрувач Laravel використовує OpenSSL для забезпечення шифрування AES-256 та AES-128. Вам настійно рекомендується використовувати вбудовані засоби шифрування Laravel і не намагатися застосовувати власні "доморощені" алгоритми шифрування. Всі зашифровані значення Laravel підписані за допомогою коду автентифікації повідомлень (MAC), так що їх базове значення не може бути змінено після шифрування.

<a name="configuration"></a>

## Конфігурація

Перш ніж використовувати шифрувач Laravel, ви повинні встановити a`key`варіант у вашому`config/app.php`файл конфігурації. Ви повинні використовувати`php artisan key:generate`команда для генерації цього ключа, оскільки ця команда Artisan використовуватиме захищений генератор випадкових байтів PHP для створення вашого ключа. Якщо це значення встановлено неправильно, усі значення, зашифровані Laravel, будуть небезпечними.

<a name="using-the-encrypter"></a>

## Використання шифрувача

<a name="encrypting-a-value"></a>

#### Шифрування значення

Ви можете зашифрувати значення за допомогою`encryptString`метод`Crypt`фасад. Всі зашифровані значення шифруються за допомогою OpenSSL та`AES-256-CBC`шифр. Крім того, всі зашифровані значення підписуються кодом автентифікації повідомлень (MAC) для виявлення будь-яких змін у зашифрованому рядку:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\User;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Crypt;

    class UserController extends Controller
    {
        /**
         * Store a secret message for the user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function storeSecret(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $user->fill([
                'secret' => Crypt::encryptString($request->secret),
            ])->save();
        }
    }

<a name="decrypting-a-value"></a>

#### Розшифровка значення

Ви можете розшифрувати значення за допомогою`decryptString`метод`Crypt`фасад. Якщо значення неможливо належним чином розшифрувати, наприклад, коли MAC недійсний, з'явиться`Illuminate\Contracts\Encryption\DecryptException`буде кинуто:

    use Illuminate\Contracts\Encryption\DecryptException;
    use Illuminate\Support\Facades\Crypt;

    try {
        $decrypted = Crypt::decryptString($encryptedValue);
    } catch (DecryptException $e) {
        //
    }
