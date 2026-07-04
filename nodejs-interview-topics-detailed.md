# Node.js Interview Guide — Detailed Answers (Beginner to Advanced)

---

# BEGINNER LEVEL

## 1. Node.js Fundamentals

**What is Node.js and why use it?**
Node.js is a JavaScript runtime built on Google's V8 engine that lets you run JavaScript outside the browser, on a server. It's single-threaded but uses an event-driven, non-blocking I/O model, making it efficient for I/O-heavy applications (APIs, real-time apps, streaming). Instead of a new thread per request, Node.js handles many connections on one thread via an event loop, delegating expensive I/O to the system/kernel and picking up results via callbacks.

Why use it:
- Same language (JavaScript) on frontend and backend
- Huge ecosystem (npm)
- Excellent for I/O-bound, real-time apps (chat, streaming, APIs)
- Non-blocking I/O gives high throughput with low memory overhead vs thread-per-request models

Not ideal for heavy CPU-bound work (image/video processing) unless offloaded to worker threads or another service, since that blocks the single main thread.

**Node.js vs browser JavaScript**
- No DOM/`window`/`document` in Node.js — instead `global`, `process`, `require`, `module`.
- Node.js exposes OS-level APIs: file system, network sockets, child processes.
- Module systems differ: browsers use `<script>`/ES modules; Node.js historically used CommonJS, now also supports ES modules.
- Security model differs — Node.js code runs with the OS user's permissions (no browser sandbox).

**npm and package.json**
`package.json` is the manifest: metadata, scripts, dependencies.
- `dependencies`: runtime packages (e.g. `express`)
- `devDependencies`: dev-only packages (e.g. `jest`, `nodemon`)
- `scripts`: shortcuts like `"start": "node index.js"`
- Semver `MAJOR.MINOR.PATCH`:
  - `^4.18.2` → compatible minor/patch updates (up to but excluding 5.0.0)
  - `~4.18.2` → patch-level updates only (up to but excluding 4.19.0)
  - Exact `4.18.2` → locked

`package-lock.json` pins exact resolved versions (incl. transitive deps) for reproducible installs.

**Node.js REPL**
Read-Eval-Print-Loop — running `node` with no args opens an interactive shell for quick experiments/debugging.

**Global objects**
- `global`: global namespace object (loosely like `window`). Top-level `var` in a module does NOT attach to `global` (unlike browsers).
- `process`: info/control over the current process — `process.env`, `process.argv`, `process.exit()`, `process.on(...)`.
- `__dirname`/`__filename`: absolute paths (CommonJS only).
- `module`/`exports`: what the current module exposes.
- `require`: imports modules (CommonJS).

---

## 2. Modules

**CommonJS**
Each file is its own module/scope. Export with `module.exports`, import with `require`. Loads synchronously; cached after first `require`.

```js
// math.js
function add(a, b) { return a + b; }
module.exports = { add };

// app.js
const { add } = require('./math');
console.log(add(2, 3)); // 5
```

**Built-in modules**: `fs` (files), `path` (paths), `os` (OS info), `http`/`https` (servers/requests), `events` (EventEmitter), `url` (URL parsing), `util` (promisify, inspect, etc.)

**ES Modules**
Enabled via `.mjs` or `"type": "module"`. Uses `import`/`export`, statically analyzable (tree-shaking), dynamic `import()` returns a Promise. No `require`/`__dirname` natively — derive from `import.meta.url`.

**CommonJS vs ESM**

| Aspect | CommonJS | ESM |
|---|---|---|
| Syntax | require/module.exports | import/export |
| Loading | Synchronous | Async, statically analyzed |
| Top-level `this` | module.exports | undefined |
| Tree-shaking | No | Yes |

**Module caching**
First `require` executes and caches `module.exports` by resolved path; later `require`s reuse the cache — modules act as singletons unless you export a factory function.

---

## 3. File System (fs module)

**Sync vs async**
- `fs.readFileSync` blocks the event loop until done — OK for CLI startup scripts, bad for servers under load.
- `fs.readFile(path, cb)` is non-blocking, uses libuv's thread pool.
- `fs.promises.readFile` / `fs/promises` gives a Promise API for `async`/`await`.

