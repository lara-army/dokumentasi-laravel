# Membundel Aset (Vite)

- [Pendahuluan](#introduction)
- [Instalasi & Pengaturan](#installation)
  - [Menginstal Node](#installing-node)
  - [Menginstal Vite dan _Plugin_ Laravel](#installing-vite-and-laravel-plugin)
  - [Mengkonfigurasi Vite](#configuring-vite)
  - [Memuat _Script_ dan _Style_](#loading-your-scripts-and-styles)
- [Menjalankan Vite](#running-vite)
- [Bekerja dengan JavaScript](#working-with-scripts)
  - [Alias](#aliases)
  - [Vue](#vue)
  - [React](#react)
  - [Inertia](#inertia)
  - [Pemrosesan URL](#url-processing)
- [Bekerja dengan _Stylesheet_](#working-with-stylesheets)
- [Bekerja dengan Blade & Rute](#working-with-blade-and-routes)
  - [Pemrosesan Aset Statis dengan Vite](#blade-processing-static-assets)
  - [Melakukan _Refresh_ saat Menyimpan](#blade-refreshing-on-save)
  - [Alias](#blade-aliases)
- [URL Dasar yang Kustom](#custom-base-urls)
- [Variabel _Environment_](#environment-variables)
- [Menonaktifkan Vite di dalam Pengujian](#disabling-vite-in-tests)
- [_Server-Side Rendering_ (SSR)](#ssr)
- [Atribut _Tag Script_ & _Style_](#script-and-style-attributes)
  - [_Nonce Content Security Policy_ (CSP)](#content-security-policy-csp-nonce)
  - [_Subresource Integrity_ (SRI)](#subresource-integrity-sri)
  - [Atribut yang Berubah-ubah](#arbitrary-attributes)
- [Kustomisasi Tingkat Lanjut](#advanced-customization)

<a name="introduction"></a>
## Pendahuluan 

[Vite](https://vitejs.dev) merupakan alat pembuat _frontend_ modern yang menyediakan kecepatan yang ekstrim untuk lingkungan pengembangan dan pembundelan kode untuk _production_. Ketika membangun aplikasi menggunakan Laravel, umumnya Anda akan menggunakan Vite untuk membundel _file_-_file_ CSS dan Javascript pada aplikasi Anda menjadi aset yang siap untuk _production_.

Laravel terintegrasi secara mulus dengan Vite dengan menyediakan _plugin_ resmi dan _directive_ Blade untuk memuat aset Anda pada lingkungan pengembangan dan _production_.

> **Note**  
> Apakah Anda menggunakan Laravel Mix? Vite telah menggantikan Laravel Mix pada instalasi Laravel yang baru. Untuk dokumentasi Mix, silakan mengunjungi laman web [Laravel Mix](https://laravel-mix.com/). Jika Anda ingin beralih ke Vite. silakan lihat [panduan migrasi](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-laravel-mix-to-vite) dari kami.

<a name="vite-or-mix"></a>
#### Memilih Antara Vite dan Laravel Mix

Sebelum beralih ke Vite, aplikasi Laravel yang baru diinstal menggunakan [Mix] (https://laravel-mix.com/), yang didukung oleh [webpack] (https://webpack.js.org/) untuk membundel aset. Vite berfokus untuk memberikan pengalaman yang lebih cepat dan lebih produktif ketika membangun aplikasi yang kaya akan JavaScript. Jika Anda mengembangkan _Single Page Application_ (SPA) dan aplikasi yang dikembangkan dengan alat seperti [Inertia](https://inertiajs.com), Vite akan menjadi perkakas yang tepat.

Vite juga berfungsi dengan baik pada aplikasi tradisional yang di-_render_ di sisi server yang memiliki "taburan" JavaScript, termasuk yang menggunakan [Livewire](https://laravel-livewire.com). Namun, Vite tidak memiliki beberapa fitur yang didukung oleh Laravel Mix, seperti kemampuan untuk menyalin aset sembarang ke dalam build yang tidak tereferensi secara langsung dalam aplikasi JavaScript Anda.

<a name="migrating-back-to-mix"></a>
#### Migrasi Kembali ke Mix

Apakah aplikasi Laravel baru Anda telah menggunakan perancah Vite namun Anda perlu kembali ke Laravel Mix dan webpack? Tidak masalah. Silakan baca [panduan resmi tentang migrasi dari Vite ke Mix](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-vite-to-laravel-mix).

<a name="installation"></a>
## Instalasi & Pengaturan

> **Note**  
> Dokumentasi berikut ini membahas cara menginstal dan mengonfigurasi _plugin_ Laravel Vite secara manual. Namun, [_starter kits_] Laravel (/docs/{{version}}/starter-kits) yang merupakan cara tercepat untuk bermain dengan Laravel dan Vite sudah menyertakan semua perancah ini.

<a name="installing-node"></a>
### Menginstal Node

Anda harus memastikan bahwa Node.js (16+) dan NPM telah terinstal sebelum menjalankan Vite dan _plugin_ Laravel.

```sh
node -v
npm -v
```

Anda dapat menginstal versi mutakhir dari Node dan NPM secara mudah dengan menggunakan penginstal grafis yang sederhana dari [laman resmi Node](https://nodejs.org/en/download/). Atau, jika Anda menggunakan [Laravel Sail](https://laravel.com/docs/{{version}}/sail), Anda dapat memanggil Node dan NPM langsung dari Sail:

```sh
./vendor/bin/sail node -v
./vendor/bin/sail npm -v
```

<a name="installing-vite-and-laravel-plugin"></a>
### Menginstal Vite dan _Plugin_ Laravel

Pada instalasi Laravel yang _fresh_, Anda akan menemukan _file_ `package.json` pada _root_ dari struktur direktori aplikasi Anda. _File_ `package.json` yang _default_ telah mencakup semua yang Anda perlukan untuk mulai menggunakan Vite dan _plugin_ laravel. Anda dapat menginstal dependensi _frontend_ untuk aplikasi Anda melalui NPM:

```sh
npm install
```

<a name="configuring-vite"></a>
### Mengkonfigurasi Vite

Vite dikonfigurasi pada _file_ `vite.config.js` pada _root_ proyek Anda. Anda bebas melakukan kustomisasi pada _file_ tersebut sesuai dengan kebutuhan Anda, dan Anda juga dapat menginstal _plugin_-_plugin_ lain yang dibutuhkan oleh aplikasi Anda, seperti `@vitejs/plugin-vue` atau `@vitejs/plugin-react`.

_Plugin_ Vite Laravel mewajibkan Anda untuk menentukan beberapa titik masuk (_entry points_) untuk aplikasi Anda. Titik masuk tersebut dapat berupa _file_ JavaScript atau CSS, juga termasuk bahasa praproses seperti TypeScript, JSX, TSX, dan Sass.

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css',
            'resources/js/app.js',
        ]),
    ],
});
```

Jika Anda sedang membangun sebuah SPA, termasuk aplikasi yang dibangun menggunakan Inertia, Vite dapat berfungsi maksimal tanpa titik masuk CSS:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css', // [tl! remove]
            'resources/js/app.js',
        ]),
    ],
});
```

Sebagai gantinya, Anda harus mengimpor CSS melalui JavaScript. Biasanya, hal ini akan dilakukan pada _file_ `resources/js/app.js` milik aplikasi Anda:

```js
import './bootstrap';
import '../css/app.css'; // [tl! add]
```

_Plugin_ Laravel juga mendukung titik masuk yang jamak dan opsi konfigurasi tingkat lanjut seperti [titik masuk SSR](#ssr).

<a name="working-with-a-secure-development-server"></a>
#### Bekerja dengan Server Pengembangan yang Aman

Jika server web untuk pengembangan lokal Anda menyajikan aplikasi anda melalui HTTPS, Anda mungkin akan mengalami masalah untuk menghubungkannya dengan server pengembangan Vite.

Jika Anda menggunakan [Laravel Valet](/docs/{{version}}/valet) untuk pengembangan lokal dan harus menjalankan [perintah yang aman](/docs/{{version}}/valet#securing-sites) untuk aplikasi Anda, Anda dapat mengonfigurasi server pengembangan Vite untuk menggunakan sertifikat TLS yang dihasilkan oleh Valet secara otomatis:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            valetTls: 'my-app.test', // [tl! add]
        }),
    ],
});
```

Ketika menggunakan server web yang lain, Anda harus membuat sertifikat kepercayaan (_trusted certificate_) dan mengonfigurasi Vite secara manual untuk menggunakan sertifikat-sertifikat tersebut: 

```js
// ...
import fs from 'fs'; // [tl! add]

const host = 'my-app.test'; // [tl! add]

export default defineConfig({
    // ...
    server: { // [tl! add]
        host, // [tl! add]
        hmr: { host }, // [tl! add]
        https: { // [tl! add]
            key: fs.readFileSync(`/path/to/${host}.key`), // [tl! add]
            cert: fs.readFileSync(`/path/to/${host}.crt`), // [tl! add]
        }, // [tl! add]
    }, // [tl! add]
});
```

Jika tidak memungkinkan untuk membuat sertifikat kepercayaan untuk sistem Anda. Anda dapat menginstal dan mengonfigurasi [_plugin_ `@vitejs/plugin-basic-ssl`](https://github.com/vitejs/vite-plugin-basic-ssl). Ketika menggunakan sertifikat kepercayaan yang imitasi, anda perlu menerima peringatan sertifikat untuk server pengembangan Vite pada _browser_ Anda yang disertai dengan pranala "lokal" pada konsol Anda ketika menjalankan perintah `npm run dev`.

<a name="loading-your-scripts-and-styles"></a>
### Memuat _Script_ dan _Style_ milik Anda

Dengan titik masuk yang telah terkonfigurasi pada Vite, Anda hanya perlu mereferensikan _file_-_file_ tersebut dengan _directive_ `@vite()` milik Blade di dalam elemen `<head>` pada _template_ yang digunakan sebagai pondasi untuk tampilan pada aplikasi Anda:

```blade
<!doctype html>
<head>
    {{-- ... --}}

    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
```

Jika Anda telah mengimpor CSS via JavaScript, Anda hanya perlu untuk menambahkan titik masuk untuk JavaScript saja:

```blade
<!doctype html>
<head>
    {{-- ... --}}

    @vite('resources/js/app.js')
</head>
```

_Directive_ `@vite` secara otomatis akan mendeteksi server pengembangan Vite dan menginjeksikan klien Vite untuk mengaktifkan Hot Module Replacement. Pada mode _build_, _directive_ tersebut akan memuat aset-aset Anda yang sudah terkompilasi dan diberi versi, termasuk semua CSS yang telah Anda impor.

Jika diperlukan, Anda juga dapat menentukan lokasi tempat peletakan aset-aset yang sudah dikompilasi saat memanggil _directive_ `@vite`:

```blade
<!doctype html>
<head>
    {{-- Lokasi peletakan aset merupakan lokasi relatif dari direktori publik. --}}

    @vite('resources/js/app.js', 'vendor/courier/build')
</head>
```

<a name="running-vite"></a>
## Menjalankan Vite

Terdapat dua cara untuk menjalankan Vite. Anda dapat menjalankan server pengembangan dengan perintah `dev`, yang sangat berguna saat mengembangkan aplikasi secara lokal. Server pengembangan akan mendeteksi perubahan _file_ yang terjadi secara otomatis dan perubahan tersebut akan terefleksi pada jendela _browser_ yang sedang terbuka secara instan.

atau, jalankan perintah `build` yang akan memberikan bundel beserta versi dari aset-aset yang terdapat pada aplikasi anda sehingga siap untuk di-_deploy_ pada _production_.

```shell
# Menjalankan server pengembangan Vite...
npm run dev

# Membentuk dan memberi versi aset-aset untuk production...
npm run build
```

<a name="working-with-scripts"></a>
## Bekerja dengan JavaScript

<a name="aliases"></a>
### Alias

Secara _default_, _plugin_ Laravel menyediakan alias yang umum untuk membantu Anda untuk memulai dan mengimpor aset-aset pada aplikasi Anda secara mudah:

```js
{
    '@' => '/resources/js'
}
```

Anda dapat menimpa alias `'@'` dengan menambahkan alias Anda sendiri pada _file_ konfigurasi `vite.config.js`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel(['resources/ts/app.tsx']),
    ],
    resolve: {
        alias: {
            '@': '/resources/ts',
        },
    },
});
```

<a name="vue"></a>
### Vue

Jika Anda ingin membangun _front-end_ Anda dengan menggunakan _framework_ [Vue](https://vuejs.org/), maka Anda juga harus menginstal _plugin_ `@vitejs/plugin-vue`:

```sh
npm install --save-dev @vitejs/plugin-vue
```

Anda kemudian dapat memasukkan _plugin_ tersebut di dalam _file_ konfigurasi `vite.config.js`. Ada beberapa opsi tambahan yang Anda perlukan saat menggunakan _plugin_ Vue dengan Laravel:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.js']),
        vue({
            template: {
                transformAssetUrls: {
                    // Plugin Vue akan menulis ulang URL aset, ketika direferensikan
                    // pada Single File Component, untuk mengarah ke server web
                    // Laravel. Atur opsi ini menjadi `null` memungkinkan
                    // plugin Laravel untuk menulis ulang URL aset untuk mengarah ke
                    // server Vite sebagai gantinya.
                    base: null,

                    // Plugin Vue akan mem-parsing URL absolut dan memperlakukannya
                    // sebagai jalur absolut ke file pada cakram penyimpanan. Atur
                    // opsi ini menjadi `false` untuk membiarkan URL absolut tidak tersentuh sehingga mereka
                    // dapat dapat merujuk aset yang berada di direktori publik seperti yang diharapkan.
                    includeAbsolute: false,
                },
            },
        }),
    ],
});
```

> **Note**  
> [Perlengkapan untuk pemula](/docs/{{version}}/starter-kits) milik Laravel sudah disertai dengan konfigurasi Laravel, Vue, dan Vite yang efektif. Periksa [Laravel Breeze](/docs/{{version}}/starter-kits#breeze-and-inertia) yang merupakan cara tercepat untuk memulai mengembangkan aplikasi dengan Laravel, Vue dan Vite.

<a name="react"></a>
### React

Jika Anda ingin membangun _front-end_ Anda menggunakan _framework_ [React](https://reactjs.org/), maka Anda juga harus menginstal _plugin_ `@vitejs/plugin-react`:

```sh
npm install --save-dev @vitejs/plugin-react
```

Anda kemudian dapat menyertakan plugin tersebut di dalam _file_ konfigurasi `vite.config.js`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.jsx']),
        react(),
    ],
});
```

Anda perlu memastikan bahwa semua _file_ yang mengandung JSX telah memiliki eksistensi `.jsx` atau `.tsx`. Jangan lupa juga untuk memperbaharui titik masuk jika diperlukan, seperti yang telah dijelaskan [di atas](#configuring-vite).

Anda juga perlu untuk memasukkan _directive_ Blade tambahan `@viteReactRefresh` berdampingan dengan _directive_ `@vite` yang sudah ada.

```blade
@viteReactRefresh
@vite('resources/js/app.jsx')
```

_Directive_ `@viteReactRefresh` harus dipanggil sebelum _directive_ `@vite`.

> **Note**  
> [Perlengkapan untuk pemula](/docs/{{version}}/starter-kits) milik Laravel sudah disertai dengan konfigurasi Laravel, React, dan Vite yang efektif. Periksa [Laravel Breeze](/docs/{{version}}/starter-kits#breeze-and-inertia) yang merupakan cara tercepat untuk memulai mengembangkan aplikasi dengan Laravel, Vue dan Vite.

<a name="inertia"></a>
### Inertia

_Plugin_ Laravel Vite menyediakan fungsi `resolvePageComponent` yang nyaman untuk membantu Anda melakukan _resolve_ pada komponen halaman Inertia Anda. Di Bawah ini merupakan contoh penggunaan _helper_ pada Vue 3; bagaimanapun, Anda juga dapat menggunakan fungsi ini pada _framework_ yang lain misalnya React:

```js
import { createApp, h } from 'vue';
import { createInertiaApp } from '@inertiajs/vue3';
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';

createInertiaApp({
  resolve: (name) => resolvePageComponent(`./Pages/${name}.vue`, import.meta.glob('./Pages/**/*.vue')),
  setup({ el, App, props, plugin }) {
    return createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  },
});
```

> **Note**  
> [Perlengkapan untuk pemula](/docs/{{version}}/starter-kits) milik Laravel sudah disertai dengan konfigurasi Laravel, Inertia, dan Vite yang efektif. Periksa [Laravel Breeze](/docs/{{version}}/starter-kits#breeze-and-inertia) yang merupakan cara tercepat untuk memulai mengembangkan aplikasi dengan Laravel, Inertia dan Vite.

<a name="url-processing"></a>
### Pemrosesan URL

Ketika menggunakan Vite kemudian mereferensikan aset-aset pada HTML, CSS, atau JS milik aplikasi Anda, terdapat beberapa peringatan yang perlu diperhatikan. Pertama, jika Anda mereferensikan aset menggunakan _path_ absolut, Vite tidak akan memasukkan aset tersebut ke dalam hasil _build_; oleh karena itu, Anda harus memastikan bahwa aset tersebut tersedia di dalam direktori publik Anda.

Ketika mereferensikan aset menggunakan _path_ yang relatif, Anda harus ingat bahwa _path_ tersebut bersifat relatif terhadap lokasi _file_ yang mereferensikannya. Aset-aset yang direferensikan menggunakan _path_ relatif akan ditulis ulang, diberi versi, dan dibundel oleh Vite.

Pertimbangan struktur direktori proyek:

```nothing
public/
  taylor.png
resources/
  js/
    Pages/
      Welcome.vue
  images/
    abigail.png
```

Contoh berikut akan mendemonstrasikan bagaimana Vite memperlakukan URL relatif dan absolut:

```html
<!-- Aset ini tidak diurus oleh Vite dan tidak akan disertakan ke dalam hasil build -->
<img src="/taylor.png">

<!-- Aset ini akan ditulis ulang, diberi versi, dan dibundel oleh Vite -->
<img src="../../images/abigail.png">
```

<a name="working-with-stylesheets"></a>
## Bekerja dengan _Stylesheet_

Anda dapat mempelajari lebih lanjut tentang dukungan CSS milik Vite pada [dokumentasi Vite](https://vitejs.dev/guide/features.html#css). Jika Anda menggunakan _plugin_ PostCSS seperti [Tailwind](https://tailwindcss.com), Anda dapat membuat _file_ `postcss.config.js` di dalam _root_ proyek Anda dan Vite akan menerapkan pengaturan tersebut secara otomatis:

```js
module.exports = {
    plugins: {
        tailwindcss: {},
        autoprefixer: {},
    },
};
```

<a name="working-with-blade-and-routes"></a>
## Bekerja dengan Blade & Rute

<a name="blade-processing-static-assets"></a>
### Memproses Aset Statis dengan Vite

Ketika mereferensikan aset-aset pada JavaScript atau CSS milik Anda, Vite memproses dan memberi versi pada aset tersebut secara otomatis. Selain itu, ketika membangun aplikasi berbasis Blade, Vite juga dapat memproses dan membuat versi aset statis yang Anda rujuk pada _template_ Blade.

Namun, untuk mencapai hal ini, Anda perlu membuat Vite untuk mengawasi aset Anda dengan cara mengimpor aset statis tersebut ke dalam titik masuk aplikasi. Sebagai contoh, jika Anda ingin memproses dan memberi versi semua gambar yang disimpan di dalam `resources/images` dan semua fon yang disimpan di `resources/fonts`, Anda harus menambahkan direktori-direktori tersebut ke dalam titik masuk `resources/js/app.js` aplikasi Anda seperti contoh di bawah ini:

```js
import.meta.glob([
  '../images/**',
  '../fonts/**',
]);
```

Aset-aset tersebut akan diproses oleh Vite ketika menjalankan `npm run build`. Kemudian Anda dapat mereferensikan aset-aset ini pada _template_ Blade menggunakan metode `Vite::asset`, yang akan mengembalikan URL berversi untuk aset yang sudah ditambahkan:

```blade
<img src="{{ Vite::asset('resources/images/logo.png') }}">
```

<a name="blade-refreshing-on-save"></a>
### Me-_refresh_ saat Disimpan

Ketika aplikasi Anda dibuat menggunakan _server-side rendering_ yang tradisional dengan Blade, Vite dapat meningkatkan alur kerja pengembangan Anda dengan cara me-_refresh browser_ Anda secara otomatis ketika Anda membuat perubahan pada _file view_ milik aplikasi Anda. Untuk memulai, Anda cukup menambahkan opsi `refresh` dengan nilai `true`.

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: true,
        }),
    ],
});
```

Ketika opsi `refresh` bernilai `true`, melakukan penyimpanan perubahan pada _file_-_file_ yang terdapat pada direktori yang terdaftar akan memicu _browser_ untuk me-_refresh_ halaman secara penuh saat `npm run dev` sedang berjalan:

- `app/View/Components/**`
- `lang/**`
- `resources/lang/**`
- `resources/views/**`
- `routes/**`

Mengamati direktori `routes/**` akan berguna jika Anda menggunakan [Ziggy](https://github.com/tighten/ziggy) untuk membuat tautan rute pada _frontend_ milik aplikasi Anda.

Jika _path_-_path_ di atas tidak sesuai dengan  Anda, Anda dapat menentukan daftar _path_ yang lain untuk diawasi:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: ['resources/views/**'],
        }),
    ],
});
```

Di balik layar, _plugin_ Laravel Vite menggunakan _package_ [`vite-plugin-full-reload`](https://github.com/ElMassimo/vite-plugin-full-reload), yang menawarkan beberapa konfigurasi tingkat lanjut untuk menyempurnakan perilaku dari fitur tersebut. Jika Anda memerlukan kustomisasi pada tingkat ini, Anda dapat mendefinisikan sebuah `config`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: [{
                paths: ['path/to/watch/**'],
                config: { delay: 300 }
            }],
        }),
    ],
});
```

<a name="blade-aliases"></a>
### Alias

Sudah menjadi hal yang umum dalam aplikasi JavaScript untuk [membuat alias](#aliases) yang mereferensikan direktori secara berkala. Namun, Anda juga dapat membuat alias yang dapat digunakan pada Blade dengan menggunakan metode `macro` pada kelas `Illuminate\Support\Facades\Vite`. Biasanya, "_macro_" harus didefinisikan dalam metode `boot` pada [_service provider_](/docs/{{version}}/providers):

    /**
     * Melakukan bootstrap layanan aplikasi.
     *
     * @return void
     */
    public function boot()
    {
        Vite::macro('gambar', fn ($aset) => $this->asset("resources/images/{$aset}"));
    }

Setelah _macro_ didefinisikan, _macro_ dapat dipanggil pada _template_ Anda. Sebagai contoh, kita dapat menggunakan _macro_ `gambar` yang sudah didefinisikan di atas untuk mereferensikan sebuah aset yang berlokasi di `resources/images/logo.png`:

```blade
<img src="{{ Vite::gambar('logo.png') }}" alt="Logo Laravel">
```

<a name="custom-base-urls"></a>
## URL Dasar yang Kustom

Jika aset-aset yang telah dikompilasi oleh Vite di-_deploy_ pada domain yang terpisah dengan aplikasi Anda, seperti melalui CDN, Anda harus mendefinisikan variabel _environment_ `ASEET_URL` pada _file_ `.env` milik aplikasi Anda:

```env
ASSET_URL=https://cdn.example.com
```

Setelah mengkonfigurasi URL aset, semua URL yang ditulis ulang ke aset-aset Anda akan ditambahkan prefiks yakni nilai `ASSET_URL` yang telah dikonfigurasi:

```nothing
https://cdn.example.com/build/assets/app.9dce8d17.js
```

Perlu diingat bahwa [URL absolut tidak akan ditulis ulang oleh Vite](#url-processing), sehingga URL-URL tersebut tidak akan diberi prefiks.

<a name="environment-variables"></a>
## Variabel _Environment_

Anda dapat menginjeksikan variabel _environment_ ke dalam JavaScript dengan menambahkan prefiks `VITE_` pada _file_ `.env` milik aplikasi:

```env
VITE_SENTRY_DSN_PUBLIC=http://example.com
```

Anda dapat mengakses variabel-variabel yang telah diinjeksikan sebelumnya melalui objek `import.meta.env`:

```js
import.meta.env.VITE_SENTRY_DSN_PUBLIC
```

<a name="disabling-vite-in-tests"></a>
## Menonaktifkan Vite di dalam Pengujian

Integrasi Vite milik Laravel akan mencoba untuk me-_resolve_ aset-aset Anda ketika melakukan pengujian, yang mengharuskan Anda untuk menjalankan server pengembangan Vite atau melakukan _build_ terhadap aset Anda.

Jika Anda lebih memilih untuk tidak menggunakan Vite selama pengujian, Anda dapat memanggil metode `withoutVite`, yang tersedia untuk semua kelas pengujian yang melakukan _extend_ terhadap kelas `TestCase` milik Laravel:

```php
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_without_vite_example()
    {
        $this->withoutVite();

        // ...
    }
}
```

Jika Anda ingin menonaktifkan Vite untuk semua pengujian, Anda dapat memanggil metode `withoutVite` di dalam metode `setUp` pada kelas `TestCase`:

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    use CreatesApplication;

    protected function setUp(): void// [tl! add:start]
    {
        parent::setUp();

        $this->withoutVite();
    }// [tl! add:end]
}
```

<a name="ssr"></a>
## _Server-Side Rendering_ (SSR)

_Plugin_ Laravel Vite memudahkan Anda untuk menyiapkan _server-side rendering_ dengan Vite. Untuk memulai, buatlah titik masuk SSR di `resources/js/ssr.js` dan tentukan titik masuknya dengan mengoper opsi konfigurasi ke _plugin_ Laravel:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            ssr: 'resources/js/ssr.js',
        }),
    ],
});
```

Untuk memastikan bahwa Anda tidak lupa melakukan _build_ ulang untuk titik masuk SSR, kami sarankan untuk menambahkan skrip "build" di `package.json` milik aplikasi Anda untuk mem-_build_ SSR:

```json
"scripts": {
     "dev": "vite",
     "build": "vite build" // [tl! remove]
     "build": "vite build && vite build --ssr" // [tl! add]
}
```

Kemudian, untuk melakukan _build_ dan menjalankan server SSR, Anda dapat menggunakan perintah:

```sh
npm run build
node bootstrap/ssr/ssr.mjs
```

> **Note**  
> [Perlengkapan untuk pemula](/docs/{{version}}/starter-kits) milik Laravel sudah disertai dengan konfigurasi Laravel, SSR Inertia, dan Vite. Periksa [Laravel Breeze](/docs/{{version}}/starter-kits#breeze-and-inertia) yang merupakan cara tercepat untuk memulai mengembangkan aplikasi dengan Laravel, SSR Inertia dan Vite.

<a name="script-and-style-attributes"></a>
## Atribut pada _Tag Script_ & _Style_

<a name="content-security-policy-csp-nonce"></a>
### _Nonce Content Security Policy_ (CSP)

Jika Anda ingin menyertakan [atribut `nonce` attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce) pada _tag script_ dan _style_ Anda sebagai bagian dari [_Content Security Policy_](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP), Anda dapat menghasilkan atau menentukan sebuah _nonce_ menggunakan metode `useCspNonce` pada sebuah [_middleware_](/docs/{{version}}/middleware) kustom:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Support\Facades\Vite;

class AddContentSecurityPolicyHeaders
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
        Vite::useCspNonce();

        return $next($request)->withHeaders([
            'Content-Security-Policy' => "script-src 'nonce-".Vite::cspNonce()."'",
        ]);
    }
}
```

Setelah memanggil metode `useCspNonce`, Laravel akan menyertakan atribut `nonce` secara otomatis pada semua _tag script_ dan _style_ yang dihasilkan.

Jika Anda perlu menentukan nonce di tempat lain, termasuk [_directive `@route` Ziggy](https://github.com/tighten/ziggy#using-routes-with-a-content-security-policy) yang disertakan pada [perlengkapan pemula](/docs/{{version}}/starter-kits) milik Laravel, Anda bisa mengambilnya dengan menggunakan metode `cspNonce`:

```blade
@routes(nonce: Vite::cspNonce())
```

Jika Anda telah memiliki sebuah _nonce_ dan Anda ingin menginstruksikan Laravel untuk menggunakannya, Anda dapat mengoper _nonce_ tersebut pada metode `useCspNonce`:

```php
Vite::useCspNonce($nonce);
```

<a name="subresource-integrity-sri"></a>
### _Subresource Integrity_ (SRI)

Jika manifes Vite Anda disertai dengan _hash_ `integrity` untuk aset Anda, Laravel akan menambahkan atribut `integrity` secara otomatis pada semua _tag script_ dan _style_ yang dihasilkan untuk memberlakukan [_Subresource Integrity_](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity). Secara _default_, Vite tidak menyertakan _hash_ `integrity` dalam manifesnya, tetapi Anda dapat mengaktifkannya dengan menginstal _plugin_ NPM [`vite-plugin-manifest-uri`](https://www.npmjs.com/package/vite-plugin-manifest-sri):

```shell
npm install --save-dev vite-plugin-manifest-sri
```

Anda dapat mengaktifkan _plugin_ tersebut pada _file_ `vite.config.js`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import manifestSRI from 'vite-plugin-manifest-sri';// [tl! add]

export default defineConfig({
    plugins: [
        laravel({
            // ...
        }),
        manifestSRI(),// [tl! add]
    ],
});
```

Jika diperlukan, Anda juga dapat melakukan kustomisasi kunci manifes di mana _hash integrity_ dapat ditemukan:

```php
use Illuminate\Support\Facades\Vite;

Vite::useIntegrityKey('kunci-integrity-kustom');
```

Jika Anda ingin menonaktifkan deteksi otomatis ini sepenuhnya, Anda dapat mengoper nilai `false` ke metode `useIntegrityKey`:

```php
Vite::useIntegrityKey(false);
```

<a name="arbitrary-attributes"></a>
### Atribut yang Berubah-ubah

Jika Anda perlu menyertakan atribut tambahan pada _tag script_ dan _style_ Anda, seperti atribut [`data-turbo-track`](https://turbo.hotwired.dev/handbook/drive#reloading-when-assets-change), Anda melakukannya menggunakan metode `useScriptTagAttributes` dan `useStyleTagAttributes`. Biasanya, metode ini harus dipanggil dari sebuah [_service provider_](/docs/{{version}}/providers):

```php
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes([
    'data-turbo-track' => 'reload', // Menentukan nilai untuk atribut...
    'async' => true, // Menentukan sebuah atribut tanpa nilai...
    'integrity' => false, // Mengecualikan atribut yang seharusnya disertakan...
]);

Vite::useStyleTagAttributes([
    'data-turbo-track' => 'reload',
]);
```

Jika Anda perlu menambahkan atribut secara kondisional, Anda dapat memberikan _callback_ yang akan menerima _path_ sumber aset, URL-nya, potongan manifes, dan seluruh manifes:

```php
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $src === 'resources/js/app.js' ? 'reload' : false,
]);

Vite::useStyleTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $chunk && $chunk['isEntry'] ? 'reload' : false,
]);
```

> **Warning**  
> Argumen `$chunk` and `$manifest` akan berisi `null` ketika server pengembangan Vite dijalankan.

<a name="advanced-customization"></a>
## Kustomisasi Tingkat Lanjut

_Plugin_ Vite Laravel disertai dengan konvensi yang masuk akal yang seharusnya dapat digunakan untuk sebagian besar aplikasi; namun, terkadang Anda mungkin perlu menyesuaikan perilaku Vite. Untuk mengaktifkan opsi kustomisasi tambahan, kami menawarkan metode dan opsi berikut yang dapat digunakan sebagai pengganti _directive_ `@vite` pada Blade:

```blade
<!doctype html>
<head>
    {{-- ... --}}

    {{
        Vite::useHotFile(storage_path('vite.hot')) // Mengkustomisasi file "hot"...
            ->useBuildDirectory('bundle') // Mengkustomisasi direktori build...
            ->useManifestFilename('assets.json') // Mengkustomisasi nama file manifes...
            ->withEntryPoints(['resources/js/app.js']) // Mendefinisikan titik masuk...
    }}
</head>
```

Di dalam berkas `vite.config.js`, Anda harus menentukan konfigurasi yang sama:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            hotFile: 'storage/vite.hot', // Mengkustomisasi file "hot"...
            buildDirectory: 'bundle', // Mengkustomisasi direktori build...
            input: ['resources/js/app.js'], // Mendefinisikan titik masuk...
        }),
    ],
    build: {
      manifest: 'assets.json', // Mengkustomisasi nama file manifes...
    },
});
```
