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

### Mengapa Risk Storming Diterapkan
 
BidMart adalah platform lelang real-time berbasis microservices. Ketika platform ini sukses dan digunakan oleh ribuan pengguna secara bersamaan, arsitektur awal yang cukup untuk tahap pengembangan akan menghadapi tekanan yang serius. Risk storming diterapkan karena tidak ada satu pun anggota tim yang dapat menilai risiko seluruh sistem sendirian setiap service memiliki karakteristik risiko yang berbeda, dan tanpa evaluasi kolaboratif, risiko tersembunyi baru teridentifikasi ketika sudah berada di lingkungan produksi.
 
---
 
### Risk Assessment: Identifikasi Risiko Utama
 
Risk matrix yang digunakan mengalikan dua dimensi: **dampak** (1=rendah, 2=sedang, 3=tinggi) dan **kemungkinan terjadi** (1=rendah, 2=sedang, 3=tinggi). Skor 1-2 = risiko rendah, 3-4 = sedang, 6-9 = tinggi.
 
#### Tabel Risk Assessment
 
| Kriteria Risiko  | Auth Service | Bidding Service | Wallet Service | Catalog Service | Order Service | Total Risiko |
|------------------|:------------:|:---------------:|:--------------:|:---------------:|:-------------:|:------------:|
| Skalabilitas     | 2            | **6**           | 3              | 4               | 2             | 17           |
| Ketersediaan     | 3            | **9**           | **9**          | 3               | 3             | 27           |
| Performa         | 2            | **9**           | **6**          | 4               | 2             | 23           |
| Keamanan         | **6**        | **6**           | **9**          | 3               | 4             | 28           |
| Integritas Data  | 3            | **6**           | **9**          | 2               | **6**         | 26           |
| **Total Risiko** | **16**       | **36**          | **36**         | **16**          | **17**        |              |
 
---
 
#### Bidding Service
 
| Kriteria        | Skor    | Alasan |
|-----------------|---------|--------|
| Ketersediaan    | 9 (3x3) | Bidding adalah inti dari platform; jika service ini down saat lelang sedang berlangsung, semua transaksi gagal. Lelang real-time tidak toleran terhadap downtime. |
| Performa        | 9 (3x3) | WebSocket harus mengirimkan pembaruan harga secara real-time ke semua peserta lelang. Di bawah beban tinggi, latensi yang meningkat menyebabkan bid dianggap tidak sah karena melewati anti-sniping window. |
| Integritas Data | 6 (3x2) | Bid yang diterima harus direkam secara atomik dan dikirimkan ke Kafka. Kehilangan event `auction.bid-placed` menyebabkan pemenang lelang tidak tercatat. |
| Keamanan        | 6 (3x2) | Tanpa API Gateway, tidak ada satu titik enforcement untuk validasi JWT ada kemungkinan bid dikirimkan dengan token yang dipalsukan. |
 
---
 
#### Wallet Service
 
| Kriteria        | Skor    | Alasan |
|-----------------|---------|--------|
| Keamanan        | 9 (3x3) | Dana pengguna disimpan dan dikelola di sini. Akses tidak sah ke API hold/capture berdampak finansial secara langsung. |
| Integritas Data | 9 (3x3) | Operasi hold-capture-release harus bersifat idempoten. Jika terjadi duplikasi akibat retry tanpa deduplication, saldo pengguna bisa terpotong dua kali. |
| Ketersediaan    | 9 (3x3) | Jika Wallet down saat auction settlement berlangsung, pemenang tidak bisa membayar dan order tidak pernah terbentuk mengakibatkan kehilangan pendapatan secara langsung. |
| Performa        | 6 (3x2) | Ketika banyak lelang selesai secara bersamaan, Wallet menerima lonjakan request `capture`. Tanpa antrian, lonjakan ini dapat menyebabkan service crash. |
 
---
 
### Current Architecture
#### Database Bersama
 
Semua service (Auth, Bidding, Catalog, Wallet, Order, Notification) menggunakan satu instance PostgreSQL/Neon yang sama. Kondisi ini menciptakan dua masalah utama:
 
- **Single point of failure untuk ketersediaan** jika database down, semua service gagal secara bersamaan.
- **Risiko integritas data lintas service** schema yang terlalu berdekatan memungkinkan query dari satu service secara tidak sengaja mempengaruhi tabel milik service lain.
---
 
#### Tidak Ada API Gateway
 
Frontend Next.js memanggil masing-masing service secara langsung. Kondisi ini berarti:
 
- Tidak ada enforcement CORS dan rate limiting secara terpusat.
- Validasi JWT diimplementasikan secara duplikat di masing-masing service.
- Tidak ada satu titik masuk untuk memblokir traffic berbahaya sebelum mencapai service manapun.
---
 
#### Kafka: Single Broker
 
