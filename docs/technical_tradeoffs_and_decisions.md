# 🧠 Keputusan Teknis & Trade-Off Arsitektur Aplikasi

Dokumen ini mendokumentasikan analisis mendalam di balik keputusan teknis utama yang diambil dalam pengembangan **AI SaaS Platform**. Dokumen ini menyoroti perbandingan antara pilihan-pilihan alternatif yang tersedia, menimbang kelebihan dan kekurangannya (*trade-offs*), serta memberikan pembenaran rasional mengapa keputusan tertentu akhirnya dipilih.

---

## 1. Database: Neon Serverless PostgreSQL vs Hosted PostgreSQL Tradisional (AWS RDS / Supabase)

### 📊 Perbandingan Arsitektur Database

| Fitur | Neon Serverless PostgreSQL (Dipilih) | Hosted PostgreSQL Tradisional (AWS RDS / Supabase) |
| :--- | :--- | :--- |
| **Kelebihan** | * **Autoscaling Instan:** Komputasi dinamis membesar sesuai beban, dan mengecil hingga **nol** saat tidak aktif (menghemat biaya secara drastis saat development/traffic rendah).<br>* **Database Branching:** Memungkinkan pembuatan cabang database instan untuk pengujian fitur baru (mirip Git branch).<br>* **Serverless Friendly:** Terhubung secara efisien via HTTP/Websockets tanpa kehabisan batas koneksi di serverless function. | * **Koneksi Stabil Permanen:** Sangat ideal untuk server berumur panjang (*long-running VPS*) yang menjaga koneksi persisten.<br>* **Performa Konsisten:** Tidak ada waktu tunggu *cold-start* karena komputasi selalu aktif.<br>* **Ekosistem Matang:** Dukungan penuh untuk berbagai ekstensi PostgreSQL yang kompleks. |
| **Kekurangan** | * **Cold Start:** Memerlukan waktu jeda 1-3 detik saat database "bangun" kembali setelah masa tidak aktif.<br>* **Ketergantungan Vendor:** Fitur branching dan autoscaling terikat erat pada platform Neon. | * **Biaya Tetap Tinggi:** Biaya berjalan terus menerus meskipun server database tidak menerima kueri sama sekali.<br>* **Batas Koneksi Kaku:** Jika ditaruh di serverless function (seperti Vercel), server database akan cepat crash karena kehabisan slot koneksi (*Connection Exhaustion*). |

### 🎯 Rasional Keputusan
Dipilih **Neon Serverless PostgreSQL** karena aplikasi ini dideploy di platform serverless (**Vercel**). Dalam arsitektur serverless, setiap request melahirkan instansi server NodeJS baru yang membuka koneksi database baru. PostgreSQL tradisional akan langsung kehabisan slot koneksi dalam waktu singkat jika diserang traffic tinggi. `@neondatabase/serverless` menyelesaikan masalah ini dengan menyalurkan kueri secara efisien melalui HTTP/Websockets. Selain itu, fitur auto-suspend hingga nol biaya saat sepi sangat cocok untuk menekan biaya operasional awal startup.

---

## 2. Autentikasi: Clerk Auth-as-a-Service vs Custom JWT Auth (Express + Bcrypt + JWT Lokal)

### 📊 Perbandingan Sistem Autentikasi

| Fitur | Clerk Auth-as-a-Service (Dipilih) | Custom JWT Autentikasi Lokal |
| :--- | :--- | :--- |
| **Kelebihan** | * **Waktu Rilis Super Cepat:** Menyediakan UI Login, Register, dan Profil siap pakai yang sangat premium.<br>* **Keamanan Maksimal:** Hashing password, perlindungan brute-force, dan kepatuhan GDPR diurus secara profesional oleh tim keamanan Clerk.<br>* **Fitur Bawaan Kaya:** Integrasi login sosial (Google/GitHub) dan penanganan Metadata Kustom secara instan tanpa menulis database terpisah. | * **Kontrol Penuh 100%:** Seluruh alur data user, enkripsi, dan basis data diatur sendiri tanpa bergantung pihak ketiga.<br>* **Bebas Biaya Vendor:** Tidak ada biaya bulanan seiring bertambahnya Monthly Active Users (MAU).<br>* **Portabilitas Tinggi:** Kode tidak terikat pada SDK eksternal dan bisa dipindahkan ke infrastruktur apa pun kapan saja. |
| **Kekurangan** | * **Biaya Skalabilitas:** Gratis hingga 10.000 pengguna aktif bulanan (MAU), namun tarif akan meningkat secara progresif setelahnya.<br>* **Ketergantungan Eksternal:** Jika server Clerk mengalami downtime, pengguna tidak akan bisa login ke aplikasi Anda. | * **Beban Pengembangan Besar:** Harus mendesain database user, sistem reset password lewat email, penanganan masa berlaku token (*refresh tokens*), dan proteksi brute-force secara manual.<br>* **Rawan Celah Keamanan:** Risiko kesalahan konfigurasi enkripsi/JWT yang dapat membahayakan data sensitif pengguna. |

