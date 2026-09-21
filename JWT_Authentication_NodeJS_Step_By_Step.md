# 🔐 JWT Authentication in Node.js + Express

> A beautiful, step-by-step implementation guide for building JWT-based authentication with **Node.js, Express, MongoDB, Mongoose, bcrypt, and jsonwebtoken**.

---

# 🎯 What We Are Building

We will implement this authentication flow:

```text
                    ┌───────────────┐
                    │     Client    │
                    │ Postman / UI  │
                    └───────┬───────┘
                            │
                 ┌──────────┴──────────┐
                 │                     │
             REGISTER                 LOGIN
                 │                     │
                 ▼                     ▼
             bcrypt hash          bcrypt compare
                 │                     │
                 ▼                     ▼
              MongoDB              Generate JWT
                                       │
                                       ▼
                                  Return Token
                                       │
                                       ▼
                              Protected API
                                       │
                              Authorization:
                              Bearer <token>
                                       │
                                       ▼
                              JWT Middleware
                                       │
                              jwt.verify()
                                       │
                                       ▼
                                  Controller
```

---

# 📚 Technologies

| Technology | Purpose |
|---|---|
| **Node.js** | JavaScript runtime |
| **Express** | REST API framework |
| **MongoDB** | Database |
| **Mongoose** | MongoDB ODM |
| **bcrypt** | Password hashing |
| **jsonwebtoken** | JWT creation and verification |
| **dotenv** | Environment variables |
| **Postman** | API testing |

---

# 🧠 1. What Is JWT?

**JWT = JSON Web Token**

JWT is a token that a server gives to a user after successful authentication.

The client sends that token with future requests to prove:

> "I have already authenticated."

Example:

```text
Login
  ↓
Server verifies email + password
  ↓
Server generates JWT
  ↓
Client receives JWT
  ↓
Client sends JWT with protected requests
  ↓
Server verifies JWT
  ↓
Request is allowed
```

---

# 🔐 2. JWT Authentication vs Password Authentication

Without JWT:

```text
Client → email + password → Server
```

With JWT:

```text
LOGIN
Client → email + password → Server
                         ↓
                      JWT Token
                         ↓
Client ←─────────────────┘

NEXT REQUEST
Client → JWT → Server
               ↓
            Verify
               ↓
             Allow
```

The password is normally used during login, while the JWT is used for subsequent authenticated API requests.

---

# 🧩 3. JWT Has Three Parts

A JWT normally looks like:

```text
xxxxx.yyyyy.zzzzz
```

It contains:

```text
HEADER.PAYLOAD.SIGNATURE
```

### Header

Contains token metadata such as the signing algorithm.

Example:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload

Contains claims.

Example:

```json
{
  "userId": "64abc123",
  "email": "john@example.com"
}
```

### Signature

The signature allows the server to verify that the token was signed using the expected secret.

---

## ⚠️ Important JWT Security Rule

JWT payloads are **encoded, not encrypted**.

Do NOT store:

```text
password
credit card number
secret keys
private information
```

inside a JWT payload.

---

# 🚀 STEP 1 — Create the Node.js Project

Create the project:

```powershell
mkdir node-jwt-auth
cd node-jwt-auth
npm init -y
```

Set ES Modules in `package.json`:

```json
{
  "type": "module"
}
```

---

# 📦 STEP 2 — Install Dependencies

Install:

```powershell
npm install express mongoose bcrypt jsonwebtoken dotenv
```

Development dependency:

```powershell
npm install --save-dev nodemon
```

---

# 📁 STEP 3 — Create the Project Structure

Recommended structure:

```text
node-jwt-auth/
│
├── controller/
│   └── authController.js
│
├── middleware/
│   ├── authMiddleware.js
│   └── errorHandler.js
│
├── model/
│   └── User.js
│
├── routes/
│   └── auth.js
│
├── utils/
│   ├── env.js
│   └── jwt.js
│
├── .env
├── .gitignore
├── app.js
├── package.json
└── package-lock.json
```

---

# 🔒 STEP 4 — Configure Environment Variables

Create:

```text
.env
```

