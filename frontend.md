# _Frontend_

- [Pengantar](#introduction)
- [Menggunakan PHP](#using-php)
    - [PHP and Blade](#php-and-blade)
    - [Livewire](#livewire)
    - [Kit Pemula PHP](#php-starter-kits)
- [Menggunakan Vue / React](#using-vue-react)
    - [Inertia](#inertia)
    - [Starter Kits](#inertia-starter-kits)
    - [Kit Pemula Inertia](#inertia-starter-kits)
- [_Bundling assets_](#bundling-assets)

<a id="introduction" name="introduction"></a>
## Pengantar

Laravel adalah _framework_ _backend_ yang menyediakan semua fitur yang Anda perlukan untuk membangun aplikasi web modern, seperti [_route_](/docs/{{version}}/routing), [validasi](/docs/{{version}}/validasi), [_cache_](/docs/{{version}}/cache), [_queues_](/docs/{{version}}/queues), [penyimpanan _file_](/docs/{{version}}/filesystem ), dan masih banyak lagi. Namun, kami yakin penting untuk menawarkan pengalaman full-stack yang indah kepada developer, termasuk pendekatan yang andal untuk membangun aplikasi _frontend_ anda.

Ada dua cara utama untuk menangani pengembangan _frontend_ saat membangun aplikasi dengan Laravel, dan pendekatan mana yang Anda pilih yang telah ditentukan oleh anda apakah Anda ingin membangun _frontend_ dengan memanfaatkan PHP atau dengan menggunakan _framework_ JavaScript seperti Vue dan React. Kami akan membahas kedua opsi tersebut di bawah ini sehingga Anda dapat membuat keputusan berdasarkan informasi mengenai pendekatan terbaik untuk pengembangan _frontend_ untuk aplikasi Anda.

<a name="using-php"></a>
## Menggunakan PHP

<a name="php-and-blade"></a>
### PHP and Blade

Di masa lalu, sebagian besar aplikasi PHP menampilkan HTML ke browser menggunakan template HTML sederhana **diselingi** dengan pernyataan PHP `echo` yang menampilkan data yang diambil dari database selama permintaan:

```php
<div>
    <?php foreach ($users as $user): ?>
        Hello, <?php echo $user->name; ?> <br />
    <?php endforeach; ?>
</div>
```

Di Laravel, pendekatan menampilkan HTML ini masih dapat dicapai dengan menggunakan [_views_](/docs/{{version}}/views) dan [_Blade_](/docs/{{version}}/blade). Blade adalah bahasa templating yang sangat ringan yang menyediakan sintaks pendek yang nyaman untuk menampilkan data, mengulangi data, dan banyak lagi:

```php
<div>
    @foreach ($users as $user)
        Hello, {{ $user->name }} <br />
    @endforeach
</div>
```

Saat membuat aplikasi dengan cara ini, pengiriman formulir dan interaksi halaman lainnya biasanya menerima dokumen HTML yang sama sekali baru dari server dan seluruh halaman dirender ulang oleh browser. Bahkan saat ini, banyak aplikasi mungkin sangat cocok untuk membangun _frontend_ mereka dengan cara ini menggunakan template _Blade_ sederhana.

<a name="growing-expectations" id="growing-expectations"></a>
#### Tumbuhnya Harapan

Namun, karena ekspektasi pengguna terkait aplikasi web telah matang, banyak developer menemukan kebutuhan untuk membangun frontend yang lebih dinamis dengan interaksi yang terasa lebih halus. Mengingat hal ini, beberapa developer memilih untuk mulai membangun frontend aplikasi mereka menggunakan _framework_ JavaScript seperti Vue dan React.

Lainnya, lebih memilih untuk tetap menggunakan bahasa backend yang mereka sukai, telah mengembangkan solusi yang memungkinkan konstruksi UI aplikasi web modern sambil tetap menggunakan bahasa backend pilihan mereka. Misalnya, di ekosistem [_Rails_](https://rubyonrails.org/), hal ini telah mendorong pembuatan library seperti [_Turbo_](https://turbo.hotwired.dev/) [_Hotwire_](https://hotwired.dev/), dan [_Stimulus_](https://stimulus.hotwired.dev/).

Dalam ekosistem Laravel, kebutuhan untuk membuat frontend yang modern dan dinamis terutama dengan menggunakan PHP telah menyebabkan terciptanya [Laravel Livewire](https://laravel-livewire.com) dan [Alpine.js](https://alpinejs.dev/).

<a name="livewire"></a>
### Livewire

[Laravel Livewire](https://laravel-livewire.com) _framework_ untuk membangun frontend bertenaga Laravel yang terasa dinamis, modern, dan hidup seperti frontend yang dibuat dengan _framework_ JavaScript modern seperti Vue dan React.

Saat menggunakan Livewire, Anda akan membuat "komponen" Livewire yang merender bagian terpisah dari UI Anda dan menampilkan metode dan data yang dapat dipanggil dan berinteraksi dari frontend aplikasi Anda. Misalnya, komponen "Penghitung" sederhana mungkin terlihat seperti berikut:

```php
<?php

namespace App\Http\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public $count = 0;

    public function increment()
    {
        $this->count++;
    }

    public function render()
    {
        return view('livewire.counter');
    }
}
```

Dan, _template_ penghitung yang sesuai akan ditulis seperti ini:

```html
<div>
    <button wire:click="increment">+</button>
    <h1>{{ $count }}</h1>
</div>
```

Seperti yang Anda lihat, Livewire memungkinkan Anda menulis atribut HTML baru seperti `wire:click` yang menghubungkan frontend dan backend aplikasi Laravel Anda. Selain itu, Anda dapat _merender_ status komponen Anda saat ini menggunakan ekspresi Blade sederhana.

Bagi banyak orang, Livewire telah merevolusi pengembangan frontend dengan Laravel, memungkinkan mereka untuk tetap berada dalam kenyamanan Laravel sambil membangun aplikasi web yang modern dan dinamis. Biasanya, developer yang menggunakan Livewire juga akan memanfaatkan [Alpine.js](https://alpinejs.dev/) untuk "memercikan" JavaScript ke frontend mereka hanya jika diperlukan, misalnya untuk _merender_ jendela dialog.

Jika Anda baru menggunakan Laravel, sebaiknya pahami penggunaan dasar [views](/docs/{{version}}/views) dan [Blade](/docs/{{version}}/blade). Kemudian, lihat [dokumentasi Laravel Livewire](https://laravel-livewire.com/docs) resmi untuk mempelajari cara membawa aplikasi Anda ke level selanjutnya dengan komponen Livewire interaktif.

<a name="php-starter-kits"></a>
### Kit Pemula PHP

Jika Anda ingin membuat frontend menggunakan PHP dan Livewire, Anda dapat memanfaatkan [starter kit](/docs/{{version}}/starter-kits) Breeze atau Jetstream kami untuk memulai pengembangan aplikasi Anda. Kedua starter kit ini menyusun alur autentikasi backend dan frontend aplikasi Anda menggunakan [Blade](/docs/{{version}}/blade) dan [Tailwind](https://tailwindcss.com) sehingga Anda dapat langsung mulai membangun ide besar berikutnya.

<a name="using-vue-react"></a>
## Menggunakan Vue / React

Meskipun dimungkinkan untuk membangun frontend modern menggunakan Laravel dan Livewire, banyak pengembang masih lebih memilih untuk memanfaatkan kekuatan kerangka kerja JavaScript seperti Vue atau React. Hal ini memungkinkan para pengembang untuk memanfaatkan ekosistem yang kaya akan paket dan alat JavaScript yang tersedia melalui NPM.

Namun, tanpa alat tambahan, memasangkan Laravel dengan Vue atau React akan membuat kita harus menyelesaikan berbagai masalah rumit seperti rute sisi klien, hidrasi data, dan autentikasi. Rute sisi klien sering disederhanakan dengan menggunakan _framework_ Vue / React yang memiliki opini seperti [Nuxt](https://nuxtjs.org/) dan [Next](https://nextjs.org/); namun, hidrasi dan autentikasi data tetap menjadi masalah yang rumit dan tidak praktis untuk dipecahkan saat memasangkan kerangka kerja backend seperti Laravel dengan kerangka kerja frontend ini.

Selain itu, pengembang harus mengelola dua repositori kode yang terpisah, dan sering kali harus mengoordinasikan pemeliharaan, rilis, dan penerapan di kedua repositori tersebut. Meskipun masalah-masalah ini bukannya tidak dapat diatasi, kami tidak percaya bahwa ini adalah cara yang produktif atau menyenangkan untuk mengembangkan aplikasi.

<a name="inertia"></a>
### Inertia

Untungnya, Laravel menawarkan yang terbaik dari kedua dunia. [Inertia] (https://inertiajs.com) menjembatani celah antara aplikasi Laravel Anda dan frontend Vue atau React modern Anda, memungkinkan Anda untuk membangun frontend modern yang lengkap menggunakan Vue atau React sambil memanfaatkan rute dan pengontrol Laravel untuk perutean, hidrasi data, dan otentikasi - semuanya dalam satu repositori kode. Dengan pendekatan ini, Anda dapat menikmati kekuatan penuh dari Laravel dan Vue / React tanpa melumpuhkan kemampuan salah satu dari keduanya.

Setelah menginstal Inertia ke dalam aplikasi Laravel Anda, Anda akan menulis rute dan controller seperti biasa. Namun, alih-alih mengembalikan templat Blade dari controller Anda, Anda akan mengembalikan halaman Inertia:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\User;
use Inertia\Inertia;

class UserController extends Controller
{
    /**
     * Show the profile for a given user.
     *
     * @param  int  $id
     * @return \Inertia\Response
     */
    public function show($id)
    {
        return Inertia::render('Users/Profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

Halaman Inertia berhubungan dengan komponen Vue atau React, biasanya disimpan di dalam direktori `resources/js/Pages` pada aplikasi Anda. Data yang diberikan ke halaman melalui metode `Inertia::render` akan digunakan untuk mengisi "props" dari komponen halaman:

```js
<script setup>
import Layout from '@/Layouts/Authenticated.vue';
import { Head } from '@inertiajs/inertia-vue3';

const props = defineProps(['user']);
</script>

<template>
    <Head title="User Profile" />

    <Layout>
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                Profile
            </h2>
        </template>

        <div class="py-12">
            Hello, {{ user.name }}
        </div>
    </Layout>
</template>
```

Seperti yang Anda lihat, Inertia memungkinkan Anda untuk memanfaatkan kekuatan penuh Vue atau React ketika membangun frontend Anda, sambil menyediakan jembatan yang ringan antara backend yang didukung oleh Laravel dengan frontend yang didukung oleh JavaScript.

#### Rendering Sisi Server

Jika Anda khawatir untuk menyelami Inertia karena aplikasi Anda membutuhkan rendering sisi server, jangan khawatir. Inertia menawarkan [dukungan rendering sisi server](https://inertiajs.com/server-side-rendering). Dan, ketika men-_deploy_ aplikasi Anda melalui [Laravel Forge](https://forge.laravel.com), sangat mudah untuk memastikan bahwa proses rendering sisi server Inertia selalu berjalan.

<a name="inertia-starter-kits"></a>
### Kit Pemula Inertia

Jika Anda ingin membangun frontend menggunakan Inertia dan Vue / React, Anda dapat memanfaatkan [kit pemula breeze dan inertia](/docs/{{version}}/starter-kits#breeze-dan-inertia) untuk memulai pengembangan aplikasi Anda. Kedua starter kit ini akan membangun alur otentikasi backend dan frontend aplikasi Anda menggunakan Inertia, Vue/React, [Tailwind](https://tailwindcss.com), dan [Vite](https://vitejs.dev) sehingga Anda bisa mulai membangun ide besar Anda berikutnya.

<a name="bundling-assets"></a>
## Aset Bundel

Terlepas dari apakah Anda memilih untuk mengembangkan frontend menggunakan Blade dan Livewire atau Vue / React dan Inertia, Anda mungkin perlu memaketkan CSS aplikasi Anda ke dalam aset yang siap untuk produksi. Tentu saja, jika Anda memilih untuk membangun frontend aplikasi Anda dengan Vue atau React, Anda juga perlu memaketkan komponen-komponen Anda ke dalam aset-aset JavaScript yang siap digunakan di peramban.

Secara default, Laravel menggunakan [Vite](https://vitejs.dev) untuk memaketkan aset Anda. Vite menyediakan waktu pembuatan secepat kilat dan Penggantian Modul Panas (Hot Module Replacement/HMR) yang hampir seketika selama pengembangan lokal. Di semua aplikasi Laravel baru, termasuk yang menggunakan [starter kits](/docs/{{version}}/starter-kits), Anda akan menemukan file `vite.config.js` yang memuat plugin Laravel Vite kami yang ringan yang membuat Vite sangat mudah digunakan dengan aplikasi Laravel.

Cara tercepat untuk memulai dengan Laravel dan Vite adalah dengan memulai pengembangan aplikasi Anda menggunakan [Laravel Breeze](/docs/{{version}}/starter-kits#laravel-breeze), starter kit kami yang paling sederhana yang dapat memulai aplikasi Anda dengan menyediakan perancah otentikasi frontend dan backend.

> **Catatan**  
> Untuk dokumentasi yang lebih mendetail tentang cara menggunakan Vite dengan Laravel, silakan lihat [dokumentasi khusus tentang bundling dan kompilasi aset Anda] (/docs/{{versi}}/vite).