Kafka digunakan untuk event `auction.bid-placed`, `auction.settled`, dan `wallet.hold`. Jika satu-satunya broker ini crash, event tidak terkirim dan sistem kehilangan konsistensi data antarservice.
 
---
 
### Mitigasi Risiko dan Justifikasi Perubahan Arsitektur
 
#### Penambahan API Gateway / BFF
 
**Risiko yang dimitigasi:** Keamanan pada Auth Service (6) dan Bidding Service (6); tidak adanya rate limiting.
 
API Gateway / BFF ditambahkan sebagai single entrypoint untuk semua request yang berasal dari frontend. Gateway ini menangani auth routing terpusat, validasi JWT, CORS policy, dan rate limiting. Dengan adanya komponen ini, masing-masing service tidak perlu lagi mengimplementasikan validasi token secara duplikat. Serangan brute-force juga dapat diblokir di satu titik sebelum sempat mencapai service manapun.
 
---
 
#### Separate Schema per Service di PostgreSQL
 
**Risiko yang dimitigasi:** Ketersediaan 9 (database bersama sebagai single point of failure) dan integritas data lintas service.
 
Database dipecah menjadi schema yang terisolasi per service: `auth_db`, `bidding_db`, `wallet_db`, `catalog_db`, `order_db`, dan `notif_db`. Pendekatan ini memastikan bahwa masalah pada satu schema tidak mengakibatkan cascade failure ke service lain. Selain itu, isolasi schema mencegah query antarservice yang tidak diinginkan, sehingga integritas data masing-masing domain tetap terjaga.
 
---
 
#### Kafka Cluster dengan Replication Factor 3
 
**Risiko yang dimitigasi:** Integritas data pada Bidding Service (6) dan Wallet Service (6) akibat kehilangan event.
 
Kafka dikonfigurasi sebagai cluster dengan minimal 3 broker dan replication factor 3. Konfigurasi ini memastikan bahwa apabila satu broker crash, event `auction.bid-placed`, `auction.settled`, dan `wallet.hold` tetap tersedia di broker lain. Tanpa mitigasi ini, kehilangan event Kafka berarti pemenang lelang tidak tercatat dan proses settlement tidak pernah terjadi sebuah skenario yang berdampak finansial serius bagi pengguna dan platform.
 
---
 
#### CDN / HTTPS Endpoint
 
**Risiko yang dimitigasi:** Ketersediaan dan performa pada Bidding Service dan Auth Service.
 
CDN ditambahkan di depan Next.js Frontend untuk mendistribusikan beban traffic dan mengurangi latensi bagi pengguna yang tersebar secara geografis. CDN juga meningkatkan ketersediaan frontend karena static assets dapat di-cache meskipun origin server sedang mengalami beban tinggi.
 
---
 
#### Monitoring / Logging dan Object Storage
 
**Risiko yang dimitigasi:** Observabilitas untuk semua service mendukung deteksi dini sebelum ketersediaan atau integritas data terdampak.
 
Monitoring terpusat dan centralized logging ditambahkan untuk memantau health, metrics, dan audit trail seluruh service. Object storage digunakan khusus untuk menyimpan gambar listing pada Catalog Service, sehingga beban penyimpanan data biner tidak membebani database relasional.
 
---
 
### Ringkasan
 
| Komponen            | Sebelum (Current Architecture)              | Sesudah (Future Architecture)        |
|---------------------|--------------------------------------------|----------------------------------------|
| Titik masuk         | Frontend langsung ke masing-masing service | API Gateway / BFF                      |
| Database            | Satu instance bersama                      | Schema terpisah per service            |
| Kafka               | Single broker                              | Cluster 3 broker, replication factor 3 |
| Pengiriman frontend | Langsung dari origin server                | CDN / HTTPS endpoint                   |
| Observabilitas      | Tidak ada                                  | Monitoring + Logging terpusat          |
| Penyimpanan file    | Di database atau tidak terstruktur         | Object Storage                         |
 
Risk storming memungkinkan tim mengidentifikasi bahwa **Bidding Service** dan **Wallet Service** adalah titik paling kritis dalam sistem bukan karena kompleksitasnya, melainkan karena dampak langsungnya terhadap pengguna, yaitu dana dan transaksi real-time. Perubahan arsitektur yang dihasilkan bukan hanya mengurangi risiko teknis secara signifikan, tetapi juga membentuk fondasi yang lebih kokoh untuk mendukung skala yang lebih besar seiring pertumbuhan BidMart.


## Pekerjaan Individu

### Lessyarta Kamali Sopamena Pirade (2406356643)

**Deskripsi singkat kontribusi individu (1-2 kalimat):** Saya bertanggung jawab atas pengembangan modul Authentication, termasuk fitur registrasi, login, verifikasi email, reset password, JWT, refresh token, two-factor authentication, manajemen role, dan validasi user. Modul ini menjadi pusat autentikasi dan otorisasi dasar yang digunakan oleh service lain di BidMart.

