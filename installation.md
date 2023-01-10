# Instalasi

- [Instalasi](#instalasi)
  - [Bertemu Laravel](#bertemu-laravel)
    - [Kenapa Laravel?](#kenapa-laravel)
      - [Sebuah Kerangka Kerja Progresif](#sebuah-kerangka-kerja-progresif)
      - [Sebuah Kerangka Kerja yang Dapat Dikembangkan](#sebuah-kerangka-kerja-yang-dapat-dikembangkan)
      - [Sebuah Kerangka Kerja Komunitas](#sebuah-kerangka-kerja-komunitas)
  - [Proyek Laravel Pertamamu](#proyek-laravel-pertamamu)
  - [Laravel \& Docker](#laravel--docker)
    - [Mulai Di macOS](#mulai-di-on-macos)
    - [Mulai Di Windows](#getting-started-on-windows)
      - [Mengembangkan Di Dalam WSL2](#developing-within-wsl2)
    - [Mulai Di Linux](#getting-started-on-linux)
    - [Pilih Layanan Sail Kamu](#choosing-your-sail-services)
  - [Konfigurasi Awal](#initial-configuration)
    - [Konfigurasi Berbasis _Environment_](#environment-based-configuration)
    - [Basis Data \& Migrasi](#databases--migrations)
  - [Langkah Selanjutnya](#next-steps)
    - [Laravel Sebagai Kerangka Kerja _Full Stack_](#laravel-the-full-stack-framework)
    - [Laravel Sebagai _API Backend_](#laravel-the-api-backend)

<a name="meet-laravel"></a>

## Bertemu Laravel

Laravel adalah kerangka kerja aplikasi web dengan sintaksis yang ekspresif dan elegan. Sebuah kerangka kerja web memberikan struktur dan titik awal untuk membuat aplikasi kamu, sehingga kamu bisa fokus membuat sesuatu yang keren sementara kita bekerja dengan detail-detailnya.

Laravel berusaha memberikan pengalaman pengembang yang luar biasa sambil menyediakan fitur-fitur yang kuat seperti injeksi dependensi yang mendalam, lapisan abstraksi basis data yang ekspresif, antrian dan pekerjaan yang dijadwalkan, uji unit dan integrasi, dan lainnya.

Tidak peduli apakah kamu baru memulai kerangka kerja web PHP atau sudah bertahun-tahun pengalaman, Laravel adalah kerangka kerja yang bisa tumbuh bareng kamu. Kita bakal bantu kamu mengambil langkah pertama sebagai seorang pengembang web atau memberikan dorongan saat kamu meningkatkan keahlianmu ke tingkat berikutnya. Kita nggak sabar mau liat apa yang bakal kamu buat.

> **Catatan**
> Belum pernah coba Laravel? Lihat [Laravel Bootcamp](https://bootcamp.laravel.com) untuk tur langsung tentang kerangka kerja ini, dan kami akan ajak kamu membuat aplikasi Laravel pertamamu.

<a name="why-laravel"></a>

### Kenapa Laravel?

Banyak alat dan kerangka kerja yang bisa kamu pakai untuk membuat aplikasi web. Tapi menurut kita, Laravel adalah pilihan terbaik untuk membuat aplikasi web _full-stack_ modern.

#### Sebuah Kerangka Kerja Progresif

Kita suka menyebut Laravel "kerangka kerja progresif". Maksudnya, Laravel bisa tumbuh bareng kamu. Kalau kamu baru mulai belajar membuat aplikasi web, koleksi dokumentasi, panduan, dan [tutorial video](https://laracasts.com) Laravel yang luas bakal bantu kamu belajar tanpa jadi kewalahan.

Kalau kamu developer senior, Laravel kasih alat-alat keren untuk [injeksi dependensi](/docs/{{version}}/container), [uji unit](/docs/{{version}}/testing), [antrian](/docs/{{version}}/queues), [event-event real-time](/docs/{{version}}/broadcasting), dan lainnya. Laravel di-tuning dengan rapi untuk membuat aplikasi web profesional dan siap menangani beban kerja enterprise.

#### Sebuah Kerangka Kerja yang Dapat Dikembangkan

Laravel memiliki skalabilitas yang luar biasa. Dibantu oleh sifat ramah skala dari PHP dan dukungan terintegrasi Laravel untuk sistem cache terdistribusi yang cepat seperti Redis, proses skalabilitas horizontal dengan Laravel menjadi mudah. Bahkan, aplikasi Laravel telah dengan mudah dapat dioptimalkan untuk menangani jumlah permintaan yang sangat tinggi, seperti ratusan juta permintaan per bulan.

Need extreme scaling? Platforms like [Laravel Vapor](https://vapor.laravel.com) allow you to run your Laravel application at nearly limitless scale on AWS's latest serverless technology.

#### Sebuah Kerangka Kerja Komunitas

Laravel menggabungkan paket-paket terbaik dalam ekosistem PHP untuk menawarkan kerangka kerja yang paling kokoh dan ramah untuk pengembang yang tersedia. Selain itu, ribuan pengembang berbakat dari seluruh dunia telah [berkontribusi pada kerangka kerja](https://github.com/laravel/framework). Siapa tahu, mungkin kamu bahkan akan menjadi kontributor Laravel.

<a name="your-first-laravel-project"></a>

## Proyek Laravel Pertamamu

Sebelum membuat proyek Laravel pertamamu, kamu sebaiknya memastikan bahwa di perangkat kamu sudah terinstal PHP dan [Composer](https://getcomposer.org). Jika kamu melakukan pengembangan di macOS, PHP dan Composer dapat diinstal melalui [Homebrew](https://brew.sh/). Tambahan, kami juga menyarankan untuk [menginstal Node and NPM](https://nodejs.org).

Setelah kamu menginstal PHP dan Composer, kamu dapat membuat proyek Laravel melalui perintah Composer `create-project`:

```nothing
composer create-project laravel/laravel example-app
```

Atau, kamu bisa membuat proyek Laravel dengan menginstal _Laravel installer_ secara global melalui Composer:

```nothing
composer global require laravel/installer

laravel new example-app
```

Setelah proyek dibuat, jalankan server pengembangan local milik Laravel dengan menggunakan perintah _Laravel's Artisan CLI_ `serve`:

```nothing
cd example-app

php artisan serve
```

Setelah kamu menjalankan server pengembangan Artisan, aplikasi kamu akan dapat diakses pada peramban web kamu di `http://localhost:8000`. Selanjutnya, kamu siap untuk [memulai mengambil langkah selanjutnya ke dalam ekosistem Laravel](#next-steps). Tentu saja, kamu juga mau [mengkonfigurasikan sebuah basis data](#databases-and-migrations).

> **Catatan**  
> Jika kamu mau memulai mengembangkan aplikasi Laravelmu, pertimbangkan untuk menggunakan salah satu [_starter kits_](/docs/{{version}}/starter-kits). _Starter kits_ Laravel menyediakan perancah autentikasi _backend_ dan _frontend_ untuk aplikasi Laravel barumu.

<a name="laravel-and-docker"></a>

## Laravel & Docker

Kami ingin semudah mungkin untuk memulai dengan Laravel terlepas dari sistem operasi pilihan kamu. Jadi, ada berbagai pilihan untuk mengembangkan dan menjalankan proyek Laravel pada mesin lokal kamu.
Meskipun kamu mungkin ingin menjelajahi opsi-opsi ini di lain waktu, Laravel menyediakan [Sail](/docs/{{version}}/sail), solusi bawaan untuk menjalankan proyek Laravel kamu menggunakan [Docker](https://www.docker.com).

Docker adalah alat untuk menjalankan aplikasi dan layanan dalam "kontainer" kecil dan ringan yang tidak mengganggu perangkat lunak atau konfigurasi yang diinstal mesin lokal kamu. Ini berarti kamu tidak perlu khawatir tentang mengkonfigurasi atau menyiapkan alat pengembangan yang rumit seperti server web dan basis data pada mesin lokal kamu. Untuk memulainya, kamu hanya perlu menginstal [Docker Desktop](https://www.docker.com/products/docker-desktop).

Laravel Sail adalah antarmuka baris perintah ringan untuk berinteraksi dengan konfigurasi standar Docker Laravel. Sail menyediakan titik awal yang bagus untuk membangun aplikasi Laravel menggunakan PHP, MySQL, dan Redis tanpa memerlukan pengalaman Docker sebelumnya.

> **Catatan**  
> Sudah menjadi ahli Docker? Jangan khawatir! Segala sesuatu tentang Sail dapat dikustomisasi menggunakan _file_ `docker-compose.yml` yang disertakan dengan Laravel.

<a name="getting-started-on-macos"></a>

### Mulai di macOS

Jika kamu melakukan pengembangan di Mac dan [Docker Desktop](https://www.docker.com/products/docker-desktop) sudah terinstal, kamu bisa menggunakan perintah terminal sederhana untuk membuat proyek Laravel baru. Sebagai contoh, untuk membuat proyek Laravel baru di dalam sebuah direktori dengan nama "example-app", kamu dapat menjalakan perintah ini di teriminal kamu:

```shell
curl -s "https://laravel.build/example-app" | bash
```

Tentu saja, kamu dapat mengubah "example-app" di URL ini menjadi apapun yang kamu mau - hanya saja pastikan nama aplikasi hanya mengandung karakter angka-huruf, tanda hubung, dan garis bawah. Direktori aplikasi Laravel akan dibuat di dalam direktori dimana kamu mengeksekusi perintahnya.

Instalasi Sail mungkin memakan waktu beberapa menit sementara kontainer aplikasi Sail dibuat di mesin lokal kamu.

Setelah proyek selesai dibuat, kamu dapat menavigasi ke direktori aplikasi dan menjalankan Laravel Sail. Laravel Sail menyediakan antarmuka baris perintah yang sederhana untuk berinteraksi dengan konfigurasi standar Docker Laravel:

```shell
cd example-app

./vendor/bin/sail up
```

Setelah kontainer Docker aplikasi dijalankan, kamu dapat mengakses aplikasinya pada peramban web kamu di: http://localhost.

> **Catatan**  
> Untuk mempelajari Laravel Sail lebih lanjut, tinjau [dokumentasi lengkapnya](/docs/{{version}}/sail).

<a name="getting-started-on-windows"></a>

### Mulai di Windows

Sebelum kita membuat aplikasi Laravel baru di mesin Windowsmu, pastikan untuk menginstal [Docker Desktop](https://www.docker.com/products/docker-desktop). Lalu, pastikan bahwa _Windows Subsystem for Linux 2 (WSL2)_ sudah terinstal dan sudah diaktifkan. WSL memungkinkan kamu untuk menjalankan _binary executables_ Linux secara _native_ di Windows 10. Informasti tentang cara menginstal dan mengaktifkan WSL2 dapat ditemukan di dalam [dokumentasi lingkungan pengembang _Microsoft_](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

> **Catatan**  
> Setelah menginstal dan mengaktifkan WSL2, kamu harus pastikan bahwa _Docker Desktop_ sudah [dikonfigurasi untuk menggunakan WSL2 _backend_](https://docs.docker.com/docker-for-windows/wsl/).

Selanjutnya, kamu siap untuk membuat proyek Laravel pertamamu. Jalankan [_Windows Terminal_](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701?rtc=1&activetab=pivot:overviewtab) dan mulai sesi terminal baru untuk WSL2 sistem operasi Linux. Lalu, kamu dapat menggunakan perintah terminal sederhana untuk membuat proyek Laravel baru. Sebagai contoh, untuk membuat aplikasi Laravel di dalam direktori dengan nama "example-app", kamu dapat menjalankan perintah ini di terminal kamu:

```shell
curl -s https://laravel.build/example-app | bash
```

Tentu saja, kamu dapat mengubah "example-app" di URL ini menjadi apapun yang kamu mau - hanya saja pastikan nama aplikasi hanya mengandung karakter angka-huruf, tanda hubung, dan garis bawah. Direktori aplikasi Laravel akan dibuat di dalam direktori dimana kamu mengeksekusi perintahnya.

Instalasi Sail mungkin memakan waktu beberapa menit sementara kontainer aplikasi Sail dibuat di mesin lokal kamu.

Setelah proyek selesai dibuat, kamu dapat menavigasi ke direktori aplikasi dan menjalankan Laravel Sail. Laravel Sail menyediakan antarmuka baris perintah yang sederhana untuk berinteraksi dengan konfigurasi standar Docker Laravel:

```shell
cd example-app

./vendor/bin/sail up
```

Setelah kontainer Docker aplikasi dijalankan, kamu dapat mengakses aplikasinya pada peramban web kamu di: http://localhost.

> **Catatan**  
> Untuk mempelajari Laravel Sail lebih lanjut, tinjau [dokumentasi lengkapnya](/docs/{{version}}/sail).

#### Mengembangkan Di Dalam WSL2

Tentu saja, kamu akan butuh untuk bisa memodifikasi aplikasi Laravel yang dibuat di dalam instalasi WSL2-mu. Untuk menyelesaikan ini, kami merekomendasikan untuk menggunakan [Visual Studio Code](https://code.visualstudio.com) editor dari _Microsoft_ dan ekstensi dari mereka untuk [Pengembangan Jarak Jauh](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack).

Setelah alat-alat ini terinstal, kamu dapat membuka proyek Laravel dengan mengeksekusi perintah `code .` dari direktori _root_ aplikasi kamu dengan menggunakan terminal Windows.

<a name="getting-started-on-linux"></a>

### Mulai di Linux

Jika kamu melakukan pengembangan di Linux dan [Docker Compose](https://docs.docker.com/compose/install/) sudah terinstal, kamu bisa menggunakan perintah terminal sederhana untuk membuat proyek Laravel baru. Sebagai contoh, untuk membuat proyek Laravel baru di dalam sebuah direktori dengan nama "example-app", kamu dapat menjalakan perintah ini di teriminal kamu:

```shell
curl -s https://laravel.build/example-app | bash
```

Tentu saja, kamu dapat mengubah "example-app" di URL ini menjadi apapun yang kamu mau - hanya saja pastikan nama aplikasi hanya mengandung karakter angka-huruf, tanda hubung, dan garis bawah. Direktori aplikasi Laravel akan dibuat di dalam direktori dimana kamu mengeksekusi perintahnya.

Instalasi Sail mungkin memakan waktu beberapa menit sementara kontainer aplikasi Sail dibuat di mesin lokal kamu.

Setelah proyek selesai dibuat, kamu dapat menavigasi ke direktori aplikasi dan menjalankan Laravel Sail. Laravel Sail menyediakan antarmuka baris perintah yang sederhana untuk berinteraksi dengan konfigurasi standar Docker Laravel:

```shell
cd example-app

./vendor/bin/sail up
```

Setelah kontainer Docker aplikasi dijalankan, kamu dapat mengakses aplikasinya pada peramban web kamu di: http://localhost.

> **Catatan**  
> Untuk mempelajari Laravel Sail lebih lanjut, tinjau [dokumentasi lengkapnya](/docs/{{version}}/sail).

<a name="choosing-your-sail-services"></a>

### Pilih Layanan Sail Kamu

Saat membuat aplikasi Laravel baru melalui Sail, kamu dapat menggunakan variabel _string_ kueri `with` untuk memilih layanan mana yang harus dikonfigurasikan dalam _file_ `docker-compose.yml` aplikasi barumu. Layanan yang tersedia meliputi `mysql`, `pgsql`, `mariadb`, `redis`, `memcached`, `meilisearch`, `minio`, `selenium`, dan `mailhog`:

```shell
curl -s "https://laravel.build/example-app?with=mysql,redis" | bash
```

Jika kamu tidak menentukan layanan mana yang ingin dikonfigurasi, _stack_ bawaan dari `mysql`, `redis`, `meilisearch`, `mailhog`, and `selenium` akan dikonfigurasi.

Kamu dapat menginstruksikan Sail untuk menginstal [Devcontainer](/docs/{{version}}/sail#using-devcontainers) bawaan dengan menambahkan `devcontainer` parameter ke URL:

```shell
curl -s "https://laravel.build/example-app?with=mysql,redis&devcontainer" | bash
```

<a name="initial-configuration"></a>

## Konfigurasi Awal

Semua konfigurasi _file_ untuk kerangka kerja Laravel disimpan di dalam direktori `config`. Setiap opsi didokumentasikan, jadi silahkan melihat-lihat _file_ dan membiasakan diri dengan opsi-opsi yang tersedia untuk kamu.

Laravel hampir tidak memerlukan konfigurasi tambahan diluar kotak. Kamu bebas untuk memulai mengembangkan! Namun, kamu mungkin ingin meninjau _file_ `config/app.php` dan dokumentasinya. Ini berisi beberapa opsi seperti `timezone` dan `locale` yang mungkin ingin kamu ubah sesuai dengan aplikasimu.

<a name="environment-based-configuration"></a>

### Konfigurasi Berbasis _Environment_

Karena banyak nilai dari opsi konfigurasi Laravel dapat bervariasi tergantung pada aplikasimu berjalan di mesin lokal atau server web produksi, banyak nilai konfigurasi penting didefinisikan menggunakan _file_ `.env` yang ada di _root_ aplikasimu.

_File_ `.env` jangan dikomit ke _source control_ aplikasimu, karena setiap pengembang / server yang menggunakan aplikasimu mungkin memerlukan konfigurasi yang berbeda. Selain itu, ini akan menjadi resiko keamanan jika penyusup mendapatkan akses ke repositori _source control_ kamu, karena setiap kredensial sensitif akan terkspos.

> **Catatan**  
> Untuk informasi lebih lanjut mengenai _file_ `.env` dan konfigurasi berbasis _environment_, lihat [dokumentasi lengkap konfigurasi](/docs/{{version}}/configuration#environment-configuration).

<a name="databases-and-migrations"></a>

### Basis Data & Migrasi

Setelah kamu membuat aplikasi Laravelmu, kamu mungkin ingin menyimpan beberapa data di dalam sebuah basis data. Secara _default_, _file_ konfigurasi `.env` aplikasimu menentukan bahwa Laravel akan berinteraksi dengan basis data MySQL dan akan mengakses basis data di `127.0.0.1`. Jika kamu melakukan pengembangan di macOS dan ingin menginstal MySQL, Postgres, atau Redis di lokal, kamu mungkin merasa nyaman menggunakan [DBngin](https://dbngin.com/).

Jika kamu tidak ingin menginstal MySQL atau Postgres di mesin lokalmu, kamu selalu dapat menggunakan basis data [SQLite](https://www.sqlite.org/index.html). SQLite adalah mesin basis data yang kecil, cepat, dan mandiri. Untuk memulai, buat sebuah basis data SQLite dengan membuat sebuah _file_ SQLite kosong. Biasanya, _file_ ini akan berada pada direktori `database` dari aplikasi Laravelmu:

```shell
touch database/database.sqlite
```

Selanjutnya, perbarui _file_ konfigurasi `.env` untuk menggunakan _driver_ basis data `sqlite` Laravel. Kamu dapat menghapus opsi-opsi konfigurasi basis data lainnya:

```ini
DB_CONNECTION=sqlite # [tl! add]
DB_CONNECTION=mysql # [tl! remove]
DB_HOST=127.0.0.1 # [tl! remove]
DB_PORT=3306 # [tl! remove]
DB_DATABASE=laravel # [tl! remove]
DB_USERNAME=root # [tl! remove]
DB_PASSWORD= # [tl! remove]
```

Setelah mengonfigurasi basis data SQLitemu, kamu dapat menjalankan [migrasi basis data](/docs/{{version}}/migrations), yang akan membuat tabel basis data aplikasimu:

```shell
php artisan migrate
```

<a name="next-steps"></a>

## Langkah Selanjutnya

Setelah kamu membuat proyek Laravelmu, kamu mungkin bertanya-tanya apa yang harus dipelajari selanjutnya. Pertama, kami sangat menyarankan untuk mengenal cara kerja Laravel dengan membaca dokumentasi berikut:

<div class="content-list" markdown="1">

- [Siklus Permintaan](/docs/{{version}}/lifecycle)
- [Konfigurasi](/docs/{{version}}/configuration)
- [Struktur Direktori](/docs/{{version}}/structure)
- [_Frontend_](/docs/{{version}}/frontend)
- [Kontainer Layanan](/docs/{{version}}/container)
- [Fasad](/docs/{{version}}/facades)

</div>

Bagaimana kamu ingin menggunakan Laravel juga akan menentukan langkah selanjutnya dalam perjalananmu. Ada berbagai cara untuk menggunakan Laravel, dan kami akan mengeksplorasi dua kasus penggunaan utama untuk kerangka kerja di bawah ini.

> **Catatan**
> Belum pernah coba Laravel? Lihat [Laravel Bootcamp](https://bootcamp.laravel.com) untuk tur langsung tentang kerangka kerja ini, dan kami akan ajak kamu membuat aplikasi Laravel pertamamu.

<a name="laravel-the-fullstack-framework"></a>

### Laravel Sebagai Kerangka Kerja _Full Stack_

Laravel dapat berfungsi sebagai kerangka kerja _full stack_. Maksud kami dari kerangka kerja _full stack_ yaitu kamu akan menggunakan Laravel untuk merutekan permintaan ke aplikasimu dan menampilkan _frontend_ kamu dengan menggunakan [Template blade](/docs/{{version}}/blade) atau teknologi _hybrid_ aplikasi satu halaman seperti [Inertia](https://inertiajs.com). Ini adalah cara paling umum untuk menggunakan kerangka kerja Laravel, dan, menurut pendapat kami, ini adalah cara paling produktif untuk menggunakan Laravel.

Jika kamu berencana menggunakan Laravel seperti ini, kamu mungkin ingin melihat dokumentasi kami di [pengembangan _frontend_](/docs/{{version}}/frontend), [perutean](/docs/{{version}}/routing), [tampilan](/docs/{{version}}/views), atau [Eloquent ORM](/docs/{{version}}/eloquent). Selain itu, kamu mungkin tertarik untuk mempelajari tentang paket komunitas seperti [Livewire](https://laravel-livewire.com) dan [Inertia](https://inertiajs.com). Paket-paket ini memungkinkanmu untuk menggunakan Laravel sebagai kerangka kerja _full stack_ sambil menikmati banyaknya manfaat UI yang disediakan oleh aplikasi satu halaman Javascript.

Jika kamu menggunakan Laravel sebagai kerangka kerja _full stack_, kami juga sangat merekomendasikan kamu untuk mempelajari bagaimana cara untuk menyusun CSS dan Javascript aplikasimu menggunakan [Vite](/docs/{{version}}/vite).

> **Catatan**  
> Jika kamu ingin mulai membangun aplikasimu, lihatlah salah satu [_starter kits_ aplikasi](/docs/{{version}}/starter-kits) resmi kami.

<a name="laravel-the-api-backend"></a>

### Laravel Sebagai _API Backend_

Laravel juga dapat digunakan sebagai _API backend_ ke aplikasi satu halaman ataupun aplikasi seluler. Misalnya, kamu mungkin ingin menggunakan Laravel sebagai _API backend_ untuk aplikasi [Next.js](https://nextjs.org)-mu. Dalam konteks ini, kamu dapat menggunakan Laravel untuk menyediakan [autentikasi](/docs/{{version}}/sanctum) dan penyimpanan/pengambilan data untuk aplikasimu, sekaligus memanfaatkan layanan hebat dari Laravel seperti antrean, email, notifikasi, dan banyak lagi.

Jika kamu berencana menggunakan Laravel seperti ini, kamu mungkin ingin melihat dokumentasi kami di [perutean](/docs/{{version}}/routing), [Laravel Sanctum](/docs/{{version}}/sanctum), dan [Eloquent ORM](/docs/{{version}}/eloquent).

> **Catatan**  
> Butuh permulaan untuk merancah _backend_ Laravel dan _frontend_ Next.js? Laravel Breeze menawarkan sebuah [_API stack_](/docs/{{version}}/starter-kits#breeze-and-next) serta [implementasi _frontend_](https://github.com/laravel/breeze-next) sehingga kamu dapat memulai dalam hitungan menit.
