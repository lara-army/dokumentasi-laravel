# CSRF Protection

- [Pendahuluan](#csrf-introduction)
- [Mencegah _request_ CSRF](#preventing-csrf-requests)
    - [Excluding URIs](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## Pendahuluan


Cross-site request forgeries adalah jenis eksploitasi berbahaya di mana perintah yang tidak sah dilakukan atas nama pengguna yang terautentikasi. Untungnya, Laravel memudahkan untuk melindungi aplikasi Anda dari serangan [cross-site request forgery] (https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF).

Cross-site request forgeries are a type of malicious exploit whereby unauthorized commands are performed on behalf of an authenticated user. Thankfully, Laravel makes it easy to protect your application from [cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF) attacks.

<a name="csrf-explanation"></a>
#### An Explanation Of The Vulnerability

Jika Anda tidak terbiasa dengan pemalsuan permintaan lintas-situs, mari kita bahas contoh bagaimana kerentanan ini dapat dieksploitasi. Bayangkan aplikasi Anda memiliki rute `/pengguna/email` yang menerima permintaan `POST` untuk mengubah alamat email pengguna yang terotentikasi. Kemungkinan besar, rute ini mengharapkan kolom input `email` berisi alamat email yang ingin mulai digunakan pengguna.

In case you're not familiar with cross-site request forgeries, let's discuss an example of how this vulnerability can be exploited. Imagine your application has a `/user/email` route that accepts a `POST` request to change the authenticated user's email address. Most likely, this route expects an `email` input field to contain the email address the user would like to begin using.

Tanpa perlindungan CSRF, situs web berbahaya dapat membuat formulir HTML yang mengarah ke rute `/user/email` aplikasi Anda dan mengirimkan alamat email pengguna berbahaya itu sendiri:

Without CSRF protection, a malicious website could create an HTML form that points to your application's `/user/email` route and submits the malicious user's own email address:

```blade
<form action="https://your-application.com/user/email" method="POST">
    <input type="email" value="malicious-email@example.com">
</form>

<script>
    document.forms[0].submit();
</script>
```

Jika situs web berbahaya secara otomatis mengirimkan formulir ketika halaman dimuat, pengguna jahat hanya perlu memikat pengguna aplikasi Anda yang tidak menaruh curiga untuk mengunjungi situs web mereka dan alamat email mereka akan diubah dalam aplikasi Anda.

 If the malicious website automatically submits the form when the page is loaded, the malicious user only needs to lure an unsuspecting user of your application to visit their website and their email address will be changed in your application.

Untuk mencegah kerentanan ini, kita perlu memeriksa setiap permintaan `POST`, `PUT`, `PATCH`, atau `DELETE` yang masuk untuk nilai sesi rahasia yang tidak dapat diakses oleh aplikasi jahat.

 To prevent this vulnerability, we need to inspect every incoming `POST`, `PUT`, `PATCH`, or `DELETE` request for a secret session value that the malicious application is unable to access.

<a name="preventing-csrf-requests"></a>
## Mencegah _request_ CSRF

Laravel secara otomatis menghasilkan "token" CSRF untuk setiap [sesi pengguna] aktif (/docs/{{version}}/session) yang dikelola oleh aplikasi. Token ini digunakan untuk memverifikasi bahwa pengguna yang diautentikasi adalah orang yang benar-benar membuat permintaan ke aplikasi. Karena token ini disimpan dalam sesi pengguna dan berubah setiap kali sesi diregenerasi, aplikasi jahat tidak dapat mengaksesnya.

Laravel automatically generates a CSRF "token" for each active [user session](/docs/{{version}}/session) managed by the application. This token is used to verify that the authenticated user is the person actually making the requests to the application. Since this token is stored in the user's session and changes each time the session is regenerated, a malicious application is unable to access it.

Token CSRF sesi saat ini dapat diakses melalui sesi permintaan atau melalui fungsi pembantu `csrf_token`:

The current session's CSRF token can be accessed via the request's session or via the `csrf_token` helper function:

    use Illuminate\Http\Request;

    Route::get('/token', function (Request $request) {
        $token = $request->session()->token();

        $token = csrf_token();

        // ...
    });

Setiap kali Anda mendefinisikan formulir HTML "POST", "PUT", "PATCH", atau "DELETE" dalam aplikasi Anda, Anda harus menyertakan bidang `_token` CSRF tersembunyi dalam formulir sehingga middleware perlindungan CSRF dapat memvalidasi permintaan. Untuk kenyamanan, Anda dapat menggunakan arahan Blade `@csrf` untuk menghasilkan bidang input token tersembunyi:

Anytime you define a "POST", "PUT", "PATCH", or "DELETE" HTML form in your application, you should include a hidden CSRF `_token` field in the form so that the CSRF protection middleware can validate the request. For convenience, you may use the `@csrf` Blade directive to generate the hidden token input field:

```blade
<form method="POST" action="/profile">
    @csrf

    <!-- Equivalent to... -->
    <input type="hidden" name="_token" value="{{ csrf_token() }}" />
</form>
```

`App\Http\Middleware\VerifyCsrfToken` [middleware](/docs/{{{version}}}/middleware), yang termasuk dalam grup middleware `web` secara default, akan secara otomatis memverifikasi bahwa token dalam input permintaan cocok dengan token yang tersimpan dalam sesi. Ketika kedua token ini cocok, kita tahu bahwa pengguna yang terotentikasi adalah yang memulai permintaan.

The `App\Http\Middleware\VerifyCsrfToken` [middleware](/docs/{{version}}/middleware), which is included in the `web` middleware group by default, will automatically verify that the token in the request input matches the token stored in the session. When these two tokens match, we know that the authenticated user is the one initiating the request.

<a name="csrf-tokens-and-spas"></a>
### CSRF Tokens & SPAs

Jika Anda membangun SPA yang menggunakan Laravel sebagai backend API, Anda harus berkonsultasi dengan [dokumentasi Laravel Sanctum](/docs/{{{version}}/sanctum) untuk informasi tentang otentikasi dengan API Anda dan melindungi dari kerentanan CSRF.

If you are building a SPA that is utilizing Laravel as an API backend, you should consult the [Laravel Sanctum documentation](/docs/{{version}}/sanctum) for information on authenticating with your API and protecting against CSRF vulnerabilities.

<a name="csrf-excluding-uris"></a>
### Excluding URIs From CSRF Protection

Terkadang Anda mungkin ingin mengecualikan sekumpulan URI dari perlindungan CSRF. Misalnya, jika Anda menggunakan [Stripe](https://stripe.com) untuk memproses pembayaran dan menggunakan sistem webhook mereka, Anda perlu mengecualikan rute handler webhook Stripe Anda dari perlindungan CSRF karena Stripe tidak akan tahu token CSRF apa yang akan dikirim ke rute Anda.

Sometimes you may wish to exclude a set of URIs from CSRF protection. For example, if you are using [Stripe](https://stripe.com) to process payments and are utilizing their webhook system, you will need to exclude your Stripe webhook handler route from CSRF protection since Stripe will not know what CSRF token to send to your routes.

Biasanya, Anda harus menempatkan rute-rute semacam ini di luar grup middleware `web` yang diterapkan oleh `App\Providers\RouteServiceProvider` ke semua rute dalam file `routes/web.php`. Namun, Anda juga dapat mengecualikan rute-rute tersebut dengan menambahkan URI mereka ke properti `$except` dari middleware `VerifyCsrfToken`:

Typically, you should place these kinds of routes outside of the `web` middleware group that the `App\Providers\RouteServiceProvider` applies to all routes in the `routes/web.php` file. However, you may also exclude the routes by adding their URIs to the `$except` property of the `VerifyCsrfToken` middleware:

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as Middleware;

    class VerifyCsrfToken extends Middleware
    {
        /**
         * The URIs that should be excluded from CSRF verification.
         *
         * @var array
         */
        protected $except = [
            'stripe/*',
            'http://example.com/foo/bar',
            'http://example.com/foo/*',
        ];
    }

> **Catatan**  

Untuk kenyamanan, middleware CSRF secara otomatis dinonaktifkan untuk semua rute ketika [menjalankan tes](/docs/{{version}}/testing).

> For convenience, the CSRF middleware is automatically disabled for all routes when [running tests](/docs/{{version}}/testing).

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

Selain memeriksa token CSRF sebagai parameter POST, middleware `App\Http\Middleware\VerifyCsrfToken` juga akan memeriksa header permintaan `X-CSRF-TOKEN`. Anda bisa, misalnya, menyimpan token dalam tag `meta` HTML:

In addition to checking for the CSRF token as a POST parameter, the `App\Http\Middleware\VerifyCsrfToken` middleware will also check for the `X-CSRF-TOKEN` request header. You could, for example, store the token in an HTML `meta` tag:

```blade
<meta name="csrf-token" content="{{ csrf_token() }}">
```

Kemudian, Anda dapat menginstruksikan library seperti jQuery untuk secara otomatis menambahkan token ke semua header permintaan. Ini memberikan perlindungan CSRF yang sederhana dan nyaman untuk aplikasi berbasis AJAX Anda yang menggunakan teknologi JavaScript lama:

Then, you can instruct a library like jQuery to automatically add the token to all request headers. This provides simple, convenient CSRF protection for your AJAX based applications using legacy JavaScript technology:

```js
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravel menyimpan token CSRF saat ini dalam cookie `XSRF-TOKEN` terenkripsi yang disertakan dengan setiap respons yang dihasilkan oleh framework. Anda dapat menggunakan nilai cookie untuk mengatur header permintaan `X-XSRF-TOKEN`.

Laravel stores the current CSRF token in an encrypted `XSRF-TOKEN` cookie that is included with each response generated by the framework. You can use the cookie value to set the `X-XSRF-TOKEN` request header.

Cookie ini terutama dikirim sebagai kenyamanan pengembang karena beberapa kerangka kerja dan pustaka JavaScript, seperti Angular dan Axios, secara otomatis menempatkan nilainya di tajuk `X-XSRF-TOKEN` pada permintaan asal yang sama.

This cookie is primarily sent as a developer convenience since some JavaScript frameworks and libraries, like Angular and Axios, automatically place its value in the `X-XSRF-TOKEN` header on same-origin requests.

> **Catatan**  

Secara default, file `resources/js/bootstrap.js` menyertakan pustaka HTTP Axios yang secara otomatis akan mengirimkan header `X-XSRF-TOKEN` untuk Anda.

> By default, the `resources/js/bootstrap.js` file includes the Axios HTTP library which will automatically send the `X-XSRF-TOKEN` header for you.
