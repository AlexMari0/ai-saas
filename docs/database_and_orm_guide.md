# 🗄️ Panduan Database Relasional: Neon PostgreSQL & Konsep ORM

Dokumen ini menjelaskan arsitektur database relasional pada platform **AI SaaS Platform** yang didukung oleh **Neon PostgreSQL (Serverless)**, pemahaman mendalam tentang konsep kueri SQL, serta perbandingan pendekatan kueri berbasis **Raw SQL** yang digunakan saat ini dengan sistem **ORM (Object-Relational Mapping)**.

---

## 1. Review Neon PostgreSQL dalam Aplikasi

**Neon PostgreSQL** adalah layanan database relasional serverless modern berbasis PostgreSQL. Neon memisahkan proses komputasi (*compute*) dan penyimpanan (*storage*), memberikan fitur unggulan berupa auto-scaling instan dan pencabangan (*branching*) database seperti Git.

### 🔌 Cara Koneksi & Eksekusi di Aplikasi
Di aplikasi Anda, backend terhubung ke Neon PostgreSQL tanpa menggunakan driver PostgreSQL standar (`pg`), melainkan melalui pustaka khusus serverless:
```javascript
import { neon } from '@neondatabase/serverless';
const sql = neon(`${process.env.DATABASE_URL}`);
export default sql;
```
* **Metode Eksekusi:** Backend menggunakan pendekatan *direct SQL queries* melalui tagged template literals (`sql```) untuk mengirimkan instruksi SQL secara langsung.
* **Tabel Utama:** **`creations`**
  Berdasarkan analisis file controller, skema tabel `creations` dirancang untuk menyimpan metadata hasil olahan AI dengan kolom-kolom berikut:
  * `id`: Primary Key unik (berfungsi sebagai identitas baris).
  * `user_id`: Tipe data teks, menyimpan ID unik pengguna yang bersumber dari Clerk.
  * `prompt`: Teks asli dari pengguna untuk memicu generator AI.
  * `content`: Teks hasil generate (artikel / blog title / resume feedback) ATAU string URL aman dari Cloudinary (untuk gambar hasil olahan).
  * `type`: Kategori kreasi (`'article'`, `'blog-title'`, `'image'`, atau `'resume-review'`).
  * `publish`: Boolean (default `false`), penentu apakah hasil kreasi ditampilkan di halaman Community.
  * `likes`: Array teks (`text[]`), menampung daftar string `user_id` yang menyukai kreasi tersebut.
  * `created_at`: Stempel waktu (*Timestamp*) kapan data pertama kali dimasukkan.

---

## 2. Pemahaman Konsep Kueri & Relasi Database Relasional

Berikut adalah penjelasan konsep database relasional, bagaimana konsep tersebut bekerja, serta implementasinya di dalam aplikasi Anda.

### A. Klausul `WHERE`
* **Definisi:** Digunakan untuk menyaring (*filtering*) baris data berdasarkan kondisi spesifik agar server hanya mengambil data yang relevan.
* **Penerapan di Aplikasi:** Membatasi riwayat kreasi agar hanya menampilkan milik pengguna aktif saat memuat Dashboard:
  ```sql
  SELECT * FROM creations WHERE user_id = ${userId} ORDER BY created_at DESC;
  ```

### B. Foreign Key (Kunci Asing)
* **Definisi:** Kolom atau kumpulan kolom dalam satu tabel yang nilainya merujuk pada Primary Key tabel lain. Digunakan untuk menjaga integritas data antar tabel (*Referential Integrity*).
* **Penerapan di Aplikasi:**
  Kolom `user_id` pada tabel `creations` secara logis bertindak sebagai **Foreign Key**. Karena sistem pengguna Anda dikelola secara eksternal oleh **Clerk (Auth-as-a-Service)**, integritas hubungan ini dijaga di tingkat kode aplikasi backend Express, bukan diatur sebagai relasi foreign key kaku di dalam database PostgreSQL internal Anda.

### C. Index (Indeks)
* **Definisi:** Struktur data penunjuk khusus yang membantu mesin database mencari dan mengambil baris data tertentu dengan kecepatan tinggi tanpa melakukan pemindaian penuh ke seluruh tabel (*Full Table Scan*).
* **Rekomendasi Penerapan:**
  Karena query pencarian di aplikasi Anda sering kali menyaring data berdasarkan kolom `user_id` (di Dashboard) dan `publish` (di Community), sangat disarankan untuk menambahkan indeks pada kedua kolom ini guna mempercepat performa seiring bertambahnya jumlah data pengguna:
  ```sql
  CREATE INDEX idx_creations_user_id ON creations(user_id);
  CREATE INDEX idx_creations_publish ON creations(publish);
  ```

### D. Relasi One-to-Many (Satu ke Banyak)
* **Definisi:** Relasi di mana satu baris di Tabel A dapat terhubung ke banyak baris di Tabel B, tetapi satu baris di Tabel B hanya dapat terhubung ke satu baris di Tabel A.
* **Penerapan di Aplikasi:**
  **Satu Pengguna memiliki Banyak Kreasi AI**. Satu entitas User (Clerk) dapat memiliki tak terbatas baris di tabel `creations`, namun setiap baris kreasi hanya dimiliki oleh tepat satu pengguna yang ditandai oleh nilai `user_id` di baris tersebut.

