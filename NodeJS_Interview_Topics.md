# Node.js Interview Guide — Explained in Easy Language

This is a simplified version of your detailed guide. Same topics, but explained like you're learning it for the first time — simple words, small examples, no heavy jargon.

---

# 🟢 BEGINNER LEVEL

## 1. Node.js Basics

**What is Node.js?**
Normally, JavaScript only runs inside a browser. Node.js lets JavaScript run **outside the browser** — like on a server, on your computer, anywhere. It's built using Chrome's V8 engine (the same thing that runs JS in Chrome, just without the browser around it).

**Why do people use it?**
- Same language (JS) for both frontend and backend — no need to learn a second language for the server.
- Huge library support through npm (almost anything is already built by someone).
- Very good at handling many requests at once, especially things like APIs, chat apps, live data — because it doesn't "wait" for slow tasks, it moves on and comes back later.
- Not great for heavy calculations (like video processing) because that keeps the one main "thread" busy and blocks everything else.

**Node.js vs Browser JS**
- No `window`/`document` in Node.js (there's no browser page to control). Instead, we get `process`, `global`, `require`.
- Node.js can touch the file system, network, etc. — a browser can't do that for security reasons.
- Node.js code runs like a normal program on your computer/server (same permissions as you), while browser JS is locked in a safe "sandbox."

**npm & package.json**
Think of `package.json` as your project's **profile card** — it lists the project name, what packages it needs, and shortcut commands.
- `dependencies` → packages needed to actually run the app (e.g. `express`)
- `devDependencies` → packages only needed while coding/testing (e.g. `nodemon`)
- `scripts` → shortcuts, like typing `npm start` instead of a long command

**Version numbers (semver): `MAJOR.MINOR.PATCH`**
- `^4.18.2` → allow small/medium updates, but not a big version jump
- `~4.18.2` → allow only tiny bug-fix updates
- `4.18.2` (no symbol) → exact version only, no updates

`package-lock.json` locks the **exact** versions used, so everyone on the team installs the exact same thing.

**Node REPL**
Just type `node` in your terminal with nothing after it — you get an interactive box where you can test small pieces of JS instantly.

**Global objects (available everywhere in Node)**
- `global` → like `window` in the browser, but for Node
- `process` → info & control about the running app (`process.env` for environment variables, `process.exit()` to stop the app)
- `__dirname` / `__filename` → tells you the current folder/file path
- `require` → used to import other files/packages

---

## 2. Modules (Reusable Code Files)

**CommonJS (the classic way)**
Every file is treated as its own private box. You "export" things out of the box and "require" them somewhere else.

```js
// math.js
function add(a, b) { return a + b; }
module.exports = { add };

// app.js
const { add } = require('./math');
console.log(add(2, 3)); // 5
```

**Built-in modules that come free with Node**: `fs` (files), `path` (folder/file paths), `os` (system info), `http` (create servers), `events` (custom events), `util` (helper tools).

**ES Modules (the newer way)**
Uses `import`/`export` instead of `require`/`module.exports`. This is the modern JavaScript standard (same as what you might use in React).

**Quick comparison**

| | CommonJS | ES Modules |
|---|---|---|
| Keywords | require / module.exports | import / export |
| Loads | Right away (sync) | Can load in background (async) |
| Removes unused code | No | Yes (tree-shaking) |

**Module caching**
If you `require` the same file twice, Node doesn't run it twice — it just gives you the saved (cached) result from the first time.

---

## 3. Working with Files (`fs` module)

**Sync vs Async**
- `fs.readFileSync()` → stops everything until the file is fully read. Fine for small scripts, bad for servers (freezes them).
- `fs.readFile()` → doesn't stop anything, gives you the result later through a callback.
- `fs.promises.readFile()` → same non-blocking idea, but works nicely with `async/await`.

```js
const fs = require('fs');
fs.readFile('data.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

**Streams** — instead of loading a whole huge file into memory at once, you read it bit by bit (like watching a video that streams instead of downloading the whole thing first).

**Path helpers**: `path.join()` safely joins folder names together, `path.resolve()` gives you the full absolute path, `path.basename()` gives just the file name.

---

## 4. Basic Server with plain Node.js

```js
const http = require('http');
const server = http.createServer((req, res) => {
  if (req.method === 'GET' && req.url === '/') {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello World');
  } else {
    res.writeHead(404);
    res.end('Not Found');
  }
});
server.listen(3000, () => console.log('Server running on port 3000'));
```
You manually check the URL and method yourself here — that's tiring, which is exactly why frameworks like **Express** exist (to do this for you).

Common status codes to remember: `200` OK, `201` Created, `400` Bad Request, `401` Not logged in, `403` Not allowed, `404` Not Found, `500` Server Error.

---

## 5. Async Programming Basics

**Callback Hell** — nesting too many callbacks inside each other, making code messy and hard to follow (like a staircase going down).

**Promises** — an object that says "I'll give you a value later — either success or failure."

**async/await** — makes promise code *look* like normal step-by-step code, even though it's still async underneath.

**Error-first callback** — a Node.js convention where the first argument of a callback is always the error (if any): `callback(err, data)`.

---

## 6. NPM Basics

`package.json` = what versions you're okay with. `package-lock.json` = exact versions actually installed. Publishing a package needs a unique name + version, and once published, that version can't be changed (immutable).

---

# 🟡 INTERMEDIATE LEVEL

## 7. The Event Loop (Most Important Interview Topic!)

**What is it, in simple words?**
JavaScript can only do one thing at a time (single thread). But Node.js still manages to handle thousands of things "at once" — how? By handing off slow tasks (reading files, network calls) to the background, and only coming back to run your code once those tasks are done. The **event loop** is the manager that keeps checking: "Is anything ready? Let's run it."

**The Loop's Phases (like stations in a factory line)**
1. **Timers** — run any `setTimeout`/`setInterval` whose time is up
2. **Pending callbacks** — some leftover background tasks
3. **Poll** — the main station: check for finished I/O (like a finished file read) and run those callbacks
4. **Check** — run `setImmediate()` callbacks
5. **Close callbacks** — cleanup stuff (like `socket.on('close')`)

Between every single step, Node also fully empties out something called the **microtask queue** (explained below) before moving on.

**Call stack vs Macrotasks vs Microtasks**
- **Call stack** — where your normal, line-by-line code actually executes
- **Macrotasks** — bigger tasks tied to loop phases: `setTimeout`, `setInterval`, `setImmediate`, I/O
- **Microtasks** — small, high-priority tasks: Promises (`.then`) and `process.nextTick()`. These always run **before** the loop is allowed to move to the next phase or macrotask — even ahead of `setTimeout`.

**Order of priority:** `process.nextTick()` first → then Promises → then everything else (timers, I/O, etc.)

**Classic interview code:**
```js
console.log('1: start');
setTimeout(() => console.log('2: setTimeout'), 0);
Promise.resolve().then(() => console.log('3: promise'));
process.nextTick(() => console.log('4: nextTick'));
console.log('5: end');
```
**Answer:** `1 → 5 → 4 → 3 → 2`
Why: normal code runs first fully (`1`, `5`). Then Node empties `nextTick` (`4`), then Promises (`3`). Only after that does it move to the timer phase and run `setTimeout` (`2`).

**Libuv & the thread pool**
Libuv is the background engine that actually makes async possible in Node. For things the operating system can't do async on its own (some file operations, DNS lookups, certain crypto functions), libuv uses a small pool of extra threads (default: 4) to handle them without blocking your main code. You can increase this pool size using `UV_THREADPOOL_SIZE`.

**Why blocking code is dangerous**
If you run something heavy and synchronous (a giant loop, parsing a huge JSON, etc.), it locks up the ONE main thread — meaning literally nothing else can happen (no other user's request gets served) until it's done. Fix: move heavy work to worker threads, a separate process, or a queue.

---

## 8. Streams & Buffers

**4 Types of Streams**
- **Readable** — you can read data from it (e.g., reading a file)
- **Writable** — you can write data to it (e.g., saving a file)
- **Duplex** — can do both, independently (e.g., a network socket)
- **Transform** — reads, changes the data, then writes it out (e.g., compressing a file)

**Piping** — connects a readable stream straight into a writable one, automatically:
```js
fs.createReadStream('input.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('input.txt.gz'));
```

**Backpressure** — imagine pouring water into a bottle faster than it can drink it — it overflows. Same with data: if the writing side is slower than the reading side, `.pipe()` automatically pauses the reading side until the writing side catches up.

**Buffers** — a special format for handling raw binary data (like images, videos) since normal JS strings aren't good at handling that.

---

## 9. Express.js

**Basic setup**
```js
const express = require('express');
const app = express();
app.use(express.json()); // lets you read JSON sent in requests

app.get('/users/:id', (req, res) => {
  res.json({ id: req.params.id });
});

app.listen(3000);
```

**Middleware** — a function that sits "in between" the request coming in and the response going out. It can check something, change something, or stop the request — then it calls `next()` to pass it along.

**Error-handling middleware** — special, has 4 parameters instead of 3, and must be placed at the very end:
```js
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong' });
});
```

**REST API design basics** — use nouns for URLs (`/users`), use HTTP verbs for actions (GET=read, POST=create, PUT=full update, PATCH=partial update, DELETE=remove), use proper status codes, keep it stateless (server shouldn't "remember" things between requests).

---

## 10. Async Programming — Deeper Dive

**Promise combinators (ways to handle multiple promises together)**
- `Promise.all()` → waits for all; if even ONE fails, the whole thing fails immediately
- `Promise.allSettled()` → waits for all, never fails early — tells you which succeeded/failed
- `Promise.race()` → gives you the result of whichever promise finishes FIRST (used for timeouts)
- `Promise.any()` → gives you the first one that succeeds; only fails if ALL fail

**util.promisify** — converts old callback-style functions into promise-based ones so you can use `await` with them.

**EventEmitter** — Node's built-in system for creating and listening to custom events (like a mini notification system inside your code):
```js
const EventEmitter = require('events');
const emitter = new EventEmitter();
emitter.on('greet', (name) => console.log(`Hello, ${name}`));
emitter.emit('greet', 'World');
```

---

## 11. Working with Databases

- **SQL (MySQL, PostgreSQL)** — data stored in neat tables with fixed columns. Use libraries like `pg` or ORMs like Prisma/Sequelize.
- **NoSQL (MongoDB)** — data stored as flexible JSON-like documents. Use Mongoose.
- **Connection pooling** — instead of opening a brand-new (slow) database connection for every single request, keep a small set of ready-to-use connections and reuse them.
- **Transactions** — group several database changes together so either ALL of them succeed, or NONE of them do (very important for things like money transfers).

---

## 12. Error Handling

**Ways to catch errors**
- `try/catch` for normal and `async/await` code
- `.catch()` for Promise chains
- Checking `if (err)` in old-style callbacks

**Custom error types** — lets you create your own labeled errors (like "NotFoundError") so you can handle different problems differently later.

**Safety-net handlers** (should NOT be your main error-handling strategy):
```js
process.on('uncaughtException', (err) => { ... process.exit(1); });
process.on('unhandledRejection', (reason) => { ... });
```

**Two types of errors:**
- **Operational errors** — expected problems (wrong user input, DB timeout) — handle gracefully
- **Programmer errors** — actual bugs (calling something wrong in your code) — best to just crash and restart, don't try to "patch over" it

---

## 13. Auth & Security Basics

**Session-based auth** — server remembers who's logged in (stores it somewhere like Redis), gives the browser a cookie with a session ID.

**Token-based auth (JWT)** — server gives the client a signed token; client sends it back on every request; server just checks the signature — no need to "remember" anything server-side.

**JWT structure**: `header.payload.signature` — the payload holds info like user ID and expiry time; the signature proves it hasn't been tampered with.

**Password hashing** — never save plain passwords! Use `bcrypt`, which is intentionally slow, making it very hard for hackers to guess passwords by brute force.

**CORS** — browsers block requests to a different website/domain by default unless the server explicitly allows it via special headers.

**.env files** — keep secret keys/passwords outside your actual code, loaded at runtime, never pushed to GitHub.

**Common security tools**: `helmet` (adds safety headers), `express-rate-limit` (stops spam/brute-force by IP), `joi`/`zod` (validate incoming data before trusting it).

---
# Helmet (Node.js Security Middleware)

- **Helmet** — a security middleware for Express.js that helps protect your application by automatically setting various HTTP response headers. These headers instruct browsers how to securely handle your website, reducing the risk of attacks such as XSS, Clickjacking, MIME Sniffing, and information leakage.

---

## Why do we use Helmet?

When users visit your website, the server sends two things:

1. **Response Body** (HTML, JSON, etc.)
2. **Response Headers**

Example:

```http
HTTP/1.1 200 OK

