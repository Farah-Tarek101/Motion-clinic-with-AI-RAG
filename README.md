# Motion Clinic with AI & RAG

A full-stack clinic management platform with an AI-powered medical assistant. Patients can book appointments, chat with a RAG-based health assistant, and upload medical images for analysis. Doctors and admins each have dedicated dashboards to manage care, review AI responses, and run clinic operations.

**Repository:** [github.com/Farah-Tarek101/Motion-clinic-with-AI-RAG](https://github.com/Farah-Tarek101/Motion-clinic-with-AI-RAG)

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [User Roles](#user-roles)
- [AI & RAG System](#ai--rag-system)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Environment Variables](#environment-variables)
- [Running Locally](#running-locally)
- [API Overview](#api-overview)
- [Deployment](#deployment)
- [Model Setup](#model-setup)
- [Medical Disclaimer](#medical-disclaimer)

---

## Overview

Motion Clinic is a multi-application healthcare system built for orthopedic and physical therapy services. It combines:

- A **patient-facing website** for registration, appointments, departments, and AI chat
- An **admin dashboard** for managing doctors, admins, and patient messages
- A **doctor dashboard** for appointments, patients, notifications, and reviewing AI outputs
- A **Node.js + Python backend** that powers authentication, appointments, notifications, and the RAG medical assistant

The AI layer uses **Retrieval-Augmented Generation (RAG)** over a patient case dataset, plus optional **medical image analysis** for X-ray, MRI, and CT scans.

---

## Features

### Patient Portal (`frontend`)

- User registration, login, and Google OAuth
- Multi-language support (English, Arabic, French)
- Browse departments (Orthopedics, Physical Therapy, and more)
- Book and manage appointments
- Patient profile and medical history
- Password reset and account deletion
- **AI Chat** — ask health questions and get RAG-based responses
- **Chat history** — view past AI conversations
- Contact clinic and view clinic information

### Admin Dashboard (`dashboard`)

- Admin login
- Dashboard overview
- Add and manage doctors
- Add new admins
- View and respond to contact messages
- Doctor list management

### Doctor Dashboard (`doctor-dashboard`)

- Doctor login
- Today's and upcoming appointments
- Patient list and doctor-specific patients
- Completed appointments
- Update patient medical information
- Notification bell for pending AI reviews
- **Doctor review dashboard** — review AI chat and image analysis results before they are finalized
- Theme support (light/dark)

### Backend (`backend`)

- REST API with Express.js
- MongoDB database with Mongoose
- JWT authentication and role-based access control
- Google OAuth via Passport.js
- Email notifications (password reset, etc.)
- Appointment scheduling and management
- In-app notifications
- Contact message handling
- **RAG medical assistant** (Python + FAISS + Sentence Transformers)
- **Medical image analysis** (X-ray, MRI, CT)
- Chat history storage
- Doctor notification when AI responses need review

---

## Architecture

```
┌─────────────────┐   ┌──────────────────┐   ┌─────────────────────┐
│    Frontend     │   │ Admin Dashboard  │   │  Doctor Dashboard   │
│   (Patients)    │   │                  │   │                     │
│  React + Vite   │   │  React + Vite    │   │   React + Vite      │
└────────┬────────┘   └────────┬─────────┘   └──────────┬──────────┘
         │                     │                        │
         └─────────────────────┼────────────────────────┘
                               │  REST API (HTTPS)
                               ▼
                    ┌──────────────────────┐
                    │   Backend (Node.js)  │
                    │   Express + MongoDB  │
                    └──────────┬───────────┘
                               │  spawns Python subprocess
                               ▼
                    ┌──────────────────────┐
                    │   AI / RAG Layer     │
                    │  Python + PyTorch    │
                    │  FAISS + Transformers│
                    └──────────────────────┘
```

---

## Project Structure

```
motion-clinic-main/
├── backend/                 # Express API + AI integration
│   ├── controller/          # Route handlers (users, AI, appointments, etc.)
│   ├── models/              # Mongoose schemas
│   ├── router/              # API routes
│   ├── middlewares/         # Auth, upload, error handling
│   ├── utils/               # RAG bridge, email, JWT helpers
│   ├── scripts/             # Python RAG and image analysis scripts
│   ├── data/                # Datasets, embeddings, FAISS index, models
│   ├── database/            # MongoDB connection
│   └── server.js            # Entry point
│
├── frontend/                # Patient website (React + Vite)
│   └── src/
│       ├── Pages/           # Home, Login, AI Chat, Appointments, etc.
│       ├── components/      # Navbar, Footer, forms
│       └── locales/         # i18n translations
│
├── dashboard/               # Admin dashboard (React + Vite)
│   └── src/components/      # Doctors, Messages, Add Doctor/Admin
│
├── doctor-dashboard/        # Doctor dashboard (React + Vite)
│   └── src/pages/           # Appointments, Patients, Settings
│
├── package.json             # Root workspace helper
├── vercel.json              # Vercel deployment config
└── README.md
```

---

## User Roles

| Role    | Access |
|---------|--------|
| **Patient** | Frontend app — appointments, profile, AI chat, chat history |
| **Doctor**  | Doctor dashboard — patients, appointments, AI review, notifications |
| **Admin**   | Admin dashboard — manage doctors/admins, view messages |

Authentication is handled with JWT tokens. Protected routes check the user's role before allowing access.

---

## AI & RAG System

### Text-based RAG Assistant

The backend connects Node.js to Python through `backend/utils/ragModel.js`, which runs `scripts/lightweight_rag_processor.py`.

**How it works:**

1. Patient sends a question through the AI Chat UI
2. Backend forwards the query to the Python RAG processor
3. The processor embeds the query using **Sentence Transformers**
4. **FAISS** retrieves similar patient cases from the medical dataset
5. A response is generated from retrieved complaints, diagnoses, and treatment plans
6. The conversation is saved to chat history
7. Doctors can be notified to review pending AI responses

**Required model files** (in `backend/data/models/`):

- `faiss_index.bin` — vector search index
- `corpus_embeddings.npy` — document embeddings
- `cleaned_patients.csv` — processed patient case data
- `model_config.json` — model configuration
- `embedder_model/` — sentence transformer model files

### Medical Image Analysis

The backend supports uploading and analyzing medical images:

- **X-ray**
- **MRI**
- **CT**

Image analysis uses Python scripts in `backend/scripts/` with embeddings stored under `backend/data/medical_images/`. Results can be saved to chat history and flagged for doctor review.

### Downloading AI Models

For full image analysis capabilities, run:

```bash
cd backend
pip install -r requirements.txt
python download_models.py
```

This downloads Phi-2, DenseNet121, and BLIP models used for text generation and image processing.

> **Note:** The large `model.safetensors` embedder file may need to be downloaded separately. See [Model Setup](#model-setup).

---

## Tech Stack

| Layer | Technologies |
|-------|-------------|
| **Frontend** | React 18, Vite, React Router, Axios, i18next, React Toastify |
| **Admin / Doctor UI** | React, Vite, React Router, React Icons |
| **Backend** | Node.js, Express, Mongoose, JWT, Passport (Google OAuth), Multer, Nodemailer |
| **Database** | MongoDB |
| **AI / ML** | Python, PyTorch, Transformers, Sentence Transformers, FAISS, scikit-learn, Pillow |
| **Deployment** | Vercel (frontend), Railway (backend) |

---

## Prerequisites

- **Node.js** 18+ and npm
- **Python** 3.9+
- **MongoDB** database (local or MongoDB Atlas)
- **Git** and **Git LFS** (for large model files)
- Google OAuth credentials (optional, for Google login)
- SMTP credentials (optional, for password reset emails)

---

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/Farah-Tarek101/Motion-clinic-with-AI-RAG.git
cd Motion-clinic-with-AI-RAG
```

### 2. Install backend dependencies

```bash
cd backend
npm install
pip install -r requirements.txt
```

### 3. Install frontend dependencies

```bash
cd ../frontend
npm install
```

### 4. Install dashboard dependencies

```bash
cd ../dashboard
npm install
```

### 5. Install doctor dashboard dependencies

```bash
cd ../doctor-dashboard
npm install
```

### 6. Set up environment variables

Create a `.env` file in `backend/` (see [Environment Variables](#environment-variables)).

Create `.env` files in each frontend app with:

```env
VITE_API_URL=http://localhost:4000
```

---

## Environment Variables

### Backend (`backend/.env`)

```env
# Server
PORT=4000
NODE_ENV=development

# Database
MONGO_URI=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/clinic

# JWT
JWT_SECRET_KEY=your_secure_secret_key
JWT_EXPIRES=7d

# CORS — frontend URLs
FRONTEND_URL_ONE=http://localhost:5173
FRONTEND_URL_TWO=http://localhost:5174
FRONTEND_URL_THREE=http://localhost:5175

# Google OAuth (optional)
CLIENT_ID=your_google_client_id
CLIENT_SECRET=your_google_client_secret
CALLBACK_URL=http://localhost:4000/api/v1/user/auth/google/callback

# Email (optional)
SMTP_MAIL=your_email@gmail.com
SMTP_PASSWORD=your_app_password

# Python
PYTHONUNBUFFERED=1
```

### Frontend apps

Each React app (`frontend`, `dashboard`, `doctor-dashboard`) needs:

```env
VITE_API_URL=http://localhost:4000
```

---

## Running Locally

Open **four terminals** and run each app:

**Terminal 1 — Backend**

```bash
cd backend
npm run dev
```

Server runs at `http://localhost:4000`

**Terminal 2 — Patient Frontend**

```bash
cd frontend
npm run dev
```

Runs at `http://localhost:5173`

**Terminal 3 — Admin Dashboard**

```bash
cd dashboard
npm run dev
```

**Terminal 4 — Doctor Dashboard**

```bash
cd doctor-dashboard
npm run dev
```

### Health check

```bash
curl http://localhost:4000/api/v1/user/health
```

Expected response:

```json
{
  "status": "OK",
  "message": "Server is running",
  "timestamp": "..."
}
```

---

## API Overview

### Users & Auth (`/api/v1/user`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/patient/register` | Register a patient |
| POST | `/login` | Login |
| GET | `/profile` | Get user profile |
| GET | `/auth/google` | Google OAuth login |
| POST | `/password/forgot` | Request password reset |
| PUT | `/password/reset/:token` | Reset password |
| GET | `/doctors` | List all doctors |
| POST | `/doctor/addnew` | Add doctor (admin) |
| POST | `/admin/addnew` | Add admin (admin) |

### Appointments (`/api/v1/appointment`)

Book, view, update, and cancel appointments.

### Messages (`/api/v1/message`)

Send contact messages; admins can view all messages.

### Notifications (`/api/v1/notifications`)

Get, mark as read, and delete notifications.

### AI (`/api/ai`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/chat` | AI medical chat (RAG) |
| POST | `/detailed` | Detailed RAG response |
| POST | `/image-analyze` | Upload and analyze medical image |
| GET | `/health` | AI service health check |
| GET | `/history/:userId` | Chat history (authenticated) |

### Chat History (`/api/chat-history`)

Store and retrieve AI conversation sessions.

---

## Deployment

### Backend — Railway

The backend is configured for [Railway](https://railway.app) with Node.js and Python support.

Key files:

- `backend/railway.json`
- `backend/nixpacks.toml`
- `backend/Procfile`
- `backend/start.sh`

Set all environment variables in the Railway dashboard. See `backend/RAILWAY_DEPLOYMENT.md` for the full guide.

Health endpoint: `GET /api/v1/user/health`

### Frontend — Vercel

The root `vercel.json` routes:

- `/api/*` → backend API
- `/*` → frontend static build

Deploy each dashboard as a separate Vercel project and set `VITE_API_URL` to your Railway backend URL.

---

## Model Setup

The embedder model file `backend/data/models/embedder_model/model.safetensors` (~90 MB) is tracked with **Git LFS** and may not be included in a fresh clone.

To restore it locally:

```bash
cd backend
git lfs pull
# or regenerate/download via project scripts
python download_models.py
```

Without this file, the RAG system may fall back or fail to initialize until model artifacts are present.

---

## Medical Disclaimer

This application provides **general health information and educational support only**. It is **not** a substitute for professional medical advice, diagnosis, or treatment.

- Always consult a qualified healthcare provider for medical decisions
- Do not use AI responses for emergency situations — call emergency services instead
- AI outputs may require doctor review before being shared with patients
- Patient data should be handled according to applicable privacy regulations

---

## License

This project is provided as-is for educational and development purposes.

---

## Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Open a pull request

---

**Motion Clinic** — Smart clinic management powered by AI and RAG.
