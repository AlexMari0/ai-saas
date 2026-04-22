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

  * OpenAI (AI features)
  * Cloudinary (image storage & processing)
  * Neon Database (serverless DB)
  * Clerk (authentication)

---

## ⚙️ Tech Stack

### Frontend

* React
* Vite
* TailwindCSS
* Axios
* React Router DOM
* React Markdown
* Clerk (Authentication)
* Lucide React (Icons)

### Backend

* Express.js
* OpenAI API
* Cloudinary
* Multer (File uploads)
* Neon Database
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
PORT=5000
OPENAI_API_KEY=your_key
CLOUDINARY_CLOUD_NAME=your_name
CLOUDINARY_API_KEY=your_key
CLOUDINARY_API_SECRET=your_secret
DATABASE_URL=your_neon_db_url
CLERK_SECRET_KEY=your_clerk_secret
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
VITE_API_URL=http://localhost:5000
VITE_CLERK_PUBLISHABLE_KEY=your_clerk_key
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

## 📌 Notes

* Ensure all environment variables are correctly configured
* OpenAI usage may incur costs
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
