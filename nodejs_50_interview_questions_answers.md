# Node.js Interview Questions and Answers

**50 questions from the supplied Node.js interview-question image**

These answers are written in simple English for interview preparation.
Each answer includes the main idea and, where useful, a practical
example or important point.

------------------------------------------------------------------------

## 1. What is Node.js and why is it used?

Node.js is an open-source, cross-platform JavaScript runtime
environment. It allows JavaScript to run outside the browser, especially
on the server.

It is commonly used to build REST APIs, backend services, real-time
applications, command-line tools, and applications that perform many I/O
operations.

**Why use it?** - Developers can use JavaScript on both frontend and
backend. - It supports asynchronous, non-blocking I/O. - It has a large
package ecosystem through npm. - It is useful for applications that
handle many concurrent connections.

**Interview answer:** Node.js is a JavaScript runtime built on Google's
V8 engine. It lets us execute JavaScript on the server and build backend
applications such as APIs. Its event-driven, non-blocking I/O model
makes it suitable for I/O-intensive applications.

## 2. How does Node.js work internally?

Node.js runs JavaScript using the V8 engine. The main JavaScript
execution normally happens on one main thread. Node.js uses libuv to
provide the event loop and asynchronous I/O support. Depending on the
operation, work can be handled by the operating system or libuv's worker
thread pool.

When an asynchronous operation completes, its callback is scheduled. The
event loop runs eligible callbacks when the JavaScript call stack is
available.

**Main components:** - **V8:** Executes JavaScript. - **libuv:**
Provides the event loop, asynchronous I/O support, and a worker thread
pool. - **Event loop:** Coordinates when callbacks are executed. -
**Worker thread pool:** Handles certain tasks, such as some filesystem,
crypto, and DNS operations.

**Important:** Node.js is not simply "one thread for everything."
JavaScript execution is normally single-threaded per isolate, while some
work is handled by the OS or background threads.

## 3. What is the V8 JavaScript engine?

V8 is Google's open-source JavaScript and WebAssembly engine. It is
written mainly in C++ and is used by Chrome and Node.js to execute
JavaScript.

V8 parses and compiles JavaScript, uses interpretation and just-in-time
compilation techniques, and executes the resulting machine code.

**Example:** When we run `node app.js`, Node.js uses V8 to execute the
JavaScript in `app.js`.

**Interview answer:** V8 is the JavaScript engine that executes
JavaScript code in Node.js. It helps convert JavaScript into machine
code so the computer can run it efficiently.

## 4. Why is Node.js single-threaded?

Node.js uses a single main JavaScript thread by default to execute
JavaScript callbacks. This simplifies application logic and avoids many
shared-memory synchronization problems.

However, Node.js can still handle many concurrent I/O operations because
it uses asynchronous APIs, the operating system, and libuv's thread
pool. It also supports Worker Threads for CPU-intensive JavaScript work.

**Interview answer:** Node.js uses a single main thread for JavaScript
execution, but it does not perform every operation on that thread.
Asynchronous I/O and background workers allow it to handle concurrent
work without blocking JavaScript execution.

## 5. What is the Event Loop?

The Event Loop is the mechanism that coordinates the execution of
JavaScript callbacks and asynchronous operations in Node.js.

When synchronous JavaScript is running, it uses the call stack.
Asynchronous work can be handled outside the main JavaScript execution.
When callbacks become eligible, the event loop processes them in its
phases.

**Example:**

``` js
console.log("Start");

setTimeout(() => {
  console.log("Timer");
}, 0);

console.log("End");
```

Typical output:

``` text
Start
End
Timer
```

The synchronous statements run first. The timer callback runs later when
the event loop reaches the appropriate phase.

## 6. What are Node.js global objects?

Global objects are values and functions that are available throughout a
Node.js application without importing them explicitly.

Common examples include: - `globalThis` -- standard way to access the
global object. - `process` -- information and control for the current
Node.js process. - `Buffer` -- handles binary data. - `console` --
logging utilities. - `setTimeout()` and `setInterval()` -- schedule
timer callbacks. - `URL` and `URLSearchParams` -- work with URLs.

Some names, such as `__dirname` and `__filename`, are available in
CommonJS modules but are not universal globals in the same way.

## 7. CommonJS vs ES Modules?

