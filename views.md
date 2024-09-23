# _View_

- [Pendahuluan](#introduction)
    - [Menulis _View_ dengan React / Vue](#writing-views-in-react-or-vue)
- [Membuat & Me-_render_ _View_](#creating-and-rendering-views)
    - [Direktori _View_ yang Bersarang](#nested-view-directories)
    - [Membuat _View_ yang Tersedia Saat Pertama Kali](#creating-the-first-available-view)
    - [Penentuan Keberadaan _View_](#determining-if-a-view-exists)
- [Mengoper Data ke _View_](#passing-data-to-views)
    - [Berbagi Data dengan Semua _View_](#sharing-data-with-all-views)
- [Komposer _View_](#view-composers)
    - [Kreator _View_](#view-creators)
- [Optimasi _View_](#optimizing-views)

<a name="introduction"></a>
## Pendahuluan

Tentu saja, tidak praktis untuk mengembalikan seluruh _string_ dokumen HTML secara langsung dari rute dan _controller_ Anda. Untungnya, _view_ menyediakan cara yang nyaman untuk menempatkan semua HTML kita ke dalam _file_ yang terpisah.

_View_ memisahkan logika _controller_/aplikasi Anda dari logika tampilan (presentasi) Anda yang disimpan di dalam direktori `resources/views`. Ketika menggunakan Laravel, _template_ view biasanya ditulis menggunakan [bahasa penataan letak bernama Blade](/docs/{{version}}/blade). Tampilan sederhana mungkin terlihat seperti ini:

```blade
<!-- View ini tersimpan di resources/views/greeting.blade.php -->

<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```

Karena _view_ ini disimpan di `resources/views/greeting.blade.php`, kita dapat mengembalikannya menggunakan _helper_ global `view` seperti ini:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

> **Catatan**  
> Mencari informasi lebih lanjut tentang cara menulis _template_ Blade? Lihat dokumentasi lengkap [Blade](/docs/{{version}}/blade) untuk memulai.

<a name="writing-views-in-react-or-vue"></a>
### Menulis _View_ dengan React / Vue

Daripada menulis _template frontend_ di PHP menggunakan Blade, banyak pengembang mulai menyukai menulis _template_ mereka menggunakan React atau Vue. Laravel telah membuat hal ini menjadi mudah berkat [Inertia](https://inertiajs.com/), sebuah _library_ yang memudahkan pengikatan _frontend_ React/Vue Anda ke _backend_ Laravel tanpa kerumitan yang biasanya terjadi pada pembangunan SPA.

[_Starter kit_](/docs/{{version}}/starter-kits) Breeze dan Jetstream memberikan Anda sebuah permulaan yang baik untuk aplikasi Laravel Anda yang berikutnya dengan bantuan Inertia. Selain itu, [Bootcamp Laravel](https://bootcamp.laravel.com) memberikan demonstrasi lengkap untuk membangun aplikasi Laravel yang didukung oleh Inertia, disertai contoh-contoh dalam Vue dan React.

<a name="creating-and-rendering-views"></a>
## Membuat & Me-_render_ _View_

Anda dapat membuat tampilan dengan menempatkan _file_ dengan ekstensi `.blade.php` di direktori `resources/views` aplikasi Anda. Ekstensi `.blade.php` menginformasikan _framework_ bahwa _file_ tersebut berisi [_template_ Blade](/docs/{{version}}/blade). _Template_ Blade berisi HTML serta _directive_ Blade yang memungkinkan Anda untuk dengan mudah menampilkan nilai, membuat pernyataan "if", melakukan iterasi data, dan banyak lagi.

Setelah Anda membuat _view_, Anda dapat mengembalikannya dari salah satu rute atau _controller_ aplikasi Anda menggunakan _helper_ global `view`:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

_View_ juga dapat dikembalikan menggunakan _facade_ `View`:

    use Illuminate\Support\Facades\View;

    return View::make('greeting', ['name' => 'James']);

Seperti yang anda lihat, argumen pertama yang dioper ke _helper_ `view` telah sesuai dengan nama _file view_ di direktori `resources/views`. Argumen kedua adalah _array_ data yang harus tersedia untuk _view_. Dalam hal ini, kita mengoper variabel `name`, yang nantinya ditampilkan di dalam _view_ menggunakan [sintaks Blade](/docs/{{version}}/blade).

<a name="nested-view-directories"></a>
### Direktori _View_ yang Bersarang

_View_ juga dapat dimasukkan ke dalam subdirektori di dalam `resources/views`. Tanda baca "titik" dapat digunakan untuk mereferensikan _view_ yang bersarang (berada di dalam subdirektori). Misalnya, jika _view_ Anda disimpan di `resources/views/admin/profile.blade.php`, Anda dapat mengembalikannya dari salah satu rute / _controller_ aplikasi Anda seperti ini:

    return view('admin.profile', $data);

> **Peringatan**  
> Nama untuk direktori seharusnya tidak mengandung karakter `.` (titik)

<a name="creating-the-first-available-view"></a>
### Membuat _View_ yang Tersedia Saat Pertama Kali

Dengan menggunakan metode `first` milik _facade_ `View`, Anda dapat membuat prioritas _view_ yang diurutkan di dalam _array_ tampilan. Ini dapat berguna jika aplikasi atau _package_ Anda memungkinkan tampilan untuk dikustomisasi atau ditimpa:

    use Illuminate\Support\Facades\View;

    return View::first(['custom.admin', 'admin'], $data);

Seperti yang terlihat pada contoh di atas, `return` akan mengembalikan _view_ `admin` jika _file_ `custom.admin` tidak ditemukan. Namun, _view_ `admin` akan ditimpa jika _view_ `custom.admin` telah dibuat pada aplikasi Anda.

<a name="determining-if-a-view-exists"></a>
### Penentuan Keberadaan _View_

Jika Anda perlu menentukan apakah sebuah view telah dibuat, Anda dapat menggunakan _facade_ `View`. Metode `exists` akan mengembalikan `true` jika view berada pada tempatnya:

    use Illuminate\Support\Facades\View;

    if (View::exists('emails.customer')) {
        //
    }

<a name="passing-data-to-views"></a>
## Mengoper Data ke _View_

Seperti yang Anda lihat dalam contoh-contoh sebelumnya, Anda dapat mengoper _array_ data ke _view_ untuk membuat data tersebut "terbaca" pada _view_:

    return view('greetings', ['name' => 'Victoria']);

Ketika meneruskan informasi dengan cara ini, data harus berupa _array_ dengan pasangan kunci-nilai. Setelah memberikan data ke _view_, Anda kemudian dapat mengakses setiap nilai dalam _view_ Anda menggunakan kunci (_array_ asosiatif) data, seperti `<?php echo $name; ?>`.

Sebagai alternatif untuk mengoper semua _array_ data ke fungsi pembantu `view`, anda dapat menggunakan metode `with` untuk menambahkan potongan-potongan data individual ke _view_. Metode `with` mengembalikan sebuah _instance_ dari objek _view_ sehingga Anda dapat melakukan perantaian metode sebelum mengembalikan _view_:

    return view('greeting')
                ->with('name', 'Victoria')
                ->with('occupation', 'Astronaut');

<a name="sharing-data-with-all-views"></a>
### Berbagi Data dengan Semua _View_

Terkadang, Anda mungkin perlu berbagi data dengan semua _view_ yang di-_render_ oleh aplikasi Anda. Anda dapat melakukannya dengan menggunakan metode `share` milik _facade_ `View`. Biasanya, Anda harus menempatkan pemanggilan metode `share` di dalam metode `boot` milik _service provider_. Anda bebas menambahkan ke kelas `App\Providers\AppServiceProvider` atau membuat _service provider_ lain untuk menampungnya:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Mendaftarkan service pada aplikasi.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Bootstrap service milik aplikasi.
         *
         * @return void
         */
        public function boot()
        {
            View::share('key', 'value');
        }
    }

<a name="view-composers"></a>
## Komposer _View_

Komposer _view_ adalah _callback_ atau metode kelas yang dipanggil saat tampilan di-_render_. Jika Anda memiliki data yang ingin diikat ke tampilan setiap kali tampilan tersebut di-_render_, komposer _view_ dapat membantu Anda mengatur logika tersebut ke dalam satu lokasi. Komposer _view_ akan sangat berguna jika tampilan yang sama dikembalikan oleh beberapa rute atau _controller_ di dalam aplikasi Anda selalu membutuhkan bagian data tertentu.

Biasanya, komposer _view_ akan didaftarkan dalam salah satu [penyedia layanan](/docs/{{version}}/providers) aplikasi Anda. Dalam contoh ini, kita akan mengasumsikan bahwa kita telah membuat `App\Providers\ViewServiceProvider` baru untuk menampung logika ini.

Kita akan menggunakan metode `composer` milik _facade_ `View` untuk mendaftarkan komposer _view_. Laravel tidak menyertakan direktori _default_ untuk komposer _view_ berbasis kelas, jadi Anda bebas mengaturnya sesuai keinginan Anda. Sebagai contoh, Anda dapat membuat direktori `App/View/Composers` untuk menampung semua komposer _view_ untuk aplikasi Anda:

    <?php

    namespace App\Providers;

    use App\View\Composers\ProfileComposer;
    use Illuminate\Support\Facades\View;
    use Illuminate\Support\ServiceProvider;

    class ViewServiceProvider extends ServiceProvider
    {
        /**
         * Mendaftarkan service pada aplikasi.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Bootstrap service milik aplikasi.
         *
         * @return void
         */
        public function boot()
        {
            // Mengunakan komposes berbasis kelas...
            View::composer('profile', ProfileComposer::class);

            // Menggunakan komposer berbasis closure...
            View::composer('dashboard', function ($view) {
                //
            });
        }
    }

> **Peringatan**  
> Ingat, jika Anda membuat _service provider_ baru yang memuat pendaftaran komposer _view_, Anda perlu menambahkan _service provider_ tersebut ke _array_ `providers` di file konfigurasi `config/app.php`.

Sekarang kita telah mendaftarkan komposer, metode `compose` dari kelas `App\View\Composers\ProfileComposer` akan dieksekusi setiap kali tampilan `profile` di-_render_. Mari kita lihat contoh kelas komposer:

    <?php

    namespace App\View\Composers;

    use App\Repositories\UserRepository;
    use Illuminate\View\View;

    class ProfileComposer
    {
        /**
         * Implementasi repository user.
         *
         * @var \App\Repositories\UserRepository
         */
        protected $users;

        /**
         * Membuat profil komposer yang baru.
         *
         * @param  \App\Repositories\UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Mengikat data ke view.
         *
         * @param  \Illuminate\View\View  $view
         * @return void
         */
        public function compose(View $view)
        {
            $view->with('count', $this->users->count());
        }
    }

Seperti yang Anda lihat, semua komposer _view_ diselesaikan melalui [_service container_](/docs/{{version}}/container), jadi Anda dapat melakukan _type-hint_ dependensi apa pun yang Anda perlukan dalam _constructor_ milik komposer.

<a name="attaching-a-composer-to-multiple-views"></a>
#### Menempelkan Komposer pada Beberapa _View_

Anda dapat menempelkan komposer ke beberapa _view_ sekaligus dengan cara mengoper _array view_ sebagai argumen pertama pada metode `composer`:

    use App\Views\Composers\MultiComposer;

    View::composer(
        ['profile', 'dashboard'],
        MultiComposer::class
    );

Metode `composer` juga menerima karakter `*` sebagai _wildcard_, yang memungkinkan Anda untuk melampirkan komposer ke semua _view_:

    View::composer('*', function ($view) {
        //
    });

<a name="view-creators"></a>
### Kreator _View_

"Kreator" _view_ sangat mirip dengan komposer _view_; namun, kreator _view_ dieksekusi segera setelah _view_ diinstansiasi, bukannya menunggu sampai _view_ akan di-_render_ seperti pada komposer. Untuk mendaftarkan kreator _view_, Anda dapat menggunakan metode `creator`:

    use App\View\Creators\ProfileCreator;
    use Illuminate\Support\Facades\View;

    View::creator('profile', ProfileCreator::class);

<a name="optimizing-views"></a>
## Optimasi _View_

Secara _default_, _view_ yang menggunakan _template_ Blade akan dikompilasi sesuai permintaan. Ketika mengeksekusi _request_ yang me-_render_ tampilan, Laravel akan menentukan apakah versi tampilan yang dikompilasi tersebut sudah ada. Jika _file_ tersebut ada, Laravel kemudian akan menentukan apakah _view_ yang belum dikompilasi telah dimodifikasi (berbeda dengan _view_ yang sudah dikompilasi). Jika tidak ada _view_ yang sudah dikompilasi, atau _view_ yang belum dikompilasi telah dimodifikasi, Laravel akan mengkompilasi ulang _view_ tersebut.

Mengkompilasi tampilan pada saat melakukan _request_ mungkin memiliki sedikit dampak negatif pada kinerja, sehingga Laravel menyediakan perintah Artisan `view:cache` untuk mengkompilasi semua _view_ yang digunakan oleh aplikasi Anda sebelum melakukan _request_. Untuk meningkatkan kinerja, Anda mungkin ingin menjalankan perintah ini sebagai bagian dari proses _deployment_ Anda:

```shell
php artisan view:cache
```

Anda bisa menggunakan perintah `view:clear` untuk menghapus _cache view_ yang telah dibuat sebelumnya:

```shell
php artisan view:clear
```
