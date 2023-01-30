# Pengujian Konsol

- [Pengantar](#introduction)
- [Ekspektasi Keberhasilan / Kegagalan](#success-failure-expectations)
- [Ekspektasi Masukan / Keluaran](#input-output-expectations)

<a name="introduction"></a>
## Pengantar

Selain menyederhanakan pengujian HTTP, Laravel menyediakan API sederhana untuk menguji dalam aplikasi anda yaitu [perintah konsol khusus](/docs/{{versi}}/artisan).

<a name="success-failure-expectations"></a>
## Ekspektasi Keberhasilan / Kegagalan

Untuk memulai, mari kita jelajahi bagaimana membuat pernyataan mengenai _exit code_ pada perintah Artisan. Untuk mencapai hal ini, kita akan menggunakan metode `artisan` untuk memanggil perintah Artisan dari pengujian kita. Kemudian, kita akan menggunakan metode `assertExitCode` untuk menyatakan bahwa perintah selesai dengan kode _exit_ sebagai berikut:

    /**
     * Test a console command.
     *
     * @return void
     */
    public function test_console_command()
    {
        $this->artisan('inspire')->assertExitCode(0);
    }

Anda dapat menggunakan metode `assertNotExitCode` untuk menyatakan bahwa perintah tidak berakhir dengan kode akhir yang diberikan:

    $this->artisan('inspire')->assertNotExitCode(1);

Tentu saja, semua perintah terminal biasanya keluar dengan kode status `0` bila berhasil dan kode keluar bukan nol jika tidak berhasil. Oleh karena itu, untuk memudahkan, Anda dapat menggunakan perintah `assertSuccessful` dan `assertFailed` untuk memberikan pernyataan bahwa perintah yang diberikan berakhir dengan kode keluar yang berhasil atau tidak:

    $this->artisan('inspire')->assertSuccessful();

    $this->artisan('inspire')->assertFailed();

<a name="input-output-expectations"></a>
## Ekspektasi Masukan / Keluaran

Laravel memungkinkan Anda untuk dengan mudah "menirukan" input pengguna untuk perintah konsol Anda menggunakan metode `expectsQuestion`. Selain itu, Anda dapat menentukan _exit code_ dan teks yang Anda harapkan sebagai output dari perintah konsol menggunakan metode `assertExitCode` dan `expectsOutput`. Sebagai contoh, perhatikan perintah konsol berikut ini:

    Artisan::command('question', function () {
        $name = $this->ask('What is your name?');

        $language = $this->choice('Which language do you prefer?', [
            'PHP',
            'Ruby',
            'Python',
        ]);

        $this->line('Your name is '.$name.' and you prefer '.$language.'.');
    });

Anda dapat menguji perintah ini dengan pengujian berikut yang menggunakan metode `expectsQuestion`, `expectsOutput`, `doesn'tExpectOutput`, `expectsOutputToContain`, `doesn'tExpectOutputToContain`, dan `assertExitCode`:

    /**
     * Test a console command.
     *
     * @return void
     */
    public function test_console_command()
    {
        $this->artisan('question')
             ->expectsQuestion('What is your name?', 'Taylor Otwell')
             ->expectsQuestion('Which language do you prefer?', 'PHP')
             ->expectsOutput('Your name is Taylor Otwell and you prefer PHP.')
             ->doesntExpectOutput('Your name is Taylor Otwell and you prefer Ruby.')
             ->expectsOutputToContain('Taylor Otwell')
             ->doesntExpectOutputToContain('you prefer Ruby')
             ->assertExitCode(0);
    }

<a name="confirmation-expectations"></a>
#### Harapan Konfirmasi

Ketika menulis perintah yang membutuhkan konfirmasi berupa jawaban "ya" atau "tidak", Anda dapat menggunakan metode `expectsConfirmation`:

    $this->artisan('module:import')
        ->expectsConfirmation('Do you really wish to run this command?', 'no')
        ->assertExitCode(1);

<a name="table-expectations"></a>
#### Ekspektasi Tabel

Jika perintah Anda menampilkan tabel informasi menggunakan metode `table` pada Artisan, mungkin akan menjadi tidak praktis untuk menulis hasil ekspektasi pada seluruh tabel. Sebagai gantinya, Anda dapat menggunakan metode `expectsTable`. Metode ini menerima header tabel sebagai argumen pertama dan data tabel sebagai argumen kedua:

    $this->artisan('users:all')
        ->expectsTable([
            'ID',
            'Email',
        ], [
            [1, 'taylor@example.com'],
            [2, 'abigail@example.com'],
        ]);