Both are JavaScript module systems.

  -----------------------------------------------------------------------
  CommonJS                            ES Modules
  ----------------------------------- -----------------------------------
  Uses `require()` to load modules    Uses `import` to load modules

  Uses `module.exports` or `exports`  Uses `export` and `export default`

  Traditionally common in Node.js     Standard JavaScript module system

  Commonly loaded synchronously       Supports static imports and
                                      asynchronous module loading
  -----------------------------------------------------------------------

**CommonJS example:**

``` js
const fs = require("node:fs");
module.exports = myFunction;
```

**ES Modules example:**

``` js
import fs from "node:fs";
export default myFunction;
```

In Node.js, ES module interpretation can be configured using `.mjs`
files or `"type": "module"` in `package.json`.

## 8. What is package.json?

`package.json` is the main metadata and configuration file for a Node.js
project.

It commonly contains: - Project name and version. - Entry points and
scripts. - Dependencies and development dependencies. - Package
configuration, such as module type. - Other metadata such as description
and license.

**Example:**

``` json
{
  "name": "my-api",
  "version": "1.0.0",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "express": "^5.0.0"
  }
}
```

Run a script with `npm run dev`, or run the start script with
`npm start`.

## 9. Dependencies vs devDependencies?

Both list packages required by a project, but they have different
purposes.

-   **dependencies:** Packages needed by the application at runtime,
    such as Express or a database driver.
-   **devDependencies:** Packages mainly needed during development,
    testing, linting, or building, such as Nodemon, ESLint, or test
    tools.

Install a runtime dependency using `npm install package-name`.

Install a development dependency using `npm install -D package-name`.

**Interview answer:** Dependencies are required when the application
runs, while devDependencies are tools used to develop, test, or build
the application.

## 10. What is the difference between npm and npx?

-   **npm** is the Node Package Manager. It installs packages, manages
    dependencies, and runs package scripts.
-   **npx** runs a package executable. It can run a locally installed
    tool or, when appropriate, fetch and execute a package without
    requiring a global installation.

**Examples:**

``` bash
npm install express
npx create-vite@latest
```

Use npm to manage packages and npx when you want to execute a package
command.

## 11. What is callback hell?

Callback hell is code that becomes deeply nested when one asynchronous
operation depends on the result of another.

It can make code difficult to read, debug, and maintain.

**Example of nested callbacks:**

``` js
getUser(id, (user) => {
  getOrders(user.id, (orders) => {
    getPayment(orders[0].id, (payment) => {
      console.log(payment);
    });
  });
});
```

**How to avoid it:** - Use named functions to reduce nesting. - Use
Promises and chain `.then()`. - Use `async`/`await`. - Handle errors
consistently.

## 12. Callbacks vs Promises vs async/await?

These are different ways to handle asynchronous operations.

  -----------------------------------------------------------------------
  Approach                            Explanation
  ----------------------------------- -----------------------------------
  Callback                            A function passed to another
                                      function to be called later

  Promise                             An object representing the eventual
                                      success or failure of an
                                      asynchronous operation

  async/await                         Syntax built on Promises that makes
                                      asynchronous code easier to read
  -----------------------------------------------------------------------

**Callback:**

``` js
readFile("notes.txt", (err, data) => {
  if (err) return console.error(err);
  console.log(data.toString());
});
```

**Promise with async/await:**

``` js
async function loadData() {
  try {
    const data = await readFilePromise("notes.txt");
    console.log(data);
  } catch (error) {
    console.error(error);
  }
}
```

**Interview answer:** Callbacks use functions to handle completion,
Promises represent future results, and async/await provides a cleaner
syntax for working with Promises.

## 13. How does async/await work internally?

An `async` function always returns a Promise. The `await` keyword pauses
that async function's progress until the awaited Promise settles. It
does not block the entire JavaScript thread.

When the awaited Promise settles, the remaining part of the async
function is scheduled to continue through Promise job processing.

**Example:**

``` js
async function getUser() {
  console.log("First");
  const user = await Promise.resolve("Vineeth");
  console.log(user);
}

getUser();
console.log("Outside");
```

Output:

``` text
First
Outside
Vineeth
```

The code after `await` continues asynchronously. Use `try/catch` to
handle rejected Promises.

## 14. Promise.all() vs Promise.allSettled()?

Both methods accept an iterable of Promises and wait for the input
operations to settle.

