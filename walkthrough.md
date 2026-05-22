# NexaUI Backend — Walkthrough

## ✅ What Was Built

A complete, production-ready **Node.js + Express** backend for the NexaUI developer platform.

---

## 📁 Final File Structure

```
nexaui-backend/
├── server.js                        ← Express app entry point
├── package.json                     ← Dependencies
├── .env.example                     ← Environment variable template
│
├── config/
│   ├── db.js                        ← MongoDB connection
│   └── mailer.js                    ← Nodemailer (Gmail SMTP)
│
├── middleware/
│   ├── auth.js                      ← JWT protect + requirePlan()
│   ├── rateLimiter.js               ← General, auth & proxy rate limits
│   ├── errorHandler.js              ← Global error handler
│   └── validate.js                  ← express-validator helper
│
├── models/
│   ├── User.js                      ← Users (plan, password hash)
│   ├── Project.js                   ← Workspace projects
│   ├── RequestLog.js                ← API test history
│   ├── Message.js                   ← Contact form messages
│   └── Subscription.js             ← Stripe subscription data
│
├── routes/
│   ├── auth.js                      ← /api/auth/*
│   ├── users.js                     ← /api/users/*
│   ├── projects.js                  ← /api/projects/*
│   ├── apiProxy.js                  ← /api/proxy/*
│   ├── contact.js                   ← /api/contact
│   ├── payments.js                  ← /api/payments/*
│   └── ai.js                        ← /api/ai/*
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

## 🌐 API Endpoint Reference

### Auth — `/api/auth`
| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/register` | ❌ | Create account |
| POST | `/login` | ❌ | Login, get tokens |
| POST | `/logout` | ❌ | Invalidate refresh token |
| POST | `/refresh` | ❌ | Get new access token |
| POST | `/forgot-password` | ❌ | Send reset email |
| POST | `/reset-password/:token` | ❌ | Reset password |

### Users — `/api/users`
| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/me` | ✅ | Get my profile |
| PUT | `/me` | ✅ | Update name/email |
| PUT | `/me/password` | ✅ | Change password |
| DELETE | `/me` | ✅ | Deactivate account |

### Projects — `/api/projects`
| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/` | ✅ | List my projects |
| POST | `/` | ✅ | Create project (limit: 3 free) |
| GET | `/:id` | ✅ | Get project + history |
| PUT | `/:id` | ✅ | Update project |
| DELETE | `/:id` | ✅ | Delete project + logs |

### API Proxy — `/api/proxy`
| Method | Endpoint | Auth | Rate Limit | Description |
|---|---|---|---|---|
| POST | `/` | ✅ | 5/min (free) | Forward API request |
| GET | `/history` | ✅ | — | Paginated request history |

### Contact — `/api/contact`
| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/` | ❌ | Submit contact form |
| GET | `/` | ✅ | List all messages (admin) |

### Payments — `/api/payments`
| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/create-checkout` | ✅ | Create Stripe checkout |
| POST | `/webhook` | Stripe-signed | Handle Stripe events |
| GET | `/subscription` | ✅ | Get subscription status |
| POST | `/cancel` | ✅ | Cancel at period end |

### AI — `/api/ai` (Pro/Enterprise only)
| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/describe` | ✅ Pro+ | Describe API response |
| POST | `/generate-body` | ✅ Pro+ | Generate request body |
| POST | `/detect-error` | ✅ Pro+ | Analyze & explain errors |

---

## 🚀 How to Run

### 1. Set up environment
```bash
cd nexaui-backend
cp .env.example .env
# Fill in your values in .env
```

### 2. Install dependencies
```bash
npm install
```

### 3. Start the server
```bash
# Development (auto-restart)
npm run dev

# Production
npm start
```

Server runs at: `http://localhost:5000`
Health check: `http://localhost:5000/api/health`

---

## 🔗 Connect to Frontend

In the frontend `index.html`, update the API base URL:
```js
const API = 'http://localhost:5000/api';

// Example: run API test via proxy
fetch(`${API}/proxy`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${accessToken}`
  },
  body: JSON.stringify({ url, method, headers, body })
})
```

---

## 🔒 Security Highlights

- Passwords hashed with **bcrypt (12 rounds)**
- **JWT** access tokens expire in 15 minutes
- **Refresh tokens** rotated on each login
- **Helmet** sets secure HTTP headers
- **mongo-sanitize** prevents NoSQL injection
- **Rate limiting** on all auth + proxy routes
- Stripe webhooks verified with **signature secret**
- Reset tokens are **SHA-256 hashed** before DB storage

---

> [!IMPORTANT]
> Before going live: set `NODE_ENV=production`, use **MongoDB Atlas**, configure **Stripe webhook** endpoint, and deploy behind **HTTPS**.