```js
const fs = require('fs');
fs.readFile('data.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

**Streams basics**: `fs.createReadStream()`/`createWriteStream()` process files in chunks instead of loading everything into memory — essential for large files.

**Directories**: `fs.mkdir`, `fs.readdir`, `fs.unlink`.

**Path utilities**: `path.join('/a','b','c')` → `/a/b/c`; `path.resolve('b','c')` → absolute path from cwd; `path.basename('/a/b/file.txt')` → `file.txt`.

---

## 4. Basic HTTP & Servers

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

- `req` is a readable stream; `res` is a writable stream (`res.end()` completes it).
- Manual routing = checking `req.method`/`req.url` yourself, which is why frameworks like Express exist.
- Common status codes: 200, 201, 204, 400, 401, 403, 404, 500.

---

## 5. Asynchronous Programming Basics

**Callback hell**: deeply nested callbacks are hard to read/maintain and duplicate error handling at every level.

**Promises**: represent a value available now/later/never; states pending/fulfilled/rejected. Enable chaining and composition (`Promise.all`, etc.) instead of nesting.

**async/await**: syntactic sugar over Promises so async code reads like sync code; `await` pauses the async function (without blocking the event loop) until the Promise settles.

**Error-first callback pattern**: `callback(err, data)` — Node convention (not enforced by the language) used throughout core APIs and most libraries.

---

## 6. NPM Ecosystem Basics

`package.json` declares intended ranges; `package-lock.json` pins exact resolved versions for reproducibility. Semver ranges control auto-update behavior. Publishing requires a unique `name`+`version` and `npm publish`; published versions are immutable.

---

# INTERMEDIATE LEVEL

## 7. Event Loop & Concurrency Model (Core Topic — Always Asked)

**What is the Event Loop?**
The event loop is the mechanism that lets Node.js perform non-blocking I/O despite JavaScript being single-threaded. It continuously checks the call stack and various queues, executing callbacks when the stack is empty. Node.js delegates I/O operations (file reads, network requests, timers) to the OS kernel or libuv's thread pool, and when those operations complete, their callbacks are queued to run on the main thread via the event loop.

**Phases of the Event Loop** (each phase has its own FIFO queue of callbacks):
1. **Timers** — executes callbacks scheduled by `setTimeout`/`setInterval` whose threshold has elapsed.
2. **Pending callbacks** — executes I/O callbacks deferred to the next loop iteration (some system-level ones).
3. **Idle, prepare** — internal use only.
4. **Poll** — retrieves new I/O events; executes I/O-related callbacks (e.g., file read completions). If no timers are pending, this phase can block waiting for callbacks.
5. **Check** — executes `setImmediate()` callbacks, right after the poll phase.
6. **Close callbacks** — e.g., `socket.on('close', ...)`.

Between each phase transition (and after each callback), Node processes the **microtask queue** in full.

**Call stack, callback queue, microtask vs macrotask queue**
- Call stack: where synchronous JS execution happens.
- Macrotasks: `setTimeout`, `setInterval`, `setImmediate`, I/O callbacks — each tied to an event loop phase.
- Microtasks: Promise callbacks (`.then`/`.catch`/`.finally`) and `process.nextTick()` — these run **before** the event loop proceeds to the next phase, and even before the next macrotask, no matter how many microtasks are queued (a microtask can queue another microtask, delaying macrotasks indefinitely — a real starvation risk in poorly written code).
- Priority order: `process.nextTick()` queue is drained first, then the Promise microtask queue, then the loop moves to the next phase/macrotask.

**process.nextTick() vs setImmediate() vs setTimeout(fn, 0)**
```js
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('promise'));
console.log('sync');
```
Typical output: `sync` → `nextTick` → `promise` → then `timeout`/`immediate` in an order that depends on context (inside an I/O callback, `setImmediate` reliably fires before `setTimeout`; at the top level, order can vary based on process startup timing).
- `process.nextTick()`: runs immediately after the current operation, before the event loop continues — even before Promise microtasks. Overusing it can starve I/O.
- `setImmediate()`: runs in the "check" phase, after the poll phase — good for "run this right after I/O".
- `setTimeout(fn, 0)`: runs in the "timers" phase; the delay is a *minimum*, not a guarantee (actual delay may be longer under load).

**Libuv and the thread pool**
Libuv is the C library underlying Node.js that implements the event loop and provides a thread pool (default size 4) for operations that can't be done asynchronously at the OS level on all platforms (e.g., some `fs` operations, DNS lookups via `dns.lookup`, `crypto.pbkdf2`). Network I/O (sockets) typically doesn't need the thread pool because the OS itself provides async APIs (epoll/kqueue/IOCP); file I/O often does need it because many OSes lack good async file APIs. You can tune the pool size via `process.env.UV_THREADPOOL_SIZE` (must be set before any thread-pool-using work starts).

**Blocking vs non-blocking, and how blocking hurts you**
A CPU-heavy synchronous operation (e.g., a large synchronous loop, `JSON.parse` on a huge payload, `crypto.pbkdf2Sync`) blocks the single main thread, meaning no other requests can be processed until it finishes. This is one of the most common Node.js performance pitfalls in production and a frequent interview follow-up: "how would you fix this?" — Answer: offload to worker threads, a child process, a queue/separate service, or use the async/streaming variant of the API.

---

## 8. Streams & Buffers

**Types of Streams**
- **Readable**: source of data you can read from (e.g., `fs.createReadStream`, incoming HTTP request).
- **Writable**: destination you can write data to (e.g., `fs.createWriteStream`, outgoing HTTP response).
- **Duplex**: both readable and writable, independent of each other (e.g., a TCP socket).
- **Transform**: a duplex stream that modifies data as it passes through (e.g., `zlib.createGzip()`).

**Piping**
`readable.pipe(writable)` automatically manages reading from source and writing to destination, including flow control:
```js
const fs = require('fs');
const zlib = require('zlib');
fs.createReadStream('input.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('input.txt.gz'));
```

**Backpressure**
If a writable stream can't consume data as fast as a readable stream produces it, data would pile up in memory. `pipe()` handles this automatically by pausing the readable stream when the writable's internal buffer is full (`write()` returns `false`) and resuming once it drains (`drain` event). When manually piping data (not using `.pipe()`), you must implement this yourself by checking the return value of `write()`.

**Buffers**
A `Buffer` is a fixed-size chunk of raw binary data allocated outside the V8 heap, used for handling binary data (file contents, network packets) before it's decoded into a string. `Buffer.from('hello')` creates a buffer from a string; `Buffer.alloc(10)` allocates a zeroed 10-byte buffer. Buffers exist because JS strings are UTF-16 by default and inefficient/unsuitable for raw binary manipulation.

---

## 9. Express.js / Web Frameworks

**Setup and routing**
```js
const express = require('express');
const app = express();
app.use(express.json()); // built-in body parser middleware

app.get('/users/:id', (req, res) => {
  res.json({ id: req.params.id });
});

app.listen(3000);
```

**Middleware**
A middleware function has access to `(req, res, next)` and can modify req/res, end the request, or call `next()` to pass control to the next handler. Middleware runs in the order it's registered.
- Built-in: `express.json()`, `express.static()`
- Third-party: `cors`, `morgan`, `helmet`
- Custom:
```js
function logger(req, res, next) {
  console.log(`${req.method} ${req.url}`);
  next();
}
app.use(logger);
```

**Error-handling middleware**
Defined with **four** arguments `(err, req, res, next)` — Express recognizes it by that arity:
```js
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong' });
});
```
Must be registered last, after all routes.

**Request/response lifecycle**: request comes in → passes through middleware stack in order → matched route handler executes → response sent → if an error is thrown/passed to `next(err)`, it skips to error-handling middleware.

**RESTful API design**: use nouns for resources (`/users`, `/orders`), HTTP verbs for actions (GET=read, POST=create, PUT=full update, PATCH=partial update, DELETE=remove), proper status codes, statelessness (no server-side session state that couples requests together), and versioning (`/api/v1/...`).

---

## 10. Asynchronous Programming (Deep Dive)

**Promise combinators**
- `Promise.all([...])`: resolves when ALL resolve, rejects immediately if ANY rejects (fail-fast).
- `Promise.allSettled([...])`: waits for all to settle (resolve or reject), never short-circuits — gives you an array of `{status, value|reason}`.
- `Promise.race([...])`: settles as soon as the FIRST promise settles (resolve or reject) — useful for timeouts.
- `Promise.any([...])`: resolves as soon as the FIRST one resolves; rejects only if ALL reject (with an `AggregateError`).

**util.promisify**
Converts an error-first callback-style function into one that returns a Promise:
```js
const util = require('util');
const fs = require('fs');
const readFileAsync = util.promisify(fs.readFile);
```

**EventEmitter**
Foundation of Node's event-driven architecture. Many core objects (streams, servers) extend `EventEmitter`.
```js
const EventEmitter = require('events');
class MyEmitter extends EventEmitter {}
const emitter = new MyEmitter();
emitter.on('greet', (name) => console.log(`Hello, ${name}`));
emitter.emit('greet', 'World');
emitter.once('init', () => console.log('runs only once'));
```
Note: by default, if an `EventEmitter` has more than 10 listeners for a single event, Node warns of a possible memory leak (`MaxListenersExceededWarning`) — adjustable via `emitter.setMaxListeners(n)`.

---

## 11. Database Integration

- **SQL** (PostgreSQL, MySQL): use a driver (`pg`, `mysql2`) directly for raw queries, or an ORM/query builder (Sequelize, Prisma, TypeORM, Knex) for higher-level modeling, migrations, and validation.
- **NoSQL** (MongoDB): typically via Mongoose (schema-based ODM) or the native MongoDB driver.
- **Connection pooling**: instead of opening a new DB connection per request (expensive), a pool maintains a set of reusable connections; a request borrows one, uses it, and returns it to the pool.
- **CRUD**: Create/Read/Update/Delete — the basic operations any data layer exposes.
- **Transactions**: group multiple operations so they either all succeed or all roll back together (important for operations like "transfer money between two accounts" where partial failure would corrupt data).

---

## 12. Error Handling

**Approaches**
- `try/catch` for synchronous code and `async/await`.
- `.catch()` for Promise chains.
- Error-first callbacks: check `if (err) { ... }` at the top of every callback.

**Custom error classes**
```js
class NotFoundError extends Error {
  constructor(message) {
    super(message);
    this.name = 'NotFoundError';
    this.statusCode = 404;
  }
}
```
This lets you distinguish error types downstream (e.g., in centralized error-handling middleware) and attach metadata like HTTP status codes.

**Global error handling**
```js
process.on('uncaughtException', (err) => {
  console.error('Uncaught exception:', err);
  process.exit(1); // recommended: don't keep running in an unknown state
});
process.on('unhandledRejection', (reason) => {
  console.error('Unhandled rejection:', reason);
});
```
These are safety nets, not substitutes for proper error handling — an app that relies on `uncaughtException` to "keep going" is usually in an unpredictable state and should typically restart (which is why process managers like PM2 pair well with this pattern).

**Operational vs programmer errors**
- Operational errors: expected failure modes of a working system (invalid user input, failed DB connection, timeout) — should be handled gracefully, often recoverable.
- Programmer errors: bugs (calling a function with the wrong type, undefined is not a function) — these indicate the program is in an unknown state; generally best to crash and restart rather than trying to "handle" them.

---

## 13. Authentication & Security Basics

**Session-based vs token-based auth**
- Session-based: server stores session state (often in a store like Redis), client holds a session ID cookie; server looks up session on each request. Easy to revoke, but requires server-side storage/state.
- Token-based (JWT): server issues a signed token containing claims; client sends it (usually in an `Authorization: Bearer <token>` header) on each request; server verifies the signature without needing to store session state. Stateless and scales well horizontally, but harder to revoke before expiry (mitigated with short-lived tokens + refresh tokens, or a blocklist).

**JWT structure**: `header.payload.signature`, each Base64URL-encoded. The header specifies the algorithm (e.g., HS256, RS256); the payload holds claims (user id, roles, expiry `exp`); the signature is generated over header+payload using a secret (HMAC) or private key (RSA/ECDSA), and lets the server verify the token hasn't been tampered with.

**Password hashing**: never store plaintext passwords. Use `bcrypt` (or `argon2`), which is deliberately slow and includes a per-password salt, making brute-force and rainbow-table attacks impractical.

**CORS**: browsers block cross-origin requests by default unless the server responds with appropriate `Access-Control-Allow-Origin` (and related) headers. In Express, the `cors` middleware package configures this.

**Environment variables / dotenv**: keep secrets (DB passwords, API keys) out of source code by loading them from a `.env` file (via the `dotenv` package) into `process.env` at runtime; `.env` should never be committed to version control.

**Basic security practices**:
- `helmet`: sets various security-related HTTP headers (prevents clickjacking, MIME sniffing, etc.).
- Rate limiting (`express-rate-limit`): prevents brute-force/DoS by capping requests per IP/time window.
- Input validation/sanitization (`joi`, `zod`, `express-validator`): reject malformed/malicious input before it reaches business logic.

---

## 14. Testing Basics

**Unit testing**: test individual functions/modules in isolation, typically with Jest or Mocha+Chai.
```js
test('adds 2 + 3 to equal 5', () => {
  expect(add(2, 3)).toBe(5);
});
```

**Testing routes**: use `supertest` alongside Jest/Mocha to make HTTP requests against your Express app in tests without needing a running server process.

**Mocking**: replace real dependencies (DB calls, external APIs) with fake implementations so tests are fast, deterministic, and don't hit real services — Jest's `jest.mock()`, or libraries like `sinon`.

**TDD**: write a failing test first, write the minimum code to make it pass, then refactor — "red, green, refactor."

---

## 15. Debugging & Tooling

- `console.log/error/table/time/trace` — quick and common, but limited for complex issues.
- `node --inspect index.js` starts the app with a debugging protocol you can attach Chrome DevTools (`chrome://inspect`) or VS Code's debugger to — set breakpoints, step through code, inspect variables.
- `nodemon`: watches files and auto-restarts the server on changes during development.
- Environment-specific config: use `NODE_ENV` (`development`/`production`/`test`) to toggle behavior (e.g., verbose logging in dev, minimal in prod), often combined with `.env` files per environment.

