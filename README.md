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


# Commit 2 Reflection Notes

Pada milestone ini saya mempelajari langkah berikutnya setelah server berhasil menerima request, yaitu mengirimkan HTTP response yang bisa dibaca dan dirender oleh browser. Jika pada milestone sebelumnya server hanya mencetak isi request ke terminal, sekarang server mulai benar-benar mengirimkan balasan ke client.

Perubahan utama ada di fungsi `handle_connection`. Setelah request dibaca, program membuat `status_line` dengan nilai `HTTP/1.1 200 OK`. Dari sini saya memahami bahwa response HTTP harus diawali dengan status line agar browser tahu hasil dari request yang dikirim. Kode ini juga membaca isi file `hello.html` menggunakan `fs::read_to_string`, sehingga konten HTML tidak ditulis langsung di dalam source code Rust.

Saya juga belajar bahwa `Content-Length` adalah bagian penting dari response header. Nilai ini memberi tahu browser berapa panjang isi response yang akan diterima. Setelah itu seluruh response digabung menggunakan `format!`, lalu dikirim ke browser melalui `stream.write_all(response.as_bytes())`. Dari proses ini saya jadi lebih memahami bahwa browser hanya bisa menampilkan halaman jika server mengirim response HTTP yang valid, bukan sekadar menerima koneksi.

Menurut saya, bagian ini menjelaskan hubungan yang lebih nyata antara server dan browser. Server membaca request dari client, lalu menyusun response secara manual yang terdiri dari status line, header, baris kosong, dan body berupa HTML. Ini membuat saya lebih paham struktur dasar protokol HTTP yang biasanya tidak terlihat saat menggunakan framework web tingkat tinggi.

Saya juga memahami kenapa file `hello.html` harus berada pada direktori yang sesuai saat program dijalankan. Karena program membaca file itu secara relatif, maka `cargo run` perlu dijalankan dari folder project yang memang berisi file `hello.html`. Jika tidak, program bisa gagal menemukan file tersebut.

Berikut adalah hasil tampilan halaman HTML yang berhasil dikirim oleh server:

![Commit 2 screen capture](./assets/commit2.png)
