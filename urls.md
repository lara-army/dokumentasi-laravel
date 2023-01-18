# Pembuatan URL

- [Pengantar](#introduction)
- [Dasar](#the-basics)
    - [Membuat URL](#generating-urls)
    - [Mengakses URL Saat ini](#accessing-the-current-url)
- [URL Untuk _Named Route_](#urls-for-named-routes)
    - [_Signed_ URL](#signed-urls)
- [URL Untuk Aksi _Controller_](#urls-for-controller-actions)
- [Nilai _Default_](#default-values)

<a name="introduction"></a>
## Pengantar

Laravel menyediakan beberapa _helper_ untuk mendampingi Anda dalam membuat URL untuk aplikasi Anda. _Helper_ ini akan sangat berguna ketika membuat pranala (_link_) pada _template_ dan respons API Anda, atau ketika membuat respons _redirect_ (pengalihan) yang menuju bagian lain aplikasi Anda.

<a name="the-basics"></a>
## Dasar

<a name="generating-urls"></a>
### Membuat URL

_Helper_ `url` dapat digunakan untuk membuat URL untuk aplikasi Anda. URL yang dihasilkan akan secara otomatis menggunakan skema (HTTP atau HTTPS) dan _host_ dari _request_ (permintaan) yang sedang ditangani oleh aplikasi:

    $post = App\Models\Post::find(1);

    echo url("/posts/{$post->id}");

    // http://example.com/posts/1

<a name="accessing-the-current-url"></a>
### Mengakses URL Saat Ini

Jika tidak ada _path_ yang diberikan pada _helper_ `url`, _instance_ dari `Illuminate\Routing\UrlGenerator` akan dikembalikan, yang memungkinkan Anda untuk mengakses informasi terkait URL saat ini:

    // Mengambil URL saat ini tanpa string kueri...
    echo url()->current();

    // Mengambil URL saat ini dengan string kueri...
    echo url()->full();

    // Mengambil URL lengkap dari request sebelumnya...
    echo url()->previous();

Semua metode (_method_) ini dapat diakses via [_facade_](/docs/{{version}}/facades) `URL`:

    use Illuminate\Support\Facades\URL;

    echo URL::current();

<a name="urls-for-named-routes"></a>
## URL Untuk _Named Route_

_Helper_ `route` dapat digunakan untuk membuat URL ke [_named route_](/docs/{{version}}/routing#named-routes) (rute bernama). Rute bernama memungkinkan Anda untuk membuat URL tanpa harus ke URL aktual yang didefinisikan pada rute (_route_). Oleh karena itu, jika URL pada rute telah berubah, Anda tidak perlu merubah pemanggilan fungsi (_function_) `route`-nya. Sebagai contoh, bayangkan aplikasi Anda berisi rute yang didefinisikan seperti berikut ini:

    Route::get('/post/{post}', function (Post $post) {
        //
    })->name('post.show');

Untuk membuat URL ke rute ini, Anda dapat menggunakan _helper_ `route` seperti ini:

    echo route('post.show', ['post' => 1]);

    // http://example.com/post/1

Tentu saja, _helper_ `route` juga dapat digunakan untuk membuat URL untuk rute yang memiliki parameter yang banyak:

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
     * middleware yang terdaftar disini dapat ditambatkan pada rute grup atau rute tunggal.
     *
     * @var array
     */
    protected $routeMiddleware = [
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
    ];

Setelah Anda mendaftarkan _middleware_ di kernel Anda, Anda dapat manambatkannya pada sebuah _route_. Jika permintaan yang masuk tidak memiliki tanda tangan yang valid, _middleware_ akan secara otomatis mengembalikan respons HTTP `403`:

    Route::post('/unsubscribe/{user}', function (Request $request) {
        // ...
    })->name('unsubscribe')->middleware('signed');

<a name="responding-to-invalid-signed-routes"></a>
#### Merespons _Signed Route_ yang tidak valid

Ketika seseorang mengunjungi _signed_ URL yang telah kedaluwarsa, mereka akan menerima halaman eror yang umum untuk kode status HTTP `403`. Namun, Anda dapat menyesuaikan perilaku ini dengan mendefinisikan sebuah _closure_ pada metode `renderable` khusus untuk _exception_ `InvalidSignatureException` pada penanganan _exception_ Anda. _Closure_ ini harus mengembalikan respons HTTP:

    use Illuminate\Routing\Exceptions\InvalidSignatureException;

    /**
     * Mendaftarkan callback untuk penanganan exception pada aplikasi.
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
## URL untuk Aksi _Controller_

Fungsi `action` dapat menghasilkan URL untuk aksi pada _controller_ yang ditentukan:

    use App\Http\Controllers\HomeController;

    $url = action([HomeController::class, 'index']);

Jika metode pada _controller_ menerima parameter rute, Anda dapat mengoper _array_ asosiatif untuk parameter rute sebagai argumen kedua untuk fungsi:

    $url = action([UserController::class, 'profile'], ['id' => 1]);

<a name="default-values"></a>
## Nilai _Default_

Untuk beberapa aplikasi, Anda mungkin ingin menentukan nilai _default_ (bawaan) untuk parameter URL tertentu. Misalnya, bayangkan rute Anda mendefinisikan parameter `{locale}` di banyak tempat:

    Route::get('/{locale}/posts', function () {
        //
    })->name('post.index');

Akan sangat merepotkan untuk selalu mengoper `locale` setiap kali Anda memanggil _helper_ `route`. Jadi, Anda dapat menggunakan metode `URL::defaults` untuk mendefinisikan nilai _default_ untuk parameter ini yang akan selalu diterapkan pada permintaan saat ini. Anda mungkin ingin memanggil metode ini dari [_middleware_ rute](/docs/{{version}}/middleware#assigning-middleware-to-routes) sehingga Anda dapat mengakses permintaan saat ini:

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

Setelah nilai _default_ untuk parameter `locale` telah ditetapkan, Anda tidak lagi diharuskan untuk meneruskan nilainya ketika membuat URL melalui _helper_ `route`.

<a name="url-defaults-middleware-priority"></a>
#### _Default_ URL & Prioritas _Middleware_

Pengaturan nilai _default_ URL dapat mengganggu penanganan Laravel terhadap _binding_ model secara implisit. Oleh karena itu, Anda harus [memprioritaskan middleware Anda](/docs/{{version}}/middleware#sorting-middleware) yang mengatur _default_ URL untuk dieksekusi sebelum _middleware_ `SubstituteBindings` Laravel sendiri. Anda dapat mewujudkan hal ini dengan cara memastikan _middleware_ yang diharapkan berjalan sebelum _middleware_ `SubstituteBindings` yang ada di dalam properti `$middlewarePriority` pada kernel HTTP aplikasi Anda.

Properti `$middlewarePriority` didefinisikan dalam kelas dasar `Illuminate\Foundation\Http\Kernel`. Anda dapat menyalin definisinya dari kelas tersebut dan memasukkan/menimpa pada kernel HTTP aplikasi Anda untuk memodifikasinya:

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
