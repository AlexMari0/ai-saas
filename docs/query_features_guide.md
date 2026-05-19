# 🔍 Panduan Fitur Kueri: Pagination, Search & Category Filtering

Dokumen ini menjelaskan konsep teoritis dan langkah praktis untuk mengimplementasikan tiga fitur kueri penting pada aplikasi **AI SaaS Platform**: **Pagination** (Offset/Limit), **Search** (LIKE / ILIKE Query), dan **Category Filtering**.

Saat ini, kueri di backend Anda (`server/controllers/userController.js`) masih mengambil seluruh data sekaligus tanpa penyaringan:
```sql
SELECT * FROM creations WHERE publish = true ORDER BY created_at DESC;
```
Di bawah ini adalah penjelasan dan panduan implementasi untuk meningkatkan performa serta kenyamanan pencarian data di aplikasi Anda.

---

## 1. Implementasi Pagination di Backend (Limit & Offset)

### ❓ Mengapa Pagination Penting?
Jika database memiliki ribuan hasil kreasi, mengirimkan semua data tersebut sekaligus dalam satu request HTTP akan:
1. Membebani memori server Express.
2. Memperlambat transfer data jaringan.
3. Memperlambat browser frontend saat merender ribuan komponen UI (*DOM overload*).

**Pagination** memotong data menjadi potongan-potongan kecil (halaman) secara dinamis menggunakan dua parameter SQL:
* **`LIMIT`:** Jumlah baris data maksimal yang diambil dalam satu halaman (misal: 10 data).
* **`OFFSET`:** Jumlah baris data awal yang dilewati/diabaikan sebelum data mulai diambil.

### 📐 Rumus Perhitungan Offset
$$\text{OFFSET} = (\text{Page} - 1) \times \text{Limit}$$

* *Halaman 1:* $(\text{Page } 1 - 1) \times 10 = \text{OFFSET } 0$ (Ambil data ke-1 sampai ke-10).
* *Halaman 2:* $(\text{Page } 2 - 1) \times 10 = \text{OFFSET } 10$ (Lewati 10 data pertama, ambil data ke-11 sampai ke-20).

### 🛠️ Contoh Kode Implementasi di Backend Express & Neon
```javascript
export const getPublishedCreations = async (req, res) => {
  try {
    // Ambil parameter kueri dari URL (default: halaman 1, limit 10 item)
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 10;
    const offset = (page - 1) * limit;

    // 1. Ambil data dengan LIMIT dan OFFSET
    const creations = await sql`
      SELECT * FROM creations 
      WHERE publish = true 
      ORDER BY created_at DESC 
      LIMIT ${limit} OFFSET ${offset}
    `;

    // 2. Ambil total keseluruhan data untuk informasi UI di frontend
    const [totalResult] = await sql`
      SELECT COUNT(*) as count FROM creations WHERE publish = true
    `;
    const totalItems = parseInt(totalResult.count);
    const totalPages = Math.ceil(totalItems / limit);

    res.json({
      success: true,
      creations,
      pagination: {
        currentPage: page,
        limit,
        totalItems,
        totalPages,
        hasNextPage: page < totalPages,
        hasPrevPage: page > 1
      }
    });
  } catch (error) {
    res.json({ success: false, message: error.message });
  }
};
```

---

## 2. Cara Kerja Fitur Search (LIKE / ILIKE Query)

### ❓ Bagaimana Cara Kerja Search?
Pencarian data dilakukan dengan mencocokkan kata kunci (*keyword*) yang dikirimkan oleh pengguna dengan nilai di kolom tertentu (misalnya, mencocokkan keyword pencarian dengan kolom `prompt` di tabel `creations`).

Di database relasional SQL, kita menggunakan operator **`LIKE`** atau **`ILIKE`**:
* **`LIKE`:** Mencocokkan string secara sensitif terhadap huruf besar/kecil (*Case-Sensitive*).
* **`ILIKE`:** Mencocokkan string tanpa mempedulikan huruf besar/kecil (*Case-Insensitive*). PostgreSQL sangat efisien dalam memproses kueri `ILIKE`.