-   **`Promise.all()`** fulfills when all input Promises fulfill. It
    rejects as soon as one input rejects.
-   **`Promise.allSettled()`** waits until every input Promise settles
    and returns each outcome, whether fulfilled or rejected.

**Example:**

``` js
const results = await Promise.all([
  fetchUser(),
  fetchProducts()
]);
```

Use `Promise.all()` when every result is required.

``` js
const results = await Promise.allSettled([
  sendEmail(),
  updateAnalytics()
]);
```

Use `Promise.allSettled()` when you want to know the result of every
operation even if some fail.

## 15. Promise.race() vs Promise.any()?

-   **`Promise.race()`** settles with the outcome of the first input
    Promise to settle. That outcome may be fulfilled or rejected.
-   **`Promise.any()`** fulfills with the first fulfilled input Promise.
    It rejects with an `AggregateError` only if all inputs reject.

**Example:**

``` js
const fastest = await Promise.race([
  fetchFromServerA(),
  fetchFromServerB()
]);
```

Use `race()` when the first settled result matters, such as implementing
a timeout race.

Use `any()` when you need the first successful result from multiple
alternatives.

## 16. What is Event Loop execution order?

Node.js processes callbacks through event-loop phases. The commonly
described phases are:

1.  **Timers:** Executes eligible `setTimeout()` and `setInterval()`
    callbacks.
2.  **Pending callbacks:** Executes certain system-level callbacks
    deferred from earlier operations.
3.  **Idle/prepare:** Internal use by Node.js and libuv.
4.  **Poll:** Retrieves and processes I/O events and callbacks.
5.  **Check:** Executes `setImmediate()` callbacks.
6.  **Close callbacks:** Executes certain close-event callbacks, such as
    socket close handlers.

Promise callbacks and `queueMicrotask()` use the microtask queue.
Node.js also processes the `process.nextTick()` queue with special
priority.

**Important:** Do not assume one universal order for every combination
of timers, I/O, `setImmediate()`, Promises, and `process.nextTick()`.
The observed order can depend on where the code runs and the Node.js
version.

## 17. process.nextTick() vs setImmediate()?

Both schedule a callback, but they run at different points.

-   **`process.nextTick()`** schedules a callback to run after the
    current JavaScript operation completes, before the event loop
    continues to its next phase.
-   **`setImmediate()`** schedules a callback for the event loop's check
    phase.

**Example:**

``` js
console.log("Start");

process.nextTick(() => console.log("nextTick"));
setImmediate(() => console.log("setImmediate"));

console.log("End");
```

A typical output is:

``` text
Start
End
nextTick
setImmediate
```

Avoid recursively scheduling too much `process.nextTick()` work because
it can prevent the event loop from progressing.

## 18. What are microtasks and macrotasks?

These terms describe different categories of scheduled work.

-   **Microtasks:** Promise `.then()`, `.catch()`, `.finally()`
    callbacks and `queueMicrotask()`. Node.js also has its special
    `process.nextTick()` queue.
-   **Macrotasks (often called tasks):** Timer callbacks, many I/O
    callbacks, and `setImmediate()` callbacks, which are processed
    through event-loop phases.

In Node.js, `process.nextTick()` has special handling and should not
simply be treated as an ordinary Promise microtask.

**Example:**

``` js
console.log("Start");

setTimeout(() => console.log("Timer"), 0);
Promise.resolve().then(() => console.log("Promise"));

console.log("End");
```

Typical output:

``` text
Start
End
Promise
Timer
```

## 19. How do you handle errors in asynchronous code?

The correct approach depends on the asynchronous API.

**With async/await:**

``` js
async function getData() {
  try {
    const result = await fetchData();
    return result;
  } catch (error) {
    console.error("Failed to fetch data:", error);
    throw error;
  }
}
```

**With Promises:**

``` js
fetchData()
  .then(result => console.log(result))
  .catch(error => console.error(error));
```

**With error-first callbacks:**

``` js
readFile("data.txt", (error, data) => {
  if (error) {
    return console.error(error);
  }
  console.log(data.toString());
});
```

In Express applications, pass errors to centralized error-handling
middleware. Do not silently ignore errors or expose internal stack
traces to clients in production.

## 20. What causes Event Loop blocking?

The event loop is blocked when long-running synchronous JavaScript
prevents the main thread from processing other callbacks.

