# 1. EVENT LOOP - INTERVIEW GUIDE

## 🔄 TỔNG QUAN KIẾN TRÚC EVENT LOOP

### Kiến trúc Node.js và Event Loop
```
┌─────────────────────────────────────────────────────────────────┐
│                        V8 JavaScript Engine                     │
│ ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐   │
│ │   Call Stack    │  │   Memory Heap   │  │   Micro-tasks   │   │
│ │     (LIFO)      │  │                 │  │   - nextTick    │   │
│ └─────────────────┘  └─────────────────┘  │   - Promises    │   │
│                                            └─────────────────┘   │
└─────────────────┬───────────────────────────────────────────────┘
                  │
                  ▼ (Binding Layer)
┌─────────────────────────────────────────────────────────────────┐
│                     Node.js Core (C++)                          │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    libuv Event Loop                         ││
│  │                                                             ││
│  │  ┌─────┐ ┌─────────┐ ┌──────┐ ┌─────┐ ┌─────┐ ┌─────────┐  ││
│  │  │Timer│→│Pending  │→│I/P   │→│Poll │→│Check│→│Close    │  ││
│  │  │     │ │Callback │ │Prep  │ │     │ │     │ │Callback │  ││
│  │  └─────┘ └─────────┘ └──────┘ └─────┘ └─────┘ └─────────┘  ││
│  │                                                             ││
│  │  ┌─────────────────────────────────────────────────────────┐││
│  │  │           Thread Pool (4-128 threads)                   │││
│  │  │  File I/O │ DNS │ CPU-intensive │ Crypto │ Compression  │││
│  │  └─────────────────────────────────────────────────────────┘││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────┬───────────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Operating System                             │
│           (epoll, kqueue, IOCP, select)                        │
└─────────────────────────────────────────────────────────────────┘
```

### Event Loop vs Call Stack
```javascript
console.log('1: Start');

setTimeout(() => {
  console.log('2: Timer callback');
}, 0);

setImmediate(() => {
  console.log('3: Immediate callback');  
});

process.nextTick(() => {
  console.log('4: NextTick callback');
});

Promise.resolve().then(() => {
  console.log('5: Promise callback');
});

console.log('6: End');

// Output: 1 → 6 → 4 → 5 → 3 → 2 (hoặc 2 → 3)
```

---

## 📊 CHI TIẾT 6 PHASES CỦA EVENT LOOP

### 1. **Timer Phase** - Thực thi setTimeout/setInterval
```javascript
// Timer accuracy - minimum 1ms
setTimeout(() => console.log('timer 1'), 0);   // Actually ~1ms
setTimeout(() => console.log('timer 2'), 1);   // ~1ms  
setTimeout(() => console.log('timer 3'), 100); // ~100ms

// Timer có thể bị delay bởi poll phase
const fs = require('fs');
setTimeout(() => console.log('timer'), 1);
fs.readFile('large-file.txt', () => {
  console.log('file read complete');
}); // Timer có thể chạy sau file I/O
```

### 2. **Pending Callbacks Phase** - I/O callbacks từ vòng trước
```javascript
// Callbacks bị defer từ poll phase trước
const net = require('net');
const server = net.createServer();

server.on('connection', (socket) => {
  socket.on('error', (err) => {
    // Error callback có thể được xử lý ở pending phase
    console.log('Socket error:', err.message);
  });
});
```

### 3. **Idle, Prepare Phase** - Internal Node.js sử dụng
```javascript
// Không can thiệp trực tiếp, nhưng có thể quan sát
process.nextTick(() => {
  console.log('nextTick trong idle phase');
});
```

### 4. **Poll Phase** - Fetch I/O events và execute callbacks
```javascript
const fs = require('fs');

// Poll phase sẽ chờ I/O events hoàn thành
fs.readFile('data.txt', (err, data) => {
  console.log('File read in poll phase');
  
  // Trong I/O callback, setImmediate > setTimeout
  setTimeout(() => console.log('timer in I/O'), 0);
  setImmediate(() => console.log('immediate in I/O'));
  // Output: immediate in I/O → timer in I/O
});

// Poll phase sẽ block nếu:
// 1. Poll queue rỗng
// 2. Không có pending timers/immediates
// 3. Không có close events
```

### 5. **Check Phase** - Execute setImmediate callbacks
```javascript
// setImmediate luôn chạy sau poll phase trong cùng tick
setImmediate(() => console.log('immediate 1'));
setImmediate(() => console.log('immediate 2'));

// Nested setImmediate
setImmediate(() => {
  console.log('immediate parent');
  setImmediate(() => console.log('immediate child'));
}); // Child sẽ chạy ở tick tiếp theo
```

