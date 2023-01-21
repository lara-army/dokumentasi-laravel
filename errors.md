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

Ketika kamu memulai proyek Laravel baru, eror dan penanganan exception sudah dikonfigurasi. Pada kelas `App\Exceptions\Handler` adalah tempat semua exception yang dilemparkan oleh aplikasi kamu akan dicatat dan kemudian dirender ke pengguna. Jadi, kita akan mendalami kelas lebih dalam pada dokumentasi ini.

<a name="configuration"></a>
## Konfigurasi

Opsi `debug` dalam file konfigurasi `config/app.php` kamu menentukan berapa banyak informasi tentang kesalahan yang akan ditampilkan kepada pengguna. Secara default, opsi ini sudah diatur untuk mengikuti nilai variabel environment `APP_DEBUG`, yang disimpan di file `.env` kamu.

Selama pengembangan pada tahap lokal, kamu harus menetapkan variabel environment `APP_DEBUG` ubah ke `true`. **Pada environment production kamu, nilai pada variabel `APP_DEBUG` harus selalu `false`. Jika nilai diatur ke `true` dalam production, kamu akan memiliki risiko yaitu mengekspos nilai pada konfigurasi yang sensitif ke pengguna aplikasi kamu.**

<a name="the-exception-handler"></a>
## Penanganan Exeception

<a name="reporting-exceptions"></a>
### Melaporkan Exception

