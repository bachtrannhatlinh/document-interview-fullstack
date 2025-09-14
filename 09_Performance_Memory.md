# 9. Performance & Memory

## Memory Management

### Understanding Node.js Memory Structure
```js
// Memory usage breakdown
const memUsage = process.memoryUsage();
console.log({
  rss: memUsage.rss,           // Resident Set Size - total memory allocated
  heapTotal: memUsage.heapTotal, // Total heap allocated
  heapUsed: memUsage.heapUsed,   // Actual heap used
  external: memUsage.external,   // C++ objects bound to JS objects
  arrayBuffers: memUsage.arrayBuffers // ArrayBuffers and SharedArrayBuffers
});

// V8 heap statistics
const v8 = require('v8');
const heapStats = v8.getHeapStatistics();
console.log({
  totalHeapSize: heapStats.total_heap_size,
  totalHeapSizeExecutable: heapStats.total_heap_size_executable,
  usedHeapSize: heapStats.used_heap_size,
  heapSizeLimit: heapStats.heap_size_limit,
  mallocedMemory: heapStats.malloced_memory
});
```

### Memory Leak Detection & Prevention

#### Common Memory Leak Patterns
```js
// ❌ Global variables accumulation
global.cache = {}; // Never cleaned up
function addToCache(key, value) {
  global.cache[key] = value; // Grows indefinitely
}

// ✅ Use proper cache with size limit
const cache = new Map();
const MAX_CACHE_SIZE = 1000;

function addToCache(key, value) {
  if (cache.size >= MAX_CACHE_SIZE) {
    const firstKey = cache.keys().next().value;
    cache.delete(firstKey);
  }
  cache.set(key, value);
}
```

```js
// ❌ Event listener leaks
const EventEmitter = require('events');
const emitter = new EventEmitter();

function createHandler() {
  const handler = () => console.log('Event handled');
  emitter.on('data', handler);
  // Forgot to remove listener!
}

// ✅ Proper cleanup
function createHandler() {
  const handler = () => console.log('Event handled');
  emitter.on('data', handler);
  
  // Cleanup function
  return () => emitter.removeListener('data', handler);
}

const cleanup = createHandler();
// Later...
cleanup();
```

```js
// ❌ Closure leaks
function createClosure() {
  const largeData = new Array(1000000).fill('data');
  
  return function() {
    console.log('Function called');
    // largeData is still referenced even if not used
  };
}

// ✅ Avoid unnecessary closures
function createClosure() {
  const largeData = new Array(1000000).fill('data');
  
  function processData() {
    // Use largeData here
    return largeData.length;
  }
  
  // Return only what's needed
  return function() {
    console.log('Function called');
  };
}
```

#### Memory Monitoring Tools
```js
// Memory monitoring middleware
const memoryMonitor = (req, res, next) => {
  const startMemory = process.memoryUsage();
  
  res.on('finish', () => {
    const endMemory = process.memoryUsage();
    const memoryDiff = {
      rss: endMemory.rss - startMemory.rss,
      heapTotal: endMemory.heapTotal - startMemory.heapTotal,
      heapUsed: endMemory.heapUsed - startMemory.heapUsed
    };
    
    if (memoryDiff.heapUsed > 10 * 1024 * 1024) { // 10MB threshold
      console.warn('High memory usage detected:', {
        url: req.originalUrl,
        method: req.method,
        memoryDiff
      });
    }
  });
  
  next();
};
```

```js
// Heap snapshot comparison
const v8 = require('v8');
const fs = require('fs');

class HeapAnalyzer {
  constructor() {
    this.snapshots = [];
  }
  
  takeSnapshot(label = 'snapshot') {
    const filename = `heap-${label}-${Date.now()}.heapsnapshot`;
    v8.writeHeapSnapshot(filename);
    
    this.snapshots.push({
      label,
      filename,
      timestamp: new Date(),
      memUsage: process.memoryUsage()
    });
    
    return filename;
  }
  
  getMemoryTrend() {
    if (this.snapshots.length < 2) return null;
    
    const recent = this.snapshots.slice(-5); // Last 5 snapshots
    return recent.map((snapshot, index) => ({
      label: snapshot.label,
      heapUsed: snapshot.memUsage.heapUsed,
      growth: index > 0 ? snapshot.memUsage.heapUsed - recent[index-1].memUsage.heapUsed : 0
    }));
  }
}
```

## Performance Profiling