Content-Type: text/html
Content-Length: 1200
```

Helmet adds additional **security headers** to the response.

Example

```http
HTTP/1.1 200 OK

Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Referrer-Policy: no-referrer
```

These headers tell the browser how to securely display your application.

---

## Installation

```bash
npm install helmet
```

---

## Basic Usage

```javascript
const express = require("express");
const helmet = require("helmet");

const app = express();

app.use(helmet());

app.listen(3000);
```

That's it.

Helmet automatically enables several security protections.

---

# What Problems Does Helmet Solve?

Without Helmet

```text
User

↓

Browser

↓

Website

↓

No Security Headers

↓

Higher Risk of Attacks
```

With Helmet

```text
User

↓

Browser

↓

Helmet Headers

↓

Browser Enforces Security

↓

Website
```

---

# Security Headers Added by Helmet

---

## 1. Content-Security-Policy (CSP)

- **Definition** — controls which sources (scripts, styles, images, fonts, etc.) the browser is allowed to load, helping prevent Cross-Site Scripting (XSS) attacks.

Example

```http
Content-Security-Policy:
default-src 'self';
script-src 'self';
```

Meaning

- Only load JavaScript from your own website.
- Block scripts from unknown websites.

Without CSP

```text
Website

↓

Attacker injects JavaScript

