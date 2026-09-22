# 🚀 Socket.io in Node.js --- Complete Implementation Guide

> A practical, beginner-friendly guide to building real-time
> communication with Node.js, Express and Socket.io.

## 📚 Table of Contents

1.  What is Socket.io?
2.  Why use Socket.io?
3.  HTTP vs WebSocket vs Socket.io
4.  Real-world use cases
5.  Project architecture
6.  Install Socket.io
7.  Create the HTTP server
8.  Attach Socket.io
9.  Handle connections
10. Custom events
11. Rooms
12. Room messaging
13. Disconnect handling
14. Client implementation
15. Important methods
16. Methods vs events
17. Complete chat flow
18. Common mistakes
19. Interview questions
20. Quick cheat sheet

------------------------------------------------------------------------

# 🔌 What is Socket.io?

Socket.io is a JavaScript library for real-time, bidirectional,
event-based communication between a client and a server.

### Normal HTTP

``` text
Client  ───── Request ─────> Server
Client  <──── Response ───── Server
```

### Socket.io

``` text
Client  <══════════════════> Server
          Persistent
          connection
```

Both sides can send events when needed.

### 🍕 Simple analogy

**Normal HTTP:** The customer repeatedly asks the waiter for updates.

**Socket.io:** The customer and waiter keep an open communication line,
so the waiter can immediately send an update.

------------------------------------------------------------------------

# 🎯 Why do we use Socket.io?

Socket.io is useful when information needs to update immediately.

  Application        Example
  ------------------ --------------------------------------
  💬 Chat            Real-time messaging
  🔔 Notifications   New notification appears immediately
  📊 Dashboard       Live metrics
  🚚 Tracking        Delivery status/location
  🎮 Games           Multiplayer events
  💹 Market apps     Live price updates
  👥 Collaboration   Multiple users working together
  🛎️ Support         Live customer-support chat

------------------------------------------------------------------------

# 🔄 HTTP vs WebSocket vs Socket.io

## HTTP

``` text
Client → Request → Server
Client ← Response ← Server
```

The client normally starts the communication.

## WebSocket

``` text
Client ═════════ Server
       two-way
       connection
```

The server can send information to the client whenever necessary.

## Socket.io

Socket.io provides a convenient event-based API for real-time
connections and features such as:

-   Event-based communication
-   Rooms
-   Broadcasting
-   Reconnection
-   Connection management
-   A client library

> **Important:** Socket.io is not exactly the same protocol as a plain
> WebSocket. A Socket.io client communicates with a Socket.io server.

------------------------------------------------------------------------

# 🏗️ Project Architecture

For our Day 6 project:

``` text
node-day-6/
│
├── app.js
│
├── chat/
│   └── socketController.js
│
├── client.html
│
├── streams/
├── processes/
├── workers/
└── security/
```

Architecture:

``` text
                 ┌──────────────────────┐
                 │      Node Server     │
                 │       Express        │
                 │         +            │
                 │      Socket.io       │
                 └──────────┬───────────┘
                            │
              ┌─────────────┴─────────────┐
              │                           │
          Client 1                    Client 2
          User 1                      User 2
```

------------------------------------------------------------------------

# 🛠️ Step 1 --- Install Socket.io

``` bash
npm install socket.io
```

For the browser client, Socket.io can serve its client script from the
server.

------------------------------------------------------------------------

# 🏗️ Step 2 --- Create the HTTP Server

Instead of:

``` js
app.listen(3000);
```

create an HTTP server:

``` js
import express from "express";
import http from "node:http";

const app = express();

const server = http.createServer(app);
```

### Why?

Socket.io needs to be attached to the HTTP server.

``` text
Express App
    ↓
HTTP Server
    ↓
Socket.io
```

------------------------------------------------------------------------

# 🔌 Step 3 --- Attach Socket.io

``` js
import { Server } from "socket.io";

const io = new Server(server, {
  cors: {
    origin: "*",
  },
});
```

### Important

Correct:

``` js
const io = new Server(server);
```

Incomplete for this setup:

``` js
const io = new Server();
```

The `server` connects Socket.io to your HTTP server.

