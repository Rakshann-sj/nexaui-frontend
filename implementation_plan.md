# NexaUI Backend — Implementation Plan

A full-featured Node.js backend for the NexaUI developer platform (API testing + workspace SaaS).

---

## Tech Stack

| Layer | Technology |
|---|---|
| Runtime | Node.js |
| Framework | Express.js |
| Database | MongoDB + Mongoose |
| Auth | JWT (Access + Refresh tokens) |
| Password | bcryptjs |
| Email | Nodemailer (Gmail / SMTP) |
| Payments | Stripe |
| AI Proxy | OpenAI / Gemini API |
| Rate Limiting | express-rate-limit |
| Security | Helmet, CORS, express-mongo-sanitize |
| Validation | express-validator |
| Environment | dotenv |

---

## Folder Structure

```
nexaui-backend/
├── server.js                  ← Entry point
├── package.json
├── .env.example               ← Environment variable template
│
├── config/
│   ├── db.js                  ← MongoDB connection
│   └── mailer.js              ← Nodemailer setup
│
├── middleware/
│   ├── auth.js                ← JWT verification middleware
│   ├── rateLimiter.js         ← Plan-based rate limiting
│   ├── errorHandler.js        ← Global error handler
│   └── validate.js            ← Request validation helper
│
├── models/
│   ├── User.js                ← User schema (name, email, plan, tokens)
│   ├── Project.js             ← Project schema (name, owner, requests)
│   ├── RequestLog.js          ← API call history (method, url, status, ms)
│   ├── Message.js             ← Contact form submissions
│   └── Subscription.js        ← Stripe subscription data
│
├── routes/
│   ├── auth.js                ← /api/auth/*
│   ├── users.js               ← /api/users/*
│   ├── projects.js            ← /api/projects/*
│   ├── apiProxy.js            ← /api/proxy
│   ├── contact.js             ← /api/contact
│   ├── payments.js            ← /api/payments/*
│   └── ai.js                  ← /api/ai/*
│
└── controllers/
    ├── authController.js
    ├── userController.js
    ├── projectController.js
    ├── apiProxyController.js
    ├── contactController.js
    ├── paymentController.js
    └── aiController.js
```

---

## Proposed Changes

### Module 1 — Server Entry Point

#### [NEW] server.js
- Express app setup with all middleware (Helmet, CORS, JSON parser)
- Mount all routes
- Global error handler
- MongoDB connection on startup
- Listen on `PORT` from `.env`

---

### Module 2 — Authentication (`/api/auth`)

#### [NEW] routes/auth.js + controllers/authController.js
- `POST /api/auth/register` — Create account, hash password, send welcome email
- `POST /api/auth/login` — Verify credentials, return JWT access + refresh token
- `POST /api/auth/refresh` — Refresh expired access token
- `POST /api/auth/logout` — Invalidate refresh token
- `POST /api/auth/forgot-password` — Send reset link via email
- `POST /api/auth/reset-password/:token` — Set new password

#### [NEW] models/User.js
- Fields: `name`, `email`, `password` (hashed), `plan` (starter/pro/enterprise), `refreshToken`, `resetToken`, `resetTokenExpiry`, `createdAt`

---

### Module 3 — User Profile (`/api/users`)

#### [NEW] routes/users.js + controllers/userController.js
- `GET /api/users/me` — Get logged-in user profile
- `PUT /api/users/me` — Update name, email
- `PUT /api/users/me/password` — Change password
- `DELETE /api/users/me` — Delete account

---

### Module 4 — Projects (`/api/projects`)

#### [NEW] routes/projects.js + controllers/projectController.js
- `GET /api/projects` — List all user projects
- `POST /api/projects` — Create new project (limit: 3 for free, unlimited for pro)
- `GET /api/projects/:id` — Get single project + its request history
- `PUT /api/projects/:id` — Rename project
- `DELETE /api/projects/:id` — Delete project

#### [NEW] models/Project.js
- Fields: `name`, `owner` (User ref), `description`, `createdAt`

---

### Module 5 — API Proxy Tester (`/api/proxy`)

#### [NEW] routes/apiProxy.js + controllers/apiProxyController.js
- `POST /api/proxy` — Forward request to any external URL
  - Accepts: `method`, `url`, `headers`, `body`
  - Returns: status code, response body, response time (ms)
  - Saves log to `RequestLog` collection
  - Rate limited by plan (5/min for free, unlimited for pro)

#### [NEW] models/RequestLog.js
- Fields: `user`, `project`, `method`, `url`, `statusCode`, `responseTime`, `responseBody`, `createdAt`

---

### Module 6 — Contact Form (`/api/contact`)

#### [NEW] routes/contact.js + controllers/contactController.js
- `POST /api/contact` — Validate & save message, send email notification
- `GET /api/contact` — (Admin only) list all messages

#### [NEW] models/Message.js
- Fields: `name`, `email`, `subject`, `message`, `createdAt`

---

### Module 7 — Payments (`/api/payments`)

#### [NEW] routes/payments.js + controllers/paymentController.js
- `POST /api/payments/create-checkout` — Create Stripe checkout session
- `POST /api/payments/webhook` — Handle Stripe events (payment success, cancel, refund)
- `GET /api/payments/subscription` — Get current subscription status
- `POST /api/payments/cancel` — Cancel subscription

#### [NEW] models/Subscription.js
- Fields: `user`, `stripeCustomerId`, `stripeSubscriptionId`, `plan`, `status`, `currentPeriodEnd`

---

### Module 8 — AI Features (`/api/ai`)

#### [NEW] routes/ai.js + controllers/aiController.js
- `POST /api/ai/describe` — Send API response to AI, get plain-English description
- `POST /api/ai/generate-body` — Generate JSON request body from a description
- `POST /api/ai/detect-error` — Analyze error response and suggest fix
- Pro/Enterprise plan only (middleware guard)

---

### Module 9 — Config & Middleware

#### [NEW] config/db.js
- Mongoose connection with retry logic

#### [NEW] config/mailer.js
- Nodemailer transporter (Gmail SMTP or custom)

#### [NEW] middleware/auth.js
- Verify JWT, attach `req.user`

#### [NEW] middleware/rateLimiter.js
- Free plan: 5 API proxy requests/minute
- Pro/Enterprise: no limit

#### [NEW] middleware/errorHandler.js
- Catch all unhandled errors, return consistent JSON error format

#### [NEW] .env.example
```
PORT=5000
MONGO_URI=mongodb://localhost:27017/nexaui
JWT_SECRET=your_jwt_secret
JWT_REFRESH_SECRET=your_refresh_secret
EMAIL_USER=your@gmail.com
EMAIL_PASS=your_app_password
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
OPENAI_API_KEY=sk-...
CLIENT_URL=http://localhost:3000
```

---

## Verification Plan

### Automated Tests (via Postman / Thunder Client)
- `POST /api/auth/register` → returns JWT
- `POST /api/auth/login` → returns tokens
- `GET /api/users/me` (with token) → returns profile
- `POST /api/projects` → creates project
- `POST /api/proxy` → forwards request, returns response + time
- `POST /api/contact` → saves message, triggers email
- `POST /api/payments/create-checkout` → returns Stripe URL

### Manual Verification
- Check MongoDB collections after each operation
- Verify rate limiting kicks in after 5 requests/min (free plan)
- Confirm email arrives after contact form submit
- Confirm plan upgrade reflected on user profile after Stripe webhook

---

> [!IMPORTANT]
> The backend will be written as a single-stack Node.js + Express app.
> MongoDB Atlas (cloud) or local MongoDB can be used.
> All secrets go in `.env` — never hardcoded.