**Common causes:** - Large CPU-intensive loops. - Complex synchronous
calculations. - Synchronous filesystem operations. - Large JSON parsing
or serialization. - Inefficient regular expressions that take a long
time.

**How to avoid it:** - Prefer asynchronous I/O APIs. - Break large work
into smaller chunks when appropriate. - Move CPU-intensive work to
Worker Threads or separate services. - Profile and optimize slow code.

**Example of blocking code:**

``` js
const fs = require("node:fs");
const data = fs.readFileSync("large-file.txt");
```

`readFileSync()` blocks JavaScript execution until the file read
finishes.

------------------------------------------------------------------------

## Express.js, Middleware and REST APIs

## 21. What is Express.js?

Express.js is a lightweight web framework for Node.js. It provides
features for building web servers and REST APIs, including routing,
middleware, and request/response handling.

**Example:**

``` js
const express = require("express");
const app = express();

app.use(express.json());

app.get("/users", (req, res) => {
  res.json({ message: "Users list" });
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

Express reduces the amount of low-level HTTP handling code developers
need to write.

## 22. How does middleware work?

Middleware is a function that runs during the request-response cycle. It
can access the request object (`req`), response object (`res`), and the
`next` function.

Middleware can: - Execute code. - Read or modify request and response
objects. - End the response. - Pass control to the next middleware. -
Pass an error to error-handling middleware using `next(error)`.

**Example:**

``` js
function logger(req, res, next) {
  console.log(req.method, req.url);
  next();
}

app.use(logger);
```

Middleware runs in the order in which it is registered.

## 23. Application-level vs router-level middleware?

-   **Application-level middleware** is attached to the Express
    application using `app.use()` or methods such as `app.get()`.
-   **Router-level middleware** is attached to an Express Router
    instance using `router.use()` or router route methods.

**Application-level example:**

``` js
app.use(express.json());
```

**Router-level example:**

``` js
const router = require("express").Router();

router.use((req, res, next) => {
  console.log("User routes");
  next();
});

router.get("/", (req, res) => res.send("Users"));
app.use("/users", router);
```

Use application-level middleware for application-wide behavior and
router-level middleware for behavior limited to a group of routes.

## 24. What is next() in Express?

`next()` is a function that passes control to the next middleware or
route handler in the current request-processing chain.

**Example:**

``` js
function checkLogin(req, res, next) {
  if (req.headers.authorization) {
    return next();
  }

  res.status(401).json({ message: "Unauthorized" });
}
```

Calling `next()` continues processing. Sending a response ends the
request. Calling `next(error)` passes an error to error-handling
middleware.

## 25. How do you implement centralized error handling?

Centralized error handling means passing application errors to one
error-handling middleware instead of writing response logic repeatedly
in every route.

Express error-handling middleware has four parameters:
`(err, req, res, next)`.

**Example:**

``` js
app.get("/example", async (req, res, next) => {
  try {
    const result = await someOperation();
    res.json(result);
  } catch (error) {
    next(error);
  }
});

app.use((err, req, res, next) => {
  console.error(err);

  res.status(err.statusCode || 500).json({
    message: err.statusCode ? err.message : "Internal server error"
  });
});
```

Register the error handler after routes. In production, log internal
details securely but do not send stack traces or secrets to clients.
Express 5 automatically forwards rejected Promises from async route
handlers; in Express 4, use `try/catch` or an async wrapper.

## 26. How do you validate incoming API requests?

Request validation checks whether incoming data has the correct fields,
types, formats, and allowed values before business logic uses it.

**Common validation checks:** - Required fields. - Email and phone
formats. - String length. - Number ranges. - Allowed enum values. -
Query parameters and route parameters.

Libraries such as Zod, Joi, and express-validator can help.

**Example using express-validator:**

``` js
const { body, validationResult } = require("express-validator");

app.post(
  "/users",
  body("email").isEmail(),
  body("name").trim().notEmpty(),
  (req, res) => {
    const errors = validationResult(req);

    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }

    res.status(201).json({ message: "Valid request" });
  }
);
```

Validate on the server even if the frontend also validates the form.

## 27. How do you implement API versioning?

API versioning allows an API to evolve while existing clients continue
using a supported version.

Common approaches: - URL path: `/api/v1/users` - Query parameter:
`/api/users?version=1` - Custom request header. - Media type versioning
through the `Accept` header.

**Example:**

``` js
app.use("/api/v1/users", userRoutesV1);
app.use("/api/v2/users", userRoutesV2);
```

URL versioning is easy to understand and commonly used. Document changes
and provide a migration or deprecation plan for older versions.

## 28. How do you handle CORS?

CORS stands for Cross-Origin Resource Sharing. It is a browser security
mechanism that controls whether a web page from one origin can access
resources from another origin.

In Express, the `cors` package can configure the allowed origins,
methods, and headers.

**Example:**

``` js
const cors = require("cors");