------------------------------------------------------------------------

# 🚀 Step 4 --- Start the Server

Use:

``` js
server.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

Not:

``` js
app.listen(3000);
```

Complete basic server:

``` js
import express from "express";
import http from "node:http";
import { Server } from "socket.io";

const app = express();

const server = http.createServer(app);

const io = new Server(server, {
  cors: {
    origin: "*",
  },
});

io.on("connection", (socket) => {
  console.log(`[Socket Connected] ${socket.id}`);
});

server.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

------------------------------------------------------------------------

# 🔗 Step 5 --- Handle Connections

The most important event is:

``` js
io.on("connection", (socket) => {
});
```

It means:

> When a client connects to the Socket.io server, execute this function.

Example:

``` js
io.on("connection", (socket) => {
  console.log("Client connected");
});
```

Every connected client gets a unique:

``` js
socket.id
```

Example:

``` text
User 1 → qcYCb7QzeFuJuZwBAAAA
User 2 → LV64cKggXYnKTXryAAAB
```

------------------------------------------------------------------------

# 📡 Step 6 --- Create Custom Events

Socket.io is event-based.

``` js
socket.on("sendMessage", (data) => {
  console.log(data);
});
```

`sendMessage` is a custom event.

Client:

``` js
socket.emit("sendMessage", {
  message: "Hello",
});
```

Server:

``` js
socket.on("sendMessage", (data) => {
  console.log(data.message);
});
```

### Flow

``` text
Client
  │
  │ emit("sendMessage")
  ↓
Server
  │
  │ on("sendMessage")
  ↓
Handle event
```

------------------------------------------------------------------------

# 🏠 Step 7 --- Create Rooms

A room is a logical group of connected sockets.

``` text
chatroom1
├── User 1
├── User 2
└── User 3

chatroom2
├── User 4
└── User 5
```

Join a room:

``` js
socket.join("chatroom1");
```

Example:

``` js
socket.on("joinRoom", (roomName) => {
  socket.join(roomName);

  console.log(
    `${socket.id} joined ${roomName}`
  );
});
```

Client:

``` js
socket.emit("joinRoom", "chatroom1");
```

------------------------------------------------------------------------

# 📢 Step 8 --- Send Messages to a Room

Use:

``` js
io.to(roomName).emit(eventName, data);
```

Example:

``` js
io.to(data.room).emit("receiveMessage", {
  user: data.user,
  message: data.message,
});
```

This sends `receiveMessage` to sockets in that room.

``` text
User 1
  │
  │ "Hello"
  ↓
chatroom1
  │
  ├── User 1
  ├── User 2 ← receives
  └── User 3 ← receives
```

------------------------------------------------------------------------

# 👤 Step 9 --- Handle Disconnect

``` js
socket.on("disconnect", () => {
  console.log(
    `[Socket Disconnected] ${socket.id}`
  );
});
```

This runs when a client disconnects.

------------------------------------------------------------------------

# 🧩 Production-style Socket Controller

``` js
const setupSocket = (io) => {

  io.on("connection", (socket) => {

    console.log(
      `[Socket Connected] ${socket.id}`
    );

    socket.on("joinRoom", (roomName) => {

      socket.join(roomName);

      console.log(
        `[Room] ${socket.id} joined ${roomName}`
      );

    });

    socket.on("sendMessage", (data) => {

      console.log(
        `[Message] ${socket.id}: ${data.message}`
      );

      io.to(data.room).emit("receiveMessage", {
        user: data.user,
        message: data.message,
      });

    });

    socket.on("disconnect", () => {

      console.log(
        `[Socket Disconnected] ${socket.id}`
      );

    });

  });

};

export default setupSocket;
```

------------------------------------------------------------------------

# 🌐 Client Implementation

Create:

``` text
client.html
```

Load Socket.io first:

``` html
<script src="http://localhost:3000/socket.io/socket.io.js"></script>
```

Then connect:

``` js
const socket = io("http://localhost:3000");
```

### Important order

Correct:

``` html
<script src="http://localhost:3000/socket.io/socket.io.js"></script>

<script>
  const socket = io("http://localhost:3000");
</script>
```

