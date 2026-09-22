# 🚦 Rate Limiting & Throttling in Node.js + Express

> **Day 6 — Real-Time Data Processing Server**
>
> A practical, beginner-friendly guide to implementing **Rate Limiting** and **Custom Throttling** in an Express application.

---

## 🎯 What You Will Build

By the end of this guide, your Node.js server will support:

- 🔐 Login rate limiting
- 🚫 `429 Too Many Requests`
- ⏱️ Request limits within a time window
- 🧩 Reusable Express middleware
- 🛠️ A custom throttle using JavaScript `Map`
- 🏗️ Production considerations for distributed systems

---

# 1. 🧠 Why Do We Need Rate Limiting?

Imagine a login API:

```text
POST /auth/login
```

Without protection, a client could send:

```text
Request 1
Request 2
Request 3
...
Request 10000
```

Very quickly.

This can cause:

- 🔴 Too many requests
- 🔴 Server overload
- 🔴 Brute-force login attempts
- 🔴 API abuse
- 🔴 Unnecessary database queries

Rate limiting puts a controlled limit on how many requests a client can make.

### Simple analogy

Think of a movie theatre.

The theatre can only allow a certain number of people inside during a period.

Rate limiting works similarly:

```text
Client
  │
  │  Request
  ▼
┌──────────────────┐
│  Rate Limiter    │
│                  │
│  Allow?          │
└───────┬──────────┘
        │
   ┌────┴────┐
   │         │
  YES        NO
   │         │
   ▼         ▼
Route      429
```

---

# 2. ⚖️ Rate Limiting vs Throttling

These terms are related but are often used slightly differently.

| Concept | Meaning |
|---|---|
| **Rate Limiting** | Limits how many requests a client can make during a defined window |
| **Throttling** | Controls request frequency or slows/rejects requests when traffic is too high |

Example:

```text
5 requests / 10 seconds
```

A client can make five requests.

The sixth request is rejected until the window resets.

---

# 3. 📦 Step 1 — Install express-rate-limit

From your project folder:

```powershell
npm install express-rate-limit
```

Check `package.json`:

```json
"dependencies": {
  "express": "...",
  "express-rate-limit": "..."
}
```

---

# 4. 📁 Step 2 — Create the Rate Limiter Folder

Project structure:

```text
node-day-6/
│
├── app.js
│
├── routes/
│   └── auth.js
│
├── security/
│   ├── rateLimiter.js
│   └── customThrottle.js
│
├── streams/
├── processes/
├── workers/
├── chat/
│
└── output/
```

We keep security middleware inside:

```text
security/
```

This makes the project easier to maintain.

---

# 5. 🛡️ Step 3 — Create `rateLimiter.js`

Create:

```text
security/rateLimiter.js
```

Add:

```js
import rateLimit from "express-rate-limit";

const loginRateLimiter = rateLimit({
  windowMs: 60 * 1000,

  max: 5,

  message: {
    success: false,
    message: "Too many login attempts. Try again later.",
  },

  standardHeaders: true,
  legacyHeaders: false,
});

export default loginRateLimiter;
```

---

# 6. 🔍 Step 4 — Understand the Configuration

### `windowMs`

```js
windowMs: 60 * 1000
```

Means:

```text
60 seconds
```

because:

```text
60 × 1000 milliseconds
= 60,000 ms
= 60 seconds
```

---

### `max`

```js
max: 5
```

Means:

```text
Maximum 5 requests
```

So:

```text
Request 1 ✅
Request 2 ✅
Request 3 ✅
Request 4 ✅
Request 5 ✅
Request 6 ❌
```

---

### `message`

```js
message: {
  success: false,
  message: "Too many login attempts. Try again later.",
}
```

This is the response sent when the client exceeds the limit.

---

### `standardHeaders`

```js
standardHeaders: true
```

Allows standard rate-limit information to be returned in response headers.

---

### `legacyHeaders`

```js
legacyHeaders: false
```

Disables older `X-RateLimit-*` style headers.

---

# 7. 🔐 Step 5 — Create the Login Route

Create:

```text
routes/auth.js
```

Add:

```js
import express from "express";
import loginRateLimiter from "../security/rateLimiter.js";

const router = express.Router();

router.post(
  "/login",
  loginRateLimiter,
  (req, res) => {
    res.status(200).json({
      success: true,
      message: "Login request received",
    });
  }
);

export default router;
```

Notice this:

```js
loginRateLimiter
```

is placed between:

```text
Request
   ↓
Rate Limiter
   ↓
Login Route
```

So the limiter runs before the login logic.

---

