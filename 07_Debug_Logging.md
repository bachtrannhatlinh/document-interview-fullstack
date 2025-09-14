# 7. Debug & Logging

## Debug Techniques

### Console-based Debugging
```js
// Basic console methods
console.log('Debug info:', data);
console.error('Error occurred:', error);
console.warn('Warning:', warning);
console.table(arrayOrObject);
console.time('operation');
console.timeEnd('operation');
console.trace(); // Stack trace
console.assert(condition, 'Message if false');
```

### Advanced Debugging Tools

#### VSCode Debugging
```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch Program",
      "program": "${workspaceFolder}/app.js",
      "request": "launch",
      "skipFiles": ["<node_internals>/**"],
      "type": "node"
    }
  ]
}
```

#### Chrome DevTools
```bash
# Start Node.js with inspector
node --inspect app.js
node --inspect-brk app.js  # Break on start

# Remote debugging
node --inspect=0.0.0.0:9229 app.js
```

#### Debug Module
```js
const debug = require('debug')('app:main');
debug('Server starting...');

// Usage: DEBUG=app:* node app.js
```

## Professional Logging

### Winston Configuration
```js
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'user-service' },
  transports: [
    new winston.transports.File({ 
      filename: 'logs/error.log', 
      level: 'error' 
    }),
    new winston.transports.File({ 
      filename: 'logs/combined.log' 
    })
  ],
});

// Production vs Development
if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}

// Usage
logger.error('Error message', { error: err });
logger.warn('Warning message');
logger.info('Info message');
logger.debug('Debug message');
```

### Morgan (HTTP Logging)
```js
const morgan = require('morgan');
const winston = require('winston');

// Custom format
morgan.token('body', (req, res) => JSON.stringify(req.body));

app.use(morgan('combined'));
app.use(morgan(':method :url :status :body', {
  stream: {
    write: (message) => logger.info(message.trim())
  }
}));
```

### Pino (High Performance)
```js
const pino = require('pino');

const logger = pino({
  level: 'info',
  prettyPrint: process.env.NODE_ENV !== 'production',
  timestamp: () => `,"time":"${new Date().toISOString()}"`
});

logger.info({ user: 'john' }, 'User logged in');
logger.error({ err: error }, 'Database connection failed');
```

## Logging Strategies

### Structured Logging
```js
// Good - Structured
logger.info('User action', {
  userId: user.id,
  action: 'purchase',
  productId: product.id,
  amount: order.total,
  timestamp: new Date().toISOString()
});

// Bad - Unstructured
logger.info(`User ${user.id} purchased product ${product.id} for ${order.total}`);
```

### Log Levels Hierarchy
```js
// FATAL > ERROR > WARN > INFO > DEBUG > TRACE
const logLevels = {
  fatal: 0,    // System unusable
  error: 1,    // Error conditions
  warn: 2,     // Warning conditions
  info: 3,     // Informational messages
  debug: 4,    // Debug-level messages
  trace: 5     // Very detailed debug info
};
```

### Context Propagation
```js
const { AsyncLocalStorage } = require('async_hooks');
const asyncLocalStorage = new AsyncLocalStorage();

// Middleware to set request context
app.use((req, res, next) => {
  const requestId = req.headers['x-request-id'] || generateId();
  asyncLocalStorage.run({ requestId }, next);
});

// Logger with context
function log(level, message, data = {}) {
  const context = asyncLocalStorage.getStore() || {};
  logger[level](message, { ...data, requestId: context.requestId });
}
```

## Debugging Best Practices

### Error Handling & Debugging
```js
// Proper error logging
try {
  await riskyOperation();
} catch (error) {
  logger.error('Operation failed', {
    error: error.message,
    stack: error.stack,
    context: { userId, operation: 'user_update' }
  });
  throw new CustomError('User update failed', error);
}

// Debug middleware
const debugMiddleware = (req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    logger.debug('Request completed', {
      method: req.method,
      url: req.originalUrl,
      statusCode: res.statusCode,
      duration
    });
  });
  
  next();
};
```

### Memory Debugging
```js
// Memory usage monitoring
setInterval(() => {
  const usage = process.memoryUsage();
  logger.debug('Memory usage', {
    rss: Math.round(usage.rss / 1024 / 1024) + ' MB',
    heapTotal: Math.round(usage.heapTotal / 1024 / 1024) + ' MB',
    heapUsed: Math.round(usage.heapUsed / 1024 / 1024) + ' MB'
  });
}, 30000);

// Heap dump for memory leaks
const v8 = require('v8');
const fs = require('fs');

function takeHeapSnapshot() {
  const snapshot = v8.writeHeapSnapshot();
  logger.info('Heap snapshot saved', { file: snapshot });
}
```

## Production Logging

### Log Rotation
```js
const DailyRotateFile = require('winston-daily-rotate-file');

const transport = new DailyRotateFile({
  filename: 'logs/application-%DATE%.log',
  datePattern: 'YYYY-MM-DD',
  maxSize: '20m',
  maxFiles: '14d'
});

logger.add(transport);
```

### Centralized Logging
```js
// ELK Stack integration
const ElasticsearchTransport = require('winston-elasticsearch');

logger.add(new ElasticsearchTransport({
  level: 'info',
  clientOpts: { node: 'http://localhost:9200' },
  index: 'logs'
}));

// Fluentd integration
const FluentTransport = require('fluent-logger').support.winstonTransport();
logger.add(new FluentTransport('myapp', {
  host: 'localhost',
  port: 24224
}));
```

## Common Interview Questions

### Q1: Phân biệt console.log và logging library?
**Trả lời:**
- Console.log: Đơn giản, sync, không có level, không rotation
- Logging library: Async, có levels, rotation, multiple transports, structured logging

### Q2: Làm sao debug async/await code?
```js
// Bad
async function badExample() {
  const result = await api.call();
  console.log(result); // Không biết context
}

// Good
async function goodExample() {
  try {
    logger.debug('Starting API call', { endpoint: '/users' });
    const result = await api.call();
    logger.debug('API call successful', { result });
    return result;
  } catch (error) {
    logger.error('API call failed', { error: error.message });
    throw error;
  }
}
```

### Q3: Performance impact of logging?
**Trả lời:**
- Sync logging blocks event loop
- Async logging: Better performance
- Log level filtering
- Disable debug logs in production

```js
// Conditional logging
if (logger.isLevelEnabled('debug')) {
  logger.debug('Expensive operation result', expensiveCalculation());
}
```

### Q4: How to debug memory leaks?
```js
// Tools: node --inspect + Chrome DevTools
// Heap snapshots comparison
// Monitor RSS vs Heap
// Use WeakMap/WeakSet for references
// Clean up event listeners

process.on('SIGTERM', () => {
  logger.info('Graceful shutdown initiated');
  // Cleanup operations
  server.close();
});
```
