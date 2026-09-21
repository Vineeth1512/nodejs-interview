# 🔐 Node.js Day 5 --- HTTPS Server Setup Guide

> **Goal:** Convert your Node.js + Express API from
> `http://localhost:3000` to a local HTTPS server using Node.js built-in
> `https` module and a self-signed SSL certificate.

------------------------------------------------------------------------

## 🎯 Final Result

Before:

``` text
http://localhost:3000
```

After:

``` text
https://localhost:3000
```

------------------------------------------------------------------------

## 📁 Step 1 --- Create the `security` Folder

Open PowerShell inside your project:

``` powershell
cd "D:\10k-React\7daysTask\nodejs\node-day-5"
```

Create the folder:

``` powershell
New-Item -ItemType Directory security
```

Expected structure:

``` text
node-day-5/
├── config/
├── controller/
├── middleware/
├── model/
├── routes/
├── security/
├── utils/
├── .env
├── app.js
└── package.json
```

------------------------------------------------------------------------

## 🔎 Step 2 --- Check OpenSSL

Run:

``` powershell
openssl version
```

If PowerShell says OpenSSL is not recognized, find it with:

``` powershell
Get-ChildItem "C:\Program Files\OpenSSL*" -Recurse -Filter openssl.exe -ErrorAction SilentlyContinue
```

A typical path is:

``` text
C:\Program Files\OpenSSL-Win64\bin\openssl.exe
```

------------------------------------------------------------------------

## 🔑 Step 3 --- Generate the SSL Certificate

If OpenSSL is not in PATH, use the full path:

``` powershell
& "C:\Program Files\OpenSSL-Win64\bin\openssl.exe" req -x509 -newkey rsa:2048 -keyout security/server.key -out security/server.crt -days 365 -nodes
```

This creates:

``` text
security/
├── server.crt    ← Certificate
└── server.key    ← Private key 🔐
```

### What the options mean

  Option               Meaning
  -------------------- ----------------------------------------
  `req`                Certificate request command
  `-x509`              Create a self-signed certificate
  `-newkey rsa:2048`   Generate a new 2048-bit RSA key
  `-keyout`            Private-key output path
  `-out`               Certificate output path
  `-days 365`          Valid for 365 days
  `-nodes`             Don't password-protect the private key

------------------------------------------------------------------------

## 📝 Step 4 --- Enter Certificate Information

When OpenSSL asks questions, for local development you can use:

  Prompt                Value
  --------------------- ----------------
  Country Name          `IN`
  State                 `Telangana`
  Locality              `Hyderabad`
  Organization          `NodeDay5`
  Organizational Unit   Press Enter
  Common Name           `localhost` ⭐
  Email Address         Press Enter

### ⭐ Important

For:

``` text
Common Name (e.g. server FQDN or YOUR name):
```

enter:

``` text
localhost
```

because the application will be accessed at:

``` text
https://localhost:3000
```

------------------------------------------------------------------------

## 📂 Step 5 --- Verify the Files

Run:

``` powershell
Get-ChildItem security
```

You should see:

``` text
server.crt
server.key
```

------------------------------------------------------------------------

## 🚨 Step 6 --- Protect the SSL Files

Add these to `.gitignore`:

``` gitignore
node_modules/
.env

# SSL certificates
security/server.key
security/server.crt
```

Never commit your private key to GitHub.

------------------------------------------------------------------------

# 🛠️ Step 7 --- Create `httpsServer.js`

Create the file:

``` powershell
New-Item security/httpsServer.js
```

Put this code inside:

``` js
import https from "https";
import fs from "fs";

const startHttpsServer = (app, port) => {
  const sslOptions = {
    key: fs.readFileSync("./security/server.key"),
    cert: fs.readFileSync("./security/server.crt"),
  };

  https.createServer(sslOptions, app).listen(port, () => {
    console.log(`[HTTPS Server Running] https://localhost:${port}`);
  });
};

export default startHttpsServer;
```

### 🧠 What this code does

``` js
fs.readFileSync("./security/server.key")
```

loads the private SSL key.

``` js
fs.readFileSync("./security/server.crt")
```

loads the certificate.

``` js
https.createServer(sslOptions, app)
```

creates an HTTPS server and uses your existing Express `app`.

------------------------------------------------------------------------

# 🔌 Step 8 --- Connect HTTPS to `app.js`

At the top of `app.js`, add:

``` js
import startHttpsServer from "./security/httpsServer.js";
```

Find your current HTTP server:

``` js
app.listen(PORT_NUMBER, () => {
  console.log(`[Server Running] http://localhost:${PORT_NUMBER}`);
});
```

Replace it with:

``` js
startHttpsServer(app, PORT_NUMBER);
```

------------------------------------------------------------------------

# 🧩 Step 9 --- MongoDB + HTTPS Startup

Your startup section should look like:

``` js
mongoose
  .connect(process.env.MONGO_URI)
  .then(() => {
    console.log("[MongoDB Connected] Database connection successful");

    startHttpsServer(app, PORT_NUMBER);
  })
  .catch((error) => {
    console.error("[MongoDB Error] Database connection failed");
    console.error(error.message);
  });
