Streams in Node.js (Complete Explanation)
What is a Stream?

A Stream is a way to process data piece by piece (in chunks) instead of loading the entire data into memory at once.

Definition

A Stream is an object in Node.js that allows you to read or write data continuously in small chunks, making data processing faster and more memory-efficient.

Why Do We Need Streams?

Suppose you have a 5 GB video file.

Without Streams
const fs = require("fs");

const data = fs.readFileSync("movie.mp4");

console.log(data);
What happens?
Disk
 │
 ▼
5 GB File
 │
 ▼
RAM (5 GB)
 │
 ▼
Application

Problems:

❌ Entire file is loaded into RAM.
❌ High memory usage.
❌ Slow startup because the app waits until the whole file is read.
❌ Can crash if the file is larger than available memory.
With Streams
const fs = require("fs");

const stream = fs.createReadStream("movie.mp4");

Now the file is read in small chunks (for example, around 64 KB by default for many file streams).

Disk
 │
 ▼
64 KB
 │
 ▼
Application

Next 64 KB
 │
 ▼
Application

Next 64 KB
 │
 ▼
Application

Only a small portion of the file is in memory at any given time.

Real-Life Analogy

Imagine you need to drink a 20-liter water bottle.

Without Streams

Drink all 20 liters at once.

Impossible.

With Streams

Drink one glass at a time.

Easy and efficient.

Streams work the same way with data.

How Streams Work

Suppose a file is 1 MB.

1 MB File

Chunk 1 → 64 KB

Chunk 2 → 64 KB

Chunk 3 → 64 KB

...

Chunk 16 → 64 KB

Each chunk is processed immediately.

No need to wait for the entire file.

Types of Streams

Node.js has four main types of streams.

1. Readable Stream

Used to read data.

Examples:

Reading a file
Reading an HTTP request body
Reading data from a socket

Example:

const fs = require("fs");

const readStream = fs.createReadStream("data.txt");

readStream.on("data", (chunk) => {
    console.log(chunk.toString());
});

Flow:

File
 │
 ▼
Readable Stream
 │
 ▼
Application
2. Writable Stream

Used to write data.

Example:

const fs = require("fs");

const writeStream = fs.createWriteStream("output.txt");

writeStream.write("Hello");
writeStream.write("World");
writeStream.end();

Flow:

Application
 │
 ▼
Writable Stream
 │
 ▼
File
3. Duplex Stream

Can read and write.

Examples:

TCP sockets
Some network connections
Read
 ▲
 │
Duplex
 │
 ▼
Write

Example:

const net = require("net");

const server = net.createServer((socket) => {
    socket.write("Welcome");

    socket.on("data", (data) => {
        console.log(data.toString());
    });
});

The socket reads and writes.

4. Transform Stream

A special type of duplex stream that modifies data while it passes through.

Examples:

Compression
Encryption
Converting text to uppercase
Input
 │
 ▼
Transform
 │
 ▼
Output

Example:

const { Transform } = require("stream");

const upperCase = new Transform({
    transform(chunk, encoding, callback) {
        callback(null, chunk.toString().toUpperCase());
    }
});

upperCase.on("data", (chunk) => {
    console.log(chunk.toString());
});

upperCase.write("hello");
upperCase.end();

Output:

HELLO
Stream Events
data

Fired when a chunk is available.

stream.on("data", (chunk) => {
    console.log(chunk);
});
end

Fired when reading finishes.

stream.on("end", () => {
    console.log("Finished");
});
error

Handles errors.

stream.on("error", (err) => {
    console.log(err);
});
close

Called when the stream is closed.

stream.on("close", () => {
    console.log("Closed");
});
Piping Streams

One of the biggest advantages of streams is piping.

Instead of manually reading chunks and writing them, connect one stream directly to another.

const fs = require("fs");

const readStream = fs.createReadStream("input.txt");
const writeStream = fs.createWriteStream("output.txt");

readStream.pipe(writeStream);

Flow:

Input File
     │
     ▼
Readable Stream
     │
     ▼
pipe()
     │
     ▼
Writable Stream
     │
     ▼
Output File
Streaming in Express

Instead of loading an entire video into memory:

app.get("/video", (req, res) => {

    const stream = fs.createReadStream("movie.mp4");

    stream.pipe(res);

});

Flow:

Disk
 │
 ▼
Readable Stream
 │
 ▼
HTTP Response
 │
 ▼
Browser

The browser starts playing the video while the rest of the file is still being read.

Backpressure

Suppose:

Disk reads 100 MB/sec
Network sends 10 MB/sec

Without control:

Disk

100 MB/sec

↓

RAM

↓

Network

10 MB/sec

Memory usage keeps increasing.

Backpressure is the mechanism that tells the readable stream to slow down or pause when the writable stream cannot keep up.

This prevents excessive memory usage.

When you use pipe(), Node.js handles backpressure automatically.

Streams vs Buffers
Buffer	Stream
Loads entire data into memory	Processes data chunk by chunk
Higher memory usage	Low memory usage
Waits until everything is available	Starts processing immediately
Best for small files	Best for large files
Advantages of Streams
✅ Low memory usage.
✅ Faster processing because work starts immediately.
✅ Ideal for large files.
✅ Built-in backpressure support.
✅ Efficient for file handling and networking.
Real-World Uses of Streams
Uploading large files.
Downloading files.
Video and audio streaming.
Reading log files.
Copying files.
Compressing files.
Creating ZIP archives.
Processing CSV files.
Sending HTTP responses.
Interview Definition

A stream in Node.js is an object that enables reading or writing data incrementally in small chunks instead of loading the entire dataset into memory. Streams improve performance and reduce memory usage, making them ideal for handling large files, network communication, and real-time data processing.

Common Interview Question

Q: Why are streams preferred over fs.readFile() for large files?

Answer:

fs.readFile() reads the entire file into memory before your code can use it. For large files, this can consume a lot of RAM and delay processing. Streams, on the other hand, read the file in small chunks, allowing processing to begin immediately while keeping memory usage low. This makes streams much more efficient for large files and continuous data transfer.





