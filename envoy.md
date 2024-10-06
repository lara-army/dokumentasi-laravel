# Laravel Envoy

- [Pengantar](#introduction)
- [Instalasi](#installation)
- [Menulis Tugas](#writing-tasks)
  - [Mendefinisikan Tugas](#defining-tasks)
  - [Multi Server](#completion-servers)
  - [Pengaturan](#setup)
  - [Variabel](#variables)
  - [Cerita](#stories)
  - [_Hook_](#completion-hooks)
- [Menjalankan Tugas](#running-tasks)
  - [Mengonfirmasi Eksekusi Tugas](#confirming-task-execution)
- [Notifikasi](#notifications)
  - [Slack](#slack)
  - [Discord](#discord)
  - [Telegram](#telegram)
  - [Microsoft Teams](#microsoft-teams)

<a name="introduction"></a>

## Pengantar

[Laravel Envoy](https://github.com/laravel/envoy) merupakan sebuah alat untuk menjalankan tugas-tugas (_task_) umum yang Anda jalankan pada server jarak jauh milik Anda. Dengan menggunakan sintaks bergaya [Blade](/docs/{{version}}/blade), Anda dapat dengan mudah mengatur tugas-tugas untuk _deployment_, perintah Artisan, dan lainnya. Saat ini, Envoy hanya mendukung sistem operasi Mac dan Linux. Namun, dukungan untuk Windows dapat dicapai menggunakan [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

<a name="installation"></a>

## Instalasi

Pertama, instal Envoy ke dalam proyek Anda menggunakan manajer paket Composer:

```shell
composer require laravel/envoy --dev
```

Setelah Envoy diinstal, _binary_ Envoy akan tersedia pada direktori `vendor/bin` milik aplikasi Anda:

```shell
php vendor/bin/envoy
```

<a name="writing-tasks"></a>

## Menulis Tugas

<a name="defining-tasks"></a>

### Mendefinsikan Tugas

Tugas (_task_) adalah blok penyusun dasar dari Envoy. Tugas tersebut mendefinisikan perintah shell yang harus dijalankan pada server jarak jauh milik Anda ketika tugas tersebut dipanggil. Misalnya, Anda dapat mendefinisikan tugas yang menjalankan perintah `php artisan queue:restart` pada semua server pekerja antrian (_queue worker_) milik aplikasi Anda.

Semua tugas Envoy Anda harus didefinisikan dalam _file_ 'Envoy.blade.php' pada _root_ aplikasi Anda. Berikut adalah contoh untuk membantu Anda memulai:

```blade
@servers(['web' => ['user@192.168.1.1'], 'workers' => ['user@192.168.1.2']])

@task('restart-queues', ['on' => 'workers'])
    cd /home/user/example.com
    php artisan queue:restart
@endtask
```

Seperti yang Anda lihat, larik pada `@servers` didefinisikan pada bagian atas _file_, memungkinkan Anda untuk mereferensikan server-server tersebut melalui opsi `on` dari deklarasi tugas Anda. Deklarasi `@servers` harus selalu ditempatkan pada satu baris. Dalam deklarasi `@task` Anda, Anda harus menempatkan perintah shell yang perlu dijalankan pada server Anda saat tugas dipanggil.

<a name="local-tasks"></a>

#### Tugas Lokal

Anda dapat memaksa sebuah skrip untuk dijalankan pada komputer lokal Anda dengan menentukan alamat IP server menjadi `127.0.0.1`:

```blade
@servers(['localhost' => '127.0.0.1'])
```

<a name="importing-envoy-tasks"></a>

#### Mengimpor Tugas Envoy

Dengan menggunakan direktif `@import`, Anda dapat mengimpor _file_ Envoy lain sehingga cerita (_story_) dan tugas (_task_) lain tersebut ditambahkan pada Envoy milik Anda. Setelah _file_-_file_ tersebut diimpor, Anda dapat menjalankan tugas-tugas yang ada di dalamnya seolah-olah tugas-tugas tersebut telah didefinisikan di dalam _file_ Envoy milik Anda sendiri:

```blade
@import('vendor/package/Envoy.blade.php')
```

<a name="multiple-servers"></a>

### Multi Servers

Envoy memungkinkan Anda untuk menjalankan tugas dengan mudah pada beberapa server. Pertama, masukkan server tambahan pada deklarasi `@servers`. Setiap server harus diberi nama yang unik. Setelah Anda mendefinisikan server tambahan, Anda dapat mendaftarkan setiap server dalam larik `on` milik tugas:

```blade
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2']])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

<a name="parallel-execution"></a>

#### Eksekusi Paralel

Secara _default_, tugas-tugas akan dijalankan pada setiap _server_ secara serial. Dengan kata lain, tugas akan selesai berjalan pada server pertama sebelum melanjutkan untuk dijalankan pada server ke-dua. Jika Anda ingin menjalankan tugas pada beberapa server secara paralel, tambahkan opsi `parallel` pada deklarasi tugas Anda:

```blade
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

<a name="setup"></a>

### Pengaturan

Terkadang, Anda mungkin perlu mengeksekusi kode PHP secara bebas (_arbitrary_) sebelum menjalankan tugas Envoy milik Anda. Anda dapat menggunakan direktif `@setup` untuk mendefinisikan blok kode PHP yang harus dieksekusi sebelum mengeksekusi tugas milik Anda:

```php
@setup
    $now = new DateTime;
@endsetup
```

Jika Anda perlu memasukkan _file_ PHP lain sebelum tugas Anda dijalankan, Anda dapat menggunakan direktif `@include` pada bagian atas _file_ `Envoy.blade.php` milik Anda:

```blade
@include('vendor/autoload.php')

@task('restart-queues')
    # ...
@endtask
```

<a name="variables"></a>

### Variabel

Jika diperlukan, Anda dapat menyertakan argumen ke tugas Envoy dengan menuliskannya pada baris perintah saat memanggil Envoy:

```shell
php vendor/bin/envoy run deploy --branch=master
```

Anda dapat mengakses opsi tersebut dari dalam tugas milik Anda menggunakan sintaks "echo" milik Blade. Anda juga dapat mendefinisikan pernyataan Blade `if` dan perulangan di dalam tugas Anda. Misalnya, mari kita verifikasi keberadaan variabel `$branch` sebelum menjalankan perintah `git pull`:

```blade
@servers(['web' => ['user@192.168.1.1']])

@task('deploy', ['on' => 'web'])
    cd /home/user/example.com

    @if ($branch)
        git pull origin {{ $branch }}
    @endif

    php artisan migrate --force
@endtask
```

<a name="stories"></a>

### Cerita

Cerita (_story_) mengelompokkan serangkaian tugas dengan satu nama untuk kenyamanan. Misalnya, cerita _deploy_ dapat menjalankan tugas `pengkinian-kode` dan `instal-paket-paket-dependensi` dengan mencantumkan nama tugas dalam definisinya:

```blade
@servers(['web' => ['user@192.168.1.1']])

@story('deploy')
    pengkinian-kode
    instal-paket-paket-dependensi
@endstory

@task('pengkinian-kode')
    cd /home/user/example.com
    git pull origin master
@endtask

@task('instal-paket-paket-dependensi')
    cd /home/user/example.com
    composer install
@endtask
```

Setelah cerita ditulis, Anda dapat memanggilnya dengan cara yang sama seperti Anda memanggil tugas:

```shell
php vendor/bin/envoy run deploy
```

<a name="completion-hooks"></a>

### _Hook_

Ketika tugas dan cerita berjalan, sejumlah _hook_ telah dieksekusi. Jenis _hook_ yang didukung oleh Envoy adalah '@before', '@after', '@error', '@success', dan '@finished'. Semua kode dalam _hook_ ini ditafsirkan sebagai PHP dan dieksekusi secara lokal, bukan pada server jarak jauh yang berinteraksi dengan tugas Anda.

Anda dapat mendefinisikan sebanyak mungkin untuk masing-masing _hook_ ini sesuka Anda. Mereka akan dieksekusi sesuai urutan yang muncul pada skrip Envoy milik Anda.

<a name="hook-before"></a>

#### `@before`

Sebelum eksekusi tugas, semua _hook_ '@before' yang terdaftar dalam skrip Envoy Anda akan dieksekusi. _Hook_ '@before' menerima nama tugas yang akan dieksekusi:

```blade
@before
    if ($task === 'deploy') {
        // ...
    }
@endbefore
```

<a name="completion-after"></a>

#### `@after`

Setelah eksekusi tugas, semua _hook_ `@after` yang terdaftar dalam skrip Envoy Anda akan dieksekusi. _Hook_ `@after` menerima nama tugas yang dieksekusi:

```blade
@after
    if ($task === 'deploy') {
        // ...
    }
@endafter
```

<a name="completion-error"></a>

#### `@error`

Setiap kegagalan tugas (keluar dengan kode status lebih besar dari `0`), semua _hook_ `@error` yang terdaftar dalam skrip Envoy Anda akan dieksekusi. _Hook_ '@error' menerima nama tugas yang dieksekusi:

```blade
@error
    if ($task === 'deploy') {
        // ...
    }
@enderror
```

<a name="completion-success"></a>

#### `@success`

Jika semua tugas telah dijalankan tanpa kesalahan, semua _hook_ `@success` yang terdaftar dalam skrip Envoy Anda akan dijalankan:

```blade
@success
    // ...
@endsuccess
```

<a name="completion-finished"></a>

#### `@finished`

Setelah semua tugas dieksekusi (terlepas dari status keluar), semua _hook_ `@finished` akan dieksekusi. _Hook_ '@finished' menerima kode status tugas yang telah selesai, yang mungkin `null` atau `integer` yang lebih besar dari atau sama dengan `0`:

```blade
@finished
    if ($exitCode > 0) {
        // There were errors in one of the tasks...
    }
@endfinished
```

<a name="running-tasks"></a>

## Menjalankan Tugas

Untuk menjalankan tugas atau cerita yang didefinisikan dalam _file_ `Envoy.blade.php` milik aplikasi Anda, jalankan perintah `run` Envoy, diiringi dengan nama tugas atau cerita yang ingin dijalankan. Envoy akan menjalankan tugas dan menampilkan _output_ dari server jarak jauh Anda saat tugas sedang berjalan:

```shell
php vendor/bin/envoy run deploy
```

<a name="confirming-task-execution"></a>

### Mengonfirmasi Eksekusi Tugas

Jika Anda ingin menampilkan permintaan konfirmasi sebelum menjalankan tugas yang diberikan pada server, Anda harus menambahkan direktif `confirm` ke deklarasi tugas Anda. Opsi ini sangat berguna untuk operasi yang bersifat destruktif:

```blade
@task('deploy', ['on' => 'web', 'confirm' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate
@endtask
```

<a name="notifications"></a>

## Notifikasi

<a name="slack"></a>

### Slack

Envoy mendukung pengiriman notifikasi ke [Slack](https://slack.com) setelah masing-masing tugas selesai dijalankan. Direktif `@slack` menerima URL _hook_ Slack dan nama saluran/pengguna. Anda dapat mengambil URL _webhook_ Anda dengan membuat integrasi "incoming Webhooks" pada panel kontrol Slack Anda.

Anda perlu menyertakan seluruh URL _webhook_ sebagai argumen pertama yang diberikan ke direktif `@slack`. Argumen kedua yang diberikan untuk direktif `@slack` harus berupa nama saluran (`#channel`) atau nama pengguna (`@user`):

```blade
@finished
    @slack('webhook-url', '#bots')
@endfinished
```

Secara _default_, notifikasi Envoy akan mengirim pesan ke saluran notifikasi yang menjelaskan tugas yang dijalankan. Namun, Anda dapat menimpa pesan ini dengan pesan kustom Anda sendiri dengan menyertakan argumen ketiga pada direktif `@slack`:

```blade
@finished
    @slack('webhook-url', '#bots', 'Hello, Slack.')
@endfinished
```

<a name="discord"></a>

### Discord

Envoy juga mendukung pengiriman notifikasi ke [Discord](https://discord.com) setelah masing-masing tugas selesai dijalankan. Direktif `@discord` menerima sebuah URL _hook_ Discord dan sebuah pesan. Anda dapat mengambil URL _webhook_ Anda dengan membuat "Webhook" pada Pengaturan server Anda dan memilih ke saluran mana yang _webhook_ perlu kirim. Anda harus menyertakan seluruh URL _Webhook_ ke dalam perintah `@discord`:

```blade
@finished
    @discord('discord-webhook-url')
@endfinished
```

<a name="telegram"></a>

### Telegram

Envoy juga mendukung pengiriman notifikasi ke [Telegram](https://telegram.org) setelah masing-masing tugas selesai dijalankan. Direktif `@telegram` menerima ID Bot Telegram dan ID Obrolan. Anda bisa mendapatkan ID Bot Anda dengan membuat bot baru menggunakan [BotFather](https://t.me/botfather). Anda bisa mendapatkan ID Chat yang valid menggunakan [@username_to_id_bot](https://t.me/username_to_id_bot). Anda harus menyertakan seluruh ID Bot dan ID Obrolan ke dalam direktif `@telegram`:

```blade
@finished
    @telegram('bot-id','chat-id')
@endfinished
```

<a name="microsoft-teams"></a>

### Microsoft Teams

Envoy juga mendukung pengiriman pemberitahuan ke [Microsoft Teams](https://www.microsoft.com/en-us/microsoft-teams) setelah masing-masing tugas selesai dijalankan. Direktif `@microsoftTeams` menerima _Webhook_ Teams (wajib), sebuah pesan, warna tema (_success_, _info_, _warning_, _error_), dan sebuah larik untuk opsi. Anda bisa menemukan _Webhook_ Teams Anda dengan membuat sebuah [_webhook incoming_](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook) yang baru. API milik Teams memiliki banyak atribut lain untuk menyesuaikan kotak pesan Anda seperti judul, ringkasan, dan bagian-bagian. Anda dapat menemukan informasi selengkapnya pada [dokumentasi Microsoft Teams](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/connectors-using?tabs=cURL#example-of-connector-message). Anda perlu menyertakan seluruh URL _Webhook_ ke dalam direktif `@microsoftTeams`:

```blade
@finished
    @microsoftTeams('webhook-url')
@endfinished
```
