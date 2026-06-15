# Offer Letter App

A full-stack web application for managing employee offer letters, originally built for **AMC Engineering College** and currently adapted for use at **Ramaiah Institute of Business Studies (RIBS)**. It allows HR teams and admins to create, manage, send, and track offer letters for candidates end-to-end.

> **Note:** This project was initially developed for AMC Engineering College. It has since been repurposed and configured for Ramaiah Institute of Business Studies (RIBS). The codebase is institution-agnostic — the company name and logo are fully configurable via environment variables (`COMPANY_NAME`, `COMPANY_LOGO_URL`), making it easy to adapt for any organization.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Features](#features)
- [Project Structure](#project-structure)
- [Backend](#backend)
  - [Models](#models)
  - [API Routes](#api-routes)
  - [Middleware](#middleware)
  - [Services](#services)
- [Frontend](#frontend)
  - [Pages](#pages)
  - [Auth](#auth)
- [Offer Status Lifecycle](#offer-status-lifecycle)
- [Infrastructure & Config](#infrastructure--config)
- [Environment Variables](#environment-variables)
- [Getting Started](#getting-started)
- [Deployment](#deployment)
- [Security Notes](#security-notes)

---

## Overview

The Offer Letter App streamlines the entire offer letter workflow for HR teams:

- Create and version offer letter templates with dynamic placeholders
- Manage candidate records (individually or via CSV bulk import)
- Generate offer letters by merging templates with candidate data
- Export offer letters as professionally styled PDFs (with DRAFT watermark)
- Send offer letters via email immediately or schedule for later
- Track offer status through a strict lifecycle state machine
- View analytics on offer activity by status, department, and over time
- Full audit trail of all write actions across the system

---

## Architecture

```
offerletter_app/
├── backend/      # Node.js + Express REST API
└── frontend/     # React + Vite SPA
```

- Monorepo with `backend/` and `frontend/` as separate deployable units
- Backend deployed on **Railway**
- Frontend deployed on **Vercel**
- Database: **Neon** (serverless PostgreSQL)
- Caching: **Redis**
- File Storage: **AWS S3**
- Email: **Gmail SMTP via Nodemailer**

---

## Tech Stack

### Backend
| Technology | Version | Purpose |
|---|---|---|
| Node.js (ESM) | — | Runtime |
| Express | ^5.1.0 | HTTP framework |
| Sequelize | ^6.37.7 | ORM |
| PostgreSQL (Neon) | — | Primary database |
| Redis | — | Response caching |
| jsonwebtoken | ^9.0.2 | JWT authentication |
| bcryptjs | ^3.0.2 | Password hashing |
| Nodemailer | ^6.10.1 | Email sending |
| Puppeteer-core | ^25.1.0 | Headless PDF generation |
| AWS SDK v3 | ^3.1048.0 | S3 file storage |
| node-cron | ^3.0.3 | Scheduled email jobs |
| Joi | ^17.13.3 | Input validation |
| Helmet | ^8.0.0 | Security headers |
| express-rate-limit | ^7.5.0 | Rate limiting |
| multer | ^2.0.1 | File uploads (CSV) |
| csv-parser | ^3.2.0 | CSV parsing |
| morgan | ^1.10.0 | HTTP request logging |

### Frontend
| Technology | Version | Purpose |
|---|---|---|
| React | ^19.2.6 | UI framework |
| Vite | ^8.0.12 | Build tool |
| React Router | ^7.15.1 | Client-side routing |
| Axios | ^1.16.1 | HTTP client |

---

## Features

1. **Role-based access control** — Admin and HR roles with route-level guards
2. **Template management** — Create/edit HTML templates with `{{placeholder}}` variables, full version history
3. **Candidate management** — Add individually or bulk import via CSV upload
4. **Offer lifecycle state machine** — Strict status transitions with version history per offer
5. **PDF generation** — Styled A4 PDF with company logo, employment details, terms, signature blocks, and DRAFT watermark
6. **Email delivery** — Send offer emails immediately or schedule for a future time; emails include the PDF as attachment
7. **AWS S3 storage** — PDFs uploaded and retrievable via presigned URLs
8. **Analytics dashboard** — Offers by status, offers by department, acceptance rate, over-time trends
9. **Audit logging** — Every POST/PUT/PATCH/DELETE action is logged with user, route, IP, and resource ID
10. **Security** — Helmet, CORS, rate limiting, input sanitization, JWT auth, bcrypt hashing

---

## Project Structure

```
offerletter_app/
├── backend/
│   ├── config/
│   │   ├── db.js               # Sequelize + Neon DB connection
│   │   └── redis.js            # Redis client
│   ├── controllers/
│   │   ├── analyticsController.js
│   │   ├── auditController.js
│   │   ├── authController.js
│   │   ├── candidateController.js
│   │   ├── emailController.js
│   │   ├── offerController.js
│   │   ├── pdfController.js
│   │   ├── templateController.js
│   │   └── userController.js
│   ├── middleware/
│   │   ├── auth.js             # JWT verification
│   │   ├── auditLogger.js      # Write action logger
│   │   ├── cache.js            # Redis caching + invalidation
│   │   ├── errorHandler.js     # Global async error handler
│   │   ├── rateLimiter.js      # express-rate-limit
│   │   ├── roleGuard.js        # Role-based access (admin/hr)
│   │   ├── sanitize.js         # Strip $ and . keys
│   │   └── validate.js         # Joi schema validation
│   ├── models/
│   │   ├── AuditLog.js
│   │   ├── Candidate.js
│   │   ├── Offer.js
│   │   ├── OfferVersion.js
│   │   ├── ScheduledEmail.js
│   │   ├── Template.js
│   │   ├── TemplateVersion.js
│   │   ├── User.js
│   │   └── index.js            # Sequelize sync + exports
│   ├── routes/
│   │   ├── analyticsRoutes.js
│   │   ├── auditRoutes.js
│   │   ├── authRoutes.js
│   │   ├── candidateRoutes.js
│   │   ├── emailRoutes.js
│   │   ├── offerRoutes.js
│   │   ├── pdfRoutes.js
│   │   ├── templateRoutes.js
│   │   └── userRoutes.js
│   ├── services/
│   │   ├── cronService.js      # node-cron scheduled email processor
│   │   ├── emailService.js     # Nodemailer email sender
│   │   ├── pdfService.js       # Puppeteer PDF generator
│   │   └── storageService.js   # AWS S3 upload + presigned URLs
│   ├── utils/
│   │   ├── authSchemas.js
│   │   ├── candidateSchemas.js
│   │   ├── emailSchemas.js
│   │   ├── fillPlaceholders.js # Template placeholder replacement
│   │   ├── multerConfig.js
│   │   ├── offerSchemas.js     # Status transitions + Joi schemas
│   │   ├── templateSchemas.js
│   │   └── userSchemas.js
│   ├── .env.development
│   ├── .env.production
│   ├── server.js               # App entry point
│   ├── package.json
│   └── railway.toml
└── frontend/
    ├── src/
    │   ├── components/
    │   │   └── Layout.jsx      # App shell with sidebar nav
    │   ├── pages/
    │   │   ├── AuditLogs.jsx
    │   │   ├── Candidates.jsx
    │   │   ├── Dashboard.jsx
    │   │   ├── Login.jsx
    │   │   ├── Offers.jsx
    │   │   ├── Templates.jsx
    │   │   └── Users.jsx
    │   ├── api.js              # Axios instance
    │   ├── App.jsx             # Router + PrivateRoute
    │   ├── AuthContext.jsx     # JWT auth context
    │   └── main.jsx
    ├── .env
    ├── vercel.json
    ├── vite.config.js
    └── package.json
```

---

## Backend

### Models

| Model | Table | Key Fields |
|---|---|---|
| User | users | id (UUID), name, email, password, role (admin/hr) |
| Template | templates | id, name, content (HTML), created_by |
| TemplateVersion | template_versions | id, template_id, version_no, content |
| Candidate | candidates | id, name, email, position, department, doj, salary |
| Offer | offers | id, candidate_id, template_id, status, salary, doj |
| OfferVersion | offer_versions | id, offer_id, version_no, content (filled HTML) |
| ScheduledEmail | scheduled_emails | id, offer_id, scheduled_at, status (pending/sent/failed) |
| AuditLog | audit_logs | id, user_id, user_email, method, route, action, resource_id, ip |

All models use UUID primary keys and are synced via `sequelize.sync({ alter: true })` on startup.

### API Routes

| Method | Route | Description | Auth |
|---|---|---|---|
| POST | /api/auth/register | Register a new user | — |
| POST | /api/auth/login | Login + get JWT | — |
| GET | /api/auth/me | Get current user info | JWT |
| GET | /api/users | List all users | Admin |
| POST | /api/users | Create user | Admin |
| PUT | /api/users/:id | Update user | Admin |
| DELETE | /api/users/:id | Delete user | Admin |
| GET | /api/templates | List templates | JWT |
| POST | /api/templates | Create template | JWT |
| PUT | /api/templates/:id | Update template | JWT |
| DELETE | /api/templates/:id | Delete template | JWT |
| GET | /api/templates/:id/versions | Get version history | JWT |
| GET | /api/candidates | List candidates | JWT |
| POST | /api/candidates | Create candidate | JWT |
| POST | /api/candidates/import | Bulk import via CSV | JWT |
| PUT | /api/candidates/:id | Update candidate | JWT |
| DELETE | /api/candidates/:id | Delete candidate | JWT |
| GET | /api/offers | List offers (filter by status) | JWT |
| POST | /api/offers | Create offer | JWT |
| GET | /api/offers/:id | Get offer by ID | JWT |
| PATCH | /api/offers/:id/status | Update offer status | JWT |
| GET | /api/offers/:id/versions | Get offer version history | JWT |
| POST | /api/offers/:id/send-email | Send offer email now | JWT |
| POST | /api/offers/:id/schedule-email | Schedule offer email | JWT |
| GET | /api/offers/:id/pdf | Generate + download PDF | JWT |
| GET | /api/analytics/overview | Summary stats | JWT |
| GET | /api/analytics/by-status | Offers grouped by status | JWT |
| GET | /api/analytics/by-department | Offers grouped by department | JWT |
| GET | /api/analytics/over-time | Offers over time | JWT |
| GET | /api/logs | List audit logs | Admin |

### Middleware

The middleware stack is applied globally in this order:

1. `helmet` — Sets secure HTTP headers
2. `cors` — Restricts to `CLIENT_URL` origin
3. `morgan` — Logs HTTP requests (`dev` in development, `combined` in production)
4. `express.json({ limit: '10kb' })` — Parses JSON body, guards against large payload attacks
5. `sanitizeInput` — Strips keys containing `$` or `.` to prevent NoSQL injection
6. `rateLimiter` — Global rate limiting via express-rate-limit
7. `auditLogger` — Logs all write actions (POST/PUT/PATCH/DELETE) to the audit_logs table
8. `auth` — Verifies JWT Bearer token on protected routes
9. `roleGuard` — Enforces role-based access (admin vs hr)
10. `validate` — Validates request body against Joi schemas per route
11. `cache` — Redis caching for GET routes with automatic invalidation on mutations
12. `errorHandler` — Catches all async errors and returns structured JSON responses

### Services

**emailService.js**
- Uses Nodemailer with Gmail SMTP
- Sends HTML email with company branding (header, body, footer)
- Attaches the PDF buffer as `offer-letter-<name>.pdf`

**pdfService.js**
- Uses Puppeteer-core with headless Chromium
- Generates a styled A4 PDF including:
  - Company logo and name in header
  - Offer badge with reference number and date
  - Employment details table (name, email, position, department, DOJ, CTC)
  - Terms and conditions (probation, background check, NDA, notice period)
  - Acceptance deadline notice
  - Dual signature blocks (candidate + authorized signatory)
  - Branded footer
  - DRAFT watermark overlay when status is `draft`

**storageService.js**
- AWS SDK v3 S3 client
- `uploadToS3(buffer, key)` — uploads a PDF buffer to S3
- `getSignedDownloadUrl(key)` — generates a presigned URL (default 1-hour expiry)

**cronService.js**
- node-cron job scheduled to run every minute (`* * * * *`)
- Queries ScheduledEmail records where `status = pending` and `scheduled_at <= now`
- For each due email: generates PDF, sends email, marks as `sent`; marks as `failed` on error
- Also updates the linked Offer status to `sent`

---

## Frontend

### Pages

| Page | Route | Access | Description |
|---|---|---|---|
| Login | /login | Public | Email + password login form |
| Dashboard | / | All authenticated | Stats cards + bar charts for status and department |
| Templates | /templates | All authenticated | Create, edit, delete offer templates |
| Candidates | /candidates | All authenticated | Manage candidates, bulk CSV import |
| Offers | /offers | All authenticated | Create offers, update status, send/schedule email, download PDF |
| Users | /users | Admin only | Manage HR/admin user accounts |
| AuditLogs | /logs | Admin only | View all system audit log entries |

### Auth

- JWT stored in `localStorage`
- Auth state managed via React Context (`AuthContext`)
- `PrivateRoute` component redirects unauthenticated users to `/login`
- Admin-only routes redirect non-admin users to `/`
- Axios instance in `api.js` automatically attaches `Authorization: Bearer <token>` header

---

## Offer Status Lifecycle

Offers follow a strict state machine — only valid transitions are allowed:

```
draft → generated → sent → viewed → accepted → expired
                                  ↘ rejected
```

| Current Status | Allowed Next Status |
|---|---|
| draft | generated |
| generated | sent |
| sent | viewed |
| viewed | accepted, rejected |
| accepted | expired |
| rejected | — |
| expired | — |

Invalid transitions return a `400` error with the list of allowed next statuses.

---

## Infrastructure & Config

| Concern | Solution |
|---|---|
| Database | Neon PostgreSQL (serverless, pooled connection) |
| ORM | Sequelize with `alter: true` auto-sync |
| Caching | Redis (localhost in dev, configurable via REDIS_URL) |
| File Storage | AWS S3 (SDK v3, multipart upload) |
| Email | Gmail SMTP via Nodemailer |
| PDF Generation | Puppeteer-core (headless Chromium, no sandbox) |
| Scheduled Jobs | node-cron (every minute) |
| Backend Hosting | Railway (configured via railway.toml) |
| Frontend Hosting | Vercel (configured via vercel.json) |
| Auth | JWT (7-day expiry) + bcryptjs (salt rounds: 10) |
| Validation | Joi schemas per route |
| Input Security | Helmet + CORS + sanitize + rate limiter |

---

## Environment Variables

### Backend (`backend/.env.development` / `backend/.env.production`)

| Variable | Description |
|---|---|
| `PORT` | Server port (default: 5000) |
| `DATABASE_URL` | Neon PostgreSQL connection string |
| `JWT_SECRET` | Secret key for signing JWTs |
| `JWT_EXPIRES_IN` | JWT expiry duration (e.g. `7d`) |
| `REDIS_URL` | Redis connection URL |
| `NODE_ENV` | `development` or `production` |
| `CLIENT_URL` | Frontend origin for CORS |
| `SMTP_HOST` | SMTP server host |
| `SMTP_PORT` | SMTP server port |
| `SMTP_USER` | SMTP username (email address) |
| `SMTP_PASS` | SMTP password / app password |
| `EMAIL_FROM` | From address for sent emails |
| `COMPANY_NAME` | Company name shown in PDFs and emails |
| `COMPANY_LOGO_URL` | URL of company logo for PDF header |
| `AWS_REGION` | AWS region for S3 |
| `AWS_ACCESS_KEY_ID` | AWS access key |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key |
| `AWS_S3_BUCKET` | S3 bucket name |
| `PUPPETEER_EXECUTABLE_PATH` | (Optional) Custom path to Chromium binary |

### Frontend (`frontend/.env`)

| Variable | Description |
|---|---|
| `VITE_API_URL` | Backend API base URL |

---

## Getting Started

### Prerequisites

- Node.js 18+
- PostgreSQL (or a Neon account)
- Redis (local or cloud)
- A Gmail account with App Password enabled (for SMTP)
- AWS account with S3 bucket (optional, for PDF storage)

### Backend

```bash
cd backend
npm install
# Create and fill in .env.development
npm run dev
```

The server starts on `http://localhost:5000`. All database tables are auto-created/altered on startup.

### Frontend

```bash
cd frontend
npm install
# Set VITE_API_URL in .env
npm run dev
```

The frontend starts on `http://localhost:5173`.

---

## Deployment

### Backend → Railway

1. Connect your GitHub repo to Railway
2. Set the root directory to `backend/`
3. Add all environment variables from `.env.production` in Railway's dashboard
4. Railway uses `npm run start` (`node --env-file=.env.production server.js`)

### Frontend → Vercel

1. Connect your GitHub repo to Vercel
2. Set the root directory to `frontend/`
3. Add `VITE_API_URL` pointing to your Railway backend URL
4. Vercel auto-deploys on push to `main`

---

## Security Notes

- Use a long, random string (32+ chars) for `JWT_SECRET` in production — never a plain readable word
- Never commit `.env` files containing real credentials to version control — add them to `.gitignore`
- SMTP credentials in `.env.development` should use Gmail App Passwords, not your real Gmail password
- AWS IAM user for S3 should have least-privilege permissions (only `s3:PutObject`, `s3:GetObject` on the specific bucket)
- The `sanitize` middleware strips `$` and `.` keys but does not replace a proper parameterized query strategy — Sequelize ORM already handles SQL injection via parameterized queries

---

## Local Setup Guide (For Beginners)

This guide assumes you have nothing installed — no VS Code, no Node.js, nothing.

### What You Need to Install

| # | Tool | Why |
|---|---|---|
| 1 | VS Code | Code editor to open and edit files |
| 2 | Node.js | Runs the backend and frontend |
| 3 | Git | To download the project from GitHub |
| 4 | Redis | Caching service used by the backend |
| 5 | npm packages | All code libraries (installed automatically via `npm install`) |

> No need to install PostgreSQL — the database is hosted online on **Neon** (already configured). AWS S3 is optional for basic local use.

---

### Step 1 — Install VS Code

1. Go to: **https://code.visualstudio.com**
2. Click the big blue **Download for Windows** button
3. Run the downloaded `.exe` file and click **Next → Next → Install → Finish**
4. Open VS Code from your Desktop or Start Menu

---

### Step 2 — Install Node.js

Node.js runs this project. It also installs `npm` (the package manager) automatically.

1. Go to: **https://nodejs.org**
2. Click the **LTS** version (says "Recommended for most users")
3. Run the downloaded `.msi` installer → click **Next → Next → Install → Finish**
4. Verify — open **Command Prompt** (search "cmd" in Windows search) and type:
   ```
   node --version
   ```
   Expected output: `v22.x.x`
   ```
   npm --version
   ```
   Expected output: `10.x.x`

---

### Step 3 — Install Git

Git is used to download the project code from GitHub.

1. Go to: **https://git-scm.com/download/win**
2. The download starts automatically — run the `.exe`
3. Click **Next** on every screen (default options are fine) → **Install → Finish**
4. Verify — open **Command Prompt** and type:
   ```
   git --version
   ```
   Expected output: `git version 2.x.x`

---

### Step 4 — Install Redis (Windows)

Redis is a background service the backend uses for caching.

1. Go to: **https://github.com/microsoftarchive/redis/releases**
2. Download the file named **Redis-x64-3.0.504.msi** (or the latest `.msi`)
3. Run the installer → click **Next → Next → Install → Finish**
4. Redis will now run automatically as a Windows service in the background
5. Verify — open **Command Prompt** and type:
   ```
   redis-cli ping
   ```
   Expected output: `PONG`

> If `redis-cli` is not found, search for "Redis" in your Start Menu and open the Redis CLI from there.

---

### Step 5 — Download the Project

1. Open **Command Prompt**
2. Navigate to where you want to save the project (e.g. Desktop):
   ```
   cd Desktop
   ```
3. Clone the project:
   ```
   git clone https://github.com/Shashank-Git1804/offerletter_app.git
   ```
4. Enter the project folder:
   ```
   cd offerletter_app
   ```

---

### Step 6 — Open in VS Code

In **Command Prompt** (still inside the `offerletter_app` folder), type:
```
code .
```

> If `code .` doesn't work, open VS Code manually → **File → Open Folder** → select the `offerletter_app` folder.

---

### Step 7 — Install Backend Dependencies

1. In VS Code, open the Terminal (top menu → **Terminal → New Terminal**)
2. Navigate to the backend folder:
   ```
   cd backend
   ```
3. Install all packages:
   ```
   npm install
   ```
   Wait for it to finish. A `node_modules` folder will appear.

---

### Step 8 — Install Frontend Dependencies

1. In the same terminal, go to the frontend folder:
   ```
   cd ../frontend
   ```
2. Install all packages:
   ```
   npm install
   ```

---

### Step 9 — Set Up the Frontend Environment File

The backend already has a working `.env.development` file. For the frontend:

1. Inside the `frontend/` folder, check if a `.env` file exists
2. If it doesn't, in VS Code right-click the `frontend` folder → **New File** → name it `.env`
3. Add this single line:
   ```
   VITE_API_URL=http://localhost:5000
   ```
4. Save the file

---

### Step 10 — Run the Project

You need **two terminals** running at the same time.

**Terminal 1 — Start the Backend:**
```bash
cd backend
npm run dev
```
Expected output:
```
Server running on port 5000 [development]
All models synced to Neon DB
Cron jobs started
```

**Terminal 2 — Start the Frontend** (click the `+` icon in the terminal panel to open a new one):
```bash
cd frontend
npm run dev
```
Expected output:
```
  VITE v8.x.x  ready in ...ms
  ➜  Local:   http://localhost:5173/
```

---

### Step 11 — Open the App

Open your browser (Chrome, Edge, etc.) and go to:

**http://localhost:5173**

You will see the Login page. Register a new account to get started.

---

### Setup Checklist

- [ ] VS Code installed
- [ ] Node.js (LTS) installed — `node --version` works
- [ ] Git installed — `git --version` works
- [ ] Redis installed and running — `redis-cli ping` returns `PONG`
- [ ] Project cloned from GitHub
- [ ] `npm install` done inside `backend/`
- [ ] `npm install` done inside `frontend/`
- [ ] `frontend/.env` created with `VITE_API_URL=http://localhost:5000`
- [ ] Backend running on port 5000
- [ ] Frontend running on port 5173
- [ ] App opens at `http://localhost:5173`

> If you get any error, copy the exact error message and search it. Most Node.js and Redis install errors on Windows are fixed by restarting Command Prompt after installation, or running it as Administrator.

---

## Origin

This project was originally developed as an internal tool for **AMC Engineering College** to manage their HR offer letter workflow. It has since been adapted and redeployed for **Ramaiah Institute of Business Studies (RIBS)**. The application is fully institution-agnostic — switching it to any other organization only requires updating `COMPANY_NAME` and `COMPANY_LOGO_URL` in the environment variables.

---

## License

ISC
