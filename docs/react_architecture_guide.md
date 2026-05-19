# ⚛️ Panduan Arsitektur React: Hooks, State Management & Komponen

Dokumen ini memberikan tinjauan mendalam tentang konsep **React Hooks**, strategi **State Management (Context API vs Props Drilling)**, serta penjelasan **Struktur Komponen** (Reusable vs Page-Level) yang diimplementasikan pada frontend aplikasi **AI SaaS Platform**.

---

## 1. Tinjauan React Hooks dalam Aplikasi

React Hooks adalah fungsi bawaan yang memungkinkan komponen fungsional menggunakan state dan fitur React lainnya tanpa menulis class. Di aplikasi ini, terdapat perpaduan antara **Standard React Hooks**, **React Router Hooks**, dan **Clerk Custom Hooks**.

### A. Standard React Hooks
* **`useState`**
  * *Tujuan:* Mengelola state lokal yang dinamis di dalam komponen.
  * *Penggunaan di Aplikasi:*
    * Menangani status loading (spinner) saat request API dikirim (misal: `const [loading, setLoading] = useState(false)` di `BlogTitles.jsx`).
    * Menyimpan hasil respon teks/gambar dari AI (misal: `const [content, setContent] = useState("")`).
    * Mengontrol visibilitas menu seluler (sidebar) di `Layout.jsx` (`const [sidebar, setSidebar] = useState(false)`).
    * Menyimpan data input form pengguna secara real-time.
* **`useEffect`**
  * *Tujuan:* Melakukan efek samping (*side effects*) seperti pemanggilan API eksternal saat komponen pertama kali dimuat (*mount*) atau saat dependensi berubah.
  * *Penggunaan di Aplikasi:*
    * Mengambil riwayat kreasi pengguna dari database Neon saat halaman Dashboard pertama kali dimuat (`Dashboard.jsx`):
      ```javascript
      useEffect(() => {
        getDashboardData();
      }, []);
      ```

### B. Clerk Custom Hooks (Autentikasi & Pengguna)
* **`useAuth()`**
  * Mengakses token autentikasi (JWT) untuk validasi backend secara aman (`const { getToken } = useAuth()`).
  * Mengecek secara berkala hak akses plan pengguna (`const { has } = useAuth(); const isPremium = has({ plan: 'premium' });`).
* **`useUser()`**
  * Mendapatkan data profil pengguna yang sedang login langsung dari server Clerk (misal: mengambil foto avatar `user.imageUrl` dan nama lengkap `user.fullName` di `Sidebar.jsx`).
* **`useClerk()`**
  * Memanggil fungsi bawaan Clerk tingkat aplikasi seperti `signOut` untuk keluar log dan `openUserProfile` untuk membuka modal pengaturan profil pengguna.

### C. React Router Hooks
* **`useNavigate()`**
  * Melakukan navigasi halaman secara terprogram (*programmatic routing*) tanpa memicu reload penuh browser (misal: kembali ke landing page saat logo diklik).

---

## 2. Pemahaman State Management: Context API vs Props Drilling

Dalam mengelola dan membagikan data antar komponen React, aplikasi ini menerapkan kombinasi metode tergantung pada skala kebutuhan data.

### 📐 Props Drilling
* **Definisi:** Proses mengalirkan data (state atau fungsi) dari komponen induk (*parent*) ke komponen anak (*child*) melalui properti (*props*), bahkan jika komponen perantara tidak membutuhkan data tersebut.
* **Penerapan di Aplikasi:**
  * Di `client/src/pages/Layout.jsx`, state menu seluler dibuat:
    ```javascript
    const [sidebar, setSidebar] = useState(false);
    ```
  * State dan pengubahnya tersebut diturunkan langsung ke komponen `<Sidebar>`:
    ```jsx
    <Sidebar sidebar={sidebar} setSidebar={setSidebar} />
    ```
* **Kapan Harus Digunakan?**
  * Hierarki komponen relatif pendek (maksimal 2-3 tingkat).
  * Data tersebut bersifat spesifik untuk lingkup kecil komponen anak tersebut.
  * *Kelebihan:* Sangat eksplisit, mudah ditelusuri arah aliran datanya, dan tidak memerlukan boilerplate tambahan.

