# Laravel Valet

- [Pengenalan](#introduction)
- [Instalasi](#installation)
    - [Memperbarui Valet](#upgrade-valet)
- [Melayani Situs](#serving-sites)
    - [Perintah "Park"](#the-park-cpmmand)
    - [Perintah "Link"](#the-link-command)
    - [Mengamankan Situs Dengan TLS](#securing-sites)
    - [Melayani Situs Default](#serving-a-default-sites)
    - [Versi PHP Per-Situs](#per-site-php-versions)
- [Berbagi Situs](#sharing-sites)
    - [Berbagi Situs Melalui Ngrok](#sharing-sites-via-ngrok)
    - [Berbagi Situs Melalui Expose](#sharing-sites-via-expose)
    - [Berbagi Situs Pada Jaringan Lokal Anda](#sharing-sites-on-your-local-network)
- [Variabel Lingkungan Khusus Situs](#site-specific-environment-variables)
- [Memproksi Layanan](#proxying-services)
- [Driver Valet Kustom](#custom-valet-drivers)
    - [Driver Lokal](#local-drivers)
- [Perintah Valet Lainnya](#other-valet-commands)
- [Direktori & File Valet](#valet-directories-and-files)
    - [Akses Disk](#disk-access)

<a name="introduction"></a>
## Introduction

[Laravel Valet](https://github.com/laravel/valet) adalah lingkungan pengembangan untuk macOS minimalis. Laravel Valet mengonfigurasi Mac Anda untuk selalu menjalankan [Nginx](https://www.nginx.com/) di latar belakang saat mesin Anda dinyalakan. Kemudian, dengan menggunakan [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq), Valet memproksi semua permintaan pada domain `*.test` untuk mengarahkan ke situs yang terinstal di mesin lokal Anda.

Dengan kata lain, Valet adalah lingkungan pengembangan Laravel yang sangat cepat yang menggunakan RAM sekitar 7 MB. Valet bukanlah pengganti yang lengkap untuk [Sail](/docs/{{version}}/sail) atau [Homestead](/docs/{{version}}/homestead), tetapi menyediakan alternatif yang bagus jika Anda menginginkan dasar-dasar yang fleksibel, lebih suka kecepatan ekstrem, atau bekerja pada mesin dengan jumlah RAM yang terbatas.

Tidak menutup kemungkinan, Valet juga tidak terbatas untuk :

<style>
    #valet-support > ul {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        line-height: 1.9;
    }
</style>

<div id="valet-support" markdown="1">

- [Laravel](https://laravel.com)
- [Bedrock](https://roots.io/bedrock/)
- [CakePHP 3](https://cakephp.org)
- [ConcreteCMS](https://www.concretecms.com/)
- [Contao](https://contao.org/en/)
- [Craft](https://craftcms.com)
- [Drupal](https://www.drupal.org/)
- [ExpressionEngine](https://www.expressionengine.com/)
- [Jigsaw](https://jigsaw.tighten.co)
- [Joomla](https://www.joomla.org/)
- [Katana](https://github.com/themsaid/katana)
- [Kirby](https://getkirby.com/)
- [Magento](https://magento.com/)
- [OctoberCMS](https://octobercms.com/)
- [Sculpin](https://sculpin.io/)
- [Slim](https://www.slimframework.com)
- [Statamic](https://statamic.com)
- Static HTML
- [Symfony](https://symfony.com)
- [WordPress](https://wordpress.org)
- [Zend](https://framework.zend.com)

</div>

Namun, Anda dapat memperluas Valet dengan [driver khusus](#custom-valet-drivers) Anda sendiri.

<a name="Instalasi"></a>
## Instalasi

> **Peringatan**  
> Valet memerlukan macOs dan [Homebrew](http://brew.sh). Sebelum melakukan instalasi, sebaiknya Anda memastikan jika tidak ada program seperti Apache atau Nginx bertaut dengan koneksi lokal dengan Port 80.

Untuk memulai, pertama-tama Anda harus memastikan jika Homebrew sudah menggunakan versi terbaru dengan menggunakan perintah `update` :

```shell
brew update
```

Selanjutnya, Anda harus menggunakan Homebrew untuk menginstal PHP:

```shell
brew install php
```

Setelah instalasi php, Anda siap untuk menginstal [Composer Package Manager](getcomposer.org). Dengan tambahan, Anda harus memastikan jika `~/.composer/vendor/bin` direktori ada di dalam "PATH". Setelah _Composer_ telah terinstal, Anda harus instal Laravel Valet secara global 

```shell
composer global require laravel/valet
```

Setelah semua selesai, silahkan jalankan Valet dengan menggunakan `valet install`. Perintah ini akan mengkonfigurasi dan menginstal Valet dan DnsMasq. Selain itu, Valet akan berjalan setelah sistem menyala.

```shell
valet install
```

Setelah Valet terinstal, coba _ping_ `*.test` _domain_ pada terminal Anda menggunakan perintah seperti `ping foobar.test`. Jika Valet telah terinstal dengan sempurna, maka domain akan merespon dengan `127.0.0.1`

Valet akan secara otomatis memulai layanan yang diperlukan setiap kali mesin Anda dinyalakan.

<a name="php-version"></a>
#### Versi PHP

Valet memungkinkan Anda untuk mengganti versi PHP menggunakan perintah `valet use php@version`. Valet akan menginstal versi PHP secara spesifik menggunakan Homebrew jika belum terinstal.

```shell
valet use php@7.2

valet use php
```

Anda juga dapat membuat berkas `.valetphprc` di dalam root proyek Anda. File `.valetphprc` harus berisi versi PHP yang akan digunakan oleh situs:

```shell
php@7.2
```

Setelah semua file telah dibuat, Anda cuku menjalankan perintah `valet use`,  PHP yang diinginkan oleh situs dengan membaca berkas tersebut.

> **Peringatan**  
> Valet hanya akan menjalankan satu versi PHP, walaupun Anda memiliki beberapa versi PHP yang berbeda.

<a name="database"></a>
#### Database

Jika aplikasi Anda membutuhkan Database, lihat [DBngin](https://dbngin.com), yang menyediakan alat manajemen database lengkap dan gratis yang mencakup MySQL, PostgreSQL, dan Redis. Setelah DBngin terinstal, Anda dapat menyambungkan database Anda di `127.0.0.1` menggunakan nama pengguna `root` dan string kosong untuk kata sandi.

<a name="resetting-your-installation"></a>
#### Mengatur Ulang Instalasi Anda 

Jika instalasi Valet anda berjalan kurang baik, jalankan perintah `composer global require laravel/valet` yang diikuti dengan `valet install`, perintah ini akan mereset instalasi Anda dan dapat menyelesaikan berbagai masalah. Dalam kasus yang jarang terjadi, mungkin perlu untuk “boot ulang” Valet dengan mengeksekusi `valet uninstall --force` diikuti dengan `valet install`.

<a name="upgrading-valet"></a>
#### Memperbarui Valet

Anda dapat memperbarui instalasi Valet dengan menjalankan perintah `composer global require laravel/valet` di terminal Anda. Setelah melakukan pembaruan, baiknya untuk menjalankan perintah `valet install` sehingga Valet dapat melakukan pembaruan tambahan pada file konfigurasi Anda jika diperlukan.

<a name="serving-sites"></a>
## Serving Sites

Setelah Valet terinstal, Anda siap untuk mulai membuka aplikasi Laravel Anda. Valet menyediakan dua perintah untuk membantu Anda melayani aplikasi Anda: `park` dan `link`.

<a name="the-park-command"></a>
### Perintah `park`

Perintah `park` mendaftarkan sebuah direktori pada mesin Anda yang berisi aplikasi-aplikasi Anda. Setelah direktori “diparkir” dengan Valet, semua direktori di dalam direktori tersebut akan dapat diakses di browser web Anda di `http://<directory-name>.test`:

```shell
cd ~/Sites

valet park
```

Sekarang, aplikasi apa pun yang Anda buat di dalam direktori “parked” akan secara otomatis dibuka dengan menggunakan `http://<directory-name>.test`. Jadi, jika direktori parked Anda berisi direktori bernama “laravel”, aplikasi di dalam direktori tersebut akan dapat diakses di `http://laravel.test`. Selain itu, Valet secara otomatis mengizinkan Anda untuk mengakses situs menggunakan subdomain wildcard (`http://foo.laravel.test`).

<a name=“the-link-command”></a>
### Perintah `link`

Perintah `link` juga dapat digunakan untuk membuka aplikasi Laravel Anda. Perintah ini berguna jika Anda ingin menyajikan satu situs dalam sebuah direktori dan bukan seluruh direktori:

```shell
cd ~/Sites/laravel

valet link
```

Setelah aplikasi ditautkan ke Valet menggunakan perintah `link`, Anda dapat mengakses aplikasi tersebut menggunakan nama direktorinya. Jadi, situs yang ditautkan pada contoh di atas dapat diakses di `http://laravel.test`. Selain itu, Valet secara otomatis mengizinkan Anda untuk mengakses situs menggunakan sub-domain wildcard (`http://foo.laravel.test`).

Jika Anda ingin membuka aplikasi pada nama host yang berbeda, Anda dapat mengoper nama host ke perintah `link`. Sebagai contoh, Anda dapat menjalankan perintah berikut untuk membuat aplikasi tersedia di `http://application.test`:

```shell
cd ~/Sites/laravel

valet link application
```

Anda juga dapat menyajikan aplikasi pada subdomain menggunakan perintah `link`:

```shell
valet link api.application
```

Jalankan perintah `links` untuk menampilkan semua daftar direktori yang anda miliki

```shell
valet links
```

Perintah `unlink` dapat digunakan untuk menghancurkan tautan simbolis untuk sebuah situs:

```shell
cd ~/Sites/laravel

valet unlink
```

<a name="securing-sites"></a>
### Mengamankan situs dengan TLS

Secara default, Valet melayani situs melalui HTTP. Namun, jika Anda ingin melayani situs melalui TLS terenkripsi menggunakan HTTP/2, Anda dapat menggunakan perintah `secure`. Sebagai contoh, jika situs Anda dilayani oleh Valet pada domain `laravel.test`, Anda harus menjalankan perintah berikut ini untuk mengamankannya:

```shell
valet secure laravel
```

Untuk “tidak mengamankan” sebuah situs dan kembali melayani lalu lintas melalui HTTP biasa, gunakan perintah `unsecure`. Seperti perintah `secure`, perintah ini menerima nama host yang ingin Anda tidak amankan:

```shell
valet unsecure laravel
```

<a name="serving-a-default-site"></a>
### Menampilkan Situs _Default_

Terkadang, Anda mungkin ingin mengonfigurasi Valet untuk menampilkan situs "default" alih-alih menampilkan `404` saat mengunjungi domain `test` yang tidak dikenal. Untuk melakukan ini, Anda dapat menambahkan opsi `default` ke file konfigurasi `~/.config/valet/config.json`, yang berisi jalur ke situs yang akan berfungsi sebagai situs default Anda:

    "default": "/Users/Sally/Sites/example-site",

<a name="per-site-php-versions"></a>
### Versi PHP Per-Situs

Secara default, Valet menggunakan instalasi PHP global Anda untuk menyajikan situs-situs Anda. Namun, jika Anda perlu mendukung beberapa versi PHP di berbagai situs, Anda dapat menggunakan perintah `isolate` untuk menentukan versi PHP yang harus digunakan oleh situs tertentu. Perintah `isolate` mengonfigurasi Valet untuk menggunakan versi PHP yang ditentukan untuk situs yang terletak di direktori kerja Anda saat ini:

```shell
cd ~/Sites/example-site

valet isolate php@8.0
```

Jika nama situs anda tidak cocok dengan direktori yang mengandung di dalamnya, gunakan `--site` untuk memanggil langsung file anda

```shell
valet isolate php@8.0 --site="site-name"
```

Demi kenyamanan, Anda dapat menggunakan perintah `valet php`, `composer`, dan `which-php` untuk menjalankan panggilan secara langsung ke CLI atau alat PHP yang sesuai berdasarkan versi PHP yang dikonfigurasikan pada situs:

```shell
valet php
vlaet composer
valet which-php
```

Anda dapat menjalankan perintah `isolated` untuk menampilkan semua daftar situs yang berada didalam Versi PHP:

```shell
valet isolated
```

Untuk mengembalikan situs ke Versi PHP yang terinstal secara global, Anda dapat menggunakan perintah `unisolate` dari direktori _root_ situs.

```shell
valet unisolate
```

<a name="sharing-sites"></a>
## Situs Berbagi

Valet menyertakan perintah untuk membagikan situs lokal anda ke seluruh Dunia, gunanya untuk mempermudah pengetesan dari sisi _Mobile_ ataupun _Desktop_, juga bisa dibagikan dengan Rekan Kerja dan Mitra.

<a name="sharing-sites-via-ngrok"></a>
### Sharing Sites Via Ngrok

Untuk berbagi situs, arahkan ke direktori situs di terminal Anda dan jalankan perintah `share` dari Valet. URL yang dapat diakses publik akan dimasukkan ke dalam clipboard Anda dan siap untuk ditempelkan langsung ke dalam browser Anda atau dibagikan kepada tim Anda:

```shell
cd ~/Sites/laravel

clear share
```

Untuk berhenti membagikan situs Anda, Anda dapat menekan `Control + C`. Membagikan situs Anda menggunakan Ngrok mengharuskan Anda untuk [buat akun Ngrok](https://dashboard.ngrok.com/signup) dan [Siapkan Autentikasi](https://dashboard.ngrok.com/get-started/your-authtoken).

> **Note**  
> Anda dapat memberikan parameter Ngrok tambahan pada perintah share, seperti `valet share --region=eu`. Untuk informasi lebih lanjut, lihat [dokumentasi ngrok](https://ngrok.com/docs).

<a name="sharing-sites-via-expose"></a>
### Membagikan situs via Expose

Jika [Expose](expose.dev) Anda telah terinstal, Anda bisa membagikan situs anda dengan mengarahkan ke direktori situs di dalam terminal Anda dan menjalankan perintah `expose`. Lihat [Dokumentasi Expose](https://expose.dev/docs) untuk informasi mengenai parameter baris perintah tambahan yang didukungnya. Setelah membagikan situs, Expose akan menampilkan URL yang dapat dibagikan yang dapat Anda gunakan di perangkat lain atau di antara anggota tim:

```shell
cd ~/Sites/laravel

expose
```

Untuk berhenti membagikan, silahkan tekan `Control + C`

<a name="sharing-sites-on-your-local-network"></a>
### Membagikan situs Di jaringan Lokal

Valet membatasi lalu lintas yang masuk ke antarmuka internal `127.0.0.1` secara default sehingga mesin pengembangan Anda tidak terpapar risiko keamanan dari Internet.

Jika Anda ingin mengizinkan perangkat lain pada jaringan lokal Anda untuk mengakses situs Valet pada mesin Anda melalui alamat IP mesin Anda (misalnya: `192.168.1.10/application.test`), Anda perlu mengedit secara manual berkas konfigurasi Nginx yang sesuai untuk situs tersebut untuk menghapus pembatasan pada direktif `listen`. Anda harus menghapus awalan `127.0.0.1:` pada direktif `listen` untuk port 80 dan 443.

Jika Anda belum menjalankan `valet secure` pada proyek, Anda dapat membuka akses jaringan untuk semua situs non-HTTPS dengan menyunting berkas `/usr/local/etc/nginx/valet/valet.conf`. Namun, jika Anda melayani situs proyek melalui HTTPS (Anda telah menjalankan `valet secure` untuk situs tersebut) maka Anda harus menyunting berkas `~/.config/valet/Nginx/app-name.test`.

Setelah Anda memperbarui konfigurasi Nginx, jalankan perintah `valet restart` untuk menerapkan perubahan konfigurasi.

Beberapa aplikasi yang menggunakan kerangka kerja lain mungkin bergantung pada variabel lingkungan server tetapi tidak menyediakan cara bagi variabel tersebut untuk dikonfigurasi di dalam proyek Anda. Valet memungkinkan Anda untuk mengonfigurasi variabel lingkungan spesifik situs dengan menambahkan file `.valet-env.php` di dalam root proyek Anda. File ini akan mengembalikan sebuah larik pasangan variabel situs/lingkungan yang akan ditambahkan ke larik `$_SERVER` global untuk setiap situs yang ditentukan dalam larik tersebut:

    <?php

    return [
        // Set $_SERVER['key'] to "value" for the laravel.test site...
        'laravel' => [
            'key' => 'value',
        ],

        // Set $_SERVER['key'] to "value" for all sites...
        '*' => [
            'key' => 'value',
        ],
    ];

<a name="proxying-services"></a>
## Proxying Services

Terkadang, Anda mungkin ingin memproksi domain Valet ke layanan lain pada mesin lokal Anda. Sebagai contoh, Anda mungkin terkadang perlu menjalankan Valet sembari menjalankan situs terpisah di Docker; namun, Valet dan Docker tidak dapat mengikat porta 80 secara bersamaan.

Untuk mengatasi hal ini, Anda dapat menggunakan perintah `proxy` untuk menghasilkan proksi. Sebagai contoh, Anda dapat memproksi semua lalu lintas dari `http://elasticsearch.test` ke `http://127.0.0.1:9200`:

```shell
#Proxy Melalui HTTP...
valet proxy elasticsearch http://127.0.0.1:9200

#Proxy Melalui TLS + HTTP/2...
valet proxy elasticsearch http://127.0.0..1:9200 --secure
```

Anda dapat menghapus proxy menggunakan perinah `unproxy`

```shell
valet unproxy elasticsearch
```

Anda dapat menggunakan perintah `proxies` untuk membuat daftar semua konfigurasi situs yang diproksi:

```shell
valet proxies
```

<a name="custom-valet-drivers"></a>
## Custom Valet Drivers

Anda dapat menulis “driver” Valet Anda sendiri untuk melayani aplikasi PHP yang berjalan pada kerangka kerja atau CMS yang tidak didukung oleh Valet. Ketika Anda menginstal Valet, direktori `~/.config/valet/Drivers` dibuat yang berisi file `SampleValetDriver.php`. File ini berisi contoh implementasi driver untuk mendemonstrasikan cara menulis driver khusus. Menulis driver hanya mengharuskan Anda mengimplementasikan tiga metode: `serves`, `isStaticFile`, dan `frontControllerPath`.

Ketiga metode tersebut menerima nilai `$sitePath`, `$siteName`, dan `$uri` sebagai argumennya. `$sitePath` adalah jalur yang memenuhi syarat ke situs yang dilayani di mesin Anda, seperti `/Users/Lisa/Sites/my-project`. `$siteName` adalah bagian “host” / “nama situs” dari domain (`my-project`). `$uri` adalah URI permintaan yang masuk (`/foo/bar`).

Setelah Anda menyelesaikan driver Valet khusus Anda, letakkan di direktori `~/.config/valet/Drivers` dengan menggunakan konvensi penamaan `FrameworkValetDriver.php`. Sebagai contoh, jika Anda menulis driver valet khusus untuk WordPress, nama file Anda adalah `WordPressValetDriver.php`.

Sekarang, kita coba mengimplementasikan setiap metode dari kustom Valet yang akan digunakan

<a name="the-servers-method"></a>
#### Metode `servers`

Metode `services` harus mengembalikan `true` jika driver Anda harus menangani permintaan yang masuk. Jika tidak, metode tersebut akan mengembalikan nilai `false`. Jadi, di dalam metode ini, Anda harus mencoba menentukan apakah `$sitePath` yang diberikan berisi proyek dengan tipe yang Anda coba layani.

Sebagai contoh, bayangkan kita sedang menulis `WordPressValetDriver`. Metode `services` kita mungkin akan terlihat seperti ini:

    /**
     * Tentukan apakah driver melayani permintaan.
     *
     * @param string $sitePath
     * @param string $ namaSitus
     * @param string $uri
     * @return bool
     */
    public function serves($sitePath, $siteName, $uri)
    {
        return is_dir($sitePath.'/wp-admin');
    }

<a name=“the-isstaticfile-method”></a>
#### Metode `isStaticFile`

Metode `isStaticFile` harus menentukan apakah permintaan yang masuk adalah untuk berkas yang “statis”, seperti gambar atau stylesheet. Jika berkas tersebut statis, metode ini harus mengembalikan jalur yang memenuhi syarat ke berkas statis pada disk. Jika permintaan yang masuk bukan untuk berkas statis, metode harus mengembalikan `false`:

    /**
     * Tentukan apakah permintaan yang masuk adalah untuk berkas statis.
     *
     * @param string $sitePath
     * @param string $ namaSitus
     * @param string $uri
     * @return string $false
     */
    public function isStaticFile($sitePath, $siteName, $uri)
    {
        if (file_exists($staticFilePath = $sitePath.'/public/'.$uri)) {
            kembalikan $staticFilePath;
        }

        return false;
    }

> **Peringatan**  
> Metode `isStaticFile` hanya akan dipanggil jika metode `serves` mengembalikan nilai `true` untuk permintaan yang masuk dan URI permintaan bukan `/`.

<a name=“the-frontcontrollerpath-method”></a>
#### Metode `frontControllerPath`

Metode `frontControllerPath` harus mengembalikan jalur yang memenuhi syarat ke “pengendali depan” aplikasi Anda, yang biasanya berupa file “index.php” atau yang setara:

    /**
     * Dapatkan jalur yang sepenuhnya terselesaikan ke pengontrol depan aplikasi.
     *
     * @param string $sitePath
     * @param string $ namaSitus
     * @param string $uri
     * @return string
     */
    public function frontControllerPath($sitePath, $siteName, $uri)
    {
        return $sitePath.'/public/index.php';
    }

<a name="local-drivers"></a>
### Local Drivers

If you would like to define a custom Valet driver for a single application, create a `LocalValetDriver.php` file in the application's root directory. Your custom driver may extend the base `ValetDriver` class or extend an existing application specific driver such as the `LaravelValetDriver`:

    use Valet\Drivers\LaravelValetDriver;

    class LocalValetDriver extends LaravelValetDriver
    {
        /**
         * Determine if the driver serves the request.
         *
         * @param  string  $sitePath
         * @param  string  $siteName
         * @param  string  $uri
         * @return bool
         */
        public function serves($sitePath, $siteName, $uri)
        {
            return true;
        }

        /**
         * Get the fully resolved path to the application's front controller.
         *
         * @param  string  $sitePath
         * @param  string  $siteName
         * @param  string  $uri
         * @return string
         */
        public function frontControllerPath($sitePath, $siteName, $uri)
        {
            return $sitePath.'/public_html/index.php';
        }
    }

<a name=“perintah-perintah valet lainnya”></a>
## Perintah Pelayan Lainnya

Perintah | Deskripsi
------------- | -------------
`valet list` | Menampilkan daftar semua perintah Valet.
`valet forget` | Menjalankan perintah ini dari direktori “parked” untuk menghapusnya dari daftar direktori parked.
`valet log` | Melihat daftar log yang ditulis oleh layanan Valet.
`valet paths` | Melihat semua jalur “parked” Anda.
`valet restart` | Memulai ulang daemon Valet.
`valet start` | Memulai daemon Valet.
`valet stop` | Menghentikan daemon Valet.
`valet trust` | Menambahkan file sudoers untuk Brew dan Valet agar perintah Valet dapat dijalankan tanpa meminta kata sandi Anda.
`valet uninstall` | Copot pemasangan Valet: menampilkan instruksi untuk pencopotan pemasangan manual. Berikan opsi `--force` untuk menghapus semua sumber daya Valet secara agresif.

<a name=“valet-direktori-dan-file”></a>
## Direktori & File Valet

Anda mungkin menemukan informasi direktori dan file berikut ini berguna saat memecahkan masalah dengan lingkungan Valet Anda:

#### `~/.config/valet`

Berisi semua konfigurasi Valet. Anda mungkin ingin menyimpan cadangan direktori ini.

#### `~/.config/valet/dnsmasq.d/`

Direktori ini berisi konfigurasi DNSMasq.

#### `~/.config/valet/Drivers/`

Direktori ini berisi driver-driver Valet. Driver menentukan bagaimana sebuah framework / CMS tertentu dilayani.

#### `~/.config/valet/Extensions/`

Direktori ini berisi ekstensi/perintah khusus Valet.

#### `~/.config/valet/Nginx/`

Direktori ini berisi semua konfigurasi situs Nginx Valet. Berkas-berkas ini dibangun ulang saat menjalankan perintah `install` dan `secure`.

#### `~/.config/valet/Sites/`

Direktori ini berisi semua tautan simbolik untuk [proyek yang ditautkan] Anda (#perintah-tautan).

#### `~/.config/valet/config.json`

File ini adalah file konfigurasi utama Valet.

#### `~/.config/valet/valet.sock`

This file is the PHP-FPM socket used by Valet's Nginx installation. This will only exist if PHP is running properly.

#### `~/.config/valet/Log/fpm-php.www.log`

This file is the user log for PHP errors.

#### `~/.config/valet/Log/nginx-error.log`

This file is the user log for Nginx errors.

#### `/usr/local/var/log/php-fpm.log`

This file is the system log for PHP-FPM errors.

#### `/usr/local/var/log/nginx`

This directory contains the Nginx access and error logs.

#### `/usr/local/etc/php/X.X/conf.d`

This directory contains the `*.ini` files for various PHP configuration settings.

#### `/usr/local/etc/php/X.X/php-fpm.d/valet-fpm.conf`

This file is the PHP-FPM pool configuration file.

#### `~/.composer/vendor/laravel/valet/cli/stubs/secure.valet.conf`

This file is the default Nginx configuration used for building SSL certificates for your sites.

<a name=“disk-access”></a>
### Disk Access

Since macOS 10.14, [access to some files and directories is restricted by default](https://manuals.info.apple.com/MANUALS/1000/MA1902/en_US/apple-platform-security-guide.pdf). Pembatasan ini termasuk direktori Desktop, Dokumen, dan Unduhan. Selain itu, akses volume jaringan dan volume yang dapat dilepas juga dibatasi. Oleh karena itu, Valet merekomendasikan folder situs Anda berada di luar lokasi yang dilindungi ini.

Namun, jika Anda ingin melayani situs dari dalam salah satu lokasi tersebut, Anda harus memberikan “Akses Disk Penuh” pada Nginx. Jika tidak, Anda mungkin akan mengalami kesalahan server atau perilaku tak terduga lainnya dari Nginx, terutama saat melayani aset statis. Biasanya, macOS akan secara otomatis meminta Anda untuk memberikan akses penuh kepada Nginx ke lokasi-lokasi ini. Atau, Anda dapat melakukannya secara manual melalui `Preferensi Sistem` > `Keamanan & Privasi` > `Keprivasian` dan memilih `Akses Disk Penuh`. Selanjutnya, aktifkan entri `nginx` pada panel jendela utama.