### 6. **Close Callbacks Phase** - socket.on('close')
```javascript
const net = require('net');
const server = net.createServer();

server.listen(3000, () => {
  server.close(() => {
    console.log('Server closed in close phase');
  });
});

const socket = new net.Socket();
socket.on('close', () => {
  console.log('Socket closed in close phase');
});
```

---

## 🚀 MICRO-TASKS & NEXTTICK QUEUE

### Priority Order trong mỗi Phase
```javascript
// Thứ tự ưu tiên sau mỗi callback:
// 1. process.nextTick queue (highest priority)
// 2. Promise microtasks
// 3. Tiếp tục phase hiện tại

console.log('=== Start ===');

// Macro tasks
setTimeout(() => console.log('Timer 1'), 0);
setImmediate(() => console.log('Immediate 1'));

// Micro tasks  
Promise.resolve().then(() => console.log('Promise 1'));
Promise.resolve().then(() => {
  console.log('Promise 2');
  process.nextTick(() => console.log('NextTick in Promise'));
});

// NextTick (highest priority)
process.nextTick(() => {
  console.log('NextTick 1');
  process.nextTick(() => console.log('NextTick 2'));
  Promise.resolve().then(() => console.log('Promise in NextTick'));
});

console.log('=== End ===');

/* Output:
=== Start ===
=== End ===
NextTick 1
NextTick 2
Promise in NextTick
Promise 1  
Promise 2
NextTick in Promise
Immediate 1
Timer 1
*/
```

### NextTick Starvation Problem
```javascript
// ❌ Có thể gây starvation - block event loop
function recursiveNextTick() {
  process.nextTick(recursiveNextTick);
}
recursiveNextTick(); // Event loop bị block

// ✅ Giải pháp - giới hạn hoặc dùng setImmediate
let count = 0;
function limitedNextTick() {
  if (count++ < 100) {
    process.nextTick(limitedNextTick);
  }
}

// Hoặc dùng setImmediate để cho phép event loop continue
function safeRecursive() {
  setImmediate(safeRecursive); // Cho phép các phase khác chạy
}
```

---

## ⏰ SETTIMEOUT VS SETIMMEDIATE

### Execution Order Context
```javascript
// 1. Trong main thread - thứ tự không đảm bảo
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
// Có thể là: timeout → immediate HOẶC immediate → timeout

// 2. Trong I/O callback - setImmediate luôn trước
const fs = require('fs');
fs.readFile(__filename, () => {
  setTimeout(() => console.log('timeout'), 0);
  setImmediate(() => console.log('immediate'));  
}); // Luôn: immediate → timeout

// 3. Trong nextTick/Promise - thứ tự predictable
process.nextTick(() => {
  setTimeout(() => console.log('timeout'), 0);
  setImmediate(() => console.log('immediate'));
}); // Luôn: immediate → timeout
```

### Timer Resolution & Accuracy
```javascript
const start = process.hrtime.bigint();

setTimeout(() => {
  const end = process.hrtime.bigint();
  console.log(`Actual delay: ${Number(end - start) / 1000000}ms`);
}, 1); // Minimum ~1ms, có thể lớn hơn

// High resolution timing
const { performance } = require('perf_hooks');
const precise_start = performance.now();
setImmediate(() => {
  console.log(`Precise delay: ${performance.now() - precise_start}ms`);
});
```

---

## 🧵 THREAD POOL & ASYNC I/O

### Thread Pool Operations
```javascript
// Các operations sử dụng thread pool:
const fs = require('fs');
const crypto = require('crypto');
const dns = require('dns');
const zlib = require('zlib');

// 1. File System I/O
fs.readFile('file.txt', callback);    // Thread pool
fs.readdir('.', callback);            // Thread pool  
fs.stat('file.txt', callback);        // Thread pool

// 2. CPU-intensive crypto
crypto.pbkdf2('secret', 'salt', 100000, 64, 'sha512', callback);

// 3. DNS resolution (some methods)
dns.lookup('google.com', callback);   // Thread pool
dns.resolve('google.com', callback);  // Network I/O (no thread pool)

// 4. Compression
zlib.gzip(buffer, callback);          // Thread pool
```

### Thread Pool Configuration
```javascript
// Default: 4 threads, có thể config via env
process.env.UV_THREADPOOL_SIZE = '8'; // Tối đa 128

// Monitoring thread pool usage
const fs = require('fs');
console.time('4 concurrent reads');

// 4 operations concurrent - fit in default pool
for (let i = 0; i < 4; i++) {
  fs.readFile(__filename, () => {
    console.log(`Read ${i} completed`);
  });
}

console.time('8 concurrent reads');
// 8 operations - sẽ queue waiting
for (let i = 0; i < 8; i++) {
  fs.readFile(__filename, () => {
    console.log(`Read ${i} completed`);
  });
}
```

