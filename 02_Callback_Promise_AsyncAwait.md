# 2. CALLBACK, PROMISE, ASYNC/AWAIT - INTERVIEW GUIDE

## 🔄 EVENT LOOP & ASYNCHRONOUS EXECUTION

### Microtasks vs Macrotasks Priority
```javascript
console.log('1: Synchronous');

setTimeout(() => console.log('2: Macrotask (Timer)'), 0);

Promise.resolve().then(() => console.log('3: Microtask (Promise)'));

process.nextTick(() => console.log('4: NextTick (Highest Priority)'));

setImmediate(() => console.log('5: Check Phase'));

console.log('6: Synchronous');

/* Output:
1: Synchronous
6: Synchronous  
4: NextTick (Highest Priority)
3: Microtask (Promise)
5: Check Phase
2: Macrotask (Timer)
*/
```

### Execution Context trong Event Loop
```javascript
// Promise callbacks are microtasks - executed before next macrotask
Promise.resolve().then(() => {
  console.log('Promise 1');
  return Promise.resolve();
}).then(() => {
  console.log('Promise 2');
});

setTimeout(() => {
  console.log('Timer 1');
  Promise.resolve().then(() => console.log('Promise in Timer'));
}, 0);

setTimeout(() => console.log('Timer 2'), 0);

/* Output:
Promise 1
Promise 2
Timer 1
Promise in Timer
Timer 2
*/
```

---

## 📞 CALLBACKS - ERROR-FIRST PATTERN

### Traditional Callback Pattern
```javascript
const fs = require('fs');

// Error-first callback convention
function readFileCallback(filename, callback) {
  fs.readFile(filename, 'utf8', (err, data) => {
    if (err) {
      return callback(err, null); // Error first
    }
    callback(null, data); // Success: null error, actual data
  });
}

// Usage
readFileCallback('config.json', (err, data) => {
  if (err) {
    console.error('Failed to read file:', err.message);
    return;
  }
  console.log('File content:', data);
});
```

### Callback Hell Problem
```javascript
// ❌ Callback Hell - "Pyramid of Doom"
const fs = require('fs');

function processFiles() {
  fs.readFile('file1.txt', 'utf8', (err1, data1) => {
    if (err1) throw err1;
    
    fs.readFile('file2.txt', 'utf8', (err2, data2) => {
      if (err2) throw err2;
      
      fs.readFile('file3.txt', 'utf8', (err3, data3) => {
        if (err3) throw err3;
        
        fs.writeFile('output.txt', data1 + data2 + data3, (err4) => {
          if (err4) throw err4;
          console.log('All files processed!');
        });
      });
    });
  });
}

// ✅ Solutions: Named functions, async library, or Promises
function readFile1() {
  fs.readFile('file1.txt', 'utf8', (err, data) => {
    if (err) throw err;
    readFile2(data);
  });
}

function readFile2(data1) {
  fs.readFile('file2.txt', 'utf8', (err, data2) => {
    if (err) throw err;
    readFile3(data1, data2);
  });
}

// ... and so on
```

### Inversion of Control Problem
```javascript
// ❌ Problem: We lose control over when/how callback is executed
function trackAnalytics(data, callback) {
  // Third-party code - we don't control:
  // - When callback is called (immediately? never? multiple times?)
  // - What arguments it receives
  // - Error handling
  thirdPartyService.send(data, callback);
}

trackAnalytics(userData, (err, result) => {
  if (!err) {
    // This might run 0, 1, or multiple times!
    chargeUserCreditCard(); // 💸 Dangerous!
  }
});

// ✅ Solution: Promise provides controlled inversion
function trackAnalyticsPromise(data) {
  return new Promise((resolve, reject) => {
    thirdPartyService.send(data, (err, result) => {
      if (err) reject(err);
      else resolve(result);
    });
  });
}
```

---

## 🎯 PROMISES - DETAILED MECHANICS

### Promise States & Transitions
```javascript
// Promise State Machine
const promise = new Promise((resolve, reject) => {
  // Initially: PENDING
  console.log('Promise state: PENDING');
  
  setTimeout(() => {
    if (Math.random() > 0.5) {
      resolve('Success!'); // -> FULFILLED (immutable)
    } else {
      reject('Error!');   // -> REJECTED (immutable)
    }
  }, 1000);
});

promise
  .then(value => {
    console.log('Fulfilled:', value);
    return 'Transformed value'; // Return new value
  })
  .then(value => {
    console.log('Chained:', value);
    throw new Error('Something went wrong'); // Throw error
  })
  .catch(error => {
    console.log('Caught:', error.message);
    return 'Recovery value'; // Recover from error
  })
  .finally(() => {
    console.log('Cleanup always runs');
  });
```

