# 🚀 AI SaaS Platform (Fullstack)

A fullstack AI-powered SaaS web application built with a modern tech stack. This project provides multiple AI tools such as image generation, background removal, object removal, resume review, and article writing — all in one platform.

---

## 🌐 Live Demo

* 🔗 Frontend (Vercel): https://ai-saas-one-livid.vercel.app/
* 🔗 Backend API (Vercel): https://ai-saas-server-liart.vercel.app/

---

## 📂 Project Structure

```
├── client     # Frontend (React + Vite + TailwindCSS)
└── server     # Backend (Express + Node.js)
```

### Client (Frontend)

* Built using **React 19 + Vite**
* Styled with **TailwindCSS**
* Authentication handled via **Clerk**
* Routing via **React Router**
* Features multiple pages:

  * Home
  * Dashboard
  * AI Tools (Image generation, background removal, etc.)
  * Blog titles & article writing
  * Resume review
  * Community page

### Server (Backend)

* Built using **Node.js + Express**
* Handles:

  * AI API requests
  * User authentication middleware
  * File uploads
  * Image processing
* Integrations:

  * Gemini AI (AI features via OpenAI SDK)
  * Stability AI (AI Image generation)
  * Cloudinary (image storage & processing)
  * Neon Database (serverless DB)
  * Clerk (authentication)

---

## ⚙️ Tech Stack

### Frontend

* React 19
* Vite 8
* TailwindCSS v4
* Axios
* React Router DOM
* React Markdown
* Clerk (Authentication)
* Lucide React (Icons)
* React Hot Toast (Notifications)

### Backend

* Express.js 5
* Gemini API (via OpenAI SDK)
* Stability AI API
* Cloudinary
* Multer (File uploads)
* unpdf (PDF text extraction)
* Neon Database (Serverless PostgreSQL)
* Clerk Express SDK
* Axios

---

## ✨ Features

* 🔐 Authentication (Clerk)
* 🎨 AI Image Generation
* 🧹 Background Removal
* 🪄 Object Removal from Images
* 📝 AI Article Writing
* 📰 Blog Title Generator
* 📄 Resume Review (PDF parsing)
* 👥 Community Page
* 📊 Dashboard Interface
* ☁️ Cloud Image Storage (Cloudinary)

---

## 📦 Installation

### 1. Clone the repository

```bash
git clone https://github.com/AlexMari0/ai-saas.git
cd ai-saas
```

---

### 2. Setup Server

```bash
cd server
npm install
```

Create a `.env` file in `/server`:

```
PORT=3000
GEMINI_API_KEY=your_gemini_key
STABILITY_API_KEY=your_stability_key
CLOUDINARY_CLOUD_NAME=your_name
CLOUDINARY_API_KEY=your_key
CLOUDINARY_API_SECRET=your_secret
DATABASE_URL=your_neon_db_url
CLERK_SECRET_KEY=your_clerk_secret
CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key
```

Run server:

```bash
npm run server
```

---

### 3. Setup Client

```bash
cd client
npm install
```

Create a `.env` file in `/client`:

```
VITE_BASE_URL=http://localhost:3000
VITE_CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key
```

Run client:

```bash
npm run dev
```

---

## 🚀 Deployment

### Client

* Deployed on **Vercel**
* Configured via `vercel.json`

### Server

* Can be deployed on **Vercel** or any Node.js hosting platform

---

## 📁 Key Directories

### Client

* `src/components` → Reusable UI components
* `src/pages` → Application pages
* `src/assets` → Images and static assets

### Server

* `controllers` → Business logic
* `routes` → API endpoints
* `middlewares` → Authentication & security
* `config` → External service configurations

---

## 🧪 Scripts

### Client

```bash
npm run dev      # Start development server
npm run build    # Build production
npm run preview  # Preview build
npm run lint     # Run ESLint
```

### Server

```bash
npm run server   # Start with nodemon
npm start        # Start production server
```

---

## 🔒 Authentication

Authentication is handled via **Clerk**, including:

* User sign-up / login
* Protected routes
* Backend middleware protection

---

## 📖 API Documentation

Semua endpoint di bawah ini memerlukan token JWT aktif di header `Authorization` dengan format: `Bearer <clerk_token>` (didapatkan dari SDK `@clerk/react`).

### 1. Endpoint Fitur AI (`/api/ai/*`)

* **`POST /api/ai/generate-article`**
  * **Fungsi:** Membuat artikel panjang secara otomatis menggunakan Gemini AI.
  * **Payload:** `{ "prompt": "Topik artikel", "length": 1000 }`
  * **Response (Success):** `{ "success": true, "content": "Teks artikel..." }`

* **`POST /api/ai/generate-blog-title`**
  * **Fungsi:** Menyarankan judul blog berdasarkan kata kunci dan kategori.
  * **Payload:** `{ "prompt": "Keyword/Kategori judul" }`
  * **Response (Success):** `{ "success": true, "content": "Rekomendasi judul..." }`

