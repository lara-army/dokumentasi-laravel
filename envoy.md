# Laravel Envoy

- [Perkenalan](#introduction)
- [Instalasi](#installation)
- [Menulis _Tasks_](#writing-tasks)
  - [Mendefinisikan _Task_](#defining-tasks)
  - [Beberapa _Servers_](#completion-servers)
  - [_Setup_](#setup)
  - [_Variables_](#variables)
  - [_Stories_](#stories)
  - [_Hooks_](#completion-hooks)
- [Menjalankan _Tasks_](#running-tasks)
  - [Mengonfirmasi Eksekusi _Task_](#confirming-task-execution)
- [Pemberitahuan](#notifications)
  - [Slack](#slack)
  - [Discord](#discord)
  - [Telegram](#telegram)
  - [Microsoft Teams](#microsoft-teams)

<a name="introduction"></a>

## Introduction

[Laravel Envoy](https://github.com/laravel/envoy) adalah alat untuk menjalankan tugas-tugas umum yang Anda jalankan di server jarak jauh Anda. Menggunakan sintaks gaya [Blade](/docs/{{version}}/blade), Anda dapat dengan mudah mengatur tugas untuk _deployment_, perintah Artisan, dan lainnya. Saat ini, Envoy hanya mendukung sistem operasi Mac dan Linux. Namun, dukungan untuk Windows dapat dicapai menggunakan [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

<a name="installation"></a>

## Instalasi

Pertama, instal Envoy ke dalam proyek Anda menggunakan manajer paket Composer:

```shell
composer require laravel/envoy --dev

```

Setelah Envoy diinstal, biner Envoy akan tersedia di direktori `vendor/bin` aplikasi Anda:

```shell
php vendor/bin/envoy
```

<a name="writing-tasks"></a>

## Menulis _Tasks_

<a name="defining-tasks"></a>

### Mendefinsikan _Tasks_

_Task_ adalah blok penyusun dasar dari Envoy. _Task_ mendefinisikan perintah shell yang harus dijalankan di server jarak jauh Anda ketika _task_ tersebut dipanggil. Misalnya, Anda dapat mendefinisikan _task_ yang menjalankan perintah `php artisan queue:restart` pada semua server pekerja antrian aplikasi Anda.

Semua _task_ Envoy Anda harus didefinisikan dalam file 'Envoy.blade.php' di root aplikasi Anda. Berikut adalah contoh untuk membantu Anda memulai:

```blade
@servers(['web' => ['user@192.168.1.1'], 'workers' => ['user@192.168.1.2']])

@task('restart-queues', ['on' => 'workers'])
    cd /home/user/example.com
    php artisan queue:restart
@endtask
```

Seperti yang Anda lihat, array `@servers` didefinisikan di bagian atas file, memungkinkan Anda untuk mereferensikan server ini melalui opsi `on` dari deklarasi _task_ Anda. Deklarasi `@servers` harus selalu ditempatkan pada satu baris. Dalam deklarasi `@task` Anda, Anda harus menempatkan perintah shell yang harus dijalankan di server Anda saat _task_ dipanggil.

<a name="local-tasks"></a>

#### Lokal Tasks

Anda dapat memaksa _script_ untuk dijalankan di komputer lokal Anda dengan menentukan alamat IP server sebagai `127.0.0.1`:

```blade
@servers(['localhost' => '127.0.0.1'])
```

<a name="importing-envoy-tasks"></a>

#### Mengimpor Envoy _Tasks_

Dengan menggunakan direktif `@import`, Anda dapat mengimpor file Envoy lain sehingga _stories_ dan _tasks_ mereka ditambahkan ke file Anda. Setelah file-file tersebut diimpor, Anda dapat menjalankan _tasks_ yang ada di dalamnya seolah-olah _tasks_ tersebut didefinisikan dalam file Envoy Anda sendiri:

```blade
@import('vendor/package/Envoy.blade.php')
```

<a name="completion-servers"></a>

### Beberapa Servers

Envoy memungkinkan Anda menjalankan _task_ dengan mudah di beberapa _servers_. Pertama, tambahkan _servers_ tambahan pada deklarasi `@servers`. Setiap _server_ harus diberi nama yang unik. Setelah Anda mendefinisikan _server_ tambahan, Anda dapat mendaftarkan setiap _server_ dalam _array_ `on` _task_:

```blade
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2']])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

<a name="parallel-execution"></a>

#### Eksekusi _parallel_

Secara bawaan, _tasks_ akan dijalankan di setiap _server_ secara serial. Dengan kata lain, _task_ akan selesai berjalan di _server_ pertama sebelum melanjutkan untuk dijalankan di _server_ kedua. Jika Anda ingin menjalankan _task_ di beberapa _servers_ secara _parallel_, tambahkan opsi `parallel` ke deklarasi _task_ Anda:

```blade
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

<a name="setup"></a>

### _Setup_

Terkadang, Anda mungkin perlu mengeksekusi kode PHP sewenang-wenang sebelum menjalankan _tasks_ Envoy Anda. Anda dapat menggunakan direktif `@setup` untuk mendefinisikan blok kode PHP yang harus dieksekusi sebelum _tasks_ Anda:

```php
@setup
    $now = new DateTime;
@endsetup
```

Jika Anda perlu memerlukan file PHP lain sebelum _task_ Anda dijalankan, Anda dapat menggunakan direktif `@include` di bagian atas file `Envoy.blade.php` Anda:

```blade
@include('vendor/autoload.php')

@task('restart-queues')
    # ...
@endtask
```

<a name="variables"></a>

### Variables

Jika perlu, Anda dapat meneruskan argumen ke _tasks_ Envoy dengan menentukannya pada baris perintah saat memanggil Envoy:

```shell
php vendor/bin/envoy run deploy --branch=master
```

Anda dapat mengakses opsi dalam _tasks_ Anda menggunakan sintaks "echo" Blade. Anda juga dapat menentukan pernyataan Blade `if` dan perulangan dalam _tasks_ Anda. Misalnya, mari kita verifikasi keberadaan variabel `$branch` sebelum menjalankan perintah `git pull`:

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

### Stories

Grup _stories_ serangkaian _tasks_ dengan satu nama yang nyaman. Misalnya, _story_ _deploy_ dapat menjalankan _task_ `update-code` dan `install-dependencies` dengan mencantumkan nama _task_ dalam definisinya:

```blade
@servers(['web' => ['user@192.168.1.1']])

@story('deploy')
    update-code
    install-dependencies
@endstory

@task('update-code')
    cd /home/user/example.com
    git pull origin master
@endtask

@task('install-dependencies')
    cd /home/user/example.com
    composer install
@endtask
```

Setelah _story_ ditulis, Anda dapat memanggilnya dengan cara yang sama seperti Anda memanggil _task_:

```shell
php vendor/bin/envoy run deploy
```

<a name="completion-hooks"></a>

### Hooks

Ketika _tasks_ dan _stories_ berjalan, sejumlah _hooks_ dieksekusi. Jenis _hook_ yang didukung oleh Envoy adalah '@before', '@after', '@error', '@success', dan '@finished'. Semua kode dalam _hooks_ ini ditafsirkan sebagai PHP dan dieksekusi secara lokal, bukan di server jarak jauh yang berinteraksi dengan _tasks_ Anda.

Anda dapat menentukan sebanyak mungkin dari masing-masing _hooks_ ini sesuka Anda. Mereka akan dieksekusi sesuai urutan yang muncul di _script_ Envoy Anda.

<a name="hook-before"></a>

#### `@before`

Sebelum setelah eksekusi _task_, semua _hooks_ '@before' yang terdaftar dalam _script_ Envoy Anda akan dieksekusi. _Hooks_ '@before' menerima nama _task_ yang akan dieksekusi:

```blade
@before
    if ($task === 'deploy') {
        // ...
    }
@endbefore
```

<a name="completion-after"></a>

#### `@after`

Setelah setiap eksekusi _task_, semua _hooks_ `@after` yang terdaftar dalam _script_ Envoy Anda akan dieksekusi. _Hooks_ `@after` menerima nama _task_ yang dieksekusi:

```blade
@after
    if ($task === 'deploy') {
        // ...
    }
@endafter
```

<a name="completion-error"></a>

#### `@error`

Setelah setiap kegagalan _task_ (keluar dengan kode status lebih besar dari '0'), semua _hooks_ `@error` yang terdaftar dalam _script_ Envoy Anda akan dieksekusi. _Hooks_ '@error' menerima nama _task_ yang dieksekusi:

```blade
@error
    if ($task === 'deploy') {
        // ...
    }
@enderror
```

<a name="completion-success"></a>

#### `@success`

Jika semua _task_ telah dijalankan tanpa kesalahan, semua _hooks_ `@success` yang terdaftar di _script_ Envoy Anda akan dijalankan:

```blade
@success
    // ...
@endsuccess
```

<a name="completion-finished"></a>

#### `@finished`

Setelah semua _tasks_ dieksekusi (terlepas dari status keluar), semua _hooks_ `@finished` akan dieksekusi. _Hooks_ '@finished' menerima kode status _tasks_ yang telah selesai, yang mungkin `null` atau `integer` lebih besar dari atau sama dengan `0`:

```blade
@finished
    if ($exitCode > 0) {
        // There were errors in one of the tasks...
    }
@endfinished
```

<a name="running-tasks"></a>

## Menjalankan Tasks

Untuk menjalankan _task_ atau _story_ yang ditentukan dalam file `Envoy.blade.php` aplikasi Anda, jalankan perintah `run` Envoy, teruskan nama _task_ atau _story_ yang ingin Anda jalankan. Envoy akan menjalankan _task_ dan menampilkan _output_ dari server jarak jauh Anda saat _task_ sedang berjalan:

```shell
php vendor/bin/envoy run deploy
```

<a name="confirming-task-execution"></a>

### Mengonfirmasi Eksekusi _Task_

Jika Anda ingin diminta untuk konfirmasi sebelum menjalankan _task_ yang diberikan di _servers_ Anda, Anda harus menambahkan direktif `confirm` ke deklarasi _task_ Anda. Opsi ini sangat berguna untuk operasi destruktif:

```blade
@task('deploy', ['on' => 'web', 'confirm' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate
@endtask
```

<a name="notifications"></a>

## Pemberitahuan

<a name="slack"></a>

### Slack

Envoy mendukung pengiriman pemberitahuan ke [Slack](https://slack.com) setelah setiap _task_ dijalankan. Direktif `@slack` menerima URL _hook_ Slack dan nama saluran/pengguna. Anda dapat mengambil URL webhook Anda dengan membuat integrasi "incoming Webhooks" di panel kontrol Slack Anda.

Anda harus meneruskan seluruh URL webhook sebagai argumen pertama yang diberikan ke direktif `@slack`. Argumen kedua yang diberikan untuk direktif `@slack` harus berupa nama saluran (`#channel`) atau nama pengguna (`@user`):

```blade
@finished
    @slack('webhook-url', '#bots')
@endfinished
```

Secara bawaan, pemberitahuan Envoy akan mengirim pesan ke saluran pemberitahuan yang menjelaskan _task_ yang dijalankan. Namun, Anda dapat menimpa pesan ini dengan pesan kustom Anda sendiri dengan meneruskan argumen ketiga ke direktif `@slack`:

```blade
@finished
    @slack('webhook-url', '#bots', 'Hello, Slack.')
@endfinished
```

<a name="discord"></a>

### Discord

Envoy juga mendukung pengiriman pemberitahuan ke [Discord](https://discord.com) setelah setiap _task_ dijalankan. Direktif `@discord` menerima URL _hook_ Discord dan pesan. Anda dapat mengambil URL webhook Anda dengan membuat "Webhook" di Pengaturan Server Anda dan memilih saluran mana webhook harus diposting. Anda harus meneruskan seluruh URL Webhook ke dalam perintah `@discord`:

```blade
@finished
    @discord('discord-webhook-url')
@endfinished
```

<a name="telegram"></a>

### Telegram

Envoy juga mendukung pengiriman pemberitahuan ke [Telegram](https://telegram.org) setelah setiap _task_ dijalankan. Direktif `@telegram` menerima ID Bot Telegram dan ID Obrolan. Anda dapat mengambil ID Bot Anda dengan membuat bot baru menggunakan [BotFather](https://t.me/botfather). Anda dapat mengambil ID Chat yang valid menggunakan [@username_to_id_bot](https://t.me/username_to_id_bot). Anda harus meneruskan seluruh ID Bot dan ID Obrolan ke dalam perintah `@telegram`:

```blade
@finished
    @telegram('bot-id','chat-id')
@endfinished
```

<a name="microsoft-teams"></a>

### Microsoft Teams

Envoy juga mendukung pengiriman pemberitahuan ke [Microsoft Teams](https://www.microsoft.com/en-us/microsoft-teams) setelah setiap _task_ dijalankan. Direktif `@microsoftTeams` menerima _Teams Webhook_ (wajib), pesan, warna tema (_success_, _info_, _warning, \_error_), dan _array_ opsi. Anda dapat mengambil _Webhook Teams_ Anda dengan membuat [webhook masuk](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook) baru. _Teams API_ memiliki banyak atribut lain untuk menyesuaikan kotak pesan Anda seperti judul, ringkasan, dan bagian. Anda dapat menemukan informasi selengkapnya tentang [dokumentasi Microsoft Teams](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/connectors-using?tabs=cURL#example-of-connector-message). Anda harus meneruskan seluruh URL Webhook ke dalam perintah `@microsoftTeams`:

```blade
@finished
    @microsoftTeams('webhook-url')
@endfinished
```
