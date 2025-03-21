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