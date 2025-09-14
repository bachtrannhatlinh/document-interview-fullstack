# 8. Best Practices

## Core Node.js Principles

### 1. Non-blocking I/O Operations
```js
// ❌ Bad - Blocks event loop
const data = fs.readFileSync('large-file.txt', 'utf8');
console.log(data);

// ✅ Good - Non-blocking
const fsPromises = require('fs').promises;
async function readFile() {
  try {
    const data = await fsPromises.readFile('large-file.txt', 'utf8');
    console.log(data);
  } catch (error) {
    console.error('Error reading file:', error);
  }
}

// ✅ Good - Stream for large files
const stream = fs.createReadStream('large-file.txt', { encoding: 'utf8' });
stream.on('data', chunk => console.log(chunk));
stream.on('error', err => console.error(err));
```

### 2. Event Loop Optimization
```js
// ❌ Bad - CPU intensive blocking operation
function fibonacci(n) {
  if (n < 2) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

// ✅ Good - Break up work with setImmediate
function fibonacciAsync(n, callback) {
  if (n < 2) {
    setImmediate(() => callback(null, n));
    return;
  }
  
  let count = 0;
  let result = 0;
  
  function compute() {
    const start = Date.now();
    while (Date.now() - start < 10) { // Work for max 10ms
      if (count >= n) {
        return callback(null, result);
      }
      // Compute fibonacci iteratively
      count++;
    }
    setImmediate(compute); // Give other operations a chance
  }
  
  setImmediate(compute);
}
```

## Code Organization & Architecture

### 3. Clean Architecture Pattern
```js
// controllers/userController.js
class UserController {
  constructor(userService) {
    this.userService = userService;
  }
  
  async createUser(req, res) {
    try {
      const user = await this.userService.createUser(req.body);
      res.status(201).json(user);
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }
}

// services/userService.js
class UserService {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }
  
  async createUser(userData) {
    const validatedData = this.validateUser(userData);
    return await this.userRepository.create(validatedData);
  }
  
  validateUser(userData) {
    // Validation logic
    if (!userData.email) throw new Error('Email is required');
    return userData;
  }
}

// repositories/userRepository.js
class UserRepository {
  async create(userData) {
    // Database operations
  }
}
```

### 4. Dependency Injection
```js
// container.js
const UserController = require('./controllers/userController');
const UserService = require('./services/userService');
const UserRepository = require('./repositories/userRepository');

class Container {
  constructor() {
    this.dependencies = {};
    this.setupDependencies();
  }
  
  setupDependencies() {
    this.dependencies.userRepository = new UserRepository();
    this.dependencies.userService = new UserService(this.dependencies.userRepository);
    this.dependencies.userController = new UserController(this.dependencies.userService);
  }
  
  get(dependency) {
    return this.dependencies[dependency];
  }
}

module.exports = new Container();
```

## Configuration Management

### 5. Environment-based Configuration
```js
// config/index.js
require('dotenv').config();

const config = {
  port: process.env.PORT || 3000,
  nodeEnv: process.env.NODE_ENV || 'development',
  database: {
    url: process.env.DB_URL,
    maxConnections: parseInt(process.env.DB_MAX_CONNECTIONS) || 10
  },
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '1h'
  },
  redis: {
    host: process.env.REDIS_HOST || 'localhost',
    port: parseInt(process.env.REDIS_PORT) || 6379
  }
};

// Validation
const requiredEnvVars = ['DB_URL', 'JWT_SECRET'];
requiredEnvVars.forEach(envVar => {
  if (!process.env[envVar]) {
    throw new Error(`Missing required environment variable: ${envVar}`);
  }
});

module.exports = config;
```

### 6. Configuration Schema Validation
```js
const Joi = require('joi');

const configSchema = Joi.object({
  port: Joi.number().port().default(3000),
  nodeEnv: Joi.string().valid('development', 'production', 'test').default('development'),
  database: Joi.object({
    url: Joi.string().uri().required(),
    maxConnections: Joi.number().min(1).max(100).default(10)
  }),
  jwt: Joi.object({
    secret: Joi.string().min(32).required(),
    expiresIn: Joi.string().default('1h')
  })
});

const { error, value: validatedConfig } = configSchema.validate(config);
if (error) {
  throw new Error(`Config validation error: ${error.message}`);
}
```