### True Async I/O vs Thread Pool
```javascript
const net = require('net');
const fs = require('fs');

// Network I/O - truly async (epoll/kqueue/IOCP)
const server = net.createServer((socket) => {
  socket.on('data', (data) => {
    console.log('Network data received'); // No thread pool
  });
});

// File I/O - thread pool based  
fs.readFile('large-file.txt', (err, data) => {
  console.log('File read completed');   // Uses thread pool
});

// Comparison: 1000 network connections vs 1000 file reads
// Network: ~0 extra threads, handled by OS
// Files: Limited by thread pool size (4-128)
```

---

## 🔍 PERFORMANCE & DEBUGGING

### Measuring Event Loop Lag
```javascript
const { performance } = require('perf_hooks');

// Simple lag measurement
function measureLag() {
  const start = performance.now();
  setImmediate(() => {
    const lag = performance.now() - start;
    console.log(`Event loop lag: ${lag.toFixed(2)}ms`);
  });
}

setInterval(measureLag, 1000);

// More precise measurement
let previous = performance.now();
function preciselag() {
  const now = performance.now();
  const lag = now - previous - 10; // Expected 10ms interval
  previous = now;
  console.log(`Precise lag: ${lag.toFixed(2)}ms`);
}
setInterval(preciselag, 10);
```

### Blocking Operations Detection
```javascript
// ❌ CPU-intensive task blocking event loop
function blockingOperation() {
  const start = Date.now();
  while (Date.now() - start < 5000) {
    // Block for 5 seconds
  }
  console.log('Blocking operation done');
}

// Monitor effect on other tasks
setInterval(() => console.log('Timer tick'), 100);
setTimeout(blockingOperation, 1000); // Will delay timer ticks

// ✅ Non-blocking alternative
function nonBlockingOperation(duration, callback) {
  const start = Date.now();
  const chunk = 100; // Process in 100ms chunks
  
  function processChunk() {
    const chunkStart = Date.now();
    while (Date.now() - chunkStart < chunk) {
      // Process work for 100ms
    }
    
    if (Date.now() - start < duration) {
      setImmediate(processChunk); // Yield control
    } else {
      callback();
    }
  }
  
  processChunk();
}
```

### Event Loop Debugging Tools
```javascript
// 1. Built-in diagnostics
console.log('Active handles:', process._getActiveHandles().length);
console.log('Active requests:', process._getActiveRequests().length);

// 2. Async hooks for tracing
const async_hooks = require('async_hooks');
const hook = async_hooks.createHook({
  init(asyncId, type, triggerAsyncId) {
    console.log(`Async ${type} created: ${asyncId}`);
  },
  before(asyncId) {
    console.log(`Before ${asyncId}`);
  },
  after(asyncId) {
    console.log(`After ${asyncId}`);
  }
});
hook.enable();

// 3. Command line debugging
// node --trace-events-enabled --trace-event-categories v8,node app.js
// node --prof app.js  // V8 profiling
// node --inspect app.js // Chrome DevTools debugging
```

---

## 🔄 NODE.JS VS BROWSER EVENT LOOP

### Key Differences
```javascript
// Browser Event Loop
// 1. Macrotasks: setTimeout, setInterval, I/O, UI events
// 2. Microtasks: Promises, queueMicrotask, MutationObserver
// 3. Animation frames: requestAnimationFrame
// 4. Rendering steps

// Node.js Event Loop  
// 1. 6 distinct phases với các queue riêng
// 2. process.nextTick queue (highest priority)
// 3. Promise microtask queue
// 4. Thread pool cho I/O
// 5. No rendering, no animation frames

// Example showing difference:
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));

// Browser: luôn timeout → immediate
// Node.js: không đảm bảo thứ tự nếu trong main thread
```

### Worker Threads vs Cluster vs Child Process
```javascript
// 1. Worker Threads (Node 10.5+) - shared memory
const { Worker, isMainThread, parentPort } = require('worker_threads');

if (isMainThread) {
  const worker = new Worker(__filename);
  worker.postMessage({ cmd: 'calculate', data: [1, 2, 3] });
  worker.on('message', (result) => console.log('Result:', result));
} else {
  parentPort.on('message', ({ cmd, data }) => {
    if (cmd === 'calculate') {
      const result = data.reduce((a, b) => a + b, 0);
      parentPort.postMessage(result);
    }
  });
}

// 2. Cluster - separate processes
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} died`);
  });
} else {
  require('http').createServer((req, res) => {
    res.writeHead(200);
    res.end('Hello from worker ' + process.pid);
  }).listen(8000);
}

