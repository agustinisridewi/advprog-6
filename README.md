# AdvProg Tutorial 6

## Commit 1 Reflection Notes

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