### E. Relasi Many-to-Many (Banyak ke Banyak)
* **Definisi:** Hubungan di mana banyak baris di Tabel A dapat terhubung ke banyak baris di Tabel B, dan sebaliknya.
* **Penerapan di Aplikasi:**
  **Sistem Likes pada Kreasi**. Banyak pengguna dapat menyukai banyak kreasi, dan satu kreasi dapat disukai oleh banyak pengguna berbeda.
  * **Cara Normalisasi Database Relasional Standard:** Menggunakan tabel penghubung (*Junction Table*), misalnya tabel `likes` dengan kolom `user_id` dan `creation_id`.
  * **Cara Denormalisasi (Pendekatan Aplikasi Anda saat ini):** PostgreSQL memiliki tipe data array. Aplikasi Anda memanfaatkan ini dengan menyimpan kumpulan ID pengguna di kolom `likes` dengan tipe array teks (`text[]`):
    ```sql
    UPDATE creations SET likes = ${formattedArray}::text[] WHERE id = ${id};
    ```
    *Analisis:* Trik array ini efisien untuk aplikasi skala kecil karena menghindari kueri `JOIN` yang berat, namun kurang ideal untuk skala besar jika Anda ingin melakukan kueri analitik lanjut yang kompleks.

### F. Kueri `JOIN`
* **Definisi:** Menggabungkan baris-baris dari dua atau lebih tabel berdasarkan kolom terkait yang bernilai sama.
* **Penerapan di Aplikasi:**
  Saat ini aplikasi Anda hanya memerlukan satu tabel (`creations`) karena data user dikelola oleh Clerk API. Namun, jika Anda menambahkan tabel internal baru seperti tabel `subscriptions` untuk mencatat riwayat transaksi pembayaran pengguna lokal, Anda akan menggunakan `JOIN` untuk mencocokkan data kreasi dengan detail status langganan:
  ```sql
  SELECT creations.*, subscriptions.status 
  FROM creations 
  INNER JOIN subscriptions ON creations.user_id = subscriptions.user_id;
  ```

---

## 3. Konsep ORM (Object-Relational Mapping) dalam Aplikasi

### ❓ Apa itu ORM?
**Object-Relational Mapping (ORM)** adalah teknik pemrograman yang memungkinkan developer berinteraksi dengan database relasional menggunakan paradigma pemrograman berorientasi objek (OOP) di bahasa pilihan mereka (misalnya Javascript/Node.js), tanpa harus menulis sintaks SQL mentah secara manual. 

Contoh ORM populer di ekosistem Node.js adalah **Prisma**, **Sequelize**, dan **TypeORM**.

---

### ⚖️ Perbandingan Pendekatan: Direct SQL Client vs ORM

Aplikasi Anda saat ini **TIDAK menggunakan ORM**. Aplikasi Anda menggunakan **Direct SQL Client** berbasis pustaka `@neondatabase/serverless`.

Berikut adalah perbandingan objektif antara kedua pendekatan tersebut untuk memandu pilihan arsitektur Anda di masa depan:

| Aspek | Direct SQL Client (Pendekatan Anda Saat Ini) | ORM (Contoh: Prisma) |
| :--- | :--- | :--- |
| **Sintaks Kode** | Menulis SQL mentah dalam bentuk string/template literals.<br>`await sql`SELECT * FROM creations`` | Memanggil method bawaan Javascript objek model.<br>`await prisma.creation.findMany()` |
| **Kecepatan & Performa** | **Sangat Cepat.** Tanpa ada overhead atau lapisan abstraksi tambahan antara kode Node.js dan database. | **Sedikit Lebih Lambat.** Memerlukan waktu pemrosesan tambahan untuk menerjemahkan method objek menjadi query SQL. |
| **Keamanan (SQL Injection)** | Rentan jika developer menggabungkan string secara manual (`+` atau `${}`). Namun, tagged template dari Neon secara otomatis melakukan sanitasi parameter sehingga aman dari injeksi SQL. | **Sangat Aman.** ORM secara otomatis melakukan parameterisasi dan sanitasi untuk setiap query yang dihasilkan. |
| **Developer Experience** | Memerlukan pemahaman sintaks SQL yang kuat. Tidak ada autocomplete otomatis (*IntelliSense*) untuk nama kolom atau tabel saat mengetik kueri di editor. | **Sangat Baik.** Menyediakan fitur autocompletion otomatis untuk skema database dan deteksi error saat penulisan kode (*Compile-time error checking*). |
| **Migrasi & Skema** | Developer harus membuat skema tabel secara manual di console database Neon atau menulis skrip DDL SQL secara terpisah. | Menyediakan tools CLI migrasi database otomatis untuk menyelaraskan skema database dengan file skema kode program secara instan. |