app.use(cors({
  origin: "https://myfrontend.com",
  methods: ["GET", "POST", "PUT", "DELETE"],
  credentials: true
}));
```

For credentialed requests, specify trusted origins rather than using
`*`. CORS is not authentication; APIs must still authenticate and
authorize requests.

## 29. How do you implement pagination and filtering?

Pagination returns a limited portion of a large dataset. Filtering
returns only records that match given conditions.

**Offset pagination example:**

``` js
const page = Math.max(Number(req.query.page) || 1, 1);
const limit = Math.min(Math.max(Number(req.query.limit) || 10, 1), 100);
const skip = (page - 1) * limit;

const products = await Product.find({ category: req.query.category })
  .skip(skip)
  .limit(limit);
```

A response can include the data and pagination metadata, such as current
page, page size, and total count.

For very large or frequently changing datasets, cursor-based pagination
can be more efficient and stable than large offsets. Validate and cap
user-provided limits.

## 30. How do you design scalable REST APIs?

A scalable REST API should be predictable, secure, maintainable, and
efficient under increasing traffic.

**Good practices:** - Use resource-based URLs, such as
`/api/products`. - Use HTTP methods correctly: GET, POST, PUT, PATCH,
DELETE. - Return suitable HTTP status codes. - Validate input and handle
errors consistently. - Add authentication and authorization. - Implement
pagination, filtering, and sorting. - Use database indexes and efficient
queries. - Add caching where appropriate. - Version APIs when breaking
changes are needed. - Log, monitor, document, and test the API.

**Example endpoints:**

``` text
GET    /api/products
GET    /api/products/:id
POST   /api/products
PATCH  /api/products/:id
DELETE /api/products/:id
```

------------------------------------------------------------------------

## Streams, Buffers and Modules

## 31. What are streams in Node.js?

Streams are an interface for processing data piece by piece instead of
loading all of it into memory at once.

They are useful for large files, network communication, and processing
continuous data.

**Example:**

``` js
const fs = require("node:fs");

const readable = fs.createReadStream("large-file.txt");

readable.on("data", chunk => {
  console.log("Received a chunk:", chunk.length);
});
```

Streams can reduce memory usage and allow processing to begin before all
data has arrived.

## 32. Readable vs Writable vs Duplex streams?

Node.js has four main stream types:

  --------------------------------------------------------------------------
  Stream type             Purpose                 Example
  ----------------------- ----------------------- --------------------------
  Readable                Provides data to be     `fs.createReadStream()`
                          read                    

  Writable                Accepts data to be      `fs.createWriteStream()`
                          written                 

  Duplex                  Reads and writes data   TCP socket
                          independently           

  Transform               Duplex stream that      Gzip compression stream
                          transforms data         
  --------------------------------------------------------------------------

**Example:**

``` js
const fs = require("node:fs");

const readStream = fs.createReadStream("input.txt");
const writeStream = fs.createWriteStream("output.txt");

readStream.pipe(writeStream);
```

`pipe()` connects a readable stream to a writable stream and helps
manage flow control.

## 33. What are Buffers?

A Buffer is a Node.js object used to work with raw binary data. It
represents a sequence of bytes.

Buffers are useful when handling files, network packets, images, audio,
and other binary content.

**Example:**

``` js
const buffer = Buffer.from("Hello");