---

# ADVANCED LEVEL

## 16. Node.js Internals & Architecture

**V8 engine internals**: V8 compiles JS to machine code using a JIT (Just-In-Time) compiler pipeline — an initial fast, unoptimized compile (Ignition, the interpreter) followed by optimization (TurboFan) for "hot" functions that run frequently, using assumptions about argument types (inline caching, hidden classes). If those assumptions are violated later (e.g., a function is called with inconsistently shaped objects), V8 "deoptimizes" back to the interpreter — a common cause of unexpected performance cliffs. Garbage collection uses a generational approach: a small, fast "young generation" (Scavenger) for short-lived objects, and a slower "old generation" (mark-sweep-compact) for long-lived ones.

**Libuv thread pool**: used for filesystem operations, DNS lookups (`dns.lookup`, not `dns.resolve`), and some crypto functions. Default size is 4; can be increased via `process.env.UV_THREADPOOL_SIZE = 8` (set before any thread-pool work is scheduled, ideally as the very first line). Increasing it can help apps that do heavy concurrent file I/O or CPU-bound crypto, but doesn't help pure network I/O bottlenecks, and won't help CPU-bound JS work run on the main thread (worker threads are needed for that).

**CPU-bound vs I/O-bound**: I/O-bound work (network calls, file access) is delegated to the OS/thread pool and doesn't block the main thread — Node excels here. CPU-bound work (heavy computation, big loops, synchronous JSON parsing of huge payloads) runs entirely on the main JS thread and blocks everything else until done — Node is a poor fit unless you offload this work.

