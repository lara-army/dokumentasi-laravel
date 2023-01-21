# Penanganan Eror

- [Penanganan Eror](#penanganan-eror)
  - [Pendahuluan](#pendahuluan)
  - [Konfigurasi](#konfigurasi)
  - [Penanganan Exeception](#penanganan-exeception)
    - [Melaporkan Exception](#melaporkan-exception)
      - [Konteks Log Global](#konteks-log-global)
      - [Konteks Log Exception](#konteks-log-exception)
      - [`report` Pembantu](#report-pembantu)
    - [Level Log Exception](#level-log-exception)
    - [Mengabaikan Exception Berdasarkan Tipe](#mengabaikan-exception-berdasarkan-tipe)
    - [Merender Exception](#merender-exception)
    - [Exception yang Dapat Dilaporkan \& Dirender](#exception-yang-dapat-dilaporkan--dirender)
  - [HTTP Exception](#http-exception)
    - [Halaman Eror Kustom HTTP](#halaman-eror-kustom-http)
      - [Halaman Eror Fallback HTTP](#halaman-eror-fallback-http)

<a name="introduction"></a>
## Pendahuluan

Saat Anda memulai proyek Laravel baru, penanganan eror dan pengecualian sudah dikonfigurasi. Di kelas `App\Exceptions\Handler`, semua exception yang dilemparkan oleh aplikasi Anda dicatat dan diteruskan ke pengguna. Jadi, kita akan mendalami kelas lebih dalam pada dokumentasi ini.

<a name="configuration"></a>
## Konfigurasi

Opsi `debug` di file konfigurasi `config/app.php` Anda menentukan berapa banyak informasi tentang eror yang akan ditampilkan kepada pengguna. Secara default, pengaturan ini sudah diatur untuk mengikuti nilai variabel environment `APP_DEBUG`, yang disimpan di file `.env` Anda.

Selama pengembangan pada tahap lokal, Anda harus menetapkan variabel environment `APP_DEBUG` ubah ke `true`. **Pada environment production Anda, nilai pada variabel `APP_DEBUG` harus selalu `false`. Jika nilai diatur ke `true` dalam production, ada risiko bahwa nilai tersebut akan diekspos dalam setelan sensitif kepada pengguna aplikasi Anda.**

<a name="the-exception-handler"></a>
## Penanganan Exeception

<a name="reporting-exceptions"></a>
### Melaporkan Exception

Semua exception ditangani oleh kelas `App\Exceptions\Handler`. Kelas ini berisi metode `register` yang dimana tempat Anda dapat mendaftarkan pelaporan exception khusus dan merender callback. Kami akan memeriksa masing-masing konsep ini secara rinci. Laporan exception digunakan untuk mencatat exception atau mengirimkannya ke layanan eksternal, seperti [Flare](https://flareapp.io), [Bugsnag](https://bugsnag.com), atau [Sentry](https://github.com/getsentry/sentry-laravel). Secara default, expcetion akan dicatat berdasarkan konfigurasi [logging](/docs/{{version}}/logging) Anda. Namun, Anda dapat menyimpan pengecualian saat mengerjakan proyek Anda.

Misalnya, jika Anda ingin melaporkan jenis exception yang berbeda dengan cara yang berbeda, Anda dapat menggunakan metode `reportable` untuk mendaftarkan penutupan yang harus dijalankan saat exception jenis tertentu perlu dilaporkan. Laravel akan menyimpulkan jenis exception apa yang dilaporkan oleh closure dengan memeriksa tipe-petunjuk dari closure:

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

Saat Anda mendaftarkan callback pelaporan exception kustom menggunakan metode `reportable`, Laravel selalu mencatat exception menggunakan konfigurasi pembuatan log default aplikasi. Jika Anda ingin berhenti menyebarkan exception ke tumpukan logging default, Anda dapat menggunakan metode `stop` saat menentukan callback laporan, atau mengembalikan `false` dari callback:

```php
    $this->reportable(function (InvalidOrderException $e) {
        //
    })->stop();

    $this->reportable(function (InvalidOrderException $e) {
        return false;
    });
```

> **Catatan**  
> Untuk menyesuaikan pelaporan exception untuk exception yang diberikan, Anda juga dapat menggunakan [reportable exceptions](/docs/{{version}}/errors#renderable-exceptions).

<a name="global-log-context"></a>
#### Konteks Log Global

Jika tersedia, Laravel akan secara otomatis menambahkan ID pengguna saat ini ke setiap pesan log exception sebagai data kontekstual. Anda dapat menentukan data kontekstual global Anda sendiri dengan mengganti metode `context` dari kelas `App\Exceptions\Handler` aplikasi Anda. Informasi ini akan disertakan dalam setiap pesan log exception yang ditulis oleh aplikasi Anda:

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

<a name="exception-log-context"></a>
#### Konteks Log Exception

Meskipun menambahkan konteks ke setiap pesan log dapat membantu, terkadang beberapa exception dapat memiliki konteks unik yang ingin Anda sertakan dalam log. Dengan menentukan metode `context` pada salah satu exception khusus aplikasi, Anda dapat menentukan bahwa semua data yang relevan untuk exception tersebut akan ditambahkan ke entri log pengecualian:

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

<a name="the-report-helper"></a>
#### `report` Pembantu

Terkadang Anda mungkin perlu melaporkan exception tetapi terus menangani permintaan saat ini. Fungsi helper `report` memungkinkan Anda melaporkan exception dengan cepat melalui pengendali exception tanpa merender halaman eror ke pengguna:

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

<a name="exception-log-levels"></a>
### Level Log Exception

Ketika pesan yang ditulis ke aplikasi Anda [logs](/docs/{{version}}/logging), pesan ditulis pada [tingkat log](/docs/{{version}}/logging#log-levels) tertentu, yang menunjukkan tingkat keseriusan atau pentingnya pesan yang sedang dicatat dalam log.

Seperti yang disebutkan di atas, bahkan saat mencatat callback pelaporan exception kustom menggunakan metode `reportable`, Laravel akan tetap mencatat exception menggunakan konfigurasi logging default untuk aplikasi. Namun, karena level log terkadang dapat memengaruhi saluran tempat pesan dicatat, Anda mungkin ingin mengonfigurasi level log tempat exception tertentu dicatat.

Untuk melakukannya, Anda dapat menentukan array dengan tipe exception dan level log yang terkait di dalam properti `$levels` dari pengendali exception aplikasi Anda :

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

<a name="ignoring-exceptions-by-type"></a>
### Mengabaikan Exception Berdasarkan Tipe

Saat membuat aplikasi Anda, akan ada beberapa jenis exception yang ingin Anda abaikan dan tidak pernah laporkan. Pengendali exception aplikasi Anda berisi properti `$dontReport` yang diinisialisasi ke array kosong. Setiap kelas yang Anda tambahkan ke properti ini tidak akan dilaporkan. Namun, mereka mungkin masih memiliki logika pada rendering kustom :

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
> Di belakang layar, Laravel sudah mengabaikan beberapa jenis eror untuk Anda, seperti exception yang dihasilkan dari eror 404 HTTP "not found" atau 419 respons HTTP yang dihasilkan oleh token CSRF yang tidak valid.

<a name="rendering-exceptions"></a>
### Merender Exception

Secara default, penanganan exception Laravel akan mengonversi exception menjadi respons HTTP untuk Anda. Namun, Anda bebas mendaftarkan closure perenderan khusus untuk exception dari jenis tertentu. Anda dapat melakukannya melalui metode `renderable` dari penanganan exception Anda.

Closure yang diteruskan ke metode `renderable` harus mengembalikan instance `Illuminate\Http\Response`, yang dapat dihasilkan menggunakan bantuan `response`. Laravel akan menyimpulkan jenis exception apa yang dibuat oleh closure dengan memeriksa tipe-hint dari closure :

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

Kamu juga dapat menggunakan metode `renderable` untuk mengganti perilaku rendering untuk exception Laravel atau Symfony bawaan seperti `NotFoundHttpException`. Jika closure yang diberikan ke metode `renderable` tidak mengembalikan nilai, perenderan exception default Laravel akan digunakan :

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

<a name="renderable-exceptions"></a>
### Exception yang Dapat Dilaporkan & Dirender

Alih-alih memeriksa exception jenis dalam metode `register` penangan exception, Anda dapat menentukan metode `report` dan `render` langsung pada exception kustom Anda. Ketika metode ini ada, mereka akan dipanggil secara otomatis oleh framework :

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

Jika exception Anda memperluas exception yang sudah dapat dirender, seperti Laravel bawaan atau exception Symfony, Anda dapat mengembalikan `false` dari metode `render` exception untuk merender respons HTTP default exception :

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

Jika exception Anda berisi logika pelaporan kustom yang hanya diperlukan ketika kondisi tertentu terpenuhi, Anda mungkin perlu menginstruksikan Laravel untuk terkadang melaporkan exception tersebut menggunakan konfigurasi penanganan exception default. Untuk menyelesaikannya, Anda dapat mengembalikan nilai `false` dari metode `report` exception :

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
> Kamu dapat menulis-petunjuk dependensi yang diperlukan dari metode dan mereka akan secara otomatis disuntikkan ke dalam metode oleh Laravel [wadah layanan](/docs/{{version}}/container).

<a name="http-exceptions"></a>
## HTTP Exception

Beberapa exception mendeskripsikan kode eror HTTP dari server. Misalnya, ini mungkin erir "page not found" error (404), "unauthorized error" (401), atau bahkan eror 500 yang dihasilkan pengembang. Untuk menghasilkan respon seperti itu, Anda dapat menggunakan bantuan `abort`:

```php
    abort(404);
```

<a name="custom-http-error-pages"></a>
### Halaman Eror Kustom HTTP

Laravel memudahkan untuk menampilkan halaman eror kustom untuk berbagai kode status HTTP yang berbeda. Misalnya, jika Anda ingin menyesuaikan halaman eror untuk kode status HTTP 404, buat template tampilan `resources/views/errors/404.blade.php`. Tampilan ini akan ditampilkan pada semua eror 404 yang dihasilkan oleh aplikasi Anda. Tampilan dalam direktori ini harus diberi nama agar cocok dengan kode status HTTP yang sesuai. Instance `Symfony\Component\HttpKernel\Exception\HttpException` yang dipicu oleh fungsi `abort` akan diteruskan ke tampilan sebagai variabel `$exception` :

```php
    <h2>{{ $exception->getMessage() }}</h2>
```

Kamu dapat mempublikasikan template halaman eror default Laravel menggunakan perintah Artisan `vendor:publish`. Setelah templat dipublikasikan, Anda dapat menyesuaikannya sesuai keinginan Anda:

```shell
php artisan vendor:publish --tag=laravel-errors
```

<a name="fallback-http-error-pages"></a>
#### Halaman Eror Fallback HTTP

Kamu juga dapat menentukan halaman eror "fallback" untuk sekumpulan kode status pada HTTP tertentu. Halaman ini akan ditampilkan jika tidak ada halaman yang sesuai untuk kode status HTTP yang ditentukan. Untuk melakukannya, tentukan template `4xx.blade.php` dan template `5xx.blade.php` di direktori `resources/views/errors` aplikasi Anda.