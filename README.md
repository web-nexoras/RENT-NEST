# Rent Nest

A full-stack property and rental management platform with separate tenant and administrator experiences. The project includes a React frontend and an Express/MongoDB API.

## Project structure

```text
Rent Nest/
├── Rent_nest_Auth/       # Express 5 API, MongoDB, authentication and business logic
├── Rent_nest_Frontend/   # React 18 + Vite web application
└── README.md
```

## Features

- **Tenant and admin portals** for dashboards, tenant records, units, payments, reports, maintenance requests, and building notices.
- **Tenant onboarding** through email OTP signup and admin approval, or direct tenant creation by an administrator.
- **Authentication** with email or phone sign-in, password reset, HTTP-only cookie sessions, and automatic access-token refresh.
- **Unit management** with a 48-unit building blueprint across floors A–L. The API seeds these units automatically when MongoDB is connected and the units collection is empty.
- **Rent and payments** with unit-based fixed rent, manual payment submission and admin verification, payment receipts, and optional SSLCommerz checkout.
- **Due-date reminders** and overdue tracking, including scheduled email reminders.
- **Maintenance and notices** with tenant submissions, admin workflows, and notice comments.
- **Redis caching** when configured, with an in-memory fallback.
- **Swagger UI** and an OpenAPI JSON specification for exploring the API.

## Tech stack

| Area | Technologies |
| --- | --- |
| Frontend | React 18, Vite, React Router, TanStack Query, Axios, Tailwind CSS |
| Backend | Node.js, Express 5, Mongoose, MongoDB |
| Optional services | Redis, Brevo email, Cloudinary uploads, SSLCommerz payments |

## Prerequisites

- Node.js 18 or later and npm
- A MongoDB connection string
- Optional credentials for email, image uploads, Redis, and SSLCommerz, depending on which features you want to use

## Run locally

Open two terminals from the repository root.

### 1. Configure and start the backend

```powershell
cd Rent_nest_Auth
Copy-Item .env.example .env
```

Edit `Rent_nest_Auth/.env` and set at least:

```dotenv
DB_URL=your_mongodb_connection_string
JWT_SEC=replace_with_a_long_random_secret
PORT=5000
CLIENT_URL=http://localhost:3000
BASE_URL=/api/v1
NODE_ENV=development
```

`DB_URL` is required for database-backed API routes. The checked-in `.env.example` does not currently include it, so add it to your local `.env`. The server defaults to port `8000` if `PORT` is omitted; use `PORT=5000` to match the frontend proxy default below.

Install dependencies and start the API:

```powershell
npm install
npm run dev
```

The API listens at `http://localhost:5000`. Once MongoDB connects, it automatically creates the standard 48 units if the units collection is empty.

### 2. Configure and start the frontend

In a second terminal:

```powershell
cd Rent_nest_Frontend
Copy-Item .env.example .env
```

For local development, set these values in `Rent_nest_Frontend/.env`:

```dotenv
VITE_API_URL=/api/v1
VITE_PROXY_TARGET=http://localhost:5000
VITE_DEV_PORT=3000
```

The frontend's checked-in example points to the hosted API. Set `VITE_API_URL=/api/v1` to use the local backend through Vite's same-origin proxy.

Then install dependencies and run Vite:

```powershell
npm install
npm run dev
```

Open [https://rent-nest-seven-ecru.vercel.app/signin].

## Useful URLs

| Purpose | URL |
| --- | --- |
| Live Backend | https://rent-nest-auth.onrender.com |
| Live API Health | https://rent-nest-auth.onrender.com/health |
| Live Swagger UI | https://rent-nest-auth.onrender.com/api-docs |

The health endpoint returns HTTP 503 until MongoDB is connected.

## Environment configuration

### Backend: `Rent_nest_Auth/.env`

Start from `.env.example`. In addition to the required `DB_URL`, configure the values that match your setup:

| Variable | Purpose |
| --- | --- |
| `PORT` | API port; the server defaults to `8000` |
| `BASE_URL` | API route prefix; defaults to `/api/v1` |
| `CLIENT_URL` | Frontend origin used for CORS and password-reset links |
| `JWT_SEC` | Secret used to sign authentication tokens |
| `REDIS_URL` | Optional Redis connection; memory cache is used if Redis is unavailable |
| `BREVO_API_KEY`, `MAIL_FROM` | Email delivery for OTPs, receipts, and reminders |
| `CLOUD_NAME`, `API_KEY`, `API_SECRET` | Cloudinary image uploads |
| `SSL_STORE_ID`, `SSL_STORE_PASSWORD`, `SSL_IS_LIVE` | SSLCommerz checkout |
| `SSL_SUCCESS_URL`, `SSL_FAIL_URL`, `SSL_CANCEL_URL` | SSLCommerz callback URLs |
| `RENT_REMINDER_HOUR` | Hour (server local time) when the daily reminder job runs; defaults to `9` |
| `NODE_ENV` | Set to `production` for production behavior |

Some integrations need valid credentials to work. Keep secrets in local environment files and never commit them.

### Frontend: `Rent_nest_Frontend/.env`

| Variable | Purpose |
| --- | --- |
| `VITE_API_URL` | API base URL, including the base path (for example, `/api/v1`) |
| `VITE_PROXY_TARGET` | Backend origin used by the Vite dev proxy |
| `VITE_DEV_PORT` | Vite dev and preview port; defaults to `3000` |

For cookie-based local authentication, use the Vite proxy so the browser and frontend share an origin.

## Available scripts

Run each command from its corresponding project directory.

### Backend

| Command | Description |
| --- | --- |
| `npm run dev` | Start the API with Node's watch mode |
| `npm start` | Start the API with Node's watch mode |

### Frontend

| Command | Description |
| --- | --- |
| `npm run dev` | Start the Vite development server |
| `npm run build` | Create a production build in `dist/` |
| `npm run preview` | Preview the production build locally |

## Authentication and onboarding

Tenants can create an account, verify their email with an OTP, and sign in while awaiting admin approval. Approval and unit assignment unlock tenant features. Administrators can also create tenant accounts directly.

The frontend expects administrator credentials to be provisioned through the backend's supported administrative process; public tenant signup is not an admin-registration flow.

## API overview

The API base path defaults to `/api/v1`. Swagger UI at `/api-docs` documents the available routes and schemas. Main route groups include:

- `/auth` — signup, OTP verification, sign-in, profile, and password flows
- `/admin` — administrator and tenant dashboards, tenant management, and approvals
- `/unit` — unit listings, assignment, and status
- `/payment` — payment submission, checkout, receipts, verification, reports, and reminders
- `/maintenance` — maintenance requests and updates
- `/notice` — building notices and comments

## Notes

- A fresh MongoDB database is automatically initialized with 48 standard units when the backend starts.
- Rent is determined by the assigned unit on the backend; the client-provided payment amount is not authoritative.
- The frontend uses HTTP-only cookie sessions and proxies `/api` requests in development.
- Configure `CLIENT_URL` to match the frontend origin used by your deployment, and configure payment callback URLs for your SSLCommerz environment.