### Promise Chaining Rules
```javascript
// Rule 1: Always return from .then()
Promise.resolve(1)
  .then(value => {
    console.log(value); // 1
    return value * 2;   // Must return for chaining
  })
  .then(value => {
    console.log(value); // 2
    // Missing return -> next .then() gets undefined
  })
  .then(value => {
    console.log(value); // undefined
  });

// Rule 2: Returning Promise in .then()
Promise.resolve(1)
  .then(value => {
    return new Promise(resolve => {
      setTimeout(() => resolve(value * 2), 100);
    });
  })
  .then(value => {
    console.log(value); // 2 (after 100ms)
  });

// Rule 3: Error propagation
Promise.resolve(1)
  .then(value => {
    throw new Error('Oops!');
  })
  .then(value => {
    console.log('This will not run');
  })
  .catch(error => {
    console.log('Caught:', error.message); // Caught: Oops!
    return 'recovered';
  })
  .then(value => {
    console.log(value); // recovered
  });
```

### Promise Static Methods
```javascript
const promise1 = new Promise(resolve => setTimeout(() => resolve('A'), 100));
const promise2 = new Promise(resolve => setTimeout(() => resolve('B'), 200));
const promise3 = new Promise((_, reject) => setTimeout(() => reject('C failed'), 150));

// Promise.all - Fail fast, preserve order
Promise.all([promise1, promise2])
  .then(results => console.log('All resolved:', results)) // ['A', 'B']
  .catch(err => console.log('One failed:', err));

Promise.all([promise1, promise3])
  .then(results => console.log('Will not run'))
  .catch(err => console.log('Failed:', err)); // Failed: C failed

// Promise.allSettled - Never fails, returns all results
Promise.allSettled([promise1, promise2, promise3])
  .then(results => {
    console.log(results);
    /* [
      { status: 'fulfilled', value: 'A' },
      { status: 'fulfilled', value: 'B' }, 
      { status: 'rejected', reason: 'C failed' }
    ] */
  });

// Promise.race - First to settle wins
Promise.race([promise1, promise2, promise3])
  .then(result => console.log('Race winner:', result)) // 'A' (fastest to resolve)
  .catch(err => console.log('Race failed:', err));

// Promise.any - First successful resolution
Promise.any([promise3, promise1, promise2])
  .then(result => console.log('First success:', result)) // 'A'
  .catch(err => console.log('All failed:', err));
```

### Advanced Promise Patterns
```javascript
// Timeout wrapper
function withTimeout(promise, timeoutMs) {
  const timeout = new Promise((_, reject) => 
    setTimeout(() => reject(new Error('Operation timed out')), timeoutMs)
  );
  
  return Promise.race([promise, timeout]);
}

// Retry with exponential backoff
async function retryWithBackoff(operation, maxRetries = 3, baseDelay = 1000) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error) {
      if (attempt === maxRetries) throw error;
      
      const delay = baseDelay * Math.pow(2, attempt - 1);
      console.log(`Attempt ${attempt} failed, retrying in ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Promisify callback functions
const { promisify } = require('util');
const fs = require('fs');

const readFilePromise = promisify(fs.readFile);
// Or manually:
function promisifyCallback(callbackFn) {
  return function(...args) {
    return new Promise((resolve, reject) => {
      callbackFn(...args, (err, result) => {
        if (err) reject(err);
        else resolve(result);
      });
    });
  };
}
```

### Unhandled Promise Rejection
```javascript
// ❌ Dangerous - silent failure
Promise.reject('Unhandled error'); // Node.js will warn and might exit

// ✅ Global error handling
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Promise Rejection:', reason);
  console.error('Promise:', promise);
  // Log to monitoring service
  // Graceful shutdown if necessary
});

process.on('uncaughtException', (error) => {
  console.error('Uncaught Exception:', error);
  process.exit(1); // Must exit, process is in undefined state
});

// ✅ Proper error handling
async function properErrorHandling() {
  try {
    await riskyOperation();
  } catch (error) {
    console.error('Handled error:', error);
    // Recover or propagate as needed
  }
}
```

---

## 🚀 ASYNC/AWAIT - ADVANCED PATTERNS

### Async/Await Mechanics
```javascript
// Async function always returns Promise
async function asyncFunction() {
  return 'Hello'; // Equivalent to Promise.resolve('Hello')
}

