# 🔐 OAuth 2.0 Authentication in Node.js

> A practical, production-oriented learning guide for implementing
> **Google OAuth 2.0** in a Node.js + Express application.

------------------------------------------------------------------------

## 📚 Table of Contents

1.  [What is OAuth?](#-what-is-oauth)
2.  [OAuth vs JWT vs bcrypt](#-oauth-vs-jwt-vs-bcrypt)
3.  [OAuth Flow](#-oauth-flow)
4.  [Project Architecture](#-project-architecture)
5.  [Prerequisites](#-prerequisites)
6.  [Step 1 --- Create a Google Cloud
    Project](#-step-1--create-a-google-cloud-project)
7.  [Step 2 --- Configure OAuth Consent
    Screen](#-step-2--configure-oauth-consent-screen)
8.  [Step 3 --- Create OAuth Client
    ID](#-step-3--create-oauth-client-id)
9.  [Step 4 --- Configure Redirect
    URI](#-step-4--configure-redirect-uri)
10. [Step 5 --- Store Credentials
    Securely](#-step-5--store-credentials-securely)
11. [Step 6 --- Install Passport](#-step-6--install-passport)
12. [Step 7 --- Configure Google
    Strategy](#-step-7--configure-google-strategy)
13. [Step 8 --- Initialize Passport](#-step-8--initialize-passport)
14. [Step 9 --- Create OAuth Routes](#-step-9--create-oauth-routes)
15. [Step 10 --- Test Google Login](#-step-10--test-google-login)
16. [Step 11 --- Save/Finding the User in
    MongoDB](#-step-11--savefinding-the-user-in-mongodb)
17. [Step 12 --- Issue Your JWT](#-step-12--issue-your-jwt)
18. [Step 13 --- Protect APIs with
    JWT](#-step-13--protect-apis-with-jwt)
19. [Step 14 --- Production Security](#-step-14--production-security)
20. [Common Errors](#-common-errors)
21. [Interview Questions](#-interview-questions)
22. [Final Flow](#-final-flow)

------------------------------------------------------------------------

# 🌟 What is OAuth?

**OAuth 2.0** is an authorization framework that allows an application
to obtain limited access to a user's information from another service
without asking the user for that service's password.

For example:

``` text
Your Node.js Application
        │
        │ "Login with Google"
        ▼
      Google
        │
        │ User authenticates
        ▼
   Google Profile
        │
        ▼
Your Node.js Application
```

Your application **does not receive the user's Google password**.

------------------------------------------------------------------------

# 🔄 OAuth vs JWT vs bcrypt

These technologies solve different problems.

  -----------------------------------------------------------------------
  Technology                          Purpose
  ----------------------------------- -----------------------------------
  **bcrypt**                          Hash and verify passwords

  **OAuth 2.0**                       Integrate
                                      authentication/authorization with
                                      an external provider

  **Passport.js**                     Simplifies authentication strategy
                                      integration

  **JWT**                             Represents authenticated identity
                                      between client and API

  **Google**                          External identity provider
  -----------------------------------------------------------------------

### Normal login

``` text
Email + Password
      ↓
bcrypt.compare()
      ↓
User authenticated
      ↓
JWT
      ↓
Protected API
```

### Google OAuth login

``` text
Google Login
      ↓
Google authenticates user
      ↓
Google profile
      ↓
Find/Create local user
      ↓
Generate YOUR JWT
      ↓
Protected API
```

> 💡 **Important:** OAuth and JWT can be used together. OAuth handles
> the external authentication flow; your application can then issue its
> own JWT for API access.

------------------------------------------------------------------------

# 🔁 OAuth Flow

A simplified Google OAuth authorization-code flow looks like this:

``` text
┌──────────────┐
│    Browser   │
└──────┬───────┘
       │
       │ GET /auth/google
       ▼
┌──────────────┐
│ Node/Express │
│ + Passport   │
└──────┬───────┘
       │
       │ Redirect
       ▼
┌──────────────┐
│    Google    │
└──────┬───────┘
       │
       │ User signs in
       ▼
┌──────────────┐
│    Google    │
└──────┬───────┘
       │
       │ authorization code
       ▼
┌────────────────────┐
│ /auth/google/      │
│ callback           │
└─────────┬──────────┘
          │
          ▼
┌────────────────────┐
│ Passport Google    │
│ Strategy           │
└─────────┬──────────┘
          │
          ▼
┌────────────────────┐
│ Find/Create User   │
│ in MongoDB         │
└─────────┬──────────┘
          │
          ▼
┌────────────────────┐
│ Generate JWT       │
└─────────┬──────────┘
          │
          ▼
       Client
```

------------------------------------------------------------------------

# 🏗️ Project Architecture

A clean Node.js project can look like:

``` text
node-day-5/
│
├── config/
│   └── passport.js
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
├── security/
│   └── httpsServer.js
│
├── utils/
│   ├── env.js
│   ├── jwt.js
│   └── logger.js
│
├── .env
├── .gitignore
├── app.js
└── package.json
```

### Responsibility of each layer

  Layer                  Responsibility
  ---------------------- ---------------------------------------
  `routes`               Define API endpoints
  `controller`           Business/request handling
  `model`                MongoDB data structure
  `middleware`           Authentication and request processing
  `config/passport.js`   OAuth provider configuration
  `utils/jwt.js`         JWT creation
  `.env`                 Secrets/configuration
  `app.js`               Application setup

------------------------------------------------------------------------

# ✅ Prerequisites

Before implementing Google OAuth, make sure you have:

-   Node.js installed
-   Express application working
-   MongoDB/Mongoose configured
-   `.env` configured
-   JWT authentication working
-   A Google account
-   Google Cloud project

Recommended packages:

``` bash
npm install express mongoose dotenv passport passport-google-oauth20 jsonwebtoken bcrypt
```

Optional security/logging packages:

``` bash
npm install helmet cors morgan
```

------------------------------------------------------------------------

# 🟢 Step 1 --- Create a Google Cloud Project

Open Google Cloud Console.

Create a new project, for example:

``` text
NodeJS Authentication
```

Select the project after creation.

------------------------------------------------------------------------

# 🟢 Step 2 --- Configure OAuth Consent Screen

In Google Cloud Console:

``` text
Google Cloud
   ↓
APIs & Services
   ↓
OAuth consent screen
```

Configure the application information.

Typical development configuration includes:

``` text
Application name
Support email
Developer contact information
```

If Google asks for scopes, request only the information your application
actually needs.

For our example:

``` text
profile
email
```

### 💡 Principle of least privilege

Do not request unnecessary scopes.

If your application only needs basic identity information, don't request
unrelated access to the user's Google account.

------------------------------------------------------------------------

# 🟢 Step 3 --- Create OAuth Client ID

Navigate to:

``` text
APIs & Services
       ↓
Credentials
       ↓
Create Credentials
       ↓
OAuth Client ID
```

Select:

``` text
Application type:
Web application
```

Give it a meaningful name:

``` text
NodeJS OAuth Client
```

Google will provide:

``` text
Client ID
Client Secret
```

### ⚠️ Keep the Client Secret private

Never:

``` text
❌ Commit it to GitHub
❌ Put it in frontend JavaScript
❌ Send it to users
❌ Share it publicly
```

------------------------------------------------------------------------

# 🟢 Step 4 --- Configure Redirect URI

For local development:

``` text
http://localhost:3000/auth/google/callback
```

Add it under:

``` text
Authorized redirect URIs
```

### Exact matching matters

These are different:

``` text
http://localhost:3000/auth/google/callback
```

``` text
http://localhost:3000/auth/google
```

``` text
https://localhost:3000/auth/google/callback
```

For local HTTP development, use the exact URL configured for your
application.

------------------------------------------------------------------------

# 🟢 Step 5 --- Store Credentials Securely

Add the values to `.env`:

``` env
PORT_NUMBER=3000

MONGO_URI=your_mongodb_connection_string

JWT_SECRET=your_long_random_jwt_secret

GOOGLE_CLIENT_ID=your_google_client_id

GOOGLE_CLIENT_SECRET=your_google_client_secret

GOOGLE_CALLBACK_URL=http://localhost:3000/auth/google/callback
```

Add `.env` to `.gitignore`:

``` gitignore
node_modules/
.env
```

### Production

In production, provide secrets through the deployment platform's
environment/secret management system rather than committing `.env`.

------------------------------------------------------------------------

# 🟢 Step 6 --- Install Passport

Install:

``` bash
npm install passport passport-google-oauth20
```

Passport provides an authentication framework and the Google strategy
provides Google-specific authentication support.

------------------------------------------------------------------------

# 🟢 Step 7 --- Configure Google Strategy

Create:

``` text
config/passport.js
```

Example:

``` js
import "dotenv/config";

import passport from "passport";
import { Strategy as GoogleStrategy } from "passport-google-oauth20";

passport.use(
  new GoogleStrategy(
    {
      clientID: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
      callbackURL: process.env.GOOGLE_CALLBACK_URL,
    },
    async (accessToken, refreshToken, profile, done) => {
      try {
        console.log("[GOOGLE] User authenticated");
        console.log("Google ID:", profile.id);
        console.log("Name:", profile.displayName);
        console.log("Email:", profile.emails?.[0]?.value);

        return done(null, profile);
      } catch (error) {
        return done(error, null);
      }
    }
  )
);

export default passport;
```

### What is `profile`?

Google returns profile information through Passport.

Common fields include:

``` js
profile.id
profile.displayName
profile.emails?.[0]?.value
```

The exact profile data available depends on the provider and requested
scopes.

------------------------------------------------------------------------

# 🟢 Step 8 --- Initialize Passport

In `app.js`:

``` js
import passport from "./config/passport.js";
```

After your normal middleware:

``` js
app.use(cors());
app.use(helmet());
app.use(express.json());
app.use(morgan("dev"));

app.use(passport.initialize());
```

### Why?

`passport.initialize()` initializes Passport for Express.

------------------------------------------------------------------------

# 🟢 Step 9 --- Create OAuth Routes

In:

``` text
routes/auth.js
```

Import Passport:

``` js
import passport from "passport";
```

Create the login route:

``` js
router.get(
  "/auth/google",
  passport.authenticate("google", {
    scope: ["profile", "email"],
  })
);
```

Create the callback route:

``` js
router.get(
  "/auth/google/callback",
  passport.authenticate("google", {
    session: false,
    failureRedirect: "/login",
  }),
  (req, res) => {
    res.status(200).json({
      success: true,
      message: "Google authentication successful",
      googleUser: req.user,
    });
  }
);
```

### Why two routes?

#### Start authentication

``` text
GET /auth/google
```

Starts the Google login process.

#### Callback

``` text
GET /auth/google/callback
```

Google redirects the browser here after authentication.

------------------------------------------------------------------------

# 🟢 Step 10 --- Test Google Login

Start your application:

``` bash
npm run dev
```

Open:

``` text
http://localhost:3000/auth/google
```

Expected flow:

``` text
Your application
      ↓
Google Login
      ↓
Select Google account
      ↓
Google authentication
      ↓
Callback route
      ↓
Passport
      ↓
Google profile
```

At this stage, returning the Google profile is enough to prove that the
OAuth flow is working.

------------------------------------------------------------------------

# 🟢 Step 11 --- Save/Find the User in MongoDB

Once OAuth works, connect the Google profile to your local `User`
collection.

A user model may eventually contain fields such as:

``` js
{
  name: String,
  email: String,
  password: String,
  provider: String,
  providerId: String
}
```

For a Google user:

``` text
provider = "google"
providerId = Google profile ID
```

Conceptually:

``` text
Google profile
      ↓
Find by provider + providerId
      ↓
Found?
 ┌────┴────┐
Yes       No
 │         │
Use      Create
user      user
```

### Important design consideration

A user authenticated through Google may not have a password created in
your local system.

Therefore, your user model and registration logic should distinguish:

``` text
Local account
→ password authentication

Google account
→ Google OAuth authentication
```

------------------------------------------------------------------------

# 🟢 Step 12 --- Issue Your JWT

After finding/creating the local user:

``` js
const token = generateToken(user);
```

Your existing JWT utility can be reused:

``` js
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

Then return the JWT to the client according to your application's
authentication design.

Conceptually:

``` text
Google
  ↓
OAuth success
  ↓
Local User
  ↓
JWT
  ↓
Client
```

------------------------------------------------------------------------

# 🟢 Step 13 --- Protect APIs with JWT

After Google login, your application's protected APIs can continue using
the same JWT middleware.

For example:

``` text
GET /profile
```

with:

``` http
Authorization: Bearer <JWT>
```

Flow:

``` text
Google OAuth
     ↓
Application JWT
     ↓
Authorization header
     ↓
authMiddleware
     ↓
jwt.verify()
     ↓
Protected API
```

This means you don't need separate authentication logic for every
protected endpoint.

------------------------------------------------------------------------

# 🔐 Step 14 --- Production Security

## 1. Protect client secrets

Never expose:

``` text
GOOGLE_CLIENT_SECRET
JWT_SECRET
```

to the frontend.

------------------------------------------------------------------------

## 2. Use HTTPS in production

Production OAuth should use HTTPS:

``` text
https://your-domain.com/auth/google/callback
```

rather than:

``` text
http://localhost:3000/auth/google/callback
```

------------------------------------------------------------------------

## 3. Use exact redirect URIs

Register only the redirect URIs you actually need.

Avoid overly broad redirect patterns.

------------------------------------------------------------------------

## 4. Request minimum scopes

Use only the scopes required by the application:

``` js
scope: ["profile", "email"]
```

------------------------------------------------------------------------

## 5. Protect JWT secrets

Use a strong random secret and store it using secure secret management.

------------------------------------------------------------------------

## 6. Don't put passwords in JWT

Never create:

``` js
{
  email,
  password
}
```

inside a JWT.

Use minimal identity information such as:

``` js
{
  userId,
  email
}
```

------------------------------------------------------------------------

## 7. Consider secure token storage

For browser-based applications, carefully choose how authentication
tokens are stored and transmitted. For many web applications, secure,
appropriately configured cookies can be preferable to exposing
long-lived tokens to JavaScript.

------------------------------------------------------------------------

# 🐛 Common Errors

## ❌ `Cannot GET /auth/google`

Check that:

``` js
router.get(
  "/auth/google",
  ...
);
```

contains the leading `/`.

Also check that the router is mounted:

``` js
app.use("/", authRouter);
```

------------------------------------------------------------------------

## ❌ `Missing required parameter: redirect_uri`

Check:

``` js
callbackURL: process.env.GOOGLE_CALLBACK_URL
```

and:

``` env
GOOGLE_CALLBACK_URL=http://localhost:3000/auth/google/callback
```

Also verify the same URL is configured in Google Cloud.

------------------------------------------------------------------------

## ❌ Redirect URI mismatch

The URI configured in Google Cloud must match the URI used by Passport.

For example:

``` text
Google Cloud:
http://localhost:3000/auth/google/callback

Passport:
http://localhost:3000/auth/google/callback
```

They must correspond exactly.

------------------------------------------------------------------------

## ❌ `process.env.GOOGLE_CALLBACK_URL` is undefined

Check:

``` text
.env
```

is in the project root:

``` text
node-day-5/
├── .env
├── app.js
└── config/
    └── passport.js
```

For an ESM project, loading environment configuration explicitly in the
Passport configuration can also be useful:

``` js
import "dotenv/config";
```

------------------------------------------------------------------------

## ❌ Google authentication succeeds but user isn't saved

OAuth authentication and application user persistence are separate
steps.

You still need:

``` text
Google profile
      ↓
Find/Create MongoDB user
```

------------------------------------------------------------------------

# 🎤 Interview Questions

### Q1. What is OAuth?

> OAuth 2.0 is an authorization framework that allows applications to
> obtain access to resources or use an external identity provider
> without requiring the user's provider password to be given to the
> application.

### Q2. Is OAuth the same as JWT?

> No. OAuth is a protocol/framework for delegated authorization and
> commonly used for external authentication flows. JWT is a token format
> commonly used to represent claims between systems.

### Q3. Why use Passport.js?

> Passport provides a middleware-based authentication framework and
> supports different authentication strategies, including Google OAuth.

### Q4. What is a redirect URI?

> It is the callback URL where the OAuth provider sends the user after
> the authorization step.

### Q5. What is a Client ID?

> It identifies the OAuth application to the provider.

### Q6. What is a Client Secret?

> It is a confidential credential associated with the OAuth client and
> must be protected on the server.

### Q7. Why shouldn't we store the Google password?

> The application never needs the user's Google password. Google handles
> authentication and provides the OAuth result to the application.

### Q8. Can OAuth and JWT be used together?

> Yes. OAuth can authenticate the user through Google, after which the
> application can create its own local user record and issue a JWT for
> access to its APIs.

------------------------------------------------------------------------

# 🏁 Final Architecture

Your complete authentication architecture can eventually look like:

``` text
                    ┌─────────────────────┐
                    │       Client        │
                    └──────────┬──────────┘
                               │
                  ┌────────────┴────────────┐
                  │                         │
             Email Login              Google Login
                  │                         │
                  ▼                         ▼
              bcrypt                    OAuth 2.0
                  │                         │
                  │                    Google
                  │                         │
                  │                    Passport
                  │                         │
                  └────────────┬────────────┘
                               │
                               ▼
                     ┌─────────────────┐
                     │  MongoDB User   │
                     │    Account      │
                     └────────┬────────┘
                              │
                              ▼
                     ┌─────────────────┐
                     │      JWT        │
                     └────────┬────────┘
                              │
                              ▼
                    ┌──────────────────┐
                    │ Protected APIs   │
                    │ /profile         │
                    │ /posts           │
                    │ /orders          │
                    │ etc.            │
                    └──────────────────┘
```

------------------------------------------------------------------------

# ✅ Implementation Checklist

### Google Cloud

-   [ ] Create Google Cloud project
-   [ ] Configure OAuth consent screen
-   [ ] Create Web OAuth Client
-   [ ] Configure authorized redirect URI
-   [ ] Copy Client ID
-   [ ] Copy Client Secret

### Node.js

-   [ ] Install Passport
-   [ ] Install Google strategy
-   [ ] Configure `.env`
-   [ ] Create `config/passport.js`
-   [ ] Initialize Passport
-   [ ] Create `/auth/google`
-   [ ] Create `/auth/google/callback`
-   [ ] Test Google authentication
-   [ ] Find/Create MongoDB user
-   [ ] Generate application JWT
-   [ ] Protect APIs with JWT middleware

### Security

-   [ ] `.env` in `.gitignore`
-   [ ] Never expose Client Secret
-   [ ] Never expose JWT Secret
-   [ ] Use minimum OAuth scopes
-   [ ] Use exact redirect URIs
-   [ ] Use HTTPS in production
-   [ ] Validate user data
-   [ ] Handle OAuth failures
-   [ ] Handle duplicate accounts
-   [ ] Protect authenticated APIs

------------------------------------------------------------------------

# 🎯 Learning Order

For a beginner-friendly implementation, follow this exact order:

``` text
1. Understand OAuth
       ↓
2. Create Google Cloud OAuth Client
       ↓
3. Configure redirect URI
       ↓
4. Install Passport
       ↓
5. Configure Google Strategy
       ↓
6. Initialize Passport
       ↓
7. Create OAuth routes
       ↓
8. Test Google Login
       ↓
9. Find/Create MongoDB User
       ↓
10. Generate JWT
       ↓
11. Protect APIs
       ↓
12. Add production security
```

> 🚀 **Recommended next step for your current Day 5 project:** you have
> completed steps 1--8 and successfully tested Google OAuth. Continue
> with **Find/Create MongoDB User → Generate JWT**.
