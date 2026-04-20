# Commit 1 Reflection Notes

Pada milestone ini saya mempelajari bagaimana web server sederhana bekerja pada level yang lebih rendah tanpa bantuan framework seperti Django. Program Rust ini menggunakan `TcpListener` untuk membuka port `127.0.0.1:7878` dan menunggu koneksi dari browser. Setiap koneksi yang masuk direpresentasikan sebagai `TcpStream`.

Pada versi awal, server hanya mencetak pesan `"Connection established!"` setiap kali browser terhubung. Dari sini saya memahami bahwa browser memang berhasil membuat koneksi ke server, tetapi karena server belum mengirimkan HTTP response apa pun, halaman di browser tidak menampilkan isi.

Pada versi berikutnya, koneksi ditangani oleh fungsi `handle_connection`. Fungsi ini menerima `TcpStream`, lalu membungkusnya dengan `BufReader` agar isi request dapat dibaca per baris. Ini penting karena HTTP request dikirim dalam format teks yang terdiri dari beberapa baris header.

Bagian yang paling penting untuk saya pahami adalah potongan berikut:

```rust
let http_request: Vec<_> = buf_reader
    .lines()
    .map(|result| result.unwrap())
    .take_while(|line| !line.is_empty())
    .collect();
```

Dari kode ini saya belajar beberapa hal:
- `.lines()` membaca stream menjadi baris-baris teks.
- `.map(|result| result.unwrap())` mengambil isi setiap baris dari `Result`.
- `.take_while(|line| !line.is_empty())` membaca hanya sampai baris kosong, karena pada protokol HTTP baris kosong menandakan akhir header.
- `.collect()` mengumpulkan semua baris tersebut ke dalam sebuah vector agar bisa dicetak atau diproses lebih lanjut.

Setelah menjalankan program dan membuka `http://127.0.0.1:7878`, saya bisa melihat isi request dari browser di terminal, misalnya `GET / HTTP/1.1`, `Host`, `User-Agent`, dan header lainnya. Dari sini saya jadi lebih paham bahwa browser dan server sebenarnya berkomunikasi menggunakan format request/response HTTP yang cukup terstruktur.

Hal lain yang saya pelajari adalah kenapa server perlu dihentikan terlebih dahulu sebelum dijalankan ulang. Jika proses sebelumnya masih berjalan, port `7878` masih dipakai sehingga program baru bisa gagal dijalankan.

Menurut saya, latihan ini membantu memahami fondasi web server yang biasanya disembunyikan oleh framework. Jika sebelumnya saya memakai framework tingkat tinggi, sekarang saya bisa melihat bahwa di baliknya ada proses mendengarkan koneksi, membaca request, lalu nanti mengirim response secara manual.
