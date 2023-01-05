# Melakukan _Route_

- [Dasar Melakukan _Route_](#basic-routing)
    - [_Route Redirect_](#redirect-routes)
    - [_Route View_](#view-routes)
    - [_Daftar Route_](#the-route-list)
- [Parameter _Route_](#route-parameters)
    - [_Parameter Wajib_](#required-parameters)
    - [_Parameter Opsional_](#parameters-optional-parameters)
    - [Batasan _Regular Expression_](#parameters-regular-expression-constraints)
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
## Dasar Melakukan _Route_

Rute dasar Laravel menerima URI dan closure, memberikan metode yang sangat sederhana dan ekspresif untuk mendefinisikan rute dan perilaku tanpa berkonfigurasi file routing yang rumit:

The most basic Laravel routes accept a URI and a closure, providing a very simple and expressive method of defining routes and behavior without complicated routing configuration files:

    use Illuminate\Support\Facades\Route;

    Route::get('/greeting', function () {
        return 'Halo Dunia';
    });

<a name="the-default-route-files"></a>
#### The Default Route Files

Semua rute Laravel didefinisikan dalam file rute Anda, yang terletak di direktori routes. File-file ini secara otomatis dimuat oleh App\Providers\RouteServiceProvider dari aplikasi Anda. File routes/web.php mendefinisikan rute yang untuk antarmuka web Anda. Rute ini ditugaskan ke grup middleware web, yang menyediakan fitur seperti keadaan sesi dan perlindungan CSRF. Rute dalam routes/api.php tidak memiliki keadaan dan ditugaskan ke grup middleware api.

All Laravel routes are defined in your route files, which are located in the `routes` directory. These files are automatically loaded by your application's `App\Providers\RouteServiceProvider`. The `routes/web.php` file defines routes that are for your web interface. These routes are assigned the `web` middleware group, which provides features like session state and CSRF protection. The routes in `routes/api.php` are stateless and are assigned the `api` middleware group.

Untuk kebanyakan aplikasi, Anda akan memulai dengan mendefinisikan rute dalam file routes/web.php Anda. Rute yang didefinisikan dalam routes/web.php dapat diakses dengan memasukkan URL rute yang didefinisikan ke dalam browser Anda. Sebagai contoh, Anda dapat mengakses rute berikut dengan mennavigasi ke http://example.com/user di browser Anda:

For most applications, you will begin by defining routes in your `routes/web.php` file. The routes defined in `routes/web.php` may be accessed by entering the defined route's URL in your browser. For example, you may access the following route by navigating to `http://example.com/user` in your browser:

    use App\Http\Controllers\UserController;

    Route::get('/user', [UserController::class, 'index']);

Rute yang didefinisikan dalam file routes/api.php tertanam dalam grup rute oleh RouteServiceProvider. Dalam grup ini, prefiks URI /api secara otomatis diterapkan sehingga Anda tidak perlu menerapkannya secara manual pada setiap rute dalam file tersebut. Anda dapat memodifikasi prefiks dan opsi grup rute lainnya dengan memodifikasi kelas RouteServiceProvider Anda.

Routes defined in the `routes/api.php` file are nested within a route group by the `RouteServiceProvider`. Within this group, the `/api` URI prefix is automatically applied so you do not need to manually apply it to every route in the file. You may modify the prefix and other route group options by modifying your `RouteServiceProvider` class.

<a name="available-router-methods"></a>
#### Available Router Methods

Router memungkinkan Anda untuk mendaftarkan rute yang merespon setiap verba HTTP:

The router allows you to register routes that respond to any HTTP verb:

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

Terkadang Anda mungkin perlu mendaftarkan rute yang merespon beberapa verba HTTP. Anda dapat melakukannya dengan menggunakan metode match. Atau, Anda bahkan dapat mendaftarkan rute yang merespon pada semua verba HTTP menggunakan metode any:

Sometimes you may need to register a route that responds to multiple HTTP verbs. You may do so using the `match` method. Or, you may even register a route that responds to all HTTP verbs using the `any` method:

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('/', function () {
        //
    });

Ketika mendefinisikan beberapa rute yang memiliki URI yang sama, rute yang menggunakan metode get, post, put, patch, delete, dan options harus didefinisikan sebelum rute yang menggunakan metode any, match, dan redirect. Hal ini memastikan permintaan yang masuk cocok dengan rute yang benar.

> **Catatan**  
> When defining multiple routes that share the same URI, routes using the `get`, `post`, `put`, `patch`, `delete`, and `options` methods should be defined before routes using the `any`, `match`, and `redirect` methods. This ensures the incoming request is matched with the correct route.

<a name="dependency-injection"></a>
#### Dependency Injection

Anda dapat memberikan tipe-hint terhadap kebutuhan dependensi yang dibutuhkan oleh rute Anda di tanda tangan callback rute Anda. Dependensi yang dideklarasikan akan secara otomatis diselesaikan dan disuntikkan ke callback oleh service container Laravel. Sebagai contoh, Anda dapat memberikan tipe-hint kelas Illuminate\Http\Request untuk memiliki permintaan HTTP saat ini secara otomatis disuntikkan ke callback rute Anda:

You may type-hint any dependencies required by your route in your route's callback signature. The declared dependencies will automatically be resolved and injected into the callback by the Laravel [service container](/docs/{{version}}/container). For example, you may type-hint the `Illuminate\Http\Request` class to have the current HTTP request automatically injected into your route callback:

    use Illuminate\Http\Request;

    Route::get('/users', function (Request $request) {
        // ...
    });

<a name="csrf-protection"></a>
#### CSRF Protection

Ingat, form HTML yang menunjuk ke rute POST, PUT, PATCH, atau DELETE yang didefinisikan dalam file rute web harus memasukkan kolom token CSRF. Jika tidak, permintaan akan ditolak. Anda dapat membaca lebih lanjut tentang perlindungan CSRF di dalam dokumentasi CSRF:

Remember, any HTML forms pointing to `POST`, `PUT`, `PATCH`, or `DELETE` routes that are defined in the `web` routes file should include a CSRF token field. Otherwise, the request will be rejected. You can read more about CSRF protection in the [CSRF documentation](/docs/{{version}}/csrf):

    <form method="POST" action="/profile">
        @csrf
        ...
    </form>

<a name="redirect-routes"></a>
### Redirect Routes

Jika Anda mendefinisikan rute yang mengarah ke URI lain, Anda dapat menggunakan metode Route::redirect. Metode ini memberikan pintasan yang praktis sehingga Anda tidak perlu mendefinisikan rute penuh atau kontroller untuk melakukan redirect sederhana:

If you are defining a route that redirects to another URI, you may use the `Route::redirect` method. This method provides a convenient shortcut so that you do not have to define a full route or controller for performing a simple redirect:

    Route::redirect('/here', '/there');

Secara default, Route::redirect mengembalikan kode status 302. Anda dapat menyesuaikan kode status menggunakan parameter ketiga yang opsional:

By default, `Route::redirect` returns a `302` status code. You may customize the status code using the optional third parameter:

    Route::redirect('/here', '/there', 301);

Atau, Anda dapat menggunakan metode Route::permanentRedirect untuk mengembalikan kode status 301:

Or, you may use the `Route::permanentRedirect` method to return a `301` status code:

    Route::permanentRedirect('/here', '/there');

> **Peringatan**  

Ketika menggunakan parameter rute di rute redirect, parameter berikut dicadangkan oleh Laravel dan tidak dapat digunakan: destination dan status.

> When using route parameters in redirect routes, the following parameters are reserved by Laravel and cannot be used: `destination` and `status`.

<a name="view-routes"></a>
### View Routes

Jika rute Anda hanya perlu mengembalikan view, Anda dapat menggunakan metode Route::view. Seperti metode redirect, metode ini memberikan pintasan yang sederhana sehingga Anda tidak perlu mendefinisikan rute penuh atau kontroller. Metode view menerima URI sebagai argumen pertamanya dan nama view sebagai argumen kedua. Selain itu, Anda dapat menyediakan array data untuk dikirim ke view sebagai argumen ketiga yang opsional:

If your route only needs to return a [view](/docs/{{version}}/views), you may use the `Route::view` method. Like the `redirect` method, this method provides a simple shortcut so that you do not have to define a full route or controller. The `view` method accepts a URI as its first argument and a view name as its second argument. In addition, you may provide an array of data to pass to the view as an optional third argument:

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

> **Peringatan**  

Ketika menggunakan parameter rute di rute view, parameter berikut dicadangkan oleh Laravel dan tidak dapat digunakan: view, data, status, dan headers

> When using route parameters in view routes, the following parameters are reserved by Laravel and cannot be used: `view`, `data`, `status`, and `headers`.

<a name="the-route-list"></a>
### The Route List

Perintah Artisan route:list dapat dengan mudah memberikan gambaran tentang semua rute yang didefinisikan oleh aplikasi Anda:

The `route:list` Artisan command can easily provide an overview of all of the routes that are defined by your application:

```shell
php artisan route:list
```

Secara default, middleware rute yang ditugaskan pada setiap rute tidak akan ditampilkan di output route:list; namun, Anda dapat meminta Laravel untuk menampilkan middleware rute dengan menambahkan opsi -v pada perintah:

By default, the route middleware that are assigned to each route will not be displayed in the `route:list` output; however, you can instruct Laravel to display the route middleware by adding the `-v` option to the command:

```shell
php artisan route:list -v
```

Anda juga dapat meminta Laravel untuk hanya menampilkan rute yang dimulai dengan URI tertentu:

You may also instruct Laravel to only show routes that begin with a given URI:

```shell
php artisan route:list --path=api
```

Selain itu, Anda dapat meminta Laravel untuk menyembunyikan rute apa pun yang didefinisikan oleh paket pihak ketiga dengan memberikan opsi --except-vendor saat menjalankan perintah route:list:

In addition, you may instruct Laravel to hide any routes that are defined by third-party packages by providing the `--except-vendor` option when executing the `route:list` command:

```shell
php artisan route:list --except-vendor
```

Sama halnya, Anda juga dapat meminta Laravel untuk hanya menampilkan rute yang didefinisikan oleh paket pihak ketiga dengan memberikan opsi --only-vendor saat menjalankan perintah route:list:

Likewise, you may also instruct Laravel to only show routes that are defined by third-party packages by providing the `--only-vendor` option when executing the `route:list` command:

```shell
php artisan route:list --only-vendor
```

<a name="route-parameters"></a>
## Route Parameters

<a name="required-parameters"></a>
### Required Parameters

Terkadang Anda akan perlu untuk menangkap segmen URI di dalam rute Anda. Sebagai contoh, Anda mungkin perlu menangkap ID pengguna dari URL. Anda dapat melakukannya dengan mendefinisikan parameter rute:

Sometimes you will need to capture segments of the URI within your route. For example, you may need to capture a user's ID from the URL. You may do so by defining route parameters:

    Route::get('/user/{id}', function ($id) {
        return 'User '.$id;
    });

Anda dapat mendefinisikan sebanyak parameter rute yang dibutuhkan oleh rute Anda:

You may define as many route parameters as required by your route:

    Route::get('/posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

Parameter rute selalu ditutup dalam {} kurung dan harus terdiri dari karakter alfabet. Garis bawah (_) juga diterima dalam nama parameter rute. Parameter rute disuntikkan ke callback rute / kontroller berdasarkan urutannya - nama argument callback rute / kontroller tidak penting.

Route parameters are always encased within `{}` braces and should consist of alphabetic characters. Underscores (`_`) are also acceptable within route parameter names. Route parameters are injected into route callbacks / controllers based on their order - the names of the route callback / controller arguments do not matter.

<a name="parameters-and-dependency-injection"></a>
#### Parameters & Dependency Injection

Jika rute Anda memiliki dependensi yang ingin disuntikkan secara otomatis ke callback rute oleh service container Laravel, Anda sebaiknya mencantumkan parameter rute setelah dependensi Anda:

If your route has dependencies that you would like the Laravel service container to automatically inject into your route's callback, you should list your route parameters after your dependencies:

    use Illuminate\Http\Request;

    Route::get('/user/{id}', function (Request $request, $id) {
        return 'User '.$id;
    });

<a name="parameters-optional-parameters"></a>
### Optional Parameters

Terkadang Anda mungkin perlu menentukan parameter rute yang mungkin tidak selalu ada di URI. Anda dapat melakukannya dengan menempatkan tanda ? setelah nama parameter. Pastikan untuk memberikan nilai default pada variabel yang sesuai dengan rute:

Occasionally you may need to specify a route parameter that may not always be present in the URI. You may do so by placing a `?` mark after the parameter name. Make sure to give the route's corresponding variable a default value:

    Route::get('/user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('/user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>
### Regular Expression Constraints

Anda dapat membatasi format parameter rute Anda dengan menggunakan metode where pada instance rute. Metode where menerima nama parameter dan ekspresi reguler yang menjelaskan bagaimana parameter harus dibatasi:

You may constrain the format of your route parameters using the `where` method on a route instance. The `where` method accepts the name of the parameter and a regular expression defining how the parameter should be constrained:

    Route::get('/user/{name}', function ($name) {
        //
    })->where('name', '[A-Za-z]+');

    Route::get('/user/{id}', function ($id) {
        //
    })->where('id', '[0-9]+');

    Route::get('/user/{id}/{name}', function ($id, $name) {
        //
    })->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

Untuk kenyamanan, beberapa pola ekspresi reguler yang sering digunakan memiliki metode helper yang memungkinkan Anda untuk dengan cepat menambahkan batasan pola ke rute Anda:

For convenience, some commonly used regular expression patterns have helper methods that allow you to quickly add pattern constraints to your routes:

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

If the incoming request does not match the route pattern constraints, a 404 HTTP response will be returned.

<a name="parameters-global-constraints"></a>
#### Global Constraints

Jika Anda ingin parameter rute selalu dibatasi oleh ekspresi reguler tertentu, Anda dapat menggunakan metode pattern. Anda sebaiknya mendefinisikan pola-pola ini di metode boot dari kelas App\Providers\RouteServiceProvider Anda:

If you would like a route parameter to always be constrained by a given regular expression, you may use the `pattern` method. You should define these patterns in the `boot` method of your `App\Providers\RouteServiceProvider` class:

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @return void
     */
    public function boot()
    {
        Route::pattern('id', '[0-9]+');
    }

Setelah pola telah didefinisikan, secara otomatis diterapkan pada semua rute yang menggunakan nama parameter tersebut:

Once the pattern has been defined, it is automatically applied to all routes using that parameter name:

    Route::get('/user/{id}', function ($id) {
        // Only executed if {id} is numeric...
    });

<a name="parameters-encoded-forward-slashes"></a>
#### Encoded Forward Slashes

Komponen routing Laravel memperbolehkan semua karakter kecuali / untuk hadir di dalam nilai parameter rute. Anda harus secara eksplisit memperbolehkan / untuk menjadi bagian dari placeholder Anda menggunakan ekspresi reguler kondisi where:

The Laravel routing component allows all characters except `/` to be present within route parameter values. You must explicitly allow `/` to be part of your placeholder using a `where` condition regular expression:

    Route::get('/search/{search}', function ($search) {
        return $search;
    })->where('search', '.*');

> **Warning**  

Garis miring yang dienkode hanya didukung dalam segmen rute terakhir.

> Encoded forward slashes are only supported within the last route segment.

<a name="named-routes"></a>
## Named Routes

Rute yang diberi nama memungkinkan pembuatan URL atau redirect yang nyaman untuk rute tertentu. Anda dapat menentukan nama untuk rute dengan menyambungkan metode name pada definisi rute:

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

When injecting a model ID to a route or controller action, you will often query the database to retrieve the model that corresponds to that ID. Laravel route model binding provides a convenient way to automatically inject the model instances directly into your routes. For example, instead of injecting a user's ID, you can inject the entire `User` model instance that matches the given ID.

<a name="implicit-binding"></a>
### Implicit Binding

Saat menyuntikkan ID model ke rute atau tindakan kontroller, Anda sering mengambil data dari database untuk mengambil model yang sesuai dengan ID tersebut. Laravel route model binding menyediakan cara yang nyaman untuk secara otomatis menyuntikkan instance model langsung ke dalam rute Anda. Sebagai contoh, daripada menyuntikkan ID pengguna, Anda dapat menyuntikkan seluruh instance model User yang cocok dengan ID yang diberikan.

Laravel secara otomatis menyelesaikan model Eloquent yang didefinisikan dalam rute atau tindakan kontroller yang nama variabel tipe-hinted-nya cocok dengan nama segmen rute. Sebagai contoh:

Laravel automatically resolves Eloquent models defined in routes or controller actions whose type-hinted variable names match a route segment name. For example:

    use App\Models\User;

    Route::get('/users/{user}', function (User $user) {
        return $user->email;
    });

Karena variabel $user diberi tipe-hint sebagai model Eloquent App\Models\User dan nama variabel cocok dengan segmen URI {user}, Laravel akan secara otomatis menyuntikkan instance model yang memiliki ID yang cocok dengan nilai yang sesuai dari URI permintaan. Jika instance model yang cocok tidak ditemukan di database, tanggapan HTTP 404 akan otomatis dihasilkan.

Since the `$user` variable is type-hinted as the `App\Models\User` Eloquent model and the variable name matches the `{user}` URI segment, Laravel will automatically inject the model instance that has an ID matching the corresponding value from the request URI. If a matching model instance is not found in the database, a 404 HTTP response will automatically be generated.

Tentu saja, binding implicit juga mungkin saat menggunakan metode kontroller. Lagi-lagi, perhatikan bahwa segmen URI {user} cocok dengan variabel $user di kontroller yang berisi tipe-hint App\Models\User:

Of course, implicit binding is also possible when using controller methods. Again, note the `{user}` URI segment matches the `$user` variable in the controller which contains an `App\Models\User` type-hint:

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
#### Customizing The Key

Kadang-kadang Anda mungkin ingin menyelesaikan model Eloquent menggunakan kolom selain id. Untuk melakukannya, Anda dapat menentukan kolom dalam definisi parameter rute:

Sometimes you may wish to resolve Eloquent models using a column other than `id`. To do so, you may specify the column in the route parameter definition:

    use App\Models\Post;

    Route::get('/posts/{post:slug}', function (Post $post) {
        return $post;
    });

Jika Anda ingin model binding selalu menggunakan kolom database selain id saat mengambil kelas model tertentu, Anda dapat menimpa method getRouteKeyName pada model Eloquent:

If you would like model binding to always use a database column other than `id` when retrieving a given model class, you may override the `getRouteKeyName` method on the Eloquent model:

    /**
     * Get the route key for the model.
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
#### Customizing Missing Model Behavior

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

You may define a route that will only be invoked if the `{category}` route segment is `fruits` or `people`. Otherwise, Laravel will return a 404 HTTP response:

```php
use App\Enums\Category;
use Illuminate\Support\Facades\Route;

Route::get('/categories/{category}', function (Category $category) {
    return $category->value;
});
```

<a name="explicit-binding"></a>
### Explicit Binding

You are not required to use Laravel's implicit, convention based model resolution in order to use model binding. You can also explicitly define how route parameters correspond to models. To register an explicit binding, use the router's `model` method to specify the class for a given parameter. You should define your explicit model bindings at the beginning of the `boot` method of your `RouteServiceProvider` class:

    use App\Models\User;
    use Illuminate\Support\Facades\Route;

    /**
     * Define your route model bindings, pattern filters, etc.
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

Since we have bound all `{user}` parameters to the `App\Models\User` model, an instance of that class will be injected into the route. So, for example, a request to `users/1` will inject the `User` instance from the database which has an ID of `1`.

If a matching model instance is not found in the database, a 404 HTTP response will be automatically generated.

<a name="customizing-the-resolution-logic"></a>
#### Customizing The Resolution Logic

If you wish to define your own model binding resolution logic, you may use the `Route::bind` method. The closure you pass to the `bind` method will receive the value of the URI segment and should return the instance of the class that should be injected into the route. Again, this customization should take place in the `boot` method of your application's `RouteServiceProvider`:

    use App\Models\User;
    use Illuminate\Support\Facades\Route;

    /**
     * Define your route model bindings, pattern filters, etc.
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
## Fallback Routes

Using the `Route::fallback` method, you may define a route that will be executed when no other route matches the incoming request. Typically, unhandled requests will automatically render a "404" page via your application's exception handler. However, since you would typically define the `fallback` route within your `routes/web.php` file, all middleware in the `web` middleware group will apply to the route. You are free to add additional middleware to this route as needed:

    Route::fallback(function () {
        //
    });

> **Warning**  
> The fallback route should always be the last route registered by your application.

<a name="rate-limiting"></a>
## Rate Limiting

<a name="defining-rate-limiters"></a>
### Defining Rate Limiters

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

If the incoming request exceeds the specified rate limit, a response with a 429 HTTP status code will automatically be returned by Laravel. If you would like to define your own response that should be returned by a rate limit, you may use the `response` method:

    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000)->response(function (Request $request, array $headers) {
            return response('Custom response...', 429, $headers);
        });
    });

Since rate limiter callbacks receive the incoming HTTP request instance, you may build the appropriate rate limit dynamically based on the incoming request or authenticated user:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()->vipCustomer()
                    ? Limit::none()
                    : Limit::perMinute(100);
    });

<a name="segmenting-rate-limits"></a>
#### Segmenting Rate Limits

Sometimes you may wish to segment rate limits by some arbitrary value. For example, you may wish to allow users to access a given route 100 times per minute per IP address. To accomplish this, you may use the `by` method when building your rate limit:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()->vipCustomer()
                    ? Limit::none()
                    : Limit::perMinute(100)->by($request->ip());
    });

To illustrate this feature using another example, we can limit access to the route to 100 times per minute per authenticated user ID or 10 times per minute per IP address for guests:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()
                    ? Limit::perMinute(100)->by($request->user()->id)
                    : Limit::perMinute(10)->by($request->ip());
    });

<a name="multiple-rate-limits"></a>
#### Multiple Rate Limits

If needed, you may return an array of rate limits for a given rate limiter configuration. Each rate limit will be evaluated for the route based on the order they are placed within the array:

    RateLimiter::for('login', function (Request $request) {
        return [
            Limit::perMinute(500),
            Limit::perMinute(3)->by($request->input('email')),
        ];
    });

<a name="attaching-rate-limiters-to-routes"></a>
### Attaching Rate Limiters To Routes

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

Typically, the `throttle` middleware is mapped to the `Illuminate\Routing\Middleware\ThrottleRequests` class. This mapping is defined in your application's HTTP kernel (`App\Http\Kernel`). However, if you are using Redis as your application's cache driver, you may wish to change this mapping to use the `Illuminate\Routing\Middleware\ThrottleRequestsWithRedis` class. This class is more efficient at managing rate limiting using Redis:

    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,

<a name="form-method-spoofing"></a>
## Form Method Spoofing

HTML forms do not support `PUT`, `PATCH`, or `DELETE` actions. So, when defining `PUT`, `PATCH`, or `DELETE` routes that are called from an HTML form, you will need to add a hidden `_method` field to the form. The value sent with the `_method` field will be used as the HTTP request method:

    <form action="/example" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

For convenience, you may use the `@method` [Blade directive](/docs/{{version}}/blade) to generate the `_method` input field:

    <form action="/example" method="POST">
        @method('PUT')
        @csrf
    </form>

<a name="accessing-the-current-route"></a>
## Accessing The Current Route

You may use the `current`, `currentRouteName`, and `currentRouteAction` methods on the `Route` facade to access information about the route handling the incoming request:

    use Illuminate\Support\Facades\Route;

    $route = Route::current(); // Illuminate\Routing\Route
    $name = Route::currentRouteName(); // string
    $action = Route::currentRouteAction(); // string

You may refer to the API documentation for both the [underlying class of the Route facade](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) and [Route instance](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html) to review all of the methods that are available on the router and route classes.

<a name="cors"></a>
## Cross-Origin Resource Sharing (CORS)

Laravel can automatically respond to CORS `OPTIONS` HTTP requests with values that you configure. All CORS settings may be configured in your application's `config/cors.php` configuration file. The `OPTIONS` requests will automatically be handled by the `HandleCors` [middleware](/docs/{{version}}/middleware) that is included by default in your global middleware stack. Your global middleware stack is located in your application's HTTP kernel (`App\Http\Kernel`).

> **Note**  
> For more information on CORS and CORS headers, please consult the [MDN web documentation on CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#The_HTTP_response_headers).

<a name="route-caching"></a>
## Route Caching

When deploying your application to production, you should take advantage of Laravel's route cache. Using the route cache will drastically decrease the amount of time it takes to register all of your application's routes. To generate a route cache, execute the `route:cache` Artisan command:

```shell
php artisan route:cache
```

After running this command, your cached routes file will be loaded on every request. Remember, if you add any new routes you will need to generate a fresh route cache. Because of this, you should only run the `route:cache` command during your project's deployment.

You may use the `route:clear` command to clear the route cache:

```shell
php artisan route:clear
```