↓

Browser executes it ❌
```

With CSP

```text
Website

↓

Injected Script

↓

Blocked by Browser ✅
```

---

## 2. X-Content-Type-Options

- **Definition** — prevents browsers from guessing a file's MIME type, reducing the risk of executing malicious files.

Header

```http
X-Content-Type-Options: nosniff
```

Without it

```text
virus.jpg

↓

Browser guesses it's JavaScript

↓

Executes ❌
```

With Helmet

```text
virus.jpg

↓

Browser treats it only as an image ✅
```

---

## 3. X-Frame-Options

- **Definition** — prevents your website from being embedded inside an `<iframe>`, protecting against Clickjacking attacks.

Header

```http
X-Frame-Options: SAMEORIGIN
```

Without it

```text
Attacker Website

↓

Invisible iframe

↓

Your Banking Website

↓

User clicks unknowingly ❌
```

With Helmet

```text
Browser blocks iframe ✅
```

---

## 4. Referrer-Policy

- **Definition** — controls how much information about the current page is sent when users navigate to another website.

Example

```http
Referrer-Policy: no-referrer
```

Meaning

No URL information is shared with other websites.

---

## 5. Cross-Origin-Opener-Policy (COOP)

- **Definition** — isolates your application from other browser windows and tabs, helping protect against cross-origin attacks and improving security.

Header

```http
Cross-Origin-Opener-Policy: same-origin
```

---

## 6. Cross-Origin-Resource-Policy (CORP)

- **Definition** — prevents other websites from loading your site's resources unless explicitly allowed.

Example

```http
Cross-Origin-Resource-Policy: same-origin
```

---

## 7. Origin-Agent-Cluster

- **Definition** — tells the browser to isolate your application into its own process, improving memory isolation and security.

Header

```http
Origin-Agent-Cluster: ?1
```

---

## 8. DNS Prefetch Control

- **Definition** — controls whether browsers perform DNS prefetching for links before users click them, helping reduce unnecessary information leakage.

Example

```http
X-DNS-Prefetch-Control: off
```

---

# Configuring Helmet

Example

```javascript
const helmet = require("helmet");

app.use(
  helmet({
    contentSecurityPolicy: false,
    crossOriginEmbedderPolicy: false,
  })
);
```

This disables specific Helmet protections when necessary.

---

# Custom Content Security Policy

```javascript
app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "https://cdn.example.com"],
      imgSrc: ["'self'", "https://images.example.com"],
    },
  })
);
```

Now only those domains can load scripts and images.

---

# Helmet vs CORS

Many developers confuse these.

### Helmet

- Protects your application using **security headers**.
- Helps prevent XSS, Clickjacking, MIME Sniffing, and information leakage.

### CORS

- Controls **which websites are allowed to access your API**.
- Prevents unauthorized cross-origin requests.

Example

```javascript
app.use(helmet());
app.use(cors());
```

Most Express applications use **both**.

---

# Advantages

- Easy to use.
- Adds multiple security headers automatically.
- Helps prevent common web attacks.
- Improves browser security.
- Recommended for almost every Express application.

---

# Disadvantages

- Some headers (especially CSP) may block legitimate third-party scripts if not configured correctly.
- Certain applications require custom Helmet configuration.
- Does not replace input validation, authentication, authorization, or other security best practices.

---

# Common Interview Questions

### Why do we use Helmet?

Helmet secures Express applications by automatically adding HTTP security headers that help prevent attacks such as XSS, Clickjacking, MIME Sniffing, and information leakage.

---

### Does Helmet encrypt data?

No.

Helmet only adds **HTTP security headers**.

Encryption is provided by **HTTPS/TLS**, not Helmet.

---

### Does Helmet prevent SQL Injection?

No.

SQL Injection is prevented by using:

- Parameterized queries
- Prepared statements
- ORM methods

Helmet only protects through browser security headers.

---

### Does Helmet replace CORS?

No.

- Helmet secures browser behavior using HTTP headers.
- CORS controls which origins can access your APIs.

They solve different security problems.

---

### Should every Express application use Helmet?

Yes, in most cases. Helmet is lightweight, easy to configure, and provides important browser-side security protections with minimal code.
## 14. Testing Basics

- **Unit tests** — test one small function alone
- **Testing routes** — use `supertest` to fake HTTP requests to your Express app without actually running a live server
- **Mocking** — replace a real database/API call with a fake version so tests run fast and don't depend on real services
- **TDD** — write the test FIRST (it fails), then write just enough code to pass it, then clean it up. "Red, Green, Refactor."

---

## 15. Debugging Tools

- `console.log`, `console.table`, `console.time` — quick and easy checks
- `node --inspect` — lets you attach Chrome DevTools or VS Code debugger to step through code line by line
- `nodemon` — auto-restarts your server every time you save a file change (saves you from restarting manually)
- `NODE_ENV` — a flag that tells your app whether it's in development, testing, or production, so it can behave differently (e.g. more logs in dev)

---

# 🔴 ADVANCED LEVEL
## 16. Node.js Internals

- **V8 Engine (JavaScript Engine)** — the engine that executes JavaScript code inside Node.js. It first compiles your code into machine code using **Ignition (Interpreter)** for fast startup, then uses **TurboFan (Optimizing Compiler)** to optimize frequently executed ("hot") functions for better performance. If the code's behavior changes unexpectedly (different data types, object shapes, etc.), V8 **de-optimizes** that function and recompiles it, which can temporarily reduce performance.

- **Just-In-Time (JIT) Compilation** — instead of interpreting JavaScript line by line forever, V8 compiles JavaScript into native machine code while the program is running, making execution much faster than traditional interpreters.

- **Garbage Collection (GC)** — automatically frees memory occupied by objects that are no longer used by the application, preventing memory leaks. V8 uses a **Generational Garbage Collector**, where short-lived objects are cleaned frequently (Young Generation) and long-lived objects are cleaned less often but more thoroughly (Old Generation).

- **Heap Memory** — the area where JavaScript objects, arrays, and functions are stored. Large objects remain here until the Garbage Collector determines they are no longer reachable.

- **Stack Memory** — stores function calls, local variables, and execution contexts. It is very fast but limited in size and follows the Last-In-First-Out (LIFO) principle.

- **Libuv** — the C library that gives Node.js its asynchronous, non-blocking capabilities. It manages the Event Loop, thread pool, timers, asynchronous file operations, networking, and many OS-level tasks behind the scenes.

- **Libuv Thread Pool** — handles operations the operating system cannot perform asynchronously, such as file system operations, DNS lookups, compression, and cryptographic functions. By default, it contains **4 threads**, but the size can be increased using `UV_THREADPOOL_SIZE`.

- **Event Loop** — continuously checks whether synchronous code has finished, processes completed asynchronous operations, executes callbacks from different queues, and keeps Node.js responsive without creating a new thread for every request.

- **Event Loop Phases** — the Event Loop executes work in multiple phases: **Timers → Pending Callbacks → Idle/Prepare → Poll → Check → Close Callbacks**, ensuring asynchronous tasks execute in the correct order.

- **Call Stack** — stores the currently executing functions. JavaScript executes one function at a time, pushing new function calls onto the stack and removing them when they complete.

- **Callback Queue (Macrotask Queue)** — stores completed asynchronous callbacks such as `setTimeout()`, `setInterval()`, and I/O operations until the Event Loop moves them to the Call Stack.

- **Microtask Queue** — stores higher-priority tasks like `Promise.then()`, `catch()`, `finally()`, `queueMicrotask()`, and `process.nextTick()`. The Event Loop always empties the Microtask Queue before processing the next Macrotask.

- **process.nextTick()** — schedules a callback that executes immediately after the current operation finishes, before Promises and before the Event Loop continues to the next phase. Overusing it can block the Event Loop.

- **CPU-bound Tasks** — operations that spend most of their time performing calculations (sorting large arrays, image processing, encryption, machine learning, complex loops). Since JavaScript runs on the main thread, these tasks block the Event Loop and should be moved to Worker Threads or Child Processes.

- **I/O-bound Tasks** — operations that spend most of their time waiting for external resources such as databases, APIs, files, or networks. Node.js handles these extremely efficiently because they do not block the Event Loop.

- **Streams** — process data piece by piece instead of loading everything into memory at once, making them ideal for handling large files, uploads, downloads, and video streaming efficiently.

- **Buffers** — raw binary memory used to store and manipulate binary data such as files, images, videos, TCP packets, and network streams.

- **Child Processes** — create completely separate operating system processes to execute external programs or CPU-intensive tasks without blocking the main Node.js process.
- **Child Processes** — create completely separate operating system processes that run independently of the main Node.js application. They are used to execute external programs, system commands, or CPU-intensive tasks without blocking the main Node.js process. Each Child Process has its own memory, Event Loop, and process ID (PID), making it completely isolated from the parent process.

### Why do we need Child Processes?

Node.js executes JavaScript on a **single main thread**.

If the application performs a heavy task or needs to run another program (Python, Java, FFmpeg, Git, etc.), it can block the Event Loop.

Instead of

```text
Main Process