#### Component Diagram Auth Module

![Individual Component](images/individual_component_lessyarta_auth.png)

#### Code Diagram Auth Module

![Code Diagram](images/individual_code_lessyarta_auth.png)
**Penjelasan:** Auth module mengelola identitas user, login, JWT, refresh token, verifikasi email, reset password, 2FA, role, dan status user. Modul lain memakai Auth untuk validasi user dan authorization context.

**Keterangan Code Diagram Auth:** Code diagram Auth memperlihatkan struktur utama modul Authentication. `AuthController` menjadi entry point untuk operasi user-facing seperti register, login, verifikasi email, reset password, 2FA, refresh token, logout, dan validasi user legacy. `InternalUserController` menjadi entry point internal untuk service lain atau admin process yang perlu membaca, membuat, mengubah, memvalidasi, atau menghapus data user. Kedua controller tersebut tidak langsung mengakses database, tetapi meneruskan request ke `AuthService` sebagai pusat business logic.

`AuthService` mengatur aliran data autentikasi dari request hingga persistence: data registrasi/login divalidasi, password diproses dengan hashing, token JWT dibuat melalui `JwtService`, kode 2FA diproses oleh `TotpService`, dan email verifikasi/reset dikirim lewat `EmailService`. Untuk penyimpanan, `AuthService` memakai repository seperti `UserRepository`, `RoleRepository`, `RefreshTokenRepository`, `EmailVerificationTokenRepository`, dan `PasswordResetTokenRepository`. Interface repository tersebut menjadi boundary ke database sehingga controller tetap bersih dari detail query dan persistence.

Aliran data utamanya adalah: client mengirim request ke controller, controller memanggil `AuthService`, service memvalidasi dan memproses data, service membaca/menulis entity melalui repository, lalu response dikembalikan ke client. Pada operasi tertentu, Auth juga menghasilkan efek samping seperti mengirim email atau menerbitkan event domain, misalnya ketika status user berubah menjadi suspended atau role user berubah.

### Component Diagram Catalog Module

![Individual Component](images/individual_component_catalog.png)

#### Code Diagram Catalog Module

![Code Diagram](images/individual_code_catalog.png)

**Penjelasan:** Catalog module bertanggung jawab atas listing, kategori, gambar, moderasi admin, dan sinkronisasi harga listing setelah ada bid. Catalog menjadi sumber data listing yang dipakai pembeli untuk browsing dan dipakai Bidding untuk memastikan barang masih valid untuk dilelang.

**Keterangan Code Diagram Catalog:** Code diagram Catalog memperlihatkan pembagian komponen berdasarkan jenis akses. `CatalogController` melayani buyer/public untuk browsing catalog dan melihat detail listing. `SellerListingController` melayani seller untuk membuat, mengedit, mengaktifkan, membatalkan, dan menghapus listing miliknya. `AdminModerationController` dan `AdminCategoryController` melayani admin untuk moderasi listing serta pengelolaan kategori. `BidSyncController` adalah endpoint internal yang dipakai Bidding Service untuk validasi listing sebelum bid dan sinkronisasi harga setelah bid.

Business logic utama listing berada di interface `ListingService` dan implementasinya `ListingServiceImpl`. Service ini menangani aturan domain seperti kepemilikan listing oleh seller, validasi kategori, status listing, aktivasi listing, moderasi, dan sinkronisasi `currentPrice` serta `bidCount`. Untuk kategori, `CategoryService` dan `CategoryServiceImpl` memisahkan logika pengelolaan kategori dari logika listing. `ListingImageController` mengatur tambah/hapus gambar listing dan memakai `ListingRepository` serta `ListingImageRepository` untuk memastikan gambar terhubung dengan listing yang benar.

Aliran data utamanya adalah: request dari frontend atau service internal masuk ke controller sesuai use case, lalu controller memanggil service yang relevan. Service melakukan validasi domain dan membaca/menulis data melalui `ListingRepository`, `CategoryRepository`, atau `ListingImageRepository`. Untuk integrasi asynchronous, `CatalogBiddingEventListener` menerima event bid dari Kafka topic `auction.bid-placed`, lalu memanggil `ListingService` untuk memperbarui harga dan jumlah bid pada listing terkait.

### Bermulya Anugrah Putra (2406347424)

**Deskripsi singkat kontribusi individu (1-2 kalimat):** Saya bertanggung jawab atas pengembangan modul Bidding, yang menjadi *core engine* untuk mekanisme lelang *real-time*. Kontribusi ini mencakup pemrosesan penawaran (*bid*), sinkronisasi harga melalui WebSocket (STOMP), penanganan konkurensi dengan *pessimistic locking*, penerapan lapisan idempotensi untuk mencegah duplikasi *request*, serta penjadwalan asinkron untuk penutupan lelang otomatis.