// 3. Child Process - spawn external programs
const { spawn } = require('child_process');
const ls = spawn('ls', ['-lh', '/usr']);
ls.stdout.on('data', (data) => console.log(data.toString()));
```

---

## 🎯 INTERVIEW QUESTIONS & CODE CHALLENGES

### **Question 1: Predict the Output**
```javascript
console.log('1');
setTimeout(() => console.log('2'), 0);
process.nextTick(() => console.log('3'));
setImmediate(() => console.log('4'));
console.log('5');

// Answer: 1, 5, 3, 4, 2 (or 2, 4 depending on timing)
```

### **Question 2: setTimeout vs setImmediate**
```javascript
const fs = require('fs');

// Main thread - order not guaranteed
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));

// I/O callback - setImmediate always first  
fs.readFile(__filename, () => {
  setTimeout(() => console.log('timeout in I/O'), 0);
  setImmediate(() => console.log('immediate in I/O'));
});

/* Answer:
- Main: timeout/immediate (either order)  
- I/O: immediate in I/O → timeout in I/O (guaranteed)
*/
```

### **Question 3: NextTick Starvation**
```javascript
let count = 0;
function recursiveNextTick() {
  console.log(`NextTick: ${++count}`);
  if (count < 10) {
    process.nextTick(recursiveNextTick);
  }
}

setTimeout(() => console.log('Timer will run?'), 0);
recursiveNextTick();

// Answer: NextTick 1-10 will run first, then Timer
// If count limit removed, Timer would never run (starvation)
```

### **Question 4: Complex Execution Order**
```javascript
console.log('start');

setTimeout(() => {
  console.log('timer1');
  Promise.resolve().then(() => console.log('promise1'));
}, 0);

setTimeout(() => {
  console.log('timer2');  
  process.nextTick(() => console.log('nextTick1'));
}, 0);

setImmediate(() => {
  console.log('immediate1');
  process.nextTick(() => {
    console.log('nextTick2');
    Promise.resolve().then(() => console.log('promise2'));
  });
});

console.log('end');

/* Answer:
start
end  
immediate1
nextTick2
promise2
timer1 (or timer2 first)
promise1 (or nextTick1 first)  
timer2 (or timer1 first)
nextTick1 (or promise1 first)
*/
```

### **Question 5: Thread Pool Saturation**
```javascript
const fs = require('fs');
const start = Date.now();

// This will use all 4 default thread pool workers
for (let i = 0; i < 4; i++) {
  fs.readFile(__filename, () => {
    console.log(`Fast read ${i}: ${Date.now() - start}ms`);
  });
}

// These will wait for thread pool workers to be available
for (let i = 0; i < 2; i++) {
  fs.readFile(__filename, () => {
    console.log(`Slow read ${i}: ${Date.now() - start}ms`);
  });
}

// Network I/O doesn't use thread pool - will be fast
require('http').get('http://google.com', () => {
  console.log(`HTTP request: ${Date.now() - start}ms`);
});
```

---

## ✅ EVENT LOOP CHECKLIST

### **Core Concepts:**
- [ ] 6 phases của Event Loop và thứ tự
- [ ] Call Stack vs Callback Queue vs Microtask Queue
- [ ] process.nextTick vs Promise microtasks
- [ ] setTimeout(0) vs setImmediate() differences
- [ ] Thread pool vs true async I/O

### **Performance:**
- [ ] Event loop lag measurement
- [ ] Blocking operations detection
- [ ] NextTick starvation prevention
- [ ] Thread pool sizing và monitoring
- [ ] CPU-intensive task handling strategies

### **Advanced Topics:**
- [ ] Worker Threads vs Cluster vs Child Process
- [ ] Async hooks và debugging
- [ ] Node.js vs Browser Event Loop differences
- [ ] Timer accuracy và resolution
- [ ] Close callbacks và cleanup

### **Practical Skills:**
- [ ] Predict execution order của complex code
- [ ] Debug event loop performance issues
- [ ] Implement non-blocking alternatives
- [ ] Proper error handling trong async context
- [ ] Choose right concurrency pattern

---

## 🚀 PRACTICAL EXERCISES

1. **Order Prediction**: Practice với 20+ code snippets khác nhau
2. **Performance Monitoring**: Implement event loop lag detection
3. **Non-blocking Algorithms**: Convert blocking operations to chunked processing
4. **Worker Implementation**: Build CPU-intensive task với Worker Threads
5. **Debugging Practice**: Use async_hooks để trace async operations

---

## 📚 ADVANCED RESOURCES

- **Official Node.js Docs**: [Event Loop Guide](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)
- **libuv Documentation**: [Design Overview](http://docs.libuv.org/en/v1.x/design.html)
- **Performance Tools**: clinic.js, node --prof, Chrome DevTools
- **Books**: "Node.js Design Patterns" Chapter 4, "Node.js in Action" Event Loop sections
