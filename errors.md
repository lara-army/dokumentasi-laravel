# Penanganan Eror

- [Penanganan Eror](#penanganan-eror)
  - [Pendahuluan](#pendahuluan)
  - [Konfigurasi](#konfigurasi)
  - [Penanganan Exeception](#penanganan-exeception)
    - [Melaporkan Exception](#melaporkan-exception)
      - [Konteks Log Exception](#konteks-log-exception)
      - [`report` Pembantu](#report-pembantu)
    - [Level Log Exception](#level-log-exception)
    - [Mengabaikan Exception Berdasarkan Tipe](#mengabaikan-exception-berdasarkan-tipe)
    - [Menampilkan Exception](#menampilkan-exception)
    - [Exception yang Dapat Dilaporkan & Ditampilkan](#exception-yang-dapat-dilaporkan-ditampilkan)
  - [HTTP Exception](#http-exception)
    - [Halaman Eror Kustom HTTP](#halaman-eror-kustom-http)
      - [Halaman Eror Fallback HTTP](#halaman-eror-fallback-http)

<a name="pendahuluan"></a>
## Pendahuluan

Saat Anda memulai proyek Laravel baru, penanganan eror dan _exception_ sudah dikonfigurasi. Di kelas `App\Exceptions\Handler`, semua _exception_ yang dilemparkan oleh aplikasi Anda dicatat dan diteruskan ke pengguna. Jadi, kita akan mendalami kelas lebih dalam pada dokumentasi ini.

<a name="konfigurasi"></a>
## Konfigurasi

Opsi `debug` di file konfigurasi `config/app.php` Anda menentukan berapa banyak informasi tentang eror yang akan ditampilkan kepada pengguna. Secara _default_, pengaturan ini sudah diatur untuk mengikuti nilai variabel environment `APP_DEBUG`, yang disimpan di file `.env` Anda.

Selama pengembangan pada tahap lokal, Anda harus menetapkan variabel environment `APP_DEBUG` menjadi `true`. **Pada environment production Anda, nilai pada variabel `APP_DEBUG` harus selalu `false`. Jika nilai diatur ke `true` dalam production, ada risiko bahwa nilai tersebut akan diekspos dalam setelan sensitif kepada pengguna aplikasi Anda.**

<a name="penanganan-exeception"></a>
## Penanganan Exeception

<a name="melaporkan-exception"></a>
### Melaporkan Exception

Semua _exception_ ditangani oleh kelas `App\Exceptions\Handler`. Kelas ini berisi metode `register` tempat Anda dapat mendaftarkan pelaporan _exception_ khusus dan menampilkan _callback_. Kami akan memeriksa masing-masing konsep ini secara rinci. Laporan _exception_ digunakan untuk mencatat _exception_ atau mengirimkannya ke layanan eksternal, seperti [Flare](https://flareapp.io), [Bugsnag](https://bugsnag.com), atau [Sentry](https://github.com/getsentry/sentry-laravel). Secara _default_, _exception_ akan dicatat berdasarkan konfigurasi [logging](/docs/{{version}}/logging) Anda. Namun, Anda dapat menyimpan _exception_ saat mengerjakan proyek Anda.

Misalnya, jika Anda ingin melaporkan jenis _exception_ yang berbeda dengan menggunakan cara yang berbeda, Anda dapat menggunakan metode `reportable` untuk mendaftarkan _closure_ yang harus dijalankan saat _exception_ jenis tertentu perlu dilaporkan. Laravel akan menyimpulkan jenis _exception_ apa yang dilaporkan oleh _closure_ dengan memeriksa tipe-petunjuk dari _closure_:

```php
    use App\Exceptions\InvalidOrderException;

    /**
     * Daftarkan callback penanganan exception untuk aplikasi.
     *
     * @return void
     */
    public function register()
    {
        $this->reportable(function (InvalidOrderException $e) {
            //
        });
    }
```

Saat Anda mendaftarkan _callback_ pelaporan _exception_ kustom menggunakan metode `reportable`, Laravel selalu mencatat _exception_ menggunakan konfigurasi pembuatan _log_ _default_ aplikasi. Jika Anda ingin berhenti menyebarkan _exception_ ke tumpukan _logging_ _default_, Anda dapat menggunakan metode `stop` saat menentukan _callback_ laporan, atau mengembalikan `false` dari _callback_:

```php
    $this->reportable(function (InvalidOrderException $e) {
        //
    })->stop();

    $this->reportable(function (InvalidOrderException $e) {
        return false;
    });
```

> **Catatan**  
> Untuk menyesuaikan pelaporan _exception_ untuk _exception_ yang diberikan, Anda juga dapat menggunakan [reportable exceptions](/docs/{{version}}/errors#renderable-exceptions).

<a name="global-log-context"></a>
#### Konteks Log Global

Jika tersedia, Laravel akan secara otomatis menambahkan ID pengguna saat ini ke setiap pesan _log_ _exception_ sebagai data kontekstual. Anda dapat menentukan data kontekstual global Anda sendiri dengan mengganti metode `context` dari kelas `App\Exceptions\Handler` aplikasi Anda. Informasi ini akan disertakan dalam setiap pesan _log_ _exception_ yang ditulis oleh aplikasi Anda:

```php
    /**
     * Dapatkan variabel konteks default untuk pencatatan.
     *
     * @return array
     */
    protected function context()
    {
        return array_merge(parent::context(), [
            'foo' => 'bar',
        ]);
    }
```

<a name="konteks-log-exception"></a>
#### Konteks Log Exception

Meskipun menambahkan konteks ke setiap pesan _log_ dapat membantu, terkadang beberapa _exception_ dapat memiliki konteks unik yang ingin Anda sertakan dalam _log_. Dengan menentukan metode `context` pada salah satu _exception_ khusus aplikasi, Anda dapat menentukan bahwa semua data yang relevan untuk _exception_ tersebut akan ditambahkan ke entri _log_ _exception_:

```php
    <?php

    namespace App\Exceptions;

    use Exception;

    class InvalidOrderException extends Exception
    {
        // ...

        /**
         * Dapatkan informasi konteks exception.
         *
         * @return array
         */
        public function context()
        {
            return ['order_id' => $this->orderId];
        }
    }
```

<a name="report-pembantu"></a>
#### `report` Pembantu

Terkadang Anda mungkin perlu melaporkan _exception_ tetapi terus menangani permintaan saat ini. Fungsi helper `report` memungkinkan Anda melaporkan _exception_ dengan cepat melalui pengendali _exception_ tanpa menampilkan halaman eror ke pengguna:

```php
    public function isValid($value)
    {
        try {
            // Validasi nilainya...
        } catch (Throwable $e) {
            report($e);

            return false;
        }
    }
```

<a name="level-log-exception"></a>
### Level Log Exception

Ketika pesan yang ditulis ke aplikasi Anda [logs](/docs/{{version}}/logging), pesan ditulis pada [tingkat log](/docs/{{version}}/logging#log-levels) tertentu, yang menunjukkan tingkat keseriusan atau pentingnya pesan yang sedang dicatat dalam log.

Seperti yang disebutkan di atas, bahkan saat mencatat _callback_ pelaporan _exception_ kustom menggunakan metode `reportable`, Laravel akan tetap mencatat _exception_ menggunakan konfigurasi _logging_ _default_ untuk aplikasi. Namun, karena level log terkadang dapat memengaruhi saluran tempat pesan dicatat, Anda mungkin ingin mengonfigurasi level log tempat _exception_ tertentu dicatat.

Untuk melakukannya, Anda dapat menentukan array dengan tipe _exception_ dan level log yang terkait di dalam properti `$levels` dari pengendali _exception_ aplikasi Anda :

```php
    use PDOException;
    use Psr\Log\LogLevel;

    /**
     * Daftar jenis exception dengan tingkat log kustom yang sesuai.
     *
     * @var array<class-string<\Throwable>, \Psr\Log\LogLevel::*>
     */
    protected $levels = [
        PDOException::class => LogLevel::CRITICAL,
    ];
```

<a name="mengabaikan-exception-berdasarkan-tipe"></a>
### Mengabaikan Exception Berdasarkan Tipe

Saat membuat aplikasi Anda, akan ada beberapa jenis _exception_ yang ingin Anda abaikan dan tidak pernah laporkan. Pengendali _exception_ aplikasi Anda berisi properti `$dontReport` yang diinisialisasi ke _array_ kosong. Setiap kelas yang Anda tambahkan ke properti ini tidak akan dilaporkan. Namun, mereka mungkin masih memiliki logika pada rendering kustom :

```php
    use App\Exceptions\InvalidOrderException;

    /**
     * Daftar tipe exception yang tidak dilaporkan.
     *
     * @var array<int, class-string<\Throwable>>
     */
    protected $dontReport = [
        InvalidOrderException::class,
    ];
```

> **Catatan**  
> Di belakang layar, Laravel sudah mengabaikan beberapa jenis eror untuk Anda, seperti _exception_ yang dihasilkan dari eror 404 HTTP "not found" atau 419 respons HTTP yang dihasilkan oleh _token CSRF_ yang tidak valid.

<a name="menampilkan-exception"></a>
### Menampilkan Exception

Secara _default_, penanganan _exception_ Laravel akan mengonversi _exception_ menjadi respons HTTP untuk Anda. Namun, Anda bebas mendaftarkan _closure_ perenderan khusus untuk _exception_ dari jenis tertentu. Anda dapat melakukannya melalui metode `renderable` dari penanganan _exception_ Anda.

_Closure_ yang diteruskan ke metode `renderable` harus mengembalikan instance `Illuminate\Http\Response`, yang dapat dihasilkan menggunakan bantuan `response`. Laravel akan menyimpulkan jenis _exception_ apa yang dibuat oleh _closure_ dengan memeriksa tipe-hint dari _closure_ :

```php
    use App\Exceptions\InvalidOrderException;

    /**
     * Daftarkan callback penanganan exception untuk aplikasi.
     *
     * @return void
     */
    public function register()
    {
        $this->renderable(function (InvalidOrderException $e, $request) {
            return response()->view('errors.invalid-order', [], 500);
        });
    }
```

Kamu juga dapat menggunakan metode `renderable` untuk mengganti perilaku _rendering_ untuk _exception_ Laravel atau Symfony bawaan seperti `NotFoundHttpException`. Jika _closure_ yang diberikan ke metode `renderable` tidak mengembalikan nilai, perenderan _exception_ _default_ Laravel akan digunakan:

```php
    use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

    /**
     * Daftarkan callback penanganan exception untuk aplikasi.
     *
     * @return void
     */
    public function register()
    {
        $this->renderable(function (NotFoundHttpException $e, $request) {
            if ($request->is('api/*')) {
                return response()->json([
                    'message' => 'Record not found.'
                ], 404);
            }
        });
    }
```

<a name="exception-yang-dapat-dilaporkan-ditampilkan"></a>
### Exception yang Dapat Dilaporkan & Ditampilkan

Alih-alih memeriksa _exception_ jenis dalam metode `register` penangan _exception_, Anda dapat menentukan metode `report` dan `render` langsung pada _exception_ kustom Anda. Ketika metode ini ada, mereka akan dipanggil secara otomatis oleh _framework_ :

```php
    <?php

    namespace App\Exceptions;

    use Exception;

    class InvalidOrderException extends Exception
    {
        /**
         * Laporkan exception.
         *
         * @return bool|null
         */
        public function report()
        {
            //
        }

        /**
         * Render exception menjadi respons HTTP.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function render($request)
        {
            return response(/* ... */);
        }
    }
```

Jika _exception_ Anda memperluas _exception_ yang sudah dapat dirender, seperti Laravel bawaan atau _exception Symfony_, Anda dapat mengembalikan `false` dari metode `render` _exception_ untuk menampilkan respons HTTP _default_ _exception_:

```php
    /**
     * Render exception menjadi respons HTTP.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function render($request)
    {
        // Tentukan apakah exception membutuhkan rendering khusus...

        return false;
    }
```

Jika _exception_ Anda berisi logika pelaporan kustom yang hanya diperlukan ketika kondisi tertentu terpenuhi, Anda mungkin perlu menginstruksikan Laravel untuk terkadang melaporkan _exception_ tersebut menggunakan konfigurasi penanganan _exception_ _default_. Untuk menyelesaikannya, Anda dapat mengembalikan nilai `false` dari metode `report` _exception_:

```php
    /**
     * Laporkan exception.
     *
     * @return bool|null
     */
    public function report()
    {
        // Tentukan apakah exception membutuhkan pelaporan khusus...

        return false;
    }
```

> **Catatan**  
> Kamu dapat menulis-petunjuk dependensi yang diperlukan dari metode dan mereka akan secara otomatis disuntikkan ke dalam metode oleh Laravel [service container](/docs/{{version}}/container).

<a name="http-exception"></a>
## HTTP Exception

Beberapa _exception_ mendeskripsikan kode eror HTTP dari server. Misalnya, ini mungkin erir "page not found" error (404), "unauthorized error" (401), atau bahkan eror 500 yang dihasilkan pengembang. Untuk menghasilkan respon seperti itu, Anda dapat menggunakan bantuan `abort`:

```php
    abort(404);
```

<a name="halaman-eror-kustom-http"></a>
### Halaman Eror Kustom HTTP

Laravel memudahkan untuk menampilkan halaman eror kustom untuk berbagai kode status HTTP yang berbeda. Misalnya, jika Anda ingin menyesuaikan halaman eror untuk kode status HTTP 404, buat template tampilan `resources/views/errors/404.blade.php`. Tampilan ini akan ditampilkan pada semua eror 404 yang dihasilkan oleh aplikasi Anda. Tampilan dalam direktori ini harus diberi nama agar cocok dengan kode status HTTP yang sesuai. Instance `Symfony\Component\HttpKernel\Exception\HttpException` yang dipicu oleh fungsi `abort` akan diteruskan ke tampilan sebagai variabel `$exception` :

```php
    <h2>{{ $exception->getMessage() }}</h2>
```

Kamu dapat mempublikasikan template halaman eror _default_ Laravel menggunakan perintah Artisan `vendor:publish`. Setelah templat dipublikasikan, Anda dapat menyesuaikannya sesuai keinginan Anda:

```shell
php artisan vendor:publish --tag=laravel-errors
```

<a name="halaman-eror-fallback-http"></a>
#### Halaman Eror Fallback HTTP

Anda juga dapat menentukan halaman eror "fallback" untuk sekumpulan kode status pada HTTP tertentu. Halaman ini akan ditampilkan jika tidak ada halaman yang sesuai untuk kode status HTTP yang ditentukan. Untuk melakukannya, tentukan template `4xx.blade.php` dan template `5xx.blade.php` di direktori `resources/views/errors` aplikasi Anda.
