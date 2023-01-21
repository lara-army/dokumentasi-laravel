# _Session_ HTTP

- [Pendahuluan](#introduction)
    - [Konfigurasi](#configuration)
    - [Prasyarat _Driver_](#driver-prerequisites)
- [Berinteraksi dengan _Session_](#interacting-with-the-session)
    - [Melakukan _Retrieve_ Data](#retrieving-data)
    - [Menyimpan Data](#storing-data)
    - [_Flash_ Data](#flash-data)
    - [Menghapus Data](#deleting-data)
    - [Re-_generate_ ID _Session_](#regenerating-the-session-id)
- [Pemblokiran _Session_](#session-blocking)
- [Menambahkan _Driver Session_ ynag Kustom](#adding-custom-session-drivers)
    - [Implementasi _Driver_](#implementing-the-driver)
    - [Mendaftarkan _Driver_](#registering-the-driver)

<a name="introduction"></a>
## Pendahuluan

Karena aplikasi yang digerakkan dengan HTTP bersifat _stateless_, _session_ memberikan sebuah cara untuk menyimpan informasi tentang pengguna yang dapat melintasi banyak _request_. Informasi pengguna seperti ini biasanya ditempatkan pada penyimpanan / _backend_ yang persisten, yang dapat diakses pada _request_ yang berikutnya.

Laravel dilengkapi dengan berbagai _session backend_ yang diakses melalui API yang ekspresif dan terpadu. Dengan dukungan untuk _backend_-_backend_ populer seperti [Memcached](https://memcached.org), [Redis](https://redis.io), dan _database_ lain yang termasuk.

<a name="configuration"></a>
### Konfigurasi

_File_ konfigurasi _session_ aplikasi Anda terdapat pada `config/session.php`. Pastikan untuk meninjau semua opsi yang tersedia untuk Anda pada _file_ tersebut. Secara _default_, Laravel dikonfigurasi untuk menggunakan `file` sebagai _driver session_, yang akan berfungsi baik pada kebanyakan aplikasi. Jika aplikasi Anda akan menerapkan _load balance_—untuk membagi beban kerja aplikasi—ke beberapa server web, Anda sebaiknya memilih penyimpanan terpusat yang dapat diakses oleh semua server, seperti Redis atau _database_.

Opsi konfigurasi `driver` untuk _session_ akan menentukan di mana data _session_ akan disimpan untuk setiap _request_. Laravel dilengkapi dengan _driver_-_driver_ hebat yang sudah termasuk di dalam "kemasan"-nya.

<div class="content-list" markdown="1">

- `file` - _session_ disimpan pada `storage/framework/sessions`.
- `cookie` - _session_ disimpan secara aman pada _cookie_ yang terenkripsi.
- `database` - _session_ disimpan pada basis data relasional.
- `memcached` / `redis` - _session_ disimpan pada salah satu layanan tersebut, penyimpanan berbasis _cache_.
- `dynamodb` - _session_ disimpan pada DynamoDB AWS.
- `array` - _session_ disimpan pada sebuah _array_ PHP yang tidak—mungkin—persisten.

</div>

> **Catatan**  
> _Driver_ dengan _array_ utamanya digunakan ketika melakukan [pengujian](/docs/{{version}}/testing) untuk mencegah data yang disimpan pada _session_ menjadi persisten.

<a name="driver-prerequisites"></a>
### Prasyarat _Driver_

<a name="database"></a>
#### _Database_

Ketika menggunakan _driver_ `database` untuk _session_, Anda akan membutuhkan sebuah tabel untuk menampung rekaman _session_. Contoh deklarasi `Schema` untuk tabel tersebut dapat dilihat pada kode di bawah ini:

    Schema::create('sessions', function ($table) {
        $table->string('id')->primary();
        $table->foreignId('user_id')->nullable()->index();
        $table->string('ip_address', 45)->nullable();
        $table->text('user_agent')->nullable();
        $table->text('payload');
        $table->integer('last_activity')->index();
    });

Anda dapat menggunakan perintah Artisan `session:table` untuk menghasilkan _migration_ tersebut. Untuk mempelajari lebih lanjut tentang _migration_ basis data, Anda dapat membacanya secara utuh pada [dokumentasi _migration_](/docs/{{version}}/migrations):

```shell
php artisan session:table

php artisan migrate
```

<a name="redis"></a>
#### Redis

Sebelum menggunakan _session_ Redis pada Laravel, Anda harus menginstal ekstensi PHP bernama PhpRedis melalui PECL atau _package_ `predis/predis` (~1.0) melalui Composer. Untuk informasi lebih lanjut tentang konfigurasi Redis, silahkan mempelajari [dokumentasi Redis](/docs/{{version}}/redis#configuration) milik Laravel.

> **Catatan**  
> Pada _file_ konfigurasi `session`, opsi `connection` dapat digunakan untuk menentukan koneksi Redis yang digunakan untuk _session_.

<a name="interacting-with-the-session"></a>
## Berinteraksi dengan _Session_

<a name="retrieving-data"></a>
### Melakukan _Retrieve_ Data

Terdapat dua cara utama untuk bekerja dengan data _session_ pada Laravel: _helper_ global `session` dan melalui _instance_ `Request`. Pertama-tama, mari lihat cara mengakses _session_ melalui _instance_ `Request` yang dapat di-_type-hint_ pada _closure route_ atau metode _controller_. Perlu diingat, dependensi pada metode _controller_ telah terinjeksi secara otomatis melalui [_service container_](/docs/{{version}}/container) Laravel:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Menampilkan profil pengguna.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function show(Request $request, $id)
        {
            $value = $request->session()->get('key');

            //
        }
    }

Ketika anda melakukan _retrieve_ untuk sebuah _item_ pada _sessiom_, Anda juga dapat mengoper nilai _default_ sebagai argumen kedua pada metode `get`. Nilai _default_ ini akan dikembalikan jika nama kunci yang sudah ditentukan tidak ditemukan pada _session_. Jika Anda mengoper sebuah _closure_ sebagai nilai _default_, maka _closure_ tersebut akan dieksekusi jika nama kunci yang ditentukan tidak terdapat pada _session_:

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function () {
        return 'default';
    });

<a name="the-global-session-helper"></a>
#### _Helper Session_ Global

Anda juga dapat menggunakan fungsi global PHP bernama `session` untuk me-_retrieve_ dan menyimpan data pada _session_. Ketika _helper_ `session` dipanggil dengan satu argumen _string_, fungsi tersebut akan mengembalikan nilai dari nama kunci pada _session_. Namun, ketika _helper_ tersebut dipanggil dengan argumen _array_ pasangan kunci-nilai (_key-value pairs_ / _array_ asosiatif), Nilai-nilai tersebut akan disimpan pada _session_:

    Route::get('/beranda', function () {
        // Retrieve data dari session...
        $value = session('nama');

        // Menentukan nilai default...
        $value = session('nama', 'Zain');

        // Menyimpan sebuah data pada session...
        session(['nama' => 'Zain Adam']);
    });

> **Catatan**  
> Dalam praktiknya, terdapat sebuah perbedaan kecil pada pemanfaatan _session_ antara menggunakan _instance_ HTTP _request_ dan _helper_ global _session_. Ke-dua metode tersebut dapat dimasukkan ke dalam [pengujian](/docs/{{version}}/testing) melalui metode `assertSessionHss` yang terdapat pada semua kasus/skenario pengujian.

<a name="retrieving-all-session-data"></a>
#### _Retrieve_ Semua Data pada _Session_

Jika Anda ingin me-_retrieve_ semua data yang terdapat pada _session_, Anda dapat menggunakan metode `all`:

    $data = $request->session()->all();

<a name="determining-if-an-item-exists-in-the-session"></a>
#### Memastikan Apakah _Item_ Terdapat pada _Session_

Untuk memastikan apakah sebuah _item_ (data) telah terdapat di dalam _session_, Anda dapat menggunakan metode `has`. Metode `has` mengembalikan nilai `true` jika _item_ tersebut telah terdapat di dalam _session_ dan tidak bernilai `null`:

    if ($request->session()->has('pengguna')) {
        //
    }

Untuk memastikan apakah sebuah _item_ telah terdapat di dalam _session_ walaupun bernilai `null`, Anda dapat menggunakan metode `exists`:

    if ($request->session()->exists('pengguna')) {
        //
    }

Untuk menentukan apakah sebuah _item_ tidak terdapat di dalam _session_, Anda dapat menggunakan metode `missing`. Metode `missing` mengembalikan nilai `true` jika _item_ tersebut tidak terdapat di dalam _session_:

    if ($request->session()->missing('pengguna')) {
        //
    }

<a name="storing-data"></a>
### Menyimpan Data

Untuk menyimpan data ke dalam _session_, Anda biasanya akan menggunakan metode `put` milik _instance request_ atau _helper_ global `session`:

    // Melalui instance request...
    $request->session()->put('nama', 'Zain');

    // Melalui the helper global "session"...
    session(['nama' => 'Zain']);

<a name="pushing-to-array-session-values"></a>
#### Mendorong Sebuah Nilai ke dalam Array pada _Session_

Metode `push` dapat digunakan untuk mendorong (menambahkan _item_ di ujung _array_) sebuah nilai baru ke dalam nilai _session_ yang berjenis _array_. Sebagai contoh, jika sebuah kunci `pengguna.daftarTeman` adalah sebuah _array_ yang berisi nama-nama teman milik pengguna, Anda dapat memasukkan sebuah nilai baru (misalnya nama teman yang baru) ke dalam _array_ tersebut:

    $request->session()->push('pengguna.daftarTeman', 'Taylor Otwell');

<a name="retrieving-deleting-an-item"></a>
#### Me-_Retrieve_ & Menghapus _Item_

Metode `pull` akan me-_retrieve_ dan menghapus sebuah _item_ pada _session_ dengan satu _statement_ (perintah):

    $value = $request->session()->pull('key', 'default');

<a name="#incrementing-and-decrementing-session-values"></a>
#### Menambah & Mengurangi Nilai dalam _Session_

Jika terdapat data berjenis _integer_ pada _session_ Anda yang nilainya ingin ditambah atau dikurangi, Anda dapat menggunakan metode `increment` dan `decrement`:

    $request->session()->increment('count');

    $request->session()->increment('count', $incrementBy = 2);

    $request->session()->decrement('count');

    $request->session()->decrement('count', $decrementBy = 2);

<a name="flash-data"></a>
### _Flash_ Data

Terkadang Anda ingin menyimpan _item_ pada _session_ yang hanya tersedia pada _request_ yang berikutnya saja. Anda dapat melakukannya dengan menggunakan metode `flash`. Dengan menggunakan metode tersebut, data yang tersimpan pada _session_ hanya akan tersedia pada _request_ HTTP yang berikutnya saja. Setelah HTTP _request_ tersebut berakhir, data yang telah di-_flash_ tadi akan terhapus. _Flash_ data sangat berguna untuk pesan status—atau informasi dari sistem—yang berumur singkat:

    $request->session()->flash('status', 'Email telah terkirim!');

Jika Anda ingin mempertahankan data _flash_ untuk beberapa _request_, Anda dapat menggunakan metode `reflash` yang akan menjaga semua data _flash_ Anda pada _request_ kedepannya. Jika Anda hanya perlu untuk mempertahankan data _flash_ tertentu saja, Anda dapat menggunakan metode `keep`:

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

Untuk mempertahankan _flash_ data pada _request_ saat ini saja, Anda dapat menggunakan metode `now`:

    $request->session()->now('status', 'Email telah terkirim!');

<a name="deleting-data"></a>
### Menghapus Data

Metode `forget` akan menghapus sebuah data pada _session_. Jika Anda ingin menghapus semua data, Anda dapat menggunakan metode `flush`:

    // "Forget" key tunggal...
    $request->session()->forget('nama');

    // "Forget" beberapa key...
    $request->session()->forget('nama', 'status']);

    $request->session()->flush();

<a name="regenerating-the-session-id"></a>
### Re-_generate_ ID _Session_

Membuat ulang (re-_generate_) ID _session_ sering dilakukan untuk mencegah pengguna jahat yang mengeksploitasi serangan [_Session Fixation_] (https://owasp.org/www-community/attacks/Session_fixation) pada aplikasi Anda.

Laravel secara otomatis membuat ulang ID _session_ pada saat autentikasi jika Anda menggunakan salah satu dari [peralatan awal aplikasi](/docs/{{version}}/starter-kits) Laravel atau [Laravel Fortify](/docs/{{version}}/fortify); namun, jika Anda perlu membuat ulang ID _session_ secara manual, Anda dapat menggunakan metode `regenerate`:

    $request->session()->regenerate();

Jika Anda perlu membuat ulang ID _session_ dan menghapus semua data pada _session_ dalam satu pernyataan, Anda dapat menggunakan metode `invalidate`:

    $request->session()->invalidate();

<a name="session-blocking"></a>
## Pemblokiran _Session_

> **Peringatan**  
> Untuk memanfaatkan pemblokiran _session_, aplikasi Anda harus menggunakan _driver cache_ yang mendukung [_atomic lock_](/docs/{{version}}/cache#atomic-locks). Saat ini, _driver cache_ tersebut meliputi _driver_ `memcached`, `dynamodb`, `redis`, dan `database`. Selain itu, Anda tidak boleh menggunakan _driver session_ `cookie`.

Secara _default_, Laravel mengizinkan semua _request_ untuk menggunakan _session_ yang sama untuk dieksekusi secara bersamaan. Sebagai contoh, jika Anda menggunakan sebuah pustaka HTTP JavaScript untuk membuat dua _request_ HTTP ke aplikasi Anda yang dieksekusi pada waktu yang sama. Untuk kebanyakan aplikasi, hal ini bukanlah sebuah masalah; walaupun begitu, hal tersebut dapat menyebabkan _session data loss_/inkonsistensi data pada halaman tersebut karena membuat beberapa _request_ secara bersamaan ke dua _endpoint_ berbeda dan masing-masing _endpoint_ dapat menulis data pada _session_.

Untuk mengatasi hal ini, Laravel menyediakan fungsionalitas yang memungkinkan Anda untuk membatasi permintaan bersamaan untuk sesi tertentu. Untuk memulai, Anda dapat dengan mudah merantai metode `block` ke dalam definisi rute Anda. Dalam contoh ini, permintaan yang masuk ke titik akhir `/profile` akan mendapatkan kunci sesi. Sementara kunci ini ditahan, setiap permintaan yang masuk ke titik akhir `/profile` atau `/order` yang memiliki ID sesi yang sama akan menunggu permintaan pertama selesai dieksekusi sebelum melanjutkan eksekusi:

Untuk memitigasi hal ini, Laravel menyediakan fungsionalitas yang memungkinkan Anda untuk membatasi _request_ yang bersamaan untuk _session_ tertentu. Untuk melakukannya, Anda dapat merantai metode `block` pada pendefinisian rute Anda. Pada contoh ini, sebuah _request_ masuk ke _endpoint_ `/profile` akan mendapatkan gembok _session_. Ketika penguncian terjadi, _request_ apapun yang masuk ke _endpoint_ `/profile` atau `/order` yang berbagi ID _session_ yang sama akan menunggu _request_ pertama selesai dieksekusi terlebih dahulu sebelum mengeksekusi _request_ yang selanjutnya:

    Route::post('/profil', function () {
        //
    })->block($lockSeconds = 10, $waitSeconds = 10)

    Route::post('/order', function () {
        //
    })->block($lockSeconds = 10, $waitSeconds = 10)

Metode `block` menerima dua argumen opsional. Argumen pertama yang diterima oleh metode `block` adalah durasi penguncian maksimal (dalam detik) yang dapat dilakukan. Tentu saja, jika _request_ telah selesai dieksekusi sebelum durasi maksimal yang ditentukan, maka gembok akan dibuka lebih awal.

Argumen kedua yang diterima oleh metode `block` adalah jumlah detik yang harus ditunggu oleh _request_ ketika ingin melakukan penguncian _session_. Sebuah `Illuminate\Contracts\Cache\LockTimeoutException` akan dilemparkan jika _request_ tersebut tidak berhasik mendapatkan gembok _session_ untuk dirinya ketika masa tunggu telah berakhir.

Jika tidak ada argumen yang dioper, penguncian akan diperoleh selama maksimum 10 detik dan permintaan akan menunggu maksimum 10 detik saat mencoba mendapatkan gembok:

    Route::post('/profil', function () {
        //
    })->block()

<a name="adding-custom-session-drivers"></a>
## Menambahkan _Driver Session_ yang Kustom

<a name="implementing-the-driver"></a>
#### Implementasi _Driver_

Jika pada _driver session_ yang tersedia tidak cocok untuk kebutuhan aplikasi Anda, Laravel memungkinkan Anda untuk menulis penanganan _session_ Anda sendiri. _Driver session_ kustom harus mengimplementasikan _built-in_ `SessionHandlerInterface` milik PHP. _Interface_ tersebut memuat beberapa metode sederhana. Sebuah implementasi MonggoDB akan terlihat seperti berikut:

    <?php

    namespace App\Extensions;

    class MongoSessionHandler implements \SessionHandlerInterface
    {
        public function open($savePath, $sessionName) {}
        public function close() {}
        public function read($sessionId) {}
        public function write($sessionId, $data) {}
        public function destroy($sessionId) {}
        public function gc($lifetime) {}
    }

> **Catatan**  
> Laravel tidak menyertakan direktori untuk menampung Ekstensi Anda. Anda bebas menempatkannya di mana saja yang Anda suka. Pada contoh ini, kita telah membuat sebuah direktori `Extensions` sebagai tempat tinggal `MongoSessionHandler`.

Karena tujuan dari metode-metode di atas tidak mudah dimengerti, mari kita bahas secara singkat, apa yang dilakukan oleh masing-masing metode:

<div class="content-list" markdown="1">

- Metode `open` biasanya digunakan pada sistem penyimpanan _session_ berbasis _file_, Karena Laravel disertai _driver_ `file` untuk _session_, Anda akan sangat jarang mengubah metode ini. Anda dapat. Anda dapat membiarkan metode ini kosong.

- Metode `close` mirip seperti metode `open` yang biasanya dapat diabaikan. Untuk kebanyakan, hal ini tidak diperlukan.
- Metode `read` harus mengembalikan data _session_ dalam bentuk _string_ pada `$sessiomId` yang terkait. Anda tidak perlu melakukan serialisasi atau pengkodean lainnya ketika melakukan _retrieve_ atau penyimpanan data _session_ dalam _driver_ Anda. Laravel akan melakukan serialisasi untuk Anda.
- Metode `write` harus menuliskan string `$data` yang diberikan dengan `$sessionId` yang terkait ke sistem penyimpanan persisten, seperti MongoDB atau sistem penyimpanan lain yang Anda pilih. Sekali lagi, Anda tidak perlu melakukan serialisasi apa pun - Laravel sudah menanganinya untuk Anda. 
- Metode `destroy` harus menghapus data dengan `$sessionId` yang terkait pada penyimpanan persisten.
- Metode `gc` harus "menghancurkan" semua data _session_ yang usianya melebihi `$lifetime` yang ditentukan yang mana adalah _timestamp_ UNIX. Untuk sistem peng-kedaluawarsa-an secara mandiri seperti Memcached dan Redis, metode ini dapat dibiarkan kosong.

</div>

<a name="registering-the-driver"></a>
#### Mendaftarkan _Driver_

Setelah driver Anda diimplementasikan, Anda siap untuk mendaftarkannya ke Laravel. Untuk menambahkan _driver_ tambahan ke _backend session_ milik Laravel, Anda dapat menggunakan metode `extend` yang disediakan oleh [_facade_](/docs/{{version}}/facades) `Session`. Anda harus memanggil metode `extend` dari metode `boot` pada [_service provider_](/docs/{{version}}/providers). Anda dapat mendaftarkannya pada `App\Providers\AppServiceProvider` yang sudah ada atau membuat _provider_ yang baru:

    <?php

    namespace App\Providers;

    use App\Extensions\MongoSessionHandler;
    use Illuminate\Support\Facades\Session;
    use Illuminate\Support\ServiceProvider;

    class SessionServiceProvider extends ServiceProvider
    {
        /**
         * Mendaftarkan layanan pada aplikasi.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Bootstrap layanan yang terdapat pada aplikasi.
         *
         * @return void
         */
        public function boot()
        {
            Session::extend('mongo', function ($app) {
                // Mengembalikan sebuah implementasi dari SessionHandlerInterface...
                return new MongoSessionHandler;
            });
        }
    }

Setelah _driver session_ didaftarkan, Anda dapat menggunakan _driver_ `mongo` pada _file_ konfigurasi `config/session.php`.