**Child processes**
- `spawn(command, args)`: launches a new process, streams stdout/stderr, good for long-running processes or large output (data streamed in chunks).
- `exec(command)`: runs a command in a shell, buffers the entire output into a callback — convenient but risky for large output (memory) and shell injection if input isn't sanitized.
- `execFile(file, args)`: like `exec` but doesn't spawn a shell, safer against injection, for executing a specific file directly.
- `fork(modulePath)`: special case of `spawn` specifically for launching new Node.js processes, with a built-in IPC (message-passing) channel between parent and child (`child.send()`, `child.on('message')`).

**Worker Threads vs Child Processes**
- Worker threads (`worker_threads` module) run JS in parallel on separate threads *within the same process*, sharing memory via `SharedArrayBuffer` if needed and communicating via message passing — lighter weight than a full process, ideal for CPU-bound JS work (e.g., image processing, heavy computation) that would otherwise block the event loop.
- Child processes are separate OS processes with their own memory space — heavier, but provide full isolation (crash containment) and can run non-Node executables.
- Rule of thumb: use worker threads for CPU-heavy JS logic within the same app; use child processes when you need OS-level isolation or to run external programs.

**Clustering**
The `cluster` module lets you fork multiple Node.js processes (typically one per CPU core) that all share the same server port, with the OS/Node internally load-balancing incoming connections across them (round-robin on most platforms). This lets a Node app use multiple cores, since a single Node process is otherwise limited to one. Each worker has its own memory/event loop — no shared state unless explicitly synced through an external store (Redis, DB).