Add:

```env
PORT_NUMBER=3000
MONGO_URI=your_mongodb_connection_string
JWT_SECRET=your_long_random_secret_key
```

Example:

```env
PORT_NUMBER=3000
MONGO_URI=mongodb+srv://username:password@cluster.mongodb.net/authdb
JWT_SECRET=change_this_to_a_long_random_secret
```

### 🚨 Never expose `JWT_SECRET`

Do not:

```js
const secret = "mysecret123";
```

Use:

```js
process.env.JWT_SECRET
```

---

# 🛡️ STEP 5 — Configure `.gitignore`

Create:

```text
.gitignore
```

Add:

```gitignore
node_modules/
.env
```

Your `.env` contains:

```text
MongoDB credentials
JWT secret
```

Therefore it should never be committed to Git.

---

# 👤 STEP 6 — Create the User Model

Create:

```text
model/User.js
```

Add:

```js
import mongoose from "mongoose";

const userSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: [true, "Name is required"],
      trim: true,
    },

    email: {
      type: String,
      required: [true, "Email is required"],
      unique: true,
      trim: true,
      lowercase: true,
    },

    password: {
      type: String,
      required: [true, "Password is required"],
    },
  },
  {
    timestamps: true,
  }
);

const User = mongoose.model("User", userSchema);

export default User;
```

---

# 🔑 STEP 7 — Understand bcrypt

Never store:

```text
password = "John@12345"
```

Instead:

```text
Plain Password
      │
      ▼
    bcrypt
      │
      ▼
Hashed Password
```

Example:

```text
$2b$10$................................................
```

### Important

bcrypt hashing is **one-way**.

You do not decrypt a bcrypt password.

During login:

```text
Entered Password
      │
      ▼
bcrypt.compare()
      │
      ▼
Stored Hash
```

---

# 📝 STEP 8 — Create Registration Controller

Create:

```text
controller/authController.js
```

Add:

```js
import bcrypt from "bcrypt";
import User from "../model/User.js";

export const register = async (req, res, next) => {
  try {
    const { name, email, password } = req.body;

    if (!name || !email || !password) {
      const error = new Error(
        "Name, email and password are required"
      );

      error.statusCode = 400;
      return next(error);
    }

    const existingUser = await User.findOne({
      email: email.trim().toLowerCase(),
    });

    if (existingUser) {
      const error = new Error("User already exists");

      error.statusCode = 409;
      return next(error);
    }

    const hashedPassword = await bcrypt.hash(password, 10);

    const user = await User.create({
      name,
      email: email.trim().toLowerCase(),
      password: hashedPassword,
    });

    res.status(201).json({
      success: true,
      message: "User registered successfully",
      user: {
        id: user._id,
        name: user.name,
        email: user.email,
      },
    });
  } catch (error) {
    next(error);
  }
};
```

---

# 🧠 Registration Flow

```text
POST /register
      │
      ▼
Read name/email/password
      │
      ▼
Validate input
      │
      ▼
Find existing user
      │
      ├── Exists → 409
      │
      ▼
bcrypt.hash()
      │
      ▼
Save user in MongoDB
      │
      ▼
201 Created
```

---

# 🔐 STEP 9 — Create JWT Utility

Create:

```text
utils/jwt.js
```

Add:

```js
import jwt from "jsonwebtoken";

export const generateToken = (user) => {
  return jwt.sign(
    {
      userId: user._id,
      email: user.email,
    },
    process.env.JWT_SECRET,
    {
      expiresIn: "1h",
    }
  );
};
```

### Why separate this file?

Instead of generating JWTs inside multiple controllers, keep token creation in one reusable utility.

```text
Controller
    │
    ▼
generateToken()
    │
    ▼
JWT
```

---

# 🔑 STEP 10 — Create Login Controller

Add to:

```text
controller/authController.js
```

