# Pembuatan URL

- [Pengantar](#introduction)
- [Dasar](#the-basics)
    - [Menghasilkan URL](#generating-urls)
    - [Mengakses URL Saat ini](#accessing-the-current-url)
- [URL Untuk Rute Bernama](#urls-for-named-routes)
    - [Signed URLs](#signed-urls)
- [URLs Untuk _Action Controller_](#urls-for-controller-actions)
- [Nilai _Default_](#default-values)

<a name="introduction"></a>
## Pengantar

Laravel menyediakan beberapa _helper_ untuk mendampingi Anda dalam menghasilkan URL untuk aplikasi Anda. _Helper_ ini akan sangat berguna ketika membuat pranala (_link_) pada _template_ dan respons API Anda, atau ketika membuat respons _redirect_ yang menuju bagian lain aplikasi Anda.

<a name="the-basics"></a>
## Dasar

<a name="generating-urls"></a>
### Menghasilkan URL

_Helper_ `url` dapat digunakan untuk menghasilkan URL untuk aplikasi Anda. URL yang dihasilkan akan secara otomatis menggunakan skema (HTTP atau HTTPS) dan _host_ dari _request_ yang saat ini ditangani oleh aplikasi:

    $post = App\Models\Post::find(1);

    echo url("/posts/{$post->id}");

    // http://example.com/posts/1

<a name="accessing-the-current-url"></a>
### Mengakses URL Saat Ini

Jika tidak ada _path_ yang diberikan pada _helper_ `url`, _instance_ `Illuminate\Routing\UrlGenerator` akan dikembalikan, memungkinkan Anda untuk mengakses informasi terkait URL saat ini:

    // Mengambil URL saat ini tanpa string query...
    echo url()->current();

    // Mengambil URL saat ini dengan string query...
    echo url()->full();

    // Mengambil URL penuh dari request sebelumnya...
    echo url()->previous();

Semua _method_ ini dapat diakses via [_facade_](/docs/{{version}}/facades) `URL`:

    use Illuminate\Support\Facades\URL;

    echo URL::current();

<a name="urls-for-named-routes"></a>
## URL Untuk Rute Bernama

_Helper_ `route` dapat digunakan untuk menghasilkan URL ke [_named routes_](/docs/{{version}}/routing#named-routes). Rute bernama memungkinkan Anda untuk menghasilkan URL tanpa harus ke URL aktual yang didefinisikan pada rute. Oleh karena itu, jika URL pada rute berubah, Anda tidak perlu merubah pemanggilan _function_ `route`-nya. Sebagai contoh, bayangkan aplikasi Anda berisi rute yang didefinisikan seperti berikut ini:

    Route::get('/post/{post}', function (Post $post) {
        //
    })->name('post.show');

Untuk menghasilkan URL ke rute ini, Anda dapat menggunakan _helper_ `route` seperti ini:

    echo route('post.show', ['post' => 1]);

    // http://example.com/post/1

Tentu saja, _helper_ `route` juga dapat digunakan untuk menghasilkan URL untuk rute yang memiliki parameter yang banyak:

    Route::get('/post/{post}/comment/{comment}', function (Post $post, Comment $comment) {
        //
    })->name('comment.show');

    echo route('comment.show', ['post' => 1, 'comment' => 3]);

    // http://example.com/post/1/comment/3

Setiap elemen _array_ tambahan yang tidak sesuai dengan parameter pada pendefinisian rute akan ditambahkan sebagai _string_ kueri URL:

    echo route('post.show', ['post' => 1, 'search' => 'rocket']);

    // http://example.com/post/1?search=rocket

<a name="eloquent-models"></a>
#### Model Eloquent

Anda akan sering membuat URL menggunakan _route key_ (biasanya _primary key_) dari [model Eloquent](/docs/{{version}}/eloquent). Untuk alasan ini, Anda dapat mengoper model Eloquent sebagai nilai parameter. _Helper_ `route` akan secara otomatis mengekstrak _route key_ milik model:

    echo route('post.show', ['post' => $post]);

<a name="signed-urls"></a>
### _Signed URL_

Laravel memungkinkan Anda untuk membuat URL yang "ditandatangani" (_signed URL_) untuk _named route_ secara mudah. URL ini memiliki hash "signature" yang ditambahkan ke _string_ kueri yang memungkinkan Laravel untuk memverifikasi bahwa URL tidak dimodifikasi sejak dibuat. URL yang "ditandatangani" ini sangat berguna untuk rute yang dapat diakses publik namun membutuhkan lapisan perlindungan terhadap manipulasi URL.

Misalnya, Anda mungkin menggunakan URL yang "ditandatangani" ini untuk mengimplementasikan tautan "berhenti berlangganan" yang dapat diakses publik, yang diemailkan ke pelanggan Anda. Untuk membuat URL yang "ditandatangani" atas _named route_, Anda dapat menggunakan metode `signedRoute` dari _facade_ `URL`:

    use Illuminate\Support\Facades\URL;

    return URL::signedRoute('unsubscribe', ['user' => 1]);

Jika Anda ingin membuat URL rute bertanda tangan sementara yang dapat kedaluwarsa setelah jangka waktu tertentu, Anda dapat menggunakan metode `temporarySignedRoute`. Ketika Laravel memvalidasi URL rute bertanda tangan sementara, Laravel akan memastikan bahwa cap waktu kedaluwarsa yang dikodekan ke dalam URL bertanda tangan ini belum terlewati:

    use Illuminate\Support\Facades\URL;

    return URL::temporarySignedRoute(
        'unsubscribe', now()->addMinutes(30), ['user' => 1]
    );

<a name="validating-signed-route-requests"></a>
#### Validasi terhadap _Request_ dari _Signed Route_

Untuk memverifikasi bahwa permintaan yang masuk memiliki tanda tangan yang valid, anda harus memanggil metode `hasValidSignature` pada _instance_ `Illuminate\Http\Request` yang masuk:

    use Illuminate\Http\Request;

    Route::get('/unsubscribe/{user}', function (Request $request) {
        if (! $request->hasValidSignature()) {
            abort(401);
        }

        // ...
    })->name('unsubscribe');

Terkadang, Anda mungkin perlu mengizinkan _frontend_ dari aplikasi Anda untuk menambahkan data ke URL yang ditandatangani ini, seperti saat melakukan _pagination_ dari sisi klien. Oleh karena itu, Anda dapat tidak mengikutsertakan parameter kueri _request_ dari "tanda tangan" yang akan diabaikan saat memvalidasi URL yang "ditandatangani" ini menggunakan metode `hasValidSignatureWhileIgnoring`. Ingat, mengabaikan parameter memungkinkan siapa pun untuk memodifikasi parameter tersebut:

    if (! $request->hasValidSignatureWhileIgnoring(['page', 'order'])) {
        abort(401);
    }

Alih-alih memvalidasi URL yang ditandatangani menggunakan _instance_ _request_ yang masuk, Anda juga dapat menugaskan  [_middleware_](/docs/{{version}}/middleware) `Illuminate\Routing\Middleware\ValidateSignature` pada sebuah rute. Jika belum ada, Anda harus menetapkan _middleware_ ini sebagai kunci dalam _array_ `routeMiddleware` kernel HTTP Anda:

    /**
     * Middleware rute pada aplikasi.
     *
     * middleware yang terdaftar disini dapat ditambatkan pada rute biasa dan rute grup.
     *
     * @var array
     */
    protected $routeMiddleware = [
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
    ];

Setelah Anda mendaftarkan _middleware_ di kernel Anda, Anda dapat melampirkannya ke sebuah rute. Jika permintaan yang masuk tidak memiliki tanda tangan yang valid, middleware akan secara otomatis mengembalikan respons HTTP `403`:

Once you have registered the middleware in your kernel, you may attach it to a route. If the incoming request does not have a valid signature, the middleware will automatically return a `403` HTTP response:

    Route::post('/unsubscribe/{user}', function (Request $request) {
        // ...
    })->name('unsubscribe')->middleware('signed');

<a name="responding-to-invalid-signed-routes"></a>
#### Responding To Invalid Signed Routes

Ketika seseorang mengunjungi URL bertanda tangan yang telah kedaluwarsa, mereka akan menerima halaman kesalahan umum untuk kode status HTTP `403`. Namun, Anda dapat menyesuaikan perilaku ini dengan mendefinisikan penutupan "renderable" khusus untuk pengecualian `InvalidSignatureException` di penangan pengecualian Anda. Penutupan ini harus mengembalikan respons HTTP:

When someone visits a signed URL that has expired, they will receive a generic error page for the `403` HTTP status code. However, you can customize this behavior by defining a custom "renderable" closure for the `InvalidSignatureException` exception in your exception handler. This closure should return an HTTP response:

    use Illuminate\Routing\Exceptions\InvalidSignatureException;

    /**
     * Daftarkan callback penanganan pengecualian untuk aplikasi.
     * Register the exception handling callbacks for the application.
     *
     * @return void
     */
    public function register()
    {
        $this->renderable(function (InvalidSignatureException $e) {
            return response()->view('error.link-expired', [], 403);
        });
    }

<a name="urls-for-controller-actions"></a>
## URLs For Controller Actions

Fungsi `action` menghasilkan URL untuk aksi pengontrol yang diberikan:

The `action` function generates a URL for the given controller action:

    use App\Http\Controllers\HomeController;

    $url = action([HomeController::class, 'index']);

Jika metode controller menerima parameter rute, Anda dapat mengoper array asosiatif parameter rute sebagai argumen kedua ke fungsi:

If the controller method accepts route parameters, you may pass an associative array of route parameters as the second argument to the function:

    $url = action([UserController::class, 'profile'], ['id' => 1]);

<a name="default-values"></a>
## Default Values

Untuk beberapa aplikasi, Anda mungkin ingin menentukan nilai default seluruh permintaan untuk parameter URL tertentu. Misalnya, bayangkan banyak rute Anda mendefinisikan parameter `{locale}`:

For some applications, you may wish to specify request-wide default values for certain URL parameters. For example, imagine many of your routes define a `{locale}` parameter:

    Route::get('/{locale}/posts', function () {
        //
    })->name('post.index');

Akan sangat merepotkan untuk selalu mengoper `locale` setiap kali Anda memanggil _helper_ `route`. Jadi, Anda dapat menggunakan metode `URL::defaults` untuk mendefinisikan nilai default untuk parameter ini yang akan selalu diterapkan selama permintaan saat ini. Anda mungkin ingin memanggil metode ini dari [route middleware](/docs/{{version}}/middleware#assigning-middleware-to-routes) sehingga Anda memiliki akses ke permintaan saat ini:

It is cumbersome to always pass the `locale` every time you call the `route` _helper_. So, you may use the `URL::defaults` method to define a default value for this parameter that will always be applied during the current request. You may wish to call this method from a [route middleware](/docs/{{version}}/middleware#assigning-middleware-to-routes) so that you have access to the current request:

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Support\Facades\URL;

    class SetDefaultLocaleForUrls
    {
        /**
         * Menangani request yang masuk.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return \Illuminate\Http\Response
         */
        public function handle($request, Closure $next)
        {
            URL::defaults(['locale' => $request->user()->locale]);

            return $next($request);
        }
    }

Setelah nilai default untuk parameter `locale` telah ditetapkan, Anda tidak lagi diharuskan untuk meneruskan nilainya ketika membuat URL melalui _helper_ `route`.

Once the default value for the `locale` parameter has been set, you are no longer required to pass its value when generating URLs via the `route` _helper_.

<a name="url-defaults-middleware-priority"></a>
#### URL Defaults & Middleware Priority

Menetapkan nilai default URL dapat mengganggu penanganan Laravel terhadap binding model implisit. Oleh karena itu, Anda harus [memprioritaskan middleware Anda](/docs/{{version}}/middleware#sorting-middleware) yang mengatur default URL untuk dieksekusi sebelum middleware `SubstituteBindings` Laravel sendiri. Anda dapat mencapai ini dengan memastikan middleware Anda terjadi sebelum middleware `SubstituteBindings` dalam properti `$middlewarePriority` dari kernel HTTP aplikasi Anda.

Setting URL default values can interfere with Laravel's handling of implicit model bindings. Therefore, you should [prioritize your middleware](/docs/{{version}}/middleware#sorting-middleware) that set URL defaults to be executed before Laravel's own `SubstituteBindings` middleware. You can accomplish this by making sure your middleware occurs before the `SubstituteBindings` middleware within the `$middlewarePriority` property of your application's HTTP kernel.

Properti `$middlewarePriority` didefinisikan dalam kelas dasar `Illuminate\Foundation\Http\Kernel`. Anda dapat menyalin definisinya dari kelas tersebut dan menimpa di kernel HTTP aplikasi Anda untuk memodifikasinya:

    /**
     * Daftar middleware yang telah diurutkan prioritasnya.
     *
     * Pengurututan ini bertujuan untuk memaksa middleware non-global untuk selalu berada dalam urutan yang diberikan.
     *
     * @var array
     */
    protected $middlewarePriority = [
        // ...
         \App\Http\Middleware\SetDefaultLocaleForUrls::class,
         \Illuminate\Routing\Middleware\SubstituteBindings::class,
         // ...
    ];