### CPU Profiling
```js
// CPU profiling with built-in profiler
const { Session } = require('inspector');
const fs = require('fs');

class CPUProfiler {
  constructor() {
    this.session = new Session();
    this.session.connect();
  }
  
  async startProfiling() {
    await this.session.post('Profiler.enable');
    await this.session.post('Profiler.start');
  }
  
  async stopProfiling(filename = 'cpu-profile.cpuprofile') {
    const { profile } = await this.session.post('Profiler.stop');
    fs.writeFileSync(filename, JSON.stringify(profile));
    return filename;
  }
}

// Usage
const profiler = new CPUProfiler();
await profiler.startProfiling();

// Run your code here
performHeavyOperation();

await profiler.stopProfiling('operation-profile.cpuprofile');
```

### Performance Timing
```js
const { performance, PerformanceObserver } = require('perf_hooks');

// Mark and measure operations
function measureOperation() {
  performance.mark('operation-start');
  
  // Your operation here
  heavyComputation();
  
  performance.mark('operation-end');
  performance.measure('operation', 'operation-start', 'operation-end');
}

// Observe performance entries
const obs = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    console.log(`${entry.name}: ${entry.duration}ms`);
  });
});
obs.observe({ entryTypes: ['measure'] });

measureOperation();
```

### Event Loop Monitoring
```js
// Event loop lag monitoring
function measureEventLoopLag() {
  const start = process.hrtime.bigint();
  
  setImmediate(() => {
    const lag = process.hrtime.bigint() - start;
    const lagMs = Number(lag / 1000000n); // Convert to milliseconds
    
    if (lagMs > 10) { // Alert if lag > 10ms
      console.warn(`Event loop lag detected: ${lagMs}ms`);
    }
    
    // Continue monitoring
    setTimeout(measureEventLoopLag, 1000);
  });
}

measureEventLoopLag();
```

## Garbage Collection Optimization

### Understanding GC Types
```js
// Monitor GC events
const v8 = require('v8');

// Enable GC monitoring
process.on('exit', () => {
  console.log('GC Statistics:', v8.getHeapStatistics());
});

// Force garbage collection (only available with --expose-gc flag)
// node --expose-gc app.js
if (global.gc) {
  global.gc();
  console.log('Forced GC completed');
}
```

### Memory-Efficient Patterns
```js
// ✅ Object pooling to reduce GC pressure
class ObjectPool {
  constructor(createFn, resetFn, initialSize = 10) {
    this.createFn = createFn;
    this.resetFn = resetFn;
    this.pool = [];
    
    // Pre-populate pool
    for (let i = 0; i < initialSize; i++) {
      this.pool.push(this.createFn());
    }
  }
  
  acquire() {
    return this.pool.length > 0 ? this.pool.pop() : this.createFn();
  }
  
  release(obj) {
    this.resetFn(obj);
    this.pool.push(obj);
  }
}

// Example usage
const bufferPool = new ObjectPool(
  () => Buffer.alloc(1024),
  (buf) => buf.fill(0),
  50
);

function processData(data) {
  const buffer = bufferPool.acquire();
  try {
    // Use buffer
    buffer.write(data);
    return buffer.toString();
  } finally {
    bufferPool.release(buffer);
  }
}
```

```js
// ✅ Streaming for large data processing
const fs = require('fs');
const { Transform } = require('stream');

// ❌ Loading entire file into memory
function processFileBad(filename) {
  const data = fs.readFileSync(filename, 'utf8');
  return data.split('\n').map(line => line.toUpperCase()).join('\n');
}

// ✅ Streaming approach
function processFileGood(inputFile, outputFile) {
  const upperCaseTransform = new Transform({
    transform(chunk, encoding, callback) {
      const processed = chunk.toString().toUpperCase();
      callback(null, processed);
    }
  });
  
  fs.createReadStream(inputFile)
    .pipe(upperCaseTransform)
    .pipe(fs.createWriteStream(outputFile));
}
```

## Performance Benchmarking

### Built-in Benchmarking
```js
// Simple benchmark function
function benchmark(fn, iterations = 1000) {
  const start = performance.now();
  
  for (let i = 0; i < iterations; i++) {
    fn();
  }
  
  const end = performance.now();
  return {
    totalTime: end - start,
    averageTime: (end - start) / iterations,
    iterations
  };
}

// Compare different implementations
function compareImplementations() {
  const results = {
    forEach: benchmark(() => {
      const arr = Array.from({length: 1000}, (_, i) => i);
      let sum = 0;
      arr.forEach(x => sum += x);
    }),
    
    forLoop: benchmark(() => {
      const arr = Array.from({length: 1000}, (_, i) => i);
      let sum = 0;
      for (let i = 0; i < arr.length; i++) {
        sum += arr[i];
      }
    }),
    
    reduce: benchmark(() => {
      const arr = Array.from({length: 1000}, (_, i) => i);
      arr.reduce((sum, x) => sum + x, 0);
    })
  };
  
  console.table(results);
}

compareImplementations();
```