```js
import { generateToken } from "../utils/jwt.js";

export const login = async (req, res, next) => {
  try {
    const email = req.body.email?.trim().toLowerCase();
    const { password } = req.body;

    if (!email || !password) {
      const error = new Error(
        "Email and password are required"
      );

      error.statusCode = 400;
      return next(error);
    }

    const user = await User.findOne({ email });

    if (!user) {
      const error = new Error("Invalid credentials");

      error.statusCode = 401;
      return next(error);
    }

    const isPasswordCorrect = await bcrypt.compare(
      password,
      user.password
    );

    if (!isPasswordCorrect) {
      const error = new Error("Invalid credentials");

      error.statusCode = 401;
      return next(error);
    }

    const token = generateToken(user);

    res.status(200).json({
      success: true,
      message: "JWT Token Issued",
      token,
    });
  } catch (error) {
    next(error);
  }
};
```

---

# 🔄 STEP 11 — Understand the Login Flow

```text
POST /login
     │
     ▼
Email + Password
     │
     ▼
Find User
     │
     ▼
bcrypt.compare()
     │
     ├── Wrong → 401
     │
     ▼
generateToken()
     │
     ▼
JWT
     │
     ▼
Send JWT to Client
```

---

# 🛡️ STEP 12 — Create JWT Middleware

Create:

```text
middleware/authMiddleware.js
```

Add:

```js
import jwt from "jsonwebtoken";

export const authMiddleware = (req, res, next) => {
  try {
    const authHeader = req.headers.authorization;

    if (!authHeader) {
      const error = new Error("No token provided");

      error.statusCode = 401;
      return next(error);
    }

    const [type, token] = authHeader.split(" ");

    if (type !== "Bearer" || !token) {
      const error = new Error(
        "Invalid authorization format"
      );

      error.statusCode = 401;
      return next(error);
    }

    const decoded = jwt.verify(
      token,
      process.env.JWT_SECRET
    );

    req.user = decoded;

    next();
  } catch (error) {
    if (error.name === "TokenExpiredError") {
      error.statusCode = 401;
      error.message = "Token has expired";
    } else if (error.name === "JsonWebTokenError") {
      error.statusCode = 401;
      error.message = "Invalid token";
    } else {
      error.statusCode = 401;
    }

    next(error);
  }
};
```

---

# 🧠 STEP 13 — Understand Middleware

The client sends:

```http
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
```

Middleware receives:

```text
Authorization Header
        │
        ▼
Check Bearer
        │
        ▼
Extract Token
        │
        ▼
jwt.verify()
        │
        ├── Invalid → 401
        ├── Expired → 401
        │
        ▼
    req.user
        │
        ▼
     next()
```

---

# 👤 STEP 14 — Create Protected Profile API

Add:

```js
export const getProfile = async (req, res, next) => {
  try {
    const user = await User.findById(
      req.user.userId
    ).select("-password");

    if (!user) {
      const error = new Error("User not found");

      error.statusCode = 404;
      return next(error);
    }

    res.status(200).json({
      success: true,
      message: "Profile fetched successfully",
      user,
    });
  } catch (error) {
    next(error);
  }
};
```

### Why `.select("-password")`?

It prevents the password hash from being returned:

```js
.select("-password")
```

---

# 🚦 STEP 15 — Create Authentication Routes

Create:

```text
routes/auth.js
```

Add:

```js
import express from "express";

import {
  register,
  login,
  getProfile,
} from "../controller/authController.js";

import {
  authMiddleware,
} from "../middleware/authMiddleware.js";

const router = express.Router();

router.post("/register", register);

router.post("/login", login);

router.get(
  "/profile",
  authMiddleware,
  getProfile
);

export default router;
```

---

# 🚀 STEP 16 — Configure Express

In `app.js`:

```js
import express from "express";
import dotenv from "dotenv";
import mongoose from "mongoose";

import authRouter from "./routes/auth.js";

dotenv.config();

const app = express();

const PORT_NUMBER =
  process.env.PORT_NUMBER || 3000;

app.use(express.json());

app.use("/", authRouter);

mongoose
  .connect(process.env.MONGO_URI)
  .then(() => {
    console.log(
      "[MongoDB Connected] Database connection successful"
    );

    app.listen(PORT_NUMBER, () => {
      console.log(
        `[Server Running] http://localhost:${PORT_NUMBER}`
      );
    });
  })
  .catch((error) => {
    console.error(
      "[MongoDB Error]",
      error.message
    );
  });
