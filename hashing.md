# Melakukan _Hash_ (_Hashing_)

- [Pengantar](#introduction)
- [Konfigurasi](#configuration)
- [Penggunaan Dasar](#basic-usage)
  - [_Hash_ Kata Sandi](#hashing-passwords)
  - [Memverifikasi Bahwa Kata Sandi Cocok Dengan _Hash_](#verifying-that-a-password-matches-a-hash)
  - [Menentukan Apakah Kata Sandi Perlu Di-_hash_ Ulang](#determining-if-a-password-needs-to-be-rehashed)

<a name="introduction"></a>
## Pengantar

[Facade](/docs/{{version}}/facades) `Hash` milik Laravel menyediakan fungsi _hash_ Bcrypt dan Argon2 yang aman untuk menyimpan kata sandi pengguna. Jika Anda menggunakan salah satu dari [_starter kit_ aplikasi Laravel](/docs/{{version}}/starter-kits), Bcrypt akan digunakan untuk registrasi dan autentikasi secara _default_.

Bcrypt merupakan pilihan yang baik untuk melakukan _hash_ kata sandi karena memiliki "_work factor_" yang dapat disesuaikan, yang berarti bahwa waktu yang dibutuhkan untuk menghasilkan _hash_ dapat ditingkatkan seiring dengan meningkatnya daya milik perangkat keras. Ketika melakukan _hash_ kata sandi, lambat itu bagus. Semakin lama sebuah algoritma melakukan _hash_ pada kata sandi, semakin lama pula waktu yang dibutuhkan oleh pengguna jahat untuk menghasilkan "_rainbow tables_" dari semua kemungkinan nilai _string_ untuk _hash_ yang dapat digunakan dalam serangan _brute force_ ke aplikasi.

<a name="configuration"></a>
## Konfigurasi

_Driver_ _default_ yang melakukan _hash_ pada aplikasi Anda dikonfigurasikan pada _file_ konfigurasi `config/hashing.php` milik aplikasi Anda. Adapun _driver_-_driver_ yang mendukung saat ini: [Bcrypt](https://en.wikipedia.org/wiki/Bcrypt) dan [Argon2](https://en.wikipedia.org/wiki/Argon2) (varian Argon2i dan Argon2id).

<a name="basic-usage"></a>
## Penggunaan Dasar

<a name="hashing-passwords"></a>
### _Hash_ Kata Sandi

Anda dapat melakukan _hash_ kata sandi dengan menggunakan metode `make` pada _facade_ `Hash`

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;

    class PasswordController extends Controller
    {
        /**
         * Memperbarui kata sandi untuk pengguna.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function update(Request $request)
        {
            // Memvalidasi panjang kata sandi yang baru...

            $request->user()->fill([
                'password' => Hash::make($request->kataSandiBaru)
            ])->save();
        }
    }

<a name="adjusting-the-bcrypt-work-factor"></a>
#### Menyesuaikan Faktor Kerja Bcrypt

Jika Anda menggunakan algoritma Bcrypt, metode `make` memungkinkan Anda untuk mengelola faktor kerja (_work factor_) algoritma menggunakan opsi `rounds`; bagaimanapun, faktor kerja _default_ yang dikelola/dipilih oleh Laravel juga pantas/akseptabel untuk kebanyakan aplikasi:

    $hashed = Hash::make('kata sandi', [
        'rounds' => 12,
    ]);

<a name="adjusting-the-argon2-work-factor"></a>
#### Menyesuaikan Faktor Kerja Argon2

Jika Anda menggunakan algoritme Argon2, metode `make` memungkinkan Anda untuk mengelola faktor kerja algoritma menggunakan opsi `memory`, `time`, dan `threads`; bagaimanapun, nilai _default_ yang dikelola/dipilih oleh Laravel juga pantas/akseptabel untuk kebanyakan aplikasi:

    $hashed = Hash::make('kata sandi', [
        'memory' => 1024,
        'time' => 2,
        'threads' => 2,
    ]);

> **Note**  
> Untuk informasi lebih lanjut tentang opsi-opsi ini, silakan lihat [dokumentasi PHP resmi mengenai peng-_hash_-an Argon](https://secure.php.net/manual/en/function.password-hash.php).

<a name="verifying-that-a-password-matches-a-hash"></a>
### Memverifikasi Bahwa Kata Sandi Cocok dengan _Hash_

Metode `check` yang disediakan oleh _facade_ `Hash` memungkinkan Anda untuk memverifikasi bahwa _string_ teks polos yang diberikan telah sesuai dengan _hash_ yang diberikan:

    if (Hash::check('teks-polos', $HashKataSandi)) {
        // Kata sandi cocok...
    }

<a name="determining-if-a-password-needs-to-be-rehashed"></a>
### Menentukan Apakah Kata Sandi Perlu Di-_hash_ Ulang

Metode `needsRehash` yang disediakan oleh _facade_ `Hash` memungkinkan Anda untuk menentukan apakah faktor kerja yang digunakan oleh _driver_ _hash_ telah berubah sejak kata sandi terakhir kali di-_hash_. Beberapa aplikasi memilih untuk melakukan pemeriksaan ini selama proses autentikasi pada aplikasi:

    if (Hash::needsRehash($hash)) {
        $hash = Hash::make('teks-polos');
    }