#### Component Diagram Bidding Module

![Component Diagram Bidding](images/component_diagram_bidding.png)

#### Code Diagram Bidding Module

![Code Diagram Bidding](images/code_diagram_bidding.png)

**Penjelasan:** Bidding module mengelola seluruh siklus hidup penawaran lelang secara komprehensif, mulai dari inisiasi lelang, penerimaan *bid* secara *real-time*, validasi nominal dan penahanan saldo (*hold funds*), perpanjangan waktu otomatis (*anti-sniping*), hingga penentuan pemenang. Modul ini beroperasi dengan dependensi yang kuat terhadap integrasi *service* lain (Auth, Catalog, Wallet) guna menjaga konsistensi state dan keabsahan transaksi.

**Keterangan Code Diagram Bidding:** Code diagram Bidding mendemonstrasikan arsitektur berorientasi lapisan (*layered architecture*) dengan pemisahan tanggung jawab yang terstruktur. `BiddingController` bertindak sebagai *entry point* untuk *request* REST HTTP sekaligus melayani koneksi STOMP WebSocket. Secara krusial, *controller* ini mengimplementasikan mekanisme pertahanan awal (*idempotency*) dengan mengakses `IdempotencyRecordRepository` secara langsung untuk memblokir anomali duplikasi *request sebelum instruksi diteruskan ke *service layer*.

Logika bisnis utama dienkapsulasi di dalam `BiddingServiceImpl`. *Service* ini mengorkestrasi validasi kompleks, memanfaatkan *pessimistic locking* pada `AuctionRepository` guna memitigasi *race condition*, serta menentukan apakah sebuah instruksi penawaran berstatus menang atau kalah secara algoritmik. Selama proses ini, *service* secara aktif berkomunikasi dengan modul eksternal melalui antarmuka *client* (`CatalogClient`, `UserClient`, `WalletClient`). Untuk mengakomodasi dinamika lelang, `BiddingScheduler` didesain untuk dieksekusi secara periodik guna mengevaluasi dan menutup lelang yang telah melewati tenggat waktu. Setiap transisi status yang definitif (seperti adanya *high bid* baru atau penutupan lelang) akan memicu penerbitan *event* spesifik (misal: `BidPlacedEvent` atau `WinnerDeterminedEvent`) melalui *Spring ApplicationEvent*, mendemonstrasikan sifat *decoupled* dalam distribusi notifikasi internal sistem.




### Jessica Tandra (2406355445)

#### Component Diagram Notification Module
```mermaid
flowchart LR
    %% External actors/systems
	NextFE[Next.js Frontend\nBidMart web app]:::external
	AuthService[Auth Service\nset/propagate X-User-Id]:::external
	BiddingService[Bidding Service\ninternal notification caller]:::external
	OrderService[Order Service\ninternal notification caller]:::external
	Postgres[(PostgreSQL / Neon)]:::db

    %% Module boundary
    subgraph Module[Notification Module - Spring Boot]
        direction TB

        subgraph API[Controllers]
            NC[NotificationController\n/api/v1/notifications]
            INC[InternalNotificationController\n/internal/v1/notifications]
        end

        subgraph SEC[Security and Transport]
            STF[ServiceTokenFilter\nvalidate X-Service-Token\nfor /internal/*]
            WSC[WebSocketConfig\n/ws/notifications\n/user prefix]
            WAI[WebSocketAuthInterceptor\nread STOMP CONNECT\nX-User-Id to Principal]
        end

        subgraph APP[Application Service]
            NSI[NotificationServiceImpl]
        end

        subgraph EV[Event Listeners]
            BPEL[BidPlacedEventListener]
            WDEL[WinnerDeterminedEventListener]
            AUEL[AuctionUnsoldEventListener]
        end

        subgraph PERSIST[Persistence]
            NR[NotificationRepository]
            NPR[NotificationPreferenceRepository]
            N[(notifications table)]
            NP[(notification_preferences table)]
        end

        subgraph RT[Realtime Delivery]
            SMT[SimpMessagingTemplate]
            UQ[/user/queue/notifications/]
        end
    end

    %% External -> module
	NextFE -->|Notification API + WebSocket| NC
	AuthService -->|Auth context: X-User-Id| NC
	BiddingService -->|POST + X-Service-Token| INC
	OrderService -->|POST + X-Service-Token| INC

    %% Security and control flow
    INC --> STF
    STF -->|authorized| NSI
    NC -->|get/mark read/preferences| NSI
    WSC --> WAI

    %% Event-driven flow
    BPEL -->|SaveNotification| NSI
    WDEL -->|SaveNotification| NSI
    AUEL -->|SaveNotification| NSI

    %% Service -> persistence
    NSI --> NR
    NSI --> NPR
    NR --> N
    NPR --> NP
    N --> Postgres
    NP --> Postgres

    %% Service -> realtime
    NSI --> SMT
    SMT --> UQ
	UQ --> NextFE

    classDef external fill:#f7f7f7,stroke:#555,stroke-width:1px,color:#111;
    classDef db fill:#eef7ff,stroke:#1d4e89,stroke-width:1px,color:#111;
```