↓

Heavy Task

↓

Everything waits ❌
```

Use

```text
Main Process

↓

Create Child Process

↙                    ↘

Continue APIs      Heavy Task

↘                    ↙

Receive Result
```

The main process continues handling incoming requests while the child process performs the heavy work independently.

---

### How Child Processes Work

When a Child Process is created:

- A new operating system process starts.
- It has its own memory.
- It has its own Event Loop.
- It runs independently of the parent process.
- The parent and child communicate using **IPC (Inter-Process Communication)**.

If the child process crashes, the parent process usually continues running.

---

### Communication

The parent and child exchange data through messages.

```text
Parent Process

↓

Send Message

↓

Child Process

↓

Do Work

↓

Send Result Back
```

Node.js provides:

- `child.send()`
- `process.send()`
- `child.on("message")`

---

### Four Ways to Create Child Processes

### **spawn()**

- Starts a new process.
- Streams output as it is generated.
- Memory efficient.
- Best for long-running commands or large outputs.

**Example Uses**

- FFmpeg
- Git
- Docker
- Ping
- Running Python scripts

---

### **exec()**

- Executes a command inside a shell.
- Returns the complete output after the command finishes.
- Stores the entire output in memory.

Best for:

- Small commands like:

```bash
ls
pwd
node -v
```

Not recommended for huge outputs because it can consume a lot of memory.

---

### **execFile()**

- Similar to `exec()`.
- Runs an executable file directly.
- Does **not** use a shell.
- Faster and safer than `exec()`.

Best for:

- Running trusted executable files.

---

### **fork()**

- Special version of `spawn()`.
- Used only for starting another **Node.js file**.
- Automatically creates an IPC communication channel between parent and child.

Best for:

- Background jobs
- Scheduled tasks
- Worker services

---

### Advantages

- Does not block the main Node.js process.
- Completely isolated memory.
- Can run programs written in any language.
- If a child crashes, the parent process usually remains unaffected.
- Better security through process isolation.

---

### Disadvantages

- Uses more memory than Worker Threads.
- Creating a new process is slower than creating a Worker Thread.
- Communication is slower because data must be passed using IPC.

---

### Best Use Cases

- Running Python scripts
- Running Java applications
- Image processing with FFmpeg
- Video conversion
- PDF generation
- Git commands
- Docker commands
- Shell scripts
- CPU-intensive background jobs
- Backup scripts

---

### Do NOT Use Child Processes For

- Simple database queries
- HTTP requests
- File reading
- API calls

These are **I/O-bound operations**, and Node.js already handles them efficiently using the Event Loop and Libuv.

---

### Child Processes vs Worker Threads

| Child Processes | Worker Threads |
|-----------------|----------------|
| Separate OS process | Same Node.js process |
| Separate memory | Can share memory |
| Slower communication (IPC) | Faster communication |
| Higher memory usage | Lower memory usage |
| Can run any executable (Python, Java, FFmpeg, etc.) | Only runs JavaScript |
| Better isolation | Better performance for JS computations |

---

### Simple Example

```js
// parent.js
const { fork } = require("child_process");

const child = fork("./worker.js");

child.send(100);