console.log(buffer);
console.log(buffer.toString());
console.log(buffer.length);
```

A Buffer is not the same as a normal JavaScript string. It stores bytes
and can convert them to text using an encoding.

## 34. What is stream backpressure?

Backpressure happens when data is produced faster than the destination
can process it.

For example, a readable stream may produce chunks faster than a writable
stream can write them. Without flow control, data can build up in
memory.

Node.js streams help manage backpressure. When `writable.write(chunk)`
returns `false`, the producer should wait for the writable stream's
`drain` event before continuing.

**Example:**

``` js
if (!writable.write(chunk)) {
  readable.pause();

  writable.once("drain", () => {
    readable.resume();
  });
}
```

Using `pipe()` or `pipeline()` is often easier because Node.js manages
flow control for you.

## 35. require() vs import?

`require()` belongs to CommonJS, while `import` is part of the ES
Modules system.

**CommonJS:**

``` js
const express = require("express");
```

**ES Modules:**

``` js
import express from "express";
```

Important differences: - CommonJS uses `module.exports` and `exports`. -
ES Modules use `export` and `export default`. - ES Modules support
static imports and exports. - Node.js determines the module format using
extensions and package configuration.

Choose a module system consistently for a project, and understand how
any dependencies support it.

## 36. How does module caching work?

When a CommonJS module is loaded with `require()`, Node.js caches the
module after its first load. Requiring the same resolved module again
normally returns the cached exports rather than executing the module
file again.

**Example:**

``` js
const first = require("./config");
const second = require("./config");

console.log(first === second); // normally true
```

This can avoid repeated module initialization. Be careful with mutable
exported objects because modules that share the cached object can also
share its state.

CommonJS `require.cache` can be inspected or modified. ES Modules use a
separate module-loading and caching mechanism.

------------------------------------------------------------------------

## Events and Multithreading

## 37. What is EventEmitter?

`EventEmitter` is a Node.js class that allows objects to emit named
events and register listener functions for those events.

It is useful for communication between parts of an application without
directly calling each other.

**Example:**

``` js
const EventEmitter = require("node:events");

const emitter = new EventEmitter();

emitter.on("userRegistered", (name) => {
  console.log(`${name} registered`);
});

emitter.emit("userRegistered", "Vineeth");
```

`on()` registers a listener, and `emit()` triggers the event. Remember
to handle the special `error` event appropriately.

## 38. What are Worker Threads?

Worker Threads allow JavaScript to run in parallel on separate threads
within a Node.js process.

They are especially useful for CPU-intensive tasks that would otherwise
block the main JavaScript thread.

**Use cases:** - Large calculations. - Image processing. - Data
transformation. - CPU-intensive parsing.

Worker Threads are generally not needed for ordinary asynchronous I/O,
because Node.js already handles I/O asynchronously.

The `node:worker_threads` module provides the Worker API and message
passing between threads.

## 39. Cluster vs Worker Threads?

Both can use multiple CPU cores, but they work differently.

  -----------------------------------------------------------------------
  Cluster                             Worker Threads
  ----------------------------------- -----------------------------------
  Creates multiple Node.js processes  Creates multiple threads

  Each worker has its own process and Workers can share memory using
  memory                              SharedArrayBuffer

  Useful for scaling network servers  Useful for CPU-intensive JavaScript
  across CPU cores                    tasks

  Communicates using process          Communicates using messages and
  messaging                           shared-memory tools
  -----------------------------------------------------------------------

**Interview answer:** Cluster is commonly used to run multiple Node.js
server processes that can share a server port. Worker Threads run
JavaScript in parallel threads and are useful for CPU-heavy work.

## 40. How do you identify and fix memory leaks?

A memory leak occurs when an application keeps references to objects
that are no longer needed, preventing garbage collection from reclaiming
their memory.

**Common causes:** - Global variables that keep growing. - Event
listeners that are never removed. - Timers that are not cleared. -
Unbounded caches. - Closures that retain large objects. - References
kept in long-lived collections.

**How to identify leaks:** - Monitor memory usage with
`process.memoryUsage()`. - Take and compare heap snapshots using Chrome
DevTools or Node.js inspector. - Use profiling and monitoring tools. -
Reproduce the issue under load and check whether memory keeps
increasing.

**How to fix them:** - Remove unused event listeners. - Clear timers
when they are no longer needed. - Set limits or expiration policies for
caches. - Release unnecessary references. - Fix the underlying retention
issue rather than relying on forced garbage collection.

------------------------------------------------------------------------

## Authentication, Security and Performance

## 41. Authentication vs authorization?

These are two different security concepts.

-   **Authentication:** Verifies who the user is.
-   **Authorization:** Determines what an authenticated user is allowed
    to do.

**Example:**

A user logs in using email and password. This is authentication.

After login, the application checks whether the user has an admin role
before allowing access to the admin dashboard. This is authorization.

**Interview answer:** Authentication confirms identity, while
authorization checks permissions.

## 42. How does JWT authentication work?

JWT stands for JSON Web Token. It is a compact token format often used
to carry signed claims between a client and a server.

**Typical login flow:** 1. The user sends email and password to the
login API. 2. The backend verifies the credentials. 3. The backend
creates a signed JWT containing claims such as the user ID and expiry
time. 4. The client stores the token using an appropriate security
strategy. 5. The client sends the token with later requests, commonly in
the `Authorization: Bearer <token>` header. 6. The backend verifies the
token's signature and expiry before accepting the claims.

**Example using jsonwebtoken:**

``` js
const jwt = require("jsonwebtoken");

