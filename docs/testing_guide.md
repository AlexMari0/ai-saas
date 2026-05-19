# 🧪 Panduan Testing: Unit Testing (Jest) & Manual API Testing (Postman)

Dokumen ini menjelaskan strategi pengujian untuk platform **AI SaaS Platform**, yang mencakup konsep dasar dan implementasi **Unit Testing** pada Node.js menggunakan **Jest**, serta panduan langkah demi langkah untuk melakukan **Manual API Testing** menggunakan **Postman**.

---

## 1. Unit Testing di Node.js Menggunakan Jest

**Unit Testing** adalah pengujian terhadap bagian terkecil dari kode program (biasanya berupa fungsi individu atau modul) secara terisolasi. Tujuannya adalah memastikan bahwa logika internal fungsi tersebut berjalan 100% benar sesuai spesifikasi, tanpa bergantung pada jaringan luar atau database eksternal.

### ❓ Mengapa Menggunakan Jest?
**Jest** adalah kerangka pengujian JavaScript yang sangat populer karena:
* **Zero Config:** Siap digunakan secara langsung dengan konfigurasi minimal.
* **Built-in Assertions:** Menyediakan fungsi pemeriksaan hasil (`expect()`) yang sangat lengkap tanpa perlu menginstal pustaka tambahan (seperti Chai atau Should).
* **Powerful Mocking:** Memudahkan pembuatan fungsi tiruan (*mocks*) untuk menggantikan modul pihak ketiga (seperti API Gemini atau Neon Database).

---

### 🗝️ Konsep Kunci Jest

1. **Test Suite & Test Case:**
   * **`describe(name, fn)`:** Mengelompokkan beberapa kasus pengujian yang saling berhubungan (Test Suite).
   * **`test(name, fn)`** (atau **`it()`**): Menulis kasus pengujian tunggal spesifik (Test Case).
2. **Matchers (Assertion):**
   Membandingkan nilai hasil aktual dari kode dengan ekspektasi hasil.
   * `toBe(value)`: Untuk pencocokan nilai primitif (seperti angka atau string).
   * `toEqual(value)`: Untuk pencocokan struktur objek atau array secara mendalam.
   * `toBeNull()`, `toBeDefined()`, `toBeTruthy()`.
3. **Mocking & Spying:**
   Mengisolasi unit test dengan memalsukan request API jaringan atau query database menggunakan data buatan (`jest.fn()`). Ini penting agar unit test berjalan super cepat dan **tidak menghabiskan kuota biaya API eksternal** saat testing.
4. **Hooks (Lifecycle):**
   * `beforeEach()` / `afterEach()`: Dijalankan sebelum/sesudah setiap kasus tes dimulai. Sangat berguna untuk mereset data tiruan (*mock reset*).

---

### 🛠️ Contoh Kode Unit Test Sederhana (Node.js)

Misalkan kita memiliki fungsi utilitas di backend untuk mengecek sisa kuota gratis pengguna (`quotaChecker.js`):
```javascript
// quotaChecker.js
export const isAllowedToUse = (plan, freeUsage) => {
  if (plan === "premium") return true;
  if (plan === "free" && freeUsage < 10) return true;
  return false;
};
```

Berikut adalah file unit test-nya menggunakan Jest (`quotaChecker.test.js`):
```javascript
import { isAllowedToUse } from "./quotaChecker.js";

describe("Pengujian Logika Kuota Pengguna", () => {
  test("Harus mengizinkan pengguna dengan paket Premium", () => {
    const result = isAllowedToUse("premium", 15);
    expect(result).toBe(true); // Ekspektasi: Selalu true
  });

  test("Harus mengizinkan pengguna paket Free jika pemakaian di bawah 10 kali", () => {
    const result = isAllowedToUse("free", 5);
    expect(result).toBe(true); // Ekspektasi: true karena 5 < 10
  });

  test("Harus menolak pengguna paket Free jika pemakaian sudah 10 kali atau lebih", () => {
    const result = isAllowedToUse("free", 10);
    expect(result).toBe(false); // Ekspektasi: false karena kuota habis
  });
});
```

---

## 2. Manual API Testing Menggunakan Postman

**Postman** adalah alat bantu kolaborasi pengembangan API yang memudahkan developer untuk menyusun, mengirim, dan menguji request HTTP secara manual dan visual sebelum diintegrasikan ke frontend React.

