# 🚀 Deploy a MERN Stack Application

A practical, step-by-step guide to deploying a **MongoDB + Express.js +
React + Node.js** application using:

-   **MongoDB Atlas** --- Cloud database
-   **Render** --- Backend hosting (Node.js + Express)
-   **Vercel** --- Frontend hosting (React)
-   **GitHub** --- Source code repository

------------------------------------------------------------------------

## 🗺️ Deployment Architecture

``` text
User's Browser
      |
      v
React Frontend (Vercel)
      |
      | HTTPS API requests
      v
Node.js + Express Backend (Render)
      |
      | MongoDB connection URI
      v
MongoDB Atlas (Cloud Database)
```

## ✅ Prerequisites

-   A working MERN application (frontend and backend)
-   MongoDB Atlas, Render, and Vercel accounts
-   Source code pushed to GitHub
-   Application running successfully locally

------------------------------------------------------------------------

# 1. Deploy the Database on MongoDB Atlas

## Step 1: Create a Project and Cluster

1.  Open [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) and sign
    in.
2.  Create a project or select an existing one.
3.  Choose **Build a Database**.
4.  Select an available shared/free deployment option if suitable.
5.  Choose a cloud provider and region.
6.  Create the cluster and wait until it is ready.

## Step 2: Configure Database Security

### Add a database user

1.  Open **Database Access**.
2.  Select **Add New Database User**.
3.  Create a username and strong password.
4.  Assign only the database permissions your application needs.

### Configure network access

1.  Open **Network Access**.
2.  Add the IP address that needs to connect to Atlas.

> `0.0.0.0/0` allows connections from any IP. It may be used when
> necessary for cloud hosting, but increases exposure. Prefer a narrower
> rule when possible, and always use a strong password and
> least-privilege database user.

## Step 3: Copy the MongoDB Connection URI

1.  Open your cluster and click **Connect**.
2.  Choose **Drivers** / **Connect your application**.
3.  Copy the connection string.

``` text
mongodb+srv://<username>:<password>@<cluster-address>/<database-name>?retryWrites=true&w=majority
```

Replace the placeholders with your database user's credentials and
database name.

> Do not place the real URI in frontend code or commit it to GitHub.

------------------------------------------------------------------------

# 2. Configure MongoDB in the Node.js Backend

In your backend connection file (`server.js`, `app.js`, or `index.js`),
use an environment variable:

``` javascript
const mongoose = require("mongoose");

mongoose
  .connect(process.env.MONGODB_URI)
  .then(() => {
    console.log("MongoDB Atlas connected");
  })
  .catch((error) => {
    console.error("MongoDB connection error:", error.message);
  });
```

Install dotenv if needed:

``` bash
npm install dotenv
```

Local backend `.env`:

``` env
MONGODB_URI=mongodb+srv://<username>:<password>@<cluster-address>/<database-name>
PORT=5000
```

Load it near the top of your entry file:

``` javascript
require("dotenv").config();
```

Add `.env` to `.gitignore`:

``` gitignore
node_modules/
.env
```

------------------------------------------------------------------------

# 3. Deploy the Backend on Render

## Step 1: Push Backend Code to GitHub

1.  Confirm the backend works locally.
2.  Commit and push the backend code to GitHub.
3.  Keep `.env` out of the repository.

## Step 2: Create a Render Web Service