child.on("message", (result) => {
    console.log(result);
});
```

```js
// worker.js
process.on("message", (num) => {
    let sum = 0;

    for (let i = 0; i <= num; i++) {
        sum += i;
    }

    process.send(sum);
});
```

Here:

- The parent process starts a new Node.js process.
- The child process performs the calculation independently.
- The parent process continues serving users without blocking.
- Once the calculation is complete, the child sends the result back.

**Interview Tip:** Use **Worker Threads** for **CPU-intensive JavaScript code** that benefits from shared memory and low overhead. Use **Child Processes** when you need complete isolation or want to run external programs such as Python, Java, FFmpeg, Git, or shell commands.

- **spawn()** — starts a new process and streams its output as it is produced, making it ideal for long-running commands or programs that generate large amounts of data.

- **exec()** — executes a command inside a shell and returns the complete output after the process finishes. Convenient for small outputs, but can consume significant memory and is vulnerable to shell injection if user input is not sanitized.

- **execFile()** — executes a specific executable file directly without creating a shell, making it faster and more secure than `exec()`.

- **fork()** — creates another Node.js process with a built-in communication channel (IPC) between the parent and child process, making it ideal for running separate Node.js scripts.



- **Child Processes vs Worker Threads** — Worker Threads share the same process and consume less memory, making them ideal for heavy JavaScript computations. Child Processes are completely isolated with separate memory, making them better for running external applications, improving fault isolation, or executing non-JavaScript programs.

- **Clustering** — creates multiple Node.js processes (usually one per CPU core) so the application can utilize all available CPU cores. Each worker handles incoming requests independently while sharing the same server port through the primary process.

- **Single-Threaded Event Loop** — although JavaScript execution runs on a single thread, Node.js achieves high concurrency through asynchronous I/O, the Event Loop, and the Libuv thread pool instead of creating one thread per client request.

- **Node.js Process** — every running Node.js application is an operating system process with its own memory space, Event Loop, Heap, Stack, and thread pool.
```

---
- **Worker Threads** — allow JavaScript code to run in parallel threads within the same Node.js process, making them ideal for CPU-intensive tasks without blocking the Event Loop. Unlike the main thread, Worker Threads execute JavaScript independently while still allowing communication with the parent thread through messages or shared memory.

### Why do we need Worker Threads?

Node.js executes JavaScript on a **single main thread**.

For example:

```js
console.log("Start");

// Heavy calculation
for (let i = 0; i < 10_000_000_000; i++) {}

console.log("End");
```

While this loop is running:

- API requests cannot be processed.
- Database responses cannot be handled.
- Other users must wait.
- The Event Loop is blocked.

This is called a **CPU-bound task**.

Worker Threads solve this problem by moving heavy computations to another thread.

---

### How Worker Threads Work

Instead of

```text
Main Thread

↓

Heavy Calculation

↓

Everything waits ❌
```

Use

```text
Main Thread                     Worker Thread

Receive Request      ─────────► Heavy Calculation

Continue Handling APIs ◄──────── Result Returned
```

The main thread remains free to serve other users.

---

### Communication

Worker Threads communicate with the parent thread using **messages**.

```text
Parent Thread

↓

Send Data

↓

Worker Thread

↓

Process Data

↓

Send Result Back
```

Communication is done using:

- `parentPort.postMessage()`
- `worker.postMessage()`
- `worker.on("message")`

---

### Shared Memory

Unlike Child Processes, Worker Threads **can share memory**.

They use:

- `SharedArrayBuffer`
- `Atomics`

This avoids copying large amounts of data between threads, making communication much faster.

---

### Advantages

- Run JavaScript in parallel.
- Prevent Event Loop blocking.
- Better CPU utilization.
- Lower memory usage than Child Processes.
- Faster communication through shared memory.
- Excellent for heavy computations.

---

### Disadvantages

- More complex than normal JavaScript.
- Not useful for I/O operations (Node.js already handles those efficiently).
- Creating many workers increases CPU and memory usage.

---

### Best Use Cases

- Image processing
- Video processing
- Data compression
- Encryption / Decryption
- Machine Learning
- PDF generation
- Excel generation
- Large file parsing
- Sorting huge datasets
- Mathematical calculations

---

### Do NOT Use Worker Threads For

- Database queries
- API calls
- File reading/writing
- HTTP requests
- Socket communication

These are **I/O-bound operations**, and Node.js already handles them efficiently using the Event Loop and Libuv.

---

### Worker Threads vs Child Processes

| Worker Threads | Child Processes |
|----------------|-----------------|
| Run inside the same Node.js process | Run as completely separate OS processes |
| Share memory when needed | Do not share memory |
| Faster communication | Slower communication (IPC) |
| Lower memory usage | Higher memory usage |
| Best for CPU-intensive JavaScript | Best for running external programs or isolated services |

---

### Simple Example

```js
// main.js
const { Worker } = require("worker_threads");

const worker = new Worker("./worker.js");

worker.postMessage(100000000);

worker.on("message", (result) => {
    console.log(result);
});
```

```js
// worker.js
const { parentPort } = require("worker_threads");

parentPort.on("message", (num) => {
    let sum = 0;

    for (let i = 0; i < num; i++) {
        sum += i;
    }

    parentPort.postMessage(sum);
});
```

Here:

- Main Thread receives requests from users.
- Worker Thread performs the heavy calculation.
- Main Thread never blocks and continues serving other requests.

**Interview Tip:** Worker Threads are used for **CPU-bound JavaScript tasks**, while the **Event Loop + Libuv** already handle **I/O-bound tasks** efficiently. If your Node.js server becomes slow because of heavy calculations, move those calculations to Worker Threads instead of running them on the main thread.

## 17. Performance Optimization

**Memory leaks — common causes**
- Global variables that keep growing forever
- Forgotten timers/event listeners never cleared
- Caches with no size limit

Fix by taking "heap snapshots" at different times and comparing what's growing.

**Profiling** — tools like `clinic.js` show you exactly where your app is spending its time/getting stuck.

**Caching** — save frequently-used results so you don't repeat expensive work. Use a simple `Map` for single-server caching, or Redis when you have multiple servers sharing the same cache.

**Load testing** — tools like `k6` or `Artillery` simulate lots of fake users hitting your app to see how much it can handle before breaking.

---

## 18. Scalability & System Design



- **Vertical Scaling (Scale Up)** — increase the power of a single server by adding more CPU, RAM, or storage; simple to implement but limited by the maximum capacity of one machine and creates a single point of failure.

- **Horizontal Scaling (Scale Out)** — increase capacity by adding more servers and distributing traffic between them; provides better scalability and fault tolerance, but requires load balancing and stateless application design.

- **Load Balancing** — distributes incoming requests across multiple servers so no single server becomes overloaded; improves performance, availability, and reliability using tools like Nginx, HAProxy, or AWS ELB.

- **Microservices** — split one large application into small independent services (Users, Orders, Payments, Notifications, etc.); each service can be developed, deployed, and scaled independently, but communication between services becomes more complex.

- **Message Queues (RabbitMQ, Kafka, Amazon SQS)** — allow services to send tasks asynchronously without waiting for another service to process them immediately; commonly used for emails, notifications, image processing, report generation, and handling traffic spikes.

- **Circuit Breaker Pattern** — when a dependent service keeps failing, temporarily stop sending requests to it and return a fallback response instead; periodically test the service and resume traffic once it recovers.