### 🎯 Rasional Keputusan
Dipilih **Clerk Auth-as-a-Service** karena filosofi pengembangan aplikasi ini mengutamakan **kecepatan rilis ke pasar (Time-to-Market)** dan **fokus pada fitur inti AI**. Menulis sistem keamanan login kustom dari nol memerlukan waktu berminggu-minggu dan sangat rawan celah keamanan jika tidak diuji secara ketat. Clerk memotong waktu tersebut menjadi hitungan menit, sekaligus memberikan visualisasi UI profil yang premium dan penanganan metadata kuota (`free_usage`) yang andal di sisi server mereka.

---

## 3. Komunikasi Database: Raw SQL (Neon Direct Client) vs ORM (Prisma / Sequelize)

### 📊 Perbandingan Pendekatan Akses Database

| Fitur | Raw SQL via Neon Client (Dipilih) | Object-Relational Mapping / ORM (Prisma) |
| :--- | :--- | :--- |
| **Kelebihan** | * **Performa Maksimal:** Eksekusi langsung ke database tanpa ada lapisan konversi data perantara (overhead zero).<br>* **Kontrol Kueri Penuh:** Developer dapat menulis SQL canggih secara tepat (seperti manipulasi array array postgres `text[]` untuk fitur Likes).<br>* **Ukuran Bundle Kecil:** Tanpa perlu menginstal engine ORM yang besar dan berat. | * **Type Safety & Autocomplete:** Mendeteksi kesalahan ketik nama kolom saat menulis kode (*Compile-time safety*).<br>* **Migrasi Database Otomatis:** Perubahan skema dikelola lewat CLI secara terstruktur.<br>* **Abstraksi Database:** Kode javascript tetap sama meskipun database diganti dari PostgreSQL ke MySQL atau SQLite. |
| **Kekurangan** | * **Rentan Kesalahan Tulis:** Kesalahan ketik nama tabel/kolom baru terdeteksi saat program dijalankan (*runtime error*).<br>* **Menulis Query Manual:** Developer harus menulis sintaks INSERT, UPDATE, SELECT yang panjang untuk relasi kompleks. | * **Penurunan Performa:** Proses penerjemahan dari method objek JavaScript ke teks kueri SQL memakan waktu mikrodetik tambahan.<br>* **Kurang Fleksibel untuk Fitur Non-Standar:** Penanganan tipe data khusus PostgreSQL (seperti tipe Array) terkadang memerlukan trik yang rumit di ORM. |

### 🎯 Rasional Keputusan
Dipilih **Raw SQL via Neon Client** karena kesederhanaan skema database aplikasi saat ini (hanya memiliki satu tabel utama `creations`). Dalam skenario tabel tunggal dengan relasi sederhana, penggunaan ORM raksasa seperti Prisma dianggap sebagai *over-engineering* yang memperlambat startup serverless function. Selain itu, fitur *likes* yang memanfaatkan tipe array PostgreSQL (`text[]`) sangat mudah dieksekusi menggunakan SQL mentah dibandingkan harus mengonfigurasinya melalui skema relasi ORM yang ketat.

---

## 4. Akses AI: Gemini API via OpenAI SDK Wrapper vs Official Google Gen AI SDK

### 📊 Perbandingan Integrasi SDK AI

