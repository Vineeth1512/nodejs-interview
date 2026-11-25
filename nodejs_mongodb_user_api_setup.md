# 🚀 Building a Node.js + Express + MongoDB User Authentication API

A step-by-step guide to creating a Node.js project, connecting MongoDB
Atlas, organizing the backend, and implementing **Signup, Login, Get All
Users, and Delete User** APIs.

> **Stack:** Node.js · Express.js · MongoDB Atlas · Mongoose · bcrypt ·
> dotenv

------------------------------------------------------------------------

## 📚 Contents

1.  [Install Node.js](#1-install-nodejs)
2.  [Create the project](#2-create-the-project)
3.  [Install dependencies](#3-install-dependencies)
4.  [Configure package.json](#4-configure-packagejson)
5.  [Create the project structure](#5-create-the-project-structure)
6.  [Configure environment
    variables](#6-configure-environment-variables)
7.  [Connect MongoDB Atlas and start the
    server](#7-connect-mongodb-atlas-and-start-the-server)
8.  [Create the User model](#8-create-the-user-model-entity)
9.  [Install bcrypt](#9-install-bcrypt-for-password-hashing)
10. [Create the User controller](#10-create-the-user-controller)
11. [Create the User routes](#11-create-the-user-routes)
12. [Test the API](#12-test-the-api)
13. [Important notes](#13-important-notes)

------------------------------------------------------------------------

## 1. Install Node.js

If Node.js is not installed on your computer:

1.  Visit [nodejs.org](https://nodejs.org/).
2.  Download and install the recommended **LTS** version.
3.  Open a terminal and verify the installation:

``` bash
node -v
npm -v
```

------------------------------------------------------------------------

## 2. Create the Project

Create a folder for your project and open it in VS Code.

``` bash
mkdir blood-user-api
cd blood-user-api
```

Initialize the Node.js project:

``` bash
npm init -y
```

This creates a `package.json` file containing project metadata and
scripts.

------------------------------------------------------------------------

## 3. Install Dependencies

Install the packages used in this project:

``` bash
npm install express mongoose cors dotenv bcrypt
```

Install Nodemon as a development dependency:

``` bash
npm install --save-dev nodemon
```

  Package      Purpose
  ------------ ---------------------------------------------------------
  `express`    Creates the backend server and API routes
  `mongoose`   Connects Node.js to MongoDB and provides schemas/models
  `cors`       Controls cross-origin requests
  `dotenv`     Loads environment variables from `.env`
  `bcrypt`     Hashes passwords securely
  `nodemon`    Restarts the server automatically during development

------------------------------------------------------------------------

## 4. Configure `package.json`

In `package.json`, update the `scripts` section. Keep the other
generated properties.

``` json
"scripts": {
  "start": "node index.js",
  "dev": "nodemon index.js"
}
```

Run the application in development mode:

``` bash
npm run dev
```

Run it normally:

``` bash
npm start
```

> Use `start` for the regular Node.js command and `dev` for Nodemon. The
> `test` script can be removed or replaced if you are not using tests
> yet.

------------------------------------------------------------------------

## 5. Create the Project Structure

Create the following folders and files:

``` text
blood-user-api/
│
├── models/
│   └── user.model.js
│
├── controller/
│   └── user.controller.js
│
├── routes/
│   └── user.route.js
│
├── .env
├── .gitignore
├── index.js
├── package.json
└── package-lock.json
```

**Folder responsibilities**

-   `models` (Entity): Defines the MongoDB document structure using a
    Mongoose schema.
-   `controller`: Contains the business logic for API requests.
-   `routes`: Defines API endpoints and connects them to controller
    functions.
-   `index.js`: Configures Express, connects to MongoDB, and starts the
    server.

------------------------------------------------------------------------

## 6. Configure Environment Variables

Create a file named **`.env`** in the project root.

``` env
PORT=3001
MONGODB_URI=mongodb+srv://<dbUser>:<dbPassword>@<cluster-address>/Blood?retryWrites=true&w=majority
```

Replace the placeholders with your MongoDB Atlas database user's
credentials and the connection address copied from Atlas.

> **Security:** Never publish your real database password or connection
> URI. Do not commit `.env` to GitHub.

Create a `.gitignore` file:

``` gitignore
node_modules/
.env
```

------------------------------------------------------------------------

## 7. Connect MongoDB Atlas and Start the Server

Create **`index.js`**:

``` javascript
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
require("dotenv").config();

const userRouter = require("./routes/user.route");

const app = express();

app.use(cors());
app.use(express.json());

app.get("/", (req, res) => {
  return res.status(200).json({
    message: "Welcome back to Node.js",
  });
});

app.use("/user", userRouter);

const PORT = process.env.PORT || 3001;
const MONGODB_URI = process.env.MONGODB_URI;

mongoose
  .connect(MONGODB_URI)
  .then(() => {
    console.log("MongoDB is connected");

    app.listen(PORT, () => {
      console.log(`Server is started on port ${PORT}`);
    });
  })
  .catch((error) => {
    console.error("MongoDB connection failed:", error.message);
  });
```

### What happens here?

1.  Import Express and Mongoose.
2.  Load values from `.env`.
3.  Create the Express application.
4.  Enable JSON request-body parsing and CORS.
5.  Register the user routes.
6.  Connect to MongoDB Atlas.
7.  Start the server only after the database connection succeeds.

------------------------------------------------------------------------

## 8. Create the User Model (Entity)

Create **`models/user.model.js`**:

``` javascript
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema(
  {
    userId: {
      type: String,
      required: true,
    },
    name: {
      type: String,
      required: true,
    },
    email: {
      type: String,
      required: true,
      unique: true,
    },
    password: {
      type: String,
      required: true,
    },
  },
  {
    timestamps: true,
  }
);

const User = mongoose.model("UserBlood", userSchema);

module.exports = User;
```

### Explanation

-   `mongoose.Schema()` defines the structure of a user document.
-   `required: true` makes a field mandatory.
-   `unique: true` creates a uniqueness constraint/index for the email
    field.
-   `timestamps: true` automatically adds `createdAt` and `updatedAt`.
-   `mongoose.model()` creates the model used to query the collection.

------------------------------------------------------------------------

## 9. Install bcrypt for Password Hashing

The package was installed in Step 3. If you need to install it
separately:

``` bash
npm install bcrypt
```

`bcrypt` hashes a password before it is stored in the database.

``` javascript
const hashedPassword = await bcrypt.hash(password, 10);
```

The number `10` is the salt-rounds value.

To check a password during login:

``` javascript
const isMatch = await bcrypt.compare(password, user.password);
```

> Never store plain-text passwords in MongoDB.

------------------------------------------------------------------------

## 10. Create the User Controller

Create **`controller/user.controller.js`**:

``` javascript
const User = require("../models/user.model");
const bcrypt = require("bcrypt");

// SIGNUP
module.exports.signup = async (req, res) => {
  try {
    const { email, password, name, userId } = req.body;

    if (!email || !password || !name || !userId) {
      return res.status(400).json({
        message: "All fields are required.",
      });
    }

    const existingUser = await User.findOne({ email });

    if (existingUser) {
      return res.status(409).json({
        message: "User already exists.",
      });
    }

    const hashedPassword = await bcrypt.hash(password, 10);

    const currentUser = new User({
      userId,
      name,
      email,
      password: hashedPassword,
    });

    await currentUser.save();

    return res.status(201).json({
      message: "User created successfully.",
      user: {
        id: currentUser._id,
        userId: currentUser.userId,
        name: currentUser.name,
        email: currentUser.email,
      },
    });
  } catch (err) {
    console.error(err);

    return res.status(500).json({
      message: "Internal Server Error",
    });
  }
};

// LOGIN
module.exports.login = async (req, res) => {
  try {
    const { email, password } = req.body;

    if (!email || !password) {
      return res.status(400).json({
        message: "Email and password are required.",
      });
    }

    const user = await User.findOne({ email });

    if (!user) {
      return res.status(401).json({
        message: "Invalid credentials.",
      });
    }

    const isMatchPassword = await bcrypt.compare(
      password,
      user.password
    );

    if (!isMatchPassword) {
      return res.status(401).json({
        message: "Invalid credentials.",
      });
    }

    return res.status(200).json({
      message: "User logged in successfully.",
      user: {
        id: user._id,
        userId: user.userId,
        name: user.name,
        email: user.email,
      },
    });
  } catch (err) {
    console.error(err);

    return res.status(500).json({
      message: "Internal Server Error",
    });
  }
};

// GET ALL USERS
module.exports.getAllUsers = async (req, res) => {
  try {
    const users = await User.find().select("-password");

    return res.status(200).json({
      message: "All users details.",
      users,
    });
  } catch (err) {
    console.error(err);

    return res.status(500).json({
      message: "Internal Server Error",
    });
  }
};

// DELETE USER
module.exports.deleteUser = async (req, res) => {
  try {
    const { id } = req.params;

    const deletedUser = await User.findByIdAndDelete(id);

    if (!deletedUser) {
      return res.status(404).json({
        message: "User not found.",
      });
    }

    return res.status(200).json({
      message: "User deleted successfully.",
    });
  } catch (err) {
    console.error(err);

    return res.status(500).json({
      message: "Internal Server Error",
    });
  }
};
```

### Controller methods

  -----------------------------------------------------------------------
  Method                              Purpose
  ----------------------------------- -----------------------------------
  `signup`                            Validates details, checks existing
                                      email, hashes password, and saves
                                      user

  `login`                             Finds user and compares password
                                      using bcrypt

  `getAllUsers`                       Retrieves all users without
                                      exposing password hashes

  `deleteUser`                        Deletes a user using MongoDB
                                      document ID
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 11. Create the User Routes

Create **`routes/user.route.js`**:

``` javascript
const express = require("express");
const router = express.Router();

const userController = require("../controller/user.controller");

router.post("/signup", userController.signup);
router.post("/login", userController.login);
router.get("/", userController.getAllUsers);
router.delete("/:id", userController.deleteUser);

module.exports = router;
```

Because the router is mounted at `/user` in `index.js`, the complete
endpoints are:

  Method   Endpoint         Description
  -------- ---------------- ---------------------
  POST     `/user/signup`   Register a user
  POST     `/user/login`    Login a user
  GET      `/user`          Get all users
  DELETE   `/user/:id`      Delete a user by ID

------------------------------------------------------------------------

## 12. Test the API

Start your server:

``` bash
npm run dev
```

You should see messages similar to:

``` text
MongoDB is connected
Server is started on port 3001
```

Use Postman or another API client to test the endpoints.

### Signup

**POST** `http://localhost:3001/user/signup`

``` json
{
  "userId": "USER001",
  "name": "Vineeth",
  "email": "vineeth@example.com",
  "password": "MyStrongPassword123"
}
```

### Login

**POST** `http://localhost:3001/user/login`

``` json
{
  "email": "vineeth@example.com",
  "password": "MyStrongPassword123"
}
```

### Get All Users

**GET** `http://localhost:3001/user`

### Delete User

**DELETE** `http://localhost:3001/user/<mongodb-document-id>`

Replace the placeholder with the user's actual MongoDB `_id`.

------------------------------------------------------------------------

## 13. Important Notes

-   Use `&&` or proper logical operators when validating multiple
    fields. In JavaScript, `!email || !password` checks whether either
    value is missing.
-   Always use `await` for database operations when you need their
    result. For example, `await User.findByIdAndDelete(id)`.
-   Do not return password hashes in API responses.
-   A successful signup should generally return HTTP `201 Created`.
-   A duplicate email can return HTTP `409 Conflict`.
-   The `userId` field is required in this example. If you do not need a
    custom user ID, remove it from the schema and signup validation;
    MongoDB already creates `_id`.
-   For a production application, protect user routes with
    authentication and authorization. Password hashing alone does not
    provide access control.

------------------------------------------------------------------------

## 🎯 Backend Request Flow

``` text
Client / Postman
      ↓
Express Route
      ↓
Controller
      ↓
Mongoose Model
      ↓
MongoDB Atlas
      ↓
Controller Response
      ↓
Client
```

**You have now created a basic Node.js + Express + MongoDB user API with
a structured folder architecture.**
