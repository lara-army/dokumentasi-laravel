# Laravel Sail

- [Pengantar](#introduction)
- [Instalasi & Konfigurasi](#installation)
    - [Menginstal Sail ke Aplikasi yang sudah ada](#installing-sail-into-existing-applications)
    - [Melakukan Konfigurasi Alias Shell](#configuring-a-shell-alias)
- [Menjalankan & Menghentikan Sail](#starting-and-stopping-sail)
- [Menjalankan Perintah](#executing-sail-commands)
    - [Mengeksekusi Perintah PHP](#executing-php-commands)
    - [Mengeksekusi Perintah Composer](#executing-composer-commands)
    - [Mengeksekusi Perintah Artisan](#executing-artisan-commands)
    - [Mengeksekusi Perintah Node / NPM](#executing-node-npm-commands)
- [Berinteraksi Dengan Basis Data](#interacting-with-sail-basis datas)
    - [MySQL](#mysql)
    - [Redis](#redis)
    - [MeiliSearch](#meilisearch)
- [Penyimpanan Berkas](#file-storage)
- [Menjalankan Pengujian](#running-tests)
    - [Laravel Dusk](#laravel-dusk)
- [Melakukan Peninjauan Surel](#previewing-emails)
- [CLI Kontainer](#sail-container-cli)
- [Versi-versi PHP](#sail-php-versions)
- [Versi-versi Node](#sail-node-versions)
- [Membagikan Situs](#sharing-your-site)
- [Melakukan Debug Menggunakan Xdebug](#debugging-with-xdebug)
  - [Penggunaan Xdebug CLI](#xdebug-cli-usage)
  - [Penggunaan Xdebug Browser](#xdebug-browser-usage)
- [Kustomisasi](#sail-customization)

<a name="introduction"></a>
## Pengantar

[Laravel Sail](https://github.com/laravel/sail) adalah antarmuka baris perintah (CLI) yang ringan untuk berinteraksi dengan lingkungan pengembangan Docker bawaan Laravel. Sail menyediakan permulaan yang bagus untuk membangun aplikasi Laravel menggunakan PHP, MySQL, dan Redis yang tidak memerlukan pengalaman dengan Docker sebelumnya.

Sail adalah _file_ `docker-compose.yml` dan skrip `sail` yang tersimpan di dalam _root_ Proyek Anda, Skrip `sail` menyediakan CLI dengan pengopreasian yang mudah untuk berinteraksi dengan kontainer Docker yang dipanggil melalui _file_ `docker-compose.yml`.

Laravel Sail dapat beroperasi pada macOS, Linux, dan Windows (melalui [WSL2](https://docs.microsoft.com/en-us/windows/wsl/about)).

<a name="installation"></a>
## Instalasi & Konfigurasi

Secara otomatis, Laravel Sail terinstal pada semua aplikasi Laravel sehingga Anda bisa menggunakannya secara langsung. Untuk mempelajari bagaimana membuat Aplikasi Laravel baru, silakan mengunjungi [dokumentasi instalasi](/docs/{{version}}/installation) untuk sistem operasi Anda. Selama instalasi, Anda akan diberikan pertanyaan mengenai layanan dukungan Sail mana yang akan diinteraksikan.

<a name="installing-sail-into-existing-applications"></a>
### Menginstal Sail ke Aplikasi yang sudah ada

Jika Anda tertarik untuk menggunakan Sail dengan aplikasi Laravel yang sudah ada, Anda dapat secara mudah menginstal Sail menggunakan pengelola paket Composer. Tentu saja, langkah ini dilakukan dengan mengasumsikan bahwa lingkukan pengembangan Anda mengijinkan untuk menginstal paket-paket dengan Composer:

```shell
composer require laravel/sail --dev
```

Setelah Sail terinstal, Anda dapat menjalankan perintah Artisan `sail:install`. Perintah ini akan menambahkan _file_ `docker-compose.yml` di dalam _root_ aplikasi Anda.

```shell
php artisan sail:install
```

Akhirnya, Anda dapat menjalankan `sail` Anda. Untuk mempelajari lebih lanjut bagaimana cara menggunakan `Sail`, silakan membaca lanjutan dokumentasi ini:

```shell
./vendor/bin/sail up
```

<a name="adding-additional-services"></a>
#### Menambahkan Layanan Tambahan

Jika Anda ingin menambahkan layanan tambahan pada instalasi `sail`, silakan jalankan perintah Artisan `sail:add`:

```shell
php artisan sail:add
```

<a name="using-devcontainers"></a>
#### Menggunakan Devcontainer

Jika Anda ingin mengembangkan aplikasi dengan [Devcontainer](https://code.visualstudio.com/docs/remote/containers), Anda bisa menambahakan opsi `--devcontainer` pada perintah `sail:install`. Opsi `--devcontainer` akan menginstruksikan `sail:install` untuk menerbitkan _file_ `.devcontainer/devcontainer.json` pada direktori _root_ pada aplikasi Anda.

```shell
php artisan sail:install --devcontainer
```

<a name="configuring-a-shell-alias"></a>
### Melakukan Konfigurasi Alias Shell

Secara _default_, Perintah Sail dipanggil menggunakan skrip `vendor/bin/sail` yang secara _default_ terdapat pada semua Aplikasi Laravel yang baru:

```shell
./vendor/bin/sail up
```

Daripada Anda mengulangi `vendor/bin/sail` untuk mengeksekusi perintah-perintah Sail, silakan untuk melakukan konfigurasi `shell` yang memungkinkan Anda untuk menjalankan perintah-perintah Sail lebih mudah.

```shell
alias sail='[ -f sail ] && sh sail || sh vendor/bin/sail'
```

Untuk memastikan jika selalu dijalankan, Silahknan tambahkan konfigurasi ini didalam direktori `Home`, seperti `~./zshrc` atau `~./bashrc`, lalu jalankan ulang `shell` Anda.

Setelah `Shell` alias telah dikonfigurasi, silakan mengeksekusi perintah Sail Anda dengan mengetikkan `sail`. Sisa dokumentasi ini mencontohkan jika Anda sudah melakukan konfigurasi alias:

```shell
sail up
```

<a name="starting-and-stopping-sail"></a>
## Menjalankan & Menghentikan Sail

_File_ `docker-compose.yml` milik Laravel Sail mendefinisikan berbagai macam kontainer Docker yang bekerja bersama untuk membantu Anda membangun aplikasi Laravel. Setiap kontainer adalah sebuah entri dalam konfigurasi `services` di dalam _file_ `docker-compose.yml`. Kontainer `laravel.test` adalah kontainer aplikasi utama yang akan melayani aplikasi Anda.

Sebelum menjalankan Sail, baiknya tidak ada server web atau basis data yang sedang berjalan pada komputer lokal Anda. Untuk menjalankan semua kontainer Docker yang didefinisikan pada _file_ `docker-compose.yml`, Anda perlu mengeksekusi perintah `up`:

```shell
sail up
```

Untuk menjalankan kontainer Docker di latar belakang, silakan menjalankan Sail dalam mode "detached":

```shell
sail up -d
```

Setelah kontainer-kontainer milik aplikasi telah berjalan, Anda dapat mengakses proyek Anda melalui peramban web pada: http://localhost.

Untuk menghentikan semua kontainer, Anda dapat dengan mudah menekan Control + C untuk menghentikan eksekusi milik kontainer, jika kontainer-kontainer tersebut berjalan pada latar belakang, Anda dapat menggunakan perintah `stop`:

```shell
sail stop
```

<a name="executing-sail-commands"></a>
## Menjalankan Perintah

Ketika menggunakan Laravel Sail, aplikasi Anda dijalankan menggunakan kontainer Docker yang terisolasi/terpisah dari komputer lokal Anda. Bagaimanapun, Sail menyediakan cara yang mudah dan nyaman untuk menjalankan berbagai perintah terhadap aplikasi Anda seperti perintah-perintah PHP, Artisan, Composer, dan Node / NPM.

**Saat membaca dokumentasi Laravel, Anda akan sering menemukan referensi untuk Composer, Artisan dan Node / NPM yang tidak merujuk ke Sail.** Contoh-contoh tersebut mengasumsikan bahwa Anda menjalankan perintah-perintah ini secara langsung pada komputer lokal Anda. Jika Anda menggunakan Sail sebagai lingkungan pengembangan Laravel Anda, silakan jalankan perintah ini menggunakan Sail:

```shell
# Menjalankan perintah Artisan secara lokal...
php artisan queue:work

# Menjalankan perintah Artisan dengan Laravel Sail...
sail artisan queue:work
```

<a name="executing-php-commands"></a>
### Menjalankan Perintah PHP

Perintah-perintah PHP akan dijalankan menggunakan perintah `php`. Tentu saja, perintah-perintah ini akan dieksekusi menggunakan PHP dengan versi yang sesuai dengan Aplikasi Anda. Untuk mempelajari lebih lanjut tentang versi PHP yang tersedia pada Laravel Sail, silakan baca [dokumentasi versi PHP](#sail-php-versions):

```shell
sail php --version

sail php script.php
```

<a name="executing-composer-commands"></a>
### Menjalankan Perintah Composer

Perintah Composer akan dieksekusi menggunakan perintah `composer`. Kontainer milik Laravel Sail sudah termasuk instalasi Composer 2.x:

```nothing
sail composer require laravel/sanctum
```

<a name="installing-composer-dependencies-for-existing-projects"></a>
#### Instalasi Dependensi Composer Untuk Aplikasi Yang Sudah Ada

Jika Anda mengembangkan aplikasi bersama tim, Anda mungkin bukan orang yang menginisiasi (penginstalan awal) aplikasi Laravel tersebut. Oleh karena itu, tidak ada dependensi Composer milik aplikasi, termasuk Sail, yang akan terinstal setelah Anda mengkloning repositori aplikasi ke komputer lokal Anda.

Anda dapat menginstal dependensi milik aplikasi dengan menavigasi ke direktori aplikasi dan menjalankan perintah di bawah. Perintah ini menggunakan kontainer Docker kecil yang berisi PHP dan Composer untuk menginstal dependensi yang diperlukan oleh aplikasi:

```shell
docker run --rm \
    -u "$(id -u):$(id -g)" \
    -v "$(pwd):/var/www/html" \
    -w /var/www/html \
    laravelsail/php82-composer:latest \
    composer install --ignore-platform-reqs
```

Saat menggunakan _image_ `laravelsail/phpXX-composer`, Anda perlu menggunakan versi PHP yang sama yang digunakan pada aplikasi Anda (`74`, `80`, `81`, atau `82`).

<a name="executing-artisan-commands"></a>
### Mengeksekusi Perintah Artisan

Perintah Laravel Artisan dapat dijalankan dengan perintah `artisan`:

```shell
sail artisan queue:work
```

<a name="executing-node-npm-commands"></a>
### Menjalankan Perintah Node / NPM

Perintah-perintah Node dapat dijalankan menggunakan perintah `node` sedangkan perintah NPM dapat dijalankan dengan perintah `npm`:

```shell
sail node --version

sail npm run dev
```

Jika Anda ingin, Anda juga dapat menggunakan Yarn selain NPM:

```shell
sail yarn
```

<a name="interacting-with-sail-basis datas"></a>
## Berinteraksi dengan Basis Data

<a name="mysql"></a>
### MySQL

Seperti yang Anda ketahui, _file_ `docker-compose.yml` aplikasi Anda berisi entri untuk kontainer MySQL. Kontainer ini menggunakan [volume Docker](https://docs.docker.com/storage/volumes/) sehingga data yang tersimpan di dalam basis data Anda akan tetap ada (persisted), bahkan ketika kontainer-kontainer Anda dihentikan dan dimulai ulang.

Sebagai tambahan, saat pertama kali kontainer MySQL dijalankan, kontainer ini akan membuat dua basis data untuk Anda. Basis data yang pertama diberi nama yang sama dengan nilai variabel lingkungan (_environment variable_) `DB_DATABASE` dan digunakan untuk pengembangan lokal Aplikasi. Basis data kedua adalah basis data pengujian khusus yang diberi nama `testing` dan akan memastikan bahwa pengujian-pengujian yang akan dijalankan tidak mengganggu data pengembangan Anda (basis data yang pertama).

Setelah kontainer-kontainer Anda dijalankan, Anda dapat menyambungkan ke _instance_ MySQL yang terdapat di dalam aplikasi Anda dengan mengatur variabel lingkungan `DB_HOST` di dalam file `.env` aplikasi Anda ke `mysql`.

Untuk menghubungkan mesin lokal Anda dengan basis data MySQL milik aplikasi, Anda dapat menggunakan aplikasi manajemen basis data grafis seperti [TablePlus](https://tableplus.com). Secara _default_, basis data MySQL dapat diakses psfs `localhost` dengan _port_ 3306 dan kredensial aksesnya sesuai dengan nilai variabel lingkungan `DB_USERNAME` dan `DB_PASSWORD`. Atau, Anda juga dapat masuk/terhubung sebagai pengguna `root`, yang juga menggunakan nilai variabel lingkungan `DB_PASSWORD` sebagai kata sandinya.

<a name="redis"></a>
### Redis

_File_ `docker-compose.yml` milik aplikasi Anda juga berisi entri untuk kontainer [Redis](https://redis.io). Kontainer ini menggunakan [volume Docker](https://docs.docker.com/storage/volumes/) sehingga data yang tersimpan di dalam data Redis milik Anda akan tetap ada (_persisted_), bahkan ketika menghentikan dan memulai ulang kontainer Anda. Setelah Anda menjalankan kontainer, Anda dapat terhubung ke _instance_ Redis di dalam aplikasi Anda dengan mengatur variabel lingkungan `REDIS_HOST` pada _file_ `.env` aplikasi bernilai `redis`.

Untuk menghubungkan mesin lokal Anda dengan basis data Redis milik aplikasi, Anda dapat menggunakan aplikasi manajemen basis data grafis seperti [TablePlus](https://tableplus.com). Secara _default_, basis data Redis dapat diakses pada _port_ 6379 `localhost`.

<a name="meilisearch"></a>
### MeiliSearch

Jika Anda memilih untuk menginstal layanan [MeiliSearch](https://www.meilisearch.com) ketika menginstal Sail, _file_ `docker-compose.yml` aplikasi Anda akan berisi entri untuk mesin pencari yang kuat ini yang [kompatibel](https://github.com/meilisearch/meilisearch-laravel-scout) dengan [Laravel Scout](/docs/{{version}}/scout). Setelah menjalankan kontainer Anda, Anda dapat terhubung ke _instance_ MeiliSearch di dalam aplikasi Anda dengan mengatur variabel lingkungan `MEILISEARCH_HOST` ke `http://meilisearch:7700`.

Dari mesin lokal Anda, Anda dapat mengakses panel administrasi berbasis web MeiliSearch dengan menavigasi ke `http://localhost:7700` pada peramban web Anda.

<a name="file-storage"></a>
## Penyimpanan Berkas

Jika Anda berencana untuk menggunakan Amazon S3 untuk menyimpan _file_-_file_ aplikasi pada lingkungan produksi, Anda dapat menginstal layanan [MinIO] (https://min.io) saat menginstal Sail. MinIO menyediakan API yang kompatibel dengan S3 yang dapat Anda gunakan untuk pengembangan lokal menggunakan _driver_ penyimpanan _file_ `s3` milik Laravel harus tanpa membuat ember penyimpanan (_storage bucket_) “uji coba” pada lingkungan S3 produksi milik Anda. Jika Anda memilih untuk menginstal MinIO saat menginstal Sail, sebuah konfigurasi MinIO akan ditambahkan ke dalam _file_ `docker-compose.yml` milik aplikasi Anda.

Secara _default_, berkas konfigurasi `filesystems` milik aplikasi Anda telah berisi sebuah konfigurasi _disk_ untuk (_disk_) `s3`. Selain dapat digunakan untuk berinteraksi dengan Amazon S3, Anda juga dapat menggunakannya untuk berinteraksi dengan layanan penyimpanan berkas yang kompatibel dengan S3 seperti MinIO hanya dengan memodifikasi variabel lingkungan terkait yang mengontrol konfigurasinya. Misalnya, saat menggunakan MinIO, konfigurasi variabel lingkungan sistem berkas Anda harus ditetapkan sebagai berikut:

```ini
FILESYSTEM_DISK=s3
AWS_ACCESS_KEY_ID=sail
AWS_SECRET_ACCESS_KEY=password
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=local
AWS_ENDPOINT=http://minio:9000
AWS_USE_PATH_STYLE_ENDPOINT=true
```

Agar integrasi Flysystem Laravel dapat menghasilkan URL yang tepat ketika menggunakan MinIO, Anda harus mendefinisikan variabel lingkungan `AWS_URL` agar sesuai dengan URL lokal milik aplikasi Anda dan menyertakan nama _bucket_ dalam _path_ URL:

```ini
AWS_URL=http://localhost:9000/local
```

Anda dapat membuat _bucket_ melalui konsol MinIO, yang tersedia pada `http://localhost:8900`. Nama pengguna _default_ untuk konsol MinIO adalah `sail`, sedangkan kata sandi _default_ adalah `password`.

> **Warning**
> Membuat URL penyimpanan sementara melalui metode `temporaryUrl` tidak dapat dilakukan (_not supported_) saat menggunakan MinIO.

<a name="running-test"></a>
## Menjalankan Pengujian

Laravel menyediakan dukungan pengujian yang luar biasa, dan Anda dapat menggunakan perintah `test` milik Sail untuk menjalankan [pengujian fitur dan unit](/docs/{{versi}}/testing) milik aplikasi Anda. Semua opsi CLI yang diterima oleh PHPUnit juga dapat diteruskan ke perintah `test`:

```shell
sail test

sail test --group orders
```

Perintah `test` pada Sail setara dengan menjalankan perintah Artisan `test`:

```shell
sail artisan test
```

Secara _default_, Sail akan membuat basis data `testing` khusus sehingga pengujian Anda tidak mengganggu kondisi basis data Anda saat ini. Dalam instalasi Laravel yang _default_, Sail juga akan mengonfigurasi file `phpunit.xml` Anda untuk menggunakan basis data ini ketika mengeksekusi pengujian-pengujian milik Anda:

```xml
<env name="DB_DATABASE" value="testing"/>
```

<a name="laravel-dusk"></a>
### Laravel Dusk

[Laravel Dusk](/docs/{{version}}/dusk) menyediakan otomatisasi peramban dan API pengujian yang ekspresif dan mudah digunakan. Berkat Sail, Anda dapat menjalankan pengujian ini tanpa perlu menginstal Selenium atau alat lain pada komputer lokal Anda. Untuk memulai, _uncomment_ layanan Selenium pada _file_ `docker-compose.yml` milik aplikasi Anda:

```yaml
selenium:
    image: 'selenium/standalone-chrome'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

Selanjutnya, pastikan bahwa layanan `laravel.test` dalam _file_ `docker-compose.yml` milik aplikasi Anda memiliki entri `depends_on` untuk `selenium`:

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

Jika mesin lokal Anda berisi chip Apple Silicon, layanan `selenium` Anda harus menggunakan _image_ `seleniarm/standalone-chromium`

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

_File_ `docker-compose.yml` bawaan Laravel Sail berisi entri untuk layanan [Mailpit](https://github.com/axllent/mailpit). Mailpit mencegah email yang dikirim oleh aplikasi Anda selama pengembangan lokal dan menyediakan antarmuka web yang nyaman sehingga Anda dapat melihat pratinjau pesan email pada peramban. Ketika menggunakan Sail, _host default_ untuk Mailpit adalah `mailpit` dan tersedia melalui _port_ 1025:

```ini
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_ENCRYPTION=null
```

Ketika Sail dijalankan, Anda dapat mengakses antarmuka web milik Mailpit pada: http://localhost:8025

<a name="sail-container-cli"></a>
## Kontainer CLI

Terkadang Anda mungkin ingin memulai sesi Bash di dalam kontainer aplikasi Anda. Anda dapat menggunakan perintah `shell` untuk terhubung ke kontainer aplikasi Anda, sehingga Anda dapat memeriksa berkas dan layanan yang terinstal, serta mengeksekusi perintah _shell_ apapun di dalam kontainer.

```shell
sail shell

sail root-shell
```

Untuk menjalankan sesi [Laravel Tinker](https://github.com/laravel/tinker) yang baru, Anda dangan menggunakan perintah `tinker`:

```shell
sail tinker
```

<a name="sail-php-versions"></a>
## Versi PHP

Sail saat ini mendukung untuk melayani aplikasi Anda melalui PHP 8.2, 8.1, PHP 8.0, atau PHP 7.4. Versi PHP _default_ yang digunakan oleh Sail saat ini adalah PHP 8.2. Untuk mengubah versi PHP yang digunakan untuk melayani aplikasi Anda, Anda harus memperbarui definisi `build` dari kontainer `laravel.test` pada file `docker-compose.yml` milik aplikasi Anda:

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

Selain itu, Anda mungkin ingin memperbarui nama `image` untuk mencerminkan versi PHP yang digunakan oleh aplikasi Anda. Opsi ini juga didefinisikan pada _file_ `docker-compose.yml` milik aplikasi Anda:

```yaml
image: sail-8.1/app
```

Setelah melakukan pembaruan pada _file_ `docker-compose.yml`, Anda perlu membangun ulang _Image_ kontainer milik Anda:

```shell
sail build --no-cache

sail up
```

<a name="sail-node-versions"></a>
## Versi Node

Sail menginstal Node 18 secara _default_. Untuk mengubah versi Node yang terinstal saat membangun _image_, Anda dapat memperbarui definisi `build.args` milik layanan `laravel.test` pada _file_ `docker-compose.yml` milik aplikasi Anda:

```yaml
build:
    args:
        WWWGROUP: '${WWWGROUP}'
        NODE_VERSION: '14'
```

Setelah melakukan pembaruan pada _file_ `docker-composer.yml` milik aplikasi, Anda perlu membangun ulang aplikasi Anda:

```shell
sail build --no-cache

sail up
```

<a name="sharing-your-site"></a>
## Membagikan Aplikasi Anda

terkadang Anda mungkin perlu membagikan situs Anda secara publik untuk melihat pratinjau situs Anda kepada kolega atau untuk menguji integrasi antara _webhook_ dengan aplikasi Anda. Untuk membagikan situs, Anda dapat menggunakan perintah `share`. Setelah menjalankan perintah ini, Anda akan mendapatkan sebuah URL acak `laravel-sail.site` yang dapat digunakan untuk mengakses aplikasi Anda:

```shell
sail share
```

Ketika membagikan situs Anda melalui perintah `share`, Anda harus mengonfigurasi proksi tepercaya (_trusted proxy_) milik aplikasi Anda di dalam _middleware_ `TrustProxies`. Jika tidak, alat bantu pembuatan URL seperti `url` dan `route` tidak akan dapat menentukan _host_ HTTP yang diperlukan selama pembuatan URL:

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
> Perintah `share` ditenagai oleh [Expose](https://github.com/beyondcode/expose), sebuah layanan _tunneling_ sumber terbuka dari [BeyondCode](https://beyondco.de).

<a name="debugging-with-xdebug"></a>
## Melakukan Debug dengan Xdebug

Konfigurasi Docker milik Laravel Sail mencakup dukungan untuk [Xdebug](https://xdebug.org/), sebuah _debugger_ yang populer dan _powerful_ untuk PHP. Untuk mengaktifkan Xdebug, Anda perlu menambahkan beberapa variabel pada _file_ `.env` milik aplikasi Anda ke [configure Xdebug](https://xdebug.org/docs/step_debug#mode). Untuk mengaktifkan Xdebug, Anda harus mengatur mode yang sesuai sebelum memulai Sail:

```ini
SAIL_XDEBUG_MODE=develop,debug,coverage
```

#### Konfigurasi IP Linux Host

Secara internal, variabel lingkungan `XDEBUG_CONFIG` didefinisikan menjadi `client_host = host.docker.internal` sehingga Xdebug akan terkonfigurasi secara sempurna untuk Mac dan Windows (WSL2). Jika mesin lokal Anda menjalankan Linux, Anda harus memastikan bahwa Anda menjalankan Docker Engine 17.06.0+ dan Compose 1.16.0+. Jika tidak, Anda perlu mendefinisikan variabel lingkungan ini secara manual seperti yang ditunjukkan di bawah ini.

Pertama, Anda harus menentukan alamat IP _host_ yang sesuai untuk ditambahkan ke variabel lingkungan dengan menjalankan perintah berikut. Biasanya, `<container-name>` adalah nama kontainer yang melayani aplikasi Anda dan biasanya diakhiri dengan `_laravel.test_1`:

```shell
docker inspect -f {{range.NetworkSettings.Networks}}{{.Gateway}}{{end}} <container-name>
```

Setelah Anda mendapatkan alamat IP _host_ yang benar, Anda harus mendefinisikan variabel `SAIL_XDEBUG_CONFIG` pada _file_ `.env` milik aplikasi Anda:

```ini
SAIL_XDEBUG_CONFIG="client_host=<host-ip-address>"
```

<a name="xdebug-cli-usage"></a>
### Penggunaan Xdebug CLI

Perintah `sail debug` dapat digunakan untuk memulai sebuah sesi _debugging_ ketika menjalankan perintah Artisan:

```shell
# Menjalankan perintah Artisan tanpa Xdebug...
sail artisan migrate

# Menjalankan perintah Artisan dengan Xdebug...
sail debug migrate
```

<a name="xdebug-browser-usage"></a>
### Penggunaan Xdebug Browser

Untuk melakukan _debug_ terhadap aplikasi Anda ketika berinteraksi dengan aplikasi melalui peramban web, ikuti [petunjuk yang disediakan oleh Xdebug](https://xdebug.org/docs/step_debug#web-application) untuk memulai sebuah sesi Xdebug melalui peramban web.

Jika Anda menggunakan PhpStorm, silakan tinjau dokumentasi milik JetBrain mengenai [debugging tanpa konfigurasi](https://www.jetbrains.com/help/phpstorm/zero-configuration-debugging.html).

> **Warning**  
> Laravel Sail bergantung pada `artisan serve` untuk menyajikan aplikasi Anda. Perintah `artisan serve` hanya menerima variabel `XDEBUG_CONFIG` dan `XDEBUG_MODE` pada Laravel versi 8.53.0. Versi Laravel yang lebih lama (8.52.0 ke bawah) tidak mendukung variabel-variabel ini dan tidak akan menerima koneksi _debug_.

<a name="sail-customization"></a>
## Kustomisasi

Karena Sail hanyalah Docker, Anda bebas untuk menyesuaikan hampir semua hal tentangnya. Untuk mempublikasikan Dockerfile milik Sail sendiri, Anda dapat menjalankan perintah `sail:publish`:

```shell
sail artisan sail:publish
```

Setelah menjalankan perintah ini, _file-file_ Dockerfile dan konfigurasi lain yang digunakan oleh Laravel Sail akan ditempatkan di dalam direktori `docker` pada _root_ milik aplikasi Anda. Setelah melakukan kustomisasi instalasi Sail, Anda mungkin ingin mengubah nama _image_ untuk kontainer aplikasi pada _file_ `docker-compose.yml` milik aplikasi Anda. Setelah melakukannya, bangun ulang kontainer-kontainer milik aplikasi Anda menggunakan perintah `build`. Menetapkan yang nama unik pada _image_ aplikasi sangat penting jika Anda menggunakan Sail untuk mengembangkan beberapa aplikasi Laravel pada satu mesin:

```shell
sail build --no-cache
```
