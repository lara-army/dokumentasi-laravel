# Penanganan Eror

- [Pendahuluan](#pendahuluan)
- [Konfigurasi](#konfigurasi)
- [Penanganan _Exeception_](#penanganan-exeception)
    - [Melaporkan _Exception_](#melaporkan-exception)
    - [Tingkatan _Log Exception_](#tingkatan-log-exception)
    - [Mengabaikan _Exception_ Berdasarkan Jenis](#mengabaikan-exception-berdasarkan-jenis)
    - [Menampilkan _Exception_](#menampilkan-exception)
    - [_Exception_ yang Dapat Dilaporkan & Ditampilkan](#exception-yang-dapat-dilaporkan-ditampilkan)
- [HTTP _Exception_](#http-exception)
    - [Halaman Eror Kustom HTTP](#halaman-eror-kustom-http)

<a name="pendahuluan"></a>
## Pendahuluan

Saat Anda memulai proyek Laravel yang baru, penanganan eror dan _exception_ telah terkonfigurasi. Di kelas `App\Exceptions\Handler`, semua _exception_ yang dilemparkan oleh aplikasi Anda telah dicatat dan diteruskan ke pengguna. Jadi, Kita akan membahas lebih dalam mengenai kelas ini di sepanjang halaman dokumentasi ini.

<a name="konfigurasi"></a>
## Konfigurasi

Opsi `debug` pada _file_ konfigurasi `config/app.php` menentukan berapa banyak informasi tentang eror yang akan ditampilkan kepada pengguna. Secara _default_, opsi ini sudah diatur untuk mengikuti nilai variabel _environment_ `APP_DEBUG`, yang disimpan di file `.env` Anda.

Selama pengembangan pada tahap lokal, Anda harus menetapkan variabel _environment_ `APP_DEBUG` menjadi `true`. **Pada _environment production_ Anda, nilai pada variabel `APP_DEBUG` harus selalu `false`. Jika nilai diatur ke `true` dalam _production_, Anda membuat risiko mengekspos nilai konfigurasi yang sensitif kepada pengguna aplikasi Anda.**

<a name="penanganan-exeception"></a>
## Penanganan _Exeception_

<a name="melaporkan-exception"></a>
### Melaporkan _Exception_

Semua _exception_ ditangani oleh kelas `App\Exceptions\Handler`. Kelas ini berisi metode `register` tempat Anda dapat mendaftarkan pelaporan _exception_ khusus dan menampilkan _callback_. Kami akan memeriksa masing-masing konsep ini secara rinci. Laporan _exception_ digunakan untuk mencatat _exception_ atau mengirimkannya ke layanan eksternal, seperti [Flare](https://flareapp.io), [Bugsnag](https://bugsnag.com), atau [Sentry](https://github.com/getsentry/sentry-laravel). Secara _default_, _exception_ akan dicatat berdasarkan konfigurasi [pencatatan _log_](/docs/{{version}}/logging) Anda. Namun, Anda boleh mencatat _exception_ saat mengerjakan proyek Anda.

Misalnya, jika Anda ingin melaporkan jenis _exception_ yang berbeda dengan menggunakan cara yang berbeda, Anda dapat menggunakan metode `reportable` untuk mendaftarkan _closure_ yang harus dijalankan saat _exception_ jenis tertentu perlu dilaporkan. Laravel akan menyimpulkan jenis _exception_ apa yang dilaporkan oleh _closure_ dengan memeriksa tipe-petunjuk dari _closure_:

    use App\Exceptions\InvalidOrderException;

    /**
     * Mendaftarkan callback untuk penanganan exception pada aplikasi.
     *
     * @return void
     */
    public function register()
    {
        $this->reportable(function (InvalidOrderException $e) {
            //
        });
    }

Saat Anda mendaftarkan _callback_ untuk pelaporan _exception_ yang kustom menggunakan metode `reportable`, Laravel selalu mencatat _exception_ menggunakan konfigurasi pencatatan _log_ yang _default_. Jika Anda ingin berhenti menyebarkan _exception_ ke tumpukan _logging_ _default_, Anda dapat menggunakan metode `stop` saat mendefinisikan _callback_ untuk pelaporan, atau mengembalikan `false` dari _callback_ tersebut:

    $this->reportable(function (InvalidOrderException $e) {
        //
    })->stop();

    $this->reportable(function (InvalidOrderException $e) {
        return false;
    });

> **Catatan**  
> Untuk menyesuaikan pelaporan _exception_ pada _exception_ tertentu, Anda juga dapat menggunakan [_reportable exception_](/docs/{{version}}/errors#renderable-exceptions).

<a name="global-log-context"></a>
#### Konteks _Log_ Global

Jika tersedia, Laravel akan secara otomatis menambahkan ID pengguna saat ini ke setiap pesan _log_ _exception_ sebagai data kontekstual. Anda dapat menentukan data kontekstual global Anda sendiri dengan mengganti metode `context` dari kelas `App\Exceptions\Handler` pada aplikasi Anda. Informasi ini akan disertakan dalam setiap pesan _log_ _exception_ yang ditulis oleh aplikasi Anda:

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

<a name="konteks-log-exception"></a>
#### Konteks _Log Exception_

Meskipun menambahkan konteks ke setiap pesan _log_ dapat membantu, terkadang beberapa _exception_ dapat memiliki konteks unik yang ingin Anda sertakan dalam _log_. Dengan mendefinisikan metode `context` pada pada salah satu _exception_ kustom milik Anda, Anda dapat menentukan data apa saja yang relevan untuk _exception_ tersebut dan akan ditambahkan ke entri _log_ _exception_:

    <?php

    namespace App\Exceptions;

    use Exception;

    class InvalidOrderException extends Exception
    {
        // ...

        /**
         * Mengambil informasi konteks exception.
         *
         * @return array
         */
        public function context()
        {
            return ['order_id' => $this->orderId];
        }
    }

<a name="report-pembantu"></a>
#### `report` Pembantu

Terkadang Anda mungkin perlu melaporkan _exception_ tetapi terus menangani permintaan saat ini. Fungsi helper `report` memungkinkan Anda melaporkan _exception_ dengan cepat melalui pengendali _exception_ tanpa menampilkan halaman eror ke pengguna:

    public function isValid($value)
    {
        try {
            // Validasi nilainya...
        } catch (Throwable $e) {
            report($e);

            return false;
        }
    }

<a name="tingkatan-log-exception"></a>
### Tingkatan _Log Exception_

Ketika pesan ditulis ke [_log_](/docs/{{version}}/logging) milik aplikasi Anda, pesan ditulis pada [tingkatan _log_](/docs/{{version}}/logging#log-levels) tertentu, yang menunjukkan tingkat keseriusan atau pentingnya pesan yang sedang dicatat dalam _log_.

Seperti yang disebutkan di atas, bahkan saat mendaftarkan _callback_ pelaporan _exception_ yang kustom menggunakan metode `reportable`, Laravel akan tetap mencatat _exception_ menggunakan konfigurasi _logging_ _default_ untuk aplikasi. Namun, karena level _log_ terkadang dapat memengaruhi kanal tempat pesan dicatat, Anda mungkin ingin mengonfigurasi level _log_ di mana _exception_ tertentu akan dicatat.

Untuk melakukannya, Anda dapat menentukan _array_ dengan tipe _exception_ dan tingkat _log_ yang terkait di dalam properti `$levels` dari pengendali _exception_ aplikasi Anda :

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

<a name="mengabaikan-exception-berdasarkan-jenis"></a>
### Mengabaikan _Exception_ Berdasarkan Jenis

Saat membuat aplikasi Anda, akan ada beberapa jenis _exception_ yang ingin Anda abaikan dan tidak pernah laporkan. Pengendali _exception_ aplikasi Anda berisi properti `$dontReport` yang diinisialisasi ke _array_ kosong. Setiap kelas yang Anda tambahkan ke properti ini tidak akan dilaporkan. Namun, mereka mungkin masih memiliki logika _rendering_ yang kustom :

    use App\Exceptions\InvalidOrderException;

    /**
     * Daftar tipe exception yang tidak dilaporkan.
     *
     * @var array<int, class-string<\Throwable>>
     */
    protected $dontReport = [
        InvalidOrderException::class,
    ];

> **Catatan**  
> Di belakang layar, Laravel sudah mengabaikan beberapa jenis eror untuk Anda, seperti _exception_ yang dihasilkan dari eror 404 HTTP "not found" atau 419 respons HTTP yang dihasilkan oleh _token CSRF_ yang tidak valid.

<a name="menampilkan-exception"></a>
### Menampilkan _Exception_

Secara _default_, penanganan _exception_ Laravel akan mengonversi _exception_ menjadi respons HTTP untuk Anda. Namun, Anda bebas mendaftarkan _closure_ pe-_render_-an khusus untuk _exception_ dari jenis tertentu. Anda dapat melakukannya melalui metode `renderable` pada penanganan _exception_ Anda.

_Closure_ yang diteruskan ke metode `renderable` harus mengembalikan _instance_ `Illuminate\Http\Response`, yang dapat dihasilkan menggunakan bantuan `response`. Laravel akan menyimpulkan jenis _exception_ apa yang dibuat oleh _closure_ dengan memeriksa _type-hint_ dari _closure_ :

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

Kamu juga dapat menggunakan metode `renderable` untuk mengganti perilaku _rendering_ untuk _exception_ Laravel atau Symfony bawaan seperti `NotFoundHttpException`. Jika _closure_ yang diberikan ke metode `renderable` tidak mengembalikan nilai, pe-_render_-an _exception_ yang _default_ milik Laravel akan digunakan:

    use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

    /**
     * Mendaftarkan callback penanganan exception untuk aplikasi.
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

<a name="exception-yang-dapat-dilaporkan-ditampilkan"></a>
### _Exception_ yang Dapat Dilaporkan & Ditampilkan

Alih-alih memeriksa jenis _exception_ dalam metode `register` milik penangan _exception_, Anda dapat mendefinisikan metode `report` dan `render` langsung pada _exception_ kustom Anda. Ketika metode ini didefinisikan, mereka akan dipanggil secara otomatis oleh _framework_ :

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

Jika _exception_ Anda melakukan _extend_ dari _exception_ yang sudah bisa di-_render_, seperti pada bawaan Laravel atau _exception Symfony_, Anda dapat mengembalikan `false` dari metode `render` _exception_ untuk menampilkan respons HTTP yang _default_ untuk _exception_:

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

Jika _exception_ Anda berisi logika pelaporan kustom yang hanya diperlukan ketika kondisi tertentu terpenuhi, Anda mungkin perlu menginstruksikan Laravel untuk jarang melaporkan _exception_ tersebut menggunakan konfigurasi penanganan _exception_ _default_. Untuk mencapai hal tersebut, Anda dapat mengembalikan nilai `false` dari metode `report` _exception_:

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

> **Catatan**  
> Kamu dapat melakukan _type-hint_ dependensi pada metode `report`. _type-hint_ tersebut akan secara otomatis disuntikkan ke dalam metode oleh [_service container_](/docs/{{version}}/container) milik Laravel.

<a name="http-exception"></a>
## HTTP _Exception_

Beberapa _exception_ mendeskripsikan kode eror HTTP dari server. Misalnya, ini mungkin eror "page not found" error (404), "unauthorized error" (401), atau bahkan eror 500 yang dihasilkan pengembang. Untuk menghasilkan respon seperti itu, Anda dapat menggunakan bantuan `abort`:

    abort(404);

<a name="halaman-eror-kustom-http"></a>
### Halaman Eror Kustom HTTP

Laravel mempermudah Anda untuk menampilkan halaman eror kustom untuk berbagai kode status HTTP yang berbeda. Misalnya, jika Anda ingin menyesuaikan halaman eror untuk kode status HTTP 404, Anda dapat membuat _template_ tampilan pada `resources/views/errors/404.blade.php`. Tampilan ini akan ditampilkan pada semua eror 404 yang dihasilkan oleh aplikasi Anda. Tampilan dalam direktori ini harus diberi nama yang sama dengan kode status HTTP yang dikembalikan. _Instance_ dari `Symfony\Component\HttpKernel\Exception\HttpException` yang dipicu oleh fungsi `abort` akan diteruskan ke tampilan sebagai variabel `$exception` :

    <h2>{{ $exception->getMessage() }}</h2>

Kamu dapat mempublikasikan _template_ halaman eror _default_ milik Laravel menggunakan perintah Artisan `vendor:publish`. Setelah _file template_ dipublikasikan, Anda dapat mengubahnya sesuai dengan keinginan Anda:

```shell
php artisan vendor:publish --tag=laravel-errors
```

<a name="halaman-eror-fallback-http"></a>
#### Halaman Eror _Fallback_ HTTP

Anda juga dapat menentukan halaman eror "fallback" untuk sekumpulan kode status pada HTTP tertentu. Halaman ini akan ditampilkan jika tidak ada halaman yang sesuai untuk kode status HTTP yang ditentukan. Untuk melakukannya, Anda dapat membuat _file template_ `4xx.blade.php` dan _template_ `5xx.blade.php` di direktori `resources/views/errors` aplikasi Anda.
