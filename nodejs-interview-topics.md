# Node.js Interview Topics: Beginner to Advanced

## BEGINNER LEVEL

### 1. Node.js Fundamentals
- What is Node.js and why use it (V8 engine, single-threaded, event-driven)
- Node.js vs browser JavaScript — differences in global objects, modules
- npm and package.json — dependencies, devDependencies, scripts, versioning (semver)
- Installing packages: `npm install`, `-g`, `--save-dev`
- Node.js REPL
- `node_modules` folder structure
- Global objects: `global`, `process`, `__dirname`, `__filename`, `module`, `exports`, `require`

### 2. Modules
- CommonJS module system (`require`, `module.exports`)
- Creating and exporting custom modules
- Built-in/core modules: `fs`, `path`, `os`, `http`, `events`, `url`, `util`
- ES Modules (`import`/`export`) in Node.js — `type: "module"` in package.json
- Differences between CommonJS and ES Modules
- Module caching in Node.js

### 3. File System (fs module)
- Reading/writing files: sync vs async methods (`readFileSync` vs `readFile`)
- Streams basics: `createReadStream`, `createWriteStream`
- Working with directories: `mkdir`, `readdir`, `unlink`
- File paths: `path.join`, `path.resolve`, `path.basename`

### 4. Basic HTTP & Servers
- Creating a basic HTTP server using the `http` module
- Handling requests and responses
- Routing basics (manual routing without a framework)
- HTTP methods: GET, POST, PUT, DELETE, PATCH
- Status codes and headers

### 5. Asynchronous Programming Basics
- Synchronous vs asynchronous code
- Callbacks and callback hell
- Introduction to Promises
- `async`/`await` basics
- Error-first callback pattern

### 6. NPM Ecosystem Basics
- `package.json` vs `package-lock.json`
- Semantic versioning (`^`, `~`, exact versions)
- Publishing a basic package (conceptual understanding)

---

## INTERMEDIATE LEVEL

### 7. Event Loop & Concurrency Model (Core Topic — Always Asked)
- What is the Event Loop and how it works
- Phases of the Event Loop: timers, pending callbacks, idle/prepare, poll, check, close callbacks
- Call stack, callback queue, microtask queue vs macrotask queue
- `process.nextTick()` vs `setImmediate()` vs `setTimeout(fn, 0)`
- How Node.js handles non-blocking I/O with a single thread
- Libuv — role in the event loop and thread pool
- Blocking vs non-blocking operations, and how blocking code affects the event loop

### 8. Streams & Buffers (Deep Dive)
- Types of streams: Readable, Writable, Duplex, Transform
- Piping streams (`pipe()`), chaining streams
- Backpressure handling
- Buffers — what they are, why Node.js uses them, `Buffer.alloc`, `Buffer.from`
- Working with large files efficiently using streams

### 9. Express.js / Web Frameworks
- Setting up an Express app, routing
- Middleware — what it is, how it works, `next()`
- Built-in vs third-party vs custom middleware
- Error-handling middleware
- Request/response lifecycle in Express
- Route parameters, query parameters, body parsing
- Serving static files
- RESTful API design principles

### 10. Asynchronous Programming (Deep Dive)
- Promise chaining, `Promise.all`, `Promise.race`, `Promise.allSettled`, `Promise.any`
- Error handling with async/await (try/catch)
- Converting callback-based functions to promises (`util.promisify`)
- Event emitters (`EventEmitter` class) — custom events, `.on()`, `.emit()`, `.once()`

### 11. Database Integration
- Connecting to SQL databases (e.g., PostgreSQL, MySQL) using drivers/ORMs (Sequelize, Prisma, TypeORM)
- Connecting to NoSQL databases (MongoDB with Mongoose)
- Connection pooling
- CRUD operations
- Transactions basics

### 12. Error Handling
- Try/catch vs error-first callbacks vs `.catch()`
- Custom error classes
- Global error handling (`process.on('uncaughtException')`, `process.on('unhandledRejection')`)
- Handling errors in Express (centralized error middleware)
- Operational vs programmer errors

### 13. Authentication & Security Basics
- Session-based vs token-based authentication
- JWT (JSON Web Tokens) — structure, signing, verification
- Password hashing (bcrypt)
- CORS — what it is and how to configure it
- Environment variables and `.env` files (dotenv)
- Basic security practices: helmet, rate limiting, input validation/sanitization

### 14. Testing Basics
- Unit testing with Jest/Mocha/Chai
- Writing test cases for functions and routes
- Mocking dependencies
- Basic concept of TDD

### 15. Debugging & Tooling
- Using `console` methods effectively
- Debugging with `node --inspect` and Chrome DevTools
- Using nodemon for development
- Environment-specific configs (development/production)

---

## ADVANCED LEVEL