```

---

# 🧪 STEP 17 — Test Registration

### Request

```text
POST http://localhost:3000/register
```

### Body → raw → JSON

```json
{
  "name": "John",
  "email": "john@example.com",
  "password": "John@12345"
}
```

### Expected Response

```json
{
  "success": true,
  "message": "User registered successfully",
  "user": {
    "id": "...",
    "name": "John",
    "email": "john@example.com"
  }
}
```

### Status

```text
201 Created
```

---

# 🔑 STEP 18 — Test Login

### Request

```text
POST http://localhost:3000/login
```

### Body

```json
{
  "email": "john@example.com",
  "password": "John@12345"
}
```

### Expected Response

```json
{
  "success": true,
  "message": "JWT Token Issued",
  "token": "eyJhbGciOiJIUzI1NiIs..."
}
```

### Status

```text
200 OK
```

---

# 🔐 STEP 19 — Test Protected API

### Request

```text
GET http://localhost:3000/profile
```

In Postman:

```text
Authorization
     ↓
Type: Bearer Token
     ↓
Paste JWT
```

Or manually:

```http
Authorization: Bearer YOUR_JWT_TOKEN
```

### Expected Response

```json
{
  "success": true,
  "message": "Profile fetched successfully",
  "user": {
    "_id": "...",
    "name": "John",
    "email": "john@example.com"
  }
}
```

---

# ❌ STEP 20 — Test Authentication Errors

## No Token

```text
GET /profile
```

Expected:

```text
401 Unauthorized
```

```json
{
  "success": false,
  "message": "No token provided"
}
```

---

## Invalid Token

```text
Authorization: Bearer abc123
```

Expected:

```text
401 Unauthorized
```

```json
{
  "success": false,
  "message": "Invalid token"
}
```

---

## Expired Token

After the JWT expires:

```text
401 Unauthorized
```

```json
{
  "success": false,
  "message": "Token has expired"
}
```

---

# 🔒 STEP 21 — Important Security Rules

### ❌ Never store plain passwords

Bad:

```text
password: "John@12345"
```

Good:

```text
password: "$2b$10$..."
```

---

### ❌ Never put passwords inside JWT

Bad:

```json
{
  "email": "john@example.com",
  "password": "John@12345"
}
```

Good:

```json
{
  "userId": "64abc123",
  "email": "john@example.com"
}
```

---

### ❌ Never hard-code JWT secret

Bad:

```js
jwt.sign(data, "my-secret");
```

Good:

```js
jwt.sign(
  data,
  process.env.JWT_SECRET
);
```

---

### ❌ Never commit `.env`

Add:

```gitignore
.env
```

---

### ✅ Use HTTPS

JWT should be transmitted over HTTPS in production:

```text
HTTPS
  +
JWT
  +
Secure client-side token handling
```

---

# 🧩 STEP 22 — JWT vs bcrypt

These two are often confused.

| bcrypt | JWT |
|---|---|
| Password security | Authentication token |
| Hashes passwords | Represents authenticated session/claims |
| Used during register/login | Used after login |
| One-way hash | Signed token |
| `bcrypt.hash()` | `jwt.sign()` |
| `bcrypt.compare()` | `jwt.verify()` |

### Simple analogy

```text
bcrypt
  ↓
Locks your password safely in the database

JWT
  ↓
