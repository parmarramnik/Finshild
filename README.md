# FinShield — Cybersecurity Simulation & Risk Assessment Platform

FinShield is a full-stack MERN application that enables organizations to run controlled phishing simulations, track employee responses in real time, assess risk across departments, and gamify cybersecurity awareness training.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture Overview](#architecture-overview)
- [Scoring System](#scoring-system)
- [Role-Based Access](#role-based-access)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Environment Variables](#environment-variables)
  - [Running the App](#running-the-app)
  - [Seeding the Database](#seeding-the-database)
- [Project Structure](#project-structure)
- [API Reference](#api-reference)
- [Phishing Simulation Flow](#phishing-simulation-flow)
- [Employee Onboarding Flow](#employee-onboarding-flow)
- [Security & Privacy Notes](#security--privacy-notes)
- [Screenshots](#screenshots)

---

## Features

- **Phishing Campaign Management** — Create, launch, reset, and complete phishing email campaigns targeting specific departments
- **Real-Time Tracking** — Track email opens (pixel), link clicks, credential form submissions, and reports independently
- **Gamification** — Points-based scoring with security level badges (Beginner / Aware / Security Champion)
- **Leaderboard** — Live-refreshing employee security score rankings with department breakdown
- **AI-Powered Explanations** — Rule-based phishing indicator detection explains why an email was a drill
- **Template Engine** — Built-in AI template generator (4 themes × 3 difficulty levels) and custom HTML templates
- **Bulk Employee Upload** — CSV / Excel import to onboard employees instantly
- **Analytics Dashboard** — Department risk scores, funnel charts, click/report rates, high-risk employees
- **PDF Report Generation** — One-click PDF export of the full analytics dashboard
- **Audit Logs** — Admin-visible log of all major actions in the organization
- **Multi-Tenant** — Full data isolation between organizations

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18, React Router v6, Tailwind CSS, Recharts |
| Backend | Node.js, Express.js |
| Database | MongoDB with Mongoose |
| Auth | JWT (jsonwebtoken), bcryptjs |
| Email | Nodemailer (SMTP / Gmail App Password) |
| File Upload | Multer, csv-parser, xlsx |
| PDF Export | jsPDF, jspdf-autotable |

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        Browser (React)                      │
│  LoginPage  RegisterPage  DashboardPage  CampaignDetailPage │
│  LeaderboardPage  AnalyticsDashboard  PhishingDrillPage     │
└───────────────────────┬─────────────────────────────────────┘
                        │ HTTP / REST (Axios + JWT)
┌───────────────────────▼─────────────────────────────────────┐
│                  Express.js API Server                      │
│                                                             │
│  /api/auth        /api/campaigns    /api/templates          │
│  /api/users       /api/track        /api/analytics          │
│  /api/gamification  /api/reports    /api/audit              │
│                                                             │
│  Middleware: JWT auth · RBAC authorize() · Multer           │
│  Services:  emailService · gamificationService · aiService  │
└───────────────────────┬─────────────────────────────────────┘
                        │ Mongoose ODM
┌───────────────────────▼─────────────────────────────────────┐
│                        MongoDB                              │
│  User  Organization  Campaign  Template  InteractionLog     │
│  TrackingToken  EmailDeliveryLog  Report  AuditLog          │
└─────────────────────────────────────────────────────────────┘
```

---

## Scoring System

FinShield uses two separate, complementary scoring metrics:

### 1. Points (Gamification)
Cumulative integer stored on the `User` document. Cannot go below 0.

| User Action | Points |
|---|---|
| Reports phishing email | +10 |
| Ignores email (never interacts) | +5 *(awarded when campaign is completed)* |
| Clicks the phishing link | -5 |
| Submits credentials on fake form | -10 |

Points determine the **Security Level badge**:

| Points Range | Badge |
|---|---|
| 0 – 20 | Beginner |
| 21 – 50 | Aware |
| 51+ | Security Champion |

---

### 2. Security Score (0 – 100)
Computed live from `InteractionLog` aggregation. A higher score is better.

```
Security Score = 100
  - (clicked / total) × 30
  - (submitted / total) × 40
  + (reported / total) × 20
```

Shown on the **Leaderboard** and **Employee Dashboard**.

---

### 3. Vulnerability Score (0 – 100)
Only shown in the **Admin Dashboard** high-risk employees table. A higher score means more at risk.

```
Vulnerability Score = (clicked × 20 + submitted × 40) / total × (100 / 60)
```

---

## Role-Based Access

| Feature | Admin | Cybersecurity | Analyst | Employee |
|---|:---:|:---:|:---:|:---:|
| Create / launch campaigns | ✅ | | | |
| View campaign details | ✅ | ✅ | ✅ | |
| Manage templates | ✅ | ✅ | | |
| Upload / manage employees | ✅ | | | |
| View analytics dashboard | ✅ | ✅ | ✅ | |
| View leaderboard | ✅ | ✅ | ✅ | |
| Create reports | | | ✅ | |
| View audit logs | ✅ | | | |
| Personal dashboard | ✅ | ✅ | ✅ | ✅ |

---

## Getting Started

### Prerequisites

- **Node.js** v18+
- **MongoDB** running locally on port `27017` (or a MongoDB Atlas URI)
- **Gmail account** with an [App Password](https://support.google.com/accounts/answer/185833) for SMTP

---

### Installation

```bash
# Clone the repository
git clone https://github.com/Gracy1475/Finshield.git
cd Finshield

# Install all dependencies (backend + frontend)
npm run install-all
```

---

### Environment Variables

#### `backend/.env`

```ini
PORT=5000
MONGODB_URI=mongodb://127.0.0.1:27017/finshield

JWT_SECRET=your_strong_jwt_secret_here

# Gmail SMTP — use an App Password, not your account password
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASS=your_16_char_app_password

# Tracking URLs — must be reachable from target devices
# Use your LAN IP when testing across devices on the same network
FRONTEND_URL=http://192.168.x.x:3000
BACKEND_URL=http://192.168.x.x:5000
```

> **Important:** `BACKEND_URL` is embedded directly into phishing email links. If you are testing across multiple devices on the same Wi-Fi network, replace `localhost` with your machine's LAN IP address (e.g., `192.168.1.79`). Run `ipconfig` (Windows) or `ifconfig` (Mac/Linux) to find it.

#### `frontend/.env`

```ini
REACT_APP_API_URL=http://192.168.x.x:5000/api
```

---

### Running the App

Open two terminals:

```bash
# Terminal 1 — Backend (http://localhost:5000)
npm run backend

# Terminal 2 — Frontend (http://localhost:3000)
npm run frontend
```

---

### Seeding the Database

```bash
npm run seed
```

This creates a sample organization, admin user, templates, and employees for testing.

**Default seed credentials:**

| Role | Email | Password |
|---|---|---|
| Admin | admin@finshield.com | Admin@123 |
| Employee | (see seed.js) | FinShield@2024 |

---

## Project Structure

```
Finshield/
├── package.json                  # Root scripts: backend, frontend, seed, install-all
│
├── backend/
│   ├── server.js                 # Express app entry point
│   ├── seed.js                   # Database seed script
│   ├── .env                      # Environment variables (not committed)
│   │
│   ├── config/
│   │   └── db.js                 # Mongoose connection
│   │
│   ├── middleware/
│   │   └── auth.js               # JWT verification + RBAC authorize()
│   │
│   ├── models/
│   │   ├── User.js
│   │   ├── Organization.js
│   │   ├── Campaign.js
│   │   ├── Template.js
│   │   ├── InteractionLog.js     # Primary per-user tracking record
│   │   ├── TrackingToken.js      # Lightweight token state
│   │   ├── EmailDeliveryLog.js
│   │   ├── Report.js
│   │   └── AuditLog.js
│   │
│   ├── routes/
│   │   ├── auth.js               # Register, login, /me
│   │   ├── campaigns.js          # CRUD, launch, reset, complete, stats
│   │   ├── users.js              # Bulk upload, add single, list
│   │   ├── templates.js          # CRUD + AI generation
│   │   ├── tracking.js           # Open pixel, click, report, submit (unauthenticated)
│   │   ├── analytics.js          # Dashboard data, filters, insights
│   │   ├── gamification.js       # Leaderboard, dept ranking, employee XAI
│   │   ├── reports.js            # Analyst reports
│   │   └── audit.js              # Admin audit logs
│   │
│   └── services/
│       ├── emailService.js       # Nodemailer SMTP + tracking pixel injection
│       ├── gamificationService.js# Points update + level badge logic
│       ├── aiService.js          # Rule-based phishing indicator engine
│       └── auditService.js       # Audit log helper
│
└── frontend/
    ├── .env                      # REACT_APP_API_URL
    ├── tailwind.config.js
    │
    └── src/
        ├── App.js                # Router + AuthProvider
        ├── context/
        │   └── AuthContext.js    # Global auth state
        ├── services/
        │   └── api.js            # Axios instance with JWT interceptors
        ├── components/
        │   ├── Navbar.js
        │   └── ProtectedRoute.js
        └── pages/
            ├── LoginPage.js
            ├── RegisterPage.js       # 3-step org create/join wizard
            ├── DashboardPage.js      # Role-split: admin analytics / employee stats
            ├── CampaignPage.js       # Campaign list + actions
            ├── CampaignDetailPage.js # Live stats (5s refresh) + user table
            ├── TemplatePage.js       # Template CRUD + AI generator
            ├── UserUploadPage.js     # CSV/Excel upload + user table
            ├── AnalyticsDashboard.js # Charts + PDF export
            ├── LeaderboardPage.js    # Security score rankings (30s refresh)
            ├── PhishingDrillPage.js  # Phishing simulation landing page
            └── ReportPage.js         # Analyst report viewer
```

---

## API Reference

### Authentication — `/api/auth`

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/register` | No | Create or join an organization |
| POST | `/login` | No | Login, returns JWT |
| GET | `/me` | Yes | Get current user profile |
| GET | `/organizations` | No | List all organizations (for join flow) |

### Campaigns — `/api/campaigns`

| Method | Endpoint | Roles | Description |
|---|---|---|---|
| GET | `/` | admin, cybersecurity, analyst | List all campaigns |
| GET | `/:id` | admin, cybersecurity, analyst | Get campaign details |
| POST | `/` | admin | Create campaign (draft) |
| PUT | `/:id` | admin | Update campaign |
| POST | `/:id/launch` | admin | Send phishing emails to targets |
| POST | `/:id/reset` | admin | Delete tracking data, revert to draft |
| POST | `/:id/complete` | admin | Mark complete, award ignore points |
| GET | `/:id/stats` | admin, cybersecurity, analyst | Live interaction stats |

### Users — `/api/users`

| Method | Endpoint | Roles | Description |
|---|---|---|---|
| GET | `/` | admin | List employees |
| GET | `/departments` | admin | List distinct departments |
| POST | `/upload` | admin | Bulk import via CSV / Excel |
| POST | `/add` | admin | Add single employee |

### Tracking — `/api/track` *(no auth required)*

| Method | Endpoint | Description |
|---|---|---|
| GET | `/open/:token` | Email open pixel (1×1 GIF) |
| GET | `/click/:token` | Record click, redirect to PhishingDrillPage |
| POST | `/report/:token` | User reports the email as suspicious |
| POST | `/submit/:token` | User submits fake form, returns XAI explanation |
| GET | `/info/:token` | Get interaction state for drill page |

### Analytics — `/api/analytics`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/my-stats` | Personal stats for the logged-in employee |
| GET | `/dashboard-v2` | Full org analytics with filters |
| GET | `/insights` | AI-generated insight cards |
| GET | `/risk-score` | Department risk scores |
| GET | `/email-delivery` | SMTP delivery status |

### Gamification — `/api/gamification`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/leaderboard` | Employees ranked by security score |
| GET | `/department-ranking` | Departments ranked by average points |
| GET | `/employee/:id` | Deep per-employee XAI breakdown |

### Templates — `/api/templates`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/` | List templates |
| POST | `/create` | Create custom template |
| PUT | `/update/:id` | Update template |
| DELETE | `/delete/:id` | Delete template |
| POST | `/generate-ai` | Generate from theme + difficulty + department |

---

## Phishing Simulation Flow

```
1. Admin creates a phishing Template (custom or AI-generated)
         ↓
2. Admin creates a Campaign targeting departments + launch date
         ↓
3. Admin launches the campaign
   → Backend finds all employees in target departments
   → Creates one TrackingToken (UUID) per employee
   → Creates one InteractionLog per employee
   → Sends phishing email via SMTP with:
       • Open tracking pixel  →  GET /api/track/open/:token
       • Phishing link        →  GET /api/track/click/:token
         ↓
4. Employee receives email
   [Opens email]   → open pixel fires  → email_opened = true
   [Clicks link]   → click route fires → link_clicked = true, -5 pts
                   → redirected to /phishing/:token
         ↓
5. PhishingDrillPage loads
   [Reports email] → POST /api/track/report/:token → reported_email = true, +10 pts
   [Ignores]       → fake login form shown
   [Submits form]  → POST /api/track/submit/:token → form_submitted = true, -10 pts
                   → XAI explanation of phishing indicators shown
         ↓
6. Admin views live stats in CampaignDetailPage (auto-refreshes every 5s)
         ↓
7. Admin completes campaign
   → +5 ignore points awarded to users who never interacted
```

---

## Employee Onboarding Flow

```
1. Admin uploads employees via CSV / Excel
   Required columns: name, email, department
   → Employees created with default password: FinShield@2024
         ↓
2. Admin shares the Organization Code with employees
         ↓
3. Employee visits /register
   → Chooses "Join an Organization"
   → Enters Organization Code + their email
   → Sets a new password
         ↓
4. Employee logs in and sees their personal dashboard
```

**CSV format example:**

```csv
name,email,department
Alice Johnson,alice@company.com,Finance
Bob Smith,bob@company.com,IT
Carol White,carol@company.com,HR
```

---

## Security & Privacy Notes

- **No credentials stored:** The `POST /api/track/submit` endpoint intentionally reads nothing from submitted form data. It only records that a submission occurred.
- **Password hashing:** All passwords are hashed with bcrypt (12 salt rounds) before storage.
- **JWT expiry:** Tokens expire after 24 hours. Invalid tokens redirect to `/login`.
- **Data isolation:** Every database query is scoped to `organization_id`, so organizations cannot access each other's data.
- **Points floor:** User points can never drop below 0.
- **Change defaults in production:**
  - Replace `JWT_SECRET` with a strong random secret
  - Use environment-specific SMTP credentials
  - Change the default employee password from `FinShield@2024`

---

## Screenshots

| Page | Description |
|---|---|
| Dashboard | Role-split view — admin sees org analytics, employees see personal score |
| Campaigns | List of campaigns with status badges and quick-action buttons |
| Campaign Detail | Live per-user interaction table with 5s auto-refresh |
| Leaderboard | Employee security score rankings with 30s live refresh |
| Phishing Drill | 4-step simulation: warn → form → result / reported |
| Template Editor | Custom HTML template editor + AI generator |
| Analytics | Bar charts, pie charts, department risk breakdown, PDF export |

---

## License

This project is for educational and authorized internal use only. Do not use FinShield to send phishing emails to individuals without their organization's consent.
