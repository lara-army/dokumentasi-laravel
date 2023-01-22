# Konsol Artisan

- [Pengantar](#introduction)
  - [Tinker (REPL)](#tinker)
- [Menulis Perintah](#writing-commands)
  - [Membuat Perintah](#generating-commands)
  - [Struktur Perintah](#command-structure)
  - [Perintah _Closure_](#closure-commands)
  - [Perintah Terisolasi](#isolatable-commands)
- [Menentukan _Input_ yang Diharapkan](#defining-input-expectations)
  - [Argumen](#arguments)
  - [Opsi](#options)
  - [_Input_ dalam Bentuk _Array_](#input-arrays)
  - [Deskripsi _Input_](#input-descriptions)
- [I/O Perintah](#command-io)
  - [Mengambil _Input_](#retrieving-input)
  - [_Prompting_ untuk _Input_](#prompting-for-input)
  - [Menulis _Output_](#writing-output)
- [Mendaftarkan Perintah](#registering-commands)
- [Mengeksekusi Perintah Secara Terprogram](#programmatically-executing-commands)
  - [Memanggil Perintah di Dalam Perintah](#calling-commands-from-other-commands)
- [Penanganan Sinyal](#signal-handling)
- [Kostumisasi _Stub_](#stub-customization)
- [_Event_](#events)

<a name="introduction"></a>
## Pengantar

Artisan merupakan command line interface—antarmuka dalam bentuk baris perintah— (CLI) yang terdapat pada Laravel. Artisan terdapat pada _root_ aplikasi Anda sebagai skrip `artisan` yang menyediakan beberapa perintah yang dapat membantu Anda untuk membangun aplikasi. Untuk melihat daftar perintah yang terdapat pada Artisan, Anda dapat menggunakan perintah `list`:

```shell
php artisan list
```

Setiap perintah memiliki sebuah layar "help" (bantuan) yang menampilkan dan menjelaskan argumen dan opsi yang tersedia pada perintah tersebut. Untuk melihat layar bantuan, gunakan perintah `help` diikuti dengan nama perintah yang ingin dilihat:

```shell
php artisan help migrate
```

<a name="laravel-sail"></a>
#### Laravel Sail

Jika Anda menggunakan [Laravel Sail](/docs/{{version}}/sail) sebagai lingkungan _development_ pada lokal, jangan lupa untuk menggunakan perintah `sail` untuk melakukan perintah Artisan. Sail akan mengeksekusi perintah Artisan Anda pada kontainer Docker:

```shell
./vendor/bin/sail artisan list
```

<a name="tinker"></a>
### Tinker (REPL)

Laravel Tinker merupakan sebuah REPL yang _powerful_ untuk _framework_ Laravel, ditenagai oleh _package_ [PsySH](https://github.com/bobthecow/psysh).

<a name="installation"></a>
#### Instalasi

Semua aplikasi Laravel secara _default_ telah memiliki Tinker. Namun, Anda dapat memasang Tinker menggunakan Composer jika Tinker telah terhapus dari aplikasi Anda.

```shell
composer require laravel/tinker
```

> **Catatan**  
> Mencari UI grafis untuk berinteraksi dengan aplikasi Laravel Anda? Lihatlah [Tinkerwell](https://tinkerwell.app)!

<a name="usage"></a>
#### Penggunaan

Tinker memungkinkan Anda untuk berinteraksi dengan aplikasi Anda secara keseluruhan menggunakan baris perintah, termasuk model Eloquent, _jobs_, _events_, dan lainnya. Untuk memasuki lingkungan Tinker, jalankan perintah Artisan `tinker`:

```shell
php artisan tinker
```

Anda dapat menerbitkan _file_ konfigurasi Tinker menggunakan perintah `vendor:publish`:

```shell
php artisan vendor:publish --provider="Laravel\Tinker\TinkerServiceProvider"
```

> **Peringatan**  
> _Function_ `dispatch` pada _helper_ dan _method_ `dispatch` pada _class_ `Dispatchable` bergantung pada _garbage collection_ untuk menempatkan _job_ pada _queue_. Karena itu, ketika menggunakan Tinker, Anda harus menggunakan `Bus::dispatch` atau `Queue::push` untuk mengirimkan (_dispatch_) _job_.

<a name="command-allow-list"></a>
#### Daftar Perintah yang Diizinkan

Tinker dilengkapi dengan sebuah daftar "allow" untuk menentukan perintah Artisan mana yang diizinkan untuk dijalankan pada _shell_. secara _default_, Anda dapat menjalankan perintah `clear-compiled`, `down`, `env`, `inspire`, `migrate`, `optimize`, dan `up`. Jika ingin menjalankan perintah-perintah lain, Anda bisa menambahkannya pada _array_ `commands` pada _file_ konfigurasi `tinker.php`:

    'commands' => [
        // App\Console\Commands\ExampleCommand::class,
    ],

<a name="classes-that-should-not-be-aliased"></a>
#### _Class_-_Class_ yang Seharusnya Tidak Diberi Alias

Biasanya, Tinker secara otomatis membuat alias terhadap kelas-kelas saat Anda berinteraksi dengan kelas-kelas tersebut. Namun, Anda mungkin mengharapkan beberapa kelas untuk tidak diberi alias. Anda dapat mewujudkan hal ini dengan mendaftarkan kelas-kelas tersebut ke _array_ `dont_alias` pada _file_ konfigurasi `tinker.php` Anda:

    'dont_alias' => [
        App\Models\User::class,
    ],

<a name="writing-commands"></a>
## Menulis Perintah

Untuk menambah perintah yang disediakan oleh Artisan, Anda dapat membuat perintah kustom Anda ssendiri. Perintah-perintah tersebut biasanya disimpan pada direktori `app/Console/Commands`; bagaimanapun, Anda bebas untuk memilih lokasi penyimpanan perintah Anda selama dapat dimuat oleh Composer.

<a name="generating-commands"></a>
### Membuat Perintah

Untuk membuat perintah (_command_) baru, Anda dapat menggunakan perintah Artisan `make:command`. Perintah ini akan membuat _class command_ yang baru pada direktori `app/Console/Commands`. Jangan khawatir jika direktori ini belum ada pada aplikasi Anda - direktori secara otomatis akan muncul ketika menjalankan perintah Artisan `make:command` untuk pertama kali:

```shell
php artisan make:command SendEmails
```

<a name="command-structure"></a>
### Struktur Perintah

Setelah membuat perintah baru, Anda harus mendefinisikan nilai-nilai yang sesuai untuk properti `signature` dan `description` pada _class_ tersebut. properti-properti ini akan digunakan ketika perintah baru Anda ditampilkan pada daftar perintah (`list` _screen_). Properti `signature` juga memungkinkan Anda untuk mmendefinisikan [_input_ yang diharapkan perintah](#defining-input-expectations). _Tethod_ `handle` akan dipanggil ketika perintah Anda dieksekusi. Anda dapat menempatkan logika Anda pada _method_ ini.

Mari kita lihat sebuah contoh perintah. Perlu diingat bahwa kita dapat memasukkan _dependencies_ apa saja yang dibutuhkan melalui _method_ `handle` milik _command_. [_service container_](/docs/{{version}}/container) milik Laravel akan melakukan _inject_ semua _dependencies_ secara otomatis yang telah di-_type hint_ di dalam _signature_ milik _method_:

    <?php

    namespace App\Console\Commands;

    use App\Models\User;
    use App\Support\DripEmailer;
    use Illuminate\Console\Command;

    class SendEmails extends Command
    {
        /**
         * Nama dan signature dari perintah console.
         *
         * @var string
         */
        protected $signature = 'mail:send {user}';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Send a marketing email to a user';

        /**
         * Eksekusi perintah console.
         *
         * @param  \App\Support\DripEmailer  $drip
         * @return mixed
         */
        public function handle(DripEmailer $drip)
        {
            $drip->send(User::find($this->argument('user')));
        }
    }

> **Catatan**  
> Untuk penggunaan kembali kode (_reuse code_) yang lebih baik, praktik yang baik adalah menjaga perintah _console_ tetap ringan dan membiarkan _service_ aplikasi yang menyelesaikan tugas berat itu. Pada contoh diatas, perlu diingat kita memasukkan sebuah _class service_ untuk melakukan "angkat beban" dalam pengiriman e-mail.

<a name="closure-commands"></a>
### Perintah _Closure_

Perintah berbasis _closure_ menyediakan sebuah alternatif untuk mendefinisikan perintah _console_ sebagai _class_. Hal ini mirip seperi _closure route_ sebagai alternatif untuk _controller_, bayangkan sebuah perintah _closure_ sebagai alternatif dari _class command_. Dengan _method_ `commands` pada _file_ `app/Console/Kernel.php` Anda, Laravel akan memuat _file_ `routes/console.php`:

    /**
     * Mendaftarkan perintah berbasis closure untuk aplikasi.
     *
     * @return void
     */
    protected function commands()
    {
        require base_path('routes/console.php');
    }

Even though _file_ ini tidak mendefinisikan _route_ HTTP, _file_ ini mendefinisikan _entry points (routes)_ berbasis _console_ ke dalam aplikasi
Anda. Dengan _file_ ini, Anda dapat mendefinisikan semua perintah _console_ berbasis _closure_ menggunakan _method_ `Artisan::command`. _method_ `command` menerima dua argumen: [_signature_ perintah](#defining-input-expectations) dan sebuah _closure_ yang menerima argumen dan opsi dari pperintah:

    Artisan::command('mail:send {user}', function ($user) {
        $this->info("Mengirim email ke: {$user}!");
    });

_Closure_ terikat pada _instance_ perintah yang mendasarinya, sehingga Anda memiliki akses penuh ke semua _method_ helper yang biasanya dapat diakses pada _class_ perintah biasa.

<a name="type-hinting-dependencies"></a>
#### _Type-Hinting_ Dependensi

Selain menerima argumen dan opsi perintah Anda, perintah _closure_ juga dapat melakukan type-hint dependensi tambahan yang ingin Anda selesaikan [_service container_](/docs/{{version}}/container):

    use App\Models\User;
    use App\Support\DripEmailer;

    Artisan::command('mail:send {user}', function (DripEmailer $drip, $user) {
        $drip->send(User::find($user));
    });

<a name="closure-command-descriptions"></a>
#### Deskripsi Perintah _Closure_

Saat menentukan perintah berbasis _closure_, Anda dapat menggunakan _method_ `purpose` untuk menambahkan deskripsi untuk perintah. Deskripsi ini akan ditampilkan saat Anda menjalankan perintah `php artisan list` atau `php artisan help`:

    Artisan::command('mail:send {user}', function ($user) {
        // ...
    })->purpose('Mengirim email marketing ke pengguna');

<a name="isolatable-commands"></a>
### Perintah Terisolasi

> **Peringatan**
> Untuk menggunakan fitur ini, aplikasi Anda harus menggunakan _driver cache_ `memcached`, `redis`, `dynamodb`, `database`, `file`, atau `array` sebagai _driver cache default_ aplikasi Anda. Selain itu, semua server harus berkomunikasi dengan _server cache_ pusat yang sama.

Terkadang Anda mungkin ingin memastikan bahwa hanya ada satu _instance_ perintah yang dapat dijalankan pada suatu waktu. Untuk mencapai hal tersebut, Anda dapat mengimplementasikan _interface_ `Illuminate\Contracts\console\Isolatable` pada _class_ perintah itu:

    <?php

    namespace App\Console\Commands;

    use Illuminate\Console\Command;
    use Illuminate\Contracts\Console\Isolatable;

    class SendEmails extends Command implements Isolatable
    {
        // ...
    }

Ketika perintah ditandai sebagai `Isolatable`, Laravel secara otomatis akan menambahkan opsi `--isolated` ke perintah tersebut. Ketika perintah dipanggil dengan opsi tersebut, Laravel akan memastikan bahwa tidak ada _instance_ lain dari perintah tersebut yang sedang berjalan. Laravel mewujudkan ini dengan mencoba untuk memperoleh _lock atomic_ mmenggunakan _driver cache default_ aplikasi Anda. Jika ada _instance_ lain dari perintah yang sedang berjalan, perintah tidak akan dieksekusi; namun, perintah akan tetap berhenti dengan kode status keluar yang sukses:

```shell
php artisan mail:send 1 --isolated
```

Jika Anda ingin menentukan kode status keluar yang harus dikembalikan oleh perintah jika tidak dapat dieksekusi, Anda dapat memberikan kode status yang diinginkan melalui opsi `isolated`:

```shell
php artisan mail:send 1 --isolated=12
```

<a name="lock-expiration-time"></a>
#### Waktu Kunci Kadaluarsa

Secara _default_, penguncian isolasi akan kadaluarsa setelah perintah selesai. Atau, jika perintah diinterupsi atau tidak dapat terselesaikan, penguncian akan kadaluarsa setelah satu jam. However, Anda dapat menyesuaikan waktu kadaluarsa untuk penguncian dengan mendefinisikan sebuah _method_ `isolationLockExpiresAt` pada perintah Anda:

```php
/**
 * menentukan kapan kunci isolasi kadaluarsa pada perintah.
 *
 * @return \DateTimeInterface|\DateInterval
 */
public function isolationLockExpiresAt()
{
    return now()->addMinutes(5);
}
```

<a name="defining-input-expectations"></a>
## Menentukan _Input_ yang Diharapkan

Saat menulis perintah _console_, umumnya Anda mengumpulkan _input_ dari pengguna melalui argumen atau opsi. Laravel memudahkan dalam menentukan _input_ yang diharapkan dari pengguna menggunakan properti `signature` pada perintah Anda. Properti `signature` memungkinkan Anda untuk menentukan nama, argumen, dan opsi untuk perintah dalam satu sintaks yang ekspresif, seperti _route_.

<a name="arguments"></a>
### Argumen

Semua argumen dan opsi yang disediakan pengguna dibungkus dalam tanda kurung kurawal. Dalam contoh berikut, suatu perintah didefinisikan dengan satu argumen yang diperlukan: `user`:

    /**
     * Nama dan signature dari perintah console.
     *
     * @var string
     */
    protected $signature = 'mail:send {user}';

Anda juga dapat membuat argumen tersebut menjadi opsional atau menentukan nilai _default_-nya:

    // Argumen opsional...
    'mail:send {user?}'

    // Argumen opsional dengan nilai default...
    'mail:send {user=foo}'

<a name="options"></a>

### Opsi

Opsi seperti argumen, merupakan bentuk lain dari _input_ pengguna. Opsi diawali dengan dua tanda hubung (`--`) saat dituliskan dalam baris perintah. Ada dua jenis opsi: yang menerima nilai dan yang tidak. Opsi yang tidak menerima nilai berfungsi sebagai "_switch_" _boolean_. Mari kita lihat contoh opsi jenis ini:

    /**
     * Nama dan signature dari perintah console.
     *
     * @var string
     */
    protected $signature = 'mail:send {user} {--queue}';

Pada contoh ini, _switch_ `--queue` dapat ditambahkan saat memanggil perintah Artisan. Jika _switch_ `--queue` diberikan, nilai opsi akan menjadi `true`. Jika tidak, nilai akan menjadi `false`:

```shell
php artisan mail:send 1 --queue
```

<a name="options-with-values"></a>
#### Opsi dengan Nilai

Selanjutnya, marilah kita lihat opsi yang dapat diisi nilai. Jika pengguna harus memasukkan nilai untuk opsi, Anda harus menambahkan tanda `=` pada nama opsi:

    /**
     * Nama dan signature dari perintah console.
     *
     * @var string
     */
    protected $signature = 'mail:send {user} {--queue=}';

Dalam contoh ini, pengguna dapat mengirimkan nilai untuk opsi seperti ini. Jika opsi tidak ditentukan saat menjalankan perintah, nilainya akan menjadi `null`:

```shell
php artisan mail:send 1 --queue=default
```

Anda dapat memberikan nilai _default_ ke opsi dengan menentukan nilai _default_ setelah nama opsi. Jika tidak ada nilai opsi yang diberikan oleh pengguna, nilai _default_ akan digunakan:

    'mail:send {user} {--queue=default}'

<a name="option-shortcuts"></a>
#### _Shortcut_ Opsi

Untuk memberikan _shortcut_ saat menentukan opsi, Anda dapat menentukannya sebelum nama opsi dan menggunakan karakter `|` sebagai pembatas untuk memisahkan antara _shortcut_ dan nama lengkap untuk opsi:

    'mail:send {user} {--Q|queue}'

Saat menjalankan perintah di terminal Anda, _shortcut_ opsi harus diawali dengan tanda hubung tunggal:

```shell
php artisan mail:send 1 -Q
```

<a name="input-arrays"></a>
### _Input_ dalam Bentuk _Array_

Jika Anda ingin menentukan argumen atau opsi yang mengharapkan beberapa nilai input, Anda dapat menggunakan karakter `*`. Mari kita lihat contohnya:

    'mail:send {user*}'

Saat memanggil _method_ ini, argumen `user` dapat diberikan ke baris perintah. Misalnya, perintah berikut akan mengatur nilai `user` menjadi sebuah _array_ dengan nilai `1` dan `2`:

```shell
php artisan mail:send 1 2
```

Karakter `*` ini dapat digabungkan dengan definisi argumen opsional untuk mengijinkan _instance_ argumen menjadi nol atau lebih:

    'mail:send {user?*}'

<a name="option-arrays"></a>
#### _Array_ Opsi

Saat menentukan opsi yang mengharapkan beberapa nilai _input_, setiap nilai opsi yang diberikan ke perintah harus diawali dengan nama opsi:

    'mail:send {--id=*}'

Perintah seperti itu dapat dijalankan dengan mengirimkan beberapa argumen `--id`:

```shell
php artisan mail:send --id=1 --id=2
```

<a name="input-descriptions"></a>
### Deskripsi _Input_

Anda dapat memberikan deskripsi ke argumen dan opsi dengan memisahkan nama argumen dari deskripsi menggunakan tanda titik dua. Jika Anda membutuhkan baris tambahan untuk mendefinisikan perintah, Anda dapat menulisnya dalam beberapa baris:

    /**
     * Nama dan signature dari perintah console.
     *
     * @var string
     */
    protected $signature = 'mail:send
                            {user : The ID of the user}
                            {--queue : Whether the job should be queued}';

<a name="command-io"></a>
## I/O Perintah

<a name="retrieving-input"></a>
### Mengambil _Input_

Saat perintah Anda sedang dieksekusi, Anda mungkin akan perlu mengakses nilai untuk argumen dan opsi yang diterima oleh perintah tersebut. Untuk melakukannya, Anda dapat menggunakan _method_ `argument` dan `option`. Jika argumen atau opsi tidak ada, maka _method_ tersebut akan mengembalikan nilai `null`:

    /**
     * Eksekusi perintah console.
     *
     * @return int
     */
    public function handle()
    {
        $userId = $this->argument('user');

        //
    }

Jika Anda perlu mengambil semua argumen sebagai `array`, gunakan _method_ `arguments`:

    $arguments = $this->arguments();

Opsi dapat dengan mudah diperoleh seperti argumen menggunakan _method_ `option`. Untuk mengambil semua opsi sebagai _array_, panggil _method_ `options`:

    // Peroleh sebuah opsi...
    $queueName = $this->option('queue');

    // Peroleh semua opsi...
    $options = $this->options();

<a name="prompting-for-input"></a>
### _Prompting_ untuk _Input_

Selain menampilkan output, Anda juga dapat meminta pengguna untuk memberikan input selama eksekusi perintah Anda. _Method_ `ask` akan meminta jawaban pengguna dengan pertanyaan yang diberikan, kemudian mengembalikan _input_ tersebut ke perintah Anda:

    /**
     * Eksekusi perintah console.
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('Siapa nama Anda?');
    }

_Method_ `secret` mirip dengan _method_ `ask`, namun _input_ dari pengguna tidak akan terlihat ketika diketikan pada _console_. _Method_ ini sangat berguna ketika memasukkan informasi yang sensitif seperti kata sandi:

    $password = $this->secret('Apa kata sandi Anda?');

<a name="asking-for-confirmation"></a>
#### Meminta Konfirmasi

Jika Anda membutuhkan konfirmasi sederhana "ya atau tidak" dari pengguna, Anda dapat menggunakan _method_ `confirm`. Secara _default_, _method_ ini akan mengembalikan nilai `false`. Namun, jika pengguna menjawab `y` atau `yes` maka _method_ akan mengembalikan nilai `true`:

    if ($this->confirm('Apa Anda ingin melanjutkan?')) {
        //
    }

ika diperlukan, Anda dapat menentukan bahwa _prompt_ konfirmasi harus mengembalikan `true` secara `default` dengan mengirimkan `true` sebagai argumen kedua pada _method_ `confirm`:

    if ($this->confirm('Apakah Anda ingin melanjutkan?', true)) {
        //
    }

<a name="auto-completion"></a>
#### _Auto-Completion_

_Method_ `anticipate` dapat digunakan untuk memberikan _auto-completion_ pada pilihan yang ada. Pengguna masih dapat memasukkan jawabannya sendiri tanpa memperdulikan pilihan dari _auto-completion_:

    $name = $this->anticipate('Siapa nama Anda?', ['Taylor', 'Dayle']);

Atau, Anda dapat mengirimkan _closure_ sebagai argumen kedua ke _method_ `anticipate`. _Closure_ akan dipanggil setiap kali pengguna mengetik karakter _input_. _Closure_ harus menerima parameter _string_ yang berisi _input_ pengguna sampai saat ini, dan mengembalikan _array_ opsi untuk _auto-completion_:

    $name = $this->anticipate('Di mana alamat Anda?', function ($input) {
        // Mengembalikan opsi untuk auto-completion...
    });

<a name="multiple-choice-questions"></a>
#### Pertanyaan dengan Pilihan Ganda

Jika Anda perlu memberikan pilihan yang telah ditentukan kepada pengguna saat bertanya, Anda dapat menggunakan _method_ `choice`. Anda dapat menentukan indeks `array` dari nilai `default` yang akan dikembalikan jika tidak ada opsi yang dipilih dengan mengirimkan indeks sebagai argumen ketiga ke _method_:

    $name = $this->choice(
        'Siapa nama Anda?',
        ['Taylor', 'Dayle'],
        $defaultIndex
    );

Selain itu, _method_ `choice` menerima argumen keempat dan kelima yang opsional untuk menentukan jumlah maksimum percobaan untuk memilih tanggapan yang valid dan apakah pemilihan bersama diizinkan:

    $name = $this->choice(
        'Siapa nama Anda?',
        ['Taylor', 'Dayle'],
        $defaultIndex,
        $maxAttempts = null,
        $allowMultipleSelections = false
    );

<a name="writing-output"></a>
### Menulis _Output_

Untuk mengirim _output_ ke _console_, Anda dapat menggunakan _method_ `line`, `info`, `comment`, `question`, `warn`, dan `error`. Setiap _method_ ini akan menggunakan warna ANSI yang sesuai untuk tujuannya. Sebagai contoh, mari kita tampilkan beberapa informasi umum kepada pengguna. Biasanya, _method_ `info` akan ditampilkan di _console_ sebagai teks berwarna hijau:

    /**
     * Eksekusi perintah console.
     *
     * @return mixed
     */
    public function handle()
    {
        // ...

        $this->info('Perintah berjalan sukses!');
    }

Untuk menampilkan pesan eror, gunakan _method_ `error`. Pesan eror biasanya ditampilkan dalam warna merah:

    $this->error('Terjadi kesalahan!');

Anda dapat menggunakan _method_ `line` untuk menampilkan teks tanpa warna:

    $this->line('Tampilkan ini di layar');

Anda dapat menggunakan _method_ `newLine` untuk menampilkan baris kosong:

    // Menampilkan satu baris kosong...
    $this->newLine();

    // Menampilkan tiga baris kosong...
    $this->newLine(3);

<a name="tables"></a>
#### Tabel

_Method_ `table` memudahkan pengaturan format yang benar untuk beberapa baris / kolom data. Yang perlu Anda lakukan hanyalah memberikan nama kolom dan data untuk tabel, dan Laravel akan secara otomatis menghitung lebar dan tinggi tabel yang sesuai untuk Anda:

    use App\Models\User;

    $this->table(
        ['Name', 'Email'],
        User::all(['name', 'email'])->toArray()
    );

<a name="progress-bars"></a>
#### _Progress Bar_

Untuk tugas yang berlangsung lama, Anda dapat menampilkan _progress bar_ yang memberi tahu pengguna seberapa jauh tugas telah dilaksanakan. Dengan menggunakan _method_ `withProgressBar`, Laravel akan menampilkan _progress bar_ dan menambah nilai progresnya untuk setiap iterasi atas nilai _iterable_ yang diberikan:

    use App\Models\User;

    $users = $this->withProgressBar(User::all(), function ($user) {
        $this->performTask($user);
    });

Kadang-kadang, Anda mungkin membutuhkan kontrol yang lebih manual untuk mengendalikan _progress bar_. Pertama, tentukan jumlah langkah yang akan dilalui proses. Kemudian, tingkatkan _progress bar_ setelah memproses masing-masing _item_:

    $users = App\Models\User::all();

    $bar = $this->output->createProgressBar(count($users));

    $bar->start();

    foreach ($users as $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

> **Catatan**  
> Untuk opsi tingkat lanjut, sila periksa [Symfony Progress Bar component documentation](https://symfony.com/doc/current/components/console/helpers/progressbar.html).

<a name="registering-commands"></a>
## Mendaftarkan Perintah

Semua perintah _console_ Anda terdaftar dalam _class_ `App\console\Kernel`, yang merupakan "console kernel" aplikasi Anda. Di dalam _method_ `commands`, Anda akan melihat sebuah pemanggilan ke _method_ `load` milik _kernel_. _Method_ `load` akan memindai direktori `app/console/Commands` dan secara otomatis mendaftarkan setiap perintah yang ditemukan ke Artisan. Anda bahkan bebas membuat panggilan tambahan ke _method_ `load` untuk memindai direktori lain untuk perintah Artisan:

    /**
     * Mendaftarkan perintah untuk aplikasi.
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');
        $this->load(__DIR__.'/../Domain/Orders/Commands');

        // ...
    }

Jika perlu, Anda dapat mendaftarkan perintah secara manual dengan menambahkan nama _class_ dari perintah ke properti `$commands` di dalam _class_ `App\console\Kernel` Anda. Jika properti ini belum didefinisikan pada _kernel_ Anda, Anda harus mendefinisikannya secara manual. Saat Artisan melakukan _boot_, semua perintah yang terdaftar dalam properti ini akan diselesaikan oleh [_service container_](/docs/{{version}}/container) dan didaftarkan dengan Artisan:

    protected $commands = [
        Commands\SendEmails::class
    ];

<a name="programmatically-executing-commands"></a>
## Mengeksekusi Perintah Secara Terprogram

Terkadang Anda mungkin ingin mengeksekusi perintah Artisan di luar CLI. Sebagai contoh, Anda mungkin ingin mengeksekusi perintah Artisan dari _route_ atau _controller_. Anda dapat menggunakan _method_ `call` pada _facade_ Artisan untuk mencapainya. _Method_ `call` menerima nama _signature_ atau nama _class_ sebagai argumen pertamanya, dan sebuah _array_ berisi parameter perintah sebagai argumen kedua. Kode keluar akan dikembalikan:

    use Illuminate\Support\Facades\Artisan;

    Route::post('/user/{user}/mail', function ($user) {
        $exitCode = Artisan::call('mail:send', [
            'user' => $user, '--queue' => 'default'
        ]);

        //
    });

Atau, Anda dapat mengirimkan perintah Artisan seluruhnya ke _method_ `call` dalam bentuk _string_:

    Artisan::call('mail:send 1 --queue=default');

<a name="passing-array-values"></a>
#### Melewatkan Nilai _Array_

Jika perintah Anda mendefinisikan sebuah opsi yang menerima _array_, Anda dapat mengirimkan sebuah _array_ ke opsi tersebut:

    use Illuminate\Support\Facades\Artisan;

    Route::post('/mail', function () {
        $exitCode = Artisan::call('mail:send', [
            '--id' => [5, 13]
        ]);
    });

<a name="passing-boolean-values"></a>
#### Melewatkan Nilai _Boolean_

Jika Anda perlu menentukan nilai dari sebuah opsi yang tidak menerima nilai _string_, seperti (_flag_) `--force` pada perintah `migrate:refresh`, Anda harus mengirimkan `true` atau `false` sebagai nilai opsi tersebut:

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

<a name="queueing-artisan-commands"></a>
#### Membuat Antrian Perintah Artisan

Dengan menggunakan _method_ `queue` pada _facade_ `Artisan`, Anda bahkan dapat mengelola perintah Artisan sehingga dapat diproses di latar belakang oleh [_queue workers_](/docs/{{version}}/queues). Sebelum menggunakan _method_ ini, pastikan Anda telah mengkonfigurasi antrian dan menjalankan _queue listener_:

    use Illuminate\Support\Facades\Artisan;

    Route::post('/user/{user}/mail', function ($user) {
        Artisan::queue('mail:send', [
            'user' => $user, '--queue' => 'default'
        ]);

        //
    });

Dengan menggunakan _method_ `onConnection` dan `onQueue`, Anda dapat menentukan koneksi atau antrian perintah Artisan yang harus dikirimkan:

    Artisan::queue('mail:send', [
        'user' => 1, '--queue' => 'default'
    ])->onConnection('redis')->onQueue('commands');

<a name="calling-commands-from-other-commands"></a>
### Memanggil Perintah di Dalam Perintah

Terkadang Anda mungkin ingin memanggil perintah lain dari perintah Artisan yang ada. Anda dapat melakukannya dengan menggunakan _method_ `call`. _Method_ `call` ini menerima nama perintah dan sebuah _array_ untuk argumen / opsi:

    /**
     * Eksekusi perintah console.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->call('mail:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    }

Jika Anda ingin memanggil perintah _console_ lain dan tidak ingin menampilkan _output_ dari perintah tersebut, Anda dapat menggunakan _method_ `callSilently`. _Method_ `callSilently` memiliki _signature_ yang sama dengan _method_ `call`:

    $this->callSilently('mail:send', [
        'user' => 1, '--queue' => 'default'
    ]);

<a name="signal-handling"></a>
## Penanganan Sinyal

Seperti yang Anda tahu, sistem operasi memungkinkan sinyal dikirim ke proses yang sedang berjalan. Misalnya, sinyal `SIGTERM` adalah bagaimana sistem operasi meminta sebuah program untuk dihentikan. Jika Anda ingin mendengarkan sinyal dalam perintah _console_ Artisan Anda dan menjalankan kode saat sinyal terjadi, Anda dapat menggunakan _method_ `trap`:

    /**
     * Eksekusi perintah console.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->trap(SIGTERM, fn () => $this->shouldKeepRunning = false);

        while ($this->shouldKeepRunning) {
            // ...
        }
    }

Untuk mendengarkan beberapa sinyal sekaligus, Anda dapat menggunakan _array_ berisi sinyal-sinyal ke dalam _method_ `trap`:

    $this->trap([SIGTERM, SIGQUIT], function ($signal) {
        $this->shouldKeepRunning = false;

        dump($signal); // SIGTERM / SIGQUIT
    });

<a name="stub-customization"></a>
## Kostumisasi _Stub_

Perintah `make` pada _console_ Artisan digunakan untuk membuat berbagai macam _class_, seperti _controller_, _job_, _migrations_, dan _tests_. Kelas-kelas ini dibuat menggunakan _file_ "stub" yang diisi berdasarkan nilai-nilai yang di-_input_-kan. Namun, Anda mungkin ingin membuat perubahan kecil pada _file_ yang dihasilkan oleh Artisan. Untuk mencapai ini, Anda dapat menggunakan perintah `stub:publish` untuk mempublikasikan _stub_ ke aplikasi Anda sehingga Anda dapat mengubahnya untuk disesuikan:

```shell
php artisan stub:publish
```

Stub yang dipublis akan terletak di dalam direktori `stubs` pada _root_ aplikasi Anda. Segala perubahan yang Anda lakukan pada _stub_ tersebut akan langsung tercermin saat Anda menghasilkan _class_ menggunakan perintah `make` milik Artisan.

<a name="events"></a>
## _Event_

Artisan mengirimkan tiga _event_ saat menjalankan perintah: `Illuminate\console\Events\ArtisanStarting`, `Illuminate\console\Events\CommandStarting`, dan `Illuminate\console\Events\CommandFinished`. _Event_ `ArtisanStarting` dikirimkan segera saat Artisan mulai berjalan. Kemudian, _event_ `CommandStarting` dikirimkan segera sebelum sebuah perintah dijalankan. Terakhir, _event_ `CommandFinished` dikirimkan setelah sebuah perintah selesai dieksekusi.
