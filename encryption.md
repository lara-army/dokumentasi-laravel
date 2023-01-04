# Enkripsi

- [Pengantar](#introduction)
- [Konfigurasi](#configuration)
- [Menggunakan Pengenkripsi](#using-the-encrypter)

<a name="introduction"></a>
## Pengantar

Layanan enkripsi Laravel menyediakan _interface_ yang sederhana dan mudah untuk mengenkripsi dan mendekripsi teks melalui OpenSSL menggunakan enkripsi AES-256 dan AES-128. Semua nilai yang ter-enkripsi pada Laravel ditandai menggunakan kode autentikasi pesan (_message authentication code_)(MAC) sehingga nilai dasarnya tidak dapat dimodifikasi atau dimanipulasi setelah ter-enkripsi.

<a name="configuration"></a>
## Konfigurasi

Sebelum menggunakan pengenkripsi milik Laravel, Anda harus mengatur opsi konfigurasi key pada _file_ `config/app.php`. Nilai konfigurasi ini dikendalikan oleh `APP_KEY` pada _environment variable_. Anda harus menggunakan perintah `php artisan key:generate` untuk menghasilkan nilai variabel ini karena perintah `key:generate` akan menggunakan generator _byte_ aman milik PHP untuk membangun kunci yang aman secara kriptografis untuk aplikasi Anda. Biasanya, nilai `APP_KEY` akan dihasilkan untuk Anda pada saat [installasi Laravel](/docs/{{version}}/installation).

<a name="using-the-encrypter"></a>
## Menggunakan Pengenkripsi

<a name="encrypting-a-value"></a>
#### Mengenkripsi Sebuah Nilai

Anda dapat mengenkripsi nilai menggunakan _method_ `encryptString` yang disediakan oleh _facade_ `Crypt`. Semua nilai dienkripsi menggunakan OpenSSL dan _cipher_ AES-256-CBC. Selain itu, semua nilai ter-enkripsi ditandai dengan kode autentikasi pesan (MAC). Kode autentikasi pesan terintegrasi akan mencegah dekripsi nilai apapun yang telah dimanipulasi oleh pengguna yang tidak bertanggung jawab:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\User;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Crypt;

    class DigitalOceanTokenController extends Controller
    {
        /**
         * Menyimpan token API DigitalOcean untuk pengguna.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function storeSecret(Request $request)
        {
            $request->user()->fill([
                'token' => Crypt::encryptString($request->token),
            ])->save();
        }
    }

<a name="decrypting-a-value"></a>
#### Mendekripsi Sebuah Nilai

Anda dapat mendekripsi nilai menggunakan _method_ `decryptString` yang disediakan oleh _facade_ `Crypt`. Jika nilai tidak dapat didekripsi dengan benar, seperti ketika kode autentikasi pesan tidak valid, sebuah `Illuminate\Contracts\Encryption\DecryptException` akan dilemparkan:

    use Illuminate\Contracts\Encryption\DecryptException;
    use Illuminate\Support\Facades\Crypt;

    try {
        $decrypted = Crypt::decryptString($encryptedValue);
    } catch (DecryptException $e) {
        //
    }