# 8. 🔌 Step 6 — Connect the Route to `app.js`

In `app.js`:

```js
import authRouter from "./routes/auth.js";
```

Then:

```js
app.use("/auth", authRouter);
```

Your route becomes:

```text
POST /auth/login
```

Full flow:

```text
POST /auth/login
       │
       ▼
 loginRateLimiter
       │
       ├── Allowed ──────► Login Controller
       │
       └── Limit reached ► 429 Response
```

---

# 9. 🧪 Step 7 — Test Using Postman

Start your server:

```powershell
npm run dev
```

You should see:

```text
Server is running on port no 3000
```

Open Postman.

### Request

```text
POST http://localhost:3000/auth/login
```

Body:

```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

---

# 10. ✅ Step 8 — Test the Limit

Send the request repeatedly.

For example:

```text
Request 1 → 200 ✅
Request 2 → 200 ✅
Request 3 → 200 ✅
Request 4 → 200 ✅
Request 5 → 200 ✅
Request 6 → 429 ❌
```

The sixth request should return:

```json
{
  "success": false,
  "message": "Too many login attempts. Try again later."
}
```

HTTP status:

```text
429 Too Many Requests
```

---

# 11. 🚨 What Does 429 Mean?

HTTP `429` means:

> The client has sent too many requests in a given amount of time.

Example:

```text
Client
  │
  ├── Request 1 ✅
  ├── Request 2 ✅
  ├── Request 3 ✅
  ├── Request 4 ✅
  ├── Request 5 ✅
  │
  └── Request 6 ❌
          │
          ▼
    HTTP 429
    Too Many Requests
```

---

# 12. 🧩 Step 9 — Create a Custom Throttle

Now we will understand how throttling can be implemented manually.

Create:

```text
security/customThrottle.js
```

Add:

```js
const requests = new Map();

const customThrottle = (req, res, next) => {
  const clientId = req.ip;

  const now = Date.now();

  const windowMs = 10 * 1000;
  const maxRequests = 5;

  if (!requests.has(clientId)) {
    requests.set(clientId, {
      count: 1,
      startTime: now,
    });

    return next();
  }

  const clientData = requests.get(clientId);

  const elapsedTime = now - clientData.startTime;

  if (elapsedTime >= windowMs) {
    clientData.count = 1;
    clientData.startTime = now;

    return next();
  }

  if (clientData.count >= maxRequests) {
    return res.status(429).json({
      success: false,
      message: "Too many requests. Try again later.",
    });
  }

  clientData.count++;

  next();
};

export default customThrottle;
```

---

# 13. 🧠 How the Custom Throttle Works

We use:

```js
const requests = new Map();
```

The `Map` stores information about each client.

Example:

```text
IP Address
    ↓
192.168.1.10
    │
    ├── count: 3
    └── startTime: ...
```

Another client:

```text
192.168.1.20
    │
    ├── count: 2
    └── startTime: ...
```

---

# 14. 🔢 Request Counting

We define:

```js
const windowMs = 10 * 1000;
const maxRequests = 5;
```

Therefore:

```text
Time window = 10 seconds
Maximum requests = 5
```

Flow:

```text
0 sec
 │
 ├── Request 1 ✅
 ├── Request 2 ✅
 ├── Request 3 ✅
 ├── Request 4 ✅
 ├── Request 5 ✅
 │
 └── Request 6 ❌
        429
```

After 10 seconds:

```text
Counter resets
     ↓
Request allowed again
```

---

# 15. 🧪 Step 10 — Test Custom Throttling

You can temporarily apply it to a route:

```js
import customThrottle from "../security/customThrottle.js";

router.get(
  "/test-throttle",
  customThrottle,
  (req, res) => {
    res.json({
      success: true,
      message: "Request allowed",
    });
  }
);
```

Test:

```text
GET http://localhost:3000/auth/test-throttle
```

Send it six times quickly.

Expected:

```text
1 → 200 ✅
2 → 200 ✅
3 → 200 ✅
4 → 200 ✅
5 → 200 ✅
6 → 429 ❌
```

Wait approximately 10 seconds and try again.

```text
Request → 200 ✅
```

---

# 16. 🔄 Complete Request Flow

Your final rate-limiting architecture looks like:

```text
                    CLIENT
                      │
                      ▼
              ┌───────────────┐
              │    Express    │
              └───────┬───────┘
                      │
                      ▼
             ┌─────────────────┐
             │  Rate Limiter   │
             └────────┬────────┘
                      │
              ┌───────┴────────┐
              │                │
            Allowed          Blocked
              │                │
              ▼                ▼
        Login/API Route      HTTP 429
              │
              ▼
          Controller
              │
              ▼
          Database