- **Caching (Redis, Memcached)** — store frequently accessed data in memory instead of querying the database every time; improves application speed, reduces database load, and decreases response time.

- **Database Replication** — create multiple copies of the same database where one database handles writes (Primary) and others handle reads (Replicas); improves read performance, availability, and disaster recovery.

- **Database Sharding** — split a large database into multiple smaller databases (shards), each storing a portion of the data; improves scalability and performance for applications with massive datasets.

- **CDN (Content Delivery Network)** — store copies of static files (images, CSS, JavaScript, videos) on servers around the world so users download them from the nearest location; reduces latency and improves website loading speed.

- **API Gateway** — a single entry point for all client requests in a microservices architecture; handles routing, authentication, authorization, rate limiting, logging, and request aggregation.

- **Rate Limiting** — restrict the number of requests a user or IP address can make within a specific time period; protects APIs from abuse, brute-force attacks, and DDoS attempts.

- **Reverse Proxy** — sits between clients and backend servers, forwarding requests while handling SSL termination, caching, compression, security, and load balancing; commonly implemented using Nginx or HAProxy.

- **Session Management** — manages user login sessions across requests; can be stateful (server stores sessions) or stateless (JWT stores user information in tokens), depending on the application's architecture.

- **Sticky Sessions (Session Affinity)** — ensure a user's requests always reach the same server in a load-balanced environment; useful for stateful applications but generally avoided in favor of shared session storage like Redis.

- **Event-Driven Architecture** — services communicate by publishing and subscribing to events instead of calling each other directly; improves scalability, loose coupling, and asynchronous processing.

- **CAP Theorem** — states that a distributed system can guarantee only two of the following three properties at the same time: Consistency, Availability, and Partition Tolerance.

- **Health Checks** — endpoints used by load balancers or orchestration tools to verify whether an application instance is healthy before routing traffic to it; unhealthy servers are automatically removed until they recover.

- **Auto Scaling** — automatically adds or removes servers based on CPU usage, memory usage, or incoming traffic; helps maintain performance while reducing infrastructure costs.

- **Service Discovery** — allows services in a distributed system to automatically find and communicate with each other without hardcoding server addresses; commonly used with Kubernetes, Consul, or Eureka.

- **Failover** — automatically switches traffic to a backup server or database when the primary one becomes unavailable; ensures high availability and minimizes downtime.

- **High Availability (HA)** — design systems with redundant servers, databases, and network components so the application continues working even if one component fails.

- **Disaster Recovery (DR)** — strategies for restoring applications and data after major failures such as hardware crashes, data corruption, or natural disasters; typically involves backups, replication, and recovery plans.
---

## 19. Advanced Async Patterns

**Async generators** — let you produce values one at a time, lazily, useful for things like paginated data:
```js
async function* generateNumbers() {
  yield 1;
  yield 2;
  yield 3;
}
```

**Race conditions** — when two things happen "at the same time" and mess with the same data, causing lost updates (e.g., two people book the last seat at once). Fixed with database locks or version checks.

**Retry with backoff** — if something fails, don't retry instantly — wait a bit longer each time (1s, 2s, 4s...) so you don't overload an already-struggling system.

**AbortController** — lets you cancel an in-progress async task (like cancelling a slow network request after 5 seconds).

---

## 20. Security (Advanced)

- **SQL/NoSQL Injection** — when a hacker manipulates your database query using malicious input; fix it by using parameterized queries, prepared statements, or ORM methods instead of directly inserting user input into queries.

- **XSS (Cross-Site Scripting)** — when a hacker sneaks JavaScript into your website through user input; fix it by escaping/sanitizing anything shown back to users and validating user input.

- **CSRF (Cross-Site Request Forgery)** — tricking a logged-in user's browser into sending unwanted requests to your website without their knowledge; fix it with CSRF tokens, `SameSite` cookies, and origin validation.

- **OAuth 2.0** — a standard protocol that lets users sign in with Google, Facebook, GitHub, etc. without sharing their password with your app.

- **OpenID Connect (OIDC)** — an authentication layer built on OAuth 2.0 that verifies a user's identity and returns their profile information after login.

- **Prototype Pollution** — when a hacker modifies JavaScript's prototype objects through malicious input, changing how objects behave across your application; fix it by validating input and avoiding unsafe object merging.

- **ReDoS (Regular Expression Denial of Service)** — when a hacker sends specially crafted input that makes a regular expression take a very long time to execute, slowing down or freezing your server; fix it by writing efficient regex patterns and limiting input length.
---

## 21. type of authentication in detail


# Types of Authentication (Detailed)

Authentication is the process of **verifying the identity of a user or system** before allowing access to resources. Different authentication methods are used depending on the application's security requirements, scalability, and architecture.

---

## **1. Session-Based Authentication**

- **Definition** — after a user successfully logs in, the server creates a unique session and stores it (in memory, a database, or Redis). A **Session ID** is sent to the browser as a cookie, and every future request includes that Session ID. The server looks up the session to identify the user.

### How it works

```text
User Login
      │
      ▼
Server verifies credentials
      │
      ▼
Creates Session
(Session ID: abc123)
      │
      ▼
Stores Session on Server
      │
      ▼
Sends Cookie(Session ID)
      │
      ▼
Browser stores Cookie
      │
      ▼
Every request automatically sends Session ID
      │
      ▼
Server validates Session
```

### Advantages

- Very secure because user data remains on the server.
- Easy to invalidate by deleting the session.
- Ideal for traditional web applications.

### Disadvantages

- Requires server memory or Redis.
- Difficult to scale without shared session storage.

### Best Used For

- Banking websites
- Admin dashboards
- Traditional web applications

---

## **2. JWT (JSON Web Token) Authentication**

- **Definition** — after successful login, the server generates a digitally signed JSON Web Token (JWT) containing user information (claims). The client stores the token and sends it with every request. Since the server does not store sessions, JWT authentication is **stateless**.

### JWT Structure

```text
Header.Payload.Signature
```

Example

```text
eyJhbGciOiJIUzI1NiIs...

eyJ1c2VySWQiOjEyMy...

hR6K7Ab...
```

### How it works

```text
User Login
      │
      ▼
Server verifies credentials
      │
      ▼
Creates JWT
      │
      ▼
Returns Token
      │
      ▼
Client stores Token
      │
      ▼
Authorization: Bearer JWT
      │
      ▼
Server verifies Signature
```

### Advantages

- Stateless (no server-side session storage).
- Excellent for REST APIs.
- Easy to scale across multiple servers.

### Disadvantages

- Difficult to revoke before expiration.
- Larger than Session IDs.
- Must be stored securely.

### Best Used For

- REST APIs
- Mobile apps
- MERN applications
- Microservices

---

## **3. OAuth 2.0**