Gives the authenticated user a temporary digital ID
```

---

# 🧠 STEP 23 — JWT Interview Explanation

### Question: How does JWT authentication work?

**Answer:**

> During login, the server verifies the user's credentials. If they are valid, the server creates a signed JWT containing necessary claims such as the user ID. The client sends that token in the Authorization header for protected requests. Middleware verifies the token using the server's secret key and allows the request to continue if the token is valid.

---

# 🎯 STEP 24 — Common Interview Questions

### Q1. What is JWT?

JWT is a compact, signed token format commonly used to carry authentication claims between a client and server.

### Q2. Is JWT encrypted?

No. A normal JWT is encoded and signed, not encrypted.

### Q3. What is the purpose of the signature?

The signature allows the server to verify that the token was signed with the expected key and has not been modified.

### Q4. What is `jwt.sign()`?

It creates a signed JWT.

```js
jwt.sign(payload, secret, options)
```

### Q5. What is `jwt.verify()`?

It verifies the JWT's signature and validates registered claims such as expiration.

### Q6. What does `expiresIn: "1h"` mean?

The token is valid for approximately one hour from issuance.

### Q7. What is Bearer authentication?

The client sends a token using:

```http
Authorization: Bearer <token>
```

### Q8. What is middleware doing?

It intercepts the request before the controller, extracts the token, verifies it, and attaches authenticated information to the request.

### Q9. What happens if a JWT expires?

`jwt.verify()` throws a token-expiration error, and the API should normally return `401 Unauthorized`.

### Q10. Why shouldn't passwords be stored in JWT?

JWT payloads are not confidential by default. Their contents can be decoded by anyone who possesses the token.

---

# 🏆 Complete JWT Authentication Flow

```text
                    REGISTER
                       │
                       ▼
              Email + Password
                       │
                       ▼
                    bcrypt
                       │
                       ▼
                    MongoDB
                       │
                       │
                       ▼
                     LOGIN
                       │
                       ▼
              Email + Password
                       │
                       ▼
               Find User
                       │
                       ▼
             bcrypt.compare()
                       │
                ┌──────┴──────┐
                │             │
              FAIL          SUCCESS
                │             │
                ▼             ▼
               401       generateToken()
                              │
                              ▼
                         Return JWT
                              │
                              ▼
                       Client stores token
                              │
                              ▼
                       Protected Request
                              │
                              ▼
                  Authorization: Bearer JWT
                              │
                              ▼
                     authMiddleware
                              │
                              ▼
                       jwt.verify()
                              │
                     ┌────────┴────────┐
                     │                 │
                   INVALID           VALID
                     │                 │
                     ▼                 ▼
                    401            req.user
                                       │
                                       ▼
                                  Controller
                                       │
                                       ▼
                                   Response
```

---

# 📁 Final Project Structure

```text
node-jwt-auth/
│
├── controller/
│   └── authController.js
│
├── middleware/
│   ├── authMiddleware.js
│   └── errorHandler.js
│
├── model/
│   └── User.js
│
├── routes/
│   └── auth.js
│
├── utils/
│   ├── env.js
│   └── jwt.js
│
├── .env
├── .gitignore
├── app.js
├── package.json
└── package-lock.json
```

---

# ✅ JWT Implementation Checklist

- [ ] Create Node.js project
- [ ] Install Express
- [ ] Install Mongoose
- [ ] Install bcrypt
- [ ] Install jsonwebtoken
- [ ] Install dotenv
- [ ] Create User model
- [ ] Create registration API
- [ ] Hash password with bcrypt
- [ ] Store hashed password
- [ ] Create login API
- [ ] Compare password with bcrypt
- [ ] Create JWT utility
- [ ] Generate JWT after successful login
- [ ] Create JWT middleware
- [ ] Read Authorization header
- [ ] Validate `Bearer` format
- [ ] Verify JWT
- [ ] Attach decoded user to `req.user`
- [ ] Create protected API
- [ ] Hide password from profile response
- [ ] Handle expired tokens
- [ ] Handle invalid tokens
- [ ] Keep JWT secret in `.env`
- [ ] Add `.env` to `.gitignore`
- [ ] Use HTTPS in production

---

# 🌟 Final Mental Model

Remember these four lines:

```text
REGISTER
Password → bcrypt.hash() → Database

LOGIN
Password → bcrypt.compare() → JWT

PROTECTED REQUEST
JWT → jwt.verify() → Controller

SECURITY
JWT_SECRET → Environment Variable
```

> 🔐 **bcrypt protects passwords.**
>
> 🎟️ **JWT identifies authenticated requests.**
>
> 🛡️ **Middleware protects routes.**
>
> 🔒 **HTTPS protects data in transit.**
