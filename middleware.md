# Middleware

- [Pendahuluan](#introduction)
- [Mendefinisikan _Middleware_](#defining-middleware)
- [Mendaftarkan _Middleware_](#registering-middleware)
    - [_Middleware_ Global](#global-middleware)
    - [Menugaskan _Middleware_ pada Rute](#assigning-middleware-to-routes)
    - [Grup _Middleware_](#middleware-groups)
    - [Mengurutkan _Middleware_](#sorting-middleware)
- [Parameter _Middleware_](#middleware-parameters)
- [_Middleware_ yang Dapat Dihentikan](#terminable-middleware)

<a name="introduction"></a>
## Pendahuluan

_Middleware_ menyediakan mekanisme yang mudah untuk memeriksa dan menyaring _request_ HTTP yang masuk ke aplikasi Anda. Misalnya, Laravel menyertakan _middleware_ yang memverifikasi bahwa pengguna pada aplikasi Anda telah terautentikasi. Jika pengguna tidak terautentikasi, _middleware_ akan mengarahkan pengguna ke halaman _login_. Namun, jika pengguna telah terautentikasi, _middleware_ akan mengizinkan _request_ untuk melakukan proses lebih lanjut terhadap aplikasi.  

_Middleware_ tambahan dapat ditulis untuk melakukan berbagai tugas selain autentikasi. Misalnya, _middleware_ untuk _log_ mungkin akan mencatat semua _request_ yang masuk ke aplikasi Anda. Ada beberapa _middleware_ yang disertakan dalam _framework_ Laravel, termasuk _middleware_ untuk autentikasi dan perlindungan CSRF. Semua _middleware_ ini terletak di dalam direktori `app/Http/Middleware`.

<a name="defining-middleware"></a>
## Mendefinisikan _Middleware_

Untuk membuat _middleware_ baru, Anda dapat menggunakan perintah Artisan `make:middleware`:

```shell
php artisan make:middleware EnsureTokenIsValid
```

Perintah ini akan menempatkan kelas `EnsureTokenIsValid` yang baru di dalam direktori `app/Http/Middleware` Anda. Dalam _middleware_ ini, kita hanya mengizinkan akses ke _route_ tujuan jika _input_ `token` yang disediakan telah sesuai dengan nilai yang ditentukan. Jika tidak, kita akan mengarahkan pengguna kembali ke URI `home`:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class EnsureTokenIsValid
    {
        /**
         * Menangani request yang masuk.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            if ($request->input('token') !== 'my-secret-token') {
                return redirect('home');
            }

            return $next($request);
        }
    }

Seperti yang Anda lihat, jika `token` yang diberikan tidak cocok dengan token rahasia kita, _middleware_ akan mengembalikan HTTP _redirect_ ke klien; jika sebaliknya, _request_ akan diteruskan ke dalam aplikasi. Untuk meneruskan _request_ lebih jauh ke dalam aplikasi, Anda harus memanggil _callback_ `$next` disertai `$request`.

Untuk lebih mudah memahaminya, Anda dapat membayangkan _middleware_ sebagai serangkaian "lapisan" yang harus dilewati oleh _request_ HTTP sebelum mencapai aplikasi Anda. Setiap lapisan akan memeriksa _request_ dan bahkan menolaknya secara keseluruhan.

> **Catatan**  
> Semua _middleware_ dideklarasikan melalui [_service container_](/docs/{{version}}/container), jadi Anda dapat melakukan _type-hint_ dependensi apa pun yang Anda perlukan di dalam _constructor_ milik _middleware_.

<a name="middleware-and-responses"></a>
#### Sebelum-sesudah _Middleware_

Tentu saja, _middleware_ dapat melakukan tugas-tugas yang berjalan sebelum atau sesudah meneruskan _request_ yanh lebih dalam ke aplikasi. Sebagai contoh, _middleware_ berikut ini akan melakukan beberapa tugas **sebelum** _request_ ditangani oleh aplikasi:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class BeforeMiddleware
    {
        public function handle($request, Closure $next)
        {
            // Perform action

            return $next($request);
        }
    }

Namun, middleware ini akan melakukan tugasnya **setelah** _request_ ditangani oleh aplikasi:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class AfterMiddleware
    {
        public function handle($request, Closure $next)
        {
            $response = $next($request);

            // Perform action

            return $response;
        }
    }

<a name="registering-middleware"></a>
##  Mendaftarkan _Middleware_

<a name="global-middleware"></a>
### _Middleware_ Global

Jika Anda ingin sebuah _middleware_ dapat berjalan di setiap _request_ HTTP pada aplikasi Anda, Anda dapat mendaftarkan kelas _middleware_ tersebut pada properti `$middleware` dari kelas `app/Http/Kernel.php` Anda.

<a name="assigning-middleware-to-routes"></a>
### Menugaskan _Middleware_ pada Rute

Jika Anda ingin menugaskan _middleware_ ke _route_ tertentu, Anda harus menetapkan sebuah kunci (_array_) untuk _middleware_ tersebut di dalam _file_ `app/Http/Kernel.php` pada aplikasi Anda. Secara default, properti `$routeMiddleware` dari kelas ini berisi entri untuk _middleware_ bawaan dari Laravel. Anda dapat menambahkan _middleware_ Anda sendiri ke dalam daftar ini dan menetapkan kunci yang Anda pilih:

    // Kelas App\Http\Kernel...

    protected $routeMiddleware = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
        'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
    ];

Setelah _middleware_ didefinisikan dalam kernel HTTP, Anda dapat menggunakan metode `middleware` untuk menetapkan _middleware_ pada suatu _route_:

    Route::get('/profile', function () {
        //
    })->middleware('auth');

Anda dapat menugaskan beberapa _middleware_ pada satu _route_ dengan mengoper _array_ nama (kunci) _middleware_ pada metode `middleware`:

    Route::get('/', function () {
        //
    })->middleware(['pertama', 'kedua']);

Ketika menetapkan _middleware_, Anda juga dapat mengoper nama kelas yang lengkap:

    use App\Http\Middleware\EnsureTokenIsValid;

    Route::get('/profile', function () {
        //
    })->middleware(EnsureTokenIsValid::class);

<a name="excluding-middleware"></a>
#### Mengecualikan _Middleware_

Ketika menugaskan _middleware_ ke sekelompok _route_ (grup), Anda mungkin sesekali perlu mencegah _middleware_ untuk diterapkan ke salah satu _route_di dalam grup tersebut. Anda dapat melakukannya dengan menggunakan metode `withoutMiddleware`:

    use App\Http\Middleware\EnsureTokenIsValid;

    Route::middleware([EnsureTokenIsValid::class])->group(function () {
        Route::get('/', function () {
            //
        });

        Route::get('/profile', function () {
            //
        })->withoutMiddleware([EnsureTokenIsValid::class]);
    });

Anda juga dapat mengecualikan sekumpulan _middleware_ tertentu untuk seluruh [grup](/docs/{{version}}/routing#route-groups) ketika mendefinisikan _route_:

    use App\Http\Middleware\EnsureTokenIsValid;

    Route::withoutMiddleware([EnsureTokenIsValid::class])->group(function () {
        Route::get('/profile', function () {
            //
        });
    });

Metode `withoutMiddleware` hanya dapat menghapus _middleware_ _route_ dan tidak berlaku untuk [_middleware_ global](#global-middleware).

<a name="middleware-groups"></a>
### Grup _Middleware_

Terkadang Anda ingin mengelompokkan beberapa _middleware_ ke dalam satu (kata) kunci untuk memudahkan penugasannya ke _route_-_route_. Anda dapat mencapai hal ini menggunakan properti `$middlewareGroups` pada kernel HTTP Anda.

Laravel menyertakan grup _middleware_ yang telah didefinisikan sebelumnya bernama `web` dan `api`  yang berisi _middleware_ umum yang mungkin ingin Anda terapkan ke rute web dan API Anda. Ingat, grup _middleware_ ini secara otomatis diterapkan oleh penyedia layanan `App\Providers\RouteServiceProvider` aplikasi Anda ke _route_-_route_ yang terdapat di dalam file `web` dan `api` milik Anda:

    /**
     * Grup middleware untuk _route_ pada aplikasi.
     *
     * @var array
     */
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],

        'api' => [
            'throttle:api',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],
    ];

Grup _middleware_ dapat ditugaskan ke _route_ dan _action_ pada _controller_ menggunakan sintaks yang sama seperti _middleware_ tunggal. Sekali lagi, grup _middleware_ berguna untuk memudahkan penugasan banyak _middleware_ sekaligus ke sebuah _route_:

    Route::get('/', function () {
        //
    })->middleware('web');

    Route::middleware(['web'])->group(function () {
        //
    });

> **Catatan**  
> Laravel secara _default_ memiliki grup _middleware_ bernama `web` dan `api` yang secara otomatis diterapkan pada _file_ `routes/web.php` dan `routes/api.php` pada aplikasi Anda yang terdapat pada _file_ `App\Providers\RouteServiceProvider`.

<a name="sorting-middleware"></a>
### Mengurutkan _Middleware_

Walau jarang, Anda mungkin akan memerlukan _middleware_ yang dieksekusi dalam urutan tertentu, namun Anda tidak dapat mengendalikan urutannya ketika _middleware_ tersebut ditugaskan pada sebuah _route_. Dalam hal ini, Anda dapat menentukan prioritas _middleware_ Anda dengan menggunakan properti `$middlewarePriority` pada _file_ `app/Http/Kernel.php`. Properti ini mungkin tidak ada pada kernel HTTP secara _default_. Anda dapat menyalin definisi _default_ untuk properti tersebut pada kode di bawah ini:

    /**
     * Daftar middleware yang diurutkan berdasarkan prioritasnya.
     *
     * Hal ini akan memaksa middleware non-global untuk selalu berada pada urutan yang sudah ditentukan.
     *
     * @var string[]
     */
    protected $middlewarePriority = [
        \Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests::class,
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Contracts\Auth\Middleware\AuthenticatesRequests::class,
        \Illuminate\Routing\Middleware\ThrottleRequests::class,
        \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
        \Illuminate\Contracts\Session\Middleware\AuthenticatesSessions::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \Illuminate\Auth\Middleware\Authorize::class,
    ];

<a name="middleware-parameters"></a>
## Parameter _Middleware_

_Middleware_ juga dapat menerima parameter tambahan. Misalnya, jika aplikasi Anda perlu memverifikasi bahwa pengguna yang diautentikasi memiliki "peran" tertentu sebelum melakukan tindakan tertentu, Anda dapat membuat _middleware_ `EnsureUserHasRole` yang menerima nama peran sebagai argumen tambahan.

Parameter tambahan untuk _middleware_ akan diteruskan ke _middleware_ setelah argumen `$next`:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class EnsureUserHasRole
    {
        /**
         * Menangani request yang masuk.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @param  string  $role
         * @return mixed
         */
        public function handle($request, Closure $next, $role)
        {
            if (! $request->user()->hasRole($role)) {
                // Redirect...
            }

            return $next($request);
        }

    }

Argumen untuk _middleware_ dapat ditentukan ketika mendefinisikan _route_ dengan memisahkan nama _middleware_ dan parameter dengan tanda `:`. Jika terdapat lebih dari satu parameter tambahan, antar argumen harus dipisahkan dengan koma:

    Route::put('/post/{id}', function ($id) {
        //
    })->middleware('role:editor');

<a name="terminable-middleware"></a>
## _Middleware_ yang Dapat Dihentikan

Terkadang _middleware_ mungkin perlu melakukan beberapa pekerjaan setelah mengirimkan respons HTTP ke _browser_. Jika Anda mendefinisikan metode `terminate` pada _middleware_ Anda dan server web Anda telah menggunakan FastCGI, metode `terminate` akan secara otomatis dipanggil setelah respons dikirim ke _browser_:

    <?php

    namespace Illuminate\Session\Middleware;

    use Closure;

    class TerminatingMiddleware
    {
        /**
         * Menangani request yang masuk.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            return $next($request);
        }

        /**
         * Menangani tugas setelah respons dikirimkan ke browser.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Illuminate\Http\Response  $response
         * @return void
         */
        public function terminate($request, $response)
        {
            // ...
        }
    }

Metode `terminate` harus menerima _request_ dan _response_. Setelah Anda mendefinisikan _middleware_ yang dapat dihentikan (_terminable_), Anda harus _middleware_ tersebut ke daftar _route_ atau _middleware_ global pada _file_ `app/Http/Kernel.php`.

Ketika memanggil metode `terminate` pada _middleware_, Laravel akan membuat _instance_ baru _middleware_ yang berasal dari [_service container_](/docs/{{version}}/container). Jika Anda ingin menggunakan _instance middleware_ yang sama ketika metode `handle` dan `terminate` dipanggil, Anda dapat mendaftarkan _middleware_ pada _container_ menggunakan metode `singleton` milik _container_. Biasanya ini harus dilakukan di dalam metode `register` pada `AppServiceProvider` Anda:

    use App\Http\Middleware\TerminatingMiddleware;

    /**
     * Mendaftarkan layanan pada aplikasi.
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(TerminatingMiddleware::class);
    }
