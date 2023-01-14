# _View_

- [Pendahuluan](#introduction)
    - [Menulis _View_ dengan React / Vue](#writing-views-in-react-or-vue)
- [Membuat & Me-_render_ _View_](#creating-and-rendering-views)
    - [Direktori View yang Bersarang](#nested-view-directories)
    - [Membuat _View_ yang Tersedia Saat Pertama Kali](#creating-the-first-available-view)
    - [Penentuan Keberadaan _View_](#determining-if-a-view-exists)
- [Mengoper Data ke _View_](#passing-data-to-views)
    - [Berbagi Data dengan Semua _View_](#sharing-data-with-all-views)
- [Komposer _View_](#view-composers)
    - [Kreator _View_](#view-creators)
- [Optimasi _View_](#optimizing-views)

<a name="introduction"></a>
## Pendahuluan

Of course, it's not practical to return entire HTML documents strings directly from your routes and controllers. Thankfully, views provide a convenient way to place all of our HTML in separate files.

Views separate your controller / application logic from your presentation logic and are stored in the `resources/views` directory. When using Laravel, view templates are usually written using the [Blade templating language](/docs/{{version}}/blade). A simple view might look something like this:

Tentu saja, tidak praktis untuk mengembalikan seluruh string dokumen HTML secara langsung dari rute dan pengontrol Anda. Untungnya, view menyediakan cara yang nyaman untuk menempatkan semua HTML kita dalam file terpisah.

View memisahkan logika controller/aplikasi Anda dari logika presentasi Anda dan disimpan di direktori `resources/views`. Ketika menggunakan Laravel, templat view biasanya ditulis menggunakan [Blade templating language] (/docs/{{version}}/blade). Tampilan sederhana mungkin terlihat seperti ini:

```blade
<!-- View ini tersimpan di resources/views/greeting.blade.php -->

<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```

Since this view is stored at `resources/views/greeting.blade.php`, we may return it using the global `view` helper like so:

Karena view ini disimpan di `resources/views/greeting.blade.php`, kita dapat mengembalikannya menggunakan helper `view` global seperti ini:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

> **Catatan**  
> Looking for more information on how to write Blade templates? Check out the full [Blade documentation](/docs/{{version}}/blade) to get started.

> Mencari informasi lebih lanjut tentang cara menulis template Blade? Lihat dokumentasi lengkap [Blade](/docs/{{version}}/blade) untuk memulai.

<a name="writing-views-in-react-or-vue"></a>
### Menulis _View_ dengan React / Vue

Instead of writing their frontend templates in PHP via Blade, many developers have begun to prefer to write their templates using React or Vue. Laravel makes this painless thanks to [Inertia](https://inertiajs.com/), a library that makes it a cinch to tie your React / Vue frontend to your Laravel backend without the typical complexities of building an SPA.

Our Breeze and Jetstream [starter kits](/docs/{{version}}/starter-kits) give you a great starting point for your next Laravel application powered by Inertia. In addition, the [Laravel Bootcamp](https://bootcamp.laravel.com) provides a full demonstration of building a Laravel application powered by Inertia, including examples in Vue and React.

Daripada menulis templat frontend mereka di PHP melalui Blade, banyak pengembang mulai lebih suka menulis templat mereka menggunakan React atau Vue. Laravel membuat hal ini menjadi mudah berkat [Inertia](https://inertiajs.com/), sebuah library yang membuatnya mudah untuk mengikat frontend React/Vue Anda ke backend Laravel Anda tanpa kerumitan yang khas dalam membangun SPA.

Breeze dan Jetstream [starter kit](/docs/{{{version}}/starter-kits) memberikan Anda titik awal yang bagus untuk aplikasi Laravel Anda berikutnya yang didukung oleh Inertia. Selain itu, [Laravel Bootcamp](https://bootcamp.laravel.com) memberikan demonstrasi lengkap untuk membangun aplikasi Laravel yang didukung oleh Inertia, termasuk contoh-contoh dalam Vue dan React.

<a name="creating-and-rendering-views"></a>
## Membuat & Me-_render_ _View_

You may create a view by placing a file with the `.blade.php` extension in your application's `resources/views` directory. The `.blade.php` extension informs the framework that the file contains a [Blade template](/docs/{{version}}/blade). Blade templates contain HTML as well as Blade directives that allow you to easily echo values, create "if" statements, iterate over data, and more.

Once you have created a view, you may return it from one of your application's routes or controllers using the global `view` helper:

Anda dapat membuat tampilan dengan menempatkan file dengan ekstensi `.blade.php` di direktori `resources/views` aplikasi Anda. Ekstensi `.blade.php` menginformasikan framework bahwa file tersebut berisi [Blade template] (/docs/{{version}}/blade). Templat Blade berisi HTML serta arahan Blade yang memungkinkan Anda untuk dengan mudah menggemakan nilai, membuat pernyataan "if", mengulang data, dan banyak lagi.

Setelah Anda membuat tampilan, Anda dapat mengembalikannya dari salah satu rute atau pengontrol aplikasi Anda menggunakan helper `view` global:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

Views may also be returned using the `View` facade:

Tampilan juga dapat dikembalikan menggunakan fasad `View`:

    use Illuminate\Support\Facades\View;

    return View::make('greeting', ['name' => 'James']);

As you can see, the first argument passed to the `view` helper corresponds to the name of the view file in the `resources/views` directory. The second argument is an array of data that should be made available to the view. In this case, we are passing the `name` variable, which is displayed in the view using [Blade syntax](/docs/{{version}}/blade).

Seperti yang anda lihat, argumen pertama yang dioper ke helper `view` sesuai dengan nama file view di direktori `resources/views`. Argumen kedua adalah array data yang harus tersedia untuk view. Dalam hal ini, kita mengoper variabel `nama`, yang ditampilkan dalam tampilan menggunakan [sintaks Blade](/docs/{{{version}}/blade).

<a name="nested-view-directories"></a>
### Direktori View yang Bersarang

Views may also be nested within subdirectories of the `resources/views` directory. "Dot" notation may be used to reference nested views. For example, if your view is stored at `resources/views/admin/profile.blade.php`, you may return it from one of your application's routes / controllers like so:

Tampilan juga dapat disarangkan di dalam subdirektori dari direktori `resources/views`. Notasi "Dot" dapat digunakan untuk mereferensikan view bersarang. Misalnya, jika view Anda disimpan di `resources/views/admin/profile.blade.php`, Anda dapat mengembalikannya dari salah satu rute / pengontrol aplikasi Anda seperti ini:

    return view('admin.profile', $data);

> **Peringatan**  
> View directory names should not contain the `.` character.

<a name="creating-the-first-available-view"></a>
### Membuat _View_ yang Tersedia Saat Pertama Kali

Using the `View` facade's `first` method, you may create the first view that exists in a given array of views. This may be useful if your application or package allows views to be customized or overwritten:

Dengan menggunakan metode `first` dari `View` facade, Anda dapat membuat tampilan pertama yang ada dalam larik tampilan yang diberikan. Ini mungkin berguna jika aplikasi atau paket Anda memungkinkan tampilan untuk dikustomisasi atau ditimpa:

    use Illuminate\Support\Facades\View;

    return View::first(['custom.admin', 'admin'], $data);

<a name="determining-if-a-view-exists"></a>
### Penentuan Keberadaan _View_

If you need to determine if a view exists, you may use the `View` facade. The `exists` method will return `true` if the view exists:

Jika Anda perlu menentukan apakah sebuah view ada, Anda dapat menggunakan fasad `View`. Metode `exists` akan mengembalikan `true` jika view ada:

    use Illuminate\Support\Facades\View;

    if (View::exists('emails.customer')) {
        //
    }

<a name="passing-data-to-views"></a>
## Mengoper Data ke _View_

As you saw in the previous examples, you may pass an array of data to views to make that data available to the view:

Seperti yang Anda lihat dalam contoh-contoh sebelumnya, Anda dapat mengoper larik data ke view untuk membuat data itu tersedia untuk view:

    return view('greetings', ['name' => 'Victoria']);

When passing information in this manner, the data should be an array with key / value pairs. After providing data to a view, you can then access each value within your view using the data's keys, such as `<?php echo $name; ?>`.

Ketika meneruskan informasi dengan cara ini, data harus berupa larik dengan pasangan kunci/nilai. Setelah memberikan data ke tampilan, Anda kemudian dapat mengakses setiap nilai dalam tampilan Anda menggunakan kunci data, seperti `<?php echo $nama; ?>`.

As an alternative to passing a complete array of data to the `view` helper function, you may use the `with` method to add individual pieces of data to the view. The `with` method returns an instance of the view object so that you can continue chaining methods before returning the view:

Sebagai alternatif untuk mengoper array data lengkap ke fungsi pembantu `view`, anda dapat menggunakan metode `with` untuk menambahkan potongan-potongan data individual ke view. Metode `with` mengembalikan sebuah instance dari objek view sehingga Anda dapat melanjutkan metode chaining sebelum mengembalikan view:

    return view('greeting')
                ->with('name', 'Victoria')
                ->with('occupation', 'Astronaut');

<a name="sharing-data-with-all-views"></a>
### Berbagi Data dengan Semua _View_

Occasionally, you may need to share data with all views that are rendered by your application. You may do so using the `View` facade's `share` method. Typically, you should place calls to the `share` method within a service provider's `boot` method. You are free to add them to the `App\Providers\AppServiceProvider` class or generate a separate service provider to house them:

Terkadang, Anda mungkin perlu berbagi data dengan semua view yang dirender oleh aplikasi Anda. Anda dapat melakukannya dengan menggunakan metode `share` facade `View`. Biasanya, Anda harus menempatkan panggilan ke metode `share` dalam metode `boot` penyedia layanan. Anda bebas menambahkannya ke kelas `App\Providers\AppServiceProvider` atau membuat penyedia layanan terpisah untuk menampungnya:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Bootstrap any application services.
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

View composers are callbacks or class methods that are called when a view is rendered. If you have data that you want to be bound to a view each time that view is rendered, a view composer can help you organize that logic into a single location. View composers may prove particularly useful if the same view is returned by multiple routes or controllers within your application and always needs a particular piece of data.

Typically, view composers will be registered within one of your application's [service providers](/docs/{{version}}/providers). In this example, we'll assume that we have created a new `App\Providers\ViewServiceProvider` to house this logic.

We'll use the `View` facade's `composer` method to register the view composer. Laravel does not include a default directory for class based view composers, so you are free to organize them however you wish. For example, you could create an `app/View/Composers` directory to house all of your application's view composers:

Komposer tampilan adalah callback atau metode kelas yang dipanggil saat tampilan dirender. Jika Anda memiliki data yang ingin diikat ke tampilan setiap kali tampilan tersebut dirender, view composer dapat membantu Anda mengatur logika tersebut ke dalam satu lokasi. Komposer tampilan mungkin terbukti sangat berguna jika tampilan yang sama dikembalikan oleh beberapa rute atau pengontrol di dalam aplikasi Anda dan selalu membutuhkan bagian data tertentu.

Biasanya, komposer tampilan akan didaftarkan dalam salah satu [penyedia layanan] aplikasi Anda (/docs/{{version}}/providers). Dalam contoh ini, kita akan mengasumsikan bahwa kita telah membuat `App\Providers\ViewServiceProvider` baru untuk menampung logika ini.

Kita akan menggunakan metode `composer` facade `View` untuk mendaftarkan view composer. Laravel tidak menyertakan direktori default untuk komposer tampilan berbasis kelas, jadi Anda bebas mengaturnya sesuai keinginan Anda. Sebagai contoh, Anda dapat membuat direktori `app/View/Composers` untuk menampung semua view composer aplikasi Anda:

    <?php

    namespace App\Providers;

    use App\View\Composers\ProfileComposer;
    use Illuminate\Support\Facades\View;
    use Illuminate\Support\ServiceProvider;

    class ViewServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            // Using class based composers...
            View::composer('profile', ProfileComposer::class);

            // Using closure based composers...
            View::composer('dashboard', function ($view) {
                //
            });
        }
    }

> **Peringatan**  
> Remember, if you create a new service provider to contain your view composer registrations, you will need to add the service provider to the `providers` array in the `config/app.php` configuration file.

> Ingat, jika Anda membuat penyedia layanan baru untuk memuat pendaftaran komposer tampilan Anda, Anda perlu menambahkan penyedia layanan ke array `providers` di file konfigurasi `config/app.php`.

Sekarang kita telah mendaftarkan komposer, metode `compose` dari kelas `App\View\Composers\ProfileComposer` akan dieksekusi setiap kali tampilan `profil` sedang dirender. Mari kita lihat contoh kelas komposer:

    <?php

    namespace App\View\Composers;

    use App\Repositories\UserRepository;
    use Illuminate\View\View;

    class ProfileComposer
    {
        /**
         * The user repository implementation.
         *
         * @var \App\Repositories\UserRepository
         */
        protected $users;

        /**
         * Create a new profile composer.
         *
         * @param  \App\Repositories\UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Bind data to the view.
         *
         * @param  \Illuminate\View\View  $view
         * @return void
         */
        public function compose(View $view)
        {
            $view->with('count', $this->users->count());
        }
    }

As you can see, all view composers are resolved via the [service container](/docs/{{version}}/container), so you may type-hint any dependencies you need within a composer's constructor.

Seperti yang Anda lihat, semua komposer tampilan diselesaikan melalui [service container](/docs/{{version}}/container), jadi Anda dapat mengetikkan petunjuk ketergantungan apa pun yang Anda perlukan dalam konstruktor komposer.

<a name="attaching-a-composer-to-multiple-views"></a>
#### Attaching A Composer To Multiple Views

You may attach a view composer to multiple views at once by passing an array of views as the first argument to the `composer` method:

Anda dapat melampirkan view composer ke beberapa view sekaligus dengan mengoper array view sebagai argumen pertama ke metode `composer`:

    use App\Views\Composers\MultiComposer;

    View::composer(
        ['profile', 'dashboard'],
        MultiComposer::class
    );

The `composer` method also accepts the `*` character as a wildcard, allowing you to attach a composer to all views:

Metode `composer` juga menerima karakter `*` sebagai wildcard, yang memungkinkan Anda untuk melampirkan komposer ke semua tampilan:

    View::composer('*', function ($view) {
        //
    });

<a name="view-creators"></a>
### Kreator _View_

View "creators" are very similar to view composers; however, they are executed immediately after the view is instantiated instead of waiting until the view is about to render. To register a view creator, use the `creator` method:

View "creator" sangat mirip dengan view composer; namun, mereka dieksekusi segera setelah view diinstansiasi, bukannya menunggu sampai view akan dirender. Untuk mendaftarkan kreator view, gunakan metode `creator`:

    use App\View\Creators\ProfileCreator;
    use Illuminate\Support\Facades\View;

    View::creator('profile', ProfileCreator::class);

<a name="optimizing-views"></a>
## Optimasi _View_

By default, Blade template views are compiled on demand. When a request is executed that renders a view, Laravel will determine if a compiled version of the view exists. If the file exists, Laravel will then determine if the uncompiled view has been modified more recently than the compiled view. If the compiled view either does not exist, or the uncompiled view has been modified, Laravel will recompile the view.

Compiling views during the request may have a small negative impact on performance, so Laravel provides the `view:cache` Artisan command to precompile all of the views utilized by your application. For increased performance, you may wish to run this command as part of your deployment process:

Secara default, tampilan template Blade dikompilasi sesuai permintaan. Ketika permintaan dieksekusi yang merender tampilan, Laravel akan menentukan apakah versi tampilan yang dikompilasi ada. Jika file tersebut ada, Laravel kemudian akan menentukan apakah tampilan yang belum dikompilasi telah dimodifikasi lebih baru daripada tampilan yang dikompilasi. Jika tampilan yang dikompilasi tidak ada, atau tampilan yang belum dikompilasi telah dimodifikasi, Laravel akan mengkompilasi ulang tampilan tersebut.

Mengkompilasi tampilan selama permintaan mungkin memiliki dampak negatif kecil pada kinerja, sehingga Laravel menyediakan perintah Artisan `view:cache` untuk mengkompilasi sebelumnya semua tampilan yang digunakan oleh aplikasi Anda. Untuk meningkatkan kinerja, Anda mungkin ingin menjalankan perintah ini sebagai bagian dari proses penyebaran Anda:

```shell
php artisan view:cache
```

Anda bisa menggunakan perintah `view:clear` untuk menghapus _cache view_ yang telah dibuat sebelumnya:

```shell
php artisan view:clear
```