Mengingat platform Anda mengamankan setiap endpoint menggunakan middleware **Clerk Auth**, berikut adalah panduan langkah demi langkah untuk menguji API Anda di Postman secara lancar:

### 🔌 Langkah 1: Persiapan Environment
1. Buka aplikasi **Postman**.
2. Buat **Environment** baru (klik ikon mata di pojok kanan atas -> *Add*).
3. Beri nama environment (misal: `AI-SaaS-Local`).
4. Tambahkan variable:
   * **Variable:** `base_url`
   * **Initial/Current Value:** `http://localhost:3000` (atau port server lokal Express Anda).
5. Klik **Save** dan pastikan environment tersebut aktif di dropdown kanan atas.

---

### 🔑 Langkah 2: Cara Mendapatkan & Memasang Token Clerk (Autentikasi)
Karena rute backend dilindungi oleh `auth` middleware, Anda akan mendapatkan pesan error `"Unauthorized"` jika tidak melampirkan JWT token Clerk.

1. **Cara Mendapatkan Token dari Browser:**
   * Buka aplikasi frontend React Anda di browser dan lakukan Login.
   * Klik kanan halaman -> pilih **Inspect (Periksa)** -> Buka tab **Console**.
   * Ketik instruksi berikut di konsol browser untuk mencetak token sesi aktif Anda:
     ```javascript
     await window.Clerk.session.getToken()
     ```
   * Salin string panjang token JWT yang muncul di konsol.
2. **Memasukkan Token di Postman:**
   * Buka request Anda di Postman.
   * Pilih tab **Headers** (di bawah kolom URL).
   * Tambahkan key baru:
     * **Key:** `Authorization`
     * **Value:** `Bearer <PASTE_TOKEN_CLERK_DI_SINI>` (pastikan ada spasi setelah kata *Bearer*).

---

### 📡 Langkah 3: Mengirim Kueri Request di Postman

Berikut adalah cara mengonfigurasi request Postman untuk berbagai endpoint aplikasi Anda:

#### A. Request GET (Mengambil Data Riwayat Kreasi)
* **Metode:** `GET`
* **URL:** `{{base_url}}/api/user/get-user-creations`
* **Headers:** Masukkan Bearer token Clerk seperti pada Langkah 2.
* Klik **Send**.
* *Ekspektasi Response:* Status `200 OK` dengan body JSON berisi array kreasi:
  ```json
  {
    "success": true,
    "creations": [ ... ]
  }
  ```

#### B. Request POST JSON (Generate Judul Blog)
* **Metode:** `POST`
* **URL:** `{{base_url}}/api/ai/generate-blog-title`
* **Headers:** Masukkan Bearer token Clerk.
* **Body:**
  * Klik tab **Body** di bawah URL.
  * Pilih opsi **raw** dan ubah format tipe data dropdown di sebelah kanan menjadi **JSON**.
  * Masukkan payload kueri:
    ```json
    {
      "prompt": "Generate a blog title for the keyword Artificial Intelligence in the category Technology"
    }
    ```
* Klik **Send**.
* *Ekspektasi Response:* Status `200 OK` (atau `201 Created`) berisi teks judul blog:
  ```json
  {
    "success": true,
    "content": "### 5 Rekomendasi Judul..."
  }
  ```

#### C. Request POST Multipart Form-Data (Upload Resume PDF)
Untuk fitur seperti **Resume Review** atau **Remove Background**, backend Anda menggunakan **Multer** untuk menerima unggahan file mentah melalui tipe form data.

* **Metode:** `POST`
* **URL:** `{{base_url}}/api/ai/resume-review`
* **Headers:** Masukkan Bearer token Clerk.
* **Body:**
  * Klik tab **Body** -> Pilih opsi **form-data**.
  * Pada baris kolom tabel:
    * **Key:** Ketik `"resume"` *(kunci ini wajib sama dengan nama yang diekspektasikan Multer di backend: upload.single("resume"))*.
    * **Tipe Key:** Arahkan kursor ke ujung kolom "Key", ubah tipe dari **Text** menjadi **File** via dropdown.
    * **Value:** Klik **Select Files** dan pilih file contoh resume berformat PDF di komputer Anda.
* Klik **Send**.
* *Ekspektasi Response:* Server akan mengekstrak teks PDF dan mengembalikan ulasan ulasan AI dalam waktu beberapa detik.