- **Definition** — an authorization framework that allows users to grant your application limited access to their Google, GitHub, Facebook, or Microsoft account **without sharing their password**.

### How it works

```text
User

↓

Click "Login with Google"

↓

Google Login

↓

Google verifies user

↓

Google returns Access Token

↓

Your application accesses Google APIs
```

### Advantages

- Users never share passwords.
- Secure third-party login.
- Supports delegated access.

### Disadvantages

- More complex to implement.
- Requires external providers.

### Best Used For

- Login with Google
- Login with GitHub
- Login with Facebook

---

## **4. OpenID Connect (OIDC)**

- **Definition** — an authentication layer built on top of OAuth 2.0 that verifies the user's identity and returns profile information such as name, email, and profile picture.

### OAuth vs OpenID Connect

OAuth → Authorization

OpenID Connect → Authentication

### Returns

- ID Token
- User Profile
- Email
- Name

### Best Used For

- Single Sign-On
- Google Login
- Microsoft Login

---

## **5. API Key Authentication**

- **Definition** — clients authenticate by sending a unique API Key with every request. The server validates the key before allowing access.

### Example

```http
GET /users

x-api-key: 123456789abcdef
```

### Advantages

- Very simple.
- Easy to implement.
- Good for server-to-server communication.

### Disadvantages

- Does not identify individual users.
- Difficult to rotate securely.
- Less secure if exposed.

### Best Used For

- Public APIs
- Internal APIs
- Third-party integrations

---

## **6. Basic Authentication**

- **Definition** — the client sends a username and password with every request using the `Authorization` header. The credentials are Base64 encoded (not encrypted), so HTTPS is essential.

### Example

```http
Authorization: Basic dXNlcjpwYXNzd29yZA==
```

### Advantages

- Extremely simple.
- Built into HTTP.

### Disadvantages

- Sends credentials with every request.
- Insecure without HTTPS.

### Best Used For

- Internal tools
- Testing APIs

---

## **7. Bearer Token Authentication**

- **Definition** — the client sends an access token in the `Authorization` header. The server verifies the token before processing the request.

### Example

```http
Authorization: Bearer eyJhbGc...
```

### Advantages

- Stateless.
- Widely supported.
- Ideal for APIs.

### Disadvantages

- Token theft grants access until it expires.
- Requires secure storage.

### Best Used For

- REST APIs
- Mobile apps
- Microservices

---

## **8. Cookie-Based Authentication**

- **Definition** — authentication data is stored inside browser cookies. Browsers automatically include cookies with every request to the same domain. Cookies are often used with session-based authentication or to store JWTs as **HttpOnly** cookies.

### Advantages

- Automatic handling by browsers.
- Can be protected with `HttpOnly`, `Secure`, and `SameSite` attributes.

### Disadvantages

- Vulnerable to CSRF if not configured properly.
- Mainly suited for browser-based applications.

### Best Used For

- Traditional web applications
- Session authentication
- JWT in HttpOnly cookies

---

## **9. Multi-Factor Authentication (MFA / 2FA)**

- **Definition** — requires users to verify their identity using two or more authentication factors, making accounts much harder to compromise.

### Authentication Factors

- Something you know → Password
- Something you have → Phone, Security Key
- Something you are → Fingerprint, Face ID

### Example

```text
Password

↓

OTP

↓

Login Successful
```

### Advantages

- Much higher security.
- Protects against stolen passwords.

### Best Used For

- Banking
- Enterprise systems
- Admin panels

---

## **10. Biometric Authentication**

- **Definition** — verifies users using unique biological characteristics such as fingerprints, facial recognition, iris scans, or voice recognition.

### Examples

- Fingerprint
- Face ID
- Retina Scan
- Voice Recognition

### Advantages

- Very convenient.
- Difficult to forge.

### Disadvantages

- Requires compatible hardware.
- Biometric data cannot easily be changed if compromised.

### Best Used For

- Smartphones
- Banking apps
- Secure devices

---

## **11. Passwordless Authentication**

- **Definition** — users authenticate without a password using methods such as magic links, one-time passwords (OTP), passkeys, or biometric verification.

### Example

```text
Enter Email

↓

Magic Link

↓

Click Link

↓

Logged In
```

### Advantages

- Better user experience.
- Eliminates weak passwords.
- Reduces phishing risks.

### Best Used For

- Modern SaaS applications
- Consumer apps

---

## **12. Certificate-Based Authentication**

- **Definition** — users or devices authenticate using digital certificates instead of passwords. The server verifies the certificate before granting access.

### Best Used For

- Enterprise networks
- VPNs
- Machine-to-machine communication

---

## **13. SAML Authentication**

- **Definition** — an XML-based authentication standard used for **Single Sign-On (SSO)**, allowing users to log in once and access multiple enterprise applications.

### Example

```text
Login Once

↓

Company Portal

↓

Salesforce

↓

Jira

↓

Slack
```

### Best Used For

- Enterprise applications
- Corporate environments

---

## **14. LDAP Authentication**

- **Definition** — authenticates users against a centralized directory service such as Microsoft Active Directory, allowing organizations to manage employee credentials from one place.

### Best Used For

- Corporate networks
- Internal employee systems

---

## **15. Passkey Authentication (FIDO2/WebAuthn)**

- **Definition** — authenticates users using cryptographic key pairs stored securely on their device instead of passwords. Resistant to phishing and credential theft.

### Advantages

- No passwords to remember.
- Strong phishing protection.
- Fast and secure login.

### Best Used For

- Modern web applications
- Mobile applications
- Enterprise authentication

---

## **16. Single Sign-On (SSO)**

- **Definition** — allows users to log in once and access multiple related applications without signing in again. Commonly implemented using OAuth 2.0, OpenID Connect, or SAML.

### Example

```text
Login to Google

↓

Gmail

↓

YouTube

↓

Google Drive

↓

Google Docs
```

### Advantages

- Better user experience.
- Fewer passwords.
- Centralized authentication.

### Best Used For

- Enterprise software
- Large organizations
- SaaS platforms

## **HTTP Cookie-Based Authentication**

- **Definition** — the server stores authentication information inside an HTTP cookie and sends it to the browser using the `Set-Cookie` header. The browser automatically stores the cookie and includes it in every request to the same domain using the `Cookie` header. The cookie can contain either a **Session ID** or a **JWT**, making it one of the most common authentication methods for web applications.

### How it works

```text
User Login
      │
      ▼
Server verifies credentials
      │
      ▼
Creates Session ID or JWT
      │
      ▼
Set-Cookie Header
      │
      ▼
Browser stores Cookie
      │
      ▼
Every request automatically sends Cookie
      │
      ▼
Server verifies Session or JWT
      │
      ▼
Access Granted
```

### Example

**Server Response**

```http
Set-Cookie: token=eyJhbGc...; HttpOnly; Secure; SameSite=Lax
```

