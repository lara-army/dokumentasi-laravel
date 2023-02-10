# Laravel Passport

- [Introduction](#introduction)
    - [Passport Or Sanctum?](#passport-or-sanctum)
- [Installation](#installation)
    - [Deploying Passport](#deploying-passport)
    - [Migration Customization](#migration-customization)
    - [Upgrading Passport](#upgrading-passport)
- [Configuration](#configuration)
    - [Client Secret Hashing](#client-secret-hashing)
    - [Token Lifetimes](#token-lifetimes)
    - [Overriding Default Models](#overriding-default-models)
    - [Overriding Routes](#overriding-routes)
- [Issuing Access Tokens](#issuing-access-tokens)
    - [Managing Clients](#managing-clients)
    - [Requesting Tokens](#requesting-tokens)
    - [Refreshing Tokens](#refreshing-tokens)
    - [Revoking Tokens](#revoking-tokens)
    - [Purging Tokens](#purging-tokens)
- [Authorization Code Grant with PKCE](#code-grant-pkce)
    - [Creating The Client](#creating-a-auth-pkce-grant-client)
    - [Requesting Tokens](#requesting-auth-pkce-grant-tokens)
- [Password Grant Tokens](#password-grant-tokens)
    - [Creating A Password Grant Client](#creating-a-password-grant-client)
    - [Requesting Tokens](#requesting-password-grant-tokens)
    - [Requesting All Scopes](#requesting-all-scopes)
    - [Customizing The User Provider](#customizing-the-user-provider)
    - [Customizing The Username Field](#customizing-the-username-field)
    - [Customizing The Password Validation](#customizing-the-password-validation)
- [Implicit Grant Tokens](#implicit-grant-tokens)
- [Client Credentials Grant Tokens](#client-credentials-grant-tokens)
- [Personal Access Tokens](#personal-access-tokens)
    - [Creating A Personal Access Client](#creating-a-personal-access-client)
    - [Managing Personal Access Tokens](#managing-personal-access-tokens)
- [Protecting Routes](#protecting-routes)
    - [Via Middleware](#via-middleware)
    - [Passing The Access Token](#passing-the-access-token)
- [Token Scopes](#token-scopes)
    - [Defining Scopes](#defining-scopes)
    - [Default Scope](#default-scope)
    - [Assigning Scopes To Tokens](#assigning-scopes-to-tokens)
    - [Checking Scopes](#checking-scopes)
- [Consuming Your API With JavaScript](#consuming-your-api-with-javascript)
- [Events](#events)
- [Testing](#testing)

<a name="introduction"></a>
## perkenalan

Laravel Passport menyediakan implementasi server OAuth2 penuh untuk aplikasi Laravel Anda dalam beberapa menit. Passport dibangun di atas server League OAuth2 yang dikelola oleh Andy Millington dan Simon Hamp.

> **Warning**  
>Dokumentasi ini mengasumsikan bahwa Anda sudah familiar dengan OAuth2. Jika Anda tidak tahu apa-apa tentang OAuth2, pertimbangkan untuk membiasakan diri dengan [terminologi](https://oauth2.thephpleague.com/terminology/)dan fitur OAuth2 sebelum melanjutkan.


<a name="passport-or-sanctum"></a>
### Passport atau Sanctum?
Sebelum memulai, mungkin Anda ingin menentukan apakah aplikasi Anda akan lebih baik dilayani oleh Laravel Passport atau Laravel Sanctum. Jika aplikasi Anda harus mendukung OAuth2, maka Anda harus menggunakan Laravel Passport.

Namun, jika Anda mencoba untuk mengotentikasi aplikasi satu halaman, aplikasi seluler, atau mengeluarkan token API, Anda harus menggunakan Laravel Sanctum. Laravel Sanctum tidak mendukung OAuth2; Namun, ini menyediakan pengalaman pengembangan otentikasi API yang jauh lebih sederhana dan mudah.
<a name="installation"></a>
## Installation

untuk memulai install passport lewat composer package manager:

```shell
composer require laravel/passport
```

Passport [service provider](/docs/{{version}}/providers)  mendaftarkan direktori migrasi database sendir,  jadi anda harus migrasi setelah menginstall package. migrasi passport akan membuat tabel di aplikasi yang anda butuhkan untuk menyimpan OAuth2 klien dan access tokens:

```shell
php artisan migrate
```

Langkah selanjutnya, Anda harus menjalankan perintah Artisan passport:install. Perintah ini akan menciptakan kunci enkripsi yang dibutuhkan untuk mengenerate token akses yang aman. Selain itu, perintah ini akan membuat client "personal access" dan "password grant" yang akan digunakan untuk mengenerate token akses.

```shell
php artisan passport:install
```

> **Note**  
> Jika Anda ingin menggunakan UUID sebagai nilai kunci utama dari model Client Passport bukan angka auto-meningkat, silakan install Passport menggunakan opsi uuids.

setyellah menjalankan `passport:install` command, tambahkan `Laravel\Passport\HasApiTokens` trait ke `App\Models\User` model.Trait ini akan memberikan beberapa metode helper ke model Anda yang memungkinkan Anda untuk memeriksa token pengguna yang diotentikasi dan skop. Jika model Anda sudah menggunakan trait Laravel\Sanctum\HasApiTokens, maka Anda dapat menghapus trait tersebut dan menggunakan trait baru ini. Ini membantu Anda untuk membuat aplikasi yang lebih aman dan terkontrol dengan baik. Namun, pastikan untuk melakukan tes dan memeriksa apakah ada efek samping dari penghapusan trait sebelum melakukannya.


    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Factories\HasFactory;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Laravel\Passport\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, HasFactory, Notifiable;
    }

Akhirnya, pada file konfigurasi config/auth.php aplikasi Anda, Anda harus mendefinisikan guard autentikasi api dan mengatur opsi driver menjadi passport. Ini akan memberitahukan aplikasi Anda untuk menggunakan TokenGuard Passport saat mengotentikasi permintaan API yang masuk:

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'passport',
            'provider' => 'users',
        ],
    ],

<a name="client-uuids"></a>
#### Client UUIDs

Anda juga dapat menjalankan perintah passport:install dengan opsi --uuids yang ada. Opsi ini akan memberitahukan Passport bahwa Anda ingin menggunakan UUID sebagai nilai kunci utama model Client Passport bukan angka auto-incrementing. Setelah menjalankan perintah passport:install dengan opsi --uuids, Anda akan diberikan instruksi tambahan tentang menonaktifkan migrasi default Passport:

```shell
php artisan passport:install --uuids
```

<a name="deploying-passport"></a>
### Deploying Passport

Saat menyebarkan Passport untuk server aplikasi Anda untuk pertama kalinya, Anda kemungkinan perlu menjalankan perintah passport:keys. Perintah ini akan menghasilkan kunci enkripsi yang dibutuhkan Passport untuk menghasilkan token akses. Kunci yang dihasilkan biasanya tidak dikelola pada kontrol sumber:

```shell
php artisan passport:keys
```

Jika diperlukan, Anda dapat mendefinisikan lokasi dimana kunci Passport harus dimuat dari. Anda dapat menggunakan metode Passport::loadKeysFrom untuk menyelesaikannya. Biasanya, metode ini harus dipanggil dari metode boot kelas App\Providers\AuthServiceProvider aplikasi Anda:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::loadKeysFrom(__DIR__.'/../secrets/oauth');
    }

<a name="loading-keys-from-the-environment"></a>
#### Loading Keys From The Environment

Atau, Anda juga dapat mempublikasikan berkas konfigurasi Passport menggunakan perintah Artisan vendor:publish:

```shell
php artisan vendor:publish --tag=passport-config
```

Setelah berkas konfigurasi telah dipublikasikan, Anda dapat memuat kunci enkripsi aplikasi Anda dengan menentukannya sebagai variabel environment:


```ini
PASSPORT_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
<private key here>
-----END RSA PRIVATE KEY-----"

PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----
<public key here>
-----END PUBLIC KEY-----"
```

<a name="migration-customization"></a>
### kostumisasi migrasi

Jika Anda tidak akan menggunakan migrasi default Passport, Anda harus memanggil metode Passport::ignoreMigrations di metode register kelas App\Providers\AppServiceProvider Anda. Anda dapat mengekspor migrasi default menggunakan perintah Artisan vendor:publish:


```shell
php artisan vendor:publish --tag=passport-migrations
```

<a name="upgrading-passport"></a>
### Upgrading Passport

Penting untuk memperhatikan panduan upgrade ketika melakukan upgrade ke versi  baru dari Passport. Pastikan untuk membaca dengan hati-hati panduan upgrade (https://github.com/laravel/passport/blob/master/UPGRADE.md) untuk memastikan bahwa proses upgrade berjalan dengan lancar.

<a name="configuration"></a>
## Configuration

<a name="client-secret-hashing"></a>
### Client Secret Hashing
Jika Anda ingin rahasia klien Anda dienkripsi ketika disimpan di database, Anda harus memanggil metode Passport::hashClientSecrets pada method boot dari kelas App\Providers\AuthServiceProvider:

    use Laravel\Passport\Passport;

    Passport::hashClientSecrets();

Setelah diaktifkan, semua rahasia klien hanya akan dapat ditampilkan pada pengguna secara langsung setelah mereka dibuat. Karena nilai rahasia klien dalam teks biasa tidak pernah disimpan dalam database, tidak mungkin untuk memulihkan nilai rahasia jika hilang.

<a name="token-lifetimes"></a>
### 
 Secara default, Passport mengeluarkan token akses yang masa berlakunya panjang yang kedaluwarsa setelah satu tahun. Jika Anda ingin mengkonfigurasi masa berlakunya token yang lebih lama / pendek, Anda dapat menggunakan metode tokensExpireIn, refreshTokensExpireIn, dan personalAccessTokensExpireIn. Metode ini harus dipanggil dari metode boot dari kelas App\Providers\AuthServiceProvider dalam aplikasi Anda:
 
    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::tokensExpireIn(now()->addDays(15));
        Passport::refreshTokensExpireIn(now()->addDays(30));
        Passport::personalAccessTokensExpireIn(now()->addMonths(6));
    }

> **peringatan**  
> Kolom expires_at pada tabel database Passport hanya bisa dibaca dan hanya untuk tujuan tampilan. Saat mengeluarkan token, Passport menyimpan informasi masa berlaku di dalam token yang ditandatangani dan dienkripsi. Jika Anda perlu membatalkan sebuah token, Anda harus  [revoke it](#revoking-tokens)..

<a name="overriding-default-models"></a>
### Overriding Default Models

Anda bebas untuk memperluas model yang digunakan secara internal oleh Passport dengan mendefinisikan model Anda sendiri dan memperluas model Passport yang sesuai:

    use Laravel\Passport\Client as PassportClient;

    class Client extends PassportClient
    {
        // ...
    }

Setelah mendefinisikan modelmu, kamu dapat memberitahu Passport untuk menggunakan model kustommu melalui kelas Laravel\Passport\Passport. Biasanya, kamu harus memberi tahu Passport tentang model kustommu di method boot dari kelas App\Providers\AuthServiceProvider aplikasimu.

Setelah mendefinisikan modelmu, kamu dapat memberitahu Passport untuk menggunakan model kustommu melalui kelas Laravel\Passport\Passport. Biasanya, kamu harus memberi tahu Passport tentang model kustommu di method boot dari kelas App\Providers\AuthServiceProvider aplikasimu.

    use App\Models\Passport\AuthCode;
    use App\Models\Passport\Client;
    use App\Models\Passport\PersonalAccessClient;
    use App\Models\Passport\RefreshToken;
    use App\Models\Passport\Token;

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::useTokenModel(Token::class);
        Passport::useRefreshTokenModel(RefreshToken::class);
        Passport::useAuthCodeModel(AuthCode::class);
        Passport::useClientModel(Client::class);
        Passport::usePersonalAccessClientModel(PersonalAccessClient::class);
    }

<a name="overriding-routes"></a>
### Overriding Routes

Kadang", mungkin kamu  ingin menyesuaikan rute yang didefinisikan oleh Passport. Untuk mencapainya, pertama kamu perlu mengabaikan rute yang didaftarkan oleh Passport dengan menambahkan Passport::ignoreRoutes pada metode register dari AppServiceProvider aplikasi kamu:

    use Laravel\Passport\Passport;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        Passport::ignoreRoutes();
    }

Kemudian, Anda dapat menyalin rute yang didefinisikan oleh Passport di [file routenya](https://github.com/laravel/passport/blob/11.x/routes/web.php) aplikasi Anda dan memodifikasinya sesuai keinginan Anda:

    Route::group([
        'as' => 'passport.',
        'prefix' => config('passport.path', 'oauth'),
        'namespace' => 'Laravel\Passport\Http\Controllers',
    ], function () {
        // Passport routes...
    });

<a name="issuing-access-tokens"></a>
## Issuing Access Tokens

OAuth2 dengan menggunakan authorization codes adalah cara yang paling dikenal oleh para developer dalam menggunakan OAuth2. Saat menggunakan authorization codes, aplikasi client akan mengarahkan user ke server Anda, di mana mereka akan menyetujui atau menolak permintaan untuk mengeluarkan access token ke client.

<a name="managing-clients"></a>
### Mengelola Klien

Pertama, pengembang yang membangun aplikasi yang perlu berinteraksi dengan API aplikasi Anda harus mendaftarkan aplikasi mereka dengan Anda dengan membuat "klien". Biasanya, ini berisi memberikan nama aplikasi mereka dan URL dimana aplikasi Anda dapat mengarahkan setelah pengguna menyetujui permintaan otorisasi mereka.

<a name="the-passportclient-command"></a>
#### The `passport:client` Command

Cara termudah untuk membuat client adalah menggunakan command Artisan passport:client. Perintah ini dapat digunakan untuk membuat client sendiri untuk menguji fungsi OAuth2 Anda. Ketika Anda menjalankan perintah client, Passport akan meminta Anda untuk memberikan informasi lebih lanjut tentang client Anda dan akan memberikan Anda ID client dan rahasia.


```shell
php artisan passport:client
```

**Redirect URLs**

Jika Anda ingin memperbolehkan beberapa URL redirect untuk klien Anda, Anda dapat menentukannya menggunakan daftar yang dipisahkan dengan koma saat diminta URL oleh perintah passport:client. URL apa pun yang berisi koma harus dienkode URL:

```shell
http://example.com/callback,http://examplefoo.com/callback
```

<a name="clients-json-api"></a>
#### JSON API

Anda juga dapat menggunakan API JSON yang disediakan oleh Passport untuk membuat, memperbarui, dan menghapus client. Ini akan menghemat waktu Anda dari perlu menulis kontroller secara manual untuk hal-hal tersebut.

Namun, Anda harus menggabungkan API JSON Passport dengan frontend Anda sendiri untuk menyediakan dashboard bagi pengguna untuk mengelola klien mereka. Berikut, kami akan meninjau semua titik akhir API untuk mengelola klien. Untuk kenyamanan, kami akan menggunakan [Axios] (https://github.com/axios/axios) untuk menunjukkan pembuatan permintaan HTTP ke titik akhir.

API JSON dilindungi oleh middleware web dan auth; oleh karena itu, hanya bisa dipanggil dari aplikasi Anda sendiri. Tidak bisa dipanggil dari sumber eksternal.

<a name="get-oauthclients"></a>
#### `GET /oauth/clients`

Rute ini mengembalikan semua klien untuk pengguna yang terautentikasi. Ini terutama berguna untuk mencantumkan semua klien pengguna sehingga mereka dapat mengedit atau menghapusnya:

```js
axios.get('/oauth/clients')
    .then(response => {
        console.log(response.data);
    });
```

<a name="post-oauthclients"></a>
#### `POST /oauth/clients`

Rute ini digunakan untuk membuat klien baru. Ini membutuhkan dua buah data: nama klien dan URL redirect. URL redirect adalah tempat pengguna akan dialihkan setelah menyetujui atau menolak permintaan otorisasi.

Ketika klien dibuat, akan diterbitkan ID klien dan rahasia klien. Nilai-nilai ini akan digunakan ketika meminta token akses dari aplikasi Anda. Rute pembuatan klien akan mengembalikan instance klien baru:

```js
const data = {
    name: 'Client Name',
    redirect: 'http://example.com/callback'
};

axios.post('/oauth/clients', data)
    .then(response => {
        console.log(response.data);
    })
    .catch (response => {
        // List errors on response...
    });
```

<a name="put-oauthclientsclient-id"></a>
#### `PUT /oauth/clients/{client-id}`

Rute ini digunakan untuk memperbarui klien. Ini membutuhkan dua buah data: nama klien dan URL redirect. URL redirect adalah tempat pengguna akan dialihkan setelah menyetujui atau menolak permintaan otorisasi. Rute ini akan mengembalikan instance klien yang diperbarui:

```js
const data = {
    name: 'New Client Name',
    redirect: 'http://example.com/callback'
};

axios.put('/oauth/clients/' + clientId, data)
    .then(response => {
        console.log(response.data);
    })
    .catch (response => {
        // List errors on response...
    });
```

<a name="delete-oauthclientsclient-id"></a>
#### `DELETE /oauth/clients/{client-id}`

rute ini digunakan untuk delete klien:
```js
axios.delete('/oauth/clients/' + clientId)
    .then(response => {
        //
    });
```

<a name="requesting-tokens"></a>
### Requesting Tokens

<a name="requesting-tokens-redirecting-for-authorization"></a>
#### Redirecting For Authorization

Setelah klien dibuat, pengembang dapat menggunakan ID klien dan rahasia mereka untuk meminta kode otorisasi dan token akses dari aplikasi Anda. Pertama, aplikasi yang mengonsumsi harus membuat permintaan redirect ke rute /oauth/authorize aplikasi Anda seperti ini:

    use Illuminate\Http\Request;
    use Illuminate\Support\Str;

    Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'response_type' => 'code',
            'scope' => '',
            'state' => $state,
            // 'prompt' => '', // "none", "consent", or "login"
        ]);

        return redirect('http://passport-app.test/oauth/authorize?'.$query);
    });

Parameter prompt dapat digunakan untuk menentukan perilaku autentikasi dari aplikasi Passport.

Jika nilai prompt adalah none, Passport akan selalu mengirimkan kesalahan autentikasi jika pengguna belum terautentikasi dengan aplikasi Passport. Jika nilainya consent, Passport akan selalu menampilkan layar persetujuan otorisasi, meskipun semua scope sudah diberikan sebelumnya ke aplikasi yang mengonsumsi. Saat nilainya login, aplikasi Passport akan selalu meminta pengguna untuk masuk kembali ke aplikasi, meskipun mereka sudah memiliki sesi yang ada.

Jika tidak ada nilai prompt yang diberikan, pengguna akan diminta untuk menyetujui otorisasi hanya jika mereka belum menyetujui akses ke aplikasi yang mengonsumsi untuk scope yang diminta.

> **Note**  
>Ingatlah, rute /oauth/authorize sudah didefinisikan oleh Passport. Anda tidak perlu menentukan rute ini secara manual.

<a name="approving-the-request"></a>
#### Approving The Request

Ketika menerima permintaan otorisasi, Passport akan secara otomatis menanggapi berdasarkan nilai parameter prompt (jika ada) dan mungkin akan menampilkan template ke pengguna yang memungkinkan mereka untuk menyetujui atau menolak permintaan otorisasi. Jika mereka menyetujui permintaan, mereka akan dialihkan kembali ke redirect_uri yang ditentukan oleh aplikasi yang mengonsumsi. Redirect_uri harus cocok dengan URL redirect yang ditentukan saat client dibuat.

Jika Anda ingin menyesuaikan layar persetujuan otorisasi, Anda dapat mempublikasikan tampilan Passport menggunakan perintah Artisan vendor:publish. Tampilan yang diterbitkan akan ditempatkan pada direktori resources/views/vendor/passport:


```shell
php artisan vendor:publish --tag=passport-views
```

Terkadang Anda mungkin ingin melewati prompt otorisasi, seperti ketika mengotorisasi client pihak pertama. Anda dapat menyelesaikan hal ini dengan memperluas model Client dan menentukan metode skipsAuthorization. Jika skipsAuthorization mengembalikan true, client akan disetujui dan pengguna akan dialihkan kembali ke redirect_uri segera, kecuali jika aplikasi yang mengonsumsi secara eksplisit menentukan parameter prompt saat dialihkan untuk otorisasi:

    <?php

    namespace App\Models\Passport;

    use Laravel\Passport\Client as BaseClient;

    class Client extends BaseClient
    {
        /**
         * Determine if the client should skip the authorization prompt.
         *
         * @return bool
         */
        public function skipsAuthorization()
        {
            return $this->firstParty();
        }
    }

<a name="requesting-tokens-converting-authorization-codes-to-access-tokens"></a>
#### Converting Authorization Codes To Access Tokens

Jika pengguna menyetujui permintaan otorisasi, mereka akan dialihkan kembali ke aplikasi yang mengonsumsi. Konsumen harus memverifikasi parameter state terhadap nilai yang disimpan sebelum dialihkan. Jika parameter state cocok, konsumen harus mengeluarkan permintaan POST ke aplikasi Anda untuk meminta token akses. Permintaan harus mencakup kode otorisasi yang diterbitkan oleh aplikasi Anda ketika pengguna menyetujui permintaan otorisasi:


    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Http;

    Route::get('/callback', function (Request $request) {
        $state = $request->session()->pull('state');

        throw_unless(
            strlen($state) > 0 && $state === $request->state,
            InvalidArgumentException::class
        );

        $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
            'grant_type' => 'authorization_code',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'code' => $request->code,
        ]);

        return $response->json();
    });

Rute `/oauth/token` ini akan mengembalikan respons JSON yang berisi atribut `access_token`, `refresh_token`, dan `expires_in`. Atribut expires_in berisi jumlah detik sampai token akses kedaluwarsa.




> **Note**  
Seperti rute `/oauth/authorize`, rute `/oauth/token` didefinisikan untuk Anda oleh Passport. Tidak perlu secara manual menentukan rute ini.
<a name="tokens-json-api"></a>
#### JSON API

Passport juga menyertakan API JSON untuk mengelola token akses yang diotorisasi. Anda dapat menggabungkannya dengan frontend Anda sendiri untuk menawarkan dashboard pengelolaan token akses bagi pengguna Anda. Untuk kenyamanan, kami akan menggunakan [Axios](https://github.com/mzabriskie/axios)untuk menunjukkan membuat permintaan HTTP ke titik akhir. API JSON dilindungi oleh middleware web dan auth; oleh karena itu, hanya dapat dipanggil dari aplikasi Anda sendiri.




<a name="get-oauthtokens"></a>
#### `GET /oauth/tokens`

Rute ini mengembalikan semua token akses yang diautorisasi yang dibuat oleh pengguna yang diautentikasi. Ini terutama berguna untuk menampilkan semua token pengguna sehingga mereka bisa membatalkan mereka.


```js
axios.get('/oauth/tokens')
    .then(response => {
        console.log(response.data);
    });
```

<a name="delete-oauthtokenstoken-id"></a>
#### `DELETE /oauth/tokens/{token-id}`

Rute ini bisa digunakan untuk membatalkan token akses yang diautorisasi dan refresh token yang terkait dengannya.

```js
axios.delete('/oauth/tokens/' + tokenId);
```

<a name="refreshing-tokens"></a>
### Refreshing Tokens

Jika aplikasi Anda mengeluarkan token akses berlangganan pendek, pengguna akan perlu memperbarui token akses mereka melalui refresh token yang diberikan kepada mereka saat token akses dikeluarkan:

    use Illuminate\Support\Facades\Http;

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'refresh_token',
        'refresh_token' => 'the-refresh-token',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'scope' => '',
    ]);

    return $response->json();

Rute `/oauth/token` ini akan mengembalikan respons JSON yang berisi atribut `access_token`, `refresh_token`, dan `expires_in`. Atribut expires_in berisi jumlah detik sampai token akses kedaluwarsa.




<a name="revoking-tokens"></a>
### Revoking Tokens

Anda dapat membatalkan token dengan menggunakan metode `revokeAccessToken` pada `Laravel\Passport\TokenRepository`. Anda dapat membatalkan refresh token token menggunakan metode `revokeRefreshTokensByAccessTokenId` pada `Laravel\Passport\RefreshTokenRepository`. Kelas-kelas ini dapat diselesaikan menggunakan container layanan Laravel:

    use Laravel\Passport\TokenRepository;
    use Laravel\Passport\RefreshTokenRepository;

    $tokenRepository = app(TokenRepository::class);
    $refreshTokenRepository = app(RefreshTokenRepository::class);

    // Revoke an access token...
    $tokenRepository->revokeAccessToken($tokenId);

    // Revoke all of the token's refresh tokens...
    $refreshTokenRepository->revokeRefreshTokensByAccessTokenId($tokenId);

<a name="purging-tokens"></a>
### Purging Tokens

Ketika token sudah dibatalkan atau kedaluwarsa, Anda mungkin ingin membersihkan mereka dari database. Perintah Artisan passport:purge yang disertakan dalam Passport dapat melakukan ini untuk Anda:


```shell
# Purge revoked and expired tokens and auth codes...
php artisan passport:purge

# Only purge revoked tokens and auth codes...
php artisan passport:purge --revoked

# Only purge expired tokens and auth codes...
php artisan passport:purge --expired
```

Anda juga dapat mengkonfigurasi pekerjaan yang dijadwalkan dalam kelas `App\Console\Kernel` aplikasi Anda untuk secara otomatis membersihkan token Anda sesuai jadwal:


    /**
     * Define the application's command schedule.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->command('passport:purge')->hourly();
    }

<a name="code-grant-pkce"></a>
## Authorization Code Grant with PKCE

"Grant Kode Otorisasi dengan "Bukti Kunci untuk Tukar Kode" (PKCE) adalah cara yang aman untuk mengotentikasi aplikasi halaman tunggal atau aplikasi native untuk mengakses API Anda. Grant ini harus digunakan ketika Anda tidak dapat menjamin bahwa rahasia klien akan disimpan secara rahasia atau untuk mengurangi ancaman adanya kode otorisasi yang dicuri oleh peretas. Kombinasi "verifikasi kode" dan "tantangan kode" menggantikan rahasia klien saat menukar kode otorisasi untuk token akses."

<a name="creating-a-auth-pkce-grant-client"></a>
### buat klien

Sebelum aplikasi Anda bisa mengeluarkan token melalui grant kode otorisasi dengan PKCE, Anda harus membuat klien yang didukung PKCE. Anda dapat melakukan ini menggunakan perintah Artisan `passport:client` dengan opsi `--public`:

```shell
php artisan passport:client --public
```

<a name="requesting-auth-pkce-grant-tokens"></a>
### Requesting Tokens

<a name="code-verifier-code-challenge"></a>
#### Code Verifier & Code Challenge

Karena grant otorisasi ini tidak menyediakan rahasia klien, para pengembang harus mengenerate gabungan dari verifikasi kode dan tantangan kode agar dapat meminta token.

Verifikasi kode harus berupa string acak antara 43 dan 128 karakter yang mengandung huruf, angka, dan karakter "-", ".", "_", "~", seperti yang didefinisikan RFC 7636 dalam spesifikasi (https://tools.ietf.org/html/rfc7636).

Tantangan kode harus berupa string yang dienkode dengan Base64 dengan karakter yang aman untuk URL dan nama file. Karakter `'='` pada akhir harus dihapus dan tidak ada baris baru, spasi, atau karakter lain yang harus ada.

    $encoded = base64_encode(hash('sha256', $code_verifier, true));

    $codeChallenge = strtr(rtrim($encoded, '='), '+/', '-_');

<a name="code-grant-pkce-redirecting-for-authorization"></a>
#### Redirecting For Authorization

Setelah client dibuat, Anda dapat menggunakan ID client dan verifikasi kode dan tantangan kode yang diterbitkan untuk meminta kode otorisasi dan token akses dari aplikasi Anda. Pertama, aplikasi yang memakai harus membuat permintaan redirect ke rute `/oauth/authorize` dari aplikasi Anda:

    use Illuminate\Http\Request;
    use Illuminate\Support\Str;

    Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $request->session()->put(
            'code_verifier', $code_verifier = Str::random(128)
        );

        $codeChallenge = strtr(rtrim(
            base64_encode(hash('sha256', $code_verifier, true))
        , '='), '+/', '-_');

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'response_type' => 'code',
            'scope' => '',
            'state' => $state,
            'code_challenge' => $codeChallenge,
            'code_challenge_method' => 'S256',
            // 'prompt' => '', // "none", "consent", or "login"
        ]);

        return redirect('http://passport-app.test/oauth/authorize?'.$query);
    });

<a name="code-grant-pkce-converting-authorization-codes-to-access-tokens"></a>
#### Converting Authorization Codes To Access Tokens

Jika pengguna menyetujui permintaan otorisasi, mereka akan dialihkan kembali ke aplikasi yang memakai. Konsumen harus memverifikasi parameter `state` terhadap nilai yang disimpan sebelum redirect, seperti pada standar Authorization Code Grant.

Jika parameter state cocok, konsumen harus mengeluarkan permintaan `POST` ke aplikasi Anda untuk meminta token akses. Permintaan harus mencakup kode otorisasi yang diterbitkan oleh aplikasi Anda saat pengguna menyetujui permintaan otorisasi bersama dengan code verifier yang awalnya dibuat:

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Http;

    Route::get('/callback', function (Request $request) {
        $state = $request->session()->pull('state');

        $codeVerifier = $request->session()->pull('code_verifier');

        throw_unless(
            strlen($state) > 0 && $state === $request->state,
            InvalidArgumentException::class
        );

        $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
            'grant_type' => 'authorization_code',
            'client_id' => 'client-id',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'code_verifier' => $codeVerifier,
            'code' => $request->code,
        ]);

        return $response->json();
    });

<a name="password-grant-tokens"></a>
## Password Grant Tokens

> **Warning**  
> We no longer recommend using password grant tokens. Instead, you should choose [a grant type that is currently recommended by OAuth2 Server](https://oauth2.thephpleague.com/authorization-server/which-grant/).

Grant OAuth2 password memungkinkan client aplikasi pertama seperti aplikasi mobile untuk mendapatkan access token menggunakan alamat email / username dan password. Ini memungkinkan Anda untuk mengeluarkan access token dengan aman ke client aplikasi pertama tanpa memerlukan pengguna untuk melalui aliran redirect authorization code OAuth2 secara keseluruhan.

<a name="creating-a-password-grant-client"></a>
### Creating A Password Grant Client

Sebelum aplikasi Anda dapat mengeluarkan token melalui grant password, Anda perlu membuat client grant password. Anda bisa melakukan ini menggunakan perintah Artisan `passport:client` dengan opsi `--password`. Jika Anda sudah menjalankan perintah `passport:install`, Anda tidak perlu menjalankan perintah ini lagi:


```shell
php artisan passport:client --password
```

<a name="requesting-password-grant-tokens"></a>
### Requesting Tokens

Setelah Anda membuat client grant password, Anda dapat meminta token akses dengan mengeluarkan permintaan `POST` ke rute `/oauth/token`dengan alamat email dan password pengguna. Ingatlah, rute ini sudah terdaftar oleh Passport sehingga tidak perlu didefinisikan secara manual. Jika permintaan berhasil, Anda akan menerima `access_token` dan `refresh_token` dalam respons JSON dari server.

    use Illuminate\Support\Facades\Http;

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'password',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'username' => 'taylor@laravel.com',
        'password' => 'my-password',
        'scope' => '',
    ]);

    return $response->json();

> **Note**  
> Remember, access tokens are long-lived by default. However, you are free to [configure your maximum access token lifetime](#configuration) if needed.

<a name="requesting-all-scopes"></a>
### Requesting All Scopes

Ketika menggunakan grant password atau grant credentials client, mungkin Anda ingin memberikan otorisasi token untuk semua scope yang didukung oleh aplikasi Anda. Anda dapat melakukan ini dengan meminta scope `*.` Jika Anda meminta scope `*`, metode `can` pada instance token akan selalu mengembalikan true. Scope ini hanya dapat diterapkan pada token yang diterbitkan menggunakan grant `password` atau `client_credentials`.

    use Illuminate\Support\Facades\Http;

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'password',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'username' => 'taylor@laravel.com',
        'password' => 'my-password',
        'scope' => '*',
    ]);

<a name="customizing-the-user-provider"></a>
### Customizing The User Provider

Untuk aplikasi yang menggunakan lebih dari satu penyedia pengguna otentikasi, Anda dapat menentukan penyedia pengguna yang digunakan oleh klien password grant dengan memberikan opsi `--provider`saat membuat klien melalui perintah `artisan passport:client --password`. Nama penyedia yang diberikan harus cocok dengan penyedia yang valid yang didefinisikan dalam file konfigurasi config/auth.php aplikasi Anda. Kemudian Anda dapat melindungi rute Anda menggunakan middleware untuk memastikan bahwa hanya pengguna dari guard penyedia tertentu yang di


<a name="customizing-the-username-field"></a>
### Customizing The Username Field

Ketika melakukan autentikasi menggunakan password grant, Passport akan menggunakan atribut `email` dari model authenticatable Anda sebagai "nama pengguna". Namun, Anda dapat menyesuaikan perilaku ini dengan mendefinisikan metode findForPassport pada model Anda.



    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Laravel\Passport\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;

        /**
         * Find the user instance for the given username.
         *
         * @param  string  $username
         * @return \App\Models\User
         */
        public function findForPassport($username)
        {
            return $this->where('username', $username)->first();
        }
    }

<a name="customizing-the-password-validation"></a>
### Customizing The Password Validation

Saat melakukan otentikasi menggunakan grant password, Passport akan menggunakan atribut `password` pada model Anda untuk memvalidasi password yang diberikan. Jika model Anda tidak memiliki atribut `password` atau ingin menyesuaikan logika validasi password, Anda dapat menentukan metode `validateForPassportPasswordGrant` pada model Anda.




    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Support\Facades\Hash;
    use Laravel\Passport\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;

        /**
         * Validate the password of the user for the Passport password grant.
         *
         * @param  string  $password
         * @return bool
         */
        public function validateForPassportPasswordGrant($password)
        {
            return Hash::check($password, $this->password);
        }
    }

<a name="implicit-grant-tokens"></a>
## Implicit Grant Tokens

> **Warning**  
> We no longer recommend using implicit grant tokens. Instead, you should choose [a grant type that is currently recommended by OAuth2 Server](https://oauth2.thephpleague.com/authorization-server/which-grant/).

Grant implicit adalah mirip dengan grant kode otorisasi, tetapi token dikembalikan langsung ke client tanpa menukar kode otorisasi. Jenis grant ini biasanya digunakan untuk aplikasi JavaScript atau mobile di mana tidak aman untuk menyimpan kredensial client. Untuk mengaktifkan grant ini, Anda perlu memanggil metode enableImplicitGrant pada method boot dari kelas App\Providers\AuthServiceProvider aplikasi Anda.



    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::enableImplicitGrant();
    }

Setelah grant diaktifkan, pengembang dapat menggunakan ID client mereka untuk meminta token akses dari aplikasi Anda. Aplikasi yang menggunakan harus membuat permintaan redirect ke rute `/oauth/authorize` aplikasi Anda seperti berikut:




    use Illuminate\Http\Request;

    Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'response_type' => 'token',
            'scope' => '',
            'state' => $state,
            // 'prompt' => '', // "none", "consent", or "login"
        ]);

        return redirect('http://passport-app.test/oauth/authorize?'.$query);
    });

> **Note**  
> Remember, the `/oauth/authorize` route is already defined by Passport. You do not need to manually define this route.

<a name="client-credentials-grant-tokens"></a>
## Client Credentials Grant Tokens

Grant client credentials cocok digunakan untuk autentikasi mesin-ke-mesin. Sebagai contoh, Anda mungkin menggunakan grant ini pada tugas pemeliharaan yang dijalankan melalui API.

Sebelum aplikasi Anda bisa mengeluarkan token melalui grant client credentials, Anda harus membuat sebuah client credentials grant client. Anda bisa melakukan ini menggunakan opsi `--client `pada perintah `Artisan passport:client`:

```shell
php artisan passport:client --client
```

Setelah membuat client grant client credentials, selanjutnya Anda perlu menambahkan middleware `CheckClientCredentials` ke properti `$routeMiddleware` pada file `app/Http/Kernel.php` untuk menggunakan jenis grant ini:

    use Laravel\Passport\Http\Middleware\CheckClientCredentials;

    protected $routeMiddleware = [
        'client' => CheckClientCredentials::class,
    ];

Kemudian, lampirkan middleware pada sebuah rute:
    Route::get('/orders', function (Request $request) {
        ...
    })->middleware('client');

Untuk membatasi akses ke rute hanya untuk skop tertentu, Anda dapat memberikan daftar skop yang dibutuhkan yang dipisahkan oleh koma saat melampirkan middleware `client` ke rute:

    Route::get('/orders', function (Request $request) {
        ...
    })->middleware('client:check-status,your-scope');

<a name="retrieving-tokens"></a>
### Retrieving Tokens

Untuk mendapatkan token menggunakan jenis grant ini, lakukan permintaan ke titik akhir `oauth/token`:

    use Illuminate\Support\Facades\Http;

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'client_credentials',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'scope' => 'your-scope',
    ]);

    return $response->json()['access_token'];

<a name="personal-access-tokens"></a>
## Personal Access Tokens

Kadang-kadang, pengguna Anda mungkin ingin memberikan token akses ke diri mereka sendiri tanpa melalui aliran pengalihan kode otorisasi biasa. Memperbolehkan pengguna untuk memberikan token ke diri mereka sendiri melalui UI aplikasi Anda bisa berguna untuk memungkinkan pengguna untuk bereksperimen dengan API Anda atau bisa menjadi pendekatan yang lebih sederhana untuk memberikan token akses secara umum.

> **Note**  
> If your application is primarily using Passport to issue personal access tokens, consider using [Laravel Sanctum](/docs/{{version}}/sanctum), Laravel's light-weight first-party library for issuing API access tokens.

<a name="creating-a-personal-access-client"></a>
### Creating A Personal Access Client

Sebelum aplikasi Anda dapat mengeluarkan token akses pribadi, Anda harus membuat sebuah klien akses pribadi. Anda dapat melakukan ini dengan menjalankan perintah `Artisan passport:client` dengan opsi `--personal`. Jika Anda sudah menjalankan perintah `passport:install`, Anda tidak perlu menjalankan perintah ini.




```shell
php artisan passport:client --personal
```

Setelah membuat klien akses pribadi Anda, tempatkan ID klien dan nilai rahasia teks biasa di aplikasi Anda`.env` file:

```ini
PASSPORT_PERSONAL_ACCESS_CLIENT_ID="client-id-value"
PASSPORT_PERSONAL_ACCESS_CLIENT_SECRET="unhashed-client-secret-value"
```

<a name="managing-personal-access-tokens"></a>
### Managing Personal Access Tokens

Setelah Anda membuat klien akses pribadi, Anda dapat mengeluarkan token untuk pengguna tertentu menggunakan`createToken` metode di `App\Models\User` model instance. Metode `createToken` menerima nama token sebagai argumen pertama dan array opsional [scopes](#token-scopes) sebagai argumen kedua:

    use App\Models\User;

    $user = User::find(1);

    // Creating a token without scopes...
    $token = $user->createToken('Token Name')->accessToken;

    // Creating a token with scopes...
    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="personal-access-tokens-json-api"></a>
#### JSON API

Paspor juga menyertakan API JSON untuk mengelola token akses pribadi. Anda dapat memasangkan ini dengan frontend Anda sendiri untuk menawarkan kepada pengguna Anda sebuah dasbor untuk mengelola token akses pribadi. Di bawah, kami akan meninjau semua titik akhir API untuk mengelola token akses pribadi. Demi kenyamanan, kami akan menggunakan [Axios](https://github.com/mzabriskie/axios) untuk mendemonstrasikan pembuatan permintaan HTTP ke endpoint.

API JSON dijaga oleh middleware `web` dan `auth`; oleh karena itu, itu hanya dapat dipanggil dari aplikasi Anda sendiri. Itu tidak dapat dipanggil dari sumber eksternal.

<a name="get-oauthscopes"></a>
#### `GET /oauth/scopes`

Rute ini menampilkan semua [scopes](#token-scopes) yang ditentukan untuk aplikasi Anda. Anda dapat menggunakan rute ini untuk mencantumkan cakupan yang dapat ditetapkan pengguna ke token akses pribadi:

```js
axios.get('/oauth/scopes')
    .then(response => {
        console.log(response.data);
    });
```

<a name="get-oauthpersonal-access-tokens"></a>
#### `GET /oauth/personal-access-tokens`

Rute ini mengembalikan semua token akses pribadi yang telah dibuat oleh pengguna yang diautentikasi. Ini terutama berguna untuk mencantumkan semua token pengguna sehingga mereka dapat mengedit atau mencabutnya:

```js
axios.get('/oauth/personal-access-tokens')
    .then(response => {
        console.log(response.data);
    });
```

<a name="post-oauthpersonal-access-tokens"></a>
#### `POST /oauth/personal-access-tokens`

Rute ini membuat token akses pribadi baru. Ini memerlukan dua bagian data: `nama` token dan `cakupan` yang harus ditetapkan ke token:

```js
const data = {
    name: 'Token Name',
    scopes: []
};

axios.post('/oauth/personal-access-tokens', data)
    .then(response => {
        console.log(response.data.accessToken);
    })
    .catch (response => {
        // List errors on response...
    });
```

<a name="delete-oauthpersonal-access-tokenstoken-id"></a>
#### `DELETE /oauth/personal-access-tokens/{token-id}`

Rute ini dapat digunakan untuk mencabut token akses pribadi:

```js
axios.delete('/oauth/personal-access-tokens/' + tokenId);
```

<a name="protecting-routes"></a>
## Protecting Routes

<a name="via-middleware"></a>
### Via Middleware

Paspor mencakup [pelindung autentikasi](/docs/{{version}}/authentication#adding-custom-guards) yang akan memvalidasi token akses pada permintaan masuk. Setelah Anda mengonfigurasi penjaga `api` untuk menggunakan driver `passport`, Anda hanya perlu menentukan middleware `auth:api` pada rute mana pun yang memerlukan token akses yang valid:
    Route::get('/user', function () {
        //
    })->middleware('auth:api');

> **Warning**  
> If you are using the [client credentials grant](#client-credentials-grant-tokens), you should use [the `client` middleware](#client-credentials-grant-tokens) to protect your routes instead of the `auth:api` middleware.

<a name="multiple-authentication-guards"></a>
#### Multiple Authentication Guards

Jika aplikasi Anda mengautentikasi jenis pengguna yang berbeda yang mungkin menggunakan model Eloquent yang sama sekali berbeda, Anda mungkin perlu menentukan konfigurasi penjaga untuk setiap jenis penyedia pengguna dalam aplikasi Anda. Ini memungkinkan Anda melindungi permintaan yang ditujukan untuk penyedia pengguna tertentu. Misalnya, diberikan konfigurasi penjaga berikut file konfigurasi `config/auth.php`:

    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],

    'api-customers' => [
        'driver' => 'passport',
        'provider' => 'customers',
    ],

Rute berikut akan menggunakan penjaga `api-customers`, yang menggunakan penyedia pengguna `customers`, untuk mengautentikasi permintaan yang masuk:

    Route::get('/customer', function () {
        //
    })->middleware('auth:api-customers');

> **Note**  
> For more information on using multiple user providers with Passport, please consult the [password grant documentation](#customizing-the-user-provider).

<a name="passing-the-access-token"></a>
### Passing The Access Token

Saat memanggil rute yang dilindungi oleh Passport, konsumen API aplikasi Anda harus menetapkan token akses mereka sebagai token `Bearer` di header `Authorization` permintaan mereka. Misalnya, saat menggunakan pustaka HTTP Guzzle:
    
    use Illuminate\Support\Facades\Http;

    $response = Http::withHeaders([
        'Accept' => 'application/json',
        'Authorization' => 'Bearer '.$accessToken,
    ])->get('https://passport-app.test/api/user');

    return $response->json();

<a name="token-scopes"></a>
## Token Scopes

Cakupan memungkinkan klien API Anda untuk meminta serangkaian izin tertentu saat meminta otorisasi untuk mengakses akun. Misalnya, jika Anda sedang membangun aplikasi e-niaga, tidak semua konsumen API memerlukan kemampuan untuk melakukan pemesanan. Sebagai gantinya, Anda dapat mengizinkan konsumen hanya meminta otorisasi untuk mengakses status pengiriman pesanan. Dengan kata lain, cakupan memungkinkan pengguna aplikasi Anda membatasi tindakan yang dapat dilakukan oleh aplikasi pihak ketiga atas nama mereka.
<a name="defining-scopes"></a>
### Defining Scopes

Anda dapat menentukan cakupan API menggunakan metode `Passport::tokensCan` dalam metode `boot` dari kelas `App\Providers\AuthServiceProvider` aplikasi Anda. Metode `tokenCan` menerima array nama ruang lingkup dan deskripsi ruang lingkup. Deskripsi ruang lingkup dapat berupa apa pun yang Anda inginkan dan akan ditampilkan kepada pengguna di layar persetujuan otorisasi:
    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::tokensCan([
            'place-orders' => 'Place orders',
            'check-status' => 'Check order status',
        ]);
    }

<a name="default-scope"></a>
### Default Scope

Jika klien tidak meminta cakupan tertentu, Anda dapat mengonfigurasi server Passport untuk melampirkan cakupan default ke token menggunakan metode `setDefaultScope`. Biasanya, Anda harus memanggil metode ini dari metode `boot` dari kelas `App\Providers\AuthServiceProvider` aplikasi Anda:
    use Laravel\Passport\Passport;

    Passport::tokensCan([
        'place-orders' => 'Place orders',
        'check-status' => 'Check order status',
    ]);

    Passport::setDefaultScope([
        'check-status',
        'place-orders',
    ]);

> **Note**
> Passport's default scopes do not apply to personal access tokens that are generated by the user.

<a name="assigning-scopes-to-tokens"></a>
### Assigning Scopes To Tokens

<a name="when-requesting-authorization-codes"></a>
#### When Requesting Authorization Codes

Saat meminta token akses menggunakan pemberian kode otorisasi, konsumen harus menentukan cakupan yang diinginkan sebagai parameter string kueri `scope`. Parameter `scope` harus berupa daftar cakupan yang dibatasi ruang:
    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => 'place-orders check-status',
        ]);

        return redirect('http://passport-app.test/oauth/authorize?'.$query);
    });

<a name="when-issuing-personal-access-tokens"></a>
#### When Issuing Personal Access Tokens

Jika Anda menerbitkan token akses pribadi menggunakan metode `createToken` model `App\Models\User`, Anda dapat meneruskan array cakupan yang diinginkan sebagai argumen kedua ke metode:
    
    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="checking-scopes"></a>
### Checking Scopes

Paspor menyertakan dua middleware yang dapat digunakan untuk memverifikasi bahwa permintaan yang masuk diautentikasi dengan token yang telah diberikan cakupan tertentu. Untuk memulai, tambahkan middleware berikut ke properti `$routeMiddleware` dari file `app/Http/Kernel.php` Anda:
    'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
    'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,

<a name="check-for-all-scopes"></a>
#### Check For All Scopes

Middleware `scopes` dapat ditugaskan ke rute untuk memverifikasi bahwa token akses permintaan yang masuk memiliki semua cakupan yang terdaftar:
    Route::get('/orders', function () {
        // Access token has both "check-status" and "place-orders" scopes...
    })->middleware(['auth:api', 'scopes:check-status,place-orders']);

<a name="check-for-any-scopes"></a>
#### Check For Any Scopes

Middleware `scope` dapat ditugaskan ke rute untuk memverifikasi bahwa token akses permintaan yang masuk memiliki *setidaknya satu* dari cakupan yang terdaftar:
    Route::get('/orders', function () {
        // Access token has either "check-status" or "place-orders" scope...
    })->middleware(['auth:api', 'scope:check-status,place-orders']);

<a name="checking-scopes-on-a-token-instance"></a>
#### Checking Scopes On A Token Instance

Setelah permintaan autentikasi token akses masuk ke aplikasi Anda, Anda masih dapat memeriksa apakah token memiliki cakupan tertentu menggunakan metode `tokenCan` pada instance `App\Models\User` yang diautentikasi:
    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        if ($request->user()->tokenCan('place-orders')) {
            //
        }
    });

<a name="additional-scope-methods"></a>
#### Additional Scope Methods

Metode `scopeIds` akan mengembalikan array dari semua ID/nama yang ditentukan:
    use Laravel\Passport\Passport;

    Passport::scopeIds();

Metode `scopes` akan mengembalikan array dari semua cakupan yang ditentukan sebagai turunan dari `Laravel\Passport\Scope`:
    Passport::scopes();

Metode `scopesFor` akan mengembalikan larik instance `Laravel\Passport\Scope` yang cocok dengan ID/nama yang diberikan:
    Passport::scopesFor(['place-orders', 'check-status']);

Anda dapat menentukan apakah cakupan tertentu telah ditentukan menggunakan metode `hasScope`:

    Passport::hasScope('place-orders');

<a name="consuming-your-api-with-javascript"></a>
## Consuming Your API With JavaScript

Saat membuat API, akan sangat berguna untuk dapat menggunakan API Anda sendiri dari aplikasi JavaScript Anda. Pendekatan pengembangan API ini memungkinkan aplikasi Anda sendiri menggunakan API yang sama dengan yang Anda bagikan dengan dunia. API yang sama dapat dikonsumsi oleh aplikasi web, aplikasi seluler, aplikasi pihak ketiga, dan SDK apa pun yang mungkin Anda terbitkan di berbagai pengelola paket.
Biasanya, jika Anda ingin menggunakan API dari aplikasi JavaScript, Anda perlu mengirimkan token akses ke aplikasi secara manual dan meneruskannya dengan setiap permintaan ke aplikasi Anda. Namun, Passport menyertakan middleware yang dapat menangani ini untuk Anda. Yang perlu Anda lakukan adalah menambahkan middleware `CreateFreshApiToken` ke grup middleware `web` Anda di file `app/Http/Kernel.php`:

    'web' => [
        // Other middleware...
        \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
    ],

> **Warning**  
> Anda harus memastikan bahwa middleware `CreateFreshApiToken` adalah middleware terakhir yang terdaftar di tumpukan middleware Anda.

Middleware ini akan melampirkan cookie `laravel_token` ke respons keluar Anda. Cookie ini berisi JWT terenkripsi yang akan digunakan Passport untuk mengautentikasi permintaan API dari aplikasi JavaScript Anda. JWT memiliki masa pakai yang sama dengan nilai konfigurasi `session.lifetime` Anda. Sekarang, karena browser akan mengirimkan cookie secara otomatis dengan semua permintaan berikutnya, Anda dapat membuat permintaan ke API aplikasi Anda tanpa meneruskan token akses secara eksplisit:

    axios.get('/api/user')
        .then(response => {
            console.log(response.data);
        });

<a name="customizing-the-cookie-name"></a>
#### Customizing The Cookie Name

Jika perlu, Anda dapat menyesuaikan nama cookie `laravel_token` menggunakan metode `Passport::cookie`. Biasanya, metode ini harus dipanggil dari metode `boot` kelas `App\Providers\AuthServiceProvider` aplikasi Anda:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::cookie('custom_name');
    }

<a name="csrf-protection"></a>
#### CSRF Protection

Saat menggunakan metode autentikasi ini, Anda perlu memastikan header token CSRF yang valid disertakan dalam permintaan Anda. Scaffolding Laravel JavaScript default menyertakan instance Axios, yang secara otomatis akan menggunakan nilai cookie `XSRF-TOKEN` terenkripsi untuk mengirim header `X-XSRF-TOKEN` pada permintaan asal yang sama.

> **Note**  
> Jika Anda memilih untuk mengirim header `X-CSRF-TOKEN` daripada `X-XSRF-TOKEN`, Anda harus menggunakan token tidak terenkripsi yang disediakan oleh `csrf_token()`.

<a name="events"></a>
## Events

Paspor memunculkan acara saat mengeluarkan token akses dan menyegarkan token. Anda dapat menggunakan kejadian ini untuk memangkas atau mencabut token akses lain di database Anda. Jika mau, Anda dapat melampirkan pemroses ke peristiwa ini di kelas `App\Providers\EventServiceProvider` aplikasi Anda:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Laravel\Passport\Events\AccessTokenCreated' => [
            'App\Listeners\RevokeOldTokens',
        ],

        'Laravel\Passport\Events\RefreshTokenCreated' => [
            'App\Listeners\PruneOldTokens',
        ],
    ];

<a name="testing"></a>
## Testing

Metode `actingAs` Passport dapat digunakan untuk menentukan pengguna yang saat ini diautentikasi serta cakupannya. Argumen pertama yang diberikan ke metode `actingAs` adalah instance pengguna dan yang kedua adalah larik cakupan yang harus diberikan ke token pengguna:

    use App\Models\User;
    use Laravel\Passport\Passport;

    public function test_servers_can_be_created()
    {
        Passport::actingAs(
            User::factory()->create(),
            ['create-servers']
        );

        $response = $this->post('/api/create-server');

        $response->assertStatus(201);
    }

Metode `actingAsClient` Passport dapat digunakan untuk menentukan klien yang saat ini diautentikasi serta cakupannya. Argumen pertama yang diberikan ke metode `actingAsClient` adalah instance klien dan yang kedua adalah larik cakupan yang harus diberikan ke token klien:

    use Laravel\Passport\Client;
    use Laravel\Passport\Passport;

    public function test_orders_can_be_retrieved()
    {
        Passport::actingAsClient(
            Client::factory()->create(),
            ['check-status']
        );

        $response = $this->get('/api/orders');

        $response->assertStatus(200);
    }
