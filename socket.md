# Socket.io in Node.js

- **Socket.io** — a library that enables **real-time, bidirectional communication** between the client and the server. Unlike HTTP, where the client sends a request and waits for a response, Socket.io keeps a persistent connection open so both the client and server can send data to each other at any time.

---

## Why do we need Socket.io?

HTTP works like this:

```text
Client

↓

Request

↓

Server

↓

Response

↓

Connection Closed
```

Every new request creates a new communication cycle.

If you want real-time updates (chat messages, notifications, live scores), the client would need to repeatedly ask the server:

```text
Any new message?

↓

No

↓

Any new message?

↓

No

↓

Any new message?

↓

Yes
```

This is called **Polling**, and it wastes network resources.

---

Socket.io works differently:

```text
Client

⇅

Persistent Connection

⇅

Server
```

Once connected, both the client and server can send messages instantly without creating a new connection each time.

---

# How Socket.io Works

### Step 1

Client connects.

```javascript
const socket = io("http://localhost:3000");
```

↓

### Step 2

Server accepts the connection.

```javascript
io.on("connection", (socket) => {
    console.log("User Connected");
});
```

↓

### Step 3

Connection remains open.

↓

### Step 4

Either side can send data anytime.

---

# Installation

Server

```bash
npm install socket.io
```

Client

```bash
npm install socket.io-client
```

---

# Basic Server Example

```javascript
const express = require("express");
const http = require("http");
const { Server } = require("socket.io");

const app = express();

const server = http.createServer(app);

const io = new Server(server);

io.on("connection", (socket) => {

    console.log("User Connected");

    socket.on("message", (data) => {
        console.log(data);
    });

    socket.emit("welcome", "Hello Client");
});

server.listen(3000);
```

---

# Basic Client Example

```javascript
import { io } from "socket.io-client";

const socket = io("http://localhost:3000");

socket.on("welcome", (msg) => {
    console.log(msg);
});

socket.emit("message", "Hello Server");
```

---

# Common Socket Events

## connection

Runs when a client connects.

```javascript
io.on("connection", (socket) => {

});
```

---

## disconnect

Runs when the client disconnects.

```javascript
socket.on("disconnect", () => {

});
```

---

## emit()

Send data.

```javascript
socket.emit("message", data);
```

---

## on()

Receive data.

```javascript
socket.on("message", callback);
```

---

## broadcast

Send data to everyone except the sender.

```javascript
socket.broadcast.emit("message", data);
```

---

## io.emit()

Send to everyone.

```javascript
io.emit("message", data);
```

---

## socket.emit()

Send only to one client.

```javascript
socket.emit("message", data);
```

---

# Rooms

Rooms allow grouping users.

Example

```text
Room A

Alice

Bob

Charlie
```

```javascript
socket.join("room1");
```

Send message only to that room.

```javascript
io.to("room1").emit("message");
```

---

# Namespaces

Instead of one connection, create multiple channels.

```text
/socket

/chat

/admin

/game
```

Example

```javascript
const admin = io.of("/admin");
```

---

# Socket Lifecycle

```text
Client

↓

Connect

↓

Authentication

↓

Join Rooms

↓

Send / Receive Events

↓

Disconnect
```

---

# Real-World Uses

- Chat Applications
- WhatsApp Clone
- Facebook Messenger
- Live Cricket Score
- Uber Driver Tracking
- Food Delivery Tracking
- Multiplayer Games
- Live Notifications
- Stock Market Dashboard

---

# Socket.io Middleware

- **Socket.io Middleware** — a function that executes **before a socket connection is established** or before processing specific events. It is mainly used for authentication, authorization, validation, logging, and rate limiting. Similar to Express middleware, it can either allow the connection to continue or reject it with an error.

---

## Why do we need Socket Middleware?

Without middleware

```text
Client

↓

Socket Connected

↓

Anyone can connect ❌
```

With middleware

```text
Client

↓

Middleware

↓

Verify JWT

↓

Connected ✅

or

Rejected ❌
```

---

# Basic Middleware

```javascript
io.use((socket, next) => {

    console.log("Middleware");

    next();
});
```

`next()` allows the connection.

---

# Authentication Middleware

Client

```javascript
const socket = io("http://localhost:3000", {
    auth: {
        token: "JWT_TOKEN"
    }
});
```

Server

```javascript
io.use((socket, next) => {

    const token = socket.handshake.auth.token;

    if (!token) {
        return next(new Error("Unauthorized"));
    }

    next();
});
```

---

# JWT Authentication Example

```javascript
const jwt = require("jsonwebtoken");

io.use((socket, next) => {

    try {

        const token = socket.handshake.auth.token;

        const user = jwt.verify(token, "SECRET");

        socket.user = user;

        next();

    } catch {

        next(new Error("Authentication Failed"));
    }

});
```

Now every connected socket has

```javascript
socket.user
```

---

# Logging Middleware

```javascript
io.use((socket, next) => {

    console.log(socket.id);

    next();

});
```

---

# Role-Based Authorization

```javascript
io.use((socket, next) => {

    if(socket.user.role !== "admin"){

        return next(new Error("Forbidden"));

    }

    next();

});
```

---

# Event Middleware

Middleware can also be applied to every event from a connected socket.

```javascript
socket.use((packet, next) => {

    console.log(packet);

    next();

});
```

Example

```javascript
socket.use((packet, next) => {

    if(packet[0] === "deleteUser"){

        return next(new Error("Not Allowed"));

    }

    next();

});
```

This checks each incoming event before it is handled.

---

# Connection Flow

```text
Client

↓

Connect

↓

Socket Middleware

↓

JWT Verification

↓

Connection Accepted

↓

Join Room

↓

Emit Events

↓

Disconnect
```

---

# Advantages of Socket Middleware

- Authenticate users before connecting.
- Verify JWT tokens.
- Check user roles and permissions.
- Log socket activity.
- Validate incoming data.
- Apply rate limiting.
- Reject unauthorized users before they establish a connection.

---

# Socket Middleware vs Express Middleware

| Express Middleware | Socket Middleware |
|---------------------|-------------------|
| Runs on every HTTP request | Runs before a socket connection or on socket events |
| Uses `req`, `res`, `next` | Uses `socket`, `next` |
| Handles REST API requests | Handles WebSocket connections |
| Authentication via request headers/cookies | Authentication via `socket.handshake` |
| Executes for each HTTP request | Executes once per connection (or per event with `socket.use()`) |

---

# Common Interview Questions

### Why use Socket.io instead of HTTP?

Socket.io maintains a persistent connection, enabling real-time two-way communication. HTTP follows a request-response model and closes the connection after each response.

---

### How is authentication done in Socket.io?

Typically by sending a JWT or session information during the handshake (`socket.handshake.auth`) and validating it in Socket.io middleware before accepting the connection.

---

### Can Socket.io work without WebSockets?

Yes. Socket.io automatically falls back to other transport mechanisms (such as HTTP long polling) if WebSockets are unavailable, while still providing the same API to the developer.

---

### What is the difference between `io.emit()` and `socket.emit()`?

- `io.emit()` sends an event to **all connected clients**.
- `socket.emit()` sends an event **only to the current connected client**.

---

### What is the difference between Rooms and Namespaces?

- **Rooms** group clients within the same namespace so messages can be sent to specific groups.
- **Namespaces** create separate communication channels (e.g., `/chat`, `/admin`) with their own event handlers and middleware.