### Load Testing Tools Integration
```js
// Autocannon programmatic usage
const autocannon = require('autocannon');

async function loadTest() {
  const instance = autocannon({
    url: 'http://localhost:3000',
    connections: 10,
    pipelining: 1,
    duration: 10
  });
  
  autocannon.track(instance, {
    renderProgressBar: true
  });
  
  const result = await instance;
  console.log('Load test results:', result);
  
  return result;
}

// Clinic.js integration (run via CLI)
// npx clinic doctor -- node app.js
// npx clinic flame -- node app.js
// npx clinic bubbleprof -- node app.js
```

## Production Monitoring

### APM Integration
```js
// New Relic integration example
require('newrelic');

// Custom metrics
const newrelic = require('newrelic');

function trackCustomMetric(name, value) {
  newrelic.recordMetric(name, value);
}

// Track memory usage
setInterval(() => {
  const memUsage = process.memoryUsage();
  trackCustomMetric('Memory/Heap/Used', memUsage.heapUsed);
  trackCustomMetric('Memory/Heap/Total', memUsage.heapTotal);
  trackCustomMetric('Memory/RSS', memUsage.rss);
}, 30000);
```

### Resource Monitoring
```js
// Comprehensive resource monitor
class ResourceMonitor {
  constructor(options = {}) {
    this.interval = options.interval || 30000;
    this.thresholds = {
      memory: options.memoryThreshold || 500 * 1024 * 1024, // 500MB
      cpu: options.cpuThreshold || 80, // 80%
      eventLoopLag: options.lagThreshold || 100 // 100ms
    };
    this.startTime = Date.now();
  }
  
  start() {
    this.timer = setInterval(() => {
      this.collectMetrics();
    }, this.interval);
  }
  
  stop() {
    if (this.timer) {
      clearInterval(this.timer);
    }
  }
  
  collectMetrics() {
    const memUsage = process.memoryUsage();
    const cpuUsage = process.cpuUsage();
    
    const metrics = {
      timestamp: new Date().toISOString(),
      uptime: Date.now() - this.startTime,
      memory: {
        rss: memUsage.rss,
        heapTotal: memUsage.heapTotal,
        heapUsed: memUsage.heapUsed,
        external: memUsage.external
      },
      cpu: {
        user: cpuUsage.user,
        system: cpuUsage.system
      }
    };
    
    this.checkThresholds(metrics);
    this.logMetrics(metrics);
  }
  
  checkThresholds(metrics) {
    if (metrics.memory.heapUsed > this.thresholds.memory) {
      console.warn('Memory threshold exceeded:', metrics.memory.heapUsed);
    }
  }
  
  logMetrics(metrics) {
    console.log('Resource metrics:', JSON.stringify(metrics, null, 2));
  }
}
```

## Common Interview Questions

### Q1: Node.js memory leaks có thể xảy ra ở đâu?
**Trả lời:**
- **Global variables**: Tích lũy data trong global scope
- **Event listeners**: Không remove listeners khi không cần
- **Closures**: Giữ reference đến large objects
- **Timers**: setInterval/setTimeout không clear
- **Callbacks**: Circular references trong callbacks

```js
// Example of closure memory leak
function createLeak() {
  const largeArray = new Array(1000000);
  return function() {
    // largeArray is still referenced
  };
}
```

### Q2: Làm sao phát hiện và fix memory leaks?
**Trả lời:**
- **Monitoring**: process.memoryUsage(), heap snapshots
- **Tools**: Chrome DevTools, heapdump, clinic.js
- **Patterns**: WeakMap/WeakSet, proper cleanup
- **Testing**: Load testing để detect leaks

### Q3: Event Loop lag ảnh hưởng như nào?
**Trả lời:**
- **Performance**: Request response time tăng
- **Concurrency**: Không xử lý được multiple requests
- **User experience**: App trở nên unresponsive
- **Solutions**: Break up CPU-intensive tasks, use Worker threads

### Q4: Garbage Collection tối ưu như nào?
**Trả lời:**
- **Object pooling**: Reuse objects thay vì create new
- **Avoid large objects**: Break into smaller chunks
- **Proper references**: Avoid circular references
- **Timing**: Understand generational GC (young vs old generation)

### Q5: Streaming vs loading toàn bộ data?
**Trả lời:**
```js
// ❌ Memory intensive
const data = fs.readFileSync('large-file.txt');

// ✅ Memory efficient
fs.createReadStream('large-file.txt')
  .pipe(processTransform)
  .pipe(outputStream);
```
- **Memory usage**: Constant vs linear growth
- **Performance**: Better for large files
- **Scalability**: Handle multiple large requests

### Q6: CPU profiling tools nào dùng?
**Trả lời:**
- **Built-in**: node --prof, --cpu-prof
- **Chrome DevTools**: --inspect flag
- **Third-party**: clinic.js, 0x, perf
- **APM**: New Relic, DataDog, AppDynamics
