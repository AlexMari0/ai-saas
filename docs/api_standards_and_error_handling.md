# 🌐 Standar API & Penanganan Error Backend

Dokumen ini menganalisis kualitas arsitektur API yang berjalan pada platform **AI SaaS Platform** dari sudut pandang **RESTful API Conventions**, penggunaan **HTTP Status Codes**, dan strategi **Error Handling** di sisi backend (Express.js).

---

## 1. Review Struktur API: Konvensi REST vs RPC-Style

REST (Representational State Transfer) adalah gaya arsitektur standar untuk merancang jaringan API yang menggunakan metode HTTP untuk menentukan aksi pada data resource (sumber daya).

### 🔍 Kondisi Aktual API Aplikasi Saat Ini
Saat ini, rute API pada aplikasi Anda menggunakan pendekatan **RPC-Style (Remote Procedure Call)** daripada standar **RESTful**.

Berikut adalah daftar endpoint aktual di aplikasi Anda:
* `GET /api/user/get-user-creations`
* `GET /api/user/get-published-creations`
* `POST /api/user/toggle-like-creation`
* `POST /api/ai/generate-article`
* `POST /api/ai/generate-blog-title`
* `POST /api/ai/generate-image`

### ⚠️ Mengapa Ini Disebut RPC-Style (Bukan Pure REST)?
1. **Menggunakan Kata Kerja pada URL:**
   * Di REST, URL harus menggambarkan suatu *resource* (kata benda), bukan tindakan (kata kerja). Penggunaan kata kerja seperti `get-user-creations` atau `toggle-like` di dalam URL dianggap redundan karena metode HTTP-nya sendiri (`GET` dan `POST`) sudah merepresentasikan tindakan tersebut.
2. **Resource-oriented Routing:**
   * REST memfokuskan URL pada entitas data (misalnya: `/creations`).

### 💡 Rekomendasi Transformasi ke Konvensi RESTful
Untuk menyelaraskan rute dengan konvensi industri REST, rute-rute di atas sebaiknya diubah menjadi:

| Fitur / Aksi | Rute Saat Ini (RPC-Style) | Rute Rekomendasi (RESTful) | Penjelasan REST |
| :--- | :--- | :--- | :--- |
| Ambil kreasi user | `GET /api/user/get-user-creations` | `GET /api/creations/me` atau `GET /api/creations?scope=user` | Mengambil koleksi resource kreasi yang dimiliki oleh user aktif. |
| Ambil kreasi publik | `GET /api/user/get-published-creations` | `GET /api/creations` atau `GET /api/creations?published=true` | Mengambil koleksi resource kreasi yang bersifat publik. |
| Sukai / unlike kreasi | `POST /api/user/toggle-like-creation` | `POST /api/creations/:id/likes` atau `DELETE /api/creations/:id/likes` | Menambahkan/menghapus relasi "like" pada resource kreasi tertentu. |
| Generate artikel/judul | `POST /api/ai/generate-article` | `POST /api/creations` | Membuat resource kreasi baru dengan mengirimkan tipe (`type: "article"`) pada body. |

---

## 2. Pemahaman & Pentingnya HTTP Status Code

HTTP Status Code adalah kode standar dari server untuk memberi tahu client (browser/Axios) tentang hasil pemrosesan request secara langsung melalui jaringan (network level).

### 🚨 Isu Kritis pada Backend Saat Ini
Di semua controller backend Anda, kode penanganan error ditulis seperti ini:
```javascript
try {
  // ...
  res.json({ success: true, content });
} catch (error) {
  res.json({ success: false, message: error.message });
}
```
* **Masalah:** Express secara default akan mengembalikan HTTP Status **`200 OK`** jika Anda memanggil `res.json()` tanpa mendefinisikan status secara spesifik.
* **Akibatnya:** Meskipun terjadi error parah di backend (database mati, token tidak valid, dll.), jaringan browser melihat transaksi sebagai **`200 OK` (Sukses)**. Ini membuat sistem pemantauan (seperti Sentry/Datadog) tidak mendeteksi adanya error, dan Axios di frontend tidak akan otomatis memicu blok catch-nya karena menganggap request berhasil.

---

### 📚 Daftar HTTP Status Code Penting & Kegunaannya

Untuk meningkatkan kualitas API, server harus mengirimkan status code yang tepat menggunakan `.status()` sebelum `.json()`:

#### 🟢 2xx: Success (Operasi Berhasil)
* **`200 OK`**
  * *Arti:* Request berhasil dieksekusi dan data yang diminta dikembalikan.
  * *Kapan digunakan:* Menjawab kueri `GET` data riwayat kreasi atau memperbarui status suka (`toggle-like`).
  * *Implementasi:* `res.status(200).json(...)`
* **`201 Created`**
  * *Arti:* Request berhasil dan server berhasil membuat resource baru.
  * *Kapan digunakan:* Setelah AI berhasil men-generate gambar/artikel dan menyimpannya ke database Neon.
  * *Implementasi:* `res.status(201).json(...)`

