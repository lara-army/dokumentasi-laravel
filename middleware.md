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

Middleware provide a convenient mechanism for inspecting and filtering HTTP requests entering your application. For example, Laravel includes a middleware that verifies the user of your application is authenticated. If the user is not authenticated, the middleware will redirect the user to your application's login screen. However, if the user is authenticated, the middleware will allow the request to proceed further into the application.

Middleware menyediakan mekanisme yang nyaman untuk memeriksa dan menyaring permintaan HTTP yang masuk ke aplikasi Anda. Misalnya, Laravel menyertakan middleware yang memverifikasi pengguna aplikasi Anda telah diautentikasi. Jika pengguna tidak terautentikasi, middleware akan mengarahkan pengguna ke layar login aplikasi Anda. Namun, jika pengguna diautentikasi, middleware akan mengizinkan permintaan untuk melanjutkan lebih jauh ke dalam aplikasi.

Additional middleware can be written to perform a variety of tasks besides authentication. For example, a logging middleware might log all incoming requests to your application. There are several middleware included in the Laravel framework, including middleware for authentication and CSRF protection. All of these middleware are located in the `app/Http/Middleware` directory.

Middleware tambahan dapat ditulis untuk melakukan berbagai tugas selain otentikasi. Misalnya, middleware logging mungkin mencatat semua permintaan yang masuk ke aplikasi Anda. Ada beberapa middleware yang disertakan dalam kerangka kerja Laravel, termasuk middleware untuk otentikasi dan perlindungan CSRF. Semua middleware ini terletak di direktori `app/Http/Middleware`.

<a name="defining-middleware"></a>
## Mendefinisikan _Middleware_

To create a new middleware, use the `make:middleware` Artisan command:

Untuk membuat middleware baru, gunakan perintah Artisan `make:middleware`:

```shell
php artisan make:middleware EnsureTokenIsValid
```

This command will place a new `EnsureTokenIsValid` class within your `app/Http/Middleware` directory. In this middleware, we will only allow access to the route if the supplied `token` input matches a specified value. Otherwise, we will redirect the users back to the `home` URI:

Perintah ini akan menempatkan kelas `EnsureTokenIsValid` baru di dalam direktori `app/Http/Middleware` Anda. Dalam middleware ini, kita hanya akan mengizinkan akses ke rute jika input `token` yang disediakan cocok dengan nilai yang ditentukan. Jika tidak, kita akan mengarahkan pengguna kembali ke URI `home`:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class EnsureTokenIsValid
    {
        /**
         * Handle an incoming request.
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

As you can see, if the given `token` does not match our secret token, the middleware will return an HTTP redirect to the client; otherwise, the request will be passed further into the application. To pass the request deeper into the application (allowing the middleware to "pass"), you should call the `$next` callback with the `$request`.

Seperti yang Anda lihat, jika `token` yang diberikan tidak cocok dengan token rahasia kita, middleware akan mengembalikan HTTP redirect ke klien; jika tidak, permintaan akan diteruskan lebih jauh ke dalam aplikasi. Untuk meneruskan permintaan lebih jauh ke dalam aplikasi (memungkinkan middleware untuk "lulus"), Anda harus memanggil callback `$next` dengan `$request`.

It's best to envision middleware as a series of "layers" HTTP requests must pass through before they hit your application. Each layer can examine the request and even reject it entirely.

Yang terbaik adalah membayangkan middleware sebagai serangkaian "lapisan" yang harus dilewati permintaan HTTP sebelum mereka mencapai aplikasi Anda. Setiap lapisan dapat memeriksa permintaan dan bahkan menolaknya sepenuhnya.

> **Catatan**  
> All middleware are resolved via the [service container](/docs/{{version}}/container), so you may type-hint any dependencies you need within a middleware's constructor.

> Semua middleware diselesaikan melalui [service container](/docs/{{version}}/container), jadi Anda dapat mengetikkan petunjuk ketergantungan apa pun yang Anda perlukan dalam konstruktor middleware.

<a name="before-after-middleware"></a>
<a name="middleware-and-responses"></a>
#### Middleware & Responsnya

Of course, a middleware can perform tasks before or after passing the request deeper into the application. For example, the following middleware would perform some task **before** the request is handled by the application:

Tentu saja, middleware dapat melakukan tugas-tugas sebelum atau sesudah meneruskan permintaan lebih dalam ke aplikasi. Sebagai contoh, middleware berikut ini akan melakukan beberapa tugas **sebelum** permintaan ditangani oleh aplikasi:

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

However, this middleware would perform its task **after** the request is handled by the application:

Namun, middleware ini akan melakukan tugasnya **setelah** permintaan ditangani oleh aplikasi:

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

If you want a middleware to run during every HTTP request to your application, list the middleware class in the `$middleware` property of your `app/Http/Kernel.php` class.

Jika Anda ingin middleware berjalan selama setiap permintaan HTTP ke aplikasi Anda, daftarkan kelas middleware di properti `$middleware` dari kelas `app/Http/Kernel.php` Anda.

<a name="assigning-middleware-to-routes"></a>
### Menugaskan _Middleware_ pada Rute

If you would like to assign middleware to specific routes, you should first assign the middleware a key in your application's `app/Http/Kernel.php` file. By default, the `$routeMiddleware` property of this class contains entries for the middleware included with Laravel. You may add your own middleware to this list and assign it a key of your choosing:

Jika Anda ingin menugaskan middleware ke rute tertentu, Anda harus terlebih dahulu menetapkan middleware sebagai kunci dalam file `app/Http/Kernel.php` aplikasi Anda. Secara default, properti `$routeMiddleware` dari kelas ini berisi entri untuk middleware yang disertakan dengan Laravel. Anda dapat menambahkan middleware Anda sendiri ke dalam daftar ini dan menetapkan kunci yang Anda pilih:

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

Once the middleware has been defined in the HTTP kernel, you may use the `middleware` method to assign middleware to a route:

Setelah middleware didefinisikan dalam kernel HTTP, Anda dapat menggunakan metode `middleware` untuk menetapkan middleware ke rute:

    Route::get('/profile', function () {
        //
    })->middleware('auth');

You may assign multiple middleware to the route by passing an array of middleware names to the `middleware` method:

Anda dapat menugaskan beberapa middleware ke rute dengan mengoper array nama middleware ke metode `middleware`:

    Route::get('/', function () {
        //
    })->middleware(['first', 'second']);

When assigning middleware, you may also pass the fully qualified class name:

Ketika menetapkan middleware, Anda juga dapat mengoper nama kelas yang sepenuhnya memenuhi syarat:

    use App\Http\Middleware\EnsureTokenIsValid;

    Route::get('/profile', function () {
        //
    })->middleware(EnsureTokenIsValid::class);

<a name="excluding-middleware"></a>
#### Mengecualikan _Middleware_

When assigning middleware to a group of routes, you may occasionally need to prevent the middleware from being applied to an individual route within the group. You may accomplish this using the `withoutMiddleware` method:

Ketika menugaskan middleware ke sekelompok rute, Anda mungkin sesekali perlu mencegah middleware diterapkan ke rute individual dalam grup. Anda dapat melakukannya dengan menggunakan metode `withoutMiddleware`:

    use App\Http\Middleware\EnsureTokenIsValid;

    Route::middleware([EnsureTokenIsValid::class])->group(function () {
        Route::get('/', function () {
            //
        });

        Route::get('/profile', function () {
            //
        })->withoutMiddleware([EnsureTokenIsValid::class]);
    });

You may also exclude a given set of middleware from an entire [group](/docs/{{version}}/routing#route-groups) of route definitions:

Anda juga dapat mengecualikan sekumpulan middleware tertentu dari seluruh [grup](/docs/{{version}}/routing#route-groups) dari definisi rute:

    use App\Http\Middleware\EnsureTokenIsValid;

    Route::withoutMiddleware([EnsureTokenIsValid::class])->group(function () {
        Route::get('/profile', function () {
            //
        });
    });

The `withoutMiddleware` method can only remove route middleware and does not apply to [global middleware](#global-middleware).

<a name="middleware-groups"></a>
### Grup _Middleware_

Sometimes you may want to group several middleware under a single key to make them easier to assign to routes. You may accomplish this using the `$middlewareGroups` property of your HTTP kernel.

Metode `withoutMiddleware` hanya dapat menghapus middleware rute dan tidak berlaku untuk [global middleware] (#global-middleware).

Laravel includes predefined `web` and `api` middleware groups that contain common middleware you may want to apply to your web and API routes. Remember, these middleware groups are automatically applied by your application's `App\Providers\RouteServiceProvider` service provider to routes within your corresponding `web` and `api` route files:

Laravel menyertakan grup middleware `web` dan `api` yang telah ditentukan sebelumnya yang berisi middleware umum yang mungkin ingin Anda terapkan ke rute web dan API Anda. Ingat, grup middleware ini secara otomatis diterapkan oleh penyedia layanan `App\Providers\RouteServiceProvider` aplikasi Anda ke rute-rute di dalam file rute `web` dan `api` Anda yang sesuai:

    /**
     * Grup middleware untuk rute aplikasi.
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

Middleware groups may be assigned to routes and controller actions using the same syntax as individual middleware. Again, middleware groups make it more convenient to assign many middleware to a route at once:

Grup middleware dapat ditugaskan ke rute dan aksi pengontrol menggunakan sintaks yang sama seperti middleware individual. Sekali lagi, grup middleware membuatnya lebih nyaman untuk menetapkan banyak middleware ke rute sekaligus:

    Route::get('/', function () {
        //
    })->middleware('web');

    Route::middleware(['web'])->group(function () {
        //
    });

> **Catatan**  
> Out of the box, the `web` and `api` middleware groups are automatically applied to your application's corresponding `routes/web.php` and `routes/api.php` files by the `App\Providers\RouteServiceProvider`.

> Di luar kotak, grup middleware `web` dan `api` secara otomatis diterapkan ke file `routes/web.php` dan `routes/api.php` yang sesuai dengan aplikasi Anda oleh file `App\Providers\RouteServiceProvider`.

<a name="sorting-middleware"></a>
### Mengurutkan _Middleware_

Rarely, you may need your middleware to execute in a specific order but not have control over their order when they are assigned to the route. In this case, you may specify your middleware priority using the `$middlewarePriority` property of your `app/Http/Kernel.php` file. This property may not exist in your HTTP kernel by default. If it does not exist, you may copy its default definition below:

Jarang, Anda mungkin memerlukan middleware Anda untuk dieksekusi dalam urutan tertentu tetapi tidak memiliki kontrol atas urutannya ketika mereka ditugaskan ke rute. Dalam hal ini, Anda dapat menentukan prioritas middleware Anda menggunakan properti `$middlewarePriority` dari file `app/Http/Kernel.php` Anda. Properti ini mungkin tidak ada di kernel HTTP Anda secara default. Jika tidak ada, Anda dapat menyalin definisi default-nya di bawah ini:

    /**
     * The priority-sorted list of middleware.
     *
     * This forces non-global middleware to always be in the given order.
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
## Parameten _Middleware_

Middleware can also receive additional parameters. For example, if your application needs to verify that the authenticated user has a given "role" before performing a given action, you could create an `EnsureUserHasRole` middleware that receives a role name as an additional argument.

Middleware juga dapat menerima parameter tambahan. Misalnya, jika aplikasi Anda perlu memverifikasi bahwa pengguna yang diautentikasi memiliki "peran" tertentu sebelum melakukan tindakan tertentu, Anda dapat membuat middleware `EnsureUserHasRole` yang menerima nama peran sebagai argumen tambahan.

Additional middleware parameters will be passed to the middleware after the `$next` argument:

Parameter middleware tambahan akan diteruskan ke middleware setelah argumen `$next`:

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

Middleware parameters may be specified when defining the route by separating the middleware name and parameters with a `:`. Multiple parameters should be delimited by commas:

Parameter middleware dapat ditentukan ketika mendefinisikan rute dengan memisahkan nama middleware dan parameter dengan tanda `:`. Beberapa parameter harus dipisahkan dengan koma:

    Route::put('/post/{id}', function ($id) {
        //
    })->middleware('role:editor');

<a name="terminable-middleware"></a>
## _Middleware_ yang Dapat Dihentikan

Sometimes a middleware may need to do some work after the HTTP response has been sent to the browser. If you define a `terminate` method on your middleware and your web server is using FastCGI, the `terminate` method will automatically be called after the response is sent to the browser:

Kadang-kadang middleware mungkin perlu melakukan beberapa pekerjaan setelah respons HTTP dikirim ke browser. Jika Anda mendefinisikan metode `terminate` pada middleware Anda dan server web Anda menggunakan FastCGI, metode `terminate` akan secara otomatis dipanggil setelah respons dikirim ke browser:

    <?php

    namespace Illuminate\Session\Middleware;

    use Closure;

    class TerminatingMiddleware
    {
        /**
         * Handle an incoming request.
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
         * Handle tasks after the response has been sent to the browser.
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

The `terminate` method should receive both the request and the response. Once you have defined a terminable middleware, you should add it to the list of routes or global middleware in the `app/Http/Kernel.php` file.

Metode `terminate` harus menerima permintaan dan respons. Setelah Anda mendefinisikan middleware yang dapat dihentikan, Anda harus menambahkannya ke daftar rute atau middleware global di file `app/Http/Kernel.php`.

When calling the `terminate` method on your middleware, Laravel will resolve a fresh instance of the middleware from the [service container](/docs/{{version}}/container). If you would like to use the same middleware instance when the `handle` and `terminate` methods are called, register the middleware with the container using the container's `singleton` method. Typically this should be done in the `register` method of your `AppServiceProvider`:

Ketika memanggil metode `terminate` pada middleware Anda, Laravel akan menyelesaikan instance baru middleware dari [service container](/docs/{{version}}/container). Jika Anda ingin menggunakan instance middleware yang sama ketika metode `handle` dan `terminate` dipanggil, daftarkan middleware dengan container menggunakan metode `singleton` container. Biasanya ini harus dilakukan di metode `register` dari `AppServiceProvider` Anda:

    use App\Http\Middleware\TerminatingMiddleware;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(TerminatingMiddleware::class);
    }
