# Perutean

- [Dasar Perutean](#basic-routing)
    - [Rute _Redirect_](#redirect-routes)
    - [Rute _View_](#view-routes)
    - [Daftar _Rute_](#the-route-list)
- [Parameter Rute](#route-parameters)
    - [Parameter Wajib](#required-parameters)
    - [Parameter Opsional](#parameters-optional-parameters)
    - [Pembatasan dengan _Regular Expression_](#parameters-regular-expression-constraints)
- [Penamaan Rute](#named-routes)
- [Grup Rute](#route-groups)
    - [_Middleware_](#route-group-middleware)
    - [_Controller_](#route-group-controllers)
    - [Perutean Subdomain](#route-group-subdomain-routing)
    - [Prefiks Rute](#route-group-prefixes)
    - [Prefiks Nama Rute](#route-group-name-prefixes)
- [Pengikatan Rute dan Model](#route-model-binding)
    - [Pengikatan Implisit](#implicit-binding)
    - [Pengikatan Enum Implisit](#implicit-enum-binding)
    - [Pengikatan Eksplisit](#explicit-binding)
- [Rute _Fallback_](#fallback-routes)
- [Pembatasan Laju](#rate-limiting)
    - [Mendefinisikan Pembatas Laju](#defining-rate-limiters)
    - [Melekatkan Pembatas Laju pada Rute](#attaching-rate-limiters-to-routes)
- [Pemalsuan _Method Form_](#form-method-spoofing)
- [Mengakses Rute Saat Ini](#accessing-the-current-route)
- [_Cross-Origin Resource Sharing_ (CORS)](#cors)
- [Melakukan _Cache_ untuk Rute](#route-caching)

<a name="basic-routing"></a>
## Dasar Perutean

Rute (_route_) Laravel yang paling dasar adalah menerima URI dan _closure_ yang memungkinkan untuk mendefinisikan rute dan perilaku secara sederhana dan ekspresif tanpa _file_ konfigurasi _routing_ yang rumit:

    use Illuminate\Support\Facades\Route;

    Route::get('/greeting', function () {
        return 'Halo Dunia';
    });

<a name="the-default-route-files"></a>
#### _File_ Rute _Default_

Semua rute Laravel didefinisikan dalam _file_ rute Anda, yang terletak di direktori `routes`. _File-file_ ini secara otomatis dimuat oleh `App\Providers\RouteServiceProvider` aplikasi Anda. _File_ `routes/web.php` mendefinisikan rute-rute untuk antarmuka web Anda. Rute-rute ini diberi grup _middleware_ `web`, yang menyediakan fitur-fitur seperti status sesi dan perlindungan CSRF. Rute-rute dalam `routes/api.php` bersifat _stateless_ dan diberi grup _middleware_ `api`.

Untuk sebagian besar aplikasi, Anda akan mulai dengan mendefinisikan rute dalam _file_ `routes/web.php` Anda. Rute-rute yang didefinisikan dalam `routes/web.php` dapat diakses dengan memasukkan URL rute yang didefinisikan di browser Anda. Sebagai contoh, Anda dapat mengakses rute berikut dengan menavigasi ke `http://example.com/user` di browser Anda:

    use App\Http\Controllers\UserController;

    Route::get('/user', [UserController::class, 'index']);

Rute-rute yang didefinisikan dalam file `routes/api.php` telah tertanam di dalam grup rute `RouteServiceProvider`. Di dalam grup ini, awalan URI `/api` secara otomatis diterapkan sehingga Anda tidak perlu menerapkannya secara manual ke setiap rute dalam _file_. Anda dapat memodifikasi prefiks dan opsi grup rute lainnya dengan memodifikasi kelas `RouteServiceProvider` Anda.

<a name="available-router-methods"></a>
#### _Method_ _Router_ yang Tersedia

_Router_ memungkinkan Anda untuk mendaftarkan rute yang merespon setiap verba HTTP:

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

Terkadang Anda mungkin perlu mendaftarkan rute yang merespon beberapa verba HTTP. Anda dapat melakukannya dengan menggunakan metode `match`. Atau, Anda bahkan dapat mendaftarkan rute yang merespon pada semua verba HTTP menggunakan metode `any`:

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('/', function () {
        //
    });

> **Catatan**  
> Ketika mendefinisikan beberapa rute yang memiliki URI yang sama, rute yang menggunakan metode `get`, `post`, `put`, `patch`, `delete`, dan `options` harus didefinisikan sebelum rute yang menggunakan metode `any`, `match`, dan `redirect`. Hal ini memastikan permintaan yang masuk cocok dengan rute yang benar.

<a name="dependency-injection"></a>
#### Injeksi Dependensi

Anda dapat memberikan _tipe-hint_ terhadap kebutuhan dependensi yang dibutuhkan oleh rute Anda dalam _signature_ _callback_ rute Anda. Dependensi yang dideklarasikan akan secara otomatis diselesaikan dan diinjeksikan ke callback oleh [_service container_](/docs/{{version}}/container) milik Laravel. Sebagai contoh, Anda dapat memberikan _tipe-hint_ pada kelas `Illuminate\Http\Request` agar permintaan HTTP saat ini secara otomatis diinjeksikan ke callback rute Anda:

    use Illuminate\Http\Request;

    Route::get('/users', function (Request $request) {
        // ...
    });

<a name="csrf-protection"></a>
#### Proteksi CSRF

Ingat, setiap _form_ HTML yang menunjuk ke rute `POST`, `PUT`, `PATCH`, atau `DELETE` yang didefinisikan dalam _file_ rute `web` harus menyertakan token CSRF. Jika tidak, permintaan akan ditolak. Anda dapat membaca lebih lanjut tentang perlindungan CSRF di [Dokumentasi CSRF](/docs/{{version}}/csrf):

    <form method="POST" action="/profile">
        @csrf
        ...
    </form>

<a name="redirect-routes"></a>
### Redirect Routes

Jika Anda mendefinisikan rute yang mengarah ke URI lain, Anda dapat menggunakan metode Route::redirect. Metode ini memberikan pintasan yang praktis sehingga Anda tidak perlu mendefinisikan rute penuh atau kontroller untuk melakukan redirect sederhana:

Jika Anda mendefinisikan rute yang mengarahkan ke URI lain, Anda dapat menggunakan metode `Rute::redirect`. Metode ini menyediakan jalan pintas yang nyaman sehingga Anda tidak perlu mendefinisikan rute penuh atau rute _controller_ untuk melakukan _redirect_ sederhana:

    Route::redirect('/here', '/there');

Secara default, `Route::redirect` mengembalikan kode status `302`. Anda dapat menyesuaikan kode status menggunakan parameter ketiga yang opsional:

    Route::redirect('/here', '/there', 301);

Atau, Anda dapat menggunakan metode `Route::permanentRedirect` untuk mengembalikan kode status `301`:

    Route::permanentRedirect('/here', '/there');

> **Peringatan**  
> Ketika menggunakan parameter rute dalam rute _redirect_, parameter berikut dicadangkan oleh Laravel dan tidak dapat digunakan: `destination` dan `status`.

<a name="view-routes"></a>
### Rute _View_

Jika rute Anda hanya perlu mengembalikan [_view_](/docs/{{version}}/views), Anda dapat menggunakan metode `Rute::view`. Seperti metode `redirect`, metode ini menyediakan jalan pintas sederhana sehingga Anda tidak perlu mendefinisikan rute penuh atau rute _controller_. Metode `view` menerima URI sebagai argumen pertama dan nama _view_ sebagai argumen kedua. Selain itu, Anda dapat menyediakan _array_ data untuk diteruskan ke _view_ sebagai argumen ketiga yang opsional:

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

> **Peringatan**  
> Ketika menggunakan parameter rute dalam rute _view_, parameter berikut dicadangkan oleh Laravel dan tidak dapat digunakan: `view`, `data`, `status`, dan `headers`.

<a name="the-route-list"></a>
### Daftar Rute

Perintah Artisan `route:list` dapat dengan mudah memberikan gambaran umum dari semua rute yang telah didefinisikan dalam aplikasi Anda:

```shell
php artisan route:list
```

Secara _default_, _middleware_ rute yang ditugaskan ke setiap rute tidak akan ditampilkan dalam output `route:list`; namun, Anda dapat menginstruksikan Laravel untuk menampilkan _middleware_ rute dengan menambahkan opsi `-v` ke perintah:

```shell
php artisan route:list -v
```

Anda juga dapat menginstruksikan Laravel untuk hanya menampilkan rute yang dimulai dengan URI tertentu:

```shell
php artisan route:list --path=api
```

Selain itu, Anda dapat menginstruksikan Laravel untuk menyembunyikan rute apa pun yang didefinisikan oleh _package_ dari pihak ketiga dengan memberikan opsi `--except-vendor` saat menjalankan perintah `route:list`:

```shell
php artisan route:list --except-vendor
```

Demikian juga, Anda juga dapat menginstruksikan Laravel untuk hanya menampilkan rute-rute yang didefinisikan oleh paket-paket pihak ketiga dengan memberikan opsi `--only-vendor` ketika menjalankan perintah `route:list`:

```shell
php artisan route:list --only-vendor
```

<a name="route-parameters"></a>
## Parameter Rute

<a name="required-parameters"></a>
### Parameter Wajib

Terkadang Anda perlu menangkap segmen URI dalam rute Anda. Misalnya, Anda mungkin perlu menangkap ID pengguna dari URL. Anda dapat melakukannya dengan mendefinisikan parameter rute:

    Route::get('/user/{id}', function ($id) {
        return 'User '.$id;
    });

Anda dapat mendefinisikan parameter rute sebanyak yang apapun diperlukan pada rute Anda:

    Route::get('/posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

Parameter rute selalu terbungkus dalam tanda kurung kurawal `{}` dan harus terdiri dari karakter alfabet. Underscore (`_`) juga dapat diterima dalam nama parameter rute. Parameter rute diinjeksikan ke dalam rute _callback_ / _controller_ berdasarkan urutannya - nama argumen rute _callback_ / _controller_ tidak menjadi masalah.

<a name="parameters-and-dependency-injection"></a>
#### Injeksi Parameter & Dependensi

Jika rute Anda memiliki dependensi yang ingin disuntikkan secara otomatis ke _callback_ milik rute oleh _service container_ Laravel, Anda sebaiknya mencantumkan parameter rute setelah dependensi Anda:

    use Illuminate\Http\Request;

    Route::get('/user/{id}', function (Request $request, $id) {
        return 'User '.$id;
    });

<a name="parameters-optional-parameters"></a>
### Parameter Opsional

Terkadang Anda mungkin perlu menentukan parameter rute yang mungkin tidak selalu ada di URI. Anda dapat melakukannya dengan menempatkan tanda `?` setelah nama parameter. Pastikan untuk memberikan nilai _default_ pada variabel yang sesuai dengan rute:

    Route::get('/user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('/user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>
### Pembatasan dengan _Regular Expression_

Anda dapat membatasi format parameter rute Anda dengan menggunakan metode `where` pada _instance_ rute. Metode `where` menerima nama parameter dan ekspresi reguler yang menjelaskan bagaimana parameter harus dibatasi:

    Route::get('/user/{name}', function ($name) {
        //
    })->where('name', '[A-Za-z]+');

    Route::get('/user/{id}', function ($id) {
        //
    })->where('id', '[0-9]+');

    Route::get('/user/{id}/{name}', function ($id, $name) {
        //
    })->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

Untuk kenyamanan, beberapa pola ekspresi reguler yang sering digunakan memiliki metode pembantu yang memungkinkan Anda untuk dengan cepat menambahkan pola batasan ke rute Anda:

    Route::get('/user/{id}/{name}', function ($id, $name) {
        //
    })->whereNumber('id')->whereAlpha('name');

    Route::get('/user/{name}', function ($name) {
        //
    })->whereAlphaNumeric('name');

    Route::get('/user/{id}', function ($id) {
        //
    })->whereUuid('id');

    Route::get('/user/{id}', function ($id) {
        //
    })->whereUlid('id');

    Route::get('/category/{category}', function ($category) {
        //
    })->whereIn('category', ['movie', 'song', 'painting']);

Jika permintaan yang masuk tidak cocok dengan batasan pola rute, maka akan dikembalikan respons HTTP 404.

<a name="parameters-global-constraints"></a>
#### Pembatasan Global

Jika Anda ingin parameter rute selalu dibatasi oleh ekspresi reguler yang diberikan, Anda dapat menggunakan metode `pattern`. Anda harus mendefinisikan pola-pola ini dalam metode `boot` dari kelas `App\Providers\RouteServiceProvider`:

    /**
     * Menentukan binding model rute Anda, filter pola, dll.
     *
     * @return void
     */
    public function boot()
    {
        Route::pattern('id', '[0-9]+');
    }

Setelah pola telah didefinisikan, secara otomatis diterapkan pada semua rute yang menggunakan nama parameter tersebut:

    Route::get('/user/{id}', function ($id) {
        // Only executed if {id} is numeric...
    });

<a name="parameters-encoded-forward-slashes"></a>
#### Garis miring yang Dienkode

Komponen routing Laravel memperbolehkan semua karakter kecuali `/` untuk hadir di dalam nilai parameter rute. Anda harus secara eksplisit memperbolehkan `/` untuk menjadi bagian dari _placeholder_ Anda menggunakan ekspresi reguler kondisi `where`:

    Route::get('/search/{search}', function ($search) {
        return $search;
    })->where('search', '.*');

> **Perhatian**  
> Garis miring yang dienkode hanya didukung dalam segmen rute terakhir.

<a name="named-routes"></a>
## Rute Bernama

Rute yang diberi nama memungkinkan pembuatan URL atau _redirect_ yang nyaman untuk rute tertentu. Anda dapat menentukan nama untuk rute dengan menyambungkan metode `name` pada pendefinisian rute:

    Route::get('/user/profile', function () {
        //
    })->name('profile');

Anda juga dapat menentukan nama rute untuk aksi _controller_:

    Route::get(
        '/user/profile',
        [UserProfileController::class, 'show']
    )->name('profile');

> **Perhatian**  
> Nama rute sebaiknya selalu unik.

<a name="generating-urls-to-named-routes"></a>

Menghasilkan URL ke Rute yang Diberi Nama.

#### _Generate_ URL Berdasarkan Nama Rute

Setelah Anda memberikan nama ke rute tertentu, Anda dapat menggunakan nama rute tersebut saat menghasilkan URL atau _redirect_ melalui fungsi helper `route` dan `redirect`:

    // Menghasilkan URL...
    $url = route('profile');

    // Menghasilkan Redirect...
    return redirect()->route('profile');

    return to_route('profile');

Jika terdapat paramete pada rute yang diberi nama, Anda dapat mengirimkan parameter sebagai argumen kedua pada fungsi `route`. Parameter yang diberikan akan otomatis dimasukkan ke dalam URL yang dihasilkan pada posisi yang tepat:

    Route::get('/user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

Jika Anda mengirimkan parameter tambahan dalam _array_, pasangan kunci dan nilai tersebut akan otomatis ditambahkan ke _query string_ URL yang dihasilkan:

    Route::get('/user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1, 'photos' => 'yes']);

    // /user/1/profile?photos=yes

> **Catatan**  

> Terkadang, Anda mungkin ingin menentukan nilai _default_ untuk parameter URL, seperti _locale_ saat ini. Untuk mencapai ini, Anda dapat menggunakan metode [`URL::defaults` method](/docs/{{version}}/urls#default-values).

<a name="inspecting-the-current-route"></a>
#### Inspeksi Rute Saat Ini

Jika Anda ingin menentukan apakah _request_ saat ini diarahkan ke nama rute yang sudah ditentukan, Anda dapat menggunakan metode `named` pada _instance_ Route. Sebagai contoh, Anda dapat memeriksa nama rute saat ini dari _middleware_ rute:

If you would like to determine if the current request was routed to a given named route, you may use the `named` method on a Route instance. For example, you may check the current route name from a route middleware:

    /**
     * Menangani request yang dikirimkan.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->route()->named('profile')) {
            //
        }

        return $next($request);
    }

<a name="route-groups"></a>
## Grup Rute

Grup rute memungkinkan Anda untuk membagikan atribut-atribut rute, seperti middleware, untuk beberapa rute tanpa perlu mendefinisikan atribut-atribut tersebut pada setiap rute satu-persatu.

Grup di dalam grup digunakan untuk "mengabungkan" atribut secara cerdas dengan kelompok induknya. _Middleware_ dan kondisi `where` dapat digabungkan saat nama dan prefiks ditambahkan. Pemisah _namespace_ dan garis miring di prefiks URI secara otomatis ditambahkan di tempat yang tepat.

<a name="route-group-middleware"></a>
### _Middleware_

Untuk menetapkan [_middleware_](/docs/{{version}}/middleware) pada semua rute dalam sebuah grup, Anda dapat menggunakan metode `middleware` sebelum mendefinisikan grup. _Middleware_ dieksekusi dalam urutan yang tercantum dalam _array_:

    Route::middleware(['pertama', 'kedua'])->group(function () {
        Route::get('/', function () {
            // Menggunakan middleware pertama & kedua...
        });

        Route::get('/user/profile', function () {
            // Menggunakan middleware pertama & kedua...
        });
    });

<a name="route-group-controllers"></a>
### _Controller_

Jika sekumpulan rute menggunakan [_controller_](/docs/{{version}}/controllers) yang sama, Anda dapat menggunakan metode `controller` untuk menentukan _controller_ untuk semua rute dalam kelompok. Kemudian, saat mendefinisikan rute, Anda hanya perlu memberikan metode _controller_ yang ingin dipanggil:

    use App\Http\Controllers\OrderController;

    Route::controller(OrderController::class)->group(function () {
        Route::get('/orders/{id}', 'show');
        Route::post('/orders', 'store');
    });

<a name="route-group-subdomain-routing"></a>
### Perutean _Subdomain_

Kelompok rute juga dapat digunakan untuk menangani perutean _subdomain_. _Subdomain_ dapat ditetapkan parameter rute seperti URI rute, sehingga Anda dapat menangkap bagian dari _subdomain_ untuk digunakan dalam rute atau _controller_ Anda. _Subdomain_ dapat ditentukan dengan memanggil metode `domain` sebelum mendefinisikan grup:

    Route::domain('{account}.example.com')->group(function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

> **Perhatian**  
> Untuk memastikan rute subdomain Anda dapat dijangkau, Anda sebaiknya mendaftarkan rute subdomain sebelum rute domain _root_/utama. Ini akan mencegah rute domain _root_ menimpa rute subdomain yang memiliki jalur URI yang sama.

<a name="route-group-prefixes"></a>
### Prefiks Rute

Metode `prefix` dapat digunakan untuk memberikan prefiks pada setiap rute dalam grup dengan URI tertentu. Sebagai contoh, Anda mungkin ingin memberikan prefiks pada semua URI rute dalam grup dengan `admin`:

    Route::prefix('admin')->group(function () {
        Route::get('/users', function () {
            // URL akan menjadi "/admin/users"
        });
    });

<a name="route-group-name-prefixes"></a>
### Prefiks Nama Rute

Metode `name` dapat digunakan untuk memberikan prefiks pada setiap nama rute di dalam grup dengan _string_ tertentu. Sebagai contoh, Anda mungkin ingin memberikan prefiks pada semua nama rute di dalam grup dengan nama `admin`. _String_ tersebut ditambahkan ke nama rute seperti yang sudah ditentukan, jadi kami akan pastikan untuk memberikan karakter `.` pada akhir prefiks:

    Route::name('admin.')->group(function () {
        Route::get('/users', function () {
            // Nama rute akan menjadi: "admin.users"...
        })->name('users');
    });

<a name="route-model-binding"></a>
## Pengikatan Rute dan Model

Saat menginjeksi ID model ke rute atau _action_ dari _controller_, Anda akan sering melakukan kueri basis data untuk mengambil model yang sesuai dengan ID tersebut. Untuk pengikatan (_Binding_) rute-model, Laravel menyediakan cara yang mudah untuk menginjeksi _instance_ model langsung ke rute Anda secara otomatis. Misalnya, alih-alih menginjeksi ID pengguna, Anda dapat menginjeksi seluruh _instance_ model `User` yang sesuai dengan ID yang diberikan.

<a name="implicit-binding"></a>
### Pengikatan Implisit

Laravel secara otomatis memberikan model Eloquent yang didefinisikan dalam rute atau _action_ milik _controller_ yang nama variabel dan _type-hint_-nya serupa dengan nama segmen rute. Sebagai contoh:

    use App\Models\User;

    Route::get('/users/{user}', function (User $user) {
        return $user->email;
    });

Karena variabel `$user` diisyaratkan sebagai model Eloquent `App\Models\User` dan nama variabel cocok dengan segmen URI `{user}`, Laravel akan secara otomatis menginjeksikan _instance_ model yang memiliki ID yang cocok dengan nilai yang sesuai dari _request_ URI. Jika _instance_ model tidak ditemukan pada basis data, respons HTTP 404 akan secara otomatis dihasilkan.

Tentu saja, pengikatan implisit juga dimungkinkan ketika menggunakan metode di dalam _controller_. Sekali lagi, perhatikan segmen URI `{user}` sesuai dengan variabel `$user` di controller yang berisi _type-hint_ `App\Models\User`:

    use App\Http\Controllers\UserController;
    use App\Models\User;

    // Pendefinisian rute...
    Route::get('/users/{user}', [UserController::class, 'show']);

    // Pendefinisian metode pada controller...
    public function show(User $user)
    {
        return view('user.profile', ['user' => $user]);
    }

<a name="implicit-soft-deleted-models"></a>
#### Model yang Telah Soft Delete

Biasanya, _binding_ model implisit tidak akan mengambil model yang telah dihapus secara [soft](/docs/{{version}}/eloquent#soft-deleting). Namun, Anda dapat menyuruh _binding_ implisit untuk mengambil model-model ini dengan merantai _method_ `withTrashed` pada definisi rute Anda:

    use App\Models\User;

    Route::get('/users/{user}', function (User $user) {
        return $user->email;
    })->withTrashed();

<a name="customizing-the-key"></a>
<a name="customizing-the-default-key-name"></a>
#### Kostumisasi Kunci

Kadang-kadang Anda mungkin ingin mengambil model Eloquent menggunakan kolom selain `id`. Untuk melakukannya, Anda dapat menentukan kolom dalam pendefinisian parameter untuk rute:

    use App\Models\Post;

    Route::get('/posts/{post:slug}', function (Post $post) {
        return $post;
    });

Jika Anda ingin model yang terikat selalu menggunakan kolom _database_ selain `id` saat mengambil kelas model tertentu, Anda dapat menimpa _method_ `getRouteKeyName` pada model Eloquent:

    /**
     * mendapatkan kunci rute untuk model.
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

<a name="implicit-model-binding-scoping"></a>
#### Custom Keys & Scoping

Saat melakukan _binding_ beberapa model Eloquent dalam satu definisi rute secara implisit, Anda mungkin ingin membatasi model Eloquent kedua sehingga harus menjadi anak dari model Eloquent sebelumnya. Sebagai contoh, pertimbangkan definisi rute berikut yang mengambil _posting_-an blog dengan _slug_ untuk pengguna tertentu:

    use App\Models\Post;
    use App\Models\User;

    Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
        return $post;
    });

Ketika menggunakan _binding_ implisit kustom yang ditandai sebagai parameter rute bersarang, Laravel secara otomatis akan membatasi _query_ untuk mengambil model bersarang dengan _parent_ menggunakan konvensi untuk menebak nama hubungan pada parent. Dalam hal ini, akan diasumsikan bahwa model `User` memiliki hubungan yang diberi nama `posts` (bentuk jamak dari nama parameter rute) yang dapat digunakan untuk mengambil model `Post`.

Jika Anda ingin agar Laravel mengikat "anak" secara terbatas bahkan tanpa kunci khusus, Anda dapat memanggil metode `scopeBindings` saat mendefinisikan rute Anda:

    use App\Models\Post;
    use App\Models\User;

    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        return $post;
    })->scopeBindings();

Atau, Anda dapat memberitahu seluruh definisi rute pada grup untuk menggunakan _binding_ yang terbatas:

    Route::scopeBindings()->group(function () {
        Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
            return $post;
        });
    });

Anda juga dapat secara eksplisit menyuruh Laravel untuk tidak melakukan pembatasan dengan menggunakan _method_ `withoutScopedBindings`:

    Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
        return $post;
    })->withoutScopedBindings();

<a name="customizing-missing-model-behavior"></a>
#### Menyesuaikan Perilaku Model yang Tidak Ada

Biasanya, respons HTTP 404 akan dihasilkan jika model yang terikat secara implisit tidak ditemukan. Namun, Anda dapat menyesuaikan perilaku ini dengan memanggil metode `missing` ketika mendefinisikan rute Anda. Metode `missing` menerima _closure_ yang akan dipanggil jika model yang terikat secara implisit tidak dapat ditemukan:

    use App\Http\Controllers\LocationsController;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Redirect;

    Route::get('/locations/{location:slug}', [LocationsController::class, 'show'])
            ->name('locations.view')
            ->missing(function (Request $request) {
                return Redirect::route('locations.index');
            });

<a name="implicit-enum-binding"></a>
### Enum _Binding_ Implisit

PHP 8.1 memperkenalkan dukungan untuk [Enum](https://www.php.net/manual/en/language.enumerations.backed.php). Untuk melengkapi fitur ini, Laravel memungkinkan Anda untuk _type-hint_ [string-backed Enum](https://www.php.net/manual/en/language.enumerations.backed.php) pada definisi rute Anda dan Laravel hanya akan memanggil rute jika segmen rute tersebut sesuai dengan nilai Enum yang valid. Jika tidak, respons HTTP 404 akan dikembalikan secara otomatis. Sebagai contoh, diberikan Enum sebagai berikut:

```php
<?php

namespace App\Enums;

enum Category: string
{
    case Fruits = 'fruits';
    case People = 'people';
}
```

Anda dapat mendefinisikan rute yang hanya akan dipanggil jika segmen rute `{category}` adalah `fruits` atau `people`. Jika tidak, Laravel akan mengembalikan respons HTTP 404:

```php
use App\Enums\Category;
use Illuminate\Support\Facades\Route;

Route::get('/categories/{category}', function (Category $category) {
    return $category->value;
});
```

<a name="explicit-binding"></a>
### Pengikatan Eksplisit

Anda tidak diharuskan menggunakan resolusi model implisit berbasis konvensi Laravel untuk menggunakan model _binding_. Anda juga dapat secara eksplisit mendefinisikan bagaimana parameter rute dapat koresponden dengan model. Untuk mendaftarkan pengikatan eksplisit, gunakan metode `model` router untuk menentukan kelas untuk parameter yang diberikan. Anda harus mendefinisikan binding model eksplisit Anda di awal metode `boot` dari kelas `RouteServiceProvider` Anda:

    use App\Models\User;
    use Illuminate\Support\Facades\Route;

    /**
     * Menentukan binding model rute Anda, filter pola, dll.
     *
     * @return void
     */
    public function boot()
    {
        Route::model('user', User::class);

        // ...
    }

Selanjutnya, definisikan sebuah rute yang memiliki parameter `{user}`:

    use App\Models\User;

    Route::get('/users/{user}', function (User $user) {
        //
    });

Karena kita telah mengikat semua parameter `{user}` ke model `App\Models\User`, sebuah instance dari kelas tersebut akan diinjeksi ke dalam rute. Jadi, misalnya, permintaan ke `users/1` akan menginjeksi instance `User` dari database yang memiliki ID `1`.

Jika _instance_ model yang cocok tidak ditemukan dalam _database_, respons HTTP 404 akan dihasilkan secara otomatis.

<a name="customizing-the-resolution-logic"></a>
#### Menyesuaikan Logika Resolusi

Jika Anda ingin mendefinisikan logika resolusi pengikatan model Anda sendiri, Anda dapat menggunakan metode `Rute::bind`. _Closure_ yang Anda berikan ke metode `bind` akan menerima nilai segmen URI dan harus mengembalikan instance dari kelas yang harus diinjeksikan ke dalam rute. Sekali lagi, kustomisasi ini harus dilakukan dalam metode `boot` pada `RouteServiceProvider` aplikasi Anda:

    use App\Models\User;
    use Illuminate\Support\Facades\Route;

    /**
     * Menentukan binding model rute Anda, filter pola, dll.
     *
     * @return void
     */
    public function boot()
    {
        Route::bind('user', function ($value) {
            return User::where('name', $value)->firstOrFail();
        });

        // ...
    }

Sebagai alternatif, Anda dapat meng-_override_ metode `resolveRouteBinding` pada model Eloquent Anda. Metode ini akan menerima nilai segmen URI dan harus mengembalikan instance dari kelas yang harus diinjeksikan ke dalam rute:

    /**
     * Mengambil model berdasarkan nilai yang diberikan.
     *
     * @param  mixed  $value
     * @param  string|null  $field
     * @return \Illuminate\Database\Eloquent\Model|null
     */
    public function resolveRouteBinding($value, $field = null)
    {
        return $this->where('name', $value)->firstOrFail();
    }

Jika sebuah rute menggunakan [_implicit binding scoping_](#implicit-model-binding-scoping), metode `resolveChildRouteBinding` akan digunakan untuk menyelesaikan pengikatan anak dari model induk:

    /**
     * Mengambil model anak berdasarkan nilai yang diberikan.
     *
     * @param  string  $childType
     * @param  mixed  $value
     * @param  string|null  $field
     * @return \Illuminate\Database\Eloquent\Model|null
     */
    public function resolveChildRouteBinding($childType, $value, $field)
    {
        return parent::resolveChildRouteBinding($childType, $value, $field);
    }

<a name="fallback-routes"></a>
## _Route Fallback_

Dengan menggunakan metode `Route::fallback`, Anda dapat mendefinisikan rute yang akan dieksekusi ketika tidak ada rute lain yang cocok dengan permintaan yang masuk. Biasanya, permintaan yang tidak tertangani akan secara otomatis me-_render_ halaman "404" melalui _exception handler_ pada aplikasi Anda. Namun, karena Anda biasanya akan mendefinisikan rute `fallback` di dalam _file_ `routes/web.php` Anda, semua _middleware_ pada grup _middleware_ `web` akan menerapkan hal tersebut. Anda bebas menambahkan _middleware_ tambahan ke rute ini sesuai kebutuhan:

    Route::fallback(function () {
        //
    });

> **Peringatan**  
> Rute fallback harus selalu merupakan rute terakhir yang didaftarkan pada aplikasi Anda.

<a name="rate-limiting"></a>
## Pembatasan _Rate_

<a name="defining-rate-limiters"></a>
### Defining Rate Limiters

Laravel menyertakan layanan pembatasan _rate_ (laju) yang kuat dan dapat disesuaikan dan dapat Anda manfaatkan untuk membatasi jumlah lalu lintas untuk rute atau grup rute tertentu. Untuk memulai, Anda harus melakukan konfigurasi pembatas _rate_ yang memenuhi kebutuhan aplikasi Anda. Biasanya, ini harus dilakukan dalam metode `configureRateLimiting` dari kelas `App\Providers\RouteServiceProvider` aplikasi Anda.

Pembatas _rate_ didefinisikan menggunakan metode `for` facade `RateLimiter`. Metode `for` menerima nama pembatas laju dan _closure_ yang mengembalikan konfigurasi batas yang harus diterapkan ke rute yang ditugaskan ke pembatas laju. Konfigurasi _limit_ adalah _instance_ dari kelas `Illuminate\Cache\RateLimiting\Limit`. Kelas ini berisi metode-metode "builder" yang berguna sehingga Anda dapat dengan cepat mendefinisikan limit Anda. Nama pembatas _rate_ dapat berupa _string_ apa pun yang Anda inginkan:

    use Illuminate\Cache\RateLimiting\Limit;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\RateLimiter;

    /**
     * Konfigurasi pembatas laju untuk aplikasi.
     *
     * @return void
     */
    protected function configureRateLimiting()
    {
        RateLimiter::for('global', function (Request $request) {
            return Limit::perMinute(1000);
        });
    }

Jika permintaan yang masuk melebihi batas laju yang ditentukan, respons dengan kode status HTTP 429 akan secara otomatis dikembalikan oleh Laravel. Jika Anda ingin menentukan respons Anda sendiri yang harus dikembalikan oleh pembatas laju, Anda dapat menggunakan metode `response`:

    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000)->response(function (Request $request, array $headers) {
            return response('Respojlns kustom...', 429, $headers);
        });
    });

Karena _callback_ dari pembatas laju menerima _instance_ permintaan HTTP yang masuk, Anda dapat membuat batas laju yang dinamis berdasarkan permintaan yang masuk atau pengguna yang diautentikasi:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()->vipCustomer()
                    ? Limit::none()
                    : Limit::perMinute(100);
    });

<a name="segmenting-rate-limits"></a>
#### Segmen Batas Kecepatan

Kadang-kadang, Anda mungkin ingin membagi batas kecepatan dengan beberapa nilai yang sewenang-wenang. Misalnya, anda mungkin ingin mengizinkan pengguna untuk mengakses rute tertentu 100 kali per menit per alamat IP. Untuk mencapai hal ini, anda dapat menggunakan metode `by` ketika membangun _rate limit_ anda:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()->vipCustomer()
                    ? Limit::none()
                    : Limit::perMinute(100)->by($request->ip());
    });

Untuk mengilustrasikan fitur ini dengan menggunakan contoh lain, kita bisa membatasi akses ke rute hingga 100 kali per menit per ID pengguna yang diautentikasi atau 10 kali per menit per alamat IP untuk tamu:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()
                    ? Limit::perMinute(100)->by($request->user()->id)
                    : Limit::perMinute(10)->by($request->ip());
    });

<a name="multiple-rate-limits"></a>
#### Batas Kecepatan Berganda

Jika diperlukan, Anda dapat mengembalikan _array_ batas laju untuk konfigurasi pembatas tarif yang diberikan. Setiap _rate limit_ akan dievaluasi untuk rute berdasarkan urutan penempatannya dalam _array_:

    RateLimiter::for('login', function (Request $request) {
        return [
            Limit::perMinute(500),
            Limit::perMinute(3)->by($request->input('email')),
        ];
    });

<a name="attaching-rate-limiters-to-routes"></a>
### Melekatkan Pembatas Laju pada Rute

Pembatas laju dapat dilampirkan ke rute atau grup rute menggunakan [_middleware_](/docs/{{version}}/mmiddleware) `throttle`. _Middleware throttle_ menerima nama pembatas laju yang ingin Anda tetapkan ke rute:

    Route::middleware(['throttle:uploads'])->group(function () {
        Route::post('/audio', function () {
            //
        });

        Route::post('/video', function () {
            //
        });
    });

<a name="throttling-with-redis"></a>
#### Perlambatan Menggunakan Redis

Biasanya, middleware `throttle` dipetakan ke kelas `Illuminate\Routing\Middleware\ThrottleRequests`. Pemetaan ini didefinisikan dalam kernel HTTP aplikasi Anda (`App\Http\Kernel`). Namun, jika Anda menggunakan Redis sebagai _driver cache_ aplikasi Anda, Anda mungkin ingin mengubah pemetaan ini untuk menggunakan kelas `Illuminate\Routing\Middleware\ThrottleRequestsWithRedis`. Kelas ini lebih efisien dalam mengelola pembatasan laju menggunakan Redis:

    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,

<a name="form-method-spoofing"></a>
## _Form Method_ Imitasi

Formulir HTML tidak mendukung aksi `PUT`, `PATCH`, atau `DELETE`. Jadi, ketika mendefinisikan rute `PUT`, `PATCH`, atau `DELETE` yang dipanggil dari _form_ HTML, Anda perlu menambahkan field `_method` yang tersembunyi pada _form_. Nilai yang dikirim dengan field `_method` akan digunakan sebagai metode permintaan HTTP:

    <form action="/example" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

Untuk kenyamanan, anda dapat menggunakan [Blade directive](/docs/{{version}}/blade) `@method` untuk menghasilkan kolom _input_ `_method`:

    <form action="/example" method="POST">
        @method('PUT')
        @csrf
    </form>

<a name="accessing-the-current-route"></a>
## Mengakses Rute Saat Ini

Anda dapat menggunakan metode `current`, `currentRouteName`, dan `currentRouteAction` pada facade `Route` untuk mengakses informasi tentang rute yang menangani permintaan yang masuk:

    use Illuminate\Support\Facades\Route;

    $route = Route::current(); // Illuminate\Routing\Route
    $name = Route::currentRouteName(); // string
    $action = Route::currentRouteAction(); // string

Anda dapat merujuk ke dokumentasi API untuk [kelas yang mendasari _facade_ _Route_](https://laravel.com/api/{{{version}}/Illuminate/Routing/Router.html) dan [_instance Route_](https://laravel.com/api/{{{version}}/Illuminate/Routing/Route.html) untuk meninjau semua metode yang tersedia pada kelas Router dan Route.

<a name="cors"></a>
## Cross-Origin Resource Sharing (CORS)

Laravel dapat secara otomatis merespons permintaan HTTP CORS `OPTIONS` dengan nilai-nilai yang Anda konfigurasikan. Semua pengaturan CORS dapat dikonfigurasi dalam file konfigurasi `config/cors.php` aplikasi Anda. Permintaan `OPTIONS` secara otomatis akan ditangani oleh `HandleCors` [_middleware_](/docs/{{version}}/middleware) yang disertakan secara _default_ dalam tumpukan _middleware_ global Anda. Tumpukan _middleware_ global Anda terletak pada kernel HTTP aplikasi Anda (`App\Http\Kernel`).

> **Catatan**  

> Untuk informasi lebih lanjut tentang CORS dan header CORS, silakan mengunjungi [dokumentasi web MDN tentang CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#The_HTTP_response_headers).

<a name="route-caching"></a>
## Melakukan _Cache Route_

Saat men-_delpoy_ aplikasi Anda ke lingkungan _production_, Anda harus memanfaatkan _cache_ rute Laravel. Menggunakan _cache_ rute akan mengurangi jumlah waktu yang diperlukan untuk mendaftarkan semua rute aplikasi Anda secara drastis. Untuk membuat _cache_ rute, jalankan perintah Artisan `route:cache`:

```shell
php artisan route:cache
```

Setelah menjalankan perintah ini, _file_ rute yang telah di-_cache_ akan dimuat pada setiap permintaan. Ingat, jika Anda menambahkan rute baru, Anda perlu melakukan cache ulang. Karena itu, Anda sebaiknya hanya menjalankan perintah `route:cache` pada saat _deployment_ saja.

Anda dapat menggunakan perintah `route:clear` untuk menghapus _cache rute_:

```shell
php artisan route:clear
```