Penjelasan:
Notification module bertugas mengelola seluruh siklus hidup notifikasi pada BidMart secara end-to-end, mulai dari pembuatan notifikasi dari event internal, penyimpanan ke database, pengambilan daftar notifikasi oleh user, penandaan notifikasi sebagai sudah dibaca, pengelolaan preferensi notifikasi, hingga pengiriman notifikasi secara real-time ke frontend melalui WebSocket. Modul ini juga menjadi jembatan antara beberapa service domain lain seperti Bidding Service dan Order Service melalui endpoint internal yang diamankan dengan service token, serta mengandalkan konteks user ID dari layer autentikasi/gateway untuk memastikan notifikasi selalu terasosiasi ke pengguna yang tepat. Secara arsitektural, modul ini menggabungkan pendekatan REST, event-driven processing, persistence layer, dan realtime messaging dalam satu bounded context yang terfokus pada distribusi notifikasi.

Keterangan Component Diagram Notification:
Component diagram Notification menunjukkan struktur internal modul dengan pemisahan tanggung jawab yang jelas. NotificationController menjadi pintu masuk untuk request user-facing pada endpoint /api/v1/notifications, sedangkan InternalNotificationController menangani request internal dari service lain pada endpoint /internal/v1/notifications. ServiceTokenFilter berperan sebagai lapisan proteksi awal untuk memastikan hanya caller internal yang sah yang bisa mengakses endpoint internal melalui header X-Service-Token. Di sisi realtime, WebSocketConfig dan WebSocketAuthInterceptor mengatur koneksi STOMP/SockJS ke endpoint /ws/notifications, sekaligus memetakan header X-User-Id menjadi principal agar notifikasi bisa dikirim spesifik ke user tujuan melalui destination /user/queue/notifications.

Logika bisnis inti berada di NotificationServiceImpl. Service ini mengorkestrasi pengambilan notifikasi terpaginasikan, validasi kepemilikan saat marking read, pembaruan massal status baca, penyimpanan notifikasi baru, serta pengelolaan preferensi email dan push notification. Di bawahnya, NotificationRepository dan NotificationPreferenceRepository menangani akses data ke tabel notifications dan notification_preferences. Aliran event-driven juga terlihat jelas melalui BidPlacedEventListener, WinnerDeterminedEventListener, dan AuctionUnsoldEventListener yang masing-masing membangun payload SaveNotification lalu menyerahkannya ke service. Dengan begitu, modul ini memperlihatkan arsitektur yang decoupled: event domain memicu notifikasi tanpa harus mengetahui detail persistence maupun realtime delivery.

#### Code Diagram Notification Module

```mermaid
classDiagram
		class NotificationService {
			<<interface>>
			+getNotifications(userId UUID, isRead Boolean, page int, size int) NotificationListResponse
			+markAsRead(notificationId UUID, userId UUID) void
			+markAllAsRead(userId UUID) void
			+saveNotification(notification SaveNotification) NotificationSaveResponse
			+pushToUser(userId UUID, notification NotificationResponse) void
			+getPreferences(userId UUID) NotificationPreferenceResponse
			+updatePreferences(userId UUID, request UpdateNotificationPreferenceRequest) NotificationPreferenceResponse
		}

		class NotificationServiceImpl {
			-notificationRepository NotificationRepository
			-objectMapper ObjectMapper
			-messagingTemplate SimpMessagingTemplate
			-preferenceRepository NotificationPreferenceRepository
			+getNotifications(userId UUID, isRead Boolean, page int, size int) NotificationListResponse
			+markAsRead(notificationId UUID, userId UUID) void
			+markAllAsRead(userId UUID) void
			+saveNotification(dto SaveNotification) NotificationSaveResponse
			+pushToUser(userId UUID, notification NotificationResponse) void
			+getPreferences(userId UUID) NotificationPreferenceResponse
			+updatePreferences(userId UUID, request UpdateNotificationPreferenceRequest) NotificationPreferenceResponse
			-defaultPreference(userId UUID) NotificationPreference
			-toResponseDTO(notification Notification) NotificationResponse
			-toPreferenceResponse(pref NotificationPreference) NotificationPreferenceResponse
		}

		class NotificationRepository {
			<<repository>>
			+findByUserId(userId UUID, pageable Pageable) Page~Notification~
			+findByUserIdAndIsRead(userId UUID, isRead Boolean, pageable Pageable) Page~Notification~
			+countByUserIdAndIsRead(userId UUID, isRead Boolean) long
			+markAllAsReadByUserId(userId UUID) void
		}

		class NotificationPreferenceRepository {
			<<repository>>
			+findById(userId UUID) Optional~NotificationPreference~
			+save(pref NotificationPreference) NotificationPreference
		}

		class ObjectMapper
		class SimpMessagingTemplate
		class Notification
		class NotificationPreference
		class NotificationListResponse
		class NotificationResponse
		class NotificationSaveResponse
		class SaveNotification
		class UpdateNotificationPreferenceRequest

		NotificationServiceImpl ..|> NotificationService
		NotificationServiceImpl --> NotificationRepository
		NotificationServiceImpl --> NotificationPreferenceRepository
		NotificationServiceImpl --> ObjectMapper
		NotificationServiceImpl --> SimpMessagingTemplate
		NotificationServiceImpl --> Notification
		NotificationServiceImpl --> NotificationPreference
		NotificationServiceImpl --> NotificationListResponse
		NotificationServiceImpl --> NotificationResponse
		NotificationServiceImpl --> NotificationSaveResponse
		NotificationServiceImpl --> SaveNotification
		NotificationServiceImpl --> UpdateNotificationPreferenceRequest
```

