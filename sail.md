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

Sail adalah file `docker-compose.yml` dan `sail` _script_ yang tersimpan di dalam _root_ folder Anda, `Sail` _script_ menyediakan CLI dengan pengopreasian yang mudah untuk berinteraksi dengan _docker container_ yang dipanggil melalui file`docker-compose.yml`.

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

File Laravel Sail `docker-compose.yml` memuat berbagai macam docker _container_ yang menjalankan Project anda untuk membangun aplikasi Laravel. Tiap _container_ adalah _entry_ yang terdapat konfigurasi `services` di dalam `docker-compose.yml` untuk menjalankan aplikasi.

Sebelum menjalankan Sail, baiknya tidak ada _web server_ atau _database_ yang berjalan di komputer lokal anda. Untuk menjalankan Docker _container_ pada `docker-compose.yml` anda wajib untuk menjalankan perintah

```shell
sail up
```

Untuk menjalankan semua _Docker Container_ di latar belakang, silahkan menambahkan perintah "detached":

```shell
sail up -d
```

Waktu aplikasi _container_ berjalan, silahkan akses aplikasi anda melalui web browser di : http://localhost.

Untuk menghentikan semua _container yang berjalan, silahkan untuk menekan Control + C, atau bisa juga menggunakan perintah `stop`:

```shell
sail stop
```

<a name="executing-sail-commands"></a>
## Menjalankan Perintah

Ketika menggunakan Laravel Sail, aplikasi anda dijalankan menggunakan _Docker Container_ yang terpisah dari komputer lokal Anda. Bagaimanapun, Sail menyediakan metode yang efisien untuk menjalankan semua perintah didalam aplikasi anda seperti perintah `php artisan` yang pasti digunakan seperti Perintah _Artisan_, _Composer_ dan _Node / NPM_.

**Saat membaca dokumentasi Laravel, Anda akan sering menemukan referensi untuk _Composer_, _Artisan_ dan _Node / NPM_ yang tidak merujuk ke _Sail_** Contoh tersebut menjelaskan jika _Docker_ telah terinstal pada komputer lokal Anda. Jika Anda mengunakan Sail sebagai Metode _developement_ anda, silahkan jalankan perintah ini menggunakan Sail:

```shell
# Menjalankan perintah Artisan secara lokal ....
php artisan queue:work

# Menjalankan perintah Artisan dengan Laravel Sail ....
sail artisan queue:work
```

<a name="executing-php-commands"></a>
### Menjalankan Perintah PHP