```js
const cluster = require('cluster');
const os = require('os');

if (cluster.isPrimary) {
  os.cpus().forEach(() => cluster.fork());
  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} died, restarting...`);
    cluster.fork();
  });
} else {
  require('./server'); // each worker runs the actual app
}
```

**IPC**: parent/child (or primary/cluster-worker) processes communicate via message passing (`process.send()`/`.on('message')`), since they don't share memory directly.

---

## 17. Performance Optimization

**Memory leaks**: common causes — global variables accidentally retained, forgotten `setInterval`/event listeners that are never cleared, closures unintentionally holding references to large objects, growing caches with no eviction policy. Diagnosed with heap snapshots (Chrome DevTools' Memory tab attached via `--inspect`) taken at different points in time and diffed to see what's growing.

**Profiling**: `node --prof app.js` generates a V8 log you process with `node --prof-process` to see where CPU time goes. Tools like `clinic.js` (`clinic doctor`, `clinic flame`) give friendlier flame graphs and diagnosis of event-loop delays, I/O bottlenecks, etc.

**GC tuning**: rarely needed for typical apps, but you can inspect/adjust V8 heap flags (`--max-old-space-size`) if you're hitting memory ceilings, and structure code to minimize churn of short-lived large objects.

**Caching**: in-memory caches (a simple `Map`, or `lru-cache`) for per-process caching; Redis for shared caching across multiple app instances — reduces redundant DB/API calls.

**Query optimization**: add indexes on frequently filtered/joined columns; avoid the N+1 query problem (issuing one query per row in a loop instead of a single batched query — solved with eager loading / `JOIN` / DataLoader-style batching).

**Load testing**: tools like `autocannon`, `k6`, or `Artillery` simulate concurrent traffic to measure throughput/latency and find breaking points before users do.

---

## 18. Scalability & System Design

**Horizontal vs vertical scaling**: vertical = bigger machine (more CPU/RAM on one instance) — simple but has a ceiling and a single point of failure. Horizontal = more instances behind a load balancer — better fault tolerance and near-limitless scaling, but requires designing the app to be stateless (no in-memory session state tied to one instance) or externalizing state (Redis, DB).

**Load balancing**: Nginx or a cloud load balancer distributes incoming traffic across instances; PM2's cluster mode does this across CPU cores of a single machine.

**Microservices with Node.js**: splitting an app into independently deployable services (e.g., users service, orders service) communicating via HTTP/gRPC or async messaging. Pros: independent scaling/deployment, technology flexibility, fault isolation. Cons: operational complexity, network latency, distributed data consistency challenges.

**Message queues & pub/sub**: RabbitMQ/Kafka decouple producers and consumers — a producer publishes an event/message without knowing who (or how many) consumers will process it, enabling async workflows, retry semantics, and load leveling (buffering bursts of work).

**API Gateway pattern**: a single entry point that routes requests to the right backend microservice, and can centralize concerns like auth, rate limiting, and logging instead of duplicating them in every service.

**Rate limiting/throttling at scale**: typically done at the gateway/load-balancer layer with a shared store (Redis) so limits apply consistently across all instances, not per-instance.

**High availability / fault tolerance**: redundancy (multiple instances, no single point of failure), health checks with automatic failover, graceful degradation (serve a cached/partial response rather than a hard failure when a dependency is down).

**Circuit breaker pattern**: wraps calls to an external dependency; after a threshold of failures, it "opens" and fails fast (without even attempting the call) for a cooldown period, instead of piling up slow/failing requests and cascading the failure — then periodically tests ("half-open") whether the dependency has recovered.

**Distributed transactions**: since you can't easily use a single DB transaction across multiple services, patterns like Sagas (a sequence of local transactions, each with a compensating action to undo it if a later step fails) are used to maintain consistency across services.

---

## 19. Advanced Asynchronous Patterns

**Async iterators/generators**
```js
async function* generateNumbers() {
  yield 1;
  yield 2;
  yield 3;
}
async function run() {
  for await (const num of generateNumbers()) {
    console.log(num);
  }
}
```
Useful for lazily processing streams of async data (e.g., paginated API results) without loading everything into memory at once.

**Microtasks vs macrotasks (deeper)**: since the microtask queue is fully drained between every macrotask (and even between synchronous operations that queue microtasks), an async function that keeps chaining `.then()` internally can, in pathological cases, starve I/O and timers — worth knowing as a "gotcha" question.

**Race conditions in async code**: e.g., two concurrent requests both read a value, both increment it, both write back — one update is lost. Solutions: database-level atomic operations/locks, optimistic concurrency control (version checks), or serializing access via a queue.

**Retry/backoff logic**: retrying a failed operation immediately can worsen an already-struggling dependency; exponential backoff (wait 1s, 2s, 4s, 8s...) with jitter (randomized variation) spreads out retries and reduces thundering-herd effects.

**Cancellation with AbortController**
```js
const controller = new AbortController();
fetch(url, { signal: controller.signal });
setTimeout(() => controller.abort(), 5000); // cancel after 5s
```
Lets you cancel in-flight async operations (fetch requests, or custom async functions that check `signal.aborted`).

---

## 20. Security (Advanced)

**SQL/NoSQL injection**: never build queries via string concatenation with user input; use parameterized queries/prepared statements (SQL) or proper query builders that escape input, and validate/sanitize input types for NoSQL (e.g., ensure a field expected to be a string isn't actually a MongoDB query operator object like `{"$gt": ""}`).

**XSS (Cross-Site Scripting)**: sanitize/escape any user-generated content rendered in HTML; use templating engines that auto-escape by default, and set a Content-Security-Policy header to restrict script sources.

**CSRF (Cross-Site Request Forgery)**: an attacker tricks a logged-in user's browser into making an unwanted request; mitigated with CSRF tokens (validated per-request) and `SameSite` cookie attributes.

**OAuth 2.0 / OpenID Connect**: OAuth 2.0 is an authorization framework (grants a client access to a resource on a user's behalf without sharing credentials) with flows like Authorization Code (with PKCE for public clients). OpenID Connect is an identity layer on top of OAuth 2.0 that adds authentication (who the user is) via an ID token (a JWT).

**Refresh token rotation**: each time a refresh token is used to get a new access token, the old refresh token is invalidated and a new one issued — limits the damage window if a refresh token is stolen (reuse of an old one signals theft and can trigger revoking the whole token family).

**Secrets management**: for production, prefer a dedicated secrets manager (AWS Secrets Manager, HashiCorp Vault) over `.env` files, enabling rotation, audit logs, and fine-grained access control.

**Dependency scanning**: `npm audit` and tools like Snyk scan `package-lock.json` against known vulnerability databases and flag/patch vulnerable dependency versions.

**Prototype pollution**: occurs when untrusted input can set properties like `__proto__` on an object, polluting `Object.prototype` globally and potentially altering behavior across the app — mitigated by validating/sanitizing input keys and using `Object.create(null)` or `Map` for user-controlled key-value data. **ReDoS** (Regular Expression Denial of Service): certain regex patterns (with nested quantifiers/catastrophic backtracking) can take exponential time on crafted input, freezing the event loop — mitigated by avoiding vulnerable patterns and/or setting execution timeouts.

---

## 21. Advanced Database Concepts

**ACID**: Atomicity (all-or-nothing), Consistency (valid state transitions), Isolation (concurrent transactions don't interfere), Durability (committed data survives crashes) — the guarantees relational transactions provide.

**Connection pool tuning**: pool size should be tuned relative to DB max connections and expected concurrency — too small causes request queuing/timeouts, too large can overwhelm the database server.

**Replicas and sharding (awareness level)**: read replicas offload read traffic from the primary (writes still go to primary); sharding splits data horizontally across multiple databases by some key (e.g., user ID range) to scale beyond a single machine's capacity — both add significant complexity around consistency and routing.

**ORM vs raw queries**: ORMs improve productivity, safety (parameterized queries built-in), and portability, but can generate inefficient queries for complex joins/aggregations — for performance-critical paths, dropping to raw SQL (or the ORM's raw-query escape hatch) is common.

**Migrations in production**: run additive, backward-compatible migrations (e.g., add nullable columns before making code depend on them) to avoid downtime; avoid destructive changes (dropping columns) until old code no longer references them.

---

## 22. Testing (Advanced)

- **Unit tests**: isolate a single function/module (mock all dependencies).
- **Integration tests**: test how multiple units work together (e.g., a route handler + real DB, often via a test database or in-memory DB).
- **E2E tests**: test the whole system from the outside, as a user/client would (e.g., via HTTP calls against a running instance) — slowest but highest confidence.
- **Coverage tools** (`istanbul`/`nyc`, built into Jest): measure what % of lines/branches are exercised by tests — a useful signal, but 100% coverage doesn't guarantee correctness.
- **Mocking external services**: `nock` intercepts outgoing HTTP calls in tests so you don't hit real third-party APIs; `sinon` provides spies/stubs/mocks for functions.
- **Contract testing** (e.g., Pact): verifies that a service's API still matches what its consumers expect, without needing to spin up the full dependency chain — useful in microservices to catch breaking changes early.
- **CI setup**: run lint + tests automatically on every push/PR (GitHub Actions, GitLab CI, Jenkins), blocking merges on failure.

---

## 23. Deployment & DevOps

**Docker**: multi-stage builds keep production images small by separating the build environment (with dev dependencies, compilers) from the final runtime image (only production dependencies + built artifacts) — reduces image size and attack surface.

**PM2**: a production process manager for Node.js — restarts crashed apps automatically, supports cluster mode (multi-core without writing your own `cluster` code), zero-downtime reloads (`pm2 reload`, which restarts workers one at a time so there's no service gap), and log management.

**CI/CD**: automated pipelines that lint, test, build, and deploy on each change — reduces manual error and enables fast, safe, frequent releases.

**Environment config across stages**: keep dev/staging/prod config separate (env vars, not hardcoded), never share production secrets with lower environments.

**Logging**: structured logging (`Winston`, `Pino`) with log levels (debug/info/warn/error) instead of raw `console.log`, shipped to a centralized system (ELK stack, Datadog, CloudWatch) so logs from many instances are searchable in one place.

**Health checks**: a `/healthz` or `/readyz` endpoint that a load balancer or orchestrator (Kubernetes) polls — "liveness" (is the process alive/should it be restarted) vs "readiness" (is it ready to accept traffic, e.g., DB connection established).

**Graceful shutdown**
```js
process.on('SIGTERM', async () => {
  server.close(() => console.log('HTTP server closed'));
  await db.close();
  process.exit(0);
});
```
Ensures in-flight requests finish and connections close cleanly before the process exits — important during deploys/scaling events so users don't see dropped connections.

---

## 24. GraphQL & Modern API Design

**REST vs GraphQL**: REST exposes fixed resource endpoints (can lead to over-fetching or under-fetching data, requiring multiple round trips); GraphQL exposes a single endpoint where clients specify exactly the fields/shape they need in one request — powerful but shifts complexity to query validation, caching, and performance (a single query can trigger many resolver calls).

**Apollo Server / GraphQL Yoga**: server libraries for building a GraphQL API — define a schema (types), and resolver functions that fetch the actual data for each field.

**Resolvers, schemas, mutations**: schema defines the shape of the graph (Query/Mutation types and object types); resolvers are functions mapping each field to its data source; mutations are the GraphQL equivalent of write operations (create/update/delete).

**N+1 problem & DataLoader**: naively resolving a list of items and then a related field per item (e.g., fetching each post's author individually) causes N+1 queries; DataLoader batches and caches those lookups within a single request tick, turning N queries into one batched query.

---

## 25. TypeScript with Node.js

**Setup**: install `typescript`, `ts-node` (or use `tsc` to compile), `@types/node`, and framework-specific types (`@types/express`); configure `tsconfig.json` (target, module, strict mode, outDir).

**Types vs interfaces**: both can describe object shapes; `interface` supports declaration merging (useful for extending third-party types) and is generally preferred for public object/API shapes, while `type` is more flexible for unions, intersections, and mapped/conditional types. Generics let you write reusable, type-safe functions/classes (e.g., a generic `Repository<T>` for different entity types).

**Build config**: `tsconfig.json` controls compilation target (`ES2020`, etc.), module system (`CommonJS` vs `ESNext`), strictness flags (`strict: true` catches more bugs at compile time), and output directory — typically you compile TS to JS for production rather than running `ts-node` directly in prod (extra runtime overhead).

---

## 26. Miscellaneous Advanced Topics

**Memory management deep dive**: the heap holds objects/closures; the stack holds function call frames and primitives. Closures retaining references to large outer-scope variables (e.g., a callback registered once that captures a huge array) is a classic, subtle memory leak source since the referenced memory can't be garbage collected while the closure exists.

**`this` pitfalls in callbacks**: a regular function passed as a callback loses its original `this` binding (e.g., a class method passed as an event handler) — fixed with arrow functions (which lexically inherit `this`) or explicit `.bind(this)`.

**Native addons (N-API)**: lets you write performance-critical parts of a Node.js module in C/C++ (e.g., for cryptography, image processing) exposed to JS — an advanced, rarely-needed-day-to-day capability, but good to know it exists and why (raw performance beyond what JS/V8 gives you).

**WebSockets / real-time**: `Socket.IO` (with fallbacks and rooms/namespaces) or the raw `ws` library provide persistent, full-duplex connections for real-time features (chat, live dashboards, notifications) — unlike HTTP's request/response model, either side can push data at any time.

**SSE vs WebSockets**: Server-Sent Events are a simpler, one-directional (server → client) stream over plain HTTP, good for things like live feeds/notifications where the client doesn't need to send data back over the same channel; WebSockets are bidirectional and needed for things like chat or collaborative editing.

**Rate limiting algorithms**: 
- Token bucket: a bucket refills tokens at a fixed rate; each request consumes a token; allows some burstiness up to the bucket size.
- Sliding window: counts requests in a rolling time window, smoothing out the "burst right at the reset boundary" issue that a simple fixed window has.

**Idempotency**: an idempotent operation produces the same result no matter how many times it's applied (e.g., `PUT`, `DELETE` should be idempotent by REST convention) — important for safe retries (e.g., a client retries a payment request after a timeout; an idempotency key ensures it isn't charged twice).

**API versioning**: via URL path (`/api/v1/...`), custom headers, or content negotiation — lets you evolve an API without breaking existing clients.

---

# COMMON CODING/PRACTICAL QUESTIONS (WITH ANSWERS)

### 1. Implement debounce and throttle

```js
// Debounce: wait until calls stop for `delay` ms, then run once
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}