If `io` is called before the client library loads, you can get:

``` text
ReferenceError: io is not defined
```

------------------------------------------------------------------------

# 📤 Client → Server

``` js
socket.emit("eventName", data);
```

Example:

``` js
socket.emit("sendMessage", {
  user: "User1",
  room: "chatroom1",
  message: "Hello World!",
});
```

# 📥 Server Receives

``` js
socket.on("sendMessage", (data) => {
  console.log(data);
});
```

------------------------------------------------------------------------

# 📤 Server → Current Client

``` js
socket.emit("welcome", {
  message: "Welcome!",
});
```

------------------------------------------------------------------------

# 📢 Server → Everyone

``` js
io.emit("announcement", {
  message: "Server announcement",
});
```

All connected clients receive it.

------------------------------------------------------------------------

# 📢 Server → Everyone Except Sender

``` js
socket.broadcast.emit("userJoined", {
  user: "User1",
});
```

The sender does not receive this broadcast.

------------------------------------------------------------------------

# 🏠 Server → Specific Room

``` js
io.to("chatroom1").emit(
  "receiveMessage",
  data
);
```

All sockets in `chatroom1` receive it.

------------------------------------------------------------------------

# 🏠 Room Except Sender

``` js
socket.to("chatroom1").emit(
  "receiveMessage",
  data
);
```

Other sockets in the room receive it, excluding the current sender.

------------------------------------------------------------------------

# 🧰 Important Socket.io Methods

## `io.on()`

Listen for server-level events.

``` js
io.on("connection", (socket) => {});
```

## `socket.on()`

Listen for an event from a client.

``` js
socket.on("sendMessage", (data) => {});
```

## `socket.emit()`

Send an event to the current socket/client.

``` js
socket.emit("welcome", data);
```

## `io.emit()`

Send an event to all connected clients.

``` js
io.emit("announcement", data);
```

## `socket.broadcast.emit()`

Send to everyone except the current socket.

``` js
socket.broadcast.emit("userJoined", data);
```

## `socket.join()`

Add a socket to a room.

``` js
socket.join("chatroom1");
```

## `socket.leave()`

Remove a socket from a room.

``` js
socket.leave("chatroom1");
```

## `io.to().emit()`

Send to everyone in a room.

``` js
io.to("chatroom1").emit(
  "receiveMessage",
  data
);
```

## `socket.to().emit()`

Send to other sockets in a room, excluding the current socket.

``` js
socket.to("chatroom1").emit(
  "receiveMessage",
  data
);
```

## `socket.disconnect()`

Disconnect a socket intentionally.

``` js
socket.disconnect();
```

------------------------------------------------------------------------

# 🧠 Methods vs Events

This is an important beginner concept.

### Methods

Actions provided by Socket.io:

``` js
socket.emit()
socket.join()
socket.leave()
io.emit()
io.to()
```

### Events

Things that happen or messages that you define:

``` text
connection
disconnect
sendMessage
joinRoom
receiveMessage
```

Example:

``` js
socket.on("sendMessage", data => {});
```

Here:

``` text
on()          → method
sendMessage   → event name
```

And:

``` js
socket.emit("sendMessage", data);
```

Here:

``` text
emit()        → method
sendMessage   → event name
```

------------------------------------------------------------------------

# 🔥 Complete Chat Flow

User 1 joins:

``` js
socket.emit("joinRoom", "chatroom1");
```

Server receives:

``` js
socket.on("joinRoom", (roomName) => {
  socket.join(roomName);
});
```

User 2 does the same.

User 1 sends:

``` js
socket.emit("sendMessage", {
  user: "User1",
  room: "chatroom1",
  message: "Hello World!",
});
```

Server receives:

``` js
socket.on("sendMessage", (data) => {
```

Then broadcasts:

``` js
io.to(data.room).emit(
  "receiveMessage",
  {
    user: data.user,
    message: data.message,
  }
);
```

Clients listen:

``` js
socket.on("receiveMessage", (data) => {
  console.log(data);
});
```

### Complete flow

