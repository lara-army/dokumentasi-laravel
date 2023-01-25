# Struktur Direktori

- [Pengantar](#pengantar)
- [Direktori _Root_](#direktori-root)
    - [Direktori `app`](#direktori-app)
    - [Direktori `bootstrap`](#direktori-bootstrap)
    - [Direktori `config`](#direktori-config)
    - [Direktori `database`](#direktori-database)
    - [Direktori `lang`](#direktori-lang)
    - [Direktori `public`](#direktori-public)
    - [Direktori `resources`](#direktori-resource)
    - [Direktori `routes`](#direktori-routes)
    - [Direktori `storage`](#direktori-storage)
    - [Direktori `tests`](#direktori-tests)
    - [Direktori `vendor`](#direktori-vendor)
- [Direktori _App_](#direktori-app)
    - [Direktori `Broadcasting`](#direktori-broadcasting)
    - [Direktori `Console`](#direktori-console)
    - [Direktori `Events`](#direktori-events)
    - [Direktori `Exceptions`](#direktori-exceptions)
    - [Direktori `Http`](#direktori-http)
    - [Direktori `Jobs`](#direktori-jobs)
    - [Direktori `Listeners`](#direktori-listeners)
    - [Direktori `Mail`](#direktori-mail)
    - [Direktori `Models`](#direktori-models)
    - [Direktori `Notifications`](#direktori-notifications)
    - [Direktori `Policies`](#direktori-policies)
    - [Direktori `Providers`](#direktori-providers)
    - [Direktori `Rules`](#direktori-rules)

<a name="introduction"></a>
## Pengantar

Struktur aplikasi Laravel secara _default_ dimaksudkan untuk memberikan titik awal yang bagus untuk aplikasi besar dan kecil. Tetapi anda bebas mengatur aplikasi sesuka anda. Laravel hampir tidak memberlakukan batasan di mana kelas tertentu berada - selama Composer dapat memuat-otomatis kelas tersebut.

> **Catatan**  
> Baru menggunakan Laravel? Lihat [Laravel Bootcamp](https://bootcamp.laravel.com) untuk melihat langsung _framework_ ini sementara kami memandu anda membuat aplikasi Laravel pertama anda.

<a name="the-root-directory"></a>
## Direktori _Root_

<a name="the-root-app-directory"></a>
#### Direktori `app` 

`app` direktori berisi kode inti aplikasi anda. Kita akan menjelajahi direktori ini lebih detail segera; namun, hampir semua _class_ di aplikasi anda akan berada di direktori ini.

<a name="the-bootstrap-directory"></a>
#### Direktori `bootstrap` 

Direktori `bootstrap` berisi _file_ `app.php` dimana terdapat _framework_ bootstrap. Direktori ini juga menampung direktori `cache` yang berisi _file_ kerangka kerja yang dihasilkan untuk pengoptimalan kinerja seperti rute dan layanan _file_ _cache_. Biasanya anda tidak perlu mengubah _file_ apa pun di dalam direktori ini.

<a name="the-config-directory"></a>
#### Direktori `config`

Direktori `config`, seperti namanya, berisi semua _file_ konfigurasi aplikasi anda. Merupakan ide bagus untuk membaca semua _file_ ini dan membiasakan diri dengan semua opsi yang tersedia untuk anda.

<a name="the-database-directory"></a>
#### Direktori `database`

Direktori `database` berisi migrasi _database_, model _factories_, dan _seeds_. Jika mau, anda juga dapat menggunakan direktori ini untuk menyimpan _database SQLite_.

<a name="the-lang-directory"></a>
#### Direktori `lang`

Direktori `lang` menampung semua _file_ bahasa aplikasi anda.

<a name="the-public-directory"></a>
#### Direktori `public`

Direktori `public` berisi _file_ `index.php`, yang merupakan titik masuk untuk semua permintaan yang masuk ke aplikasi anda dan mengonfigurasi pemuatan otomatis. Direktori ini juga menampung aset anda seperti gambar, JavaScript, dan CSS.

<a name="the-resources-directory"></a>
#### Direktori `resources`

Direktori `resources` berisi [tampilan](/docs/{{version}}/views) anda serta aset mentah anda yang belum dikompilasi seperti CSS atau JavaScript.

<a name="the-routes-directory"></a>
#### Direktori `routes`

Direktori `routes` berisi semua definisi rute untuk aplikasi anda. Secara _default_, beberapa _file_ rute disertakan dengan Laravel: `web.php`, `api.php`, `console.php`, dan `channels.php`.

_File_ `web.php` berisi rute yang ditempatkan `RouteServiceProvider` di grup _middleware_ `web`, yang menyediakan status _session_, perlindungan CSRF, dan enkripsi _cookie_. Jika aplikasi anda tidak menawarkan _stateless_, _RESTful API_ maka semua rute anda kemungkinan besar akan ditentukan dalam _file_ `web.php`.

_File_ `api.php` berisi rute yang ditempatkan `RouteServiceProvider` di grup _middleware_ `api`. Rute-rute ini dimaksudkan untuk menjadi _stateless_ jadi permintaan yang masuk ke aplikasi melalui rute ini dimaksudkan untuk diautentikasi [melalui token](/docs/{{version}}/sanctum) dan tidak akan memiliki akses ke keadaan _session_.

_File_ `console.php` adalah tempat anda dapat menentukan semua perintah konsol berbasis _closures_. Setiap _closures_ terikat pada _instance_ perintah yang memungkinkan pendekatan sederhana untuk berinteraksi dengan metode _IO_ setiap perintah. Meskipun _file_ ini tidak menentukan rute HTTP, _file_ ini menentukan titik masuk (rute) berbasis konsol ke dalam aplikasi anda.

_File_ `channels.php` adalah tempat anda dapat mendaftarkan semua saluran [penyiaran acara](/docs/{{version}}/broadcasting) yang didukung aplikasi anda.

<a name="the-storage-directory"></a>
#### Direktori `storage`

Direktori `storage` berisi log anda, template Blade yang dikompilasi, _sessions_ berbasis _file_, _cache_ _file_, dan _file_ lain yang dihasilkan oleh _framework_. Direktori ini dipisahkan menjadi direktori `app`, `framework`, dan `logs`. Direktori `app` dapat digunakan untuk menyimpan _file_ apa pun yang dihasilkan oleh aplikasi anda. Direktori `framework` digunakan untuk menyimpan _file_ dan _cache_ yang dihasilkan _framework_. Terakhir, direktori `logs` berisi _file_ log aplikasi anda.

Direktori `storage/app/public` dapat digunakan untuk menyimpan _file_ yang dibuat pengguna, seperti avatar profil, yang harus dapat diakses publik. anda harus membuat tautan simbolis di `public/storage` yang mengarah ke direktori ini. anda dapat membuat tautan menggunakan perintah `php artisan storage:link` pada perintah artisan.

<a name="the-tests-directory"></a>
#### Direktori `tests`

Direktori `tests` berisi pengujian otomatis anda. Contoh pengujian unit [PHPUnit](https://phpunit.de/) dan pengujian fitur disediakan secara langsung. Setiap kelas tes harus diakhiri dengan kata `Test`. anda dapat menjalankan pengujian menggunakan perintah `phpunit` atau `php vendor/bin/phpunit`. Atau, jika anda menginginkan representasi hasil pengujian yang lebih detail dan indah, anda dapat menjalankan pengujian menggunakan perintah Artisan `php artisan test`.

<a name="the-vendor-directory"></a>
#### Direktori `vendor`

Direktori `vendor` berisi dependensi [Composer](https://getcomposer.org) anda.

<a name="the-app-directory"></a>
## Direktori _App_

Sebagian besar aplikasi anda ditempatkan di direktori `app`. Secara _default_ direktori ini adalah _namespace_ dalam `App` dan dimuat secara otomatis oleh Composer menggunakan [standar pemuatan otomatis PSR-4](https://www.php-fig.org/psr/psr-4/).

Direktori `app` berisi berbagai direktori tambahan seperti `Console`, `Http`, dan `Providers`. Mengingat direktori `Console` dan `Http` sebagai penyedia API ke dalam inti aplikasi anda. Protokol HTTP dan CLI keduanya merupakan mekanisme untuk berinteraksi dengan aplikasi anda, tetapi sebenarnya tidak berisi logika aplikasi. Dengan kata lain, itu adalah dua cara untuk mengeluarkan perintah ke aplikasi anda. Direktori `Console` berisi semua perintah Artisan anda, sedangkan direktori `Http` berisi controller, _middleware_, dan permintaan anda.

Berbagai direktori lain akan dibuat di dalam direktori `app` saat anda menggunakan perintah `make` pada artisan _command_ untuk membuat kelas. Jadi, misalnya, direktori `app/Jobs` tidak akan ada sampai anda menjalankan perintah Artisan `make:job` untuk menghasilkan kelas pekerjaan.

> **Catatan**  
> Banyak kelas di direktori `app` dapat dihasilkan oleh Artisan melalui perintah. Untuk meninjau perintah yang tersedia, jalankan perintah `php artisan list make` di terminal anda.

<a name="the-broadcasting-directory"></a>
#### Direktori `Broadcasting`

Direktori `Broadcasting` berisi semua kelas saluran siaran untuk aplikasi anda. Kelas-kelas ini dihasilkan menggunakan perintah `make:channel`. Direktori ini tidak ada secara _default_, tetapi akan dibuat untuk anda saat anda membuat saluran pertama kali. Untuk mempelajari lebih lanjut tentang saluran, lihat dokumentasi di [penyiaran acara](/docs/{{version}}/broadcasting).

<a name="the-console-directory"></a>
#### Direktori `Console`

Direktori `Console` berisi semua perintah Artisan khusus untuk aplikasi anda. Perintah-perintah ini dapat dibuat menggunakan perintah `make:command`. Direktori ini juga menampung _kernel_ konsol anda, tempat perintah Artisan kustom anda yang didaftarkan dan [tugas terjadwal](/docs/{{version}}/scheduling) anda yang telah ditentukan.

<a name="the-events-directory"></a>
#### Direktori `Events`

Direktori ini tidak ada secara _default_, tetapi akan dibuat untuk anda dengan perintah Artisan `event:generate` dan `make:event`. Direktori `Events` berisi [_event classes_](/docs/{{version}}/events). _Events_ dapat digunakan untuk memperingatkan bagian lain dari aplikasi anda bahwa tindakan tertentu telah terjadi, memberikan banyak fleksibilitas dan _decoupling_.

<a name="the-exceptions-directory"></a>
#### Direktori `Exceptions`

Direktori `Exceptions` berisi pengendali pengecualian aplikasi anda dan juga merupakan tempat yang baik untuk menempatkan pengecualian apa pun yang dilemparkan oleh aplikasi anda. Jika anda ingin menyesuaikan bagaimana pengecualian dicatat atau dirender, anda harus memodifikasi kelas `Handler` di direktori ini.

<a name="the-http-directory"></a>
#### Direktori `Http`

Direktori `Http` berisi controller, _middleware_, dan permintaan formulir anda. Hampir semua logika untuk menangani permintaan yang masuk ke aplikasi anda akan ditempatkan di direktori ini.

<a name="the-jobs-directory"></a>
#### Direktori `Jobs`

Direktori ini tidak ada secara _default_, tetapi akan dibuat untuk anda jika anda menjalankan perintah Artisan `make:job`. Direktori `Jobs` menyimpan [_queueable jobs_](/docs/{{version}}/queues) untuk aplikasi anda. `Jobs` mungkin diantrekan oleh aplikasi anda atau dijalankan secara sinkron dalam _lifecycle_ permintaan saat itu. Tugas yang dijalankan secara sinkron selama permintaan saat ini terkadang disebut sebagai "perintah" karena merupakan implementasi dari [_command pattern_](https://en.wikipedia.org/wiki/Command_pattern).

<a name="the-listeners-directory"></a>
#### Direktori `Listeners`

Direktori ini tidak ada secara _default_, tetapi akan dibuat untuk anda jika anda menjalankan perintah Artisan `event:generate` atau `make:listener`. Direktori `Listeners` berisi kelas yang menangani [_events_](/docs/{{version}}/events) anda. _Event listeners_ menerima _instance_ _event_ dan melakukan logika sebagai respons terhadap _events_ yang dipicu. Misalnya, peristiwa `UserRegistered` mungkin ditangani oleh pendengar `SendWelcomeEmail`.

<a name="the-mail-directory"></a>
#### Direktori `Mail`

Direktori ini tidak ada secara _default_, tetapi akan dibuat untuk anda jika anda menjalankan perintah Artisan `make:mail`. Direktori `Mail` berisi semua [kelas yang mewakili email](/docs/{{version}}/mail) yang dikirim oleh aplikasi anda. Objek email memungkinkan anda merangkum semua logika pembuatan email dalam satu kelas sederhana yang dapat dikirim menggunakan metode `Mail::send`.

<a name="the-models-directory"></a>
#### Direktori `Models`

Direktori `Models` berisi semua [kelas model Eloquent](/docs/{{version}}/eloquent). _Eloquent ORM_ yang disertakan dengan Laravel menyediakan implementasi _ActiveRecord_ yang indah dan sederhana untuk bekerja dengan _database_ anda. Setiap tabel _database_ memiliki "Model" yang sesuai yang digunakan untuk berinteraksi dengan tabel tersebut. Model memungkinkan anda melakukan kueri data di tabel anda, serta menyisipkan rekaman baru ke dalam tabel.

<a name="the-notifications-directory"></a>
#### Direktori `Notifications`

Direktori ini tidak ada secara _default_, tetapi akan dibuat untuk anda jika anda menjalankan perintah Artisan `make:notification`. Direktori `Notifikasi` berisi semua [notifikasi](/docs/{{version}}/notification) "transaksional" yang dikirim oleh aplikasi anda, seperti notifikasi sederhana tentang peristiwa yang terjadi dalam aplikasi anda. Fitur notifikasi Laravel mengabstraksi pengiriman notifikasi melalui berbagai driver seperti email, Slack, SMS, atau disimpan dalam _database_.

<a name="the-policies-directory"></a>
#### Direktori `Policies`

Direktori ini tidak ada secara _default_, tetapi akan dibuat untuk anda jika anda menjalankan perintah Artisan `make:policy`. Direktori `Policies` berisi [class kebijakan otorisasi](/docs/{{version}}/authorization) untuk aplikasi anda. Kebijakan digunakan untuk menentukan apakah pengguna dapat melakukan tindakan tertentu terhadap sumber daya.

<a name="the-providers-directory"></a>
#### Direktori `Providers`

Direktori `Providers` berisi semua [penyedia layanan (_service provider_)](/docs/{{version}}/providers) untuk aplikasi anda. Penyedia layanan mem-_bootstrap_ aplikasi anda dengan mengikat layanan di wadah layanan, mendaftarkan acara, atau melakukan tugas lain untuk mempersiapkan aplikasi anda untuk permintaan masuk.

Pada aplikasi Laravel yang baru, direktori ini sudah berisi beberapa penyedia. anda bebas menambahkan penyedia anda sendiri ke direktori ini sesuai kebutuhan.

<a name="the-rules-directory"></a>
#### Direktori `Rules`

Direktori ini tidak ada secara _default_, tetapi akan dibuat untuk anda jika anda menjalankan perintah Artisan `make:rule`. Direktori `Rules` berisi objek aturan validasi khusus untuk aplikasi anda. Aturan digunakan untuk merangkum logika validasi yang rumit dalam objek sederhana. Untuk informasi lebih lanjut, lihat [dokumentasi validasi](/docs/{{version}}/validation).
