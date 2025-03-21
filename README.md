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

Saya juga menambahkan header Content-Length, yang berisi panjang isi HTML (length) sebagai informasi bagi browser tentang ukuran konten yang akan diterima agar sesuai dengan standar HTTP.

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

<details>
    <summary><strong> Commit 4 Reflection Notes </summary></strong>

```rust
use std::{
    fs,
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
    thread,
    time::Duration,
};
// --snip--

fn handle_connection(mut stream: TcpStream) {
    // --snip--

    let (status_line, filename) = match &request_line[..] {
        "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
        "GET /sleep HTTP/1.1" => {
            thread::sleep(Duration::from_secs(10));
            ("HTTP/1.1 200 OK", "hello.html")
        }
        _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
    };

    // --snip--
}
```
Perubahan pada kode dengan menambahkan simulasi delay menggunakan `thread::sleep(Duration::from_secs(10))` pada request `/sleep` ini menunjukkan kelemahan server single-threaded, di mana satu permintaan lambat dapat memblokir semua permintaan lain karena hanya ada satu thread yang memproses semua koneksi. 
</details>

<details>
    <summary><strong> Commit 5 Reflection Notes </summary></strong>

Perubahan pada main.rs:
```rust
use tutorial6::ThreadPool;

...

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }
}

...
```
src/lib:
```rust
use std::{
    sync::{mpsc, Arc, Mutex},
    thread::{self, JoinHandle},
};

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

type Job = Box<dyn FnOnce() + Send + 'static>;

impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();
        let receiver = Arc::new(Mutex::new(receiver));

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }

        ThreadPool { workers, sender }
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);

        self.sender.send(job).unwrap();
    }
}

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        
        let thread = thread::spawn(move || loop {
            let job = receiver.lock().unwrap().recv().unwrap();
            println!("Worker {id} got a job; executing.");
            job();
        });

        Worker { id, thread }
    }
}
```
Commit sebelumnya telah membuktikan single-thread akan menyebabkan masalah jika terjadi banjir permintaan ke server. Maka dari itu, pada langkah ini saya menerapkan multithreading.

Dengan `ThreadPool` yang menyimpan sejumlah `Worker`, yaitu thread-thread yang berjalan secara paralel. Masing-masing Worker akan terus mendengarkan saluran kerja (job queue) yang dikirimkan melalui `mpsc::channel`. Channel tersebut dibungkus oleh `Arc<Mutex<Receiver>>` agar bisa dibagikan dan diakses aman oleh banyak thread sekaligus. Ketika server menerima koneksi baru melalui `listener.incoming()`, method `pool.execute()` akan digunakan untuk mengirimkan closure yang berisi logika penanganan koneksi ke channel, dan salah satu Worker akan mengeksekusinya.

Desain ini secara teknis jauh lebih efisien dibanding membuat thread baru setiap kali ada koneksi masuk. Dengan `ThreadPool`, kita hanya perlu membuat sejumlah thread tetap di awal, dan pekerjaan (job) dikirim ke thread-thread tersebut sesuai giliran. Ini menghindari overhead pembuatan dan penghancuran thread terus-menerus.
</details>

<details>
    <summary><strong> Bonus </summary></strong>

Untuk meningkatkan keamanan dan keandalan sistem, cara pembuatan ThreadPool telah diubah dari `ThreadPool::new(4)` menjadi `ThreadPool::build(4)`. Metode build ini lebih aman karena mengembalikan      yang memungkinkan penanganan error saat inisialisasi. Jika jumlah thread tidak valid (misalnya 0 atau negatif), error akan dikembalikan tanpa menyebabkan panic. Untuk mendukung proses debugging, struct `PoolCreationError` juga dilengkapi implementasi Display dan Debug agar pesan kesalahan lebih informatif dan mudah dilacak.

Perubahan pada main.rs:
```rust
...
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::build(4).unwrap();
    ...
}
...
```

Perubahan pada lib.rs:
```rust

use std::{
    fmt,
    ...
};
...

pub struct PoolCreationError {
      reason: String,
 }
  
 impl fmt::Display for PoolCreationError {
      fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
          write!(
              f,
              "Unable to create ThreadPool: {}", self.reason
          )
      }
 }
  
 impl fmt::Debug for PoolCreationError {
      fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
          write!(f, "{{ file: {}, line: {} }}", file!(), line!())
      }
 }
  
 impl ThreadPool {
      pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
          if size <= 0 {
              Err(PoolCreationError {
                  reason: "Thread count must be a positive nonzero integer".to_string()
              })
          } else {
              let (sender, receiver) = mpsc::channel();
              let receiver = Arc::new(Mutex::new(receiver));
  
              let mut workers = Vec::with_capacity(size);
  
              for id in 0..size {
                  workers.push(Worker::new(id, Arc::clone(&receiver)));
              }
  
              Ok(ThreadPool { workers, sender })
          }
      }
      
      ...
 }
 ...
 ```

Dengan pendekatan ini, proses inisialisasi thread pool menjadi lebih robust karena kesalahan konfigurasi dapat dideteksi lebih awal sebelum server berjalan. Hal ini membuat sistem lebih stabil dan aman

</details>