asyncFunction().then(value => console.log(value)); // Hello

// Error throwing becomes Promise rejection
async function errorFunction() {
  throw new Error('Something went wrong'); // Equivalent to Promise.reject()
}

errorFunction().catch(err => console.log(err.message)); // Something went wrong

// await only works inside async functions
async function awaitExample() {
  const result = await Promise.resolve('Awaited value');
  console.log(result); // Awaited value
}
```

### Sequential vs Parallel Execution
```javascript
const delay = ms => new Promise(resolve => setTimeout(resolve, ms));

// ❌ Sequential execution - inefficient
async function sequentialExecution() {
  console.time('Sequential');
  
  const result1 = await delay(100); // Wait 100ms
  const result2 = await delay(100); // Wait another 100ms
  const result3 = await delay(100); // Wait another 100ms
  
  console.timeEnd('Sequential'); // ~300ms
}

// ✅ Parallel execution - efficient
async function parallelExecution() {
  console.time('Parallel');
  
  // Start all operations simultaneously
  const promise1 = delay(100);
  const promise2 = delay(100);
  const promise3 = delay(100);
  
  // Wait for all to complete
  const results = await Promise.all([promise1, promise2, promise3]);
  
  console.timeEnd('Parallel'); // ~100ms
}

// ✅ Controlled parallelism - batch processing
async function batchProcess(items, batchSize = 3) {
  const results = [];
  
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchPromises = batch.map(item => processItem(item));
    const batchResults = await Promise.all(batchPromises);
    results.push(...batchResults);
  }
  
  return results;
}
```

### Error Handling Patterns
```javascript
// Try-catch with async/await
async function errorHandlingPatterns() {
  // Pattern 1: Basic try-catch
  try {
    const result = await riskyOperation();
    console.log('Success:', result);
  } catch (error) {
    console.error('Error:', error.message);
  }
  
  // Pattern 2: Multiple operations with individual error handling
  let result1, result2;
  
  try {
    result1 = await operation1();
  } catch (error) {
    console.error('Operation 1 failed:', error.message);
    result1 = getDefaultValue1();
  }
  
  try {
    result2 = await operation2();
  } catch (error) {
    console.error('Operation 2 failed:', error.message);
    result2 = getDefaultValue2();
  }
  
  // Pattern 3: Conditional error handling
  try {
    const result = await criticalOperation();
    return result;
  } catch (error) {
    if (error.code === 'NETWORK_ERROR') {
      // Retry logic
      return await retryOperation();
    }
    throw error; // Re-throw if can't handle
  }
}

// Finally clause
async function withFinally() {
  let resource;
  try {
    resource = await acquireResource();
    return await processResource(resource);
  } catch (error) {
    console.error('Processing failed:', error);
    throw error;
  } finally {
    if (resource) {
      await releaseResource(resource); // Always cleanup
    }
  }
}
```

### Async Iteration & Streams
```javascript
// for-await-of with async iterables
async function* asyncGenerator() {
  for (let i = 0; i < 3; i++) {
    await delay(100);
    yield i;
  }
}

async function consumeAsyncIterable() {
  for await (const value of asyncGenerator()) {
    console.log('Generated:', value);
  }
}

// Stream processing
const fs = require('fs');
const { pipeline } = require('stream/promises');

async function processLargeFile() {
  try {
    await pipeline(
      fs.createReadStream('large-file.txt'),
      new Transform({
        transform(chunk, encoding, callback) {
          // Process chunk
          callback(null, chunk.toString().toUpperCase());
        }
      }),
      fs.createWriteStream('processed-file.txt')
    );
    console.log('File processed successfully');
  } catch (error) {
    console.error('Pipeline failed:', error);
  }
}
```

### Cancellation with AbortController
```javascript
// Cancellable async operations
async function cancellableOperation() {
  const controller = new AbortController();
  const { signal } = controller;
  
  // Cancel after 5 seconds
  const timeout = setTimeout(() => controller.abort(), 5000);
  
  try {
    // Fetch with cancellation
    const response = await fetch('https://api.example.com/data', { signal });
    const data = await response.json();
    
    clearTimeout(timeout);
    return data;
  } catch (error) {
    if (error.name === 'AbortError') {
      console.log('Operation was cancelled');
    }
    throw error;
  }
}

