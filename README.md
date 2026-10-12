<div align="center">

# 🛍️ RentiGO

### Hyper-Local Peer-to-Peer Rental Marketplace

**Rent what you need from neighbours. Earn from what you don't use.**

[![Live Demo](https://img.shields.io/badge/Live-rentigo--pika2.vercel.app-000000?style=for-the-badge&logo=vercel&logoColor=white)](https://rentigo-pika2.vercel.app)

<p>
  <img src="https://img.shields.io/badge/React-19-61DAFB?style=flat-square&logo=react&logoColor=black" />
  <img src="https://img.shields.io/badge/Vite-7-646CFF?style=flat-square&logo=vite&logoColor=white" />
  <img src="https://img.shields.io/badge/Tailwind_CSS-4-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white" />
  <img src="https://img.shields.io/badge/Node.js-339933?style=flat-square&logo=nodedotjs&logoColor=white" />
  <img src="https://img.shields.io/badge/Express-4-000000?style=flat-square&logo=express&logoColor=white" />
  <img src="https://img.shields.io/badge/MongoDB-47A248?style=flat-square&logo=mongodb&logoColor=white" />
  <img src="https://img.shields.io/badge/Clerk-6C47FF?style=flat-square&logo=clerk&logoColor=white" />
  <img src="https://img.shields.io/badge/Razorpay-0C2451?style=flat-square&logo=razorpay&logoColor=white" />
  <img src="https://img.shields.io/badge/Google_Gemini-8E75B2?style=flat-square&logo=googlegemini&logoColor=white" />
  <img src="https://img.shields.io/badge/Leaflet-199900?style=flat-square&logo=leaflet&logoColor=white" />
</p>

</div>

---

## 📖 About

RentiGO is a peer-to-peer rental marketplace that turns underused belongings — cameras, tools, party gear, sports equipment — into income for owners and affordable access for neighbours. Listings are **location-aware**, payments are **secured via Razorpay**, identity is handled by **Clerk**, and every listing is enriched with **AI-generated insights from Google Gemini** (demand estimate, pricing advice, renter-friendly summary).

Built for **PIH 2026** by Team **Pika2**.

---

## ✨ Features

### 🧑‍🤝‍🧑 For Renters
- **Browse & search** listings with category, keyword and max-price filters plus pagination
- **Nearby discovery** – results sorted by distance from your location (Haversine)
- **Interactive map view** of listings powered by Leaflet + OpenStreetMap
- **Item details** with images, deposit info and AI insights (value score, demand, pricing tip, summary)
- **Book by date range** with total price calculation and secure Razorpay checkout
- **My Bookings** to track request → approval → payment → completion, or cancel
- **Wishlist** to save items for later
- **In-app messaging** with owners

### 🏪 For Lenders
- **List an item** with up to 5 image uploads, price/day, security deposit and map-pinned location (reverse geocoded via Nominatim)
- **Dashboard** to manage your listings and incoming rental requests
- **Approve / reject** booking requests
- **Earnings page** with payout overview and **PDF export** (jsPDF)

### 🌐 Platform
- **Role-based access**: `buyer`, `seller`, `admin` (admin can change roles and view all rentals)
- **Notifications** (success / info / warning / error) with read, read-all and delete
- **Light / dark theme** with system preference detection
- **Welcome onboarding modal** for first-time users
- **Responsive UI** built with a reusable component set (Button, Card, Badge, Modal, Input)

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| **Frontend** | <img src="https://skillicons.dev/icons?i=react,vite,tailwind,js" height="22" align="center" /> React 19, Vite 7, Tailwind CSS 4, React Router 7 |
| **UI / Utilities** | Lucide React icons, Axios, jsPDF |
| **Maps** | <img src="https://skillicons.dev/icons?i=leaflet" height="22" align="center" /> Leaflet, React-Leaflet, OpenStreetMap tiles, Nominatim |
| **Backend** | <img src="https://skillicons.dev/icons?i=nodejs,express" height="22" align="center" /> Node.js (ES modules), Express 4, Multer (uploads) |
| **Database** | <img src="https://skillicons.dev/icons?i=mongodb" height="22" align="center" /> MongoDB with Mongoose 8 |
| **Auth** | Clerk (`@clerk/clerk-react`, `@clerk/express`) |
| **Payments** | Razorpay (order creation + signature verification) |
| **AI** | Google Gemini (`gemini-1.5-flash`) via `@google/generative-ai` |
| **Deployment** | <img src="https://skillicons.dev/icons?i=vercel" height="22" align="center" /> Vercel (frontend), Render (backend) |
| **Tooling** | <img src="https://skillicons.dev/icons?i=git,github,npm" height="22" align="center" /> ESLint 9, Nodemon, Git |

---

## 🏗️ Architecture

```
┌──────────────────────┐        HTTPS / JWT (Clerk)        ┌──────────────────────────┐
│  React + Vite (SPA)  │ ───────────────────────────────▶ │  Express REST API        │
│  Tailwind · Leaflet  │                                   │  Clerk middleware        │
│  Clerk React         │ ◀─────────────────────────────── │  syncUser · roleMiddleware│
└──────────────────────┘                                   └───────┬──────────────────┘
                                                                   │
                           ┌───────────────┬──────────────────┬────┴──────────┐
                           ▼               ▼                  ▼               ▼
                      MongoDB          Razorpay          Google Gemini    /uploads
                     (Mongoose)     (orders/verify)     (AI insights)   (Multer disk)
```

**Request flow:** Clerk issues a session token on the client → `requireAuth()` validates it → `syncUser` upserts/loads the local `User` document → `requireRole()` guards privileged routes.

### Data Models
| Model | Key fields |
|-------|-----------|
| **User** | `clerkId`, `email`, `name`, `role` (buyer/seller/admin), `hasSeenWelcome` |
| **Product** | `title`, `description`, `category`, `pricePerDay`, `securityDeposit`, `images[]`, `owner`, `availability`, `location{address,lat,lng}`, `aiInsights{…}` |
| **Rental** | `product`, `renter`, `startDate`, `endDate`, `totalPrice`, `status` (pending/approved/confirmed/rejected/completed/cancelled), `orderId`, `paymentId`, `paymentStatus` |
| **Chat** | `participants[]`, embedded `messages[]`, `lastMessageAt` |
| **Notification** | `user`, `message`, `type`, `isRead`, `route` |

### API Reference

All routes are prefixed with `/api`. 🔒 = requires Clerk auth.

| Resource | Endpoints |
|----------|-----------|
| **Health** | `GET /health` |
| **Users** | `GET /users/me` 🔒 · `POST /users/welcome` 🔒 · `PATCH /users/role` 🔒 admin |
| **Products** | `GET /products` · `GET /products/:id` · `GET /products/mine` 🔒 · `POST /products` 🔒 · `PUT /products/:id` 🔒 · `DELETE /products/:id` 🔒 |
| **Rentals** | `POST /rentals` 🔒 · `GET /rentals/me` 🔒 · `GET /rentals/seller` 🔒 · `PATCH /rentals/:id/status` 🔒 · `PATCH /rentals/:id/cancel` 🔒 |
| **Payments** | `POST /payments/create-order` 🔒 · `POST /payments/verify` 🔒 |
| **Upload** | `POST /upload` 🔒 (multipart, up to 5 images) |
| **Chats** | `GET /chats` 🔒 · `POST /chats/connect` 🔒 · `POST /chats/:chatId/messages` 🔒 |
| **Notifications** | `GET /notifications` 🔒 · `PATCH /notifications/read-all` 🔒 · `PATCH /notifications/:id/read` 🔒 · `DELETE /notifications/:id` 🔒 |
| **Admin** | `GET /admin/rentals` 🔒 admin |

> `GET /products` accepts `category`, `search`, `maxPrice`, `page`, `limit`, `userLat`, `userLng`.
> Messaging uses short-interval polling (no WebSockets).

---

## 📁 Project Structure

```
PIH2026_Pika2/
├── backend/
│   ├── server.js                # Entry point (DB connect + listen)
│   └── src/
│       ├── app.js               # Express app, CORS, routes, error handling
│       ├── config/              # db.js, clerk.js
│       ├── controllers/         # user, product, rental, payment, chat, notification
│       ├── middleware/          # syncUser, roleMiddleware
│       ├── models/              # User, Product, Rental, Chat, Notification
│       ├── routes/              # One router per resource
│       └── services/            # geminiService, razorpayService
└── frontend/
    ├── vercel.json              # SPA rewrites
    └── src/
        ├── api/                 # axios instance & helpers
        ├── components/          # layout/, ui/, items/, modals, map
        ├── context/             # Theme, Location, Rental providers
        ├── pages/               # Home, Browse, ItemDetails, Dashboard, Earnings, ...
        └── routes/AppRoutes.jsx # Route table + layout
```

---

## 🚀 Getting Started

### Prerequisites
- Node.js 18+
- MongoDB (local or Atlas)
- Accounts/keys for [Clerk](https://clerk.com), [Razorpay](https://razorpay.com) (test mode is fine) and [Google AI Studio](https://aistudio.google.com) (Gemini)

### 1. Clone
```bash
git clone https://github.com/varadisthedev/PIH2026_Pika2.git
cd PIH2026_Pika2
```

### 2. Backend
```bash
cd backend
npm install
cp .env.example .env   # then fill in the values below
npm run dev            # http://localhost:5000
```

`backend/.env`
```env
PORT=5000
MONGO_URI=mongodb://localhost:27017/rentigo
CLERK_SECRET_KEY=
CLERK_PUBLISHABLE_KEY=
GEMINI_API_KEY=
RAZORPAY_KEY_ID=
RAZORPAY_KEY_SECRET=
CLIENT_URL=http://localhost:5173
```

### 3. Frontend
```bash
cd frontend
npm install
npm run dev            # http://localhost:5173
```

`frontend/.env`
```env
VITE_API_BASE_URL=http://localhost:5000/api
VITE_CLERK_PUBLISHABLE_KEY=
```

### Scripts
| Where | Command | Purpose |
|-------|---------|---------|
| backend | `npm run dev` | Start with Nodemon |
| backend | `npm start` | Start in production |
| frontend | `npm run dev` | Vite dev server |
| frontend | `npm run build` | Production build |
| frontend | `npm run lint` | ESLint |

---

## ☁️ Deployment
- **Frontend** → Vercel (`vercel.json` rewrites all routes to `index.html` for SPA routing)
- **Backend** → Render (set all backend env vars; add the frontend URL to `CLIENT_URL`)

---

## 👥 Team

Built with ❤️ by Team **Pika2** — **Varad** & **Sharvari**

---

<div align="center">
<sub>PIH 2026 · RentiGO</sub>
</div>
