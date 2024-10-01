# Laravel Valet

- [Pengenalan](#introduction)
- [Instalasi](#installation)
    - [Memperbarui Valet](#upgrade-valet)
- [Menyediakan Situs](#serving-sites)
    - [Perintah "Park"](#the-park-cpmmand)
    - [Perintah "Link"](#the-link-command)
    - [Mengamankan Situs Dengan TLS](#securing-sites)
    - [Melayani Situs Default](#serving-a-default-sites)
    - [Versi PHP Per-Situs](#per-site-php-versions)
- [Membagikan Situs](#sharing-sites)
    - [Berbagi Situs Melalui Ngrok](#sharing-sites-via-ngrok)
    - [Berbagi Situs Melalui Expose](#sharing-sites-via-expose)
    - [Berbagi Situs Pada Jaringan Lokal Anda](#sharing-sites-on-your-local-network)
- [Variabel Lingkungan Khusus Situs](#site-specific-environment-variables)
- [Melakukan Proksi Layanan](#proxying-services)
- [Driver Valet Kustom](#custom-valet-drivers)
    - [Driver Lokal](#local-drivers)
- [Perintah Valet Lainnya](#other-valet-commands)
- [Direktori & _File_ Valet](#valet-directories-and-files)
    - [Akses _Disk_](#disk-access)

<a name="introduction"></a>
## Pengenalan

[Laravel Valet](https://github.com/laravel/valet) adalah lingkungan pengembangan minimalis untuk macOS. Laravel Valet mengonfigurasi Mac Anda untuk selalu menjalankan [Nginx](https://www.nginx.com/) pada latar belakang saat mesin Anda dinyalakan. Kemudian, dengan menggunakan [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq), Valet memproksi semua permintaan pada domain `*.test` untuk mengarahkan ke situs-situs yang terinstal di mesin lokal Anda.

Dengan kata lain, Valet adalah lingkungan pengembangan Laravel yang sangat cepat yang menggunakan RAM sekitar 7 MB. Valet bukanlah pengganti untuk [Sail](/docs/{{version}}/sail) atau [Homestead](/docs/{{version}}/homestead), tetapi menyediakan alternatif yang bagus jika Anda menginginkan dasar-dasar yang fleksibel, lebih suka kecepatan ekstrem, atau bekerja pada mesin dengan jumlah RAM yang terbatas.

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

Selain itu, Anda dapat memperluas dukungan Valet dengan menggunakan [driver khusus](#custom-valet-drivers) Anda sendiri.

<a name="Instalasi"></a>
## Instalasi

> **Peringatan**  
> Valet memerlukan macOs dan [Homebrew](http://brew.sh). Sebelum melakukan instalasi, sebaiknya Anda memastikan jika tidak ada program seperti Apache atau Nginx yang sedang bertaut pada koneksi lokal dengan _port_ 80.

Untuk memulai, pertama-tama Anda harus memastikan jika Homebrew sudah menggunakan versi terbaru dengan menggunakan perintah `update` :

```shell
brew update
```

Selanjutnya, Anda harus menggunakan Homebrew untuk menginstal PHP:

```shell
brew install php
```

Setelah menginstal PHP, Anda siap untuk menginstal [Composer Package Manager](getcomposer.org). Perlu diingat, Anda harus memastikan jika direktori `~/.composer/vendor/bin` ada di dalam "PATH". Setelah Composer terinstal, Anda dapat menginstal Laravel Valet sebagai paket Composer yang bersifat global:

```shell
composer global require laravel/valet
```

Setelah semua selesai, silahkan jalankan Valet dengan menggunakan `valet install`. Perintah ini akan mengonfigurasi dan menginstal Valet dan DnsMasq. _Daemon-daemon_ yang diperlukan oleh Valet akan dikonfigurasi untuk berjalan ketika sistem menyala.

```shell
valet install
```

Setelah Valet terinstal, cobalah melakukan _ping_ ke domain `*.test` yang mana saja pada terminal Anda menggunakan perintah seperti `ping foobar.test`. Jika Valet telah terinstal dengan sempurna, maka domain akan merespon dengan `127.0.0.1`.

Valet akan secara otomatis memulai layanan yang diperlukan setiap kali mesin Anda dinyalakan.

<a name="php-version"></a>
#### Versi PHP

Valet memungkinkan Anda untuk mengganti versi PHP menggunakan perintah `valet use php@version`. Valet akan menginstal versi PHP secara spesifik menggunakan Homebrew jika belum terinstal.

```shell
valet use php@7.2

valet use php
```

Anda juga dapat membuat _file_ `.valetphprc` di dalam direktori akar (_root_) proyek Anda. _File_ `.valetphprc` harus berisi versi PHP yang ingin digunakan oleh situs:

```shell
php@7.2
```

Setelah semua _file_ telah dibuat, Anda cukup menjalankan perintah `valet use`. Perintah tersebut akan menentukan versi PHP yang diinginkan oleh situs dengan membaca berkas tersebut.

> **Peringatan**  
> Valet hanya akan menjalankan satu versi PHP pada satu waktu, walaupun Anda memiliki beberapa versi PHP yang berbeda.

<a name="database"></a>
#### Basis Data

Jika aplikasi Anda membutuhkan basis data, Silahkan lihat [DBngin](https://dbngin.com), yang menyediakan alat manajemen basis data yang lengkap dan gratis yang mencakup MySQL, PostgreSQL, dan Redis. Setelah DBngin terinstal, Anda dapat menyambungkan basis data Anda pada `127.0.0.1` menggunakan nama pengguna `root` dan _string_ kosong untuk kata sandi.

<a name="resetting-your-installation"></a>
#### Mengatur Ulang Instalasi Anda 

Jika Anda mengalami masalah untuk menjalankan Valet yang sesuai, jalankan perintah `composer global require laravel/valet` yang diikuti dengan `valet install`, perintah ini akan mereset instalasi Anda dan dapat menyelesaikan berbagai masalah. Dalam kasus yang jarang terjadi, Anda mungkin perlu untuk melakukan "_hard reset_" Valet dengan mengeksekusi `valet uninstall --force` diikuti dengan `valet install`.

<a name="upgrading-valet"></a>
#### Memperbarui Valet

Anda dapat memperbarui instalasi Valet dengan menjalankan perintah `composer global require laravel/valet` pada terminal Anda. Setelah melakukan pembaruan, baiknya Anda menjalankan perintah `valet install` sehingga Valet dapat melakukan pembaruan tambahan pada _file_ konfigurasi Anda jika diperlukan.

<a name="serving-sites"></a>
## Menyediakan Situs

Setelah Valet terinstal, Anda siap untuk mulai membuka aplikasi Laravel Anda. Valet menyediakan dua perintah untuk membantu Anda menyediakan aplikasi-aplikasi Anda: `park` dan `link`.

<a name="the-park-command"></a>
### Perintah `park`

Perintah `park` akan mendaftarkan sebuah direktori pada mesin Anda yang berisi aplikasi-aplikasi Anda. Setelah direktori tersebut "diparkir" dengan Valet, semua direktori di dalamnya akan dapat diakses di _browser_ web Anda pada `http://<nama-direktori>.test`:

```shell
cd ~/kumpulan-proyek

valet park
```

Sekarang, aplikasi apa pun yang Anda buat di dalam direktori yang telah "terparkir" tersebut akan secara otomatis tersedia (dapat diakses) dengan menggunakan konvensi `http://<nama-direktori>.test`. Jadi, jika direktori "terparkir" Anda berisi direktori bernama "laravel", aplikasi di dalam direktori tersebut akan dapat diakses pada `http://laravel.test`. Selain itu, Valet secara otomatis mengizinkan Anda untuk mengakses situs menggunakan subdomain-subdomain _wildcard_ (`http://foo.laravel.test`).

<a name="the-link-command"></a>
### Perintah `link`

Perintah `link` juga dapat digunakan untuk menyajikan aplikasi Laravel Anda. Perintah ini berguna jika Anda ingin menyajikan satu situs dalam sebuah direktori dan bukan seluruh direktori:

```shell
cd ~/kumpulan-proyek/laravel

valet link
```

Setelah aplikasi berhasil ditautkan ke Valet menggunakan perintah `link`, Anda dapat mengakses aplikasi tersebut menggunakan nama direktorinya. Jadi, situs yang ditautkan pada contoh di atas dapat diakses pada `http://laravel.test`. Selain itu, Valet secara otomatis mengizinkan Anda untuk mengakses situs menggunakan subdomain-subdomain _wildcard_ (`http://foo.laravel.test`).

Jika Anda ingin menyediakan aplikasi dengan nama _host_ yang berbeda, Anda dapat mengoper nama _host_ ke perintah `link`. Sebagai contoh, Anda dapat menjalankan perintah berikut untuk membuat aplikasi tersedia di `http://aplikasi-gue.test`:

```shell
cd ~/kumpulan-proyek/laravel

valet link aplikasi-gue
```

Anda juga dapat menyajikan aplikasi pada/menjadi subdomain menggunakan perintah `link`:

```shell
valet link api.aplikasi-gue
```

Anda juga dapat menjalankan perintah `links` untuk menampilkan semua daftar direktori yang anda miliki:

```shell
valet links
```

Perintah `unlink` dapat digunakan untuk menghancurkan tautan simbolis (_symbolic link_) untuk sebuah situs:

```shell
cd ~/kumpulan-proyek/laravel

valet unlink
```

<a name="securing-sites"></a>
### Mengamankan Situs dengan TLS

Secara _default_, Valet menyajikan situs melalui (protokol) HTTP. Namun, jika Anda ingin menyajikan situs melalui TLS yang terenkripsi menggunakan HTTP/2, Anda dapat menggunakan perintah `secure`. Sebagai contoh, jika situs Anda disajkian oleh Valet pada domain `laravel.test`, Anda harus menjalankan perintah berikut ini untuk mengamankannya:

```shell
valet secure laravel
```

Untuk "tidak mengamankan" sebuah situs dan kembali melayani lalu lintas melalui HTTP biasa, gunakan perintah `unsecure`. Seperti perintah `secure`, perintah ini menerima nama host yang tidak ingin diamankan:

```shell
valet unsecure laravel
```

<a name="serving-a-default-site"></a>
### Menampilkan Situs _Default_

Terkadang, Anda mungkin ingin mengonfigurasi Valet untuk menampilkan situs "_default_" alih-alih menampilkan `404` saat mengunjungi domain `test` yang tidak dikenal. Untuk melakukan ini, Anda dapat menambahkan opsi `default` ke _file_ konfigurasi `~/.config/valet/config.json`, yang berisi _path_ ke situs yang akan berfungsi sebagai situs _default_ Anda:

    "default": "/Users/Sally/Sites/example-site",

<a name="per-site-php-versions"></a>
### Versi PHP Per-Situs

Secara _default_, Valet menggunakan instalasi PHP global Anda untuk menyajikan situs-situs Anda. Namun, jika Anda perlu mendukung beberapa versi PHP pada berbagai situs, Anda dapat menggunakan perintah `isolate` untuk menentukan versi PHP yang harus digunakan oleh situs tertentu. Perintah `isolate` mengonfigurasi Valet untuk menggunakan versi PHP yang ditentukan untuk situs yang terletak di direktori kerja Anda saat ini:

```shell
cd ~/kumpulan-proyek/situs-contoh

valet isolate php@8.0
```

Jika nama situs anda tidak sesuai dengan direktori tempat aplikasi ada berada, Anda dapat menentukan nama situs dengan opsi `--site`:

```shell
valet isolate php@8.0 --site="nama-situs"
```

Demi kenyamanan, Anda dapat menggunakan perintah `valet php`, `composer`, dan `which-php` untuk menjalankan panggilan secara langsung ke CLI PHP atau alat lain yang sesuai dengan versi PHP yang dikonfigurasikan pada situs tersebut:

```shell
valet php
valet composer
valet which-php
```

Anda dapat menjalankan perintah `isolated` untuk menampilkan semua daftar situs terisolasi yang Anda miliki serta versi PHP-nya:

```shell
valet isolated
```

Untuk mengembalikan situs ke Versi PHP yang terinstal secara global, Anda dapat menggunakan perintah `unisolate` dari direktori _root_ situs.

```shell
valet unisolate
```

<a name="sharing-sites"></a>
## Membagikan Situs

Valet juga menyertakan perintah untuk membagikan situs lokal anda ke seluruh dunia, gunanya untuk mempermudah pengujian pada perangkat _mobile_ atau dibagikan kepada rekan kerja dan mitra.

<a name="sharing-sites-via-ngrok"></a>
### Membagikan Situs Melalui Ngrok

Untuk membagikan situs, arahkan ke direktori situs pada terminal Anda dan jalankan perintah `share` milik Valet. URL yang dapat diakses publik akan dimasukkan ke dalam _clipboard_ Anda dan siap untuk ditempelkan langsung pada _browser_ Anda atau dibagikan kepada tim Anda:

```shell
cd ~/kumpulan-proyek/laravel

valet share
```

Untuk berhenti membagikan situs Anda, Anda dapat menekan `Control + C`. Membagikan situs Anda menggunakan Ngrok mengharuskan Anda untuk [memiliki akun Ngrok](https://dashboard.ngrok.com/signup) dan [mengatur sebuah autentikasi](https://dashboard.ngrok.com/get-started/your-authtoken).

> **Note**  
> Anda dapat memberikan parameter Ngrok tambahan pada perintah `share`, seperti `valet share --region=eu`. Untuk informasi lebih lanjut, lihat [dokumentasi Ngrok](https://ngrok.com/docs).

<a name="sharing-sites-via-expose"></a>
### Membagikan Situs melalui Expose

Jika [Expose](expose.dev) Anda telah terinstal, Anda bisa membagikan situs anda dengan mengarahkan ke direktori situs pada terminal Anda dan menjalankan perintah `expose`. Lihat [dokumentasi Expose](https://expose.dev/docs) untuk informasi mengenai parameter tambahan yang tersedia untuk perintah. Setelah membagikan situs, Expose akan menampilkan URL yang dapat dibagikan yang dapat Anda gunakan pada perangkat lain atau pada anggota tim yang lain:

```shell
cd ~/kumpulan-proyek/laravel

expose
```

Untuk berhenti membagikan, silahkan tekan `Control + C`

<a name="sharing-sites-on-your-local-network"></a>
### Membagikan Situs pada Jaringan Lokal

Valet membatasi lalu lintas yang masuk ke antarmuka internal `127.0.0.1` secara _default_ sehingga mesin pengembangan Anda tidak terpapar risiko keamanan dari Internet.

Jika Anda ingin mengizinkan perangkat lain pada jaringan lokal Anda untuk mengakses situs Valet pada mesin Anda melalui alamat IP mesin Anda (misalnya: `192.168.1.10/aplikasi-gue.test`), Anda perlu memodifikasi _file_ konfigurasi Nginx secara manual yang sesuai untuk situs tersebut untuk menghapus pembatasan pada direktif `listen`. Anda harus menghapus awalan `127.0.0.1:` pada direktif `listen` untuk _port_ 80 dan 443.

Jika Anda belum menjalankan `valet secure` pada proyek, Anda dapat membuka akses jaringan untuk semua situs non-HTTPS dengan memodifikasi _file_ `/usr/local/etc/nginx/valet/valet.conf`. Namun, jika Anda menyediakan situs proyek melalui HTTPS (Anda telah menjalankan `valet secure` untuk situs tersebut) maka Anda harus memodifikasi _file_ `~/.config/valet/Nginx/nama-aplikasi.test`.

Setelah Anda memperbarui konfigurasi Nginx, jalankan perintah `valet restart` untuk menerapkan perubahan konfigurasi.

<a name="site-specific-environment-variables"></a>
## Variabel Lingkungan Khusus Situs

Beberapa aplikasi yang menggunakan kerangka kerja lain mungkin bergantung pada variabel lingkungan server tetapi tidak menyediakan cara mengonfigurasi variabel tersebut untuk proyek Anda. Valet memungkinkan Anda untuk mengonfigurasi variabel lingkungan spesifik/khusus situs tersebut dengan menambahkan _file_ `.valet-env.php` di dalam _root_ proyek Anda. _File_ ini akan mengembalikan sebuah larik pasangan situs-variabel lingkungan yang akan ditambahkan ke larik `$_SERVER` global untuk setiap situs yang ditentukan dalam larik tersebut:

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
## Melakukan Proksi Layanan

Terkadang, Anda mungkin ingin memproksi domain Valet ke layanan lain pada mesin lokal Anda. Sebagai contoh, Anda mungkin terkadang perlu menjalankan Valet sembari menjalankan situs terpisah di Docker; namun, Valet dan Docker tidak dapat menautkan _port_ 80 secara bersamaan.

Untuk mengatasi hal ini, Anda dapat menggunakan perintah `proxy` untuk menghasilkan proksi. Sebagai contoh, Anda dapat memproksi semua lalu lintas dari `http://elasticsearch.test` ke `http://127.0.0.1:9200`:

```shell
# Proxy melalui HTTP...
valet proxy elasticsearch http://127.0.0.1:9200

# Proxy melalui TLS + HTTP/2...
valet proxy elasticsearch http://127.0.0.1:9200 --secure
```

Anda dapat menghapus sebuah proksi dengan menggunakan perintah `unproxy`:

```shell
valet unproxy elasticsearch
```

Anda juga dapat menggunakan perintah `proxies` untuk menampilkan daftar semua konfigurasi situs yang telah dilakukan proksi:

```shell
valet proxies
```

<a name="custom-valet-drivers"></a>
## _Driver_ Valet Kustom

Anda dapat menulis "driver" Valet Anda sendiri untuk menyediakan aplikasi-aplikasi PHP yang berjalan pada sebuah kerangka kerja atau CMS yang tidak didukung oleh Valet. Ketika Anda menginstal Valet, direktori `~/.config/valet/Drivers` dibuat yang berisi _file_ `SampleValetDriver.php`. _File_ ini berisi contoh implementasi _driver_ untuk mendemonstrasikan cara menulis _driver_ yang kustom. Menulis _driver_ hanya memerlukan implementasi tiga metode: `serves`, `isStaticFile`, dan `frontControllerPath`.

Ketiga metode tersebut menerima nilai `$sitePath`, `$siteName`, dan `$uri` sebagai argumennya. `$sitePath` adalah _path_ lengkap (_fully qualified path_) ke situs yang ingin disajikan pada mesin Anda, seperti `/Users/Lisa/kumpulan-proyek/aplikasi-gue`. `$siteName` adalah bagian "_host_"/"nama situs" dari domain (`aplikasi-gue`). `$uri` adalah URI untuk permintaan yang masuk (`/foo/bar`).

Setelah Anda menyelesaikan _driver_ Valet khusus Anda, letakkan pada direktori `~/.config/valet/Drivers` dengan menggunakan konvensi penamaan `FrameworkValetDriver.php`. Sebagai contoh, jika Anda menulis _driver_ valet khusus untuk WordPress, maka nama _file_ Anda akan menjadi `WordPressValetDriver.php`.

Sekarang, kita coba mengimplementasikan setiap metode dari kustom Valet yang perlu Anda tulis.

<a name="the-serves-method"></a>
#### Metode `serves`

Metode `services` harus mengembalikan `true` jika _driver_ Anda harus menangani permintaan yang masuk. Jika tidak, metode tersebut akan mengembalikan nilai `false`. Jadi, di dalam metode ini, Anda harus mencoba menentukan apakah `$sitePath` yang diberikan berisi proyek dengan tipe tertuntu yang Anda coba sajikan.

Sebagai contoh, bayangkan kita sedang menulis `WordPressValetDriver`. Metode `services` kita mungkin akan terlihat seperti ini:

    /**
     * Tentukan apakah driver melayani permintaan.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return bool
     */
    public function serves($sitePath, $siteName, $uri)
    {
        return is_dir($sitePath.'/wp-admin');
    }

<a name="the-isstaticfile-method"></a>
#### Metode `isStaticFile`

Metode `isStaticFile` harus menentukan apakah permintaan yang masuk adalah _file_ "statis", seperti gambar atau _stylesheet_. Jika _file_ tersebut besifat statis, metode ini harus mengembalikan _path_ lengkap ke _file_ statis yang terdapat pada _disk_. Jika permintaan yang masuk bukan untuk _file_ statis, metode harus mengembalikan `false`:

    /**
     * Tentukan apakah permintaan yang masuk adalah untuk berkas statis.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string|false
     */
    public function isStaticFile($sitePath, $siteName, $uri)
    {
        if (file_exists($staticFilePath = $sitePath.'/public/'.$uri)) {
            return $staticFilePath;
        }

        return false;
    }

> **Peringatan**  
> Metode `isStaticFile` hanya akan dipanggil jika metode `serves` mengembalikan nilai `true` untuk permintaan yang masuk dan URI permintaan bukan `/`.

<a name="the-frontcontrollerpath-method"></a>
#### Metode `frontControllerPath`

Metode `frontControllerPath` harus mengembalikan _path_ lengkap ke "pengendali depan" milik aplikasi Anda, yang biasanya berupa file "index.php" atau yang sejenisnya:

    /**
     * Dapatkan jalur yang sepenuhnya terselesaikan ke pengontrol depan aplikasi.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string
     */
    public function frontControllerPath($sitePath, $siteName, $uri)
    {
        return $sitePath.'/public/index.php';
    }

<a name="local-drivers"></a>
### _Driver_ Lokal

Jika Anda ingin mendefinisikan _driver_ Valet khusus untuk satu aplikasi, buat _file_ `LocalValetDriver.php` di dalam direktori _root_ milik aplikasi tersebut. _Driver_ kustom Anda perlu melakukan _extend_ terhadap kelas _driver_ dasar `ValetDriver` atau melakukan _extend_ terhadap _driver_ untuk aplikasi spesifik yang sudah ada seperti `LaravelValetDriver`:

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

<a name="other-valet-commands"></a>
## Perintah Valet Lainnya

Perintah | Deskripsi
------------- | -------------
`valet list` | Menampilkan daftar semua perintah Valet.
`valet forget` | Jalankan perintah ini dari direktori "terparkir" untuk menghapusnya dari daftar direktori terparkir.
`valet log` | Melihat daftar log yang ditulis oleh layanan milik Valet.
`valet paths` | Melihat semua _path_ yang "terparkir" milik Anda.
`valet restart` | Memulai ulang _daemon_ Valet.
`valet start` | Memulai _daemon_ Valet.
`valet stop` | Menghentikan _daemon_ Valet.
`valet trust` | Menambahkan _file sudoers_ untuk Brew dan Valet agar perintah Valet dapat dijalankan tanpa meminta kata sandi Anda.
`valet uninstall` | Copot pemasangan Valet: menampilkan instruksi untuk pencopotan pemasangan manual. Berikan opsi `--force` untuk menghapus semua sumber daya Valet secara agresif.

<a name="valet-directories-and-files"></a>
## Direktori & _File_ Valet

Anda mungkin memerlukan informasi mengenai direktori dan _file_ berikut ini saat memecahkan masalah dengan lingkungan Valet Anda:

#### `~/.config/valet`

Berisi semua konfigurasi Valet. Anda mungkin ingin menyimpan cadangan direktori ini.

#### `~/.config/valet/dnsmasq.d/`

Direktori ini berisi konfigurasi DNSMasq.

#### `~/.config/valet/Drivers/`

Direktori ini berisi driver-driver Valet. Driver menentukan bagaimana sebuah framework / CMS tertentu dilayani.

#### `~/.config/valet/Extensions/`

Direktori ini berisi ekstensi/perintah khusus Valet.

#### `~/.config/valet/Nginx/`

Direktori ini berisi semua konfigurasi situs Nginx milik Valet. _file_-_file_ ini dibangun/ditulis ulang saat menjalankan perintah `install` dan `secure`.

#### `~/.config/valet/Sites/`

Direktori ini berisi semua tautan simbolik untuk [proyek yang ditautkan](#the-link-command).

#### `~/.config/valet/config.json`

_File_ ini adalah _file_ konfigurasi utama mililk Valet.

#### `~/.config/valet/valet.sock`

This file is the PHP-FPM socket used by Valet's Nginx installation. This will only exist if PHP is running properly.

#### `~/.config/valet/Log/fpm-php.www.log`

This file is the user log for PHP errors.

#### `~/.config/valet/Log/nginx-error.log`

This file is the user log for Nginx errors.

#### `/usr/local/var/log/php-fpm.log`

This file is the system log for PHP-FPM errors.

#### `/usr/local/var/log/nginx`

Direktori ini berisi semua _log_ akses dan eror pada Nginx.

#### `/usr/local/etc/php/X.X/conf.d`

_File_ ini berisi _file-file_ `.ini` untuk berbagai pengaturan konfigurasi PHP.

#### `/usr/local/etc/php/X.X/php-fpm.d/valet-fpm.conf`

_File_ ini adalah _file_ konfigurasi PHP-FPM pool.

#### `~/.composer/vendor/laravel/valet/cli/stubs/secure.valet.conf`

_File_ ini adalah konfigurasi _default_ Nginx yang digunakan untuk membangun sertifikat SSL untuk situs Anda.

<a name="disk-access"></a>
### Akses _Disk_

Sejak macOS 10.14, [akses ke beberapa _file_ dan direktori telah dibatasi secara _default_](https://manuals.info.apple.com/MANUALS/1000/MA1902/en_US/apple-platform-security-guide.pdf). Pembatasan ini termasuk direktori Desktop, Dokumen, dan Unduhan. Selain itu, akses volume jaringan dan volume _removable_ juga dibatasi. Oleh karena itu, Valet merekomendasikan folder situs Anda berada di luar lokasi yang dilindungi ini.

Namun, jika Anda ingin menyajikan situs dari dalam salah satu lokasi tersebut, Anda harus memberikan "Akses Disk Penuh" kepada Nginx. Jika tidak, Anda mungkin akan mengalami eror server atau perilaku tak terduga lainnya dari Nginx, terutama saat melayani aset statis. Biasanya, macOS akan secara otomatis meminta Anda untuk memberikan akses penuh kepada Nginx ke lokasi-lokasi ini. Atau, Anda dapat melakukannya secara manual melalui `Preferensi Sistem` > `Keamanan & Privasi` > `Keprivasian` dan memilih `Akses Disk Penuh`. Selanjutnya, aktifkan entri `nginx` pada panel jendela utama.