##### Class Diagram Entities and Persistence

```mermaid
classDiagram
		class Notification {
			+UUID id
			+UUID userId
			+NotificationType type
			+String title
			+String message
			+String data
			+Boolean isRead
			+LocalDateTime readAt
			+UUID relatedAuctionId
			+UUID relatedOrderId
			+LocalDateTime createdAt
			+LocalDateTime updatedAt
			+prePersist() void
			+preUpdate() void
		}

		class NotificationPreference {
			+UUID userId
			+boolean emailBidPlaced
			+boolean emailOutbid
			+boolean emailAuctionWon
			+boolean emailOrderUpdate
			+boolean pushBidPlaced
			+boolean pushOutbid
			+boolean pushAuctionWon
			+boolean pushOrderUpdate
		}

		class NotificationType {
			<<enumeration>>
			BID_PLACED
			OUTBID
			AUCTION_WON
			AUCTION_LOST
			AUCTION_EXTENDED
			AUCTION_ENDED
			ORDER_CREATED
			ORDER_SHIPPED
			ORDER_COMPLETED
			DISPUTE_CREATED
			DISPUTE_RESOLVED
			WALLET_UPDATED
		}

		class NotificationRepository {
			<<repository>>
			+findByUserId(userId UUID, pageable Pageable) Page~Notification~
			+findByUserIdAndIsRead(userId UUID, isRead Boolean, pageable Pageable) Page~Notification~
			+countByUserIdAndIsRead(userId UUID, isRead Boolean) long
			+markAllAsReadByUserId(userId UUID) void
		}

		class NotificationPreferenceRepository {
			<<repository>>
			+findById(userId UUID) Optional~NotificationPreference~
			+save(pref NotificationPreference) NotificationPreference
		}

		Notification --> NotificationType : uses
		NotificationRepository --> Notification : persists
		NotificationPreferenceRepository --> NotificationPreference : persists
```

##### Sequence Internal Notification + Realtime Push

```mermaid
sequenceDiagram
		autonumber
		participant BS as Bidding Service
		participant OS as Order Service
		participant STF as ServiceTokenFilter
		participant INC as InternalNotificationController
		participant NSI as NotificationServiceImpl
		participant NR as NotificationRepository
		participant SMT as SimpMessagingTemplate
		participant WS as /user/queue/notifications
		participant FE as Next.js Frontend

		par request from bidding domain
				BS->>STF: POST /internal/v1/notifications\nHeader: X-Service-Token\nBody: SaveNotification
		and request from order domain
				OS->>STF: POST /internal/v1/notifications\nHeader: X-Service-Token\nBody: SaveNotification
		end
		alt token invalid
				STF-->>BS: 401 Unauthorized
				STF-->>OS: 401 Unauthorized
		else token valid
				STF->>INC: forward request
				INC->>NSI: saveNotification(request)
				NSI->>NR: save(notification entity)
				NR-->>NSI: saved notification (id, createdAt)
				NSI->>NSI: toResponseDTO(saved)
				NSI->>SMT: convertAndSendToUser(userId, /queue/notifications, payload)
				SMT-->>WS: STOMP user message
				WS-->>FE: realtime notification event
				NSI-->>INC: NotificationSaveResponse
				INC-->>BS: 201 Created\n{notificationId, createdAt}
				INC-->>OS: 201 Created\n{notificationId, createdAt}
		end
```

##### Sequence Event-Driven Flow (BidPlacedEvent)