const token = jwt.sign(
  { userId: user.id },
  process.env.JWT_SECRET,
  { expiresIn: "15m" }
);

const decoded = jwt.verify(token, process.env.JWT_SECRET);
```

Do not put passwords or sensitive personal information in a JWT payload.
A signed JWT is not automatically encrypted. Use HTTPS and protect
tokens from theft.

## 43. Access tokens vs refresh tokens?

  -----------------------------------------------------------------------
  Access token                        Refresh token
  ----------------------------------- -----------------------------------
  Used to access protected APIs       Used to obtain a new access token

  Usually short-lived                 Usually longer-lived

  Sent with API requests              Sent only to the refresh endpoint

  Limits exposure through short       Helps maintain a session without
  expiry                              frequent login
  -----------------------------------------------------------------------

A secure design should validate refresh tokens, rotate them where
appropriate, support revocation, and store them securely. In browser
applications, an HttpOnly, Secure, appropriately configured SameSite
cookie is often considered for refresh-token storage.

## 44. How do you prevent SQL and NoSQL injection?

Injection occurs when untrusted input is treated as part of a database
command or query.

**SQL injection prevention:** - Use parameterized queries or prepared
statements. - Avoid building SQL by concatenating user input. - Validate
input. - Use database accounts with only the permissions they need.

**NoSQL injection prevention:** - Validate input against an expected
schema. - Do not pass arbitrary user-controlled objects directly into
database query filters. - Allow only expected fields and operators. -
Use safe query-building patterns provided by the database library.

**Important:** Input validation is helpful, but parameterization and
safe query construction are the key protections.

## 45. How do you protect APIs against brute-force attacks?

A brute-force attack repeatedly tries passwords or other credentials
until it finds a valid one.

**Protection methods:** - Apply rate limits to login and sensitive
endpoints. - Add progressive delays after repeated failures. - Use
temporary account or login throttling carefully. - Use multi-factor
authentication where appropriate. - Store passwords using a strong
password-hashing algorithm such as Argon2id or bcrypt. - Monitor
suspicious activity and alert on unusual patterns. - Avoid revealing
whether an email address or account exists.

Rate limits should consider IP address and account identity, while
accounting for shared networks and distributed attacks.

## 46. How do you securely manage environment variables?

Environment variables keep configuration separate from application code.
They are commonly used for database URLs, API keys, and secret values.

**Good practices:** - Never commit `.env` files containing secrets to
Git. - Add `.env` to `.gitignore`. - Use a secret manager or deployment
platform's secret configuration in production. - Validate required
environment variables when the application starts. - Use separate
secrets for development, testing, and production. - Rotate secrets if
they are exposed. - Never send server secrets to frontend code or log
them.

**Example:**

``` js
require("dotenv").config();

const port = process.env.PORT || 5000;
const databaseUrl = process.env.DATABASE_URL;
```

## 47. How do you implement rate limiting?

Rate limiting restricts how many requests a client can make within a
time window. It helps protect APIs from excessive traffic and abuse.

Express applications can use the `express-rate-limit` package.

**Example:**

``` js
const rateLimit = require("express-rate-limit");

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  limit: 5,
  standardHeaders: "draft-8",
  legacyHeaders: false
});

app.use("/api/auth/login", loginLimiter);
```

For a multi-instance production application, use a shared store such as
Redis so limits are consistent across server instances. Configure proxy
trust carefully when using IP-based limits behind a reverse proxy.

## 48. How would you implement Redis caching?

Redis is an in-memory data store often used as a cache to reduce
repeated database queries and improve response times.

**Typical cache-aside flow:** 1. The API receives a request. 2. The
application checks Redis for the requested data. 3. If data exists,
return the cached value. 4. If not, fetch it from the database. 5. Store
the result in Redis with a suitable expiry time. 6. Return the result.
7. When the underlying data changes, update or invalidate the relevant
cache entry.

**Simplified example:**

``` js
const cached = await redis.get("products");

