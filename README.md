# Reflection
Nama : Muhammad Naufal Ramadhan<br>
NPM : 2306241700<br>
Kelas : B<br>

## Commit 1 Reflection Notes
Melalui kode ini, saya mengetahui cara kerja sebuah server HTTP menggunakan Rust. Saya memahami bagaimana TcpListener digunakan untuk menerima koneksi dari client, dan setiap koneksi diproses sebagai TcpStream. Dengan membungkus stream menggunakan BufReader, saya dapat membaca request HTTP baris demi baris hingga menemukan baris kosong sebagai penanda akhir header. Seluruh baris dikumpulkan ke dalam Vec<String> lalu dicetak, sehingga saya bisa melihat isi request dari browser secara mentah. Proses ini memperjelas bagaimana komunikasi HTTP terjadi di level rendah.

## Commit 2 Reflection Notes
```
    let status_line = "HTTP/1.1 200 OK";
    let contents = fs::read_to_string("hello.html").unwrap();
    let length = contents.len();
    let response = format!("{status_line}\r\nContent-Length:{length}\r\n\r\n{contents}");
    
    stream.write_all(response.as_bytes()).unwrap();
```
Pada bagian kode ini, saya belajar bagaimana menyusun response HTTP secara manual. Baris let status_line = "HTTP/1.1 200 OK"; digunakan untuk memberi tahu browser bahwa permintaan berhasil diproses. Selanjutnya, fs::read_to_string("hello.html").unwrap(); digunakan untuk membaca isi file HTML yang akan dikirim ke browser, dan contents.len() menghitung panjang isi tersebut sebagai informasi yang akan dimasukkan dalam header. Kemudian, saya membangun string response lengkap menggunakan format!, dengan menyusun status, header Content-Length, dan isi HTML, dipisahkan oleh \r\n sesuai standar HTTP. Akhirnya, response ini dikirim ke browser melalui stream.write_all(...). Proses ini membantu saya memahami bagaimana server merespons permintaan dengan format yang dikenali oleh browser secara low level.<br>

![Commit 2 screen capture](/assets/images/commit2.png)

## Commit 3 Reflection Notes
Pada tahap ini, Web server sekarang sudah bisa memberikan response sesuai dengan statusnya requestnya. Jika user mengirim GET request ke path "/", maka statusnya akan di set 200 OK dan diberikan response html yang sesuai. Dan jika user mengirim GET request selain dari "/" (Path tidak ada), maka Web server akan mengirim response dengan status 404 NOT FOUND, dan mengirim html yang sesuai (404.html).<br>

![Commit 3 screen capture](/assets/images/commit3.png)

Setelah di-refactor
```
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
```
Cara Kerja splittingnya adalah dengan membaca GET requestnya, jika GET requestnya mengarah ke path "/" (hanya ini routing yang disediakan) maka akan set statusnya menjadi 200 OK, dan akan mengirim filename "hello.html". Dan jika GET requestnya selain "/" maka akan di set statusnya 404 NOT FOUND dan filename yang dikirim "404.html".

Sebelum di-refactor
```
    // --snip--
    if request_line == "GET / HTTP/1.1" {
        let status_line = "HTTP/1.1 200 OK";
        let contents = fs::read_to_string("hello.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    } else {
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    }
```
Kenapa perlu di refactor menjadi yang sekarang dipakai? Pertama untuk mengurangi duplikasi kode, pada kode yang belum di refactor, dua blok let status_line ..., let contents ..., let length ..., dan format! ... mirip sepenuhnya, hanya beda isi filenya. Sekarang cukup tulis satu kali saja. sesuai dengan DRY principle: Don't Repeat Yourself. Kemudian, meningkatkan keterbacaan, setelah diRefactor, kode jadi lebih mudah dibaca. Sekarang bisa langsung terlihat, logika if request_line == ... cuma mengatur status dan nama file, sisanya bangun response → kirim. Dan juga menjadi lebih mudah untuk dikemabangkan kedepannya, jika ingin tambah route baru, Tinggal tambahin kondisi di blok if, dan mengubah filenamenya saja.

## Commit 4 Reflection Notes

Kode yang ditambahkan pada function handle_connection.
```
let (status_line, filename) = match &request_line[..] {
    "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
    "GET /sleep HTTP/1.1" => {
        thread::sleep(Duration::from_secs(10));
        ("HTTP/1.1 200 OK", "hello.html")
    }
    _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
};
```

Kode ini mensimulasikan slow response dengan di sleep selama 10 detik ketika user mengakses path atau endpoint /sleep. Karena kita masih menggunakan single-threaded server, server hanya bisa memproses satu request dalam satu waktu. Jika terdapat banyak user yang ingin mengakses servernya, pengguna harus menunggu waktu yang cukup lama untuk mendapatkan response dari server. Salah satu solusi yang bisa diterapkan adalah dengan menggunakan multi-threading.

## Commit 5 Reflection Notes
Pada tahap ini, kita coba mengimplementasikan Threadpool untuk mengubah dari singlethreaded menjadi multithreaded server, untuk mengatasi masalah sebelumnya (pada commit 4). Sebelumnya dengan singlethreaded yang hanya bisa memproses satu request dalam satu waktu. Dengan multithreaded, kita bisa membuat threadpool dengan sejumlah worker yang sudah kita tentukan berjumlah N. Semisal contoh pada sebelumnya terdapat beberapa user yang mengakses endpoint /sleep, maka user harus mengantri untuk diproses requestnya oleh server, tetapi dengan threadpool (multithreaded), request-request dari user dapat di proses secara paralel (berjumlah maksimal N), sehingga bisa membuat server menjadi lebih responsive.
Untuk effisiensi dan thread-safe, kode ini menggunakan `Arc<Mutex<T>>`. Pada contoh ini Arc<Mutex<T>> memastikan bahwa receiver (channel) dapat diakses dengan aman oleh banyak thread. Arc memungkinkan banyak worker memiliki akses shared ke channel receiver, dan Mutex memastikan bahwa hanya satu thread yang dapat mengakses channel untuk mengambil pekerjaan pada satu waktu. Ini menjamin thread-safety dan sinkronisasi antar worker yang menjalankan pekerjaan. Bisa dilihat pada Worker ketika mengambil job, channel receiver harus di lock dulu agar channelnya tidak bisa diakses oleh worker lain pada satu waktu.

Threapool
```
impl ThreadPool {
    // --snip--

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
    // --snip--
}
```

Worker
```
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