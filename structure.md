# Struktur Direktori

- [Pengantar](#introduction)
- [Direktori _Root_](#the-root-directory)
    - [Direktori `app`](#the-root-app-directory)
    - [Direktori `bootstrap`](#the-bootstrap-directory)
    - [Direktori `config`](#the-config-directory)
    - [Direktori `database`](#the-database-directory)
    - [Direktori `lang`](#the-lang-directory)
    - [Direktori `public`](#the-public-directory)
    - [Direktori `resources`](#the-resources-directory)
    - [Direktori `routes`](#the-routes-directory)
    - [Direktori `storage`](#the-storage-directory)
    - [Direktori `tests`](#the-tests-directory)
    - [Direktori `vendor`](#the-vendor-directory)
- [Direktori _App_](#the-app-directory)
    - [Direktori `Broadcasting`](#the-broadcasting-directory)
    - [Direktori `Console`](#the-console-directory)
    - [Direktori `Events`](#the-events-directory)
    - [Direktori `Exceptions`](#the-exceptions-directory)
    - [Direktori `Http`](#the-http-directory)
    - [Direktori `Jobs`](#the-jobs-directory)
    - [Direktori `Listeners`](#the-listeners-directory)
    - [Direktori `Mail`](#the-mail-directory)
    - [Direktori `Models`](#the-models-directory)
    - [Direktori `Notifications`](#the-notifications-directory)
    - [Direktori `Policies`](#the-policies-directory)
    - [Direktori `Providers`](#the-providers-directory)
    - [Direktori `Rules`](#the-rules-directory)

<a name="introduction"></a>
## Pengantar

Struktur aplikasi Laravel secara _default_ dimaksudkan untuk memberikan titik awal yang baik untuk aplikasi besar dan kecil. Tetapi Anda bebas mengatur aplikasi sesuka Anda. Laravel hampir tidak memberlakukan batasan untuk menempatkan kelas tertentu - selama Composer dapat memuat-otomatis kelas tersebut.

> **Catatan**  
> Baru menggunakan Laravel? Lihat [Laravel Bootcamp](https://bootcamp.laravel.com) untuk "memegang" langsung _framework_ ini sambil memandu Anda untuk membangun aplikasi Laravel pertama Anda.

<a name="the-root-directory"></a>
## Direktori _Root_

<a name="the-root-app-directory"></a>
#### Direktori App

Direktori `app` berisi kode inti aplikasi Anda. Kita akan menjelajahi direktori ini lebih detail segera; namun, hampir semua kelas pada aplikasi Anda akan berada di direktori ini.

<a name="the-bootstrap-directory"></a>
#### Direktori Bootstrap

Direktori `bootstrap` berisi _file_ `app.php` yang mem-_bootstrap framework_ Laravel. Direktori ini juga menampung direktori `cache` yang berisi _file_-_file_ yang dihasilkan _framework_ untuk pengoptimalan kinerja seperti rute dan _file_-_file_ layanan _cache_. Biasanya Anda tidak perlu mengubah _file_ apa pun di dalam direktori ini.

<a name="the-config-directory"></a>
#### Direktori Config

Direktori `config`, seperti namanya, berisi semua _file_ konfigurasi untuk aplikasi Anda. Merupakan hal yang baik untuk membaca semua _file_-_file_ di dalamnya dan membiasakan diri Anda dengan semua opsi yang tersedia.

<a name="the-database-directory"></a>
#### Direktori Database

Direktori `database` berisi migrasi basis data, _factory_ model, dan _seed_. Jika mau, Anda juga dapat menggunakan direktori ini untuk menyimpan basis data SQLite.

<a name="the-lang-directory"></a>
#### Direktori Lang

Direktori `lang` menampung semua _file_-_file_ bahasa milik aplikasi Anda.

<a name="the-public-directory"></a>
#### Direktori Public

Direktori `public` berisi _file_ `index.php`, yang merupakan titik awal untuk semua permintaan yang masuk ke aplikasi Anda. _File_ `index.php` juga mengkonfigurasi pemuatan otomatis. Direktori ini juga menampung aset Anda seperti gambar, JavaScript, dan CSS.

<a name="the-resources-directory"></a>
#### Direktori Resources

Direktori `resources` berisi [tampilan](/docs/{{version}}/views) Anda serta aset mentah Anda yang belum dikompilasi seperti CSS atau JavaScript.

<a name="the-routes-directory"></a>
#### Direktori Routes

Direktori `routes` berisi semua definisi rute untuk aplikasi Anda. Secara _default_, beberapa _file_ rute disertakan dalam Laravel: `web.php`, `api.php`, `console.php`, dan `channels.php`.

_File_ `web.php` berisi rute yang `RouteServiceProvider` tempatkan di grup _middleware_ `web`, yang menyediakan status _session_, perlindungan CSRF, dan enkripsi _cookie_. Jika aplikasi Anda tidak menawarkan _stateless_, RESTful API maka semua rute Anda kemungkinan besar akan ditentukan dalam _file_ `web.php` saja.

_File_ `api.php` berisi rute yang `RouteServiceProvider` tempatkan di grup _middleware_ `api`. Rute-rute ini dimaksudkan untuk menjadi _stateless_ sehingga permintaan yang masuk ke aplikasi melalui rute ini diautentikasi [melalui token](/docs/{{version}}/sanctum) dan tidak akan memiliki akses ke _state session_.

_File_ `console.php` adalah tempat di mana Anda dapat mendefinisikan semua perintah konsol berbasis _closure_. Setiap _closure_ terikat pada _instance_ perintah yang memungkinkan Anda melakukan pendekatan sederhana untuk berinteraksi dengan setiap metode IO milik perintah tersebut. Meskipun _file_ ini tidak mendefinisi rute HTTP, _file_ ini mendefinisikan titik awal (rute) berbasis konsol ke dalam aplikasi Anda.

_File_ `channels.php` adalah _file_ tempat Anda dapat mendaftarkan semua saluran [penyiaran _event_](/docs/{{version}}/broadcasting) yang didukung pada aplikasi Anda.

<a name="the-storage-directory"></a>
#### Direktori Storage

Direktori `storage` berisi _log_, _template_ Blade yang sudah dikompilasi, _session_ berbasis _file_, _cache_ dari _file_, dan _file_ lain yang dihasilkan oleh _framework_. Direktori ini dipisahkan menjadi direktori `app`, `framework`, dan `logs`. Direktori `app` dapat digunakan untuk menyimpan _file_ apa pun yang dihasilkan oleh aplikasi Anda. Direktori `framework` digunakan untuk menyimpan _file_ dan _cache_ yang dihasilkan _framework_. Terakhir, direktori `logs` berisi _file_ _log_ dari aplikasi Anda.

Direktori `storage/app/public` dapat digunakan untuk menyimpan _file_ yang dihasilkan pengguna—seperti avatar profil—yang harus dapat diakses secara publik. Anda dapat membuat tautan simbolis pada `public/storage` yang mengarah ke direktori ini. Anda dapat membuat tautan menggunakan perintah Artisan `php artisan storage:link`.

<a name="the-tests-directory"></a>
#### Direktori Tests

Direktori `tests` berisi pengujian otomatis milik Anda. Contoh pengujian unit dan pengujian fitur menggunakan [PHPUnit](https://phpunit.de/) telah tersedia pada aplikasi Laravel bawaan. Setiap kelas pengujian harus diakhiri dengan kata `Test`. Anda dapat menjalankan pengujian menggunakan perintah `phpunit` atau `php vendor/bin/phpunit`. Atau, jika Anda menginginkan representasi hasil pengujian yang lebih detail dan indah, Anda dapat menjalankan pengujian menggunakan perintah Artisan `php artisan test`.

<a name="the-vendor-directory"></a>
#### Direktori Vendor

Direktori `vendor` berisi dependensi-dependensi [Composer](https://getcomposer.org) untuk Aplikasi Anda.

<a name="the-app-directory"></a>
## Direktori App

Sebagian besar aplikasi Anda ditempatkan di direktori `app`. Secara _default_ direktori ini memiliki _namespace_ di bawah `App` dan dimuat secara otomatis oleh Composer menggunakan [standar pemuatan otomatis PSR-4](https://www.php-fig.org/psr/psr-4/).

Direktori `app` berisi berbagai direktori tambahan seperti `Console`, `Http`, dan `Providers`. Direktori `Console` dan `Http` sebagai penyedia API ke dalam inti aplikasi Anda. Protokol HTTP dan CLI merupakan mekanisme untuk berinteraksi dengan aplikasi Anda, tetapi sebenarnya tidak berisi logika aplikasi. Dengan kata lain, protokol HTTP dan CLI adalah dua cara untuk menerbitkan perintah ke aplikasi Anda. Direktori `Console` berisi semua perintah Artisan Anda, sedangkan direktori `Http` berisi _controller_, _middleware_, dan permintaan (_request_) Anda.

Berbagai direktori lain akan dibuat di dalam direktori `app` saat Anda menggunakan perintah `make` pada perintah Artisan untuk membuat beberapa kelas. Jadi, misalnya, direktori `app/Jobs` tidak akan ada sampai Anda menjalankan perintah Artisan `make:job` untuk menghasilkan kelas `Job`.

> **Catatan**  
> Banyak kelas di direktori `app` dapat dihasilkan oleh Artisan melalui perintahnya. Untuk meninjau perintah yang tersedia, jalankan perintah `php artisan list make` pada terminal Anda.

<a name="the-broadcasting-directory"></a>
#### Direktori Broadcasting

Direktori `Broadcasting` berisi semua kelas saluran siaran untuk aplikasi Anda. Kelas-kelas ini dihasilkan menggunakan perintah `make:channel`. Direktori ini tidak ada secara _default_, tetapi akan dibuat untuk Anda saat Anda membuat saluran tersebut untuk pertama kali. Untuk mempelajari lebih lanjut tentang saluran ini, lihat dokumentasinya di [penyiaran _event_](/docs/{{version}}/broadcasting).

<a name="the-console-directory"></a>
#### Direktori Console

Direktori `Console` berisi semua perintah Artisan kustom untuk aplikasi Anda. Perintah-perintah ini dapat dibuat menggunakan perintah `make:command`. Direktori ini juga menampung kernel konsol Anda, tempat di mana perintah Artisan kustom Anda yang didaftarkan, juga menjadi tempat untuk [tugas terjadwal](/docs/{{version}}/scheduling) milik Anda yang telah didefinisikan.

<a name="the-events-directory"></a>
#### Direktori Events

Direktori ini tidak ada secara _default_, tetapi akan dibuat untuk Anda dengan perintah Artisan `event:generate` dan `make:event`. Direktori `Events` berisi [kelas-kelas _event_](/docs/{{version}}/events). _Event_ dapat digunakan untuk memperingatkan bagian lain dari aplikasi Anda bahwa tindakan tertentu telah terjadi. Hal ini dapat memberikan fleksibilitas dan _decoupling_ yang baik.

<a name="the-exceptions-directory"></a>
#### Direktori Exceptions

Direktori `Exceptions` berisi penangan _exception_ milik aplikasi Anda dan juga merupakan tempat yang baik untuk menempatkan _exception_ apa saja yang ingin dilemparkan oleh aplikasi Anda. Jika Anda ingin menyesuaikan bagaimana _exception_ akan dicatat atau di-_render_, Anda harus memodifikasi kelas `Handler` pada direktori ini.

<a name="the-http-directory"></a>
#### Direktori Http

Direktori `Http` berisi _controller_, _middleware_, dan permintaan-permintaan formulir milik Anda. Hampir semua logika untuk menangani permintaan yang masuk ke aplikasi Anda akan ditempatkan di direktori ini.

<a name="the-jobs-directory"></a>
#### Direktori Jobs

Direktori ini tidak ada secara _default_, tetapi akan dibuat untuk Anda jika Anda menjalankan perintah Artisan `make:job`. Direktori `Jobs` menyimpan [_job_-_job_ yang dapat diantrekan](/docs/{{version}}/queues) untuk aplikasi Anda. `Job` mungkin diantrekan oleh aplikasi Anda atau dijalankan secara sinkron dalam siklus hidup permintaan saat ini. Tugas yang dijalankan secara sinkron selama permintaan saat ini terkadang disebut sebagai "perintah" karena merupakan implementasi dari [pola perintah](https://en.wikipedia.org/wiki/Command_pattern).

<a name="the-listeners-directory"></a>
#### Direktori Listeners

Direktori ini tidak ada secara _default_, tetapi akan dibuat untuk Anda jika Anda menjalankan perintah Artisan `event:generate` atau `make:listener`. Direktori `Listeners` berisi kelas yang menangani [_event_](/docs/{{version}}/events) Anda. _Listener_-_listener_ _event_ menerima _instance_ _event_ dan melaksanakan logika sebagai respons terhadap _event_ yang dipicu. Misalnya, _event_ `UserRegistered` mungkin ditangani oleh _listener_ `SendWelcomeEmail`.

<a name="the-mail-directory"></a>
#### Direktori Mail

Direktori ini tidak ada secara _default_, tetapi akan dibuat untuk Anda jika Anda menjalankan perintah Artisan `make:mail`. Direktori `Mail` berisi semua [kelas yang merepresentasikan email](/docs/{{version}}/mail) yang dikirim oleh aplikasi Anda. Objek email memungkinkan Anda untuk meng-enkapsulasi semua logika pembuatan email dalam satu kelas sederhana yang dapat dikirim menggunakan metode `Mail::send`.

<a name="the-models-directory"></a>
#### Direktori Models

Direktori `Models` berisi semua [kelas model Eloquent](/docs/{{version}}/eloquent). ORM Eloquent yang disertakan pada Laravel menyediakan implementasi ActiveRecord yang indah dan sederhana untuk bekerja dengan basis data Anda. Setiap tabel basis data memiliki "Model" yang sesuai yang digunakan untuk berinteraksi dengan tabel tersebut. Model memungkinkan Anda melakukan kueri data pada tabel Anda, serta menyisipkan rekaman baru ke dalam tabel.

<a name="the-notifications-directory"></a>
#### Direktori Notifications

Direktori ini tidak ada secara _default_, tetapi akan dibuat untuk Anda jika Anda menjalankan perintah Artisan `make:notification`. Direktori `Notifikasi` berisi semua [notifikasi](/docs/{{version}}/notifikasi) "transaksional" yang dikirim oleh aplikasi Anda, seperti notifikasi sederhana tentang peristiwa yang terjadi dalam aplikasi Anda. Fitur notifikasi Laravel mengabstraksi pengiriman notifikasi melalui berbagai _driver_ seperti email, Slack, SMS, atau disimpan dalam basis data.

<a name="the-policies-directory"></a>
#### Direktori Policies

Direktori ini tidak ada secara _default_, tetapi akan dibuat untuk Anda jika Anda menjalankan perintah Artisan `make:policy`. Direktori `Policies` berisi [kelas-kelas kebijakan otorisasi](/docs/{{version}}/authorization) untuk aplikasi Anda. Kebijakan (_policy_) digunakan untuk menentukan apakah pengguna dapat melakukan tindakan tertentu terhadap sumber daya Anda.

<a name="the-providers-directory"></a>
#### Direktori Providers

Direktori `Providers` berisi semua [penyedia layanan (_service provider_)](/docs/{{version}}/providers) untuk aplikasi Anda. Penyedia layanan melakukan _bootstrap_ aplikasi Anda dengan mengikat layanan pada wadah layanan (_service container_), mendaftarkan _event_, atau melakukan tugas lain untuk mempersiapkan aplikasi Anda untuk menangani permintaan yang masuk.

Pada aplikasi Laravel yang baru, direktori ini sudah berisi beberapa penyedia. anda bebas menambahkan penyedia anda sendiri ke direktori ini sesuai kebutuhan.

<a name="the-rules-directory"></a>
#### Direktori Rules

Direktori ini tidak ada secara _default_, tetapi akan dibuat untuk Anda jika Anda menjalankan perintah Artisan `make:rule`. Direktori `Rules` berisi objek aturan validasi kustom untuk aplikasi Anda. Aturan (_rule_) digunakan untuk mengenkapsulasi logika validasi yang rumit menjadi objek yang sederhana. Untuk informasi lebih lanjut, lihat [dokumentasi validasi](/docs/{{version}}/validation).
