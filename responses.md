# Respons HTTP

- [Membuat Respons](#creating-responses)
    - [Melampirkan _Header_ pada Respons](#attaching-headers-to-responses)
    - [Melampirkan _Cookie_ pada Respons](#attaching-cookies-to-responses)
    - [_Cookie_ & Enkripsi](#cookies-and-encryption)
- [_Redirect_](#redirects)
    - [Melakukan _Redirect_ ke _Named Route_](#redirecting-named-routes)
    - [Melakukan _Redirect_ ke Aksi _Controller_](#redirecting-controller-actions)
    - [Melakukan ke Domain Eksternal](#redirecting-external-domains)
    - [Melakukan _Redirect_ dengan Data _Session_ yang Di-_flash_](#redirecting-with-flashed-session-data)
- [Jenis _Respons_ Lain](#other-response-types)
    - [Respons _View_](#view-responses)
    - [Respons JSON](#json-responses)
    - [Unduhan _File_](#file-downloads)
    - [Respons _File_](#file-responses)
- [_Macro_ Respons](#response-macros)

<a name="creating-responses"></a>
## Membuat Respons

<a name="strings-arrays"></a>
#### _String_ & _Array_

Semua _route_ dan _controller_ seharusnya mengembalikan sebuah respons yang akan dikirimkan ke _browser_ milik pengguna. Laravel menyediakan beragam cara yang berbeda untuk mengembalikan respons. Respons yang paling dasar adalah mengembalikan sebuah _string_ dari _route_ atau _controller_. _Framework_ Laravel secara otomatis akan mengkonversi _string_ tersebut menjadi respons HTTP yang utuh:

    Route::get('/', function () {
        return 'Halo Dunia';
    });

In addition to returning strings from your routes and controllers, you may also return arrays. The framework will automatically convert the array into a JSON response:

Selain mengembalikan _string_ dari _route_ dan _controller_, Anda juga bisa mengembalikan _array_. Laravel secara otomatis akan mengubah _array_ menjadi respons JSON:

    Route::get('/', function () {
        return [1, 2, 3];
    });

> **Catatan**  
> Apa Anda tau bahwa Anda juga bisa mengembalikan sebuah [_collection_ Eloquent](/docs/{{version}}/eloquent-collections) dari _route_ atau _controller_? _Collection_ tersebut akan dikonversi menjadi JSON secara otomatis. Cobalah sekarang!

<a name="response-objects"></a>
#### Objek Respons

Biasanya, Anda tidak hanya mengembalikan _string_ atau _array_ yang polos dari _route_ atau aksi _controller_. Sebaliknya, Anda akan mengembalikan sebuah [_view_](/docs/{{version}}/views) atau sebuah _instance_ dari `Illuminate\Http\Response` secara lengkap.

Mengembalikan sebuah _instance_ `Response` yang lengkap memungkinkan Anda untuk menyesuaikan kode status dan _header_ milik respons HTTP. Sebuah _instance_ `Response` merupakan turunan dari kelas `Symfony\Component\HttpFoundation\Response`, yang menyediakan berbagai metode untuk membangun respons HTTP:

    Route::get('/home', function () {
        return response('Halo Dunia', 200)
                      ->header('Content-Type', 'text/plain');
    });

<a name="eloquent-models-and-collections"></a>
#### Model & _Collection_ Eloquent

Anda juga dapat mengembalikan model [ORM Eloquent](/docs/{{version}}/eloquent) dan _collection_ secara langsung dari _route_ dan _controller_. Ketika Anda melakukan hal tersebut, Laravel secara otomatis akan mengkonversi model dan _collection_ menjadi respons JSON yang tetap mengindahkan [atribut tersembunyi](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json) milik model:

    use App\Models\User;

    Route::get('/user/{user}', function (User $user) {
        return $user;
    });

<a name="attaching-headers-to-responses"></a>
### Melampirkan _Header_ pada Respons

Perlu diingat bahwa kebanyakan metode respons merupakan _chainable_ (dapat dirantaikan), hal tersebut untuk mewujudkan pembangunan _instance_ respons yang _fluent_ (lancar/fasih). Sebagai contoh, Anda dapat menggunakan metode `header` untuk menambahkan serangkaian _header_ pada respons sebelum mengirimkan balik kepada pengguna:

    return response($content)
                ->header('Content-Type', $type)
                ->header('X-Header-Satu', 'Nilai Header')
                ->header('X-Header-Dua', 'Nilai Header');

Atau, Anda dapat menggunakan metode `withHeaders` untuk menentukan _header_ yang akan ditambahkan ke respons dalam bentuk _array_:

    return response($content)
                ->withHeaders([
                    'Content-Type' => $type,
                    'X-Header-Satu' => 'Nilai Header',
                    'X-Header-Dua' => 'Nilai Header',
                ]);

<a name="cache-control-middleware"></a>
#### _Middleware Cache-Control_

Laravel menyertakan sebuah _middleware_ bernama `cache.headers` yang dapat digunakan untuk mengatur _header_ `Cache-Control` pada sekumpulan _route_ dengan singkat. _Directive_ harus disediakan menggunakan "_snake case_" yang setara _directive cache-control_ dan harus dipisahkan dengan tanda titk koma (`;`). Jika `etag` telah dimasukkan ke dalam daftar _directive_, Sebuah _hash_ MD5 dari konten respons akan mengatur ETag secara otomatis:

    Route::middleware('cache.headers:public;max_age=2628000;etag')->group(function () {
        Route::get('/privacy', function () {
            // ...
        });

        Route::get('/terms', function () {
            // ...
        });
    });

<a name="attaching-cookies-to-responses"></a>
### Melampirkan _Cookie_ pada Respons

Anda mungkin ingin melampirkan sebuah _cookie_ pada _instance_ `Illuminate\Http\Response` yang dikirimkan menggunakan metode `cookie`. Anda harus mengoper nama, nilai, dan durasi kadaluarsa _cookie_ dalam satuan menit pada metode tersebut.

    return response('Halo Dunia')->cookie(
        'nama', 'nilai', $durasi
    );

Metode `cookie` juga menerima beberapa argumen lain yang jarang sekali digunakan. Umumnya, argumen-argumen ini memiliki tujuan dan arti yang sama dengan argumen yang akan diberikan ke metode _native_ [setcookie](https://secure.php.net/manual/en/function.setcookie.php) milik PHP:

    return response('Hello World')->cookie(
        'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
    );

Jika Anda ingin melampirkan _cookie_ pada respons yang keluar (dikirimkan ke _browser_ pengguna) tetapi Anda tidak memiliki _instance_ dari respons tersebut, Anda dapat menggunakan _facade_ `Cookie` untuk "mengantrikan" _cookie_ tersebut untuk dilampirkan ke respons ketika dikirim. Metode `queue` menerima argumen yang diperlukan untuk membuat _instance cookie_. _Cookie_ ini akan dilampirkan ke respons keluar sebelum dikirim ke _browser_:

    use Illuminate\Support\Facades\Cookie;

    Cookie::queue('nama', 'nilai', $durasi);

<a name="generating-cookie-instances"></a>
#### Menghasilkan _Instance Cookie_

Jika Anda ingin membuat _instance_ dari `Symfony\Component\HttpFoundation\Cookie` yang bisa dilampirkan pada sebuah _instance_ respons nantinya, Anda dapat menggunakan _helper_ global `cookie`. _Cookie_ tersebut tidak akan dikirim balik kepada klien kecuali telah dilampirkan pada _instance_ respons.

    $cookie = cookie('nama', 'nilai', $durasi);

    return response('Halo Dunia')->cookie($cookie);

<a name="expiring-cookies-early"></a>
#### Membuat _Cookie_ Kedaluwarsa Lebih Awal

Anda dapat menghapus sebuah _cookie_ dengan cara membuatnya kedaluwarsa melalui metode `withoutCookie` pada sebuah respons yang dikembalikan:

    return response('Halo Dunia')->withoutCookie('nama');

Jika Anda tidak memiliki _instance_ untuk respons yang akan dikeluarkan, Anda dapat menggunakan metode `expire` milik _facade_ `Cookie` untuk membuat _cookie_ tersebut kedaluwarsa:

    Cookie::expire('nama');

<a name="cookies-and-encryption"></a>
### _Cookie_ & Enkripsi

Secara _default_, semua _cookie_ yang dihasilkan oleh Laravel telah dienkripsi dan ditandatangani sehingga _cookie_ tersebut tidak dapat dimodifikasi atau dibaca oleh klien. Jika Anda ingin menonaktifkan enkripsi untuk sebuah bagian dari _cookie_ yang telah dihasilkan oleh aplikasi Anda, Anda dapat menggunakan properti `$except` pada _middleware_ `App\Http\Middleware\EncryptCookies`, yang berlokasi pada direktori `app/Http/Middleware`:

    /**
     * Nama-nama _cookie_ yang seharusnya tidak dienkripsi.
     *
     * @var array
     */
    protected $except = [
        'nama_cookie',
    ];

<a name="redirects"></a>
## _Redirect_

Respons _Redirect_ merupakan _instance_ dari kelas `Illuminate\Http\RedirectResponse`, yang mengandung _header_ yang layak untuk melakukan pengalihan (_redirect_) pengguna ke URL lain. Terdapat beberapa cara untuk membuat sebuah _instance_ `RedirectResponse`. Cara yang paling sederhana adalah menggunakan _helper_ global `redirect`:

    Route::get('/dasbor', function () {
        return redirect('beranda/dasbor');
    });

Terkadang Anda ingin mengalihkan pengguna ke lokasi mereka yang sebelumnya, seperti ketika mereka mengirimkan _form_ yang tidak valid. Anda dapat melakukan hal tersebut dengan menggunakan fungsi _helper_ global bernama `back`. Karena fitur ini telah menggunakan [_session_](/docs/{{version}}/session), pastikan _route_ yang memanggil fungsi `back` telah menggunakan grup _middleware_ `web`: 

    Route::post('/pengguna/profil', function () {
        // Validasi request...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
### Melakukan _Redirect_ ke _Named Route_

Ketika Anda memanggil _helper_ `redirect` tanpa parameter, sebuah _instance_ dari `Illuminate\Routing\Redirector` akan dikembalikan, memungkinkan Anda untuk memanggil metode apa saja yang terdapat pada _instance_ `Redirector` tersebut. Sebagai contoh, untuk menghasilkan sebuah `RedirectResponse` ke sebuah _named route_, Anda dapat menggunakan metode `route`:

    return redirect()->route('login');

If your route has parameters, you may pass them as the second argument to the `route` method:

Jika _route_ Anda memiliki parameter, Anda dapat mengopernya pada argumen kedua pada metode `route`:

    // Untuk route yang memiliki URI seperti berikut: /profil/{id}

    return redirect()->route('profil', ['id' => 1]);

<a name="populating-parameters-via-eloquent-models"></a>
#### Mengisi Parameter dengan Model Eloquent

Jika Anda melakukan _redirect_ ke _route_ yang memiliki sebuah "ID" sebagai parameter yang diperoleh dari sebuah model Eloquent, Anda dapat mengoper model itu sendiri. ID dari model tersebut akan diekstraksi secara otomatis:

    // Untuk route yang memiliki URI seperti berikut: /profil/{id}

    return redirect()->route('profil', [$pengguna]);

Jika Anda ingin melakukan kustomisasi nilai yang ingin ditempatkan pada parameter _route_, Anda dapat menetapkan nama kolom pada definisi parameter untuk _route_ tersebut (`/profil/{id:slug}`) atau Anda dapat melakukan _override_ metode `getRouteKey` pada model Eloquent Anda:

    /**
     * Mengambil nilai "route key" milik model.
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
### Melakukan _Redirect_ ke Aksi _Controller_

Anda juga dapat melakukan _redirect_ ke [aksi _Controller_](/docs/{{version}}/controllers). Untuk melakukan hal tersebut, Anda harus mengoper nama _controller_ dan aksinya pada metode _action_:

    use App\Http\Controllers\UserController;

    return redirect()->action([UserController::class, 'index']);

Jika _route controller_ Anda memerlukan parameter, Anda dapat mengopernya pada argumen kedua metode `action`:

    return redirect()->action(
        [UserController::class, 'profil'], ['id' => 1]
    );

<a name="redirecting-external-domains"></a>
### Melakukan _Redirect_ ke Domain Eksternal

Terkadang Anda butuh melakukan _redirect_ ke domain di luar aplikasi Anda. Anda dapat melakukannya dengan memanggil metode `away` yang akan membuat sebuah `RedirectResponse` tanpa pengkodean URL, validasi, atau verifikasi tambahan:

    return redirect()->away('https://www.google.com');

<a name="redirecting-with-flashed-session-data"></a>
### Melakukan _Redirect_ dengan Data _Session_ yang Di-_flash_

Melakukan _redirect_ ke URL lain dan melakukan [_flash_ data pada _session_](/docs/{{version}}/session#flash-data) biasanya dilakukan pada saat yang bersamaan. Umumnya, hal tersebut dilakukan setelah berhasil melakukan sebuah aksi, Anda akan melakukan _flash_ sebuah pesan keberhasilan ke dalam _session_. Untuk kenyamanan, Anda dapat membuat sebuah _instance_ `RedirectResponse` dan melakukan _flash_ data ke _session_ dalam satu rantai metode yang _fluent_:

    Route::post('/pengguna/profil', function () {
        // ...

        return redirect('dasbor')->with('status', 'Profil berhasil diperbaharui!');
    });

Setelah pengguna berhasil dialihkan, Anda dapat menampilkan pesan yang telah di-_flash_ melalui [_session_](/docs/{{version}}/session). Sebagai contoh, dengan menggunakan [sintaks Blade](/docs/{{version}}/blade):

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif

<a name="redirecting-with-input"></a>
#### Melakukan _Redirect_ dengan _Input_

Anda dapat menggunakan metode `withInput` yang disediakan oleh _instance_ `RedirectResponse` untuk melakukan _flash_ data input yang terdapat pada _request_ saat ini ke _session_ sebelum mengalihkan pengguna ke lokasi (URL) yang baru. Hal ini biasanya dilakukan jika pengguna harus menghadapi (halaman) kegagalan validasi. Setelah _input_ di-_flash_ ke _session_, Anda dapat dengan mudah [melakukan _retrieve_](/docs/{{version}}/requests#retrieving-old-input) pada _request_ selanjutnya untuk mengisi ulang formulir (HTML):

    return back()->withInput();

<a name="other-response-types"></a>
## Jenis _Response_ Lainnya

_Helper_ `response` dapat digunakan untuk menghasilkan _instance_ _response_, ketika _helper_ `response` dipanggil tanpa argumen, sebuah implementasi dari `Illuminate\Contracts\Routing\ResponseFactory` akan dikembalikan. _Contract_ ini menyediakan beberapa metode yang berguna untuk melakukan _generate response_.

<a name="view-responses"></a>
### Respons _View_

Jika anda membutuhkan kendali atas status respons dan _header_ namun tetap harus mengembalikan sebuah [_view_](/docs/{{version}}/views) sebagai konten respons, Anda harus menggunakan metode `view`:

    return response()
                ->view('halo', $data, 200)
                ->header('Content-Type', $type);

Tentu saja, jika Anda tidak memerlukan pengoperan kode status HTTP atau _header_ yang _custom_, Anda dapat menggunakan fungsi _helper_ global bernama `view`.

<a name="json-responses"></a>
### Respons JSON

Metode `json` secara otomatis akan menetapkan _header_ `Content-Type` bernilai `application/json`, serta mengonversi _array_ yang diberikan menjado JSON dengan menggunakan fungsi PHP bernama `json_encode`:

    return response()->json([
        'nama' => 'Zain',
        'kota' => 'Surabaya',
    ]);

Jik

Jika Anda ingin membuat respons JSONP, Anda dapat menggunakan metode `json` dengan kombinasi metode `withCallback`:

    return response()
                ->json(['nama' => 'Zain', 'kota' => 'Surabaya'])
                ->withCallback($request->input('callback'));

<a name="file-downloads"></a>
### Unduhan _File_

Metode `download` dapat digunakan untuk menghasilkan respons yang memaksa _browser_ pengguna untuk mengunduh _file_ pada _path_ yang ditentukan. Metode `download` menerima nama _file_ sebagai argumen kedua yang akan menentukan nama _file_ yang akan dilihat pada unduhan _file_ pengguna. Terakhir, Anda dapat mengoper sebuah _array_ dari _header_ HTTP sebagai argumen ketiga pada metode tersebut:

    return response()->download($pathToFile);

    return response()->download($pathToFile, $name, $headers);

> **Peringatan**  
> HttpFoundation milik Symfony yang mengelola pengunduhan _file_ dan memerlukan sebuah _file_ untuk diunduh dengan nama _file_ dalam karakter ASCII.

<a name="streamed-downloads"></a>
#### Unduhan yang Di-_stream_

Terkadang Anda ingin mengubah respons _string_ dari operasi yang ditentukan menjadi respons yang bisa diunduh tanpa harus menuliskan konten tersebut ke dalam _disk_ (menyimpannya dalam bentuk _file_). Anda dapat menggunakan metode `streamDownload` untuk skenario ini. Metode ini menerima _callback_, nama _file_, dan sebuah _array header_ yang opsional sebagai argumennya:

    use App\Services\GitHub;

    return response()->streamDownload(function () {
        echo GitHub::api('repo')
                    ->contents()
                    ->readme('laravel', 'laravel')['contents'];
    }, 'laravel-readme.md');

<a name="file-responses"></a>
### Respons _File_

Metode `file` dapat digunakan untuk menampilkan _file_ seperti gambar atau PDF secara langsung pada _browser_ pengguna daripada harus mengunduhnya. Metode ini menerima _path_ untuk _file_ sebagai argumen pertama dan sebuah _array_ untuk _header_ untuk argumen kedua:

    return response()->file($pathToFile);

    return response()->file($pathToFile, $headers);

<a name="response-macros"></a>
## _Macro_ Respons

Jika Anda ingin mendefinisikan respons _costum_ yang dapat Anda gunakan kembali pada berbagai _route_ dan _controller_, Anda dapat menggunakan metode `macro` pada facade `Response`. Umumnya, Anda harus memanggil metode ini dalam metode `boot` pada salah satu [_service providers_](/docs/{{version}}/providers) milik aplikasi, seperti `App\Providers\AppServiceProvider`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Response;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap service untuk aplikasi.
         *
         * @return void
         */
        public function boot()
        {
            Response::macro('kapital', function ($value) {
                return Response::make(strtoupper($value));
            });
        }
    }

Fungsi `macro` menerima sebuah nama sebagai argumen pertama dan sebuah _closure_ sebagai argumen kedua. _Closure_ tersebut akan dieksekusi ketika memanggil nama _macro_ dari sebuah implementasi `ResponseFactory` atau dari _helper_ `response`:

    return response()->kapital('foo');