``` text
             User 1
                │
                │ emit("sendMessage")
                ↓
        ┌────────────────┐
        │  Socket.io     │
        │     Server     │
        └───────┬────────┘
                │
                │ io.to(room).emit()
                ↓
           chatroom1
          /                   ↓            ↓
      User 1        User 2
      receives      receives
```

------------------------------------------------------------------------

# ⚠️ Common Mistakes

## 1. `io is not defined`

Make sure the client library loads before calling `io()`:

``` html
<script src="http://localhost:3000/socket.io/socket.io.js"></script>
<script>
  const socket = io("http://localhost:3000");
</script>
```

## 2. Socket.io not attached to server

Wrong:

``` js
const io = new Server();
```

Correct:

``` js
const io = new Server(server);
```

## 3. Using `app.listen()`

For this setup:

``` js
server.listen(3000);
```

because Socket.io is attached to `server`.

## 4. Expecting `connection` without a client

This:

``` js
io.on("connection", () => {
  console.log("Connected");
});
```

does not print just because Node starts.

A client must actually connect.

------------------------------------------------------------------------

# 🎤 Interview Questions

### What is Socket.io?

> Socket.io is a JavaScript library that provides event-based,
> real-time, bidirectional communication between clients and servers.

### Why use Socket.io instead of normal HTTP?

> HTTP is primarily request-response based. Socket.io maintains a
> real-time connection so the server and client can exchange events
> without requiring a new HTTP request for every update.

### What is `socket.emit()`?

> It sends an event to a particular socket/client.

### What is `io.emit()`?

> It broadcasts an event to all connected sockets.

### What is `socket.broadcast.emit()`?

> It broadcasts an event to all connected sockets except the sender.

### What is a Socket.io room?

> A room is a logical grouping of sockets that allows messages to be
> broadcast to a specific group of connected clients.

### Difference between `io.to()` and `socket.to()`?

``` js
io.to("room").emit(...)
```

Sends to sockets in the room, including the sender if the sender is in
that room.

``` js
socket.to("room").emit(...)
```

Sends to other sockets in the room, excluding the current sender.

### What is `socket.id`?

> A unique identifier associated with a particular Socket.io connection.

------------------------------------------------------------------------

# 📝 Quick Cheat Sheet

``` text
on()       → Listen

emit()     → Send

join()     → Enter room

leave()    → Exit room

to()       → Target room
```

### Core API

``` text
CONNECT
────────────────────────────
io.on("connection", socket => {})

LISTEN
────────────────────────────
socket.on("event", data => {})

CURRENT CLIENT
────────────────────────────
socket.emit("event", data)

EVERYONE
────────────────────────────
io.emit("event", data)

EVERYONE EXCEPT SENDER
────────────────────────────
socket.broadcast.emit("event", data)

JOIN ROOM
────────────────────────────
socket.join("room")

LEAVE ROOM
────────────────────────────
socket.leave("room")

ROOM
────────────────────────────
io.to("room").emit("event", data)

ROOM EXCEPT SENDER
────────────────────────────
socket.to("room").emit("event", data)

DISCONNECT
────────────────────────────
socket.on("disconnect", () => {})
```

------------------------------------------------------------------------

# 🏆 Learning Path

``` text
1. Understand real-time communication
             ↓
2. Install Socket.io
             ↓
3. Create HTTP server
             ↓
4. Attach Socket.io
             ↓
5. Handle connection
             ↓
6. Create custom events
             ↓
7. Emit events
             ↓
8. Listen for events
             ↓
9. Create rooms
             ↓
10. Broadcast messages
             ↓
11. Handle disconnect
             ↓
12. Build real-time chat
```

------------------------------------------------------------------------

# 💡 Final Mental Model

Remember these five:

``` text
on()       → "Listen"

emit()     → "Send"

join()     → "Enter room"

leave()    → "Exit room"

to()       → "Target room"
```

And:

``` text
socket = ONE connected client

io     = Socket.io server / connected sockets

room   = GROUP of connected sockets
```

> **Interview shortcut:** `on` listens, `emit` sends, `join` enters a
> room, `leave` exits a room, and `to` targets a room.
