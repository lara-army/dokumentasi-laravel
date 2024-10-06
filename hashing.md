# Hashing

- [Perkenalan](#introduction)
- [Konfigurasi](#configuration)
- [Penggunaan Dasar](#basic-usage)
  - [_Hashing_ Kata Sandi](#hashing-passwords)
  - [Memverifikasi Bahwa Kata Sandi Cocok Dengan _Hash_](#verifying-that-a-password-matches-a-hash)
  - [Menentukan Apakah Kata Sandi Perlu Diulang](#determining-if-a-password-needs-to-be-rehashed)

<a name="introduction"></a>

## Perkenalan

Laravel `Hash` [facade](/docs/{{version}}/facades) menyediakan _hashing_ Bcrypt dan Argon2 yang aman untuk menyimpan kata sandi pengguna. Jika Anda menggunakan salah satu dari [starter kit aplikasi Laravel](/docs/{{version}}/starter-kits), Bcrypt akan digunakan untuk registrasi dan autentikasi secara _default_.

Bcrypt merupakan pilihan yang bagus untuk melakukan _hashing_ kata sandi karena “work factor" dapat disesuaikan, yang berarti bahwa waktu yang dibutuhkan untuk menghasilkan _hash_ dapat ditingkatkan seiring dengan meningkatnya daya perangkat keras. Ketika melakukan _hashing_ kata sandi, lambat itu bagus. Semakin lama sebuah algoritma melakukan _hash_ pada kata sandi, semakin lama pula waktu yang dibutuhkan oleh pengguna jahat untuk menghasilkan “rainbow tables” dari semua kemungkinan nilai _hash_ string yang dapat digunakan pada serangan _brute_ _force_ pada aplikasi.

<a name="configuration"></a>

## Konfigurasi

_Driver_ _hashing_ _default_ untuk aplikasi Anda dikonfigurasikan dalam _file_ konfigurasi `config/hashing.php` aplikasi Anda. Saat ini ada beberapa _driver_ yang didukung: [Bcrypt](https://en.wikipedia.org/wiki/Bcrypt) dan [Argon2](https://en.wikipedia.org/wiki/Argon2) (varian Argon2i dan Argon2id).

<a name="basic-usage"></a>

## Penggunaan Dasar

<a name="hashing-passwords"></a>

### _Hashing_ Kata Sandi

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
                'password' => Hash::make($request->newPassword)
            ])->save();
        }
    }

<a name="adjusting-the-bcrypt-work-factor"></a>

#### Menyesuaikan Faktor Kerja Bcrypt

Jika Anda menggunakan algoritma Bcrypt, metode `make` memungkinkan Anda untuk mengelola faktor kerja (_work factor_) algoritma menggunakan opsi `rounds`; namun, faktor kerja _default_ yang dikelola oleh Laravel dapat diterima untuk sebagian besar aplikasi:

    $hashed = Hash::make('password', [
        'rounds' => 12,
    ]);

<a name="adjusting-the-argon2-work-factor"></a>

#### Menyesuaikan Faktor Kerja Argon2

Jika Anda menggunakan algoritma Argon2, metode `make` memungkinkan Anda untuk mengelola faktor kerja algoritma menggunakan opsi `memory`, `time`, dan `threads`; namun, nilai _default_ yang dikelola oleh Laravel dapat diterima untuk sebagian besar aplikasi:

    $hashed = Hash::make('password', [
        'memory' => 1024,
        'time' => 2,
        'threads' => 2,
    ]);

> **Catatan**  
> Untuk informasi lebih lanjut tentang opsi-opsi ini, silakan lihat [dokumentasi PHP resmi mengenai _hashing_ Argon](https://secure.php.net/manual/en/function.password-hash.php).

<a name="verifying-that-a-password-matches-a-hash"></a>

### Memverifikasi Bahwa Kata Sandi Cocok dengan _Hash_

Metode `check` yang disediakan oleh _facade_ `Hash` memungkinkan Anda untuk memverifikasi bahwa string teks biasa yang diberikan sesuai dengan _hash_ yang diberikan:

    if (Hash::check('plain-text', $hashedPassword)) {
        // The passwords match...
    }

<a name="determining-if-a-password-needs-to-be-rehashed"></a>

### Menentukan Apakah Kata Sandi Perlu Diulang

Metode `needsRehash` yang disediakan oleh _facade_ `Hash` memungkinkan Anda untuk menentukan apakah faktor kerja yang digunakan oleh _hasher_ telah berubah sejak kata sandi di-_hash_. Beberapa aplikasi memilih untuk melakukan pemeriksaan ini selama proses autentikasi aplikasi:

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }
