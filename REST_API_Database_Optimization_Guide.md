# REST API DESIGN & DATABASE OPTIMIZATION
## Complete Interview Guide

---

# 📡 PART 1: REST API DESIGN

## 1. REST API Fundamentals

### **What is REST?**
**RE**presentational **S**tate **T**ransfer - một architectural style cho distributed systems.

**6 REST Constraints:**
1. **Client-Server** - Separation of concerns
2. **Stateless** - Mỗi request độc lập, không lưu state
3. **Cacheable** - Response phải định nghĩa có cache được không
4. **Uniform Interface** - Consistent API structure
5. **Layered System** - Client không cần biết có bao nhiêu layers
6. **Code on Demand** (optional) - Server có thể gửi executable code

---

## 2. HTTP Methods & Use Cases

| **Method** | **Purpose** | **Idempotent** | **Safe** | **Request Body** | **Response Body** |
|------------|-------------|----------------|----------|------------------|-------------------|
| **GET** | Retrieve data | ✅ Yes | ✅ Yes | ❌ No | ✅ Yes |
| **POST** | Create new resource | ❌ No | ❌ No | ✅ Yes | ✅ Yes |
| **PUT** | Update/Replace entire resource | ✅ Yes | ❌ No | ✅ Yes | ✅ Yes |
| **PATCH** | Partial update | ❌ No | ❌ No | ✅ Yes | ✅ Yes |
| **DELETE** | Remove resource | ✅ Yes | ❌ No | ❌ Optional | ✅ Optional |
| **HEAD** | Like GET but no body | ✅ Yes | ✅ Yes | ❌ No | ❌ No |
| **OPTIONS** | Get allowed methods | ✅ Yes | ✅ Yes | ❌ No | ✅ Yes |

### **Idempotent là gì?**
Gọi request nhiều lần cho kết quả giống gọi 1 lần.

```javascript
// Idempotent (PUT)
PUT /users/123
{ "name": "John", "age": 30 }
// Gọi 10 lần → user vẫn có name="John", age=30

// NOT Idempotent (POST)
POST /users
{ "name": "John", "age": 30 }
// Gọi 10 lần → tạo 10 users khác nhau
```

---

## 3. URL Design Best Practices

### ✅ **Good URL Design**

```
GET    /api/v1/users                 # Get all users
GET    /api/v1/users/123             # Get user by ID
POST   /api/v1/users                 # Create new user
PUT    /api/v1/users/123             # Replace user
PATCH  /api/v1/users/123             # Update user
DELETE /api/v1/users/123             # Delete user

# Nested resources
GET    /api/v1/users/123/posts       # Get posts of user 123
GET    /api/v1/users/123/posts/456   # Get specific post of user
POST   /api/v1/users/123/posts       # Create post for user 123

# Query parameters for filtering/sorting
GET    /api/v1/users?role=admin&status=active
GET    /api/v1/products?sort=-price&limit=20&page=2
```

### ❌ **Bad URL Design**

```
GET  /api/getUsers              # Verb trong URL
POST /api/user/create           # Verb trong URL
GET  /api/users/delete/123      # Dùng GET cho delete
GET  /api/Users                 # Inconsistent casing
GET  /api/user-management       # Quá dài, không cần thiết
```

### **Naming Conventions:**
- ✅ Use **nouns**, not verbs (`/users` not `/getUsers`)
- ✅ Use **plural** for collections (`/users` not `/user`)
- ✅ Use **lowercase** with hyphens (`/api/user-profiles`)
- ✅ Use **versioning** (`/api/v1/users`)
- ✅ Use **hierarchical structure** for relationships

---

## 4. HTTP Status Codes

### **2xx Success**
```javascript
200 OK                  // GET, PUT, PATCH thành công
201 Created            // POST tạo resource thành công
202 Accepted           // Request được accept nhưng chưa process xong
204 No Content         // DELETE thành công, không trả data
```

### **3xx Redirection**
```javascript
301 Moved Permanently  // Resource đã move vĩnh viễn
304 Not Modified       // Cache vẫn valid, không cần re-download
```

### **4xx Client Errors**
```javascript
400 Bad Request        // Request sai format, validation error
401 Unauthorized       // Chưa authenticate
403 Forbidden          // Đã authenticate nhưng không có permission
404 Not Found          // Resource không tồn tại
409 Conflict           // Conflict với state hiện tại (duplicate key)
422 Unprocessable Entity // Validation error (semantic)
429 Too Many Requests  // Rate limit exceeded
```

### **5xx Server Errors**
```javascript
500 Internal Server Error  // Lỗi không expect được
502 Bad Gateway           // Upstream server error
503 Service Unavailable   // Server đang maintenance
504 Gateway Timeout       // Upstream server timeout
```

---

## 5. Request & Response Design

### **Request Structure**

```javascript
// POST /api/v1/users
// Headers
{
  "Content-Type": "application/json",
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiIs...",
  "X-Request-ID": "unique-request-id",
  "Accept-Language": "en-US"
}

// Body
{
  "email": "john@example.com",
  "name": "John Doe",
  "age": 30,
  "role": "user"
}
```

### **Response Structure (Consistent format)**

```javascript
// ✅ Success Response
{
  "success": true,
  "data": {
    "id": 123,
    "email": "john@example.com",
    "name": "John Doe",
    "age": 30,
    "createdAt": "2024-01-15T10:30:00Z"
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "unique-request-id"
  }
}

// ✅ Error Response
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Email is required"
      },
      {
        "field": "age",
        "message": "Age must be at least 18"
      }
    ]
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "unique-request-id"
  }
}

// ✅ Paginated Response
{
  "success": true,
  "data": [
    { "id": 1, "name": "User 1" },
    { "id": 2, "name": "User 2" }
  ],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 150,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": true
  }
}
```

---

## 6. Pagination, Filtering, Sorting

### **Pagination Strategies**

#### **A. Offset-based (Traditional)**
```javascript
GET /api/v1/users?page=2&limit=20

// Implementation
const page = parseInt(req.query.page) || 1;
const limit = parseInt(req.query.limit) || 20;
const offset = (page - 1) * limit;

const users = await User.findAll({
  limit: limit,
  offset: offset
});

const total = await User.count();

res.json({
  data: users,
  pagination: {
    page,
    limit,
    total,
    totalPages: Math.ceil(total / limit)
  }
});
```

**Pros:** Dễ implement, user có thể jump to specific page  
**Cons:** Performance kém với large datasets, inconsistent nếu data thay đổi

#### **B. Cursor-based (Better for large datasets)**
```javascript
GET /api/v1/users?cursor=abc123&limit=20

// Implementation
const cursor = req.query.cursor;
const limit = parseInt(req.query.limit) || 20;

const users = await User.findAll({
  where: cursor ? { id: { $gt: cursor } } : {},
  limit: limit + 1, // Fetch 1 extra để check hasNext
  order: [['id', 'ASC']]
});

const hasNext = users.length > limit;
const data = hasNext ? users.slice(0, -1) : users;
const nextCursor = hasNext ? data[data.length - 1].id : null;

res.json({
  data,
  pagination: {
    nextCursor,
    hasNext
  }
});
```

**Pros:** Consistent, tốt cho infinite scroll, tốt với large datasets  
**Cons:** Không thể jump to specific page

---

### **Filtering**

```javascript
// Single filter
GET /api/v1/users?role=admin

// Multiple filters
GET /api/v1/users?role=admin&status=active&age=25

// Range filters
GET /api/v1/products?minPrice=100&maxPrice=500

// Advanced filters (JSON)
GET /api/v1/users?filter={"age":{"$gte":18},"status":"active"}

// Implementation
const buildFilter = (query) => {
  const filter = {};
  
  if (query.role) filter.role = query.role;
  if (query.status) filter.status = query.status;
  if (query.minPrice || query.maxPrice) {
    filter.price = {};
    if (query.minPrice) filter.price.$gte = parseFloat(query.minPrice);
    if (query.maxPrice) filter.price.$lte = parseFloat(query.maxPrice);
  }
  
  return filter;
};

const filter = buildFilter(req.query);
const users = await User.findAll({ where: filter });
```

---

### **Sorting**

```javascript
// Single sort
GET /api/v1/users?sort=createdAt        // Ascending
GET /api/v1/users?sort=-createdAt       // Descending (- prefix)

// Multiple sort
GET /api/v1/users?sort=-role,createdAt  // Sort by role DESC, then createdAt ASC

// Implementation
const parseSort = (sortString) => {
  if (!sortString) return [['createdAt', 'DESC']]; // Default
  
  return sortString.split(',').map(field => {
    if (field.startsWith('-')) {
      return [field.substring(1), 'DESC'];
    }
    return [field, 'ASC'];
  });
};

const order = parseSort(req.query.sort);
const users = await User.findAll({ order });
```

---

### **Field Selection (Sparse Fieldsets)**

```javascript
// Select specific fields
GET /api/v1/users?fields=id,name,email

// Implementation
const parseFields = (fieldsString) => {
  if (!fieldsString) return undefined;
  return fieldsString.split(',');
};

const attributes = parseFields(req.query.fields);
const users = await User.findAll({ attributes });

// Response
{
  "data": [
    { "id": 1, "name": "John", "email": "john@example.com" },
    { "id": 2, "name": "Jane", "email": "jane@example.com" }
  ]
}
```

---

## 7. Authentication & Authorization

### **Authentication Methods**

#### **A. JWT (JSON Web Token)**
```javascript
// Login endpoint
POST /api/v1/auth/login
{
  "email": "user@example.com",
  "password": "password123"
}

// Response
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 3600,
    "user": {
      "id": 123,
      "email": "user@example.com",
      "role": "user"
    }
  }
}

// Middleware implementation
const authenticateJWT = (req, res, next) => {
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ 
      error: 'No token provided' 
    });
  }
  
  const token = authHeader.substring(7);
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(403).json({ 
      error: 'Invalid or expired token' 
    });
  }
};

// Usage
app.get('/api/v1/profile', authenticateJWT, async (req, res) => {
  const user = await User.findById(req.user.id);
  res.json({ data: user });
});
```

#### **B. API Key**
```javascript
// Header-based
GET /api/v1/data
Headers: { "X-API-Key": "your-api-key-here" }

// Middleware
const authenticateAPIKey = async (req, res, next) => {
  const apiKey = req.headers['x-api-key'];
  
  if (!apiKey) {
    return res.status(401).json({ error: 'API key required' });
  }
  
  const validKey = await APIKey.findOne({ 
    key: apiKey, 
    isActive: true 
  });
  
  if (!validKey) {
    return res.status(403).json({ error: 'Invalid API key' });
  }
  
  req.apiKey = validKey;
  next();
};
```

### **Authorization (Role-Based Access Control)**

```javascript
// Authorization middleware
const authorize = (...allowedRoles) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Not authenticated' });
    }
    
    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({ 
        error: 'Insufficient permissions' 
      });
    }
    
    next();
  };
};

// Usage
app.delete('/api/v1/users/:id', 
  authenticateJWT,
  authorize('admin', 'superadmin'),
  deleteUser
);

app.get('/api/v1/posts', 
  authenticateJWT,
  authorize('user', 'admin', 'superadmin'),
  getPosts
);
```

---

## 8. Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');

// Basic rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP',
  standardHeaders: true, // Return rate limit info in headers
  legacyHeaders: false,
});

app.use('/api/', limiter);

// Advanced with Redis (for distributed systems)
const redisLimiter = rateLimit({
  store: new RedisStore({
    client: redisClient,
    prefix: 'rate_limit:',
  }),
  windowMs: 15 * 60 * 1000,
  max: 100,
  handler: (req, res) => {
    res.status(429).json({
      error: 'Too many requests',
      retryAfter: req.rateLimit.resetTime
    });
  }
});

// Different limits for different endpoints
const strictLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 10,
});

app.post('/api/v1/auth/login', strictLimiter, login);
app.post('/api/v1/auth/register', strictLimiter, register);

// Headers in response
// X-RateLimit-Limit: 100
// X-RateLimit-Remaining: 95
// X-RateLimit-Reset: 1705327800
```

---

## 9. API Versioning

### **Strategy 1: URL Versioning (Recommended)**
```javascript
GET /api/v1/users
GET /api/v2/users

// Implementation
const v1Router = express.Router();
const v2Router = express.Router();

v1Router.get('/users', getUsersV1);
v2Router.get('/users', getUsersV2);

app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);
```

### **Strategy 2: Header Versioning**
```javascript
GET /api/users
Headers: { "Accept-Version": "v2" }

// Middleware
app.use((req, res, next) => {
  const version = req.headers['accept-version'] || 'v1';
  req.apiVersion = version;
  next();
});

app.get('/api/users', (req, res) => {
  if (req.apiVersion === 'v2') {
    return getUsersV2(req, res);
  }
  return getUsersV1(req, res);
});
```

### **Strategy 3: Content Negotiation**
```javascript
GET /api/users
Headers: { "Accept": "application/vnd.myapi.v2+json" }
```

---

## 10. Caching

### **A. HTTP Cache Headers**

```javascript
// Cache-Control
app.get('/api/v1/products', (req, res) => {
  const products = await Product.findAll();
  
  res.set({
    'Cache-Control': 'public, max-age=300', // Cache 5 minutes
    'ETag': generateETag(products),
  });
  
  res.json({ data: products });
});

// ETag validation
app.get('/api/v1/products/:id', async (req, res) => {
  const product = await Product.findById(req.params.id);
  const etag = `"${product.id}-${product.updatedAt.getTime()}"`;
  
  if (req.headers['if-none-match'] === etag) {
    return res.status(304).end(); // Not Modified
  }
  
  res.set('ETag', etag);
  res.json({ data: product });
});
```

### **B. Redis Caching**

```javascript
const redis = require('redis');
const client = redis.createClient();

// Cache middleware
const cacheMiddleware = (duration) => async (req, res, next) => {
  const key = `cache:${req.originalUrl}`;
  
  try {
    const cached = await client.get(key);
    
    if (cached) {
      return res.json(JSON.parse(cached));
    }
    
    // Override res.json to cache response
    const originalJson = res.json.bind(res);
    res.json = (data) => {
      client.setEx(key, duration, JSON.stringify(data));
      return originalJson(data);
    };
    
    next();
  } catch (error) {
    next();
  }
};

// Usage
app.get('/api/v1/products', 
  cacheMiddleware(300), // Cache 5 minutes
  async (req, res) => {
    const products = await Product.findAll();
    res.json({ data: products });
  }
);

// Cache invalidation
app.put('/api/v1/products/:id', async (req, res) => {
  const product = await Product.update(req.params.id, req.body);
  
  // Invalidate related caches
  await client.del('cache:/api/v1/products');
  await client.del(`cache:/api/v1/products/${req.params.id}`);
  
  res.json({ data: product });
});
```

---

## 11. Error Handling

```javascript
// Custom error class
class APIError extends Error {
  constructor(message, statusCode, errorCode) {
    super(message);
    this.statusCode = statusCode;
    this.errorCode = errorCode;
    this.isOperational = true;
  }
}

// Error types
class ValidationError extends APIError {
  constructor(details) {
    super('Validation Error', 400, 'VALIDATION_ERROR');
    this.details = details;
  }
}

class NotFoundError extends APIError {
  constructor(resource) {
    super(`${resource} not found`, 404, 'NOT_FOUND');
  }
}

class UnauthorizedError extends APIError {
  constructor(message = 'Unauthorized') {
    super(message, 401, 'UNAUTHORIZED');
  }
}

// Usage in controllers
const getUser = async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    
    if (!user) {
      throw new NotFoundError('User');
    }
    
    res.json({ data: user });
  } catch (error) {
    next(error);
  }
};

// Global error handler (must be last)
app.use((err, req, res, next) => {
  // Log error
  console.error('Error:', {
    message: err.message,
    stack: err.stack,
    requestId: req.id,
  });
  
  // Operational errors (known errors)
  if (err.isOperational) {
    return res.status(err.statusCode).json({
      success: false,
      error: {
        code: err.errorCode,
        message: err.message,
        details: err.details || undefined,
      }
    });
  }
  
  // Programming errors (unknown errors)
  res.status(500).json({
    success: false,
    error: {
      code: 'INTERNAL_ERROR',
      message: process.env.NODE_ENV === 'production' 
        ? 'Internal server error' 
        : err.message,
    }
  });
});
```

---

## 12. Documentation (OpenAPI/Swagger)

```javascript
const swaggerJsdoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'My API',
      version: '1.0.0',
      description: 'API documentation',
    },
    servers: [
      {
        url: 'http://localhost:3000/api/v1',
        description: 'Development server',
      },
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT',
        },
      },
    },
  },
  apis: ['./routes/*.js'],
};

const specs = swaggerJsdoc(options);
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(specs));

/**
 * @swagger
 * /users:
 *   get:
 *     summary: Get all users
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *         description: Page number
 *     responses:
 *       200:
 *         description: Success
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/User'
 */
app.get('/users', getUsers);
```

---

# 💾 PART 2: DATABASE OPTIMIZATION

## 1. MySQL Optimization

### **A. Indexing Strategies**

#### **Types of Indexes**

```sql
-- 1. PRIMARY KEY (Clustered Index)
CREATE TABLE users (
  id INT PRIMARY KEY AUTO_INCREMENT,
  email VARCHAR(255) NOT NULL
);

-- 2. UNIQUE Index
CREATE UNIQUE INDEX idx_email ON users(email);

-- 3. Regular Index (B-Tree)
CREATE INDEX idx_lastname ON users(last_name);

-- 4. Composite Index
CREATE INDEX idx_name_age ON users(last_name, first_name, age);

-- 5. Full-Text Index (for text search)
CREATE FULLTEXT INDEX idx_description ON products(description);

-- 6. Spatial Index (for geo data)
CREATE SPATIAL INDEX idx_location ON stores(coordinates);
```

#### **When to Use Indexes**

```sql
-- ✅ Good for indexes
SELECT * FROM users WHERE email = 'john@example.com';  -- WHERE clause
SELECT * FROM orders ORDER BY created_at DESC;         -- ORDER BY
SELECT * FROM products WHERE category_id = 5;          -- Foreign key

-- ❌ Bad for indexes (won't use index efficiently)
SELECT * FROM users WHERE YEAR(created_at) = 2024;    -- Function on column
SELECT * FROM products WHERE name LIKE '%phone%';     -- Leading wildcard
SELECT * FROM users WHERE age + 10 > 30;              -- Expression on column
```

#### **Composite Index - Column Order Matters**

```sql
-- Index: (last_name, first_name, age)

-- ✅ Uses index (leftmost prefix)
SELECT * FROM users WHERE last_name = 'Smith';
SELECT * FROM users WHERE last_name = 'Smith' AND first_name = 'John';
SELECT * FROM users WHERE last_name = 'Smith' AND first_name = 'John' AND age = 30;

-- ❌ Does NOT use index (missing leftmost column)
SELECT * FROM users WHERE first_name = 'John';
SELECT * FROM users WHERE age = 30;
SELECT * FROM users WHERE first_name = 'John' AND age = 30;
```

#### **Covering Index**

```sql
-- Query chỉ cần columns trong index
CREATE INDEX idx_covering ON users(email, name, age);

-- ✅ Covering index - không cần truy cập table
SELECT email, name, age FROM users WHERE email = 'john@example.com';

-- ❌ Not covering - phải truy cập table để lấy address
SELECT email, name, age, address FROM users WHERE email = 'john@example.com';
```

#### **Check Index Usage**

```sql
-- Xem query có dùng index không
EXPLAIN SELECT * FROM users WHERE email = 'john@example.com';

-- Output columns:
-- type: ALL (bad - full table scan), index (ok), ref (good), eq_ref (best)
-- possible_keys: Indexes có thể dùng
-- key: Index thực tế được dùng
-- rows: Số rows MySQL estimate phải scan

-- Analyze index usage
SHOW INDEX FROM users;
SELECT * FROM sys.schema_unused_indexes; -- Find unused indexes
```

---

### **B. Query Optimization**

#### **1. Avoid SELECT ***

```sql
-- ❌ Bad - fetches all columns
SELECT * FROM users WHERE id = 123;

-- ✅ Good - only fetch needed columns
SELECT id, email, name FROM users WHERE id = 123;
```

#### **2. Use LIMIT**

```sql
-- ❌ Bad - fetches all rows
SELECT * FROM orders ORDER BY created_at DESC;

-- ✅ Good - limit results
SELECT * FROM orders ORDER BY created_at DESC LIMIT 20;
```

#### **3. Optimize JOINs**

```sql
-- ❌ Bad - no indexes on join columns
SELECT u.name, o.total
FROM users u
JOIN orders o ON u.id = o.user_id;  -- Slow if no index on user_id

-- ✅ Good - indexes on join columns
CREATE INDEX idx_user_id ON orders(user_id);

SELECT u.name, o.total
FROM users u
JOIN orders o ON u.id = o.user_id;
```

#### **4. Avoid N+1 Query Problem**

```javascript
// ❌ Bad - N+1 queries
const users = await User.findAll(); // 1 query
for (const user of users) {
  const posts = await Post.findAll({ where: { userId: user.id } }); // N queries
}

// ✅ Good - 1 query with JOIN
const users = await User.findAll({
  include: [{ model: Post }]
});

// SQL generated:
SELECT u.*, p.*
FROM users u
LEFT JOIN posts p ON u.id = p.user_id;
```

#### **5. Use EXISTS instead of COUNT**

```sql
-- ❌ Bad - counts all rows (slow)
SELECT COUNT(*) > 0 FROM orders WHERE user_id = 123;

-- ✅ Good - stops at first match
SELECT EXISTS(SELECT 1 FROM orders WHERE user_id = 123);
```

#### **6. Batch Operations**

```sql
-- ❌ Bad - multiple INSERT statements
INSERT INTO users (name, email) VALUES ('John', 'john@example.com');
INSERT INTO users (name, email) VALUES ('Jane', 'jane@example.com');
INSERT INTO users (name, email) VALUES ('Bob', 'bob@example.com');

-- ✅ Good - single batch INSERT
INSERT INTO users (name, email) VALUES 
  ('John', 'john@example.com'),
  ('Jane', 'jane@example.com'),
  ('Bob', 'bob@example.com');
```

---

### **C. Connection Pooling**

```javascript
const mysql = require('mysql2/promise');

// Create connection pool
const pool = mysql.createPool({
  host: 'localhost',
  user: 'root',
  password: 'password',
  database: 'mydb',
  waitForConnections: true,
  connectionLimit: 10,        // Max connections
  maxIdle: 10,                // Max idle connections
  idleTimeout: 60000,         // Idle timeout (ms)
  queueLimit: 0,              // Unlimited queue
  enableKeepAlive: true,
  keepAliveInitialDelay: 0
});

// Usage
const getUsers = async () => {
  const connection = await pool.getConnection();
  
  try {
    const [rows] = await connection.query('SELECT * FROM users');
    return rows;
  } finally {
    connection.release(); // Return to pool
  }
};

// Or use pool directly (auto release)
const getUser = async (id) => {
  const [rows] = await pool.query('SELECT * FROM users WHERE id = ?', [id]);
  return rows[0];
};
```

---

### **D. Partitioning**

```sql
-- Partition by RANGE (good for time-series data)
CREATE TABLE orders (
  id INT,
  user_id INT,
  total DECIMAL(10,2),
  created_at DATE
)
PARTITION BY RANGE (YEAR(created_at)) (
  PARTITION p2022 VALUES LESS THAN (2023),
  PARTITION p2023 VALUES LESS THAN (2024),
  PARTITION p2024 VALUES LESS THAN (2025),
  PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- Query only scans relevant partition
SELECT * FROM orders WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31';
-- Only scans p2024 partition

-- Partition by HASH (distribute data evenly)
CREATE TABLE logs (
  id INT,
  message TEXT,
  created_at TIMESTAMP
)
PARTITION BY HASH(id)
PARTITIONS 4;
```

---

### **E. Denormalization (When Needed)**

```sql
-- Normalized (requires JOIN)
-- orders table
id | user_id | total
1  | 123     | 100.00

-- users table
id  | name
123 | John Doe

-- Query requires JOIN
SELECT o.*, u.name FROM orders o JOIN users u ON o.user_id = u.id;

-- Denormalized (faster reads, slower writes)
-- orders table
id | user_id | user_name | total
1  | 123     | John Doe  | 100.00

-- No JOIN needed
SELECT * FROM orders;
```

**Trade-offs:**
- ✅ Faster reads (no JOINs)
- ❌ Data duplication
- ❌ Update anomalies (must update in multiple places)
- Use for: Read-heavy tables, reporting, analytics

---

## 2. MongoDB Optimization

### **A. Indexing in MongoDB**

```javascript
// 1. Single Field Index
db.users.createIndex({ email: 1 });  // 1 = ascending, -1 = descending

// 2. Compound Index
db.users.createIndex({ lastName: 1, firstName: 1, age: 1 });

// 3. Unique Index
db.users.createIndex({ email: 1 }, { unique: true });

// 4. Sparse Index (only index documents that have the field)
db.users.createIndex({ phone: 1 }, { sparse: true });

// 5. TTL Index (auto-delete after time)
db.sessions.createIndex(
  { createdAt: 1 }, 
  { expireAfterSeconds: 3600 }  // Delete after 1 hour
);

// 6. Text Index (full-text search)
db.products.createIndex({ description: "text", name: "text" });

// Search
db.products.find({ $text: { $search: "laptop gaming" } });

// 7. Geospatial Index
db.stores.createIndex({ location: "2dsphere" });

// Find nearby
db.stores.find({
  location: {
    $near: {
      $geometry: { type: "Point", coordinates: [-73.97, 40.77] },
      $maxDistance: 5000  // 5km
    }
  }
});

// View indexes
db.users.getIndexes();

// Analyze query performance
db.users.find({ email: "john@example.com" }).explain("executionStats");
```

---

### **B. Query Optimization**

#### **1. Use Projection (select fields)**

```javascript
// ❌ Bad - returns all fields
db.users.find({ status: "active" });

// ✅ Good - only return needed fields
db.users.find(
  { status: "active" },
  { name: 1, email: 1, _id: 0 }  // 1 = include, 0 = exclude
);
```

#### **2. Use Covered Queries**

```javascript
// Create index
db.users.createIndex({ email: 1, name: 1, age: 1 });

// ✅ Covered query - data from index only
db.users.find(
  { email: "john@example.com" },
  { email: 1, name: 1, age: 1, _id: 0 }
);
// totalDocsExamined: 0 (didn't scan documents)
```

#### **3. Avoid Large Skip**

```javascript
// ❌ Bad - slow for large skip values
db.products.find().skip(10000).limit(20);

// ✅ Good - use range queries
db.products.find({ _id: { $gt: lastSeenId } }).limit(20);
```

#### **4. Aggregation Pipeline Optimization**

```javascript
// ❌ Bad - $match at the end
db.orders.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "userId",
      foreignField: "_id",
      as: "user"
    }
  },
  { $match: { status: "completed" } }  // Should be first!
]);

// ✅ Good - $match first to reduce documents
db.orders.aggregate([
  { $match: { status: "completed" } },  // Filter first
  {
    $lookup: {
      from: "users",
      localField: "userId",
      foreignField: "_id",
      as: "user"
    }
  }
]);

// Use $project early to reduce data size
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $project: { userId: 1, total: 1, createdAt: 1 } },  // Only needed fields
  { $group: { _id: "$userId", totalSpent: { $sum: "$total" } } }
]);
```

---

### **C. Schema Design Patterns**

#### **1. Embedding vs Referencing**

**Embedding (Denormalized):**
```javascript
// ✅ Good for: One-to-Few, data read together
{
  _id: 1,
  name: "John Doe",
  addresses: [
    { street: "123 Main St", city: "NYC" },
    { street: "456 Oak Ave", city: "LA" }
  ]
}

// Pros: Single query, atomic updates
// Cons: Document size limit (16MB), data duplication
```

**Referencing (Normalized):**
```javascript
// ✅ Good for: One-to-Many, data accessed separately
// users collection
{ _id: 1, name: "John Doe" }

// posts collection
{ _id: 101, userId: 1, title: "Post 1" }
{ _id: 102, userId: 1, title: "Post 2" }

// Pros: No duplication, smaller documents
// Cons: Multiple queries or $lookup
```

#### **2. Extended Reference Pattern**

```javascript
// Store most-used fields, reference for details
{
  _id: 1,
  title: "Laptop",
  manufacturer: {
    id: 123,
    name: "Dell",  // Denormalized for quick access
    // Full details in manufacturers collection
  }
}
```

#### **3. Bucket Pattern (Time-Series)**

```javascript
// ❌ Bad - one document per reading
{ sensorId: 1, temp: 20, time: "2024-01-01T10:00:00Z" }
{ sensorId: 1, temp: 21, time: "2024-01-01T10:01:00Z" }
// Millions of small documents

// ✅ Good - bucket by hour
{
  sensorId: 1,
  date: "2024-01-01",
  hour: 10,
  readings: [
    { minute: 0, temp: 20 },
    { minute: 1, temp: 21 },
    // ... 60 readings
  ]
}
// Fewer, larger documents
```

---

### **D. Connection Pooling**

```javascript
const { MongoClient } = require('mongodb');

const uri = 'mongodb://localhost:27017';
const client = new MongoClient(uri, {
  maxPoolSize: 50,        // Max connections
  minPoolSize: 10,        // Min connections
  maxIdleTimeMS: 30000,   // Close idle connections after 30s
  serverSelectionTimeoutMS: 5000,
  socketTimeoutMS: 45000,
});

await client.connect();

const db = client.db('mydb');
const users = db.collection('users');

// Pool automatically manages connections
const getUsers = async () => {
  return await users.find({ status: 'active' }).toArray();
};
```

---

### **E. Sharding (Horizontal Scaling)**

```javascript
// Enable sharding on database
sh.enableSharding("mydb");

// Shard collection by key
sh.shardCollection("mydb.users", { userId: "hashed" });

// Queries with shard key are fast (single shard)
db.users.find({ userId: 123 });

// Queries without shard key are slow (all shards)
db.users.find({ email: "john@example.com" });  // Scatter-gather

// Choose shard key wisely:
// ✅ High cardinality (many unique values)
// ✅ Even distribution
// ✅ Used in most queries
```

---

## 3. General Database Best Practices

### **A. Use Transactions (ACID)**

**MySQL:**
```javascript
const connection = await pool.getConnection();
await connection.beginTransaction();

try {
  await connection.query(
    'UPDATE accounts SET balance = balance - 100 WHERE id = 1'
  );
  await connection.query(
    'UPDATE accounts SET balance = balance + 100 WHERE id = 2'
  );
  
  await connection.commit();
} catch (error) {
  await connection.rollback();
  throw error;
} finally {
  connection.release();
}
```

**MongoDB:**
```javascript
const session = client.startSession();

try {
  await session.withTransaction(async () => {
    await accounts.updateOne(
      { _id: 1 },
      { $inc: { balance: -100 } },
      { session }
    );
    await accounts.updateOne(
      { _id: 2 },
      { $inc: { balance: 100 } },
      { session }
    );
  });
} finally {
  await session.endSession();
}
```

---

### **B. Database Migrations**

```javascript
// Use migration tools
// Prisma example
npx prisma migrate dev --name add_user_age

// Sequelize example
npx sequelize-cli migration:generate --name add-user-age

// Migration file
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.addColumn('users', 'age', {
      type: Sequelize.INTEGER,
      allowNull: true,
    });
    
    await queryInterface.addIndex('users', ['age']);
  },
  
  down: async (queryInterface) => {
    await queryInterface.removeColumn('users', 'age');
  }
};
```

---

### **C. Monitoring & Alerting**

```javascript
// Log slow queries
// MySQL
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;  // Log queries > 1 second

// MongoDB
db.setProfilingLevel(1, { slowms: 100 });  // Log queries > 100ms

// View slow queries
db.system.profile.find().sort({ ts: -1 }).limit(10);

// Monitor connection pool
setInterval(() => {
  console.log('Pool stats:', {
    active: pool._allConnections.length,
    idle: pool._freeConnections.length,
  });
}, 60000);
```

---

## 🎯 INTERVIEW QUESTIONS & ANSWERS

### **Q1: Khi nào dùng SQL, khi nào dùng NoSQL?**

**A:**
- **SQL (MySQL, PostgreSQL):** Khi cần ACID transactions, complex queries, structured data, strong consistency
- **NoSQL (MongoDB):** Khi cần horizontal scaling, flexible schema, high write throughput, eventual consistency OK

**Example:** Banking → SQL, Real-time chat → NoSQL

---

### **Q2: Explain N+1 query problem và cách fix?**

**A:** 
```javascript
// Problem: 1 query cho users, N queries cho posts
const users = await User.findAll();
for (const user of users) {
  user.posts = await Post.findAll({ userId: user.id });
}

// Solution: Use JOIN/Include
const users = await User.findAll({
  include: [Post]
});
```

---

### **Q3: Index hoạt động như thế nào?**

**A:** Index là data structure (B-Tree) lưu trữ sorted pointers đến data.
- **Without index:** Full table scan O(n)
- **With index:** Binary search O(log n)
- **Trade-off:** Faster reads, slower writes (must update index)

---

### **Q4: Caching strategies?**

**A:**
1. **Cache-Aside:** App checks cache → miss → load from DB → store in cache
2. **Write-Through:** Write to cache + DB simultaneously
3. **Write-Behind:** Write to cache → async write to DB

---

### **Q5: Database connection pooling là gì?**

**A:** Tái sử dụng database connections thay vì tạo mới mỗi request.
- **Benefits:** Giảm latency, giảm overhead, handle nhiều requests hơn
- **Config:** Pool size, idle timeout, max queue

---

**🎓 Good luck với interview!**