#### 🟡 4xx: Client Error (Kesalahan di Sisi Pengguna)
* **`400 Bad Request`**
  * *Arti:* Request tidak valid karena format data salah atau ada input wajib yang kosong.
  * *Kapan digunakan:* Ketika pengguna mengunggah file resume yang melebihi batas 5MB atau form prompt kosong.
  * *Implementasi:* `res.status(400).json({ message: "Resume file size exceeds 5MB." })`
* **`401 Unauthorized`**
  * *Arti:* Pengguna belum login atau token autentikasi tidak valid / kedaluwarsa.
  * *Kapan digunakan:* Ketika token Clerk JWT hilang atau gagal diverifikasi oleh middleware `auth.js`.
  * *Implementasi:* `res.status(401).json({ message: "Unauthorized access." })`
* **`403 Forbidden`**
  * *Arti:* Server memahami request pengguna, tetapi menolak memberikan akses karena alasan kebijakan kuota/hak akses.
  * *Kapan digunakan:* Pengguna non-premium mencoba mengakses fitur khusus premium (seperti Image Generator atau Background Removal) atau kuota gratis telah habis (limit reached).
  * *Implementasi:* `res.status(403).json({ message: "This feature is only available for premium subscriptions." })`
* **`404 Not Found`**
  * *Arti:* Resource yang diminta tidak ditemukan di server.
  * *Kapan digunakan:* Ketika pengguna mencoba menyukai kreasi dengan ID yang tidak ada di database.
  * *Implementasi:* `res.status(404).json({ message: "Creation not found." })`

#### 🔴 5xx: Server Error (Kesalahan di Sisi Server)
* **`500 Internal Server Error`**
  * *Arti:* Server mengalami galat internal tak terduga yang membuatnya gagal memproses request.
  * *Kapan digunakan:* Koneksi database Neon terputus, API key eksternal habis/mati, atau terjadi error sintaks runtime.
  * *Implementasi:* `res.status(500).json({ message: error.message })`

---

## 3. Analisis Penanganan Error (Error Handling) di Backend

Saat ini, penanganan error di backend Anda menggunakan pola **Local Try-Catch Block** di setiap controller.

```javascript
export const generateArticle = async (req, res) => {
  try {
    // Logika Utama...
  } catch (error) {
    console.log(error);
    res.json({ success: false, message: error.message });
  }
};
```

### ⚖️ Kelebihan & Kekurangan Pendekatan Saat Ini

* **🟢 Kelebihan:**
  * Sederhana dan mudah dimengerti untuk aplikasi skala kecil.
  * Mencegah crash total pada aplikasi Express jika salah satu rute mengalami error.
* **🔴 Kekurangan:**
  * **Kode Redundan:** Anda menulis blok `try-catch` berulang kali di belasan fungsi controller yang berbeda.
  * **Status Code Tidak Konsisten:** Semua error mengembalikan status code `200` (seperti yang dibahas di atas).
  * **Tidak Ada Centralized Error Logger:** Tidak ada mekanisme terpusat untuk memformat pesan error, menyembunyikan stack trace di lingkungan production untuk keamanan, atau mengintegrasikan log ke monitoring tools pihak ketiga.

---

### 🚀 Cara Terbaik: Implementasi Global Error Middleware di Express

Untuk arsitektur tingkat produksi, Express merekomendasikan penggunaan **Global Error Handling Middleware** guna menyederhanakan penulisan kode controller:

#### Langkah 1: Buat Global Error Middleware (`server/middlewares/errorHandler.js`)
```javascript
export const errorHandler = (err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  const message = err.message || "Internal Server Error";

  // Cetak log error ke console/file log untuk debugging developer
  console.error(`[Error] ${req.method} ${req.url} - ${message}`);
  if (process.env.NODE_ENV !== "production") {
    console.error(err.stack); // Cetak stack trace hanya saat development
  }

  res.status(statusCode).json({
    success: false,
    message: message,
    stack: process.env.NODE_ENV === "production" ? null : err.stack
  });
};
```

#### Langkah 2: Daftarkan di file Utama (`server/server.js`)
Daftarkan di bagian paling bawah setelah rute-rute utama terdefinisi:
```javascript
app.use("/api/ai", aiRouter);
app.use("/api/user", userRouter);

// Global Error Handler Middleware
app.use(errorHandler);
```

#### Langkah 3: Gunakan di Controller dengan Bersih
Dengan middleware terpusat, controller tidak perlu lagi melakukan pemformatan error manual, cukup teruskan error ke fungsi `next(error)`:
```javascript
export const generateArticle = async (req, res, next) => {
  try {
    // Logika utama...
  } catch (error) {
    next(error); // Express otomatis melempar error ke Global Error Handler
  }
};
```
