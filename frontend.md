# _Frontend_

- [Pengantar](#introduction)
- [Menggunakan PHP](#using-php)
    - [PHP dan Blade](#php-and-blade)
    - [Livewire](#livewire)
    - [Kit Pemula PHP](#php-starter-kits)
- [Menggunakan Vue / React](#using-vue-react)
    - [Inertia](#inertia)
    - [Kit Pemula Inertia](#inertia-starter-kits)
- [Membundel Aset](#bundling-assets)

<a name="introduction"></a>
## Pengantar

Laravel adalah _framework backend_ yang menyediakan semua fitur yang Anda perlukan untuk membangun aplikasi web modern, seperti [_route_](/docs/{{version}}/routing), [validasi](/docs/{{version}}/validasi), [_cache_](/docs/{{version}}/cache), [_queues_](/docs/{{version}}/queues), [penyimpanan _file_](/docs/{{version}}/filesystem ), dan masih banyak lagi. Namun, kami yakin penting untuk menawarkan pengalaman _full-stack_ yang indah kepada developer, termasuk pendekatan yang andal untuk membangun aplikasi _frontend_ anda.

Terdapat dua cara untuk mengembangkan _frontend_ ketika membangun aplikasi dengan Laravel, dan pendekatan mana yang Anda pilih bergantung pada apakah Anda ingin membangun _frontend_ Anda dengan memanfaatkan PHP atau dengan menggunakan _framework_ JavaScript seperti Vue dan React. Kami akan membahas kedua opsi tersebut sehingga Anda dapat membuat keputusan yang tepat mengenai pendekatan terbaik untuk pengembangan _frontend_ untuk aplikasi Anda.

<a name="using-php"></a>
## Menggunakan PHP

<a name="php-and-blade"></a>
### PHP dan Blade

Di masa lalu, sebagian besar aplikasi PHP menampilkan HTML ke _browser_ menggunakan _template_ HTML sederhana **diselingi** dengan pernyataan PHP `echo` yang menampilkan data yang diambil dari _database_ selama permintaan:

```blade
<div>
    <?php foreach ($users as $user): ?>
        Hello, <?php echo $user->name; ?> <br />
    <?php endforeach; ?>
</div>
```

Pada Laravel, pendekatan menampilkan HTML ini masih dapat dicapai dengan menggunakan [_view_](/docs/{{version}}/views) dan [_Blade_](/docs/{{version}}/blade). _Blade_ adalah bahasa _templating_ yang sangat ringan yang menyediakan sintaks pendek yang nyaman untuk menampilkan data, mengulangi data, dan banyak lagi:

```blade
<div>
    @foreach ($users as $user)
        Hello, {{ $user->name }} <br />
    @endforeach
</div>
```

Saat membuat aplikasi dengan pendekatan ini, pengiriman formulir dan interaksi halaman lain biasanya menerima dokumen HTML yang (benar-benar) baru dari server kemudian seluruh halaman di-_render_ ulang oleh _browser_. Bahkan sampai hari ini, pembangunan _frontend_ dengan pendekatan ini dengan menggunakan _template Blade_ yang sederhana akan menjadi perpaduan yang sempurna pada banyak aplikasi

<a name="growing-expectations"></a>
#### Mengembangkan Ekspektasi

Namun, karena ekspektasi pengguna terkait aplikasi web telah matang, banyak developer menemukan kebutuhan untuk membangun _frontend_ yang lebih dinamis dengan interaksi yang terasa lebih halus. Mengingat hal ini, beberapa developer memilih untuk mulai membangun _frontend_ aplikasi mereka menggunakan _framework_ JavaScript seperti Vue dan React.

Lainnya lebih memilih untuk tetap menggunakan bahasa _backend_ yang mereka sukai dan telah mengembangkan solusi yang memungkinkan konstruksi UI (_user interface_) aplikasi web modern sambil tetap menggunakan bahasa _backend_ pilihan mereka. Misalnya, di ekosistem [Rails](https://rubyonrails.org/), hal ini telah mendorong pembuatan _library_ seperti [Turbo](https://turbo.hotwired.dev/) [Hotwire](https://hotwired.dev/), dan [Stimulus](https://stimulus.hotwired.dev/).

Dalam ekosistem Laravel, kebutuhan untuk membuat _frontend_ yang modern dan dinamis terutama dengan menggunakan PHP telah menyebabkan terciptanya [Laravel Livewire](https://laravel-livewire.com) dan [Alpine.js](https://alpinejs.dev/).

<a name="livewire"></a>
### Livewire

[Laravel Livewire](https://laravel-livewire.com) adalah sebuah _framework_ untuk membangun _frontend_ bertenaga Laravel yang terasa dinamis, modern, dan hidup seperti _frontend_ yang dibuat dengan _framework_ JavaScript modern seperti Vue dan React.

Saat menggunakan Livewire, Anda akan membuat "komponen" Livewire yang me-_render_ bagian terpisah dari UI (_user interface_) Anda dan menampilkan metode dan data yang dapat dipanggil dan berinteraksi dari _frontend_ aplikasi Anda. Misalnya, komponen "Penghitung" sederhana mungkin terlihat seperti berikut:

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

```blade
<div>
    <button wire:click="increment">+</button>
    <h1>{{ $count }}</h1>
</div>
```

Seperti yang Anda lihat, Livewire memungkinkan Anda menulis atribut HTML baru seperti `wire:click` yang menghubungkan _frontend_ dan _backend_ aplikasi Laravel Anda. Selain itu, Anda dapat me-_render_ status komponen Anda saat ini menggunakan ekspresi _Blade_ sederhana.

Bagi banyak orang, Livewire telah berevolusi pengembangan _frontend_ dengan Laravel, memungkinkan mereka untuk tetap berada dalam kenyamanan Laravel sambil membangun aplikasi web yang modern dan dinamis. Biasanya, developer yang menggunakan Livewire juga akan memanfaatkan [Alpine.js](https://alpinejs.dev/) untuk "memercikan" JavaScript ke _frontend_ mereka hanya jika diperlukan, misalnya untuk me-_render_ jendela dialog.

Jika Anda baru menggunakan Laravel, sebaiknya pahami penggunaan dasar [_view_](/docs/{{version}}/views) dan [Blade](/docs/{{version}}/blade). Kemudian, lihat [dokumentasi Laravel Livewire](https://laravel-livewire.com/docs) resmi untuk mempelajari cara membawa aplikasi Anda ke level selanjutnya dengan komponen Livewire interaktif.

<a name="php-starter-kits"></a>
### Kit Pemula PHP

Jika Anda ingin membuat _frontend_ menggunakan PHP dan Livewire, Anda dapat memanfaatkan [kit pemula](/docs/{{version}}/starter-kits) Breeze atau Jetstream kami untuk memulai pengembangan aplikasi Anda. Kedua starter kit ini menyusun alur autentikasi _backend_ dan _frontend_ aplikasi Anda menggunakan [Blade](/docs/{{version}}/blade) dan [Tailwind](https://tailwindcss.com) sehingga Anda dapat langsung mulai membangun ide besar berikutnya.

<a name="using-vue-react"></a>
## Menggunakan Vue / React

Meskipun dimungkinkan untuk membangun _frontend_ modern menggunakan Laravel dan Livewire, banyak pengembang masih lebih memilih untuk memanfaatkan kekuatan kerangka kerja JavaScript seperti Vue atau React. Hal ini memungkinkan para pengembang untuk memanfaatkan ekosistem yang kaya akan paket dan alat JavaScript yang tersedia melalui [NPM](https://www.npmjs.com/).

Namun, tanpa alat tambahan, memadukan Laravel dengan Vue atau React akan membuat kita harus menyelesaikan berbagai masalah rumit seperti rute sisi klien, _hydration_ data, dan autentikasi. Rute sisi klien sering disederhanakan dengan menggunakan _framework_ Vue / React yang memiliki opini seperti [Nuxt](https://nuxtjs.org/) dan [Next](https://nextjs.org/); namun, _hydration_ dan autentikasi data tetap menjadi masalah yang rumit dan tidak praktis untuk dipecahkan saat memadukan kerangka kerja _backend_ seperti Laravel dengan kerangka kerja _frontend_ ini.

Selain itu, pengembang harus mengelola dua repositori kode yang terpisah, dan sering kali harus mengoordinasikan pemeliharaan, rilis, dan penerapan di kedua repositori tersebut. Meskipun masalah-masalah ini bukannya tidak dapat diatasi, kami tidak percaya bahwa ini adalah cara yang produktif atau menyenangkan untuk mengembangkan aplikasi.

<a name="inertia"></a>
### Inertia

Untungnya, Laravel menawarkan yang terbaik pada kedua dunia. [Inertia](https://inertiajs.com) menjembatani celah antara aplikasi Laravel Anda dan _frontend_ Vue atau React modern Anda, memungkinkan Anda untuk membangun _frontend_ modern yang lengkap menggunakan Vue atau React sambil memanfaatkan rute dan pengontrol Laravel untuk perutean, _hydration_ data, dan otentikasi - semuanya dalam satu repositori kode. Dengan pendekatan ini, Anda dapat menikmati kekuatan penuh dari Laravel dan Vue / React tanpa melumpuhkan kemampuan salah satu dari keduanya.

Setelah menginstal Inertia ke dalam aplikasi Laravel Anda, Anda akan menulis rute dan _controller_ seperti biasa. Namun, alih-alih mengembalikan _template Blade_ dari _controller_ Anda, Anda akan mengembalikan halaman Inertia:

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

Halaman Inertia berhubungan dengan komponen Vue atau React, biasanya disimpan di dalam direktori `resources/js/Pages` pada aplikasi Anda. Data yang diberikan ke halaman melalui metode `Inertia::render` akan digunakan untuk mengisi "_props_" dari komponen halaman:

```vue
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

Seperti yang Anda lihat, Inertia memungkinkan Anda untuk memanfaatkan kekuatan penuh Vue atau React ketika membangun _frontend_ Anda, sambil menyediakan jembatan yang ringan antara _backend_ Laravel dengan _frontend_ JavaScript.

#### _Server-Side Rendering_

Jika Anda khawatir untuk menyelami Inertia karena aplikasi Anda membutuhkan _rendering_ sisi server, jangan khawatir. Inertia menawarkan [dukungan _rendering_ sisi server](https://inertiajs.com/server-side-rendering). Dan, ketika men-_deploy_ aplikasi Anda melalui [Laravel Forge](https://forge.laravel.com), akan sangat mudah untuk memastikan bahwa proses _rendering_ sisi server Inertia selalu berjalan.

<a name="inertia-starter-kits"></a>
### Kit Pemula Inertia

Jika Anda ingin membangun _frontend_ menggunakan Inertia dan Vue / React, Anda dapat memanfaatkan [kit pemula Breeze dan Inertia](/docs/{{version}}/starter-kits#breeze-dan-inertia) untuk memulai pengembangan aplikasi Anda. Kedua starter kit ini akan membangun alur otentikasi _backend_ dan _frontend_ aplikasi Anda menggunakan Inertia, Vue / React, [Tailwind](https://tailwindcss.com), dan [Vite](https://vitejs.dev) sehingga Anda bisa mulai membangun ide besar Anda berikutnya.

<a name="bundling-assets"></a>
## Membundel Aset

Terlepas dari apakah Anda memilih untuk mengembangkan _frontend_ menggunakan _Blade_ dan Livewire atau Vue / React dan Inertia, Anda mungkin perlu memaketkan CSS aplikasi Anda ke dalam aset yang siap untuk produksi. Tentu saja, jika Anda memilih untuk membangun _frontend_ aplikasi Anda dengan Vue atau React, Anda juga perlu memaketkan komponen-komponen Anda ke dalam aset-aset JavaScript yang siap untuk _browser_.

Secara _default_, Laravel menggunakan [Vite](https://vitejs.dev) untuk memaketkan aset Anda. Vite menyediakan waktu pembuatan secepat kilat dan Penggantian Modul Panas (_Hot Module Replacement/HMR_) yang hampir seketika selama pengembangan lokal. Di semua aplikasi Laravel baru, termasuk yang menggunakan [starter kits](/docs/{{version}}/starter-kits), Anda akan menemukan file `vite.config.js` yang memuat _plugin_ Laravel Vite kami yang ringan sehingga penggunaan Vite pada aplikasi Laravel adalah sebuah suka cita.

Cara tercepat untuk memulai dengan Laravel dan Vite adalah dengan memulai pengembangan aplikasi Anda menggunakan [Laravel Breeze](/docs/{{version}}/starter-kits#laravel-breeze), starter kit kami yang paling sederhana yang dapat memulai aplikasi Anda dengan menyediakan perancah otentikasi _frontend_ dan _backend_.

> **Catatan**  
> Untuk dokumentasi yang lebih mendetail tentang cara menggunakan Vite dengan Laravel, silakan lihat [dokumentasi khusus tentang bundel dan kompilasi aset Anda](/docs/{{version}}/vite).