1.  Visit [Render](https://render.com/) and sign in.
2.  Choose **New → Web Service**.
3.  Connect your GitHub account.
4.  Select the backend repository.
5.  Configure the service:

  Setting         Example
  --------------- ------------------------------------------------------
  Name            `my-mern-backend`
  Runtime         Node
  Region          Choose a region suitable for your users and database
  Build Command   `npm install`
  Start Command   `npm start` or `node server.js`
  Instance Type   Choose an available plan

> Use the correct start command. If your entry file is `index.js`, use
> `node index.js`, or add `"start": "node index.js"` to `package.json`
> and use `npm start`.

## Step 3: Add Backend Environment Variables

In your Render service, open **Environment** and add:

``` env
MONGODB_URI=your_atlas_connection_uri
FRONTEND_URL=https://your-frontend.vercel.app
```

Render provides a `PORT` environment variable. Your server should use
it:

``` javascript
const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

Save the variables and deploy.

## Step 4: Verify the Backend

Render provides a service URL similar to:

``` text
https://your-mern-backend.onrender.com
```

Open your root or health endpoint, or test an API endpoint using
Postman.

------------------------------------------------------------------------

# 4. Configure CORS in Express

The frontend and backend have different domains, so configure CORS.

Install:

``` bash
npm install cors
```

Recommended configuration:

``` javascript
const cors = require("cors");

const allowedOrigins = [
  "http://localhost:3000",
  "http://localhost:5173",
  process.env.FRONTEND_URL,
].filter(Boolean);

app.use(
  cors({
    origin: allowedOrigins,
    methods: ["GET", "POST", "PUT", "PATCH", "DELETE"],
    credentials: true,
  })
);
```

Set the production frontend URL in Render:

``` env
FRONTEND_URL=https://your-frontend.vercel.app
```

> If using cookie-based authentication, configure credentials on both
> client and server and set suitable cookie options such as `SameSite`
> and `Secure`.

------------------------------------------------------------------------

# 5. Connect React Frontend to the Render Backend

Replace local API URLs such as:

``` javascript
http://localhost:5000/api
```

with the deployed Render backend URL.

## Use a Vite Environment Variable

For a Vite React project, create a frontend `.env` file:

``` env
VITE_API_URL=http://localhost:5000
```

Use it in your API code:

``` javascript
const API_URL = import.meta.env.VITE_API_URL;

fetch(`${API_URL}/api/users`);
```

For production, configure `VITE_API_URL` in Vercel.

> Vite exposes variables prefixed with `VITE_` to frontend code. Never
> store database passwords or private server secrets in frontend
> environment variables.

Push the updated frontend code to GitHub.

------------------------------------------------------------------------

# 6. Deploy the Frontend on Vercel

## Step 1: Import Your GitHub Repository

1.  Visit [Vercel](https://vercel.com/) and sign in.
2.  Select **Add New → Project**.
3.  Import your frontend repository from GitHub.
4.  Configure the project.

## Step 2: Configure Build Settings

For a typical Vite React project:

  Setting            Value
  ------------------ -----------------
  Framework Preset   Vite
  Build Command      `npm run build`
  Output Directory   `dist`
  Install Command    `npm install`

For Create React App, the output directory is usually `build`.

If frontend and backend are in one monorepo, set the **Root Directory**
to the frontend folder.

## Step 3: Add Frontend Environment Variable

In Vercel project settings, add:

``` env
VITE_API_URL=https://your-mern-backend.onrender.com
```

Use your actual Render backend URL.

## Step 4: Deploy

1.  Click **Deploy**.
2.  Wait for the build to finish.
3.  Vercel provides a frontend URL, for example:

``` text
https://your-mern-frontend.vercel.app
```

------------------------------------------------------------------------

# 7. Final Deployment Verification

-   [ ] MongoDB Atlas cluster is running.
-   [ ] Database user and network access are configured.
-   [ ] Render environment variables are set.
-   [ ] Backend deployment succeeds.
-   [ ] Backend API responds at its deployed URL.
-   [ ] CORS allows the Vercel frontend domain.
-   [ ] Vercel has the correct `VITE_API_URL`.
-   [ ] Frontend deployment succeeds.
-   [ ] Signup, login, CRUD, and other API features work.
-   [ ] Browser console and Render logs show no errors.

## 🧰 Common Deployment Problems

  -----------------------------------------------------------------------
  Issue                               What to check
  ----------------------------------- -----------------------------------
  MongoDB connection error            Verify `MONGODB_URI`, database
                                      credentials, and Atlas Network
                                      Access.

  Render deployment fails             Check build/start commands and
                                      deployment logs.

  `Cannot GET /`                      Confirm a root route exists, or
                                      test the correct API endpoint.

  CORS error                          Include the exact Vercel domain in
                                      backend CORS configuration.

  Frontend API returns 404            Check the API base URL and route
                                      path.

  Environment variable is undefined   Check its name, prefix, and
                                      hosting-provider settings; redeploy
                                      after changes.

  React routes show 404 after refresh Configure Vercel SPA rewrites if
                                      your app uses client-side routing.

  Render service responds slowly      Some plans may spin down services;
  after inactivity                    check your plan's behavior.
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 🎉 Deployment Complete

Your MERN application is deployed with:

-   **MongoDB Atlas** --- Cloud database
-   **Render** --- Express and Node.js backend
-   **Vercel** --- React frontend

Keep secrets on the backend, use HTTPS in production, and test every API
flow after deployment.
