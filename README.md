# AdvProg Tutorial 6
## Agus Tini Sridewi / 2306276004 / ADPRO A

<details>
    <summary><strong> Commit 1 Reflection Notes </summary></strong>

Kode yang saya jalankan:

```rust 
use std::{
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
};

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    println!("Request: {http_request:#?}");
}
```
Fungsi `handle_connection` dalam program ini bertugas untuk menangani koneksi yang diterima oleh server. Ketika koneksi masuk diterima melalui `TcpStream`, koneksi tersebut dibungkus menggunakan `BufReader`. Dengan bantuan metode `.lines()` dari trait `BufRead`, kita dapat membaca setiap baris dari permintaan HTTP yang dikirim oleh browser. Untuk menentukan batas akhir dari header permintaan, digunakan metode `.take_while(|line| !line.is_empty())` yang akan berhenti membaca ketika menemukan baris kosong pertama sesuai dengan standar protokol HTTP.

Setiap baris yang terbaca awalnya memiliki tipe `Result<String, std::io::Error>`, sehingga perlu diubah menjadi `String` menggunakan `.map(|result| result.unwrap())`. Selanjutnya, seluruh baris dikumpulkan ke dalam sebuah vektor `Vec<String>` menggunakan metode `.collect()`. Akhirnya, hasil permintaan HTTP yang telah dikumpulkan ditampilkan dengan format debug `{:#?}` agar  mudah dibaca.

Saat server dijalankan dan saya mengakses alamat http://127.0.0.1:7878 melalui browser, saya bisa melihat dengan jelas isi permintaan HTTP yang dikirim oleh browser, seperti metode yang digunakan (GET / HTTP/1.1), informasi host, user agent, jenis konten yang diterima, hingga cookies yang dikirimkan.

Permintaan HTTP yang diterima:
 ```bash
 Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "Connection: keep-alive",
    "sec-ch-ua: \"Not(A:Brand\";v=\"99\", \"Google Chrome\";v=\"133\", \"Chromium\";v=\"133\"",
    "sec-ch-ua-mobile: ?0",
    "sec-ch-ua-platform: \"macOS\"",
    "Upgrade-Insecure-Requests: 1",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/133.0.0.0 Safari/537.36",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Dest: document",
    "Accept-Encoding: gzip, deflate, br, zstd",
    "Accept-Language: en-US,en;q=0.9,id-ID;q=0.8,id;q=0.7",
    "Cookie: csrftoken=AVQpXuTt2pLAt1RJOovVJ5VrDHot53bD",
]
 ```
</details>

<details>
    <summary><strong> Commit 2 Reflection Notes </summary></strong>

```rust
use std::{
    fs,
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
};

...

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    let status_line = "HTTP/1.1 200 OK";
    let contents = fs::read_to_string("hello.html").unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}

```
Screen Capture ⤵️
![Commit 2 screen capture](/assets/images/commit2.png)

Fungsi `handle_connection` pada milestone ini telah ditingkatkan agar tidak hanya membaca permintaan HTTP dari browser, tetapi juga memberikan respon HTML yang bisa ditampilkan di browser. Kita menambahkan `fs` ke dalam daftar use agar dapat mengakses modul sistem file dari Rust. Kemudian, file `hello.html` dibaca menggunakan `fs::read_to_string`, dan isi dari file tersebut disisipkan ke dalam respons HTTP melalui `format!`. 

Agar sesuai dengan standar HTTP, kita juga menambahkan header Content-Length, yang berisi panjang isi HTML (length) sebagai informasi bagi browser tentang ukuran konten yang akan diterima.

Bagian `{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}` adalah inti dari format respons HTTP yang dikirim ke browser. `status_line` berisi "HTTP/1.1 200 OK", menandakan bahwa permintaan telah berhasil diproses. Lalu, `Content-Length: {length}` memberi tahu browser seberapa panjang isi yang akan diterima dalam byte. Dua baris kosong \r\n\r\n adalah pemisah yang wajib dalam format HTTP, yang menandakan bahwa header telah selesai dan bagian selanjutnya adalah isi dari respons.Setelah semua elemen respons disusun, data ini dikirimkan ke browser menggunakan `stream.write_all`.
</details>

<details>
    <summary><strong> Commit 3 Reflection Notes </summary></strong>

Perubahan pada fungsi `handle_connection`:
```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
```

Perubahan pada fungsi `handle_connection` di atas terletak pada pemisahan (split) antara bagian request dan response. Kita tidak lagi hanya membaca semua baris dari permintaan HTTP, tetapi cukup mengambil baris pertama saja (`request_line`) karena baris inilah yang menentukan jenis permintaan (misalnya `GET / HTTP/1.1`). Dengan memisahkan logika ini, kita bisa dengan mudah mengenali permintaan klien dan meresponsnya dengan file yang sesuai, seperti `hello.html` untuk permintaan ke /, atau `404.html` jika permintaan tidak valid.

Saya juga menerapkan refactoring untuk menghilangkan duplikasi kode dan membuat struktur program lebih ringkas serta mudah dipelihara. Sebelumnya, blok `if` dan `else` memiliki isi yang hampir sama, hanya berbeda pada `status_line` dan `filename`. Dengan refactoring, perbedaan tersebut dipisahkan ke dalam satu pernyataan `let`, sementara proses membaca file, menghitung panjang konten, dan membentuk respons HTTP cukup dituliskan sekali di luar blok kondisi. Ini membuat kode lebih clean.

Screen Capture ⤵️
![Commit 3 screen capture](/assets/images/commit3.png)

</details>