* **`POST /api/ai/generate-image`** *(Premium Only)*
  * **Fungsi:** Menghasilkan gambar berkualitas tinggi dari Stability AI dan disimpan ke Cloudinary.
  * **Payload:** `{ "prompt": "Deskripsi gambar", "publish": false }`
  * **Response (Success):** `{ "success": true, "content": "https://res.cloudinary.com/...image.png" }`

* **`POST /api/ai/remove-image-background`** *(Premium Only)*
  * **Fungsi:** Menghapus background gambar menggunakan filter AI Cloudinary.
  * **Payload:** `multipart/form-data` dengan field `"image"` (file gambar).
  * **Response (Success):** `{ "success": true, "content": "https://res.cloudinary.com/...image.png" }`

* **`POST /api/ai/remove-image-object`** *(Premium Only)*
  * **Fungsi:** Menghapus objek tertentu dari gambar menggunakan Cloudinary GenRemove.
  * **Payload:** `multipart/form-data` dengan field `"image"` (file gambar) dan `"object"` (nama objek yang akan dihapus, tipe teks).
  * **Response (Success):** `{ "success": true, "content": "https://res.cloudinary.com/...image.png" }`

* **`POST /api/ai/resume-review`** *(Premium Only)*
  * **Fungsi:** Mengekstrak PDF resume dan meninjau kekurangannya menggunakan Gemini AI.
  * **Payload:** `multipart/form-data` dengan field `"resume"` (file PDF).
  * **Response (Success):** `{ "success": true, "content": "Teks evaluasi resume..." }`

### 2. Endpoint Data User & Komunitas (`/api/user/*`)

* **`GET /api/user/get-user-creations`**
  * **Fungsi:** Mengambil semua daftar riwayat kreasi milik user aktif.
  * **Response (Success):** `{ "success": true, "creations": [...] }`

* **`GET /api/user/get-published-creations`**
  * **Fungsi:** Mengambil daftar kreasi yang dipublikasikan secara publik ke komunitas.
  * **Response (Success):** `{ "success": true, "creations": [...] }`

* **`POST /api/user/toggle-like-creation`**
  * **Fungsi:** Menyukai atau membatalkan suka (*like/unlike*) pada kreasi tertentu.
  * **Payload:** `{ "id": "id_creation_target" }`
  * **Response (Success):** `{ "success": true, "message": "Creation Liked" | "Creation Unliked" }`

---

## 🧠 Technical Decisions (Catatan Keputusan Teknis)

Berikut adalah ringkasan keputusan teknis arsitektur yang diambil dalam pengembangan platform ini:

1. **Neon Serverless PostgreSQL:**
   * *Keputusan:* Menggunakan `@neondatabase/serverless` menggantikan pool koneksi pg standar.
   * *Rasional:* Driver PG tradisional kurang andal dan rentan kehabisan batas koneksi (*connection limit*) di lingkungan serverless seperti Vercel. Neon PostgreSQL serverless client memungkinkan komunikasi kueri SQL instan melalui HTTP websockets dan dapat melakukan skala koneksi ke nol saat database tidak aktif untuk efisiensi biaya.
2. **Clerk untuk Autentikasi & Premium Subscription:**
   * *Keputusan:* Mendelegasikan autentikasi dan penanganan kuota user ke Clerk Auth-as-a-Service.
   * *Rasional:* Membangun sistem JWT lokal, penanganan hashing, dan session stores membutuhkan banyak waktu. Clerk menyediakan antarmuka siap pakai, integrasi login sosial yang aman, dan penanganan flag kuota gratis (`free_usage`) melalui `privateMetadata` user secara bawaan.
3. **Gemini API via OpenAI SDK Wrapper:**
   * *Keputusan:* Mengakses model `gemini-3-flash-preview` menggunakan integrasi OpenAI Node SDK.
   * *Rasional:* Gemini 3 Flash menawarkan kecepatan pemrosesan super cepat dan sangat efisien secara biaya. Memanggilnya via OpenAI SDK Wrapper memudahkan modularitas kode jika di masa depan arsitektur beralih ke model GPT OpenAI tanpa harus menulis ulang kode kueri SDK.
4. **Cloudinary AI Media Transformations:**
   * *Keputusan:* Offload pemrosesan gambar (Background Removal & Object Erasure) langsung ke CDN Cloudinary.
   * *Rasional:* Melakukan komputasi AI manipulasi gambar secara lokal memerlukan resource server CPU/GPU yang sangat besar. Menggunakan Cloudinary AI Transformation menyelesaikan pengeditan secara instan di cloud hanya dengan mengubah string URL hasil unggahan.

---

## 📌 Notes

* Ensure all environment variables are correctly configured
* Gemini & Stability AI usage may incur costs
* Cloudinary is required for image features
* Neon DB is used for scalable serverless database

---

## 🤝 Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss your ideas.

---

## 📄 License

This project is licensed under the ISC License.

---

## 💡 Future Improvements

* Payment integration (Stripe)
* Usage-based billing
* AI chat assistant
* More AI tools integration
* Improved analytics dashboard

---

## 👨‍💻 Author

Alex Mario Kristian

Built with ❤️ using modern fullstack technologies.
