# Pengujian Konsol

- [Pengantar](#introduction)
- [Ekspektasi Keberhasilan / Kegagalan](#success-failure-expectations)
- [Ekspektasi Masukan / Keluaran](#input-output-expectations)

<a name="introduction"></a>
## Pengantar

Selain menyederhanakan pengujian HTTP, Laravel menyediakan API sederhana untuk menguji perintah konsol kustom milik aplikasi Anda [perintah konsol khusus](/docs/{{versi}}/artisan).

<a name="success-failure-expectations"></a>
## Ekspektasi Keberhasilan / Kegagalan

Untuk memulai, mari kita jelajahi bagaimana membuat pernyataan mengenai kode _exit_ pada perintah Artisan. Untuk mencapai hal ini, kita akan menggunakan metode `artisan` untuk memanggil perintah Artisan pada pengujian kita. Kemudian, kita akan menggunakan metode `assertExitCode` untuk menyatakan bahwa perintah selesai dengan kode _exit_ sebagai berikut:

    /**
     * Test a console command.
     *
     * @return void
     */
    public function test_console_command()
    {
        $this->artisan('inspire')->assertExitCode(0);
    }

Anda dapat menggunakan metode `assertNotExitCode` untuk menyatakan bahwa perintah tidak berakhir dengan kode _exit_ yang ditentukan:

    $this->artisan('inspire')->assertNotExitCode(1);

Tentu saja, semua perintah terminal biasanya keluar dengan kode status `0`  ketika perintah berhasil dijalankan atau ketika perintah yang gagal dijalankan memiliki kode _exit_ bukan `0`. Oleh karena itu, untuk memudahkan, Anda dapat menggunakan perintah `assertSuccessful` dan `assertFailed` untuk memberikan pernyataan bahwa perintah yang diberikan berakhir dengan kode _exit_ yang berhasil atau tidak:

    $this->artisan('inspire')->assertSuccessful();

    $this->artisan('inspire')->assertFailed();

<a name="input-output-expectations"></a>
## Ekspektasi Input / _Output_

Laravel memungkinkan Anda "menirukan" input dari pengguna untuk perintah konsol Anda dengan mudah menggunakan metode `expectsQuestion`. Selain itu, Anda dapat menentukan kode _exit_ dan teks yang Anda harapkan sebagai output dari perintah konsol menggunakan metode `assertExitCode` dan `expectsOutput`. Sebagai contoh, perhatikan perintah konsol berikut ini:

    Artisan::command('question', function () {
        $name = $this->ask('Siapa nama Anda?');

        $language = $this->choice('Bahasa apa yang Anda lebih suka?', [
            'PHP',
            'Ruby',
            'Python',
        ]);

        $this->line('Nama Anda adalah '.$name.' dan Anda lebih menyukai '.$language.'.');
    });

Anda dapat menguji perintah-perintah tersebut dengan menggunakan metode `expectsQuestion`, `expectsOutput`, `doesntExpectOutput`, `expectsOutputToContain`, `doesntExpectOutputToContain`, dan `assertExitCode`:

    /**
     * Test a console command.
     *
     * @return void
     */
    public function test_console_command()
    {
        $this->artisan('question')
             ->expectsQuestion('Siapa nama Anda?', 'Taylor Otwell')
             ->expectsQuestion('Bahasa Apa yang Anda lebih suka?', 'PHP')
             ->expectsOutput('Nama Anda adalah Taylor Otwell dan Anda lebih menyukai PHP.')
             ->doesntExpectOutput('Nama Anda adalah Taylor Otwell dan Anda lebih menyukai Ruby.')
             ->expectsOutputToContain('Taylor Otwell')
             ->doesntExpectOutputToContain('Anda lebih menyukai Ruby')
             ->assertExitCode(0);
    }

<a name="confirmation-expectations"></a>
#### Ekspektasi Konfirmasi

Ketika menulis perintah yang membutuhkan konfirmasi berupa jawaban "_yes_" atau "_no_", Anda dapat menggunakan metode `expectsConfirmation`:

    $this->artisan('module:import')
        ->expectsConfirmation('Apa Anda benar-benar ingin menjalankan perintah tersebut?', 'no')
        ->assertExitCode(1);

<a name="table-expectations"></a>
#### Ekspektasi Tabel

Jika perintah Anda menampilkan tabel informasi menggunakan metode `table` pada Artisan, mungkin akan menjadi tidak praktis untuk menulis seluruh tabel pada ekspektasi. Sebagai gantinya, Anda dapat menggunakan metode `expectsTable`. Metode ini menerima kop / kepala tabel sebagai argumen pertama dan data / badan tabel sebagai argumen kedua:

    $this->artisan('users:all')
        ->expectsTable([
            'ID',
            'Email',
        ], [
            [1, 'taylor@example.com'],
            [2, 'abigail@example.com'],
        ]);
