# Bidmart Software Architecture

# Kelompok B-02

## Daftar Anggota

| NPM          | Nama                                   |
|--------------|----------------------------------------|
| 2306275941   | Ahmad Aqeel Saniy                      |
| 2406347424   | Bermulya Anugrah Putra                 |
| 2406355445   | Jessica Tandra                         |
| 2406356643   | Lessyarta Kamali Sopamena Pirade       |
| 2406407796   | Naila Khadijah                         |


## Current Architecture

### 1. Context Diagram
 
![Context Diagram BidMart](images/current_context.jpg)
 
Context Diagram BidMart menggambarkan sistem secara keseluruhan dari sudut pandang paling tinggi (Level 1 C4 Model). Diagram ini menunjukkan siapa saja yang berinteraksi dengan sistem BidMart dan sistem eksternal apa saja yang terlibat.
 
**Aktor (Person):**
 
| Aktor  | Peran |
|--------|-------|
| **Seller** | Mendaftarkan barang dan mengelola listing lelang di dalam sistem BidMart. |
| **Buyer** | Melakukan bidding, top-up saldo, dan menyelesaikan transaksi pembelian. |
| **Admin** | Memantau aktivitas platform dan melakukan moderasi konten serta manajemen sistem. |
 
**Sistem Eksternal:**
 
| Sistem Eksternal | Interaksi |
|------------------|-----------|
| **SMTP / Email Service** | Menerima permintaan dari BidMart System untuk mengirimkan email verifikasi akun dan reset password kepada pengguna. Komunikasi menggunakan protokol SMTP. |
| **PostgreSQL / Neon** | Digunakan sebagai penyimpanan data utama aplikasi. BidMart System membaca dan menulis data melalui JDBC/JPA. |
 
**Alur Utama:**
- Seller berinteraksi dengan BidMart System untuk mengelola listing lelang.
- Buyer berinteraksi dengan BidMart System untuk melakukan bidding dan transaksi.
- Admin berinteraksi dengan BidMart System untuk memantau dan memoderasi platform.
- BidMart System mengirimkan notifikasi email melalui SMTP / Email Service.
- BidMart System menyimpan dan membaca seluruh data aplikasi ke PostgreSQL / Neon.
---
 
### 2. Container Diagram
 
![Container Diagram BidMart](images/current_container.jpg)
 
 
Container Diagram BidMart menggambarkan arsitektur internal sistem pada Level 2 C4 Model. Diagram ini memperlihatkan semua container (aplikasi yang dapat berjalan secara independen) yang membentuk BidMart Application, beserta cara masing-masing container berkomunikasi satu sama lain.
 
**Daftar Container:**
 
| Container | Teknologi | Tanggung Jawab |
|-----------|-----------|----------------|
| **Next.js Frontend** | Next.js | UI aplikasi web BidMart yang diakses pengguna melalui HTTPS. Menjadi titik masuk utama bagi Buyer, Seller, dan Admin. |
| **Auth Service** | Spring Boot :8081 | Menangani registrasi, login, JWT, 2FA, dan verifikasi email pengguna. |
| **Bidding Service** | Spring Boot :8082 | Mengelola sesi lelang, proses bidding, dan komunikasi real-time melalui WebSocket. |
| **Catalog Service** | Spring Boot :8083 | Mengelola listing barang, kategori, dan moderasi konten. |
| **Wallet Service** | Spring Boot (port dinamis) | Mengelola saldo pengguna, operasi hold, capture, release, dan audit transaksi keuangan. |
| **Order Service** | Spring Boot :8086 | Menangani proses fulfillment order, pengiriman, dan penyelesaian sengketa. |
| **Notification Service** | Spring Boot :8085 | Mengirimkan notifikasi kepada pengguna melalui WebSocket push maupun API. |
 
**Sistem Eksternal yang Terhubung:**
 
| Sistem | Peran |
|--------|-------|
| **Kafka** | Message broker yang menerima dan mendistribusikan event seperti `bid-placed` dan `auction/user event` antara Bidding Service, Catalog Service, dan Wallet Service secara asinkron. |
| **SMTP / Email Service** | Menerima permintaan dari Auth Service untuk mengirimkan email verifikasi dan reset password. |
| **PostgreSQL / Neon** | Database utama yang diakses oleh semua service menggunakan JPA untuk menyimpan data aplikasi. |
 
**Alur Komunikasi Utama:**
- User mengakses aplikasi melalui **Next.js Frontend** via HTTPS.
- Frontend berkomunikasi ke **Bidding Service** melalui Bidding API / WebSocket untuk fitur lelang real-time.
- Frontend berkomunikasi ke **Notification Service** melalui Notification API / WebSocket untuk notifikasi langsung.
- Frontend mengakses **Catalog Service** melalui Catalog API dan **Wallet Service** melalui Wallet API.
- **Bidding Service** memvalidasi listing ke Catalog Service (HTTP) dan memvalidasi user ke Auth Service (HTTP).
- **Bidding Service** berkomunikasi ke **Order Service** melalui Order API dan Auth API.
- **Order Service** membuat notifikasi ke **Notification Service** melalui HTTP.
- **Wallet Service** dan **Catalog Service** mengonsumsi event dari **Kafka**.
- Semua service menyimpan data ke **PostgreSQL / Neon** menggunakan JPA.
---
 
### Deployment Diagram
 
![Deployment Diagram BidMart](images/current_deployment.jpg)

Deployment Diagram BidMart menggambarkan bagaimana seluruh komponen sistem di-deploy ke infrastruktur nyata, mulai dari mesin pengembang hingga lingkungan produksi di AWS.
 
**Lingkungan Developer Machine:**
 