// Custom cancellable Promise
function cancellablePromise(executor) {
  const controller = new AbortController();
  
  const promise = new Promise((resolve, reject) => {
    const wrappedResolve = (value) => {
      if (!controller.signal.aborted) resolve(value);
    };
    
    const wrappedReject = (error) => {
      if (!controller.signal.aborted) reject(error);
    };
    
    controller.signal.addEventListener('abort', () => {
      reject(new Error('Operation cancelled'));
    });
    
    executor(wrappedResolve, wrappedReject, controller.signal);
  });
  
  promise.cancel = () => controller.abort();
  return promise;
}
```

---

## ⚡ CONCURRENCY CONTROL PATTERNS

### Semaphore - Limit Concurrent Operations
```javascript
class Semaphore {
  constructor(maxConcurrent) {
    this.maxConcurrent = maxConcurrent;
    this.currentConcurrent = 0;
    this.queue = [];
  }
  
  async acquire() {
    return new Promise((resolve) => {
      if (this.currentConcurrent < this.maxConcurrent) {
        this.currentConcurrent++;
        resolve();
      } else {
        this.queue.push(resolve);
      }
    });
  }
  
  release() {
    this.currentConcurrent--;
    if (this.queue.length > 0) {
      this.currentConcurrent++;
      const resolve = this.queue.shift();
      resolve();
    }
  }
  
  async execute(operation) {
    await this.acquire();
    try {
      return await operation();
    } finally {
      this.release();
    }
  }
}

// Usage: Limit API calls
const apiSemaphore = new Semaphore(5); // Max 5 concurrent API calls

async function processUrls(urls) {
  const promises = urls.map(url => 
    apiSemaphore.execute(() => fetch(url))
  );
  
  return await Promise.all(promises);
}
```

### Queue - Process Tasks Sequentially
```javascript
class AsyncQueue {
  constructor() {
    this.queue = [];
    this.processing = false;
  }
  
  async add(operation) {
    return new Promise((resolve, reject) => {
      this.queue.push({ operation, resolve, reject });
      this.process();
    });
  }
  
  async process() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    
    while (this.queue.length > 0) {
      const { operation, resolve, reject } = this.queue.shift();
      
      try {
        const result = await operation();
        resolve(result);
      } catch (error) {
        reject(error);
      }
    }
    
    this.processing = false;
  }
}

// Usage: Database operations must be sequential
const dbQueue = new AsyncQueue();

async function updateUserBalance(userId, amount) {
  return dbQueue.add(async () => {
    const user = await User.findById(userId);
    user.balance += amount;
    return await user.save();
  });
}
```

### Circuit Breaker Pattern
```javascript
class CircuitBreaker {
  constructor(threshold = 5, timeout = 60000, monitor = 10000) {
    this.threshold = threshold;
    this.timeout = timeout;
    this.monitor = monitor;
    this.failures = 0;
    this.lastFailTime = null;
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
  }
  
  async execute(operation) {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailTime > this.timeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }
    