```mermaid
sequenceDiagram
		autonumber
		participant AP as Event Publisher (Auction/Bidding)
		participant BPEL as BidPlacedEventListener
		participant NSI as NotificationServiceImpl
		participant NR as NotificationRepository
		participant SMT as SimpMessagingTemplate
		participant WS as /user/queue/notifications

		AP->>BPEL: publish BidPlacedEvent(auctionId, sellerId, newBidAmount, outbidUserId)

		alt outbidUserId != null
				BPEL->>NSI: saveNotification(OUTBID for outbid user)
				NSI->>NR: save(notification)
				NR-->>NSI: saved
				NSI->>SMT: convertAndSendToUser(outbidUserId, /queue/notifications, payload)
				SMT-->>WS: push OUTBID notification
		end

		BPEL->>NSI: saveNotification(BID_PLACED for seller)
		NSI->>NR: save(notification)
		NR-->>NSI: saved
		NSI->>SMT: convertAndSendToUser(sellerId, /queue/notifications, payload)
		SMT-->>WS: push BID_PLACED notification
```

Keterangan Code Diagram Notification:
Code diagram Notification memperlihatkan implementasi layered architecture yang cukup ketat. NotificationServiceImpl menjadi pusat orkestrasi logika bisnis, bergantung pada NotificationRepository untuk akses data notifikasi, NotificationPreferenceRepository untuk preferensi user, ObjectMapper untuk serialisasi/deserialisasi field data berbentuk JSON, dan SimpMessagingTemplate untuk pengiriman pesan ke WebSocket user queue. Alur saveNotification tidak hanya menyimpan notifikasi ke database, tetapi juga langsung memicu pushToUser agar notifikasi muncul real-time di frontend. Ini menunjukkan bahwa service di modul ini bukan sekadar persistence service, melainkan juga delivery orchestrator.

Pada layer model, Notification merepresentasikan entitas utama notifikasi dengan atribut seperti userId, type, title, message, data, isRead, readAt, createdAt, updatedAt, serta relasi data tambahan seperti relatedAuctionId dan relatedOrderId. NotificationPreference menyimpan konfigurasi channel notifikasi per user, sementara NotificationType mendefinisikan jenis-jenis notifikasi yang bisa dihasilkan sistem. Repository interface menyediakan query spesifik seperti filter berdasarkan user dan status baca, count unread, dan mark all as read. Secara keseluruhan, code diagram ini menegaskan bahwa modul Notification diimplementasikan sebagai kombinasi antara persistence, event handling, security boundary internal, dan realtime messaging yang saling terintegrasi.


#### Component Diagram Order Module
```mermaid
%% C4 Level 3 — Order Module (detailed component diagram)
flowchart TB
  subgraph Frontend["Frontend / Gateway"]
    FE["Next.js Frontend"]
  end

  subgraph Controllers["Controllers"]
    OC["OrderController\nGET /api/v1/orders\nGET /api/v1/orders/{id}"]
    AC["AdminOrderController\nPUT /orders/{id}/ship\nPOST /orders/{id}/dispute"]
    IC["InternalOrderController\nPOST /internal/orders/webhook"]
  end

  subgraph Services["Application Services"]
    OSI["OrderServiceImpl\norchestrator,\ntransactions"]
  end

  subgraph Repositories["Persistence Layer"]
    OR["OrderRepository\nJPA queries"]
  end

  subgraph Database["Data Store"]
    DB["PostgreSQL\norderstable\nauctionId, buyerId,\nshippingAddress,\nstatus, dispute"]
  end

  subgraph External["External Services"]
    AUC["Auction Service\norder create webhook"]
    NS["Notification Service\norder events"]
  end

  FE -->|GET orders, ship| OC
  FE -->|PUT dispute| AC
  AUC -->|webhook POST| IC

  OC --> OSI
  AC --> OSI
  IC --> OSI

  OSI -->|save/find Order| OR
  OR -->|SQL JPA| DB

  OSI -->|publish events| NS
  NS -->|callbacks| FE
```

Component diagram mendemonstrasikan arsitektur berorientasi lapisan (layered architecture) untuk Order Module dengan pemisahan tanggung jawab yang terstruktur. Diagram menampilkan tiga lapisan utama: Frontend/Gateway (Next.js Frontend), Controllers (entry point REST dan webhook), Application Services (business logic orchestration), dan Persistence Layer (database abstraction melalui JPA).

OrderController dan AdminOrderController bertindak sebagai entry point untuk request REST HTTP, sekaligus menangani koordinasi dengan OrderServiceImpl untuk fulfillment logic. InternalOrderController secara khusus menerima webhook dari Auction Service sebagai trigger untuk order creation flow, memastikan order dibuat segera setelah auction ditentukan pemenangnya. 

OrderServiceImpl adalah komponen inti yang mengorkestra seluruh order lifecycle: validasi state sebelum transition (misal, tidak bisa confirm receipt jika status bukan SHIPPED), akselerasi persisten Order entity melalui OrderRepository, dan publikasi events ke Notification Service untuk setiap transisi definitif (order.created, order.shipped, order.completed, dispute.created, dispute.resolved). Semua operasi service-level dilindungi dengan @Transactional boundary untuk menjamin atomic consistency terhadap database.

