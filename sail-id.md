# Laravel Sail

- [Pengenalan](#introduction)
- [Instal & Konfigurasi](#installation)
    - [Instal Sail ke Aplikasi yang sudah ada](#installing-sail-into-existing-applications)
    - [konfigurasi Shell dengan Alias](#configuring-a-shell-alias)
- [Menjalankan dan menghentikan Sail](#starting-and-stopping-sail)
- [Menjalankan Perintah](#executing-sail-commands)
    - [Mengeksekusi Perintah PHP](#executing-php-commands)
    - [Mengeksekusi Perintah Composer](#executing-composer-commands)
    - [Mengeksekusi Perintah Artisan](#executing-artisan-commands)
    - [Mengeksekusi Node / NPM](#executing-node-npm-commands)
- [Berinteraksi Dengan Database](#interacting-with-sail-databases)
    - [MySQL](#mysql)
    - [Redis](#redis)
    - [MeiliSearch](#meilisearch)
- [Penyimpanan File](#file-storage)
- [Menjalankan Tes](#running-tests)
    - [Laravel Dusk](#laravel-dusk)
- [Pengecekan Email](#previewing-emails)
- [Container CLI](#sail-container-cli)
- [Versi PHP](#sail-php-versions)
- [Versi Node](#sail-node-versions)
- [Membagikan Situs](#sharing-your-site)
- [Debug Menggunakan Xdebug](#debugging-with-xdebug)
  - [Perintah Xdebug CLI](#xdebug-cli-usage)
  - [Perintah Xdebug Browser](#xdebug-browser-usage)
- [Kustomisasi](#sail-customization)

<a name="introduction"></a>
## Pengenalan

[Laravel Sail](github.com/laravel/sail) adalah antarmuka baris perintah yang ringan untuk berinteraksi dengan lingkungan pengembangan Docker bawaan Laravel. Sail menyediakan titik awal yang bagus untuk membangun aplikasi Laravel menggunakan PHP, MySQL, dan Redis tanpa perlu pengalaman sebelumnya dengan Docker.

Intinya, Sail adalah file `docker-compose.yml` dan `sail` _script_ yang tersimpan di dalam _root_ folder Anda, `Sail` _script_ menyediakan CLI dengan pengopreasian yang mudah untuk berinteraksi dengan _docker container_ yang dipanggil melalui file`docker-compose.yml`.

Laravel Sail mendukung macOS, Linux, dan Windows (via[WSL2](https://docs.microsoft.com/en-us/windows/wsl/about)).

<a name="installation"></a>
## Instalasi Dan Konfigurasi

Secara otomatis, Laravel Sail terinstal dengan semua aplikasi Laravel dan anda bisa menggunakannya secara langsung. Untuk mempelajari bagaimana untuk membuat Aplikasi Laravel baru, silahkan buka [Intalasi Laravel](/docs/{{version}}/installation) tergantung sistem operasi anda. Selama instalasi, Anda akan diberikan pertanyaan untuk memilih _Sail_ mana yang didukung oleh mesin anda.

<a name="installing-sail-into-existing-applications"></a>
### Instal Sail ke Aplikasi yang sudah ada

Jika anda tertarik untuk menggunakan Sail dengan aplikasi Laravel yang sudah ada, Anda mungkin menginstal Sail menggunakan _composer package manager_. Tentu saja, langkah ini dilakukan jika anda sudah memiliki _environment_ lokal yang mengijinkan anda untuk menginstal _Composer dependencies_:

```shell
composer require laravel/sail --dev
```

setelah Sail terinstal, silahkan untuk menjalankan perintah _artisan_ `sail:install`. Perintah ini untuk menambahkan file `docker-compose.yml` didalam file _root_ pada aplikasi Anda.

```shell
php artisan sail:install
```

Silahkan mulai `Sail` anda, untuk mempelajari lebih lanjut bagaimana cara menggunakan `Sail`, Baca sisa dari dokumentasi ini:

```shell
./vendor/bin/sail up
```

<a name="adding-additional-services"></a>
#### Menambahkan Layanan tambahan

Jika Anda ingin menambakan layanan pada instalasi `Sail`, silahkan jalankan perintah `sail:add` diartisan:

```shell
php artisan sail:add
```

<a name="using-devcontainers"></a>
#### Menggunakan Devcontainer

Jika anda ingin mengembangkan [Devcontainer](https://code.visualstudio.com/docs/remote/containers), Anda bisa menambahakan `--devcontainer` pada perintah `sail:install`. opsi `--devcontainer` akan menginstruksikan `sail:install` untuk menampilkan standar bawaan `.devcontainer/devcontainer.json` didalam _akar_ pada aplikasi anda.

```shell
php artisan sail:install --devcontainer
```

<a name="configuring-a-shell-alias"></a>
### Konfigurasi Shell Alias

Secara bawaan, Perintah Sail dipanggil menggunakan script `vendor/bin/sail` yang sudah termasuk dalam Aplikasi bawaan Laravel:

```shell
./vendor/bin/sail up
```

Daripada Anda mengulangi `vendor/bin/sail` untuk mengeksekusi perintah `Sail`, silahkan untuk melakukan konfigurasi `shell` yang memungkinkan anda untuk menjalankan perintah `Sail` lebih mudah.

```shell
alias sail='[ -f sail ] && sh sail || sh vendor/bin/sail'
```

Untuk memastikan jika selalu dijalankan, Silahknan tambahkan konfigurasi ini didalam direktori `Home`, seperti `~./zshrc` atau `~./bashrc`, lalu jalankan ulang `shell` anda.

Setelah `Shell` alias telah dikonfigurasi, silahkan mengeksekusi perintah Sail anda dengan mengetikkan `sail`. Sisa dokumentasi ini mencontohkan jika anda sudah melakukan konfigurasi alias:

```shell
sail up
```

<a name="starting-and-stopping-sail"></a>
## Menjalankan dan menghentikan Sail

file `docker-compose.yml` memiliki bermacam-macam `Docker Container` yang berjalan beriringan untuk mendukung membuat Aplikasi Laravel. Masing-masing kontainer ini merupakan entri dalam konfigurasi `services` pada berkas `docker-compose.yml` Anda. kontainer `laravel.test` adalah Aplikasi utama yang akan menampilkan Aplikasi anda.

Sebelum menjalankan Sail, pastikan tidak ada _web server_ atau _database_ yang sedang berjalan. Untuk memulai semua kontainer Docker yang didefinisikan dalam berkas `docker-compose.yml` aplikasi Anda, Anda harus menjalankan perintah `up`:

```shell
sail up -d
```

Setelah aplikasi _container_ dijalankan, silahakn mengakses aplikasi anda di peramban anda di : http://localhost.

Untuk menghentikan semua _container_, silahkan menekan `Control + C`. Atau jika _container_ sedang dijalankan dilatar belakang, silahkan jalankan perintah `stop`.

```shell
sail stop
```

<a name="executing-sail-commands"></a>
## Menjalankan Perintah

Ketika menggunakan Laravel Sail, aplikasi Anda dieksekusi di dalam _Docker Container_ dan terdapat dari komputer lokal Anda. Namun, `Sail` menyediakan cara yang mudah untuk menjalankan berbagai perintah terhadap aplikasi Anda seperti perintah PHP _arbitrer_, _Artisan_, _Composer_ dan _Node/NPM_.

**Ketika membaca dokumentasi Laravel, seringkali Anda akan menemukan perintah _Composer_, _Artisan_, dan _Node / NPM_ yang tidak mereferensikan Sail.** Contoh ini mengasumsikan bahwa alat ini telah diinstal di mesin lokal anda. jika anda menggunakan `Sail` di Laravel _developement_, Anda harus mengeksekusi perintah ini menggunakan `Sail`:

```shell
#Menjalankan Artisan secara lokal
php artisan queue:work

sail artisan queue:work
```

<a name="executing-php-commands"></a>
### Mengeksekusi Perintah PHP

Perintah PHP akan menjalankan perintah `php`. Perintah ini akan menjalankan menggunakan versi PHP yang melakukan konfigurasi di aplikasi anda. Untuk mempelajari versi PHP yang tersedia di `Laravel Sail`, silahkan baca [Dokumentasi Konfigurasi PHP](#sail-php-versions):

```shell
sail php --version

sail php script.php
```

<a name="executing-composer-commands"></a>
### Mengeksekusi Perintah _Composer_

Perintah-perintah komposer dapat dijalankan dengan menggunakan perintah `composer`. Kontainer aplikasi Laravel Sail menyertakan instalasi Composer 2.x:

```shell
sail composer require laravel/sanctum
```

<a name="installing-composer-dependencies-for-existing-projects"></a>
#### Instal _Composer Dependencies_ untuk Aplikasi yang sudah ada

Jika Anda mengembangkan aplikasi bersama tim, Anda mungkin bukan orang yang pertama kali membuat aplikasi Laravel. Oleh karena itu, tidak ada dependensi Composer, termasuk Sail, yang akan diinstal setelah Anda _clone_ repositori aplikasi ke komputer lokal.

Silahkan instal aplikasi yang dibutuhkan dengan menavigasi ke direktori aplikasi dan menjalankan dengan perintah yang disediakan. Perintah ini menggunakan sebagian kecil _Docker Container_ yang terdapat PHP dan _Composer_ untuk menginstal aplikasi yang dibutuhkan:

```shell
docker run --rm \
    -u "$(id -u):$(id -g)" \
    -v "$(pwd):/var/www/html" \
    -w /var/www/html \
    laravelsail/php82-composer:latest \
    composer install --ignore-platform-reqs
```

Ketika menggunakan `laravelsail/phpXX-composer`, Anda wajib menggunakan Versi PHP yang sama untuk menjalankan aplikasi Anda 
(`74`, `80`, `81`, or `82`).

<a name="executing-artisan-commands"></a>
### Menjalankan Perintah Artisan

Perintah Artisan akan dijalankan dengan `artisan`:

```shell
sail artisan queue:work
```

<a name="executing-node-npm-commands"></a>
### Menjalankan Perintah Node / NPM

Perintah node akan menjalankan `node` dan perintah NPM dijalankan menggunakan perintah `NPM`:

```shell
sail node --version

sail npm run dev
```

Selain NPM, Anda juga dapat menggunakan Yarn:

```shell
sail yarn
```

<a name="interacting-with-sail-databases"></a>
## Berinteraksi dengan _Database_

<a name="mysql"></a>
### MySQL

Seperti yang anda ketahui, aplikasi anda di `docker-compose.yml` memuat berkas didalam _Container_ MySQL. _Container_ menggunakan [Docker volume](https://docs.docker.com/storage/volumes/) jadi semua data yang tersimpan dalam _database_ akan tetap tersimpan walaupun _container_ sedang di hentikan atau dimuat ulang.

Sebagai tambahan, saat pertama kali MySQL _container_ dijalankan, _container_ akan membuat dua database untuk Anda. _Database_ pertama akan dinamakan value dari nilai `DB_DATABASE` dan _database_ kedua akan dinamakan _testing_, untuk memastikan _database_ yang akan dicoba tidak bertabrakan dengan data _developement_.

Setelah alias shell dikonfigurasi, Anda dapat menjalankan perintah Sail hanya dengan mengetikkan `sail`. Contoh-contoh lain dalam dokumentasi ini akan mengasumsikan bahwa Anda telah mengonfigurasi alias ini:

```shell
sail up
```

<a name="starting-and-stopping-sail"></a>
## Menjalankan dan Menghentikan Sail