```

Flow:

``` text
Node.js starts
      ↓
Load environment variables
      ↓
Connect MongoDB
      ↓
MongoDB connected
      ↓
Start HTTPS server
      ↓
https://localhost:3000
```

------------------------------------------------------------------------

# ▶️ Step 10 --- Start the Application

Run:

``` powershell
npm run dev
```

Expected:

``` text
[MongoDB Connected] Database connection successful
[HTTPS Server Running] https://localhost:3000
```

------------------------------------------------------------------------

# 🌐 Step 11 --- Test the HTTPS Server

Open:

``` text
https://localhost:3000
```

Expected response:

``` json
{
  "success": true,
  "message": "Day 5 Authentication API is running"
}
```

------------------------------------------------------------------------

# ⚠️ Step 12 --- Self-Signed Certificate Warning

Because this is a self-signed certificate, your browser may show:

``` text
Your connection is not private
```

That is expected for this local-development setup.

For production, use a certificate issued by a trusted Certificate
Authority.

------------------------------------------------------------------------

# 🧪 Step 13 --- Test with Postman

Try:

``` text
GET https://localhost:3000/
```

Postman may also warn about the self-signed certificate.

For local development, you can disable SSL certificate verification in
Postman's settings if necessary.

------------------------------------------------------------------------

# 🔐 Step 14 --- Test Authentication Over HTTPS

Register:

``` text
POST https://localhost:3000/register
```

Login:

``` text
POST https://localhost:3000/login
```

Protected profile:

``` text
GET https://localhost:3000/profile
```

With:

``` http
Authorization: Bearer YOUR_JWT_TOKEN
```

Now the authentication flow is:

``` text
Client
  │
  │ 🔐 HTTPS
  ▼
Node.js + Express
  │
  ├── bcrypt
  ├── JWT
  ├── Google OAuth
  └── Middleware
        │
        ▼
      MongoDB
```

------------------------------------------------------------------------

# 📁 Final Project Structure

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
│   └── authRoute.js
│
├── security/
│   ├── httpsServer.js
│   ├── server.crt
│   └── server.key
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

------------------------------------------------------------------------

# 🎯 Interview Questions

### 1. What is HTTPS?

HTTPS is HTTP communication protected by TLS encryption.

### 2. Why do we need HTTPS?

It protects data exchanged between the client and server from
unauthorized reading or modification.

### 3. What is a self-signed certificate?

A certificate signed by the same entity that created it rather than by a
publicly trusted Certificate Authority.

### 4. Can a self-signed certificate be used in production?

It is generally not suitable for public production applications because
clients do not inherently trust it.

### 5. What is Node.js `https`?

Node.js provides the built-in `https` module for creating HTTPS servers.

### 6. What is the difference between HTTP and HTTPS?

``` text
HTTP
  ↓
Normal HTTP communication

HTTPS
  ↓
HTTP + TLS
  ↓
Encrypted connection
```

------------------------------------------------------------------------

# ✅ HTTPS Checklist

-   [ ] OpenSSL installed
-   [ ] `security` folder created
-   [ ] `server.key` generated
-   [ ] `server.crt` generated
-   [ ] SSL files added to `.gitignore`
-   [ ] `httpsServer.js` created
-   [ ] `https` module imported
-   [ ] Certificate loaded using `fs`
-   [ ] `app.js` connected to HTTPS
-   [ ] MongoDB connects before server starts
-   [ ] `npm run dev` works
-   [ ] `https://localhost:3000` works
-   [ ] Postman HTTPS request works
-   [ ] JWT protected API tested over HTTPS

------------------------------------------------------------------------

# 🏆 Day 5 Security Architecture

``` text
                    ┌─────────────────┐
                    │     Client      │
                    │ Browser/Postman │
                    └────────┬────────┘
                             │
                         🔐 HTTPS
                             │
                             ▼
                  ┌─────────────────────┐
                  │    Node.js Server   │
                  │      Express        │
                  └──────────┬──────────┘
                             │
            ┌────────────────┼────────────────┐
            ▼                ▼                ▼
       🔑 JWT Auth       Google OAuth      Middleware
            │                │                │
            └────────────────┼────────────────┘
                             ▼
                         MongoDB
```

## 💡 Key Takeaway

``` text
bcrypt
  → protects stored passwords

OAuth
  → handles external authentication such as Google

JWT
  → authenticates API requests

HTTPS
  → protects data while it travels between client and server
```

Together, these form the core security pieces of your Day 5
authentication system.
