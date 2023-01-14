# CSRF Protection

- [Pendahuluan](#csrf-introduction)
- [Mencegah _request_ CSRF](#preventing-csrf-requests)
    - [Pengecualian URI CSRF](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## Pendahuluan

_Cross-site request forgeries_ adalah jenis eksploitasi berbahaya di mana perintah yang tidak sah dilakukan atas nama pengguna yang terautentikasi. Untungnya, Laravel telah memudahkan perlindungan aplikasi Anda dari serangan [_cross-site request forgery_](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF).

<a name="csrf-explanation"></a>
#### Penjelasan Kerentanan

Jika Anda tidak belum akrab dengan istilah _cross-site request forgery_ (CSRF), mari kita bahas contoh bagaimana kerentanan ini dapat dieksploitasi. Bayangkan apabila aplikasi Anda memiliki _route_ `/pengguna/email` yang menerima _request_ `POST` untuk mengubah alamat email pengguna yang terotentikasi. Kemungkinan besar, _route_ ini mengharapkan kolom input `email` yang berisi alamat email yang ingin didaftarkan oleh pengguna.

Tanpa perlindungan CSRF, situs web berbahaya dapat membuat formulir HTML yang mengarah ke _route_ `/user/email` aplikasi Anda dan mengirimkan alamat email "jahat" milik peretas:

```blade
<form action="https://your-application.com/user/email" method="POST">
    <input type="email" value="malicious-email@example.com">
</form>

<script>
    document.forms[0].submit();
</script>
```

Jika situs web berbahaya dapat mengirimkan formulir secara otomatis ketika halaman dimuat, pengguna jahat hanya perlu memperdaya pengguna (asli) aplikasi Anda yang tidak menaruh curiga untuk mengunjungi situs web mereka, kemudian alamat email mereka pada aplikasi Anda akan tertimpa oleh alamat email "jahat" milik peretas.

Untuk mencegah kerentanan ini, kita perlu memeriksa nilai rahasia sesi pada setiap _request_ `POST`, `PUT`, `PATCH`, atau `DELETE` yang masuk. Nilai rahasia yang tidak dapat diakses oleh aplikasi jahat.

<a name="preventing-csrf-requests"></a>
## Mencegah _request_ CSRF

Laravel secara otomatis menghasilkan "token" CSRF untuk setiap [sesi pengguna](/docs/{{version}}/session) yang aktif yang dikelola oleh aplikasi secara otomatis. Token ini digunakan untuk memverifikasi bahwa pengguna yang telah terautentikasi adalah orang yang benar-benar membuat _request_ ke aplikasi. Karena token ini disimpan dalam sesi pengguna dan berubah setiap kali sesi diregenerasi (dihasilkan ulang), aplikasi jahat tidak akan dapat mengaksesnya.

Token CSRF untuk sesi saat ini dapat diakses melalui _request_ milik sesi atau melalui fungsi _helper_ `csrf_token`:

    use Illuminate\Http\Request;

    Route::get('/token', function (Request $request) {
        $token = $request->session()->token();

        $token = csrf_token();

        // ...
    });

Setiap kali Anda mendefinisikan formulir HTML "POST", "PUT", "PATCH", atau "DELETE" dalam aplikasi Anda, Anda harus menyertakan kolom `_token` CSRF yang tersembunyi pada formulir tersebut sehingga _middleware_ untuk perlindungan CSRF dapat memvalidasi _request_ untuk formulir tersebut. Untuk kemudahan, Anda dapat menggunakan _directive_ `@csrf` milik Blade untuk menghasilkan kolom input token yang tersembunyi:

```blade
<form method="POST" action="/profile">
    @csrf

    <!-- Setara dengan... -->
    <input type="hidden" name="_token" value="{{ csrf_token() }}" />
</form>
```

[Middleware](/docs/{{version}}/middleware) `App\Http\Middleware\VerifyCsrfToken` yang telah termasuk di dalam grup _middleware_ `web` secara _default_, akan secara otomatis memverifikasi bahwa token di dalam _input_ pada _request_ telah sesuai dengan token yang tersimpan di dalam sesi. Ketika kedua token ini cocok, maka kita mengetahui bahwa pengguna yang terotentikasi adalah yang pengguna yang melakukan _request_.

<a name="csrf-tokens-and-spas"></a>
### Token CSRF & SPA

Jika Anda membangun sebuah SPA yang menggunakan Laravel sebagai _backend_ API, Anda sebaiknya mengunjungi [dokumentasi Laravel Sanctum](/docs/{{version}}/sanctum) untuk informasi tentang autentikasi dengan API dan perlindungan terhadap kerentanan CSRF pada aplikasi Anda.

<a name="csrf-excluding-uris"></a>
### Pengecualian URI dari Perlindungan CSRF

Terkadang Anda mungkin ingin mengecualikan sekumpulan URI dari perlindungan CSRF. Misalnya, jika Anda menggunakan [Stripe](https://stripe.com) untuk memproses pembayaran dan menggunakan sistem _webhook_ mereka, Anda perlu mengecualikan rute _handler_ untuk _webhook_ milik Stripe dari perlindungan CSRF karena Stripe tidak pernah tahu token CSRF apa yang perlu dikirim ke rute Anda.

Biasanya, Anda harus menempatkan rute-rute semacam ini di luar grup _middleware_ `web` yang diterapkan oleh `App\Providers\RouteServiceProvider` ke semua rute yang terdapat di dalam _file_ `routes/web.php`. Atau, Anda juga dapat mengecualikan rute-rute tersebut dengan menambahkan URI tersebut ke properti `$except` pada _middleware_ `VerifyCsrfToken`:

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as Middleware;

    class VerifyCsrfToken extends Middleware
    {
        /**
         * URI yang seharusnya dikecualikan dari verifikasi CSRF.
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
> Untuk kenyamanan, _middleware_ CSRF secara otomatis dinonaktifkan pada semua rute ketika [melakukan pengujian](/docs/{{version}}/testing).

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

Selain memeriksa token CSRF sebagai parameter POST, _middleware_ `App\Http\Middleware\VerifyCsrfToken` juga akan memeriksa _header_ `X-CSRF-TOKEN` pada _request_. Misalnya, Anda dapat menyimpan token di dalam tag `meta` HTML:

```blade
<meta name="csrf-token" content="{{ csrf_token() }}">
```

Kemudian, Anda dapat menginstruksikan _library_ seperti jQuery untuk menambahkan token secara otomatis ke semua _header_ untuk _request_. Hal ini bertujuan untuk memberikan perlindungan CSRF yang sederhana dan nyaman untuk aplikasi berbasis AJAX yang menggunakan teknologi JavaScript lama

```js
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravel menyimpan token CSRF saat ini yang telah terenkripsi di dalam _cookie_ `XSRF-TOKEN` yang disertakan pada setiap respons yang dihasilkan oleh _framework_. Anda dapat menggunakan nilai _cookie_ ini untuk menetapkan nilai `X-XSRF-TOKEN` pada _header request_.

Pada dasarnya _cookie_ ini dikirim untuk kenyamanan pengembang karena beberapa _framework_ dan _library_ JavaScript, seperti Angular dan Axios, secara otomatis menempatkan nilai tersebut di _header_ `X-XSRF-TOKEN` untuk asal _request_ yang sama (_same-origin request_).

> **Catatan**  
> Secara _default_, _file_ `resources/js/bootstrap.js` menyertakan pustaka HTTP Axios yang secara otomatis akan mengirimkan _header_ `X-XSRF-TOKEN` untuk Anda.