Perintah PHP akan dijalankan menggunakan perintah `php`. Tentu saja, perintah ini akan menjalankan Versi PHP mana yang terkonfigurasi di Aplikasi anda. Untuk mempelajari lebih lanjut tentang Versi PHP yang dapat berjalan di Laravel Sail, silahkan baca [Dokumentasi Versi PHP](#sail-php-version)

```shell
sail php --version

sail php script.php
```

<a name="executing-composer-commands"></a>
### Menajalankan Perintah Composer

Peintah Composer akan dijalankan menggunakan perintah `composer`. Laravel Sail memuat _container_ termasuk Instalasi _Composer 2.x_:

```shell
sail composer require laravel/sanctum
```

<a name="installing-composer-dependencies-for-existing-projects"></a>
#### Instalasi Composer Pada Aplikasi Yang Sudah Ada

Jika Anda mengembangkan aplikasi bersama tim, Anda mungkin bukan orang yang pertama kali membuat aplikasi Laravel. Oleh karena itu, tidak ada dependensi Composer aplikasi, termasuk Sail, yang akan terinstal setelah Anda mengkloning repositori aplikasi ke komputasi lokal Anda.

Anda dapat menginstal dependensi aplikasi dengan menavigasi ke direktori aplikasi dan menjalankan perintah berikut. Perintah ini menggunakan sebagian kecil Docker container yang berisi PHP dan Composer untuk menginstal dependensi aplikasi:

```shell
docker run --rm \
        -u "$(id -u):$(id -g)" \
    -v "$(pwd):/var/www/html" \
    -w /var/www/html \
    laravelsail/php82-composer:latest \
    composer install --ignore-platform-reqs
```

Saat menggunakan _image_ `laravelsail/phpXX-composer`, anda wajid untuk menggunakan Versi PHP yang sama untuk digunakan pada aplikasi anda (`74`, `80`, `81`, or `82`).

<a name="executing-artisan-commands"></a>
### Mengeksekusi Perrintah Artisan

Perintah Laravel Artisan bisa dijalankan dengan perintah `artisan`:

```shell
sail artisan queue:work
```

<a name="executing-node-npm-commands"></a>
### Menjalankan Perintah Node / NPM

Perintah Node dijalankan menggunakan perintah `node` dan perintah NPM dijalankan dengan perintah `npm`:

```shell
sail node --version

sail npm run dev
```

Jika Anda ingin menggunakna Yarn silahkan gunakan:

```shell
sail yarn
```

<a name="interacting-with-sail-databases"></a>
## Berinteraksi dengan Database

<a name="mysql"></a>
### MySQL

Seperti yang Anda ketahui, berkas `docker-compose.yml` aplikasi Anda berisi entri untuk kontainer MySQL. Kontainer ini menggunakan [volume Docker](https://docs.docker.com/storage/volumes/) sehingga data yang tersimpan di dalam basis data Anda akan tetap ada, bahkan ketika kontainer Anda dihentikan dan dimulai ulang.

Selain itu, saat pertama kali kontainer MySQL dijalankan, kontainer ini akan membuat dua basis data untuk Anda. Basis data pertama diberi nama dengan menggunakan nilai variabel lingkungan `DB_DATABASE` dan digunakan untuk pengembangan lokal. Database kedua adalah database pengujian khusus yang diberi nama `testing` dan akan memastikan bahwa pengujian Anda tidak mengganggu data pengembangan Anda.

Setelah Anda memulai kontainer, Anda dapat menyambungkan ke instance MySQL di dalam aplikasi Anda dengan mengatur variabel lingkungan `DB_HOST` di dalam file `.env` aplikasi Anda ke `mysql`.

Untuk menyambung ke database MySQL aplikasi Anda dari mesin lokal, Anda dapat menggunakan aplikasi manajemen database grafis seperti [TablePlus](https://tableplus.com). Secara default, basis data MySQL dapat diakses di `localhost` port 3306 dan kredensial aksesnya sesuai dengan nilai variabel lingkungan `DB_USERNAME` dan `DB_PASSWORD`. Atau, Anda dapat menyambung sebagai pengguna `root`, yang juga menggunakan nilai variabel lingkungan `DB_PASSWORD` sebagai kata sandinya.

<a name="redis"></a>
### Redis

Berkas `docker-compose.yml` aplikasi Anda juga berisi entri untuk kontainer [Redis](https://redis.io). Kontainer ini menggunakan [Docker volume](https://docs.docker.com/storage/volumes/) sehingga data yang tersimpan di dalam data Redis Anda akan tetap ada, bahkan ketika menghentikan dan memulai ulang kontainer Anda. Setelah Anda memulai kontainer, Anda dapat menyambungkan ke instans Redis di dalam aplikasi Anda dengan menyetel variabel lingkungan `REDIS_HOST` di dalam berkas `.env` aplikasi Anda ke `redis`.

Untuk menyambung ke basis data Redis aplikasi Anda dari mesin lokal, Anda dapat menggunakan aplikasi manajemen basis data grafis seperti [TablePlus](https://tableplus.com). Secara default, basis data Redis dapat diakses di port 6379 `localhost`.

<a name="meilisearch"></a>
### MeiliSearch

Jika Anda memilih untuk menginstal layanan [MeiliSearch](https://www.meilisearch.com) ketika menginstal Sail, berkas `docker-compose.yml` aplikasi Anda akan berisi entri untuk mesin pencari yang kuat ini yang [kompatibel](https://github.com/meilisearch/meilisearch-laravel-scout) dengan [Laravel Scout](/docs/{{versi}}/scout). Setelah Anda memulai kontainer Anda, Anda dapat terhubung ke instance MeiliSearch di dalam aplikasi Anda dengan mengatur variabel lingkungan `MEILISEARCH_HOST` ke `http://meilisearch:7700`.

Dari mesin lokal Anda, Anda dapat mengakses panel administrasi berbasis web MeiliSearch dengan menavigasi ke `http://localhost:7700` di peramban web Anda.

dari mesin lokal anda, silahkan akses MeiliSearch menggunakan peramban anda dengan menavigasi ke `http://localhost:7700` diperamban anda.

<a name="file-storage"></a>
## Penyimpanan Berkas

Jika Anda berencana untuk menggunakan Amazon S3 untuk menyimpan berkas saat menjalankan aplikasi Anda di lingkungan produksinya, Anda mungkin ingin menginstal layanan [MinIO] (https://min.io) saat menginstal Sail. MinIO menyediakan API yang kompatibel dengan S3 yang dapat Anda gunakan untuk mengembangkan secara lokal menggunakan driver penyimpanan file `s3` Laravel tanpa membuat ember penyimpanan “uji coba” di lingkungan S3 produksi Anda. Jika Anda memilih untuk menginstal MinIO saat menginstal Sail, bagian konfigurasi MinIO akan ditambahkan ke berkas `docker-compose.yml` aplikasi Anda.

Secara default, berkas konfigurasi `filesystems` aplikasi Anda telah berisi konfigurasi disk untuk disk `s3`. Selain menggunakan disk ini untuk berinteraksi dengan Amazon S3, Anda dapat menggunakannya untuk berinteraksi dengan layanan penyimpanan berkas yang kompatibel dengan S3 seperti MinIO hanya dengan memodifikasi variabel lingkungan terkait yang mengontrol konfigurasinya. Misalnya, saat menggunakan MinIO, konfigurasi variabel lingkungan sistem berkas Anda harus ditetapkan sebagai berikut:

```ini
FILESYSTEM_DISK=s3
AWS_ACCESS_KEY_ID=sail
AWS_SECRET_ACCESS_KEY=password
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=local
AWS_ENDPOINT=http://minio:9000
AWS_USE_PATH_STYLE_ENDPOINT=true
```

Agar integrasi Flysystem Laravel dapat menghasilkan URL yang tepat ketika menggunakan MinIO, Anda harus mendefinisikan variabel lingkungan `AWS_URL` agar sesuai dengan URL lokal aplikasi Anda dan menyertakan nama bucket di jalur URL:

```ini
AWS_URL=http://localhost:9000/local
```

Anda dapat membuat bucket melalui konsol MinIO, yang tersedia di `http://localhost:8900`. Nama pengguna default untuk konsol MinIO adalah `sail`, sedangkan kata sandi default adalah `password`.

> **Peringatan**
> Membuat URL penyimpanan sementara melalui metode `temporaryUrl` tidak didukung saat menggunakan MinIO.

<a name="running-test"></a>
## Menjalankan Tes

Laravel menyediakan dukungan pengujian yang luar biasa, dan Anda dapat menggunakan perintah `test` dari Sail untuk menjalankan aplikasi Anda [pengujian fitur dan unit](/docs/{{versi}}/testing). Opsi CLI apa pun yang diterima oleh PHPUnit juga dapat diteruskan ke perintah `test`:

```shell
sail test

sail test --group orders
```

Perintah Sail `test` sama dengan menggunakan perintah Artisan `test`:

```shell
sail artisan test
```

Secara default, Sail akan membuat basis data `testing` khusus sehingga pengujian Anda tidak mengganggu kondisi basis data Anda saat ini. Dalam instalasi Laravel default, Sail juga akan mengonfigurasi file `phpunit.xml` Anda untuk menggunakan basis data ini ketika mengeksekusi pengujian Anda:

```xml
<env name="DB_DATABASE" value="testing"/>
```

<a name="laravel-dusk"></a>
### Laravel Dusk

[Laravel Dusk](/docs/{{version}}/dusk) menyediakan otomatisasi peramban dan API pengujian yang ekspresif dan mudah digunakan. Berkat Sail, Anda dapat menjalankan pengujian ini tanpa perlu menginstal Selenium atau alat lain di komputer lokal Anda. Untuk memulai, hapus layanan Selenium dalam berkas `docker-compose.yml` aplikasi Anda:

```yaml
selenium:
    image: 'selenium/standalone-chrome'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

Selanjutnya, pastikan bahwa layanan `laravel.test` dalam berkas `docker-compose.yml` aplikasi Anda memiliki entri `depends_on` untuk `selenium`:

```yaml
depends_on:
    - mysql
    - redis
    - selenium
```

Terakhir, Anda dapat menjalankan rangkaian uji coba Dusk dengan menjalankan Sail dan menjalankan perintah `dusk`:

```shell
sail dusk
```

<a name="selenium-on-apple-silicon"></a>
#### Selenium Pada Apple Silicon

Jika mesin lokal Anda berisi chip Apple Silicon, layanan `selenium` Anda harus menggunakan gambar `selenium/standalone-chromium`

```yaml
selenium:
    image: 'seleniarm/standalone-chromium'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

<a name="previewing-emails"></a>
## Pratinjau Email

File `docker-compose.yml` bawaan Laravel Sail berisi entri layanan untuk [Mailpit](https://github.com/axllent/mailpit). Mailpit mencegah email yang dikirim oleh aplikasi Anda selama pengembangan lokal dan menyediakan antarmuka web yang nyaman sehingga Anda dapat melihat pratinjau pesan email di peramban. Ketika menggunakan Sail, host default Mailpit adalah `mailpit` dan tersedia melalui port 1025:

```ini
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_ENCRYPTION=null
```

Ketika Sail dijalankan, silahkan akses antarmuka menggunakan: http://localhost:8025

<a name="sail-container-cli"></a>
## Contaienr CLI

Terkadang Anda mungkin ingin memulai sesi Bash di dalam kontainer aplikasi Anda. Anda dapat menggunakan perintah `shell` untuk menyambung ke kontainer aplikasi Anda, sehingga Anda dapat memeriksa berkas dan layanan yang terinstal, serta mengeksekusi perintah shell sembarang di dalam kontainer.

```shell
sail shell

sail root-shell
```

Untuk menjalankan sesi [Laravel Tinker](https://github.com/laravel/tinker), silahkan jalankan perintah `tinker`:

```shell
sail tinker
```

<a name="sail-php-versions"></a>
## Versi PHP

Sail saat ini mendukung untuk melayani aplikasi Anda melalui PHP 8.2, 8.1, PHP 8.0, atau PHP 7.4. Versi PHP default yang digunakan oleh Sail saat ini adalah PHP 8.2. Untuk mengubah versi PHP yang digunakan untuk melayani aplikasi Anda, Anda harus memperbarui definisi `build` dari kontainer `laravel.test` dalam berkas `docker-compose.yml` aplikasi Anda:

```yaml
# PHP 8.2
context: ./vendor/laravel/sail/runtimes/8.2

# PHP 8.1
context: ./vendor/laravel/sail/runtimes/8.1

# PHP 8.0
context: ./vendor/laravel/sail/runtimes/8.0

# PHP 7.4
context: ./vendor/laravel/sail/runtimes/7.4
```

Selain itu, Anda mungkin ingin memperbarui nama `image` untuk mencerminkan versi PHP yang digunakan oleh aplikasi Anda. Opsi ini juga ditentukan dalam berkas `docker-compose.yml` aplikasi Anda:

```yaml
image: sail-8.1/app
```

Setelah melakukan pembaruan di file `docker-compose.yml`, silahkan bangun ulang _Image Container_ anda:

```shell
sail build --no-cache

sail up
```

<a name="sail-node-versions"></a>
## Versi Node

Sail menginstal Node 18 secara default. Untuk mengubah versi Node yang terinstal saat membangun citra, Anda dapat memperbarui definisi `build.args` dari layanan `laravel.test` di berkas `docker-compose.yml` aplikasi Anda:

```yaml
build:
    args:
        WWWGROUP: '${WWWGROUP}'
        NODE_VERSION: '14'
```

Setelah melakukan pembaruan aplikasi pada file `docker-composer.yml`, silahkan bangun ulang aplikasi anda:

```shell
sail build --no-cache

sail up
```

<a name="sharing-your-site"></a>
## Membagikan Aplikasi Anda

Kadang-kadang Anda mungkin perlu membagikan situs Anda secara publik untuk melihat pratinjau situs Anda kepada kolega atau untuk menguji integrasi webhook dengan aplikasi Anda. Untuk membagikan situs Anda, Anda dapat menggunakan perintah `share`. Setelah menjalankan perintah ini, Anda akan mendapatkan sebuah URL acak `laravel-sail.site` yang dapat digunakan untuk mengakses aplikasi Anda:

```shell
sail share
```

Ketika membagikan situs Anda melalui perintah `share`, Anda harus mengonfigurasi proksi tepercaya aplikasi Anda di dalam middleware `TrustProxies`. Jika tidak, alat bantu pembuatan URL seperti `url` dan `route` tidak akan dapat menentukan host HTTP yang benar yang harus digunakan selama pembuatan URL

    /**
     * The trusted proxies for this application.
     *
     * @var array|string|null
     */
    protected $proxies = '*';

Jika Anda ingin memilih subdomain untuk situs yang dibagikan, Anda dapat memberikan opsi `subdomain` saat menjalankan perintah `share`:

```shell
sail share --subdomain=my-sail-site
```

> **Note**
> Perintah `share` didukung oleh [Expose](https://github.com/beyondcode/expose), sebuah layanan tunneling sumber terbuka dari [BeyondCode](https://beyondco.de).

<a name="debugging-with-xdebug"></a>
## Melakukan Debug Dengan Xdebug

Konfigurasi Docker Laravel Sail mencakup dukungan untuk [Xdebug](https://xdebug.org/), sebuah debugger yang populer dan kuat untuk PHP. Untuk mengaktifkan Xdebug, Anda perlu menambahkan beberapa variabel ke berkas `.env` aplikasi Anda ke [configure Xdebug](https://xdebug.org/docs/step_debug#mode). Untuk mengaktifkan Xdebug, Anda harus mengatur mode yang sesuai sebelum memulai Sail:

```ini
SAIL_XDEBUG_MODE=develop,debug,coverage
```

#### Konfigurasi IP Linux Host 

Secara internal, variabel lingkungan `XDEBUG_CONFIG` didefinisikan sebagai `client_host = host.docker.internal` sehingga Xdebug akan dikonfigurasi dengan benar untuk Mac dan Windows (WSL2). Jika mesin lokal Anda menjalankan Linux, Anda harus memastikan bahwa Anda menjalankan Docker Engine 17.06.0+ dan Compose 1.16.0+. Jika tidak, Anda perlu mendefinisikan variabel lingkungan ini secara manual seperti yang ditunjukkan di bawah ini.

Pertama, Anda harus menentukan alamat IP hos yang benar untuk ditambahkan ke variabel lingkungan dengan menjalankan perintah berikut. Biasanya, `<container-name>` adalah nama kontainer yang melayani aplikasi Anda dan biasanya diakhiri dengan `_laravel.test_1`:

```shell
docker inspect -f {{range.NetworkSettings.Networks}}{{.Gateway}}{{end}} <container-name>
```

Setelah Anda mendapatkan alamat IP host yang benar, Anda harus mendefinisikan variabel `SAIL_XDEBUG_CONFIG` di dalam file `.env` aplikasi Anda:

```ini
SAIL_XDEBUG_CONFIG="client_host=<host-ip-address>"
```

<a name="xdebug-cli-usage"></a>
### Xdebug Menggunakan CLI

Perintah `sail debug` dapat digunakan untuk memulai sesi debugging saat menjalankan perintah Artisan:

```shell
# Menjalankan perintah Artisan tanpa Xdebug
sail artisan migrate

# Menjalankan perintah Artisan dengan Xdebug
sail debug migrate
```

<a name="xdebug-browser-usage"></a>
### Xdebug Brwoser Usage

Untuk men-debug aplikasi Anda ketika berinteraksi dengan aplikasi melalui browser web, ikuti [petunjuk yang disediakan oleh Xdebug](https://xdebug.org/docs/step_debug#web-application) untuk memulai sesi Xdebug dari browser web.

Jika Anda menggunakan PhpStorm, silakan tinjau dokumentasi JetBrain mengenai [debugging tanpa konfigurasi](https://www.jetbrains.com/help/phpstorm/zero-configuration-debugging.html).

> **Peringatan**  
> Laravel Sail bergantung pada `artisan serve` untuk melayani aplikasi Anda. Perintah `artisan serve` hanya menerima variabel `XDEBUG_CONFIG` dan `XDEBUG_MODE` pada Laravel versi 8.53.0. Versi Laravel yang lebih lama (8.52.0 dan di bawahnya) tidak mendukung variabel-variabel ini dan tidak akan menerima koneksi debug.

<a name="sail-customization"></a>
## Cutomization

Karena Sail hanyalah Docker, Anda bebas untuk menyesuaikan hampir semua hal tentangnya. Untuk mempublikasikan Dockerfile Sail sendiri, Anda dapat menjalankan perintah `sail:publish`:

```shell
sail artisan sail:publish
```

Setelah menjalankan perintah ini, berkas Dockerfiles dan berkas konfigurasi lain yang digunakan oleh Laravel Sail akan ditempatkan di dalam direktori `docker` di direktori root aplikasi Anda. Setelah menyesuaikan instalasi Sail Anda, Anda mungkin ingin mengubah nama citra untuk kontainer aplikasi di dalam berkas `docker-compose.yml` aplikasi Anda. Setelah melakukannya, bangun ulang kontainer aplikasi Anda menggunakan perintah `build`. Menetapkan nama unik pada citra aplikasi sangat penting jika Anda menggunakan Sail untuk mengembangkan beberapa aplikasi Laravel pada satu mesin:

```shell
sail build --no-cache
```