    try {
      const result = await operation();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  onSuccess() {
    this.failures = 0;
    if (this.state === 'HALF_OPEN') {
      this.state = 'CLOSED';
    }
  }
  
  onFailure() {
    this.failures++;
    this.lastFailTime = Date.now();
    
    if (this.failures >= this.threshold) {
      this.state = 'OPEN';
    }
  }
}

// Usage: Protect external API calls
const apiBreaker = new CircuitBreaker(3, 30000); // 3 failures, 30s timeout

async function callExternalAPI(data) {
  return apiBreaker.execute(() => fetch('/external-api', {
    method: 'POST',
    body: JSON.stringify(data)
  }));
}
```

---

## 🎯 INTERVIEW QUESTIONS & SCENARIOS

### **Question 1: Event Loop Timing**
```javascript
console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve().then(() => {
  console.log('3');
  return Promise.resolve();
}).then(() => console.log('4'));

async function test() {
  console.log('5');
  await Promise.resolve();
  console.log('6');
}

test();

console.log('7');

// Answer: 1, 5, 7, 3, 6, 4, 2
```

### **Question 2: Error Propagation**
```javascript
async function question2() {
  try {
    await Promise.resolve()
      .then(() => { throw new Error('A'); })
      .catch(() => { throw new Error('B'); })
      .then(() => console.log('Will this run?'));
  } catch (error) {
    console.log('Caught:', error.message);
  }
}

// Answer: Caught: B
// The .then() after .catch() won't run because .catch() throws
```

### **Question 3: Promise.all vs Sequential**
```javascript
const delay = ms => new Promise(resolve => setTimeout(resolve, ms));

// Scenario A
async function scenarioA() {
  console.time('A');
  await delay(100);
  await delay(100);  
  await delay(100);
  console.timeEnd('A');
}

// Scenario B  
async function scenarioB() {
  console.time('B');
  await Promise.all([
    delay(100),
    delay(100), 
    delay(100)
  ]);
  console.timeEnd('B');
}

// Answer: A takes ~300ms, B takes ~100ms
```

### **Question 4: Async/Await vs Promise Chain**
```javascript
// Convert this Promise chain to async/await:
function promiseChain() {
  return fetch('/user/1')
    .then(response => response.json())
    .then(user => fetch(`/posts/${user.id}`))
    .then(response => response.json())
    .then(posts => posts.filter(post => post.published))
    .catch(error => {
      console.error('Error:', error);
      return [];
    });
}

// Answer:
async function asyncAwaitVersion() {
  try {
    const userResponse = await fetch('/user/1');
    const user = await userResponse.json();
    
    const postsResponse = await fetch(`/posts/${user.id}`);
    const posts = await postsResponse.json();
    
    return posts.filter(post => post.published);
  } catch (error) {
    console.error('Error:', error);
    return [];
  }
}
```

### **Question 5: Memory Leaks & Cleanup**
```javascript
// Find the memory leak:
class DataProcessor {
  constructor() {
    this.cache = new Map();
    this.startPolling();
  }
  
  startPolling() {
    setInterval(async () => {
      const data = await this.fetchData();
      this.cache.set(Date.now(), data); // Memory leak!
    }, 1000);
  }
  
  async fetchData() {
    const response = await fetch('/api/data');
    return response.json();
  }
}

// Answer: Cache grows infinitely. Fix:
class DataProcessor {
  constructor() {
    this.cache = new Map();
    this.maxCacheSize = 100;
    this.intervalId = this.startPolling();
  }
  
  startPolling() {
    return setInterval(async () => {
      const data = await this.fetchData();
      this.cache.set(Date.now(), data);
      
      // Cleanup old entries
      if (this.cache.size > this.maxCacheSize) {
        const oldestKey = this.cache.keys().next().value;
        this.cache.delete(oldestKey);
      }
    }, 1000);
  }
  
  destroy() {
    clearInterval(this.intervalId);
    this.cache.clear();
  }
}
```

---

## ✅ ASYNC PROGRAMMING CHECKLIST

### **Core Concepts:**
- [ ] Event Loop phases và microtask queue
- [ ] Callback error-first convention
- [ ] Promise states và immutability
- [ ] Async/await as syntactic sugar over Promises
- [ ] Difference between Promise.all/allSettled/race/any

### **Error Handling:**
- [ ] Try-catch với async/await
- [ ] Promise rejection handling
- [ ] UnhandledPromiseRejection events
- [ ] Error propagation trong Promise chains
- [ ] Global error handlers

### **Performance Patterns:**
- [ ] Sequential vs parallel execution
- [ ] Concurrency control (semaphore, queue)
- [ ] Circuit breaker pattern
- [ ] Timeout và cancellation với AbortController
- [ ] Backpressure và stream processing

### **Advanced Topics:**
- [ ] Async iteration với for-await-of
- [ ] Custom Promise implementation
- [ ] Memory leak prevention
- [ ] Testing async code
- [ ] Debugging async stack traces

---

## 🚀 PRACTICAL EXERCISES

1. **Build Retry Logic**: Implement exponential backoff với jitter
2. **Rate Limiter**: Create API rate limiter using Promise queues
3. **Batch Processor**: Process 1000 items in batches of 10
4. **Cache Layer**: Build async cache với TTL và LRU eviction
5. **Pipeline Builder**: Create async data transformation pipeline

---

## 📚 ESSENTIAL RESOURCES

- **MDN Promise Guide**: [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)
- **Node.js Async Patterns**: [Official Guide](https://nodejs.org/en/docs/guides/blocking-vs-non-blocking/)
- **Event Loop Deep Dive**: Jake Archibald's "In The Loop" talk
- **Testing Async Code**: Jest/Mocha best practices
- **Books**: "Async JavaScript" by Trevor Burnham, "You Don't Know JS: Async & Performance"