## Error Handling

### 7. Comprehensive Error Handling
```js
// errors/customErrors.js
class AppError extends Error {
  constructor(message, statusCode, isOperational = true) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = isOperational;
    this.timestamp = new Date().toISOString();
    
    Error.captureStackTrace(this, this.constructor);
  }
}

class ValidationError extends AppError {
  constructor(message, field = null) {
    super(message, 400);
    this.field = field;
  }
}

// Global error handler
const globalErrorHandler = (err, req, res, next) => {
  let error = { ...err };
  error.message = err.message;
  
  // Log error
  logger.error('Global error handler', {
    error: error.message,
    stack: error.stack,
    url: req.originalUrl,
    method: req.method
  });
  
  // Mongoose validation error
  if (err.name === 'ValidationError') {
    const message = Object.values(err.errors).map(val => val.message).join(', ');
    error = new ValidationError(message);
  }
  
  // JWT errors
  if (err.name === 'JsonWebTokenError') {
    error = new AppError('Invalid token', 401);
  }
  
  res.status(error.statusCode || 500).json({
    success: false,
    error: error.message || 'Server Error',
    ...(process.env.NODE_ENV === 'development' && { stack: error.stack })
  });
};

// Unhandled promise rejections
process.on('unhandledRejection', (err, promise) => {
  logger.error('Unhandled Promise Rejection', { error: err.message });
  process.exit(1);
});

// Uncaught exceptions
process.on('uncaughtException', (err) => {
  logger.error('Uncaught Exception', { error: err.message });
  process.exit(1);
});
```

## Security Best Practices

### 8. Security Headers & Middleware
```js
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const mongoSanitize = require('express-mongo-sanitize');
const xss = require('xss-clean');

app.use(helmet()); // Security headers

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP'
});
app.use(limiter);

// Data sanitization
app.use(mongoSanitize()); // Against NoSQL injection
app.use(xss()); // Against XSS

// CORS configuration
const cors = require('cors');
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || 'http://localhost:3000',
  credentials: true
}));
```

### 9. Input Validation & Sanitization
```js
const validator = require('validator');
const sanitizeHtml = require('sanitize-html');

const validateInput = (schema) => {
  return (req, res, next) => {
    const { error } = schema.validate(req.body);
    if (error) {
      return res.status(400).json({
        error: error.details[0].message
      });
    }
    
    // Sanitize strings
    Object.keys(req.body).forEach(key => {
      if (typeof req.body[key] === 'string') {
        req.body[key] = sanitizeHtml(req.body[key], {
          allowedTags: [],
          allowedAttributes: {}
        });
      }
    });
    
    next();
  };
};

// Usage
const userSchema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/).required(),
  name: Joi.string().min(2).max(50).required()
});

app.post('/users', validateInput(userSchema), createUser);
```

## Performance Optimization

### 10. Caching Strategies
```js
const Redis = require('redis');
const client = Redis.createClient();

// Cache middleware
const cacheMiddleware = (duration = 300) => {
  return async (req, res, next) => {
    const key = `cache:${req.originalUrl}`;
    
    try {
      const cached = await client.get(key);
      if (cached) {
        return res.json(JSON.parse(cached));
      }
      
      // Store original res.json
      const originalJson = res.json;
      res.json = function(data) {
        // Cache the response
        client.setex(key, duration, JSON.stringify(data));
        return originalJson.call(this, data);
      };
      
      next();
    } catch (error) {
      next();
    }
  };
};

// Usage
app.get('/api/users', cacheMiddleware(600), getUsers);
```

### 11. Database Optimization
```js
// Connection pooling
const mongoose = require('mongoose');

mongoose.connect(process.env.DB_URL, {
  maxPoolSize: 10, // Maximum number of connections
  serverSelectionTimeoutMS: 5000, // Keep trying to send operations for 5 seconds
  socketTimeoutMS: 45000, // Close sockets after 45 seconds of inactivity
  family: 4 // Use IPv4, skip trying IPv6
});

// Index optimization
const userSchema = new mongoose.Schema({
  email: { type: String, unique: true, index: true },
  status: { type: String, index: true },
  createdAt: { type: Date, default: Date.now, index: true }
});

// Compound index for common queries
userSchema.index({ status: 1, createdAt: -1 });

// Pagination with cursor-based approach
const getUsers = async (req, res) => {
  const { cursor, limit = 10 } = req.query;
  const query = cursor ? { _id: { $gt: cursor } } : {};
  
  const users = await User.find(query)
    .limit(parseInt(limit))
    .sort({ _id: 1 });
    
  const nextCursor = users.length > 0 ? users[users.length - 1]._id : null;
  
  res.json({ users, nextCursor });
};
```

