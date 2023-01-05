# Perutean

- [Dasar Perutean](#basic-routing)
    - [_Route Redirect_](#redirect-routes)
    - [_Route View_](#view-routes)
    - [_Daftar Route_](#the-route-list)
- [Parameter _Route_](#route-parameters)
    - [_Parameter Wajib_](#required-parameters)
    - [_Parameter Opsional_](#parameters-optional-parameters)
    - [Pembatasan dengan _Regular Expression_](#parameters-regular-expression-constraints)
- [_Route_ Bernama](#named-routes)
- [Kelompok _Route_](#route-groups)
    - [_Middleware_](#route-group-middleware)
    - [_Controller_](#route-group-controllers)
    - [Melakukan _Route Subdomain_](#route-group-subdomain-routing)
    - [Prefix _Route_](#route-group-prefixes)
    - [Prefix Nama _Route_](#route-group-name-prefixes)
- [_Route_ Model Terpadu](#route-model-binding)
    - [_Binding Implicit_](#implicit-binding)
    - [_Binding Enum Implicit_](#implicit-enum-binding)
    - [_Binding Explicit_](#explicit-binding)
- [_Route Fallback_](#fallback-routes)
- [Pembatasan _Rate_](#rate-limiting)
    - [Mendefinisikan Pembatas _Rate_](#defining-rate-limiters)
    - [Melekatkan Pembatas _Rate_ pada _Routes_](#attaching-rate-limiters-to-routes)
- [Pemalsuan _Method Form_](#form-method-spoofing)
- [Mengakses _Route_ Saat Ini](#accessing-the-current-route)
- [_Cross-Origin Resource Sharing_ (CORS)](#cors)
- [_Route Caching_](#route-caching)

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

Jika rute Anda hanya perlu mengembalikan [view] (/docs/{{version}}/views), Anda dapat menggunakan metode `Rute::view`. Seperti metode `redirect`, metode ini menyediakan jalan pintas sederhana sehingga Anda tidak perlu mendefinisikan rute penuh atau rute _controller_. Metode `view` menerima URI sebagai argumen pertama dan nama _view_ sebagai argumen kedua. Selain itu, Anda dapat menyediakan _array_ data untuk diteruskan ke _view_ sebagai argumen ketiga yang opsional:

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

Rute yang diberi nama memungkinkan pembuatan URL atau redirect yang nyaman untuk rute tertentu. Anda dapat menentukan nama untuk rute dengan menyambungkan metode name pada definisi rute:

Rute dengan nama memungkinkan pembuatan URL atau pengalihan yang mudah untuk rute tertentu. Anda dapat menspesifikasikan nama untuk sebuah rute dengan merantai metode `nama` ke dalam definisi rute:

Named routes allow the convenient generation of URLs or redirects for specific routes. You may specify a name for a route by chaining the `name` method onto the route definition:

    Route::get('/user/profile', function () {
        //
    })->name('profile');

Anda juga dapat menentukan nama rute untuk tindakan kontroller:

You may also specify route names for controller actions:

    Route::get(
        '/user/profile',
        [UserProfileController::class, 'show']
    )->name('profile');

> **Warning**  

Nama rute sebaiknya selalu unik.

> Route names should always be unique.

<a name="generating-urls-to-named-routes"></a>

Menghasilkan URL ke Rute yang Diberi Nama.

#### Generating URLs To Named Routes

Setelah Anda memberikan nama ke rute tertentu, Anda dapat menggunakan nama rute tersebut saat menghasilkan URL atau redirect melalui fungsi helper route dan redirect Laravel:

Once you have assigned a name to a given route, you may use the route's name when generating URLs or redirects via Laravel's `route` and `redirect` helper functions:

    // Generating URLs...
    $url = route('profile');

    // Generating Redirects...
    return redirect()->route('profile');

    return to_route('profile');

Jika rute yang diberi nama mendefinisikan parameter, Anda dapat mengirimkan parameter sebagai argumen kedua ke fungsi route. Parameter yang diberikan akan otomatis dimasukkan ke dalam URL yang dihasilkan pada posisi yang tepat:

If the named route defines parameters, you may pass the parameters as the second argument to the `route` function. The given parameters will automatically be inserted into the generated URL in their correct positions:

    Route::get('/user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

Jika Anda mengirimkan parameter tambahan dalam array, pasangan kunci / nilai tersebut akan otomatis ditambahkan ke query string URL yang dihasilkan:

If you pass additional parameters in the array, those key / value pairs will automatically be added to the generated URL's query string:

    Route::get('/user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1, 'photos' => 'yes']);

    // /user/1/profile?photos=yes

> **Note**  

Terkadang, Anda mungkin ingin menentukan nilai default secara luas permintaan untuk parameter URL, seperti locale saat ini. Untuk mencapai ini, Anda dapat menggunakan metode URL::defaults

> Sometimes, you may wish to specify request-wide default values for URL parameters, such as the current locale. To accomplish this, you may use the [`URL::defaults` method](/docs/{{version}}/urls#default-values).

<a name="inspecting-the-current-route"></a>
#### Inspecting The Current Route

Jika Anda ingin menentukan apakah permintaan saat ini diarahkan ke rute yang diberi nama tertentu, Anda dapat menggunakan metode named pada sebuah instance Rute. Sebagai contoh, Anda dapat memeriksa nama rute saat ini dari middleware rute:

If you would like to determine if the current request was routed to a given named route, you may use the `named` method on a Route instance. For example, you may check the current route name from a route middleware:

    /**
     * Handle an incoming request.
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
## Route Groups

Kelompok rute memungkinkan Anda untuk membagikan atribut rute, seperti middleware, di sejumlah besar rute tanpa perlu mendefinisikan atribut-atribut tersebut pada setiap rute individu.

Route groups allow you to share route attributes, such as middleware, across a large number of routes without needing to define those attributes on each individual route.

Kelompok bersarang mencoba "mengabungkan" secara cerdas atribut dengan kelompok induknya. Middleware dan kondisi where digabungkan sementara nama dan prefiks ditambahkan. Pengahapus namespace dan garis miring di prefiks URI secara otomatis ditambahkan di tempat yang tepat.

Nested groups attempt to intelligently "merge" attributes with their parent group. Middleware and `where` conditions are merged while names and prefixes are appended. Namespace delimiters and slashes in URI prefixes are automatically added where appropriate.

<a name="route-group-middleware"></a>
### Middleware

Untuk menetapkan middleware pada semua rute dalam sebuah kelompok, Anda dapat menggunakan metode middleware sebelum mendefinisikan kelompok. Middleware dieksekusi dalam urutan yang tercantum dalam array:

To assign [middleware](/docs/{{version}}/middleware) to all routes within a group, you may use the `middleware` method before defining the group. Middleware are executed in the order they are listed in the array:

    Route::middleware(['first', 'second'])->group(function () {
        Route::get('/', function () {
            // Uses first & second middleware...
        });

        Route::get('/user/profile', function () {
            // Uses first & second middleware...
        });
    });

<a name="route-group-controllers"></a>
### Controllers

Jika sekumpulan rute semua menggunakan kontroller yang sama, Anda dapat menggunakan metode controller untuk menentukan kontroller umum untuk semua rute dalam kelompok. Kemudian, saat mendefinisikan rute, Anda hanya perlu memberikan metode kontroller yang mereka panggil:

If a group of routes all utilize the same [controller](/docs/{{version}}/controllers), you may use the `controller` method to define the common controller for all of the routes within the group. Then, when defining the routes, you only need to provide the controller method that they invoke:

    use App\Http\Controllers\OrderController;

    Route::controller(OrderController::class)->group(function () {
        Route::get('/orders/{id}', 'show');
        Route::post('/orders', 'store');
    });

<a name="route-group-subdomain-routing"></a>
### Subdomain Routing

Kelompok rute juga dapat digunakan untuk menangani routing subdomain. Subdomain dapat ditetapkan parameter rute seperti URI rute, sehingga Anda dapat menangkap sebagian dari subdomain untuk digunakan dalam rute atau kontroller Anda. Subdomain dapat ditentukan dengan memanggil metode domain sebelum mendefinisikan kelompok:

Route groups may also be used to handle subdomain routing. Subdomains may be assigned route parameters just like route URIs, allowing you to capture a portion of the subdomain for usage in your route or controller. The subdomain may be specified by calling the `domain` method before defining the group:

    Route::domain('{account}.example.com')->group(function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

> **Warning**  

Untuk memastikan rute subdomain Anda dapat dijangkau, Anda sebaiknya mendaftarkan rute subdomain sebelum mendaftarkan rute domain root. Ini akan mencegah rute domain root menimpa rute subdomain yang memiliki jalur URI yang sama.

> In order to ensure your subdomain routes are reachable, you should register subdomain routes before registering root domain routes. This will prevent root domain routes from overwriting subdomain routes which have the same URI path.

<a name="route-group-prefixes"></a>
### Route Prefixes

Metode prefix dapat digunakan untuk memberikan prefiks pada setiap rute dalam kelompok dengan URI tertentu. Sebagai contoh, Anda mungkin ingin memberikan prefiks pada semua URI rute dalam kelompok dengan admin:

The `prefix` method may be used to prefix each route in the group with a given URI. For example, you may want to prefix all route URIs within the group with `admin`:

    Route::prefix('admin')->group(function () {
        Route::get('/users', function () {
            // Matches The "/admin/users" URL
        });
    });

<a name="route-group-name-prefixes"></a>
### Route Name Prefixes

Metode name dapat digunakan untuk memberikan prefiks pada setiap nama rute dalam kelompok dengan string tertentu. Sebagai contoh, Anda mungkin ingin memberikan prefiks pada semua nama rute kelompok dengan admin. String yang diberikan ditambahkan ke nama rute tepat seperti yang ditentukan, jadi kami akan pastikan untuk memberikan karakter . akhir pada prefiks:

The `name` method may be used to prefix each route name in the group with a given string. For example, you may want to prefix all of the grouped route's names with `admin`. The given string is prefixed to the route name exactly as it is specified, so we will be sure to provide the trailing `.` character in the prefix:

    Route::name('admin.')->group(function () {
        Route::get('/users', function () {
            // Route assigned name "admin.users"...
        })->name('users');
    });

<a name="route-model-binding"></a>
## Route Model Binding

Saat menginjeksi ID model ke rute atau _action_ dari _controller_, Anda akan sering melakukan kueri basis data untuk mengambil model yang sesuai dengan ID tersebut. Untuk pengikatan (_Binding_) rute-model, Laravel menyediakan cara yang mudah untuk secara otomatis menginjeksi _instance_ model langsung ke rute Anda. Misalnya, alih-alih menginjeksi ID pengguna, Anda dapat menginjeksi seluruh _instance_ model `User` yang cocok dengan ID yang diberikan.

<a name="implicit-binding"></a>
### Pengikatan Implisit

Laravel secara otomatis memberikan model Eloquent yang didefinisikan dalam rute atau _action_ milik _controller_ yang nama variabel _type-hint_-nya sama dengan nama segmen rute. Sebagai contoh:

    use App\Models\User;

    Route::get('/users/{user}', function (User $user) {
        return $user->email;
    });

Karena variabel `$user` diisyaratkan sebagai model Eloquent `App\Models\User` dan nama variabel cocok dengan segmen URI `{user}`, Laravel akan secara otomatis menginjeksikan _instance_ model yang memiliki ID yang cocok dengan nilai yang sesuai dari _request_ URI. Jika _instance_ model tidak ditemukan pada basis data, respons HTTP 404 akan secara otomatis dihasilkan.

Tentu saja, pengikatan implisit juga dimungkinkan ketika menggunakan metode dalam _controller_. Sekali lagi, perhatikan segmen URI `{user}` sesuai dengan variabel `$user` di controller yang berisi _type-hint_ `App\Models\User`:

    use App\Http\Controllers\UserController;
    use App\Models\User;

    // Route definition...
    Route::get('/users/{user}', [UserController::class, 'show']);

    // Controller method definition...
    public function show(User $user)
    {
        return view('user.profile', ['user' => $user]);
    }

<a name="implicit-soft-deleted-models"></a>
#### Soft Deleted Models

Biasanya, binding model implicit tidak akan mengambil model yang telah dihapus secara soft. Namun, Anda dapat menyuruh binding implicit untuk mengambil model-model ini dengan memanggil method withTrashed pada definisi rute Anda:

Typically, implicit model binding will not retrieve models that have been [soft deleted](/docs/{{version}}/eloquent#soft-deleting). However, you may instruct the implicit binding to retrieve these models by chaining the `withTrashed` method onto your route's definition:

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

Saat secara implisit binding beberapa model Eloquent dalam satu definisi rute, Anda mungkin ingin membatasi model Eloquent kedua sehingga harus menjadi anak dari model Eloquent sebelumnya. Sebagai contoh, pertimbangkan definisi rute berikut yang mengambil posting blog dengan slug untuk pengguna tertentu:

When implicitly binding multiple Eloquent models in a single route definition, you may wish to scope the second Eloquent model such that it must be a child of the previous Eloquent model. For example, consider this route definition that retrieves a blog post by slug for a specific user:

    use App\Models\Post;
    use App\Models\User;

    Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
        return $post;
    });

Ketika menggunakan binding implisit kustom yang ditandai sebagai parameter rute bersarang, Laravel secara otomatis akan membatasi query untuk mengambil model bersarang dengan parent menggunakan konvensi untuk menebak nama hubungan pada parent. Dalam hal ini, akan diasumsikan bahwa model User memiliki hubungan yang diberi nama posts (bentuk jamak dari nama parameter rute) yang dapat digunakan untuk mengambil model Post.

When using a custom keyed implicit binding as a nested route parameter, Laravel will automatically scope the query to retrieve the nested model by its parent using conventions to guess the relationship name on the parent. In this case, it will be assumed that the `User` model has a relationship named `posts` (the plural form of the route parameter name) which can be used to retrieve the `Post` model.

Jika Anda ingin agar Laravel mengikat "anak" secara terbatas bahkan tanpa kunci khusus, Anda dapat memanggil metode scopeBindings saat mendefinisikan rute Anda:

If you wish, you may instruct Laravel to scope "child" bindings even when a custom key is not provided. To do so, you may invoke the `scopeBindings` method when defining your route:

    use App\Models\Post;
    use App\Models\User;

    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        return $post;
    })->scopeBindings();

Atau, Anda dapat memberitahu seluruh kelompok definisi rute untuk menggunakan binding yang terbatas:

Or, you may instruct an entire group of route definitions to use scoped bindings:

    Route::scopeBindings()->group(function () {
        Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
            return $post;
        });
    });

Similarly, you may explicitly instruct Laravel to not scope bindings by invoking the `withoutScopedBindings` method:

    Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
        return $post;
    })->withoutScopedBindings();

<a name="customizing-missing-model-behavior"></a>
#### Menyesuaikan Perilaku Model yang Hilang

Biasanya, respons HTTP 404 akan dihasilkan jika model yang terikat secara implisit tidak ditemukan. Namun, Anda dapat menyesuaikan perilaku ini dengan memanggil metode `missing` ketika mendefinisikan rute Anda. Metode `missing` menerima closure yang akan dipanggil jika model yang terikat secara implisit tidak dapat ditemukan:

Typically, a 404 HTTP response will be generated if an implicitly bound model is not found. However, you may customize this behavior by calling the `missing` method when defining your route. The `missing` method accepts a closure that will be invoked if an implicitly bound model can not be found:

    use App\Http\Controllers\LocationsController;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Redirect;

    Route::get('/locations/{location:slug}', [LocationsController::class, 'show'])
            ->name('locations.view')
            ->missing(function (Request $request) {
                return Redirect::route('locations.index');
            });

<a name="implicit-enum-binding"></a>
### Implicit Enum Binding

PHP 8.1 memperkenalkan dukungan untuk [Enum](https://www.php.net/manual/en/language.enumerations.backed.php). Untuk melengkapi fitur ini, Laravel memungkinkan Anda untuk mengetikkan petunjuk [string-backed Enum](https://www.php.net/manual/en/language.enumerations.backed.php) pada definisi rute Anda dan Laravel hanya akan memanggil rute jika segmen rute tersebut sesuai dengan nilai Enum yang valid. Jika tidak, respons HTTP 404 akan dikembalikan secara otomatis. Sebagai contoh, diberikan Enum berikut:

PHP 8.1 introduced support for [Enums](https://www.php.net/manual/en/language.enumerations.backed.php). To compliment this feature, Laravel allows you to type-hint a [string-backed Enum](https://www.php.net/manual/en/language.enumerations.backed.php) on your route definition and Laravel will only invoke the route if that route segment corresponds to a valid Enum value. Otherwise, a 404 HTTP response will be returned automatically. For example, given the following Enum:

```php
<?php

namespace App\Enums;

enum Category: string
{
    case Fruits = 'fruits';
    case People = 'people';
}
```

Anda dapat mendefinisikan rute yang hanya akan dipanggil jika segmen rute `{kategori}` adalah `buah` atau `orang`. Jika tidak, Laravel akan mengembalikan respons HTTP 404:

You may define a route that will only be invoked if the `{category}` route segment is `fruits` or `people`. Otherwise, Laravel will return a 404 HTTP response:

```php
use App\Enums\Category;
use Illuminate\Support\Facades\Route;

Route::get('/categories/{category}', function (Category $category) {
    return $category->value;
});
```

<a name="explicit-binding"></a>
### Pengikatan Eksplisit

Anda tidak diharuskan menggunakan resolusi model implisit berbasis konvensi Laravel untuk menggunakan model binding. Anda juga dapat secara eksplisit mendefinisikan bagaimana parameter rute sesuai dengan model. Untuk mendaftarkan pengikatan eksplisit, gunakan metode `model` router untuk menentukan kelas untuk parameter yang diberikan. Anda harus mendefinisikan binding model eksplisit Anda di awal metode `boot` dari kelas `RouteServiceProvider` Anda:

You are not required to use Laravel's implicit, convention based model resolution in order to use model binding. You can also explicitly define how route parameters correspond to models. To register an explicit binding, use the router's `model` method to specify the class for a given parameter. You should define your explicit model bindings at the beginning of the `boot` method of your `RouteServiceProvider` class:

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

Next, define a route that contains a `{user}` parameter:

    use App\Models\User;

    Route::get('/users/{user}', function (User $user) {
        //
    });

Karena kita telah mengikat semua parameter `{user}` ke model `App\Models\User`, sebuah instance dari kelas tersebut akan diinjeksi ke dalam rute. Jadi, misalnya, permintaan ke `users/1` akan menginjeksi instance `User` dari database yang memiliki ID `1`.

Jika _instance_ model yang cocok tidak ditemukan dalam database, respons HTTP 404 akan dihasilkan secara otomatis.

<a name="customizing-the-resolution-logic"></a>
#### Menyesuaikan Logika Resolusi

Jika Anda ingin mendefinisikan logika resolusi pengikatan model Anda sendiri, Anda dapat menggunakan metode `Rute::bind`. _Closure_ yang Anda berikan ke metode `bind` akan menerima nilai segmen URI dan harus mengembalikan instance dari kelas yang harus diinjeksikan ke dalam rute. Sekali lagi, kustomisasi ini harus dilakukan dalam metode `boot` dari `RouteServiceProvider` aplikasi Anda:

If you wish to define your own model binding resolution logic, you may use the `Route::bind` method. The closure you pass to the `bind` method will receive the value of the URI segment and should return the instance of the class that should be injected into the route. Again, this customization should take place in the `boot` method of your application's `RouteServiceProvider`:

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

Sebagai alternatif, Anda dapat meng-override metode `resolveRouteBinding` pada model Eloquent Anda. Metode ini akan menerima nilai segmen URI dan harus mengembalikan instance dari kelas yang harus diinjeksikan ke dalam rute:

Alternatively, you may override the `resolveRouteBinding` method on your Eloquent model. This method will receive the value of the URI segment and should return the instance of the class that should be injected into the route:

    /**
     * Retrieve the model for a bound value.
     *
     * @param  mixed  $value
     * @param  string|null  $field
     * @return \Illuminate\Database\Eloquent\Model|null
     */
    public function resolveRouteBinding($value, $field = null)
    {
        return $this->where('name', $value)->firstOrFail();
    }

Jika sebuah rute menggunakan [implicit binding scoping](#implicit-model-binding-scoping), metode `resolveChildRouteBinding` akan digunakan untuk menyelesaikan pengikatan anak dari model induk:

If a route is utilizing [implicit binding scoping](#implicit-model-binding-scoping), the `resolveChildRouteBinding` method will be used to resolve the child binding of the parent model:

    /**
     * Retrieve the child model for a bound value.
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

Dengan menggunakan metode `Route::fallback`, Anda dapat mendefinisikan rute yang akan dieksekusi ketika tidak ada rute lain yang cocok dengan permintaan yang masuk. Biasanya, permintaan yang tidak tertangani akan secara otomatis me-render halaman "404" melalui exception handler aplikasi Anda. Namun, karena Anda biasanya akan mendefinisikan rute `fallback` di dalam file `routes/web.php` Anda, semua middleware di grup middleware `web` akan berlaku untuk rute tersebut. Anda bebas menambahkan middleware tambahan ke rute ini sesuai kebutuhan:

Using the `Route::fallback` method, you may define a route that will be executed when no other route matches the incoming request. Typically, unhandled requests will automatically render a "404" page via your application's exception handler. However, since you would typically define the `fallback` route within your `routes/web.php` file, all middleware in the `web` middleware group will apply to the route. You are free to add additional middleware to this route as needed:

    Route::fallback(function () {
        //
    });

> **Peringatan**  
> Rute fallback harus selalu merupakan rute terakhir yang didaftarkan pada aplikasi Anda.

<a name="rate-limiting"></a>
## Rate Limiting

<a name="defining-rate-limiters"></a>
### Defining Rate Limiters

Laravel menyertakan layanan pembatasan tarif yang kuat dan dapat disesuaikan yang dapat Anda manfaatkan untuk membatasi jumlah lalu lintas untuk rute atau grup rute tertentu. Untuk memulai, Anda harus menentukan konfigurasi pembatas tarif yang memenuhi kebutuhan aplikasi Anda. Biasanya, ini harus dilakukan dalam metode `configureRateLimiting` dari kelas `App\Providers\RouteServiceProvider` aplikasi Anda.

Pembatas tarif didefinisikan menggunakan metode `for` facade `RateLimiter`. Metode `for` menerima nama pembatas laju dan closure yang mengembalikan konfigurasi batas yang harus diterapkan ke rute yang ditugaskan ke pembatas laju. Konfigurasi limit adalah instance dari kelas `Illuminate\Cache\RateLimiting\Limit`. Kelas ini berisi metode-metode "builder" yang berguna sehingga Anda dapat dengan cepat mendefinisikan limit Anda. Nama rate limiter dapat berupa string apa pun yang Anda inginkan:

Laravel includes powerful and customizable rate limiting services that you may utilize to restrict the amount of traffic for a given route or group of routes. To get started, you should define rate limiter configurations that meet your application's needs. Typically, this should be done within the `configureRateLimiting` method of your application's `App\Providers\RouteServiceProvider` class.

Rate limiters are defined using the `RateLimiter` facade's `for` method. The `for` method accepts a rate limiter name and a closure that returns the limit configuration that should apply to routes that are assigned to the rate limiter. Limit configuration are instances of the `Illuminate\Cache\RateLimiting\Limit` class. This class contains helpful "builder" methods so that you can quickly define your limit. The rate limiter name may be any string you wish:

    use Illuminate\Cache\RateLimiting\Limit;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\RateLimiter;

    /**
     * Configure the rate limiters for the application.
     *
     * @return void
     */
    protected function configureRateLimiting()
    {
        RateLimiter::for('global', function (Request $request) {
            return Limit::perMinute(1000);
        });
    }

Jika permintaan yang masuk melebihi batas tarif yang ditentukan, respons dengan kode status HTTP 429 akan secara otomatis dikembalikan oleh Laravel. Jika Anda ingin menentukan respons Anda sendiri yang harus dikembalikan oleh batas tarif, Anda dapat menggunakan metode `response`:

If the incoming request exceeds the specified rate limit, a response with a 429 HTTP status code will automatically be returned by Laravel. If you would like to define your own response that should be returned by a rate limit, you may use the `response` method:

    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000)->response(function (Request $request, array $headers) {
            return response('Custom response...', 429, $headers);
        });
    });

Karena callback pembatas laju menerima instance permintaan HTTP yang masuk, Anda dapat membuat batas laju yang sesuai secara dinamis berdasarkan permintaan yang masuk atau pengguna yang diautentikasi:

Since rate limiter callbacks receive the incoming HTTP request instance, you may build the appropriate rate limit dynamically based on the incoming request or authenticated user:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()->vipCustomer()
                    ? Limit::none()
                    : Limit::perMinute(100);
    });

<a name="segmenting-rate-limits"></a>
#### Segmenting Rate Limits

Kadang-kadang, Anda mungkin ingin membagi batas kecepatan dengan beberapa nilai yang sewenang-wenang. Misalnya, anda mungkin ingin mengizinkan pengguna untuk mengakses rute tertentu 100 kali per menit per alamat IP. Untuk mencapai hal ini, anda dapat menggunakan metode `by` ketika membangun rate limit anda:

Sometimes you may wish to segment rate limits by some arbitrary value. For example, you may wish to allow users to access a given route 100 times per minute per IP address. To accomplish this, you may use the `by` method when building your rate limit:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()->vipCustomer()
                    ? Limit::none()
                    : Limit::perMinute(100)->by($request->ip());
    });

Untuk mengilustrasikan fitur ini dengan menggunakan contoh lain, kita bisa membatasi akses ke rute hingga 100 kali per menit per ID pengguna yang diautentikasi atau 10 kali per menit per alamat IP untuk tamu:

To illustrate this feature using another example, we can limit access to the route to 100 times per minute per authenticated user ID or 10 times per minute per IP address for guests:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()
                    ? Limit::perMinute(100)->by($request->user()->id)
                    : Limit::perMinute(10)->by($request->ip());
    });

<a name="multiple-rate-limits"></a>
#### Multiple Rate Limits

Jika diperlukan, Anda dapat mengembalikan larik batas tarif untuk konfigurasi pembatas tarif yang diberikan. Setiap rate limit akan dievaluasi untuk rute berdasarkan urutan penempatannya dalam array:

If needed, you may return an array of rate limits for a given rate limiter configuration. Each rate limit will be evaluated for the route based on the order they are placed within the array:

    RateLimiter::for('login', function (Request $request) {
        return [
            Limit::perMinute(500),
            Limit::perMinute(3)->by($request->input('email')),
        ];
    });

<a name="attaching-rate-limiters-to-routes"></a>
### Attaching Rate Limiters To Routes

Pembatas laju dapat dilampirkan ke rute atau grup rute menggunakan `throttle` [middleware] (/docs/{{version}}/middleware). Middleware throttle menerima nama pembatas laju yang ingin Anda tetapkan ke rute:

Rate limiters may be attached to routes or route groups using the `throttle` [middleware](/docs/{{version}}/middleware). The throttle middleware accepts the name of the rate limiter you wish to assign to the route:

    Route::middleware(['throttle:uploads'])->group(function () {
        Route::post('/audio', function () {
            //
        });

        Route::post('/video', function () {
            //
        });
    });

<a name="throttling-with-redis"></a>
#### Throttling With Redis

Biasanya, middleware `throttle` dipetakan ke kelas `Illuminate\Routing\Middleware\ThrottleRequests`. Pemetaan ini didefinisikan dalam kernel HTTP aplikasi Anda (`App\Http\Kernel`). Namun, jika Anda menggunakan Redis sebagai driver cache aplikasi Anda, Anda mungkin ingin mengubah pemetaan ini untuk menggunakan kelas `Illuminate\Routing\Middleware\ThrottleRequestsWithRedis`. Kelas ini lebih efisien dalam mengelola pembatasan laju menggunakan Redis:

Typically, the `throttle` middleware is mapped to the `Illuminate\Routing\Middleware\ThrottleRequests` class. This mapping is defined in your application's HTTP kernel (`App\Http\Kernel`). However, if you are using Redis as your application's cache driver, you may wish to change this mapping to use the `Illuminate\Routing\Middleware\ThrottleRequestsWithRedis` class. This class is more efficient at managing rate limiting using Redis:

    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,

<a name="form-method-spoofing"></a>
## Form Method Spoofing

Formulir HTML tidak mendukung aksi `PUT`, `PATCH`, atau `DELETE`. Jadi, ketika mendefinisikan rute `PUT`, `PATCH`, atau `DELETE` yang dipanggil dari form HTML, Anda perlu menambahkan field `_method` yang tersembunyi ke form. Nilai yang dikirim dengan field `_method` akan digunakan sebagai metode permintaan HTTP:

HTML forms do not support `PUT`, `PATCH`, or `DELETE` actions. So, when defining `PUT`, `PATCH`, or `DELETE` routes that are called from an HTML form, you will need to add a hidden `_method` field to the form. The value sent with the `_method` field will be used as the HTTP request method:

    <form action="/example" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

Untuk kenyamanan, anda dapat menggunakan `@method` [Blade directive] (/docs/{{version}}/blade) untuk menghasilkan kolom input `_method`:

For convenience, you may use the `@method` [Blade directive](/docs/{{version}}/blade) to generate the `_method` input field:

    <form action="/example" method="POST">
        @method('PUT')
        @csrf
    </form>

<a name="accessing-the-current-route"></a>
## Accessing The Current Route

Anda dapat menggunakan metode `current`, `currentRouteName`, dan `currentRouteAction` pada facade `Route` untuk mengakses informasi tentang rute yang menangani permintaan yang masuk:

You may use the `current`, `currentRouteName`, and `currentRouteAction` methods on the `Route` facade to access information about the route handling the incoming request:

    use Illuminate\Support\Facades\Route;

    $route = Route::current(); // Illuminate\Routing\Route
    $name = Route::currentRouteName(); // string
    $action = Route::currentRouteAction(); // string

Anda dapat merujuk ke dokumentasi API untuk [kelas yang mendasari fasad Route](https://laravel.com/api/{{{version}}/Illuminate/Routing/Router.html) dan [Route instance](https://laravel.com/api/{{{version}}/Illuminate/Routing/Route.html) untuk meninjau semua metode yang tersedia pada kelas router dan route.

You may refer to the API documentation for both the [underlying class of the Route facade](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) and [Route instance](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html) to review all of the methods that are available on the router and route classes.

<a name="cors"></a>
## Cross-Origin Resource Sharing (CORS)

Laravel dapat secara otomatis merespons permintaan HTTP CORS `OPTIONS` dengan nilai-nilai yang Anda konfigurasikan. Semua pengaturan CORS dapat dikonfigurasi dalam file konfigurasi `config/cors.php` aplikasi Anda. Permintaan `OPTIONS` secara otomatis akan ditangani oleh `HandleCors` [middleware] (/docs/{{version}}/middleware) yang disertakan secara default dalam tumpukan middleware global Anda. Tumpukan middleware global Anda terletak di kernel HTTP aplikasi Anda (`App\Http\Kernel`).

Laravel can automatically respond to CORS `OPTIONS` HTTP requests with values that you configure. All CORS settings may be configured in your application's `config/cors.php` configuration file. The `OPTIONS` requests will automatically be handled by the `HandleCors` [middleware](/docs/{{version}}/middleware) that is included by default in your global middleware stack. Your global middleware stack is located in your application's HTTP kernel (`App\Http\Kernel`).

> **Catatan**  

Untuk informasi lebih lanjut tentang CORS dan header CORS, silakan berkonsultasi dengan [dokumentasi web MDN tentang CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#The_HTTP_response_headers).

> For more information on CORS and CORS headers, please consult the [MDN web documentation on CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#The_HTTP_response_headers).

<a name="route-caching"></a>
## Melakukan _Cache Route_

Saat menerapkan aplikasi Anda ke produksi, Anda harus memanfaatkan cache rute Laravel. Menggunakan cache rute akan secara drastis mengurangi jumlah waktu yang diperlukan untuk mendaftarkan semua rute aplikasi Anda. Untuk membuat cache rute, jalankan perintah Artisan `route:cache`:

When deploying your application to production, you should take advantage of Laravel's route cache. Using the route cache will drastically decrease the amount of time it takes to register all of your application's routes. To generate a route cache, execute the `route:cache` Artisan command:

```shell
php artisan route:cache
```

Setelah menjalankan perintah ini, _file_ rute yang telah di-_cache_ akan dimuat pada setiap permintaan. Ingat, jika Anda menambahkan rute baru, Anda perlu melakukan cache ulang. Karena itu, Anda sebaiknya hanya menjalankan perintah `route:cache` pada saat _deployment_ saja.

Anda dapat menggunakan perintah `route:clear` untuk menghapus _cache rute_:

```shell
php artisan route:clear
```