// Throttle: run at most once every `limit` ms, regardless of call frequency
function throttle(fn, limit) {
  let inThrottle = false;
  return (...args) => {
    if (!inThrottle) {
      fn(...args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}
```
Debounce = "wait for a pause" (e.g., search-as-you-type). Throttle = "at most once per interval" (e.g., scroll handlers).

### 2. Build a simple rate limiter (token bucket, in-memory)

```js
class RateLimiter {
  constructor(maxTokens, refillRate) {
    this.maxTokens = maxTokens;
    this.tokens = maxTokens;
    this.refillRate = refillRate; // tokens per second
    this.lastRefill = Date.now();
  }
  allow() {
    const now = Date.now();
    const elapsed = (now - this.lastRefill) / 1000;
    this.tokens = Math.min(this.maxTokens, this.tokens + elapsed * this.refillRate);
    this.lastRefill = now;
    if (this.tokens >= 1) {
      this.tokens -= 1;
      return true;
    }
    return false;
  }
}
```

### 3. Implement an LRU Cache

```js
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.map = new Map(); // Maps preserve insertion order
  }
  get(key) {
    if (!this.map.has(key)) return -1;
    const value = this.map.get(key);
    this.map.delete(key);
    this.map.set(key, value); // move to "most recently used" position
    return value;
  }
  put(key, value) {
    if (this.map.has(key)) this.map.delete(key);
    else if (this.map.size >= this.capacity) {
      this.map.delete(this.map.keys().next().value); // evict least recently used
    }
    this.map.set(key, value);
  }
}
```

### 4. Flatten nested async operations (sequential vs parallel)

```js
// Sequential (one after another)
async function processSequential(items, asyncFn) {
  const results = [];
  for (const item of items) {
    results.push(await asyncFn(item));
  }
  return results;
}

// Parallel (all at once)
async function processParallel(items, asyncFn) {
  return Promise.all(items.map(asyncFn));
}
```
Interviewers often probe: "why would you choose sequential over parallel?" — e.g., rate-limited APIs, or when later items depend on side effects of earlier ones.

### 5. Basic pub/sub using EventEmitter

```js
const EventEmitter = require('events');
const bus = new EventEmitter();

bus.on('order:created', (order) => {
  console.log('Send confirmation email for', order.id);
});
bus.on('order:created', (order) => {
  console.log('Update inventory for', order.id);
});

bus.emit('order:created', { id: 123 });
```

### 6. Custom Express middleware (request timing example)

```js
function requestTimer(req, res, next) {
  const start = Date.now();
  res.on('finish', () => {
    console.log(`${req.method} ${req.url} - ${Date.now() - start}ms`);
  });
  next();
}
app.use(requestTimer);
```

### 7. Fixing a memory leak (typical interview snippet)

Given something like:
```js
// Leaky: interval never cleared, closure holds a growing array forever
function startLeak() {
  const bigData = [];
  setInterval(() => {
    bigData.push(new Array(1000000).fill('*'));
  }, 100);
}
```
Fix: keep a reference to the interval and clear it when no longer needed, and/or bound the size of `bigData` (evict old entries):
```js
function startFixed() {
  const bigData = [];
  const id = setInterval(() => {
    if (bigData.length > 100) bigData.shift(); // bound growth
    bigData.push(new Array(1000).fill('*'));
  }, 100);
  return () => clearInterval(id); // caller can stop it
}
```

### 8. Classic event-loop ordering question

```js
console.log('1: start');

setTimeout(() => console.log('2: setTimeout'), 0);

Promise.resolve().then(() => console.log('3: promise'));

process.nextTick(() => console.log('4: nextTick'));

console.log('5: end');
```
**Output:** `1: start` → `5: end` → `4: nextTick` → `3: promise` → `2: setTimeout`

Explanation: synchronous code (`1`, `5`) runs first and completely empties the call stack. Then, before the event loop can move to the timers phase, Node drains the `process.nextTick` queue fully (`4`), then the Promise microtask queue (`3`). Only after both microtask queues are empty does the loop proceed to the timers phase, where the `setTimeout` callback (`2`) finally runs.


---

## How to Use This Guide
- **Beginner interviews**: sections 1–6, plus a basic explanation of the event loop.
- **Mid-level interviews**: sections 7–15 in depth — Express, async patterns, error handling, and be ready for the classic event-loop ordering question.
- **Senior/advanced interviews**: sections 16–26 plus the coding questions above — internals, scalability, security, and system design, often combined with a live coding or system design round.