Semua exception ditangani oleh kelas `App\Exceptions\Handler`. Kelas ini berisi metode `register` yang dimana tempat kamu dapat mendaftarkan pelaporan exception khusus dan merender callback. Kami akan memeriksa setiap konsep ini secara rinci. Pelaporan exception digunakan untuk membuat log expcetion atau mengirimnya ke layanan eksternal seperti pada [Flare](https://flareapp.io), [Bugsnag](https://bugsnag.com), atau [Sentry](https://github.com/getsentry/sentry-laravel). Secara default, expcetion akan dicatat berdasarkan konfigurasi [logging](/docs/{{version}}/logging) kamu. Namun, kamu bebas untuk mencatat expcetion sesuai kebutuhan pada project yang dikerjakan.

Misalnya, jika kamu perlu melaporkan jenis exception yang berbeda dengan cara yang berbeda, kamu dapat menggunakan metode `reportable` untuk mendaftarkan penutupan yang harus dijalankan saat exception jenis tertentu perlu dilaporkan. Laravel akan menyimpulkan jenis exception apa yang dilaporkan oleh closure dengan memeriksa tipe-petunjuk dari closure :

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

Saat kamu mendaftarkan callback pelaporan exception kustom menggunakan metode `reportable`, Laravel akan tetap mencatat exception menggunakan konfigurasi logging default untuk aplikasi. Jika kamu ingin menghentikan penyebaran exception ke tumpukan logging default, kamu dapat menggunakan metode `stop` saat menentukan callback pelaporan atau mengembalikan `false` dari callback :

```php
    $this->reportable(function (InvalidOrderException $e) {
        //
    })->stop();

    $this->reportable(function (InvalidOrderException $e) {
        return false;
    });
```

> **Catatan**  
> Untuk menyesuaikan pelaporan exception untuk exception yang diberikan, kamu juga dapat menggunakan [reportable exceptions](/docs/{{version}}/errors#renderable-exceptions).

<a name="global-log-context"></a>
#### Konteks Log Global

Jika tersedia, Laravel secara otomatis menambahkan ID pengguna saat ini ke setiap pesan log exception sebagai data kontekstual. Kamu dapat menentukan data kontekstual global kamu sendiri dengan mengganti metode `context` dari kelas `App\Exceptions\Handler` aplikasi kamu. Informasi ini akan dimasukkan ke dalam setiap pesan log exception yang ditulis oleh aplikasi kamu :

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

Meskipun menambahkan konteks ke setiap pesan log dapat berguna, terkadang exception tertentu mungkin memiliki konteks unik yang ingin kamu sertakan di log. Dengan menentukan metode `context` pada salah satu exception khusus aplikasi kamu, kamu dapat menentukan data apa pun yang relevan dengan exception itu yang harus ditambahkan ke entri pada log exception :

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

Terkadang kamu mungkin perlu melaporkan exception tetapi terus menangani permintaan saat ini. Fungsi helper `report` memungkinkan kamu melaporkan exception dengan cepat melalui pengendali exception tanpa merender halaman kesalahan ke pengguna :

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

Ketika pesan yang ditulis ke aplikasi kamu [logs](/docs/{{version}}/logging), pesan ditulis pada [tingkat log](/docs/{{version}}/logging#log-levels) tertentu, yang menunjukkan tingkat keseriusan atau pentingnya pesan yang sedang dicatat dalam log.

Seperti yang disebutkan di atas, bahkan ketika kamu mendaftarkan callback pelaporan exception kustom menggunakan metode `reportable`, Laravel akan tetap mencatat exception menggunakan konfigurasi logging default untuk aplikasi. Namun, karena level log terkadang dapat memengaruhi saluran tempat pesan dicatat, kamu mungkin ingin mengonfigurasi level log tempat exception tertentu dicatat.

Untuk melakukannya, kamu dapat menentukan array dengan tipe exception dan level log yang terkait di dalam properti `$levels` dari pengendali exception aplikasi kamu :

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

Saat membuat aplikasi kamu, akan ada beberapa jenis exception yang ingin kamu abaikan dan tidak pernah laporkan. Pengendali exception aplikasi kamu berisi properti `$dontReport` yang diinisialisasi ke array kosong. Setiap kelas yang kamu tambahkan ke properti ini tidak akan dilaporkan. Namun, mereka mungkin masih memiliki logika pada rendering kustom :

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
> Di belakang layar, Laravel sudah mengabaikan beberapa jenis kesalahan untuk kamu, seperti exception yang dihasilkan dari kesalahan 404 HTTP "not found" atau 419 respons HTTP yang dihasilkan oleh token CSRF yang tidak valid.

<a name="rendering-exceptions"></a>
### Merender Exception

Secara default, penanganan exception Laravel akan mengonversi exception menjadi respons HTTP untuk kamu. Namun, kamu bebas mendaftarkan closure perenderan khusus untuk exception dari jenis yang diberikan. kamu dapat melakukannya melalui metode `renderable` dari penanganan exception kamu.

Closure yang diteruskan ke metode `renderable` harus mengembalikan instance `Illuminate\Http\Response`, yang dapat dihasilkan melalui bantuan `response`. Laravel akan menyimpulkan jenis exception apa yang dibuat oleh closure dengan memeriksa tipe-hint dari closure :

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

Alih-alih memeriksa exception jenis dalam metode `register` penangan exception, kamu dapat menentukan metode `report` dan `render` langsung pada exception kustom kamu. Ketika metode ini ada, mereka akan dipanggil secara otomatis oleh framework :

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

Jika exception kamu memperluas exception yang sudah dapat dirender, seperti Laravel bawaan atau exception Symfony, kamu dapat mengembalikan `false` dari metode `render` exception untuk merender respons HTTP default exception :

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

Jika exception kamu berisi logika pelaporan kustom yang hanya diperlukan ketika kondisi tertentu terpenuhi, kamu mungkin perlu menginstruksikan Laravel untuk terkadang melaporkan exception tersebut menggunakan konfigurasi penanganan exception default. Untuk menyelesaikannya, kamu dapat mengembalikan nilai `false` dari metode `report` exception :

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

Beberapa exception mendeskripsikan kode eror HTTP dari server. Misalnya, ini mungkin erir "page not found" error (404), "unauthorized error" (401), atau bahkan eror 500 yang dihasilkan pengembang. Untuk menghasilkan respon seperti itu, kamu dapat menggunakan bantuan `abort`:

```php
    abort(404);
```

<a name="custom-http-error-pages"></a>
### Halaman Eror Kustom HTTP

Laravel memudahkan untuk menampilkan halaman eror kustom untuk berbagai kode status HTTP. Misalnya, jika kamu ingin menyesuaikan halaman eror untuk kode status HTTP 404, buat template tampilan `resources/views/errors/404.blade.php`. Tampilan ini akan ditampilkan pada semua eror 404 yang dihasilkan oleh aplikasi kamu. Tampilan dalam direktori ini harus diberi nama agar sesuai dengan kode status HTTP yang sesuai. Instance `Symfony\Component\HttpKernel\Exception\HttpException` yang dimunculkan oleh fungsi `abort` akan diteruskan ke tampilan sebagai variabel `$exception` :

```php
    <h2>{{ $exception->getMessage() }}</h2>
```

Kamu dapat mempublikasikan template halaman eror default Laravel menggunakan perintah Artisan `vendor:publish`. Setelah templat dipublikasikan, kamu dapat menyesuaikannya sesuai keinginan kamu:

```shell
php artisan vendor:publish --tag=laravel-errors
```

<a name="fallback-http-error-pages"></a>
#### Halaman Eror Fallback HTTP

Kamu juga dapat menentukan halaman kesalahan "fallback" untuk rangkaian kode status pada HTTP tertentu. Halaman ini akan dirender jika tidak ada halaman yang sesuai untuk kode status HTTP tertentu yang terjadi. Untuk melakukannya, tentukan template `4xx.blade.php` dan template `5xx.blade.php` di direktori `resources/views/errors` aplikasi kamu.