## Testing Best Practices

### 12. Test Structure & Organization
```js
// tests/unit/userService.test.js
const UserService = require('../../services/userService');

describe('UserService', () => {
  let userService;
  let mockUserRepository;
  
  beforeEach(() => {
    mockUserRepository = {
      create: jest.fn(),
      findByEmail: jest.fn()
    };
    userService = new UserService(mockUserRepository);
  });
  
  describe('createUser', () => {
    it('should create user with valid data', async () => {
      // Arrange
      const userData = { email: 'test@test.com', name: 'Test User' };
      mockUserRepository.create.mockResolvedValue({ id: 1, ...userData });
      
      // Act
      const result = await userService.createUser(userData);
      
      // Assert
      expect(result).toEqual({ id: 1, ...userData });
      expect(mockUserRepository.create).toHaveBeenCalledWith(userData);
    });
    
    it('should throw error for duplicate email', async () => {
      // Arrange
      const userData = { email: 'test@test.com', name: 'Test User' };
      mockUserRepository.findByEmail.mockResolvedValue({ id: 1 });
      
      // Act & Assert
      await expect(userService.createUser(userData))
        .rejects.toThrow('Email already exists');
    });
  });
});
```

## Monitoring & Observability

### 13. Health Checks & Metrics
```js
// health.js
const healthChecks = {
  database: async () => {
    try {
      await mongoose.connection.db.admin().ping();
      return { status: 'healthy', latency: Date.now() };
    } catch (error) {
      return { status: 'unhealthy', error: error.message };
    }
  },
  
  redis: async () => {
    try {
      const start = Date.now();
      await redisClient.ping();
      return { status: 'healthy', latency: Date.now() - start };
    } catch (error) {
      return { status: 'unhealthy', error: error.message };
    }
  }
};

app.get('/health', async (req, res) => {
  const results = {};
  for (const [name, check] of Object.entries(healthChecks)) {
    results[name] = await check();
  }
  
  const isHealthy = Object.values(results).every(r => r.status === 'healthy');
  res.status(isHealthy ? 200 : 503).json({
    status: isHealthy ? 'healthy' : 'unhealthy',
    timestamp: new Date().toISOString(),
    checks: results
  });
});
```

## Common Interview Questions

### Q1: Tại sao nên tránh sync operations trong Node.js?
**Trả lời:**
- Node.js là single-threaded với event loop
- Sync operations block event loop, làm app không thể xử lý requests khác
- Gây degradation performance và poor user experience

```js
// ❌ Blocks event loop
const data = fs.readFileSync('file.txt');

// ✅ Non-blocking
const data = await fs.promises.readFile('file.txt');
```

### Q2: Làm sao organize code trong large Node.js application?
**Trả lời:**
- **Layered Architecture**: Controller → Service → Repository
- **Feature-based structure**: Group by business domain
- **Dependency Injection**: Loose coupling giữa components
- **Separation of concerns**: Mỗi module có single responsibility

### Q3: Best practices for error handling?
**Trả lời:**
- **Centralized error handling**: Global error middleware
- **Custom error classes**: Structured error information
- **Proper logging**: Log errors with context
- **Graceful degradation**: App không crash unexpected

### Q4: Security best practices trong Node.js?
**Trả lời:**
- **Input validation**: Validate & sanitize user input
- **Security headers**: Helmet middleware
- **Rate limiting**: Prevent brute force attacks
- **Authentication**: JWT, proper session management
- **Dependencies**: Keep packages updated, audit vulnerabilities

### Q5: Performance optimization techniques?
**Trả lời:**
- **Caching**: Redis, in-memory cache
- **Database optimization**: Indexes, connection pooling
- **Async operations**: Non-blocking I/O
- **Load balancing**: Multiple instances
- **Monitoring**: APM tools, health checks