if (cached) {
  return res.json(JSON.parse(cached));
}

const products = await Product.find();

await redis.set(
  "products",
  JSON.stringify(products),
  { EX: 60 }
);

res.json(products);
```

Choose cache keys carefully, set expiry times, and invalidate or update
cached data when records change. The exact Redis client syntax can vary
by package and version.

## 49. How do you scale Node.js applications?

Scaling means increasing an application's ability to handle more users,
requests, or data.

**Vertical scaling:** Give a server more CPU, memory, or other
resources.

**Horizontal scaling:** Run multiple application instances across
processes or machines.

**Common techniques:** - Run multiple instances behind a load
balancer. - Use containers and orchestration where appropriate. - Use
Cluster or multiple processes for multi-core server workloads. - Use
Worker Threads for CPU-intensive tasks. - Optimize database queries and
add indexes. - Use Redis caching. - Use queues for background jobs. -
Use pagination and efficient streaming. - Monitor latency, errors, CPU,
and memory.

Design the application to avoid relying on local in-memory session state
when requests may reach different instances.

## 50. How would you design a production-ready Node.js backend?

A production-ready backend should be secure, maintainable, reliable,
observable, and easy to scale.

**A practical design checklist:**

1.  **Project structure:** Separate routes, controllers, services,
    models, middleware, configuration, and utilities.
2.  **Configuration:** Keep environment-specific settings and secrets
    outside source code.
3.  **Authentication:** Implement secure login, password hashing, token
    validation, and refresh-token handling where required.
4.  **Authorization:** Enforce roles and resource-level permissions.
5.  **Validation:** Validate request bodies, route parameters, and query
    parameters.
6.  **Error handling:** Use centralized error-handling middleware and
    consistent error responses.
7.  **Database:** Use appropriate schemas, indexes, transactions where
    needed, and efficient queries.
8.  **Security:** Use HTTPS, secure headers, rate limiting, safe
    database queries, and appropriate CORS configuration.
9.  **Logging:** Record useful operational information without exposing
    secrets or sensitive data.
10. **Testing:** Add unit, integration, and API tests.
11. **Performance:** Use caching, pagination, streams, and background
    jobs where appropriate.
12. **Deployment:** Use a production process manager or platform, health
    checks, and a reliable deployment process.
13. **Monitoring:** Track errors, latency, resource usage, and service
    availability.
14. **Documentation:** Document endpoints, authentication, request
    formats, and response formats.

**Interview answer:** I would build a modular Node.js application using
Express, separate routes and business logic, validate all incoming
requests, implement JWT-based authentication and authorization, connect
to a properly indexed database, add centralized error handling and
logging, protect APIs with security middleware and rate limiting, write
tests, and deploy with environment-based configuration and monitoring.

------------------------------------------------------------------------

## Quick Revision: Important Node.js Concepts

  Concept          Remember
  ---------------- ---------------------------------------------
  Node.js          JavaScript runtime environment
  V8               Executes JavaScript
  libuv            Event loop and asynchronous I/O support
  Event Loop       Coordinates callback execution
  Callback         Function called later
  Promise          Represents an eventual asynchronous result
  async/await      Syntax for working with Promises
  Express          Node.js web framework
  Middleware       Function in the request-response cycle
  REST API         Resource-oriented HTTP interface
  Stream           Processes data in chunks
  Buffer           Handles binary data
  EventEmitter     Emits and listens for events
  Worker Threads   Parallel JavaScript execution
  Cluster          Multiple Node.js processes
  JWT              Signed token format
  Redis            In-memory data store often used for caching
  Rate limiting    Restricts request frequency

## Suggested Study Method

For each question:

1.  Read the simple explanation.
2.  Understand the real-world purpose.
3.  Run the code example where one is provided.
4.  Close the notes and explain the concept in your own words.
5.  Practise the interview answer aloud.
6.  Try writing a small example without copying.

**Interview tip:** Focus on understanding how Node.js behaves in real
applications. Do not memorize definitions without understanding the
reason behind them.
