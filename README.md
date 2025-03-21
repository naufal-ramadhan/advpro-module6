# Reflection

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