# Reflection

## Commit 1 Reflection Notes
Melalui kode ini, saya mengetahui cara kerja sebuah server HTTP menggunakan Rust. Saya memahami bagaimana TcpListener digunakan untuk menerima koneksi dari client, dan setiap koneksi diproses sebagai TcpStream. Dengan membungkus stream menggunakan BufReader, saya dapat membaca request HTTP baris demi baris hingga menemukan baris kosong sebagai penanda akhir header. Seluruh baris dikumpulkan ke dalam Vec<String> lalu dicetak, sehingga saya bisa melihat isi request dari browser secara mentah. Proses ini memperjelas bagaimana komunikasi HTTP terjadi di level rendah.