OrderRepository mengabstraksi JPA queries dan persisten Order entity ke PostgreSQL dengan schema orders table yang merepresentasikan complete order state (auctionId, buyerId, shippingAddress detail, courier tracking, status enum, dan dispute metadata). External dependencies (Auction Service dan Notification Service) terintegrasi melalui event-driven architecture: Auction Service publish event → InternalOrderController → OrderServiceImpl → notify Notification Service.

#### Code Diagram Order Module
##### OrderService Implementation
```mermaid
%% OrderServiceImpl — mapped to actual implementation
classDiagram
  class OrderServiceImpl {
    -OrderRepository orderRepository
    -NotificationService notificationService
    +OrderListResponse getOrders(UUID userId, String role, String status, int page, int size)
    +OrderResponse getOrderById(UUID orderId, UUID userId)
    +void createOrderFromEvent(CreateOrder dto)
    +void updateShipping(UUID orderId, UUID userId, UpdateShippingRequest request)
    +void confirmReceipt(UUID orderId, UUID userId)
    +void createDispute(UUID orderId, UUID userId, DisputeRequest request)
    +void resolveDispute(UUID orderId, ResolveDisputeRequest request)
  }

  class OrderRepository
  class NotificationService

  OrderServiceImpl --> OrderRepository
  OrderServiceImpl --> NotificationService
```

##### Order Entity
```mermaid
%% Order entity mapped to src/.../model/Order.java
classDiagram
  class Order {
    +UUID id
    +UUID auctionId
    +UUID listingId
    +String listingTitle
    +String listingImageUrl
    +UUID buyerId
    +String buyerDisplayName
    +String shippingStreet
    +String shippingCity
    +String shippingProvince
    +String shippingPostalCode
    +String courier
    +String trackingNumber
    +LocalDateTime shippedAt
    +UUID sellerId
    +String sellerDisplayName
    +OrderStatus status
    +Integer totalAmount
    +LocalDateTime createdAt
    +LocalDateTime updatedAt
    +String disputeReason
    +String disputeDescription
  }

  %% relationships: OrderRepository persists Order to orders table
```


##### DTOs / Responses
```mermaid
%% DTOs mapped to src/.../dto/OrderResponse.java
classDiagram
  class OrderResponse {
    +UUID id
    +UUID auctionId
    +ListingDTO listing
    +Integer amount
    +BuyerDTO buyer
    +SellerDTO seller
    +String status
    +ShippingDTO shipping
    +List~TimelineDTO~ timeline
    +LocalDateTime createdAt
  }

  class ShippingAddressDTO {
    +String street
    +String city
    +String province
    +String postalCode
  }

  class BuyerDTO { +UUID id; +String displayName; +ShippingAddressDTO shippingAddress }
  class ListingDTO { +UUID id; +String title; +List~String~ images }
  class ShippingDTO { +String courier; +String trackingNumber; +LocalDateTime shippedAt }

  OrderResponse --> ListingDTO
  OrderResponse --> BuyerDTO
  OrderResponse --> SellerDTO
```

##### Sequence: Order Creation (event-driven)
```mermaid
%% Sequence — createOrderFromEvent -> persist -> notify
sequenceDiagram
  participant EventProducer
  participant OrderServiceImpl
  participant OrderRepository
  participant NotificationService

  EventProducer->>OrderServiceImpl: CreateOrder DTO (from auction/listing)
  OrderServiceImpl->>OrderRepository: save(new Order status=PENDING)
  OrderRepository-->>OrderServiceImpl: persisted(orderId)
  OrderServiceImpl->>NotificationService: saveNotification(ORDER_CREATED)
  NotificationService-->>OrderServiceImpl: ok
  OrderServiceImpl-->>EventProducer: publish(order.created)

  Note over OrderServiceImpl,OrderRepository: Reservation and payment are handled externally in this codebase
```


##### Dispute Creation & Resolution
```mermaid
%% Sequence — createDispute + resolveDispute (matches OrderServiceImpl)
sequenceDiagram
  participant User
  participant OrderController
  participant OrderServiceImpl
  participant OrderRepository
  participant NotificationService

  User->>OrderController: POST /orders/{id}/dispute
  OrderController->>OrderServiceImpl: createDispute(orderId, request)
  OrderServiceImpl->>OrderRepository: findById(orderId)
  OrderRepository-->>OrderServiceImpl: order
  OrderServiceImpl->>OrderRepository: update(order.disputeReason, disputeDescription, status=DISPUTED)
  OrderServiceImpl->>NotificationService: saveNotification(DISPUTE_CREATED)
  OrderServiceImpl-->>User: 200 Accepted

  Note over OrderServiceImpl,OrderRepository: Admin resolves dispute via resolveDispute(), updates status and notifies
```