```

---

# 17. 🏭 Production Consideration

Our custom throttle uses:

```js
new Map()
```

This is useful for **learning and simple single-server applications**.

However, there is an important limitation.

Suppose you run:

```text
Node Server 1
Node Server 2
Node Server 3
```

Each process has its own memory:

```text
Server 1 → Map A
Server 2 → Map B
Server 3 → Map C
```

The counters are not automatically shared.

For distributed production systems, a shared store such as **Redis** is commonly used.

Conceptually:

```text
             Load Balancer
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
     Node 1     Node 2    Node 3
        │         │         │
        └─────────┼─────────┘
                  ▼
                Redis
                  │
           Shared counters
```

This becomes especially important when using the **Cluster module** from Day 6.

---

# 18. 🔐 Where Should Rate Limiting Be Used?

Common examples:

| Endpoint | Typical Reason |
|---|---|
| `/auth/login` | Reduce brute-force attempts |
| `/auth/register` | Reduce automated registrations |
| `/auth/forgot-password` | Protect reset functionality |
| `/api/search` | Prevent excessive API usage |
| `/api/upload` | Protect server resources |
| Public APIs | Prevent abuse |

Different endpoints can use different limits.

For example:

```text
Login
5 requests / minute

Search
100 requests / minute

Public API
1000 requests / hour
```

The exact limits should be chosen based on the application's requirements and expected traffic.

---

# 19. 🆚 Package vs Custom Implementation

| Feature | `express-rate-limit` | Custom `Map` |
|---|---|---|
| Easy to implement | ✅ | ⚠️ |
| Production-oriented middleware | ✅ | ⚠️ |
| Custom logic | Good | Excellent |
| Good for learning | ✅ | ✅ |
| Shared across servers | Requires appropriate store/configuration | ❌ |
| Maintenance | Library handles much of it | You maintain it |

For your Day 6 project:

```text
express-rate-limit
        +
customThrottle
```

is useful because you learn both **using middleware** and **understanding the underlying idea**.

---

# 20. 🎤 Interview Questions

### Q1. What is rate limiting?

Rate limiting restricts the number of requests a client can make within a defined period.

---

### Q2. What HTTP status is used when the limit is exceeded?

```text
429 Too Many Requests
```

---

### Q3. Why is rate limiting important for login APIs?

It can reduce excessive login attempts and help mitigate brute-force abuse.

---

### Q4. What is throttling?

Throttling controls how frequently requests can be processed or accepted.

---

### Q5. Why use `express-rate-limit`?

It provides reusable Express middleware for applying request limits without implementing all the rate-limit logic manually.

---

### Q6. Why is an in-memory `Map` not ideal for multiple Node.js processes?

Each process has its own memory, so each process would maintain a separate counter.

---

### Q7. What can be used for distributed rate limiting?

A shared data store such as Redis can be used so multiple application instances can access the same counters.

---

# 21. 🧾 Quick Cheat Sheet

```text
Install
npm install express-rate-limit

Middleware
rateLimit({
  windowMs: 60 * 1000,
  max: 5
})

Apply
router.post("/login", loginRateLimiter, controller)

Exceeded
HTTP 429

Custom throttle
Map + IP + timestamp + request count

Production distributed systems
Shared store such as Redis
```

---

# ✅ Day 6 Rate Limiting Checklist

- [ ] Install `express-rate-limit`
- [ ] Create `security/rateLimiter.js`
- [ ] Configure `windowMs`
- [ ] Configure `max`
- [ ] Create `/auth/login`
- [ ] Apply rate limiter middleware
- [ ] Register route in `app.js`
- [ ] Test with Postman
- [ ] Verify `429 Too Many Requests`
- [ ] Create custom throttle
- [ ] Test `5 requests / 10 seconds`
- [ ] Understand why distributed systems need shared state

---

## 🎯 Final Mental Model

Remember this:

```text
RATE LIMITING

How many requests?
        ↓
       MAX
        ↓
During what period?
        ↓
    WINDOW
        ↓
Exceeded?
        ↓
     HTTP 429
```

And for your custom throttle:

```text
Client IP
   ↓
Find counter
   ↓
Check time window
   ↓
Under limit? ── YES ──► Allow
   │
   NO
   ↓
HTTP 429
```

> 💡 **Interview one-liner:**  
> “Rate limiting controls how many requests a client can make within a specified time window, while throttling controls request frequency. In Express, `express-rate-limit` can implement rate limiting, and a custom middleware can implement application-specific throttling logic.”
