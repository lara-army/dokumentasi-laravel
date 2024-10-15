# Siklus Hidup _Request_

- [Pendahuluan](#introduction)
- [Gambaran Umum Siklus Hidup](#lifecycle-overview)
  - [Langkah Pertama](#first-steps)
  - [HTTP / Kernel Konsol](#http-console-kernels)
  - [Penyedia Layanan](#service-providers)
  - [Perutean](#routing)
  - [Akhir Siklus](#finishing-up)
- [Fokus Pada Penyedia Pelayanan](#focus-on-service-providers)

<a name="introduction"></a>
## Pendahuluan

Ketika menggunakan peralatan apa pun di dunia nyata, Anda akan merasa lebih percaya diri jika Anda memahami cara kerja alat tersebut. Pengembangan aplikasi juga demikian. Ketika Anda memahami cara kerja peralatan pengembangan Anda, Anda akan merasa lebih nyaman dan percaya diri dalam menggunakannya.

Tujuan dari dokumen ini adalah untuk memberi Anda gambaran umum yang baik dan menyeluruh tentang cara kerja kerangka kerja (_framework_) Laravel. Dengan mengenal kerangka kerja Laravel secara keseluruhan dengan lebih baik, semuanya akan terasa tidak terlalu “_magical_” dan Anda akan lebih percaya diri dalam membangun aplikasi Anda. Jika Anda tidak langsung memahami semua istilahnya, jangan berkecil hati! Cobalah untuk mendapatkan pemahaman dasar tentang apa yang sedang terjadi, dan pengetahuan Anda akan bertambah saat Anda menjelajahi bagian lain dari dokumentasi.

<a name="lifecycle-overview"></a>
## Gambaran Umum Siklus Hidup

<a name="first-steps"></a>
### Langkah Awal

Titik masuk/awal (_entry point_) untuk semua permintaan (_request_) ke aplikasi Laravel adalah _file_ `public/index.php`. Semua permintaan diarahkan ke _file_ ini karena konfigurasi server web Anda (Apache / Nginx). _File_ `index.php` tidak mengandung banyak kode. Sebaliknya, _file_ ini merupakan titik awal untuk memuat keseluruhan kerangka kerja yang tersisa.

_File_ `index.php` memuat definisi _autoloader_ yang dibuat oleh Composer, kemudian mengambil _instance_ aplikasi Laravel dari `bootstrap/app.php`. Tindakan pertama yang dilakukan oleh Laravel sendiri adalah membuat _instance_ aplikasi / [kontainer layanan](/docs/{{version}}/container).

<a name="http-console-kernels"></a>
### HTTP / Kernel Konsol

Selanjutnya, permintaan yang masuk dikirimkan ke kernel HTTP atau kernel konsol, tergantung pada jenis permintaan yang masuk ke dalam aplikasi. Kedua kernel ini berfungsi sebagai lokasi pusat di mana semua permintaan mengalir. Untuk saat ini, mari kita fokus pada kernel HTTP, yang terletak di `app/Http/Kernel.php`.

Kernel HTTP memperluas (_extends_) kelas `Illuminate\Foundation\Http\Kernel`, yang mendefinisikan sebuah larik(_array_) `bootstrappers` yang akan dijalankan sebelum permintaan tereksekusi. _Bootstrappers_ ini mengonfigurasi penanganan _error_, mengonfigurasi pencatatan _log_, [mendeteksi lingkungan aplikasi](/docs/{{version}}/configuration#environment-configuration), dan melakukan tugas-tugas lain yang perlu dilakukan sebelum penanganan yang sebernarnya. Biasanya, kelas-kelas ini menangani konfigurasi internal Laravel yang tidak perlu Anda khawatirkan.

Kernel HTTP juga mendefinisikan sebuah daftar [_middleware_](/docs/{{version}}/middleware) HTTP yang harus dilalui oleh semua permintaan sebelum ditangani oleh aplikasi. _Middleware_ ini menangani pembacaan dan penulisan [sesi HTTP](/docs/{{version}}/session), menentukan apakah aplikasi sedang dalam mode pemeliharaan, [memverifikasi token CSRF](/docs/{{version}}/csrf), dan banyak lagi. Kita akan membahas lebih lanjut tentang ini segera.

Ciri-ciri metode untuk metode `handle` milik kernel HTTP cukup sederhana: menerima sebuah `Request` dan mengembalikan `Response`. Bayangkan kernel sebagai sebuah kotak hitam besar yang mewakili seluruh aplikasi Anda. Berikan permintaan (_request_) HTTP kepadanya dan ia akan mengembalikan respons (_response_) HTTP.

<a name="service-providers"></a>
### Penyedia Layanan

Salah satu tindakan _bootstrapping_ kernel yang paling penting adalah memuat [penyedia-penyedia layanan](/docs/{{version}}/providers) untuk aplikasi Anda. Penyedia layanan bertanggung jawab untuk melakukan _bootstrapping_ terhadap semua komponen kerangka kerja, seperti basis data, antrean, validasi, dan komponen perutean (_routing_). Semua penyedia layanan untuk aplikasi dikonfigurasikan dalam larik `providers` pada berkas konfigurasi `config/app.php`.

Laravel akan melakukan perulangan (_iterate_) daftar dan _instance_ setiap penyedia tersebut. Setelah membuat _instance_ penyedia, metode `register` akan dipanggil pada semua penyedia. Kemudian, setelah semua penyedia layanan terdaftar, metode `boot` akan dipanggil pada setiap penyedia layanan. Hal ini dilakukan agar penyedia layanan dapat bergantung pada setiap pengikatan (_binding_) kontainer yang telah terdaftar dan tersedia pada saat metode `boot` dijalankan.

Pada dasarnya setiap fitur utama yang ditawarkan oleh Laravel di-_bootstrap_ dan dikonfigurasi oleh penyedia layanan. Karena mereka melakukan _bootstrap_ dan mengonfigurasi begitu banyak fitur yang ditawarkan oleh framework, _service provider_ adalah aspek terpenting dari keseluruhan proses _bootstrap_ Laravel.

<a name="routing"></a>
### Perutean

Salah satu penyedia layanan yang paling penting dalam aplikasi Anda adalah `App\Providers\RouteServiceProvider`. Penyedia layanan ini memuat _file_-_file_ rute yang terdapat di dalam direktori `routes` aplikasi Anda. Silakan buka kode `RouteServiceProvider` dan lihat cara kerjanya!

Setelah aplikasi di-_bootstrap_ dan semua penyedia layanan telah terdaftar, `Request` akan diserahkan ke _router_ untuk dikirim. _Router_ akan mengirimkan permintaan ke rute atau pengontrol (_controller_), serta menjalankan _middleware_ rute tersebut.

_Middleware_ menyediakan mekanisme yang nyaman untuk menyaring atau memeriksa permintaan HTTP yang masuk ke aplikasi Anda. Sebagai contoh, Laravel menyertakan _middleware_ yang memverifikasi apakah pengguna aplikasi Anda sudah terautentikasi. Jika pengguna tidak diautentikasi, _middleware_ akan mengarahkan pengguna ke layar _login_. Namun, jika pengguna diautentikasi, _middleware_ akan mengizinkan permintaan untuk melanjutkan lebih jauh ke dalam aplikasi. Beberapa _middleware_ ditugaskan ke semua rute di dalam aplikasi, seperti yang didefinisikan dalam properti `$middleware` pada kernel HTTP Anda, sementara beberapa lainnya hanya ditugaskan ke rute atau grup rute tertentu saja. Anda dapat mempelajari lebih lanjut tentang middleware dengan membaca [dokumentasi _middleware_](/docs/{{versi}}/middleware) yang lengkap.

Jika permintaan melewati semua _middleware_ yang ditugaskan untuk rute yang ditetapkan, metode rute atau pengontrol akan dieksekusi dan respons yang dikembalikan oleh metode rute atau pengontrol akan dikirim kembali melalui rantai _middleware_ milik rute.

<a name="finishing-up"></a>
### Akhir Siklus

Setelah metode rute atau kontroller mengembalikan sebuah respons, respons akan bergerak kembali ke luar melalui _middleware_ milik rute, sehingga aplikasi memiliki kesempatan untuk memodifikasi atau memeriksa respons yang keluar.

Akhirnya, setelah respons bergerak kembali melalui _middleware_, metode `handle` pada kernel HTTP mengembalikan objek respons dan _file_ `index.php` memanggil metode `send` pada respons yang dikembalikan. Metode `send` mengirimkan konten respons ke peramban(_browser_) web pengguna. Kita telah menyelesaikan perjalanan kita melalui seluruh siklus hidup permintaan Laravel!

<a name="focus-on-service-providers"></a>
## Fokus Pada Penyedia Pelayanan

Penyedia layanan adalah kunci utama untuk melakukan _bootstrap_ pada aplikasi Laravel. _Instance_ aplikasi dibuat, penyedia layanan didaftarkan, dan permintaan diserahkan ke aplikasi _bootstrap_. Sesederhana itu!

Memiliki pemahaman yang kuat tentang bagaimana aplikasi Laravel dibangun dan di-_bootstrap_ melalui penyedia layanan merupakan hal yang sangat berharga. Penyedia layanan _default_ aplikasi Anda disimpan dalam direktori `app/Providers`.

Secara _default_, `AppServiceProvider` tidak memuat banyak hal. Penyedia tersebut merupakan tempat yang tepat untuk menambahkan _bootstrapping_ dan _binding_ kontainer layanan milik aplikasi Anda. Untuk aplikasi yang besar, Anda mungkin ingin membuat beberapa penyedia layanan, dengan masing-masing _bootstrapping_ yang lebih terperinci untuk layanan tertentu yang digunakan oleh aplikasi Anda.