### 🎯 Penggunaan Karakter Wildcard (`%`)
Untuk melakukan pencarian kata kunci parsial (tidak harus sama persis), kita menambahkan karakter persen `%` di sekitar kata kunci:
* `'%keyword%'` : Mencari baris yang mengandung kata kunci di **mana saja** di dalam kolom target.
* `'keyword%'`  : Mencari baris yang **diawali** dengan kata kunci.
* `'%keyword'`  : Mencari baris yang **diakhiri** dengan kata kunci.

### 🛠️ Contoh Kode Implementasi di Backend
```javascript
export const getPublishedCreations = async (req, res) => {
  try {
    const search = req.query.search || "";
    // Susun pattern wildcard (contoh: '%cat%')
    const searchPattern = `%${search}%`;

    const creations = await sql`
      SELECT * FROM creations 
      WHERE publish = true AND prompt ILIKE ${searchPattern}
      ORDER BY created_at DESC
    `;

    res.json({ success: true, creations });
  } catch (error) {
    res.json({ success: false, message: error.message });
  }
};
```

---

## 3. Implementasi Filter Berdasarkan Kategori (Category Filtering)

### ❓ Bagaimana Cara Kerjanya?
Filter kategori membatasi hasil pencarian agar hanya menampilkan tipe konten tertentu secara tepat, misalnya hanya menampilkan kategori `'image'`, `'article'`, `'blog-title'`, atau `'resume-review'`.

Dalam SQL, pencocokan ini menggunakan operator perbandingan langsung (`=`) karena kategori bersifat mutlak (tidak parsial seperti pencarian teks).

### 🛠️ Contoh Kode Implementasi di Backend
Kita mengambil nilai kategori dari query parameter URL (misalnya: `/api/user/get-published-creations?category=image`):
```javascript
export const getPublishedCreations = async (req, res) => {
  try {
    const category = req.query.category || "";

    let creations;
    if (category) {
      // Jika filter kategori dikirim, saring berdasarkan kolom 'type'
      creations = await sql`
        SELECT * FROM creations 
        WHERE publish = true AND type = ${category}
        ORDER BY created_at DESC
      `;
    } else {
      // Jika kosong, ambil semua tipe
      creations = await sql`
        SELECT * FROM creations 
        WHERE publish = true 
        ORDER BY created_at DESC
      `;
    }

    res.json({ success: true, creations });
  } catch (error) {
    res.json({ success: false, message: error.message });
  }
};
```

---

## 🚀 Penggabungan Tiga Fitur Menjadi Satu Endpoint Dinamis

Sebagai bonus arsitektur, Anda dapat menggabungkan ketiga fitur di atas (Pagination + Search + Category Filtering) ke dalam **satu kueri dinamis tunggal** agar endpoint API Anda menjadi sangat fleksibel dan efisien.

### 🛠️ Contoh Kueri Terintegrasi Lengkap
```javascript
export const getPublishedCreations = async (req, res) => {
  try {
    // Ambil parameter dari request query
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 10;
    const offset = (page - 1) * limit;
    
    const search = req.query.search || "";
    const searchPattern = `%${search}%`;
    
    const category = req.query.category || "";

    // 1. Eksekusi Kueri Data Dinamis dengan filter pencarian, kategori, dan pagination
    const creations = await sql`
      SELECT * FROM creations 
      WHERE publish = true 
        AND prompt ILIKE ${searchPattern}
        AND (${category} = '' OR type = ${category})
      ORDER BY created_at DESC 
      LIMIT ${limit} OFFSET ${offset}
    `;

    // 2. Kueri Total Data dengan kondisi filter yang persis sama
    const [totalResult] = await sql`
      SELECT COUNT(*) as count FROM creations 
      WHERE publish = true 
        AND prompt ILIKE ${searchPattern}
        AND (${category} = '' OR type = ${category})
    `;
    const totalItems = parseInt(totalResult.count);
    const totalPages = Math.ceil(totalItems / limit);

    res.json({
      success: true,
      creations,
      pagination: {
        currentPage: page,
        limit,
        totalItems,
        totalPages,
        hasNextPage: page < totalPages,
        hasPrevPage: page > 1
      }
    });
  } catch (error) {
    res.json({ success: false, message: error.message });
  }
};
```
*Catatan:* Logika `(${category} = '' OR type = ${category})` dalam query di atas sangat efisien karena jika parameter `category` kosong, PostgreSQL akan mengabaikan penyaringan tipe tersebut dan langsung mencocokkan semua tipe.