**Next Request**

```http
GET /profile HTTP/1.1
Cookie: token=eyJhbGc...
```

The browser automatically sends the cookie with every request.

### Cookie Security Options

- **HttpOnly** — prevents JavaScript from accessing the cookie, protecting it from XSS attacks.
- **Secure** — ensures the cookie is sent only over HTTPS connections.
- **SameSite** — helps prevent CSRF attacks by controlling when cookies are included in cross-site requests (`Strict`, `Lax`, or `None`).
- **Max-Age / Expires** — determines how long the cookie remains valid before it expires.

### Advantages

- Browser automatically manages cookies.
- Can securely store Session IDs or JWTs.
- HttpOnly cookies protect against XSS.
- Widely used in modern web applications.

### Disadvantages

- Cookies are automatically sent with every request, making CSRF protection necessary.
- Mainly suitable for browser-based applications.
- Cookie size is limited (about 4 KB).

### Best Used For

- Web applications
- MERN applications
- Banking websites
- Admin dashboards
- E-commerce websites
## 21. Advanced Database Concepts

**ACID** (guarantees a good database gives you):
**ACID (guarantees a reliable database transaction)** — a set of four properties that ensure every database transaction is processed safely, accurately, and reliably, even when multiple users access the database simultaneously or the system crashes.

- **Atomicity** — ensures a transaction is completed entirely or not executed at all ("all or nothing"); if any step fails, the entire transaction is rolled back so the database is never left in a partially updated state.

  **Example:** Transferring ₹1000 from Account A to Account B. If money is deducted from A but adding it to B fails, the deduction is rolled back, so neither account is changed.

- **Consistency** — ensures every transaction moves the database from one valid state to another while following all rules, constraints, and relationships; invalid data is never permanently stored.

  **Example:** If a table requires every employee to have a unique email address, the database rejects any transaction that tries to insert a duplicate email.

- **Isolation** — ensures multiple transactions running at the same time do not interfere with each other; each transaction behaves as if it is running alone, preventing inconsistent or corrupted data.

  **Example:** Two users try to purchase the last product at the same time. Isolation ensures only one transaction succeeds, preventing both users from buying the same item.

- **Durability** — ensures that once a transaction is successfully committed, the data is permanently saved and will not be lost even if the database server crashes, loses power, or restarts immediately afterward.

  **Example:** A customer successfully completes a payment, and the database confirms the transaction. Even if the server crashes one second later, the payment record remains safely stored and is not lost.

**Replicas & Sharding** — replicas = copies of the DB for handling more reads; sharding = splitting data across multiple databases so no single one gets overloaded.

---

## 22. Testing (Advanced)

- **Unit** → smallest piece alone
- **Integration** → multiple pieces working together
- **E2E (End-to-End)** → testing the whole app like a real user would
- **Contract testing** — makes sure one service's API still matches what other services expect from it (important in microservices)

---

## 23. Deployment & DevOps

- **Docker** — packages your app + everything it needs into one portable "container" that runs the same everywhere.
- **PM2** — a tool that keeps your Node app running, restarts it if it crashes, and can run it across multiple CPU cores easily.
- **Health checks** — a simple endpoint (like `/healthz`) that tells a load balancer "yes, I'm alive and ready."
- **Graceful shutdown** — when your server needs to stop, finish handling current requests first before actually closing, so users don't get cut off mid-request.

---

## 24. GraphQL

**REST vs GraphQL**
- REST → fixed endpoints, sometimes you get more/less data than you need
- GraphQL → one endpoint, client asks for exactly the fields it wants, in one request

**N+1 problem** — accidentally making a separate database query for every single item in a list, instead of one combined query. Fixed with a tool called **DataLoader**, which batches these together.

---

## 25. TypeScript with Node.js

TypeScript adds "types" on top of JavaScript, catching mistakes before your code even runs. You compile TypeScript into regular JavaScript before deploying it (running TypeScript directly in production is usually avoided for performance reasons).

---

## 26. Extra Advanced Topics

- **Closures causing memory leaks** — if a function "remembers" a big variable from outside it forever, that memory can never be freed.
- **`this` in callbacks** — regular functions lose their original `this` when passed around as callbacks; arrow functions fix this automatically.
- **WebSockets** — unlike normal request/response, both sides (server & client) can send data to each other anytime — used for chat apps, live dashboards, etc.
- **Rate limiting algorithms**: Token Bucket (allows short bursts), Sliding Window (smooths out traffic spikes).
- **Idempotency** — doing the same action multiple times gives the same result (important so retrying a failed payment doesn't charge someone twice).

---

# 💻 COMMON CODING QUESTIONS (Explained Simply)

### 1. Debounce vs Throttle
- **Debounce** → wait until the user STOPS doing something (typing), then run once. Good for search boxes.
- **Throttle** → run at most once every fixed interval, no matter how often it's triggered. Good for scroll events.

### 2. Rate Limiter (Token Bucket)
Imagine a bucket that slowly refills with tokens. Every request uses 1 token. If the bucket is empty, the request is blocked until it refills a bit.

### 3. LRU Cache
A cache with a limited size — when it's full, it throws away the **Least Recently Used** item to make room for new ones. Built easily using a JS `Map` (which remembers insertion order).

### 4. Sequential vs Parallel Async
- Sequential → do one task, wait, then the next (needed when later tasks depend on earlier ones)
- Parallel → fire off everything at once using `Promise.all()` (faster, but only when tasks don't depend on each other)

### 5. Pub/Sub with EventEmitter
One event can trigger multiple independent reactions:
```js
bus.on('order:created', () => console.log('Send email'));
bus.on('order:created', () => console.log('Update inventory'));
bus.emit('order:created', { id: 123 });
```

### 6. Custom Middleware Example
A simple middleware that logs how long each request took to process.

### 7. Fixing a Memory Leak
If you have a `setInterval` that keeps adding data to an array forever without clearing it, you must either clear the interval when done, or limit how big that array is allowed to grow.

### 8. The Classic Event Loop Question
```js
console.log('1: start');
setTimeout(() => console.log('2: setTimeout'), 0);
Promise.resolve().then(() => console.log('3: promise'));
process.nextTick(() => console.log('4: nextTick'));
console.log('5: end');
```
**Output:** `1 → 5 → 4 → 3 → 2`
(Normal code first, then nextTick, then Promises, then finally setTimeout.)

---

## 💡 How to Use This Guide
- **For beginner interviews** → focus on sections 1–6, plus a simple explanation of the event loop.
- **For mid-level interviews** → know sections 7–15 well, especially Express, async patterns, and be ready to solve the event-loop ordering question live.
- **For senior interviews** → know sections 16–26 plus the coding questions — you'll likely also get a system design or live coding round.

Good luck! 🚀