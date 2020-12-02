# Консольні тести

-   [Вступ](#introduction)
-   [Очікуваний вхід / вихід](#expecting-input-and-output)

<a name="introduction"></a>

## Вступ

На додаток до спрощення тестування HTTP, Laravel пропонує простий API для тестування консольних програм, які вимагають введення користувачем.

<a name="expecting-input-and-output"></a>

## Очікуваний вхід / вихід

Laravel дозволяє вам легко "висміяти" введення користувачем команд консолі за допомогою`expectsQuestion`метод. Крім того, ви можете вказати код виходу та текст, який, як ви очікуєте, буде виведено командою консолі за допомогою`assertExitCode`і`expectsOutput`методи. Наприклад, розглянемо таку команду консолі:

    Artisan::command('question', function () {
        $name = $this->ask('What is your name?');

        $language = $this->choice('Which language do you program in?', [
            'PHP',
            'Ruby',
            'Python',
        ]);

        $this->line('Your name is '.$name.' and you program in '.$language.'.');
    });

Ви можете протестувати цю команду за допомогою наступного тесту, який використовує`expectsQuestion`,`expectsOutput`, і`assertExitCode`методи:

    /**
     * Test a console command.
     *
     * @return void
     */
    public function testConsoleCommand()
    {
        $this->artisan('question')
             ->expectsQuestion('What is your name?', 'Taylor Otwell')
             ->expectsQuestion('Which language do you program in?', 'PHP')
             ->expectsOutput('Your name is Taylor Otwell and you program in PHP.')
             ->assertExitCode(0);
    }

Під час написання команди, яка очікує підтвердження у формі відповіді "так" чи "ні", ви можете використовувати`expectsConfirmation`метод:

    $this->artisan('module:import')
        ->expectsConfirmation('Do you really wish to run this command?', 'no')
        ->assertExitCode(1);
