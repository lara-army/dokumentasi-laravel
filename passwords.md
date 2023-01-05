# Melakukan Reset Kata Sandi

- [Pengantar](#introduction)
    - [Menyiapkan Model](#model-preparation)
    - [Menyiapkan Basis Data](#database-preparation)
    - [Melakukan Konfigurasi _Host_ Terpercaya](#configuring-trusted-hosts)
- [Melakukan _Route_](#routing)
    - [Meminta Pranala Reset Kata Sandi](#requesting-the-password-reset-link)
    - [Mengatur Ulang Kata Sandi](#resetting-the-password)
- [Menghapus Token yang Kadaluarsa](#deleting-expired-tokens)
- [Kostumisasi](#password-customization)

<a name="introduction"></a>
## Pengantar

Aplikasi web pada umumnya menyediakan sebuah cara unt

Kebanyakan aplikasi web menyediakan sebuah cara bagi pengguna untuk mengatur ulang kata sandi yang terlupakan. Daripada memaksamu untuk mengimplementasikan ini secara manual, Laravel menyediakan layanan yang nyaman untuk mengirim pranala reset kata sandi dan reset kata sandi yang aman.

> **Catatan**  
> Ingin persiapan yang lebih mudah? _Install_ [perlengkapan pemula Laravel](/docs/{{version}}/starter-kits) pada aplikasi Laravel yang baru. Perlengkapan pemula Laravel akan mengurus _scaffolding_ seluruh sistem autentikasi Anda, termasuk pengaturan ulang kata sandi yang terlupa.

<a name="model-preparation"></a>
### Menyiapkan Model

Sebelum menggunakan fitur reset kata sandi milik Laravel, model `App\Models\User` pada aplikasi Anda harus menggunakan _trait_ `Illuminate\Notifications\Notifiable`. Biasanya, _trait_ ini sudah terdapat pada model `App\Models\User` secara _default_ yang dibuat pada aplikasi Laravel yang baru.

Selanjutnya, pastikan bahwa model `App\Models\User` telah mengimplementasikan kontrak `Illuminate\Contracts\Auth\CanResetPassword`. Model `App\Models\User` yang disertakan dalam _framework_ sudah mengimplementasikan _interface_ ini, dan menggunakan _trait_ `Illuminate\Auth\Passwords\CanResetPassword` untuk menyertakan metode yang dibutuhkan untuk mengimplementasikan _interface_ tersebut.

<a name="database-preparation"></a>
### Menyiapkan Basis Data

Sebuah tabel harus dibuat untuk menyimpan token reset password aplikasi Anda. _Migration_ untuk tabel ini telah disertakan juga dalam aplikasi Laravel yang _default_, sehingga Anda hanya perlu melakukan migrasi basis data Anda untuk membuat tabel ini:

```shell
php artisan migrate
```

<a name="configuring-trusted-hosts"></a>
### Konfigurasi _Host_ Terpercaya

Secara default, Laravel akan merespons semua permintaan yang diterimanya tanpa memperhatikan konten _header_ `Host` pada permintaan HTTP. Selain itu, nilai _header_ `Host` akan digunakan saat menghasilkan URL absolut ke aplikasi Anda selama permintaan web.

Biasanya, Anda harus mengkonfigurasi web server Anda, seperti Nginx atau Apache, untuk mengirim permintaan ke aplikasi yang cocok dengan nama _host_ yang diberikan. Namun, jika Anda tidak memiliki kemampuan untuk melakukan konfigurasi web server Anda secara langsung dan perlu memberi instruksi kepada Laravel untuk hanya merespon nama _host_ tertentu, Anda dapat melakukannya dengan mengaktifkan _middleware_ `App\Http\Middleware\TrustHosts` untuk aplikasi Anda. Hal ini sangat penting ketika aplikasi Anda menawarkan fitur reset kata sandi.

Untuk mempelajari lebih lanjut tentang _middleware_ ini, sila baca [dokumentasi middleware `TrustHosts`](/docs/{{version}}/requests#configuring-trusted-hosts).

<a name="routing"></a>
## MMelakukan _Route_

Untuk mengimplementasikan dukungan secara tepat yang mengizinkan pengguna untuk mengatur ulang kata sandi mereka, kita perlu menentukan beberapa _route_. Pertama, kita perlu sepasang _route_ untuk menangani perizinan pengguna untuk meminta pranala reset kata sandi melalui alamat email mereka. Kedua, kita perlu sepasang _route_ untuk menangani pengaturan ulang kata sandi yang diakses pengguna dengan cara mengunjungi tautan reset kata sandi yang dikirimkan kepada mereka melalui email dan menyelesaikan formulir reset kata sandi.

To properly implement support for allowing users to reset their passwords, we will need to define several routes. First, we will need a pair of routes to handle allowing the user to request a password reset link via their email address. Second, we will need a pair of routes to handle actually resetting the password once the user visits the password reset link that is emailed to them and completes the password reset form.

<a name="requesting-the-password-reset-link"></a>
### Requesting The Password Reset Link

<a name="the-password-reset-link-request-form"></a>
#### Formulir Permintaan Pranala Reset Kata Sandi

Pertama, kita akan mendefinisikan _route_ yang dibutuhkan untuk meminta tautan reset kata sandi. Untuk memulai, kita akan mendefinisikan _route_ yang mengembalikan _view_ formulir permintaan pranala untuk reset kata sandi:

    Route::get('/forgot-password', function () {
        return view('auth.forgot-password');
    })->middleware('guest')->name('password.request');

Tampilan yang dikembalikan oleh _route_ ini harus memiliki formulir yang berisi _field_ `email`, yang akan memungkinkan pengguna untuk meminta pranala reset kata sandi untuk alamat email yang diberikan.

<a name="password-reset-link-handling-the-form-submission"></a>
#### Penanganan Formulir yang Dikirim

Selanjutnya, kita akan mendefinisikan _route_ yang menangani formulir yang dikirimkan dari tampilan "lupa kata sandi". _Route_ ini akan bertanggung jawab untuk memvalidasi alamat email dan mengirimkan pranala reset kata sandi ke alamat email yang dikirimkan:

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Password;

    Route::post('/forgot-password', function (Request $request) {
        $request->validate(['email' => 'required|email']);

        $status = Password::sendResetLink(
            $request->only('email')
        );

        return $status === Password::RESET_LINK_SENT
                    ? back()->with(['status' => __($status)])
                    : back()->withErrors(['email' => __($status)]);
    })->middleware('guest')->name('password.email');

Sebelum lanjut, mari kita perhatikan _route_ ini lebih detail. Pertama, atribut `email` milik _request_ divalidasi. Selanjutnya, kita akan menggunakan "password broker" bawaan Laravel (melalui _facade_ `Password`) untuk mengirim pranala reset kata sandi ke pengguna. _Password broker_ akan menangani pencarian pengguna dengan _field_ yang sudah ditetapkan (dalam hal ini, alamat email) dan mengirimkan tautan reset kata sandi ke pengguna melalui [sistem notifikasi](/docs/{{version}}/notifications) bawaan Laravel.

_Method_ `sendResetLink` mengembalikan _slug,_ "status". Status ini dapat diterjemahkan menggunakan bantuan [lokalisasi](/docs/{{version}}/localization) Laravel untuk menampilkan pesan yang ramah pengguna tentang status permintaan mereka. Terjemahan status reset kata sandi ditentukan oleh _file_ bahasa `lang/{lang}/passwords.php` aplikasi Anda. Entri untuk setiap kemungkinan nilai untuk _slug_ status terletak di dalam _file_ bahasa `passwords`.

Anda mungkin bertanya-tanya bagaimana Laravel tahu bagaimana mengambil _record_ pengguna dari basis data aplikasi Anda saat memanggil metode `sendResetLink` milik _facade_ `Password`. _Password broker_ Laravel menggunakan "_user provider_" milik sistem autentikasi untuk mengambil _record_ di dalam basis data. _User provider_ yang digunakan oleh _password broker_ dikonfigurasi dalam _array_ konfigurasi `passwords` pada _file_ `config/auth.php`. Untuk mempelajari lebih lanjut tentang menulis _user providers_ yang kustom, sila mengunjungi [dokumentasi autentikasi](/docs/{{version}}/authentication#adding-custom-user-providers).

> **Catatan**  
> Saat mengimplementasikan reset kata sandi secara manual, Anda harus mendefinisikan isi tampilan dan rute sendiri. Jika Anda ingin scaffolding yang mencakup semua logika autentikasi dan verifikasi yang dibutuhkan, periksalah [perlengkapan pemula](/docs/{{version}}/starter-kits) aplikasi Laravel.

<a name="resetting-the-password"></a>
### Melakukan Reset Kata Sandi

<a name="the-password-reset-form"></a>
#### Formulir Reset Kata Sandi

Selanjutnya, kita akan mendefinisikan _route_ yang dibutuhkan untuk mereset kata sandi yang diakses pengguna melalui pranala reset kata sandi yang telah dikirimkan kepada mereka melalui email dan memberikan kata sandi baru. Pertama, mari kita definisikan _route_ yang akan menampilkan formulir reset kata sandi yang ditampilkan saat pengguna mengklik tautan reset kata sandi. Rute ini akan menerima parameter `token` yang akan kita gunakan nanti untuk memverifikasi permintaan reset kata sandi:

    Route::get('/reset-password/{token}', function ($token) {
        return view('auth.reset-password', ['token' => $token]);
    })->middleware('guest')->name('password.reset');

Tampilan yang dikembalikan oleh rute ini harus menampilkan formulir yang berisi field `email`, `password`, `password_confirmation`, dan _field_ `token` yang tersembunyi, yang harus berisi nilai `$token` rahasia yang diterima oleh _route_ kita.

<a name="password-reset-handling-the-form-submission"></a>
#### Penanganan Formulir yang Dikirim

Tentu saja, kita perlu mendefinisikan _route_ untuk menangani _submit_ formulir reset kata sandi. _Route_ ini akan bertanggung jawab untuk memvalidasi permintaan yang masuk dan memperbarui kata sandi pengguna di basis data:

    use Illuminate\Auth\Events\PasswordReset;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use Illuminate\Support\Facades\Password;
    use Illuminate\Support\Str;

    Route::post('/reset-password', function (Request $request) {
        $request->validate([
            'token' => 'required',
            'email' => 'required|email',
            'password' => 'required|min:8|confirmed',
        ]);

        $status = Password::reset(
            $request->only('email', 'password', 'password_confirmation', 'token'),
            function ($user, $password) {
                $user->forceFill([
                    'password' => Hash::make($password)
                ])->setRememberToken(Str::random(60));

                $user->save();

                event(new PasswordReset($user));
            }
        );

        return $status === Password::PASSWORD_RESET
                    ? redirect()->route('login')->with('status', __($status))
                    : back()->withErrors(['email' => [__($status)]]);
    })->middleware('guest')->name('password.update');

Sebelum melanjutkan, mari kita perhatikan _route_ ini lebih detail. Pertama, aatribu `token`, `email`, dan `password` milik _request_ divalidasi. Selanjutnya, kita akan menggunakan "_password broker_" bawaan Laravel (melalui _facade_ `Password`) untuk memvalidasi _credential_ dari permintaan reset kata sandi.

Jika token, alamat email, dan kata sandi yang diberikan kepada _password broker_ adalah valid, _closure_ yang diberikan ke _method_ `reset` akan dijalankan. _Closure_ ini menerima _instance_ pengguna dan kata sandi mentah yang dikirimkan melalui formulir reset kata sandi, kita dapat memperbarui kata sandi pengguna di basis data.

_Method_ `reset` mengembalikan _slug_ "status". Status ini dapat diterjemahkan menggunakan bantuan [lokalisasi](/docs/{{version}}/localization) Laravel untuk menampilkan pesan yang ramah pengguna tentang status permintaan mereka. Terjemahan status reset kata sandi ditentukan oleh _file_ bahasa `lang/{lang}/passwords.php` pada aplikasi Anda. Entri untuk setiap kemungkinan nilai dari _slug_ status terletak di dalam file bahasa `passwords`.

Sebelum lanjut, Anda mungkin bertanya-tanya bagaimana Laravel tahu bagaimana mengambil _record_ pengguna dari basos data saat memanggil _method_ `reset` pada _facade_ `Password`. _Password broker_ milik Laravel menggunakan "_user provider_" milik sistem autentikasi yang mengambil _record_ di basis data. _User provider_ yang digunakan oleh _password broker_ dikonfigurasi dalam array `passwords` pada file konfigurasi `config/auth.php`. Untuk mempelajari lebih lanjut tentang menulis _user provider_ yang kustom, sila mengunjungi [dokumentasi autentikasi](docs/{{version}}/authentication#adding-custom-user-providers).

<a name="deleting-expired-tokens"></a>
## Menghapus Token yang Kadaluarsa

Token reset kata sandi yang telah kadaluwarsa akan tetap ada di basis data Anda. Namun, Anda dapat dengan mudah menghapus _record_ ini dengan menggunakan perintah Artisan `auth:clear-resets`:

```shell
php artisan auth:clear-resets
```

Jika Anda ingin mengotomatiskan proses ini, pertimbangkan menambahkan perintah ke [_scheduler_](/docs/{{version}}/scheduling) pada aplikasi Anda:

    $schedule->command('auth:clear-resets')->everyFifteenMinutes();

<a name="password-customization"></a>
## Kostumisasi

<a name="reset-link-customization"></a>
#### Kostumosasi Pranala Reset

Anda dapat menyesuaikan URL pranala reset kata sandi menggunakan _method_ `createUrlUsing` yang disediakan oleh _class_ notifikasi `ResetPassword`. Metode ini menerima _closure_ yang menerima _instance class_ pengguna yang menerima notifikasi serta token pranala reset kata sandi. Biasanya, Anda harus memanggil _method_ ini dari _method_ `boot` milik _service provider_ `App\Providers\AuthServiceProvider`:

    use Illuminate\Auth\Notifications\ResetPassword;

    /**
     * Mendaftarkan layanan autentikasi / autorisasi.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        ResetPassword::createUrlUsing(function ($user, string $token) {
            return 'https://example.com/reset-password?token='.$token;
        });
    }

<a name="reset-email-customization"></a>
#### Kostumosasi Email Reset

Anda dapat dengan mudah memodifikasi kelas notifikasi yang digunakan untuk mengirim tautan reset kata sandi ke pengguna. Untuk memulai, tambahkan _method_ `sendPasswordResetNotification` pada mmodel`App\Models\User`. Dalam _method_ ini, Anda dapat mengirim notifikasi menggunakan [_class notification_](/docs/{{version}}/notifications) apapun yang Anda buat sendiri. Token reset kata sandi `$token` adalah argumen pertama yang diterima oleh _method_ tersebut. Anda dapat menggunakan `$token` ini untuk membangun URL reset kata sandi pilihan Anda dan mengirim notifikasi ke pengguna:

    use App\Notifications\ResetPasswordNotification;

    /**
     * Mengirim notifikasi reset kata sandi ke pengguna.
     *
     * @param  string  $token
     * @return void
     */
    public function sendPasswordResetNotification($token)
    {
        $url = 'https://example.com/reset-password?token='.$token;

        $this->notify(new ResetPasswordNotification($url));
    }