### 🌐 Context API
* **Definisi:** Fitur bawaan React untuk membagikan data global ke seluruh pohon komponen tanpa perlu mengalirkannya secara manual di setiap level perantara (*prop drilling*).
* **Penerapan di Aplikasi (Menggunakan Clerk Provider):**
  * Clerk membungkus aplikasi utama di `main.jsx` menggunakan `<ClerkProvider>`.
  * Akibatnya, komponen di mana saja (seperti `Sidebar.jsx`, `Layout.jsx`, atau `Dashboard.jsx`) bisa langsung menggunakan data autentikasi dan detail pengguna hanya dengan memanggil hook `useUser()` atau `useAuth()`, tanpa perlu dioper satu per satu dari `Layout.jsx`.
* **Kapan Harus Digunakan?**
  * Data diakses oleh banyak komponen di tingkat pohon hierarki yang berbeda jauh.
  * Data bersifat global untuk seluruh aplikasi (misal: Status Autentikasi, Preferensi Tema Gelap/Terang, Pengaturan Bahasa, atau State Keranjang Belanja).
  * *Kelebihan:* Menghindari *"Prop Drilling Nightmare"*, menjaga komponen perantara tetap bersih dan fokus pada tugasnya.

---

## 3. Struktur Komponen: Reusable vs Page-Level

Untuk kemudahan pemeliharaan dan modularitas kode, komponen dibagi menjadi dua kategori utama di dalam folder `client/src/`:

```
client/src/
├── components/     # 🧱 Reusable Components
└── pages/          # 📄 Page-Level Components
```

### 🧱 A. Reusable Components (Komponen Dapat Digunakan Kembali)
Komponen ini diletakkan di bawah folder `src/components/`. Ciri khasnya adalah bersifat modular, fokus pada presentasi visual, dan menerima input dinamis melalui props agar dapat disematkan di mana saja.

1. **`CreationItem.jsx`:**
   * Menampilkan item kreasi individual (gambar, artikel, blog title, dll) lengkap dengan tombol preview dan detail format. Digunakan berulang-ulang di halaman Dashboard dan Community.
2. **`Sidebar.jsx`:**
   * Bilah menu samping kiri yang menerima props status buka/tutup seluler (`sidebar`, `setSidebar`) untuk penanganan CSS responsive dinamis.
3. **`Navbar.jsx`:**
   * Navigasi atas yang modular berisi tautan menu utama dan penanganan perpindahan halaman.
4. **`Hero.jsx`, `AiTools.jsx`, `Testimonial.jsx`, `Plan.jsx`, `Footer.jsx`:**
   * Komponen presentasional mandiri yang disusun secara modular untuk membentuk kerangka Landing Page (`Home.jsx`).

---

### 📄 B. Page-Level Components (Komponen Tingkat Halaman)
Komponen ini diletakkan di bawah folder `src/pages/`. Bertanggung jawab penuh atas logika bisnis utama, penanganan routing, integrasi API Backend, pengiriman form data, dan mendefinisikan layout visual halaman tersebut.

1. **`Layout.jsx`:**
   * Berfungsi sebagai *wrapper layout* utama setelah login. Mengatur grid global halaman (mengintegrasikan komponen `Sidebar` dan menyalurkan konten rute dinamis melalui `<Outlet />`).
2. **`Home.jsx`:**
   * Halaman Landing Page utama yang menggabungkan seluruh komponen pemasaran (Hero, Testimonial, Pricing Plan) untuk pengguna sebelum login.
3. **`Dashboard.jsx`:**
   * Halaman ringkasan user setelah login. Mengatur pemanggilan data riwayat kreasi personal (`get-user-creations`) dan menyajikan data metrik agregat (Total Creations, Active Plan).
4. **Halaman Fitur AI Individual (`WriteArticle.jsx`, `BlogTitles.jsx`, `GenerateImages.jsx`, `RemoveBackground.jsx`, `RemoveObject.jsx`, `ReviewResume.jsx`):**
   * Halaman khusus untuk masing-masing tool. Mengelola form submission yang kompleks, validasi payload input, penanganan upload file (PDF/Image), dan rendering visual interaktif dari respons AI.
5. **`Community.jsx`:**
   * Halaman sosial terpusat yang memanggil data kreasi publik dari database (`get-published-creations`) untuk ditampilkan ke semua pengguna.
