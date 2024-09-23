# Pencatatan _Log_

- [Pendahuluan](#introduction)
- [Konfigurasi](#configuration)
    - [_Driver_ Kanal yang Tersedia](#available-channel-drivers)
    - [Prasyarat Kanal](#channel-prerequisites)
    - [Pencatatan _Log_ untuk Peringatan Keusangan](#logging-deprecation-warnings)
- [Membangun Tumpukan _Log_](#building-log-stacks)
- [Menulis Pesan _Log_](#writing-log-messages)
    - [Informasi Kontekstual](#contextual-information)
    - [Menulis pada Kanal Tertentu](#writing-to-specific-channels)
- [Kustomisasi Kanal Monolog](#monolog-channel-customization)
    - [Kustomisasi Monolog untuk Kanal](#customizing-monolog-for-channels)
    - [Membuat Kanal _Handler_ Monolog](#creating-monolog-handler-channels)
    - [Membuat Kanal Kustom Via _Factory_](#creating-custom-channels-via-factories)

<a name="introduction"></a>
## Pendahuluan

Untuk membantu Anda memahami apa yang terjadi pada aplikasi Anda, Laravel menyediakan layanan pencatatan _Log_ yang tangguh, yang memungkinkan Anda untuk melakukan pencatatan _log_ pada _file_, _log_ eror sistem, dan bahkan untuk melakukan pemberitahuan Slack untuk seluruh tim.

Pencatatan _log_ Laravel berlandaskan pada "kanal". Setiap kanal merepresentasikan sebuah cara tertentu untuk menulis informasi _log_. Sebagai contoh, kanal `single` melakukan penulisan _file log_ yang tunggal, sedangkan kanal `slack` akan mengirimkan pesan _log_ ke Slack. Pesan _log_ juga dapat ditulis pada banyak kanal berdasarkan tingkat keparahannya.

Di balik layar, Laravel menggunakan pustaka [Monolog](https://github.com/Seldaek/monolog), yang menyediakan dukungan untuk berbagai macam penangan log yang _powerful_. Laravel telah mempermudah Anda untuk mengonfigurasi penangan ini, sehingga Anda dapat melakukan penyesuaian penanganan _log_ untuk aplikasi Anda.

<a name="configuration"></a>
## Konfigurasi

Semua opsi konfigurasi untuk perilaku pencatatan _log_ milik aplikasi Anda berada pada _file_ `config/logging.php`. _File_ ini mengijinkan Anda untuk mengkonfigurasi kanal _log_ pada aplikasi. Pastikan untuk meninjau semua kanal yang tersedia beserta masing-masing opsinya. Pada bab ini, Kita akan meninjau beberapa opsi yang umum saja.

Secara _default_, Laravel akan menggunakan kanal `stack` ketika melakukan pencatatan pesan log. Kanal `stack` digunakan untuk mengagregat beberapa kanal _log_ menjadi kanal tunggal. Informasi lebih lanjut tentang pembangunan _stack_ dapat dilihat pada [paragraf di bawah](#building-log-stacks).

<a name="configuring-the-channel-name"></a>
#### Mengkonfigurasi Nama Kanal

Secara _default_, Monolog telah terinstantiasi dengan dengan sebuah "nama kanal" yang sesuai dengan _environtment_ (lingkungan aplikasi) saat ini. Untuk mengganti nilai ini, tambahkan opsi `nama` pada konfigurasi kanal milik Anda:

    'stack' => [
        'driver' => 'stack',
        'name' => 'nama-kanal',
        'channels' => ['single', 'slack'],
    ],

<a name="available-channel-drivers"></a>
### _Driver_ Kanal yang Tersedia

Setiap kanal _log_ telah ditenagai oleh sebuah "_driver_". _Driver_ tersebut menentukan bagaimana dan di mana pesan _log_ sebenarnya direkam. _Driver_ untuk kanal _log_ di bawah telah tersedia pada setiap aplikasi Laravel. Entri untuk sebagian besar _driver_ ini sudah ada di dalam _file_ konfigurasi `config/logging.php`, jadi pastikan untuk meninjau _file_ ini untuk mengenal isinya:

Nama | Keterangan
------------- | -------------
`custom` | _Driver_ yang memanggil _factory_ tertentu untuk membuat sebuah
`daily` | _Driver_ Monolog berbasis `RotatingFileHandler` yang berputar setiap hari
`errorlog` | _Driver_ Monolog berbasis `ErrorLogHandler`
`monolog` | _Driver factory_ Monolog yang dapat digunakan untuk penangan Monolog yang mendukung
`null` | _Driver_ yang mengabaikan semua pesan _log_
`papertrail` | _Driver_ Monolog berbasis `SyslogUdpHandler`
`single` | Kanal pencatat _log_ berbasis _file_ atau _path_ yang tunggal (`StreamHandler`)
`slack` | _Driver_ Monolog berbasis `SlackWebhookHandler`
`stack` | Sebuah pembungkus untuk memfasilitasi pembuatan kanal yang "multi-kanal"
`syslog` | _Driver_ Monolog berbasis `SyslogHandler`

> **Catatan**  
> Periksalah dokumentasi [penyesuaian kanal tingkat lanjut](#monolog-channel-customization) untuk mempelajari _driver_ `monolog` dan `custom`.

<a name="channel-prerequisites"></a>
### Prasyarat Kanal

<a name="configuring-the-single-and-daily-channels"></a>
#### Melakukan Konfigurasi pada Kanal _Single_ dan _Daily_

Kanal `single` dan `daily` memiliki tiga opsi konfigurasi yang opsional: `bubble`, `permission`, dan `locking.`

Name | Keterangan | _Default_
------------- | ------------- | -------------
`bubble` | Mengindikasikan bahwa pesan-pesan harus melayang ("_bubble up_") ke kanal lain setelah ditangani | `true`
`locking` | Mencoba mengunci _file log_ sebelum menulis ke dalam _file_ tersebut | `false`
`permission` | _Permission_ untuk _file log_ | `0644`

Selain itu, kebijakan retensi untuk kanal `daily` dapat dikonfigurasi melalui opsi `days`

Nama | Keterangan                                                       | _Default_
------------- |-------------------------------------------------------------------| -------------
`days` | Banyaknya hari yang harus dipertahankan oleh _file log_ harian | `7`

<a name="configuring-the-papertrail-channel"></a>
#### Melakukan Konfigurasi pada Kanal Papertrail

Kanal `pepertrail` mewajibkan opsi konfigurasi untuk `host` dan `port`. Anda dapat mendapatkan nilai-nilai untuk konfigurasi pada [Papertrail](https://help.papertrailapp.com/kb/configuration/configuring-centralized-logging-from-php-apps/#send-events-from-php-app).

<a name="configuring-the-slack-channel"></a>
#### Melakukan Konfigurasi pada Kanal Slack

Kanal `slack` mewajibkan sebuah opsi konfigurasi untuk `url`. URL tersebut harus sesuai dengan URL untuk [_webhook_ yang masuk](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks) yang telah Anda konfigurasi untuk tim Slack anda.

Secara _default_, Slack hanya akan menerima _log_ yang memiliki level `critical` dan diatasnya; namun, Anda dapat menyelesaikan hal ini pada _file_ konfigurasi `config/logging.php` dengan memodifikasi opsi `level` pada _array_ konfigurasi milik kanal _log_ Slack Anda.

<a name="logging-deprecation-warnings"></a>
### Pencatatan _Log_ untuk Peringatan Keusangan

PHP, Laravel, dan pustaka yang lain sering memberitahu pengguna bahwa beberapa fitur milik mereka telah ditinggalkan (_deprecated_) dan akan dihapus pada versi mendatang. Jika Anda ingin melakukan pencatatan _log_ untuk peringatan keusangan (_deprecation_) tersebut, Anda dapat mendefinisikan nama kanal _log_ untuk `deprecations` yang Anda sukai pada _file_ konfigurasi `config/logging.php` untuk aplikasi Anda:

    'deprecations' => env('LOG_DEPRECATIONS_CHANNEL', 'null'),

    'channels' => [
        ...
    ]

Atau, Anda dapat mendefinisikan sebuah kanal _log_ dengan nama `deprecations`. Jika nama kanal tersebut terdapat pada _array_ `channel`, maka kanal tersebut akan selalu digunakan untuk mencatat _log_ peringatan keusangan:

    'channels' => [
        'deprecations' => [
            'driver' => 'single',
            'path' => storage_path('logs/php-deprecation-warnings.log'),
        ],
    ],

<a name="building-log-stacks"></a>
## Membangun Tumpukan _Log_

Seperti yang sudah disebutkan sebelumnya, _driver_ `stack` memungkinkan Anda untuk mengkombinasikan beberapa kanal menjadi satu kanal untuk mempermudah. Untuk mengilustrasikan bagaimana cara "menumpuk" _log_, mari kita sebuah contoh konfigurasi yang mungkin akan Anda lihat pada aplikasi tahap _production_:

    'channels' => [
        'stack' => [
            'driver' => 'stack',
            'channels' => ['syslog', 'slack'],
        ],

        'syslog' => [
            'driver' => 'syslog',
            'level' => 'debug',
        ],

        'slack' => [
            'driver' => 'slack',
            'url' => env('LOG_SLACK_WEBHOOK_URL'),
            'username' => 'Laravel Log',
            'emoji' => ':boom:',
            'level' => 'critical',
        ],
    ],

Mari kita bedah konfigurasi tersebut. Pertama, perhatikan saluran `stack` kita yang menggabungkan dua saluran lain melalui opsi `channels`: `syslog` dan `slack`. Jadi, ketika mencatat pesan _log_, kedua saluran ini akan memiliki kesempatan untuk mencatat pesan _log_. Namun, seperti yang akan kita lihat di bawahnya, kanal-kanal tersebut hanya mencatat pesan _log_ untuk tingkat keparahan / "level" yang sudah ditentukan.

<a name="log-levels"></a>
#### Tingkatan _Log_

Perhatikan opsi konfigurasi `level` yang ada pada konfigurasi kanal `syslog` dan `slack` pada contoh di atas. Opsi ini menentukan "level" minimum yang harus dimiliki sebuah pesan agar dapat dicatat oleh kanal. Monolog, yang mendukung layanan pencatatan Laravel, menawarkan semua level _log_ yang didefinisikan pada [spesifikasi RFC 5424](https://tools.ietf.org/html/rfc5424). Diurutkan dari tingkat keparahan yang paling tinggi, adapun level-level _log_ yang terdaftar adalah: **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info**, dan **debug**.

Jadi, bayangkan kita mencatat pesan _log_ menggunakan metode `debug`:

    Log::debug('Pesan log tercatat.');

Berdasarkan konfigurasi yang sudah dijelaskan di atas, kanal `syslog` akan menulis pesan ke _log_ sistem; namun, karena pesan kesalahan tidak `critical` atau lebih tinggi, pesan tersebut tidak akan dikirim ke Slack. Namun, jika kita mencatat pesan `emergency`, pesan tersebut akan dikirim ke _log_ sistem dan Slack karena tingkat `emergency` lebih tinggi dari ambang batas minimum untuk kedua kanal:

    Log::emergency('Sistem sedang down!');

<a name="writing-log-messages"></a>
## Menulis Pesan _Log_

Anda dapat menulis sebuah informasi ke _log_ menggunakan [_facade_](/docs/{{version}}/facades) `Log`. Seperti yang sudah disebutkan sebelumnya, pencatat _log_ menyediakan delapan level _log_ yang didefinisikan pada [spesifikasi RFC 5424](https://tools.ietf.org/html/rfc5424): **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info** dan **debug**:

    use Illuminate\Support\Facades\Log;

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

Anda dapat menggunakan metode-metode ini untuk melakukan pencatatan _log_ dengan level yg sesuai. Secara _default_, pesan akan dituliskan pada kanal _log_ yang _default_ yang telah diatur pada _file_ konfigurasi untuk `logging`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\User;
    use Illuminate\Support\Facades\Log;

    class UserController extends Controller
    {
        /**
         * Menampilkan profil untuk pengguna yang telah ditentukan.
         *
         * @param  int  $id
         * @return \Illuminate\Http\Response
         */
        public function show($id)
        {
            Log::info('Menampilkan profil pengguna dengan ID: '.$id);

            return view('user.profile', [
                'user' => User::findOrFail($id)
            ]);
        }
    }

<a name="contextual-information"></a>
### Informasi Kontekstual

Sebuah _array_ untuk data kontekstual dapat dioper kepada metode-metode _log_. Data kontekstual ini akan diformat dan ditampilkan pada pesan _log_:

    use Illuminate\Support\Facades\Log;

    Log::info('Pengguna gagal melakukan login.', ['id' => $user->id]);

Terkadang, Anda mungkin ingin menentukan beberapa informasi kontekstual yang harus disertakan pada semua entri _log_ pada kanal tertentu. Sebagai contoh, Anda mungkin ingin mencatat ID permintaan (_request_) yang terasosiasi dengan setiap permintaan yang masuk ke aplikasi Anda. Untuk mencapai hal ini, Anda dapat memanggil metode `withContext` milik _facade_ `Log`:

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Support\Facades\Log;
    use Illuminate\Support\Str;

    class AssignRequestId
    {
        /**
         * Menangani permintaan yang masuk.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            $requestId = (string) Str::uuid();

            Log::withContext([
                'request-id' => $requestId
            ]);

            return $next($request)->header('Request-Id', $requestId);
        }
    }

Jika Anda ingin berbagi informasi kontekstual dengan semua kanal pecatat _log_, Anda dapat memanggil metode `Log::shareContext()`. Metode ini akan memberikan informasi kontekstual ke semua kanal yang sudah dibuat dan semua kanal yang dibuat setelahnya. Biasanya, metode `shareContext` harus dipanggil dari metode `boot` dari penyedia layanan aplikasi:

    use Illuminate\Support\Facades\Log;
    use Illuminate\Support\Str;

    class AppServiceProvider
    {
        /**
         * Bootstrap layanan untuk aplikasi.
         *
         * @return void
         */
        public function boot()
        {
            Log::shareContext([
                'invocation-id' => (string) Str::uuid(),
            ]);
        }
    }

<a name="writing-to-specific-channels"></a>
### Menulis ke Kanal Tertentu

Terkadang Anda mungkin ingin mencatat pesan ke kanal selain kanal _default_. Anda dapat menggunakan metode `channel` pada _facade_ `Log` untuk mengambil dan mencatat ke kanal lain yang yang sudah ditentukan dalam _file_ konfigurasi Anda:

    use Illuminate\Support\Facades\Log;

    Log::channel('slack')->info('Something happened!');

Jika Anda ingin membuat tumpukan pencatatan _log_—yang bersifat _on-demand_ (berjalan saat _runtime_)—yang terdiri dari beberapa kanal, Anda dapat menggunakan metode `stack`:

    Log::stack(['single', 'slack'])->info('Sesuatu terjadi!');

<a name="on-demand-channels"></a>
#### Kanal _On-Demand_

Anda juga dapat membuat kanal _on-demand_ dengan memberikan konfigurasi kanal pada saat _runtime_ walau konfigurasi tersebut tidak terdapat terdapat pada _file_ konfigurasi `logging` aplikasi Anda. Untuk mencapai hal ini, Anda dapat mengoper _array_ yang berisi konfigurasi kanal ke metode `build` milik _facade_ `Log`:

    use Illuminate\Support\Facades\Log;

    Log::build([
      'driver' => 'single',
      'path' => storage_path('logs/custom.log'),
    ])->info('Sesuat terjadi!');

Anda mungkin juga ingin menyertakan kanal _on-demand_ dalam tumpukan yang _on-demand_ juga. Hal ini dapat dicapai dengan menyertakan _instance_ dari kanal _on-demand_ Anda dalam _array_ yang dioper ke metode `stack`:

    use Illuminate\Support\Facades\Log;

    $channel = Log::build([
      'driver' => 'single',
      'path' => storage_path('logs/custom.log'),
    ]);

    Log::stack(['slack', $channel])->info('Sesuatu terjadi!');

<a name="monolog-channel-customization"></a>
## Kustomisasi Kanal Monolog

<a name="customizing-monolog-for-channels"></a>
### Kustomisasi Monolog untuk Kanal

Terkadang Anda mungkin membutuhkan kontrol penuh atas bagaimana Monolog dikonfigurasi untuk kanal yang sudah ada. Sebagai contoh, Anda mungkin ingin mengonfigurasi implementasi `FormatterInterface` Monolog yang kustom untuk kanal `single` bawaan Laravel.

Untuk memulainya, definisikan _array_ `tap` pada konfigurasi kanal. _Array_ `tap` harus berisi daftar kelas yang memiliki kesempatan untuk menyesuaikan (atau "mengetuk" ke dalam) _instance_ Monolog setelah dibuat. Tidak ada direktori yang konvensional untuk penempatan kelas-kelas ini, jadi Anda bebas membuat direktori pada aplikasi Anda untuk menampung kelas-kelas ini:

    'single' => [
        'driver' => 'single',
        'tap' => [App\Logging\CustomizeFormatter::class],
        'path' => storage_path('logs/laravel.log'),
        'level' => 'debug',
    ],

Setelah Anda mengonfigurasi opsi `tap` pada kanal Anda, Anda siap untuk mendefinisikan kelas yang akan mengkustom _instance_ Monolog Anda. Kelas ini hanya membutuhkan satu metode: `__invoke`, yang menerima _instance_ `Illuminate\Log\Logger`. _Instance_ `Illuminate\Log\Logger` memproksikan semua pemanggilan metode ke _instance_ Monolog yang mendasarinya:

    <?php

    namespace App\Logging;

    use Monolog\Formatter\LineFormatter;

    class CustomizeFormatter
    {
        /**
         * Mengkustom instance Monolog yang ditentukan.
         *
         * @param  \Illuminate\Log\Logger  $logger
         * @return void
         */
        public function __invoke($logger)
        {
            foreach ($logger->getHandlers() as $handler) {
                $handler->setFormatter(new LineFormatter(
                    '[%datetime%] %channel%.%level_name%: %message% %context% %extra%'
                ));
            }
        }
    }

> **Catatan**  
> Semua kelas-kelas "tap" telah di-_resolve_ oleh [_service container_](/docs/{{version}}/container), maka semua dependensi pada _constructor_ yang diwajibkan akan diinjeksikan secara otomatis.

<a name="creating-monolog-handler-channels"></a>
### Membuat Kanal _Handler_ Monolog

Monolog memiliki berbagai macam [_handler_] (https://github.com/Seldaek/monolog/tree/main/src/Monolog/Handler) dan Laravel tidak menyertakan kanal bawaan untuk _handler_-_handler_ tersebut. Dalam beberapa kasus, Anda mungkin ingin membuat kanal kustom yang mana merupakan sebuah _instance_ dari _handler_ Monolog tertentu yang tidak terdapat pada daftar _driver log_ Laravel bawaan. Kanal-kanal ini dapat dengan mudah dibuat menggunakan _driver_ `monolog`.

Ketika menggunakan _driver_ `monolog`, opsi konfigurasi `handler` digunakan untuk menentukan _handler_ mana yang akan di-instansiasi. Secara opsional, parameter-parameter _constructor_ yang dibutuhkan oleh _handler_ dapat ditentukan dengan menggunakan opsi konfigurasi `with`:

    'logentries' => [
        'driver'  => 'monolog',
        'handler' => Monolog\Handler\SyslogUdpHandler::class,
        'with' => [
            'host' => 'my.logentries.internal.datahubhost.company.com',
            'port' => '10000',
        ],
    ],

<a name="monolog-formatters"></a>
#### Pemformat Monolog

Saat menggunakan _driver_ `monolog`, `LineFormatter` Monolog akan digunakan sebagai pemformat yang _default_. Namun, Anda dapat menyesuaikan jenis pemformat yang dioper ke _handler_ menggunakan opsi konfigurasi `formatter` dan `formatter_with`:

    'browser' => [
        'driver' => 'monolog',
        'handler' => Monolog\Handler\BrowserConsoleHandler::class,
        'formatter' => Monolog\Formatter\HtmlFormatter::class,
        'formatter_with' => [
            'dateFormat' => 'Y-m-d',
        ],
    ],

Jika Anda menggunakan _handler_ Monolog yang mampu menyediakan pemformatnya sendiri, Anda dapat mengatur nilai opsi konfigurasi `formatter` ke `default`:

    'newrelic' => [
        'driver' => 'monolog',
        'handler' => Monolog\Handler\NewRelicHandler::class,
        'formatter' => 'default',
    ],

<a name="creating-custom-channels-via-factories"></a>
### Membuat Kanal Kustom Via _Factory_

Jika Anda ingin mendefinisikan sebuah kanal yang sepenuhnya kustom di mana Anda memiliki kendali penuh atas instansasi dan konfigurasi Monolog, Anda dapat menentukan tipe _driver_ `custom` dalam _file_ konfigurasi `config/logging.php`. Konfigurasi Anda harus menyertakan opsi `via` yang berisi nama kelas _factory_ yang akan dipanggil untuk membuat _instance_ Monolog:

    'channels' => [
        'example-custom-channel' => [
            'driver' => 'custom',
            'via' => App\Logging\CreateCustomLogger::class,
        ],
    ],

Setelah Anda mengkonfigurasi saluran _driver_ `custom`, Anda siap mendefinisikan kelas yang akan membuat _instance_ Monolog. Kelas ini hanya membutuhkan satu metode `__invoke` yang akan mengembalikan _instance_ pencatat _log_ Monolog. Metode ini akan menerima _array_ konfigurasi kanal sebagai satu-satunya argumen:

    <?php

    namespace App\Logging;

    use Monolog\Logger;

    class CreateCustomLogger
    {
        /**
         * Membuat sebuah instance Monolog kustom.
         *
         * @param  array  $config
         * @return \Monolog\Logger
         */
        public function __invoke(array $config)
        {
            return new Logger(/* ... */);
        }
    }