| Komponen | Keterangan |
|----------|------------|
| **Source Code** | Kode sumber frontend dan semua backend microservice yang dikerjakan oleh developer. |
| **Local Runtime** | Lingkungan lokal untuk menjalankan Next.js dev server, Spring Boot, H2/PostgreSQL saat pengembangan. |
 
**Alur CI/CD:**
 
1. Developer melakukan `git push` ke **BidMart Repositories** (GitHub), yang terdiri dari repository: BidMart, Auth, Bidding, Catalog, Wallet, Order, dan Notification.
2. Push ke repository memicu **GitHub Actions** yang menjalankan proses build, test, coverage, dan SonarCloud scan.
3. Hasil analisis kualitas kode dikirimkan ke **SonarCloud** sebagai external system pemantau kualitas kode.
4. Artifact hasil build di-deploy ke lingkungan produksi di **AWS**.
**Lingkungan Produksi (AWS):**
 
| Komponen | Keterangan |
|----------|------------|
| **Next.js Frontend** | Runtime aplikasi web yang melayani request pengguna melalui HTTPS. Berkomunikasi ke backend melalui REST / WebSocket. |
| **Spring Boot Services** | Semua microservice (Auth, Bidding, Catalog, Wallet, Order, Notification) berjalan sebagai runtime di AWS. |
| **PostgreSQL / Neon** | Database produksi yang dikonfigurasi melalui environment variable `DATABASE_URL`. |
| **Kafka** | Event broker produksi untuk mendistribusikan event antara Bidding Service, Catalog Service, dan Wallet Service. Terhubung ke Spring Boot Services melalui Kafka bootstrap server. |
| **SMTP / Email Service** | Layanan eksternal untuk pengiriman email verifikasi dan reset password dari Auth Service. |
 
**Alur Deployment Produksi:**
- Spring Boot Services berkomunikasi ke PostgreSQL / Neon melalui JDBC/JPA.
- Spring Boot Services terhubung ke Kafka melalui Kafka bootstrap server.
- Spring Boot Services mengirimkan email melalui SMTP ke Email Service.
- Next.js Frontend berkomunikasi ke semua Spring Boot Services melalui REST dan WebSocket.


## Future Architecture

**Ringkasan**:
Arsitektur G.2 memperbaiki beberapa risiko dari arsitektur awal:

1) Frontend saat ini memakai satu `NEXT_PUBLIC_API_BASE_URL`, sedangkan backend sudah dipisah menjadi banyak service. Tanpa gateway, frontend harus tahu semua alamat service dan integrasi mudah rusak saat deployment. Karena itu G.2 menambahkan API Gateway/BFF sebagai satu pintu masuk untuk REST dan WebSocket.

2) Komunikasi event masih belum konsisten. Bidding memakai Spring application event untuk `BidPlacedEvent`, `WinnerDeterminedEvent`, dan `AuctionUnsoldEvent`, sementara Catalog dan Wallet sudah menunggu Kafka topic seperti `auction.bid-placed`, `auction.settled`, dan `auction.unsold`. Pada runtime microservice terpisah, Spring application event tidak akan menyeberang proses. Karena itu G.2 menetapkan Kafka sebagai event backbone lintas service, sedangkan Spring event hanya boleh dipakai untuk event internal dalam satu service.

3) Konfigurasi port dan internal endpoint perlu distandardisasi. Bidding memakai port `8082`, Catalog `8083`, Notification `8085`, Order `8086`, tetapi Wallet default-nya juga `8082` walaupun diagram awal menaruh Wallet di `8084`. Pada G.2 Wallet distandardisasi ke `8084`, semua internal endpoint diamankan memakai `X-Service-Token`, dan service tidak diakses langsung oleh user.

### Future Context Diagram
![Future Context Diagram](images/future_context.png)
**Penjelasan:** Future context diagram kami menunjukkan batas sistem paling luar. Aktor utama adalah Buyer, Seller, dan Admin. Semua aktor hanya berinteraksi dengan BidMart System melalui HTTPS dan WebSocket. Database, Kafka, SMTP, object storage, dan monitoring dianggap external system karena berada di luar kode aplikasi utama tetapi dibutuhkan agar sistem berjalan.

### Future Container Diagram
![Future Container Diagram](images/future_container.png)
**Penjelasan:** Future container diagram kami menambahkan API Gateway/BFF agar frontend tidak perlu langsung memanggil enam backend service. Gateway juga menjadi tempat konsisten untuk routing, CORS, rate limit, dan validasi JWT awal. Tiap backend tetap bertanggung jawab atas domainnya sendiri. Kafka menjadi jalur async resmi, sedangkan REST dipakai untuk query/command sinkron seperti validasi user, validasi listing, dan hold saldo.

### Future Deployment Diagram
![Future Deployment Diagram](images/future_deployment.png)
**Penjelasan:** Future deployment diagram kami memperlihatkan alur dari developer ke repository, CI/CD, lalu runtime cloud. Semua service backend berjalan sebagai runtime terpisah. Database, Kafka, SMTP, object storage, dan observability berada sebagai managed external dependency agar backend dapat diskalakan dan dirilis secara terpisah.

## Risk Mitigation

### Hasil Risk Storming (Contoh)


### Tindakan Prioritas



## Pekerjaan Individu

Nama: _Isi nama Anda di sini_

**Deskripsi singkat kontribusi individu (1-2 kalimat).**

### Component Diagram

![Individual Container](images/individual_container_yourname.png)

### Code Diagram

Untuk setiap diagram kode, sertakan gambar dan keterangan singkat (module/class, interface, aliran data).

![Code Diagram 1](images/individual_code1_yourname.png)
![Code Diagram 2](images/individual_code2_yourname.png)