| Fitur | Gemini API via OpenAI SDK Wrapper (Dipilih) | Official Google Gen AI SDK (`@google/generative-ai`) |
| :--- | :--- | :--- |
| **Kelebihan** | * **Standarisasi Kode:** Menggunakan pola penulisan SDK OpenAI yang diakui secara global di industri AI.<br>* **Fleksibilitas Migrasi Tinggi:** Jika kelak ingin mengganti model ke GPT-4o (OpenAI) atau Claude (Anthropic), developer hanya perlu mengubah Base URL dan API Key tanpa menulis ulang struktur parser response. | * **Dukungan Fitur 100% Native:** Mengakses semua fitur terbaru Gemini secara eksklusif (seperti Google Search Grounding, multimodal audio input luas).<br>* **Optimasi Performa:** Performa request yang sedikit lebih efisien karena langsung menuju server Google tanpa perantara parser kompatibilitas. |
| **Kekurangan** | * **Fitur Terbatas:** Beberapa parameter eksklusif Gemini mungkin tidak didukung sepenuhnya melalui wrapper kompatibilitas OpenAI. | * **Vendor Lock-in Tinggi:** Sintaks kode sangat spesifik milik Google. Jika ingin bermigrasi ke penyedia AI lain, seluruh controller AI harus ditulis ulang dari nol. |

### 🎯 Rasional Keputusan
Dipilih **Gemini API via OpenAI SDK Wrapper**. Keuntungan memiliki kode yang **fleksibel dan tidak terikat vendor** jauh melebihi kebutuhan akses ke fitur eksklusif Gemini yang sangat spesifik. Dengan menggunakan pola standar OpenAI, aplikasi ini tetap memiliki fleksibilitas arsitektur yang tinggi untuk melakukan integrasi *Multi-LLM* (misalnya menggunakan GPT-4o untuk resume review dan Gemini untuk penulisan artikel) secara bersih di masa mendatang.

---

## 5. Manipulasi Gambar AI: Cloudinary AI Transformation vs Pemrosesan Gambar di Server Lokal

### 📊 Perbandingan Pemrosesan Manipulasi Gambar

| Fitur | Cloudinary AI Transformation (Dipilih) | Pemrosesan Gambar di Server Lokal (Sharp / TensorFlow / Python) |
| :--- | :--- | :--- |
| **Kelebihan** | * **Beban Server Nol:** Pemrosesan gambar berat (penghapusan background/objek) sepenuhnya dilakukan di server Cloudinary.<br>* **Waktu Respon Cepat:** Proses transformasi berjalan di infrastruktur CDN Cloudinary yang sangat cepat.<br>* **Kemudahan Integrasi:** Cukup mengubah/menambahkan parameter transformasi pada string URL gambar saja. | * **Bebas Biaya Bandwidth:** Tidak bergantung pada kuota bandwidth Cloudinary bulanan.<br>* **Privasi Data Penuh:** File gambar pengguna tidak diunggah ke server pihak ketiga.<br>* **Kontrol Algoritma:** Developer bisa menyesuaikan model machine learning penghapusan objek secara presisi. |
| **Kekurangan** | * **Biaya Operasional:** Cloudinary menerapkan sistem kredit transformasi berbayar jika penggunaan melebihi kuota gratis.<br>* **Keterbatasan Kustomisasi:** Developer harus mengikuti algoritma manipulasi yang disediakan secara bawaan oleh Cloudinary. | * **Beban CPU/Memori Sangat Tinggi:** Dapat membuat server Express lokal crash/hang saat memproses gambar resolusi tinggi.<br>* **Ukuran Server Gemuk:** Harus menginstal library pemrosesan gambar biner yang besar, memperlambat proses deployment. |

### 🎯 Rasional Keputusan
Dipilih **Cloudinary AI Transformation**. Menjalankan manipulasi gambar berbasis AI (seperti Background Removal dan Object Erasure) memerlukan daya komputasi CPU dan memori RAM yang sangat intensif. Jika dijalankan langsung di server Node.js lokal (atau di serverless Vercel yang memiliki batas waktu eksekusi maksimal 10-60 detik), request akan sering mengalami *Timeout* dan memicu crash. Mengalihkan beban kerja tersebut ke Cloudinary adalah keputusan arsitektur terbaik untuk menjaga backend Express tetap ringan, responsif, dan stabil.