### 16. Node.js Internals & Architecture
- V8 engine internals — JIT compilation, hidden classes, garbage collection basics
- Libuv thread pool — configuring pool size (`UV_THREADPOOL_SIZE`)
- How Node.js handles CPU-bound vs I/O-bound tasks
- Single-threaded nature and how to scale beyond it
- Child processes: `spawn`, `exec`, `execFile`, `fork` — differences and use cases
- Worker Threads — when and why to use them vs child processes
- Clustering — `cluster` module, load balancing across CPU cores
- Inter-process communication (IPC)

### 17. Performance Optimization
- Identifying and fixing memory leaks
- Profiling Node.js applications (`--prof`, clinic.js, heap snapshots)
- Garbage collection tuning
- Caching strategies (in-memory, Redis)
- Optimizing database queries (indexing, N+1 problem)
- Load testing tools (Artillery, k6, autocannon)
- Reducing startup time, lazy loading modules

### 18. Scalability & System Design
- Horizontal vs vertical scaling
- Load balancing strategies (Nginx, PM2 cluster mode)
- Microservices architecture with Node.js
- Message queues (RabbitMQ, Kafka) and pub/sub patterns
- API Gateway pattern
- Rate limiting and throttling at scale
- Designing for high availability and fault tolerance
- Circuit breaker pattern
- Handling distributed transactions

### 19. Advanced Asynchronous Patterns
- Async iterators and generators (`async function*`, `for await...of`)
- Understanding microtasks vs macrotasks in depth
- Handling race conditions in async code
- Implementing custom retry/backoff logic
- Cancellation patterns (AbortController)

### 20. Security (Advanced)
- Preventing common vulnerabilities: SQL injection, NoSQL injection, XSS, CSRF
- Secure headers and Content Security Policy
- OAuth 2.0 and OpenID Connect flows
- Refresh token rotation and secure token storage
- Handling secrets management (Vault, AWS Secrets Manager)
- Dependency vulnerability scanning (`npm audit`, Snyk)
- Preventing prototype pollution and ReDoS attacks

### 21. Advanced Database Concepts
- Database transactions and ACID properties in Node.js apps
- Connection pooling tuning
- Read/write replicas and sharding awareness
- ORMs vs raw query performance trade-offs
- Handling migrations in production

### 22. Testing (Advanced)
- Integration testing vs unit testing vs e2e testing
- Test coverage tools (istanbul/nyc)
- Mocking external services/APIs (nock, sinon)
- Contract testing for microservices
- Continuous Integration setup for Node.js projects

### 23. Deployment & DevOps
- Containerizing Node.js apps with Docker (multi-stage builds)
- Process managers (PM2) — zero-downtime restarts, clustering
- CI/CD pipelines for Node.js
- Environment configuration management across stages
- Logging strategies (Winston, Pino, centralized logging with ELK/Datadog)
- Health checks and readiness/liveness probes (Kubernetes context)
- Graceful shutdown handling (`SIGTERM`, `SIGINT`, closing DB connections)

### 24. GraphQL & Modern API Design (if relevant to role)
- REST vs GraphQL trade-offs
- Setting up Apollo Server / GraphQL Yoga with Node.js
- Resolvers, schemas, mutations
- N+1 query problem and DataLoader

### 25. TypeScript with Node.js
- Setting up a Node.js project with TypeScript
- Type definitions for Express and other libraries
- Interfaces vs types, generics in a Node.js context
- Compiling and build configuration (`tsconfig.json`)

### 26. Miscellaneous Advanced Topics
- Memory management deep dive (heap, stack, closures causing leaks)
- Understanding `this` binding pitfalls in callbacks
- Writing native addons (N-API) — conceptual awareness
- WebSockets and real-time communication (Socket.IO, `ws`)
- Server-Sent Events vs WebSockets
- Rate limiting algorithms (token bucket, sliding window)
- Idempotency in API design
- Designing versioned APIs

---

## Common Coding/Practical Questions Often Asked Alongside These Topics
- Implement a debounce/throttle function
- Build a simple rate limiter
- Implement an LRU cache
- Write a function to flatten nested async operations
- Build a basic pub/sub system using EventEmitter
- Implement custom middleware for Express
- Fix a memory leak in a given code snippet
- Explain the output of a tricky event-loop code snippet (classic `setTimeout` vs `setImmediate` vs `process.nextTick` ordering question)

---

## How to Use This List
- **Beginner interviews** typically focus on sections 1–6 plus basic event loop understanding.
- **Mid-level interviews** dig into sections 7–15, especially Express, async patterns, and error handling.
- **Senior/advanced interviews** emphasize sections 16–26 — internals, scalability, security, and system design, often combined with a live coding or system design round.

If you want, I can turn any specific section into a deeper study guide with example questions and model answers, or create a mock interview based on your target experience level.
