# Mengenal Let's Encrypt 

### Pendahuluan [Let's Encrypt](https://letsencrypt.org/) 
adalah sebuah otoritas penyedia sertifikat totomasi dan terbuka yang menggunakan protokol [ACME (Automatic Certificate Management Environment )](https://ietf-wg-acme.github.io/acme/draft-ietf-acme-acme.html) untuk memberikan layanan *free TLS/SSL certificates* kepada klien yang sesuai. Sertifikat-sertifikat ini dapat dipakai untuk meng-*encrypt* komunikasi antara server web dengan pengguna. Ada banyak jenis klien yang tersedia yang ditulis dalam berbagai bahasa pemrograman dan integrasi dengan berbagai *tool*, *service*, dan server. Klien ACME paling populer adalah, [Certbot](https://certbot.eff.org/) yang kini dikembangkan oleh [the Electronic Frontier Foundation](https://www.eff.org/). Untuk melakukan verifikasi kepemilikan domain dan membaca sertifikat, Certbot dalam secara otomatis mengatur TLS/SSL di server web baik itu Apache maupun Nginx. Tuotorial ini akan membahas tentang otoritas penyedia sertifikat dan bagaimana Let's Encrypt bekerja lalu mereviu klien ACME yang populer. 

## Apa Itu Certificate Authority? 
Certificate authority (CA) adalah suatu badan yang secara kriptografi menandai sertifikat TLS/SSL untuk menandakan bahwa sertifikat tersebut otentik. Browser dan sistem operasi memiliki daftar CA terpercaya untuk melakukan verifikasi sertifikat suatu situs. Sampai beberapa tahun terakhir, sebagian besar CA adalah layanan yang bekerja secara komersil yang meminta uang untuk melakukan verifikasi dan penandaan sertifikat. Let's Encrypt membuat proses ini menjadi gratis bagi pengguna dengan mengotomatiskan prosedurnya. Ia mengandalkan dana dari *sponsorship* dan donasi sebagai sumber penghidupan infrastruktur mereka. Untuk informasi tentang sertifikat dan perbedaan jenis CA, silahkan baca artikel “[A Comparison of Let's Encrypt, Commercial and Private Certificate Authorities, and Self-Signed SSL Certificates](https://www.digitalocean.com/community/tutorials/a-comparison-of-let-s-encrypt-commercial-and-private-certificate-authorities-and-self-signed-ssl-certificates).” Selanjutnya, kita akan melihat bagaimana Let's Encrypt mengotomatiskan verifikasi domain. 

## Bagaimana Let's Encrypt Bekerja 
Protokol ACME milik Let's Encrypt mendefinisikan bagaimana klien berkomunikasi dengan server untuk meminta sertifikat, melakukan verifikasi kepemilikan domain, dan mengunduh sertifikatnya. Ia sedang dalam proses untuk masuk menjadi standar [IETF](https://www.ietf.org/). Let's Encrypt menawarkan sertifikat *domain-validated*, artinya mereka perlu memeriksa bahwa permintaan sertifikat datang dari orang yang memang memiliki domain tersebut. Mereka melakukan hal ini dengan memberikan sebuah token unik kemudian membuat sebuah sebuah request web atau DNS untuk mengambil sebuah key dari token tadi. Contoh, dengan teknik *HTTP challenge*, klien akan mengambil sebuah key dari token lalu menyimpan filenya di server web. Server Let's Encrypt kemudian akan mengambil file tadi di `http://example.com/.well-known/acme-challenge/token`. Jika key ini benar, maka klien diasumsikan memang memiliki domain `example.com`, lalu server akan menandai dan memberikan sebuah sertifikat. Protokol ACME memberikan beberapa langkah yang dapat dilakukan untuk membuktikan kepemilikan domain. *HTTPS challenge* mirip dengan yang di atas, kecuali ia tidak menggunakan file teks. Klien akan menetapkan sebuah sertifikat yang *self-signed* dengan sebuah key di dalamnya. Sedangkan *DNS challenge* akan melihat key di dalam record DNS TXT. 

## Klien Certbot Let's Encrypt Certbot sampai saat ini merupakan klien Let's Encrypt paling populer. 
Aplikasi ini sudah tersedia didistribusi Linux besar dan sudah memiliki pengaturan otomatis untuk Apache dan Nginx. Setelah terpasang, mengambil sertifikat dan mengupdate konfigurasi Apache dapat dilakukan dengan: ``` sudo certbot --apache -d www.example.com ``` Certbot akan menanyakan beberapa pertanyaan, menjalankan *challenge*, mengunduh sertifikat, memperbarui konfigurasi Apache, dan merestart server. Setelah selesai seharusnya kita sudah bisa masuk ke `https://www.example.com` lewat sebuah browser. Gembok berwarna hijau akan muncul yang menandakan sebuah sertifikat valid dan koneksi telah terenkripsi. Karena sertifikat Let's Encrypt hanya valid untuk 90 hari, maka kita perlu mengatur proses perbaruan otomatis. Perintah berikut akan memperbarui semua sertifikat di server: ``` sudo certbot renew ``` Simpan perintah di atas dalam sebuah `crontab` untuk menjalankannya setiap hari dan sertifikat kita akan diperbarui secara otomatis 30 hari sebelum kadaluarsa. Jika sebuah sertifikat dibuat dengan opsi `--apache` atau `--nginx`, Certbot akan merestart server setelah perbaruan sertifikat sukses. Jika ingin mempelajari tentang cron dan crontabs, silahkan baca tutorial “[How To Use Cron To Automate Tasks](https://www.digitalocean.com/community/tutorials/how-to-use-cron-to-automate-tasks-on-a-vps).” 

## Klien Lain Karena protokol ACME itu terbuka dan terdokumentasi dengan baik, banyak klien alternatif lain yang dikembangkan. 
Let's Encrypt menuliskan daftar [ACME clients](https://letsencrypt.org/docs/client-options/) disitusnya. Sebagian besar klien lain tidak memberikan konfigurasi server web sercara otomatis seperti Certbot, tapi mereka mungkin memiliki fitur yang menarik hati pembaca: 
- Ada klien yang ditulis di hampir semua bahasa pemrograman termausk shell script, Go dan Node.js. Klien ini cocok jika pembaca membuat sertifikat dilingkungan yang sangat dibatasi dan tidak ingin memasukkan Pythn serta dependensi Certbot lain. 
- Beberapa client dapat berjalan tanpa akses `root`. Secara umum menjalankan kode dengan akses terbatas lebih baik dibanding menggunakan root. 
- Banyak klien dapat mengotomasi proses *DNS challenge* menggunakan API dari penyedia DNS untuk membuat record TXT yang diperlukan. DNS challenge dapat melakukan *use-case* yang lebih rumit misalnya men-*encrypt* server web yang tidak tersedia secara publik. 
- Beberapa klien terintegrasi dengan server web, *reverse proxy*, atau *load balancer*, sehingga membuatnya lebih mudah dikonfigurasi dan di-*deploy*. Beberapa klien lain yang populer adalah: - [lego](https://github.com/xenolf/lego): Ditulis dengan bahasa Go, lego adalah *one-file binary install* (pemasangannya hanya butuh satu file ini), dan mendukung banyak penyedia layanan DNS saat melakukan DNS challenge. - [acme.sh](https://github.com/Neilpang/acme.sh): acme.sh adalah shell script sederhana yang dapat dijalankan dalam lingkungan bukan root dan juga dapat berinteraksi dengan lebih dari 30 penyedia layanan DNS. - [Caddy](https://caddyserver.com/): Caddy adalah server web utuh yang ditulis dengan Go dengan dukungan langsung untuk Let's Encrypt. Ada banyak klien lain yang tersedia dan server/service lain yang dapat mengotomasi pengaturan TLS/SSL dengan menintegrasikan Let's Encrypt. 

## Penutup Kita telah membahas dasar-dasar Let's Encrypt dan mendiskusikan beberapa software klien yang tersedia. 
Tunggu terjemahan tutorial penggunaan Let's Encrypt untuk server web Apache dan Nginx. *Diterjemahkan dari [An Introduction to Let's Encrypt](https://www.digitalocean.com/community/tutorials/an-introduction-to-let-s-encrypt) di bawah [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-nc-sa/